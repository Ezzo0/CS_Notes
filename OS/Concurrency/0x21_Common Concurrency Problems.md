# Explanation
##### Non-Deadlock Bugs
- We now discuss the two major types of non-deadlock bugs: 
1. **Atomicity violation bugs**:
	- Here is a simple example, found in MySQL.
	- In the example, two different threads access the field `proc_info` in the structure `thd`. The first thread checks if the value is non-NULL and then prints its value; the second thread sets it to NULL.
	- If the first thread performs the check but then is interrupted before the call to `fputs`, the second thread could run in-between, thus setting the pointer to NULL; when the first thread resumes, it will crash, as a NULL pointer will be dereferenced by `fputs`.
	- This type of bugs is solved using [[0x18_Locks|Locks]]
```MYSQL
Thread 1::
if (thd->proc_info) {
fputs(thd->proc_info, ...);
}

Thread 2::
thd->proc_info = NULL;
```
2. **Order violation bugs**:
	-   Here is a simple example
	- As you probably figured out, the code in Thread 2 seems to assume that the variable `mThread` has already been initialized.
	- However, if Thread 2 runs immediately once created, the value of `mThread` will not be set when it is accessed within `mMain()` in Thread 2, and will likely crash with a NULL-pointer dereference.
	- This type of bugs is solved using [[0x19_Condition Variables|Condition Variables]]
```MYSQL
Thread 1::
void init() {
	mThread = PR_CreateThread(mMain, ...);
}

Thread 2::
void mMain(...) {
	mState = mThread->State;
}
```
##### Deadlock Bugs
- Deadlock occurs, for example, when a thread (say Thread 1) is holding a lock (L1) and waiting for another one (L2); unfortunately, the thread (Thread 2) that holds lock L2 is waiting for L1 to be released.
- Here is a code snippet that demonstrates such a potential deadlock:
 ![[Simple Deadlock.png]]![[Deadlock Dependency Graph.png]]
- **Conditions for Deadlock:**
	- **Mutual exclusion**: Threads claim exclusive control of resources that they require (e.g., a thread grabs a lock).
	- **Hold-and-wait**: Threads hold resources allocated to them (e.g., locks that they have already acquired) while waiting for additional resources (e.g., locks that they wish to acquire).
	- **No preemption**: Resources (e.g., locks) cannot be forcibly removed from threads that are holding them.
	- **Circular wait**: There exists a circular chain of threads such that each thread holds one or more resources (e.g., locks) that are being requested by the next thread in the chain.
- If any of these four conditions are not met, deadlock cannot occur.
##### Deadlock Prevention
1. **Circular Wait:**
	- Probably the most practical prevention technique is to write your locking code such that you never induce a circular wait. The most straightforward way to do that is to provide a **total ordering** on lock acquisition. 
	- For example, if there are only two locks in the system (L1 and L2), you can prevent deadlock by always acquiring L1 before L2.
	- Of course, in more complex systems, more than two locks will exist, and thus total lock ordering may be difficult to achieve. Thus, a **partial ordering** can be a useful way to structure lock acquisition so as to avoid deadlock.
2. **Hold-and-wait:**
	- The hold-and-wait requirement for deadlock can be avoided by acquiring all locks at once, atomically (as in below code).
	- Note that the solution is problematic for a number of reasons. 
		- when calling a routine, this approach requires us to know exactly which locks must be held and to acquire them ahead of time. 
		- This technique also is likely to decrease concurrency as all locks must be acquired early on (at once) instead of when they are truly needed.
```C
pthread_mutex_lock(prevention);   // begin acquisition
pthread_mutex_lock(L1);
pthread_mutex_lock(L2);
...
pthread_mutex_unlock(prevention); // end
```
3. **No Preemption**
	- Because we generally view locks as held until unlock is called, multiple lock acquisition often gets us into trouble because when waiting for one lock we are holding another. Many thread libraries provide a more flexible set of interfaces to help avoid this situation.
	- Specifically, the routine `pthread_mutex_trylock()` either grabs the lock (if it is available) and returns success or returns an error code indicating the lock is held; in the latter case, you can try again later if you want to grab that lock.
	- Note that another thread could follow the same protocol but grab the locks in the other order (L2 then L1) and the program would still be dead- lock free. One new problem does arise, however: **livelock**.
	- It is possible (though perhaps unlikely) that two threads could both be repeatedly attempting this sequence and repeatedly failing to acquire both locks. 
	- In this case, both systems are running through this code sequence over and over again (and thus it is not a deadlock), but progress is not being made, hence the name **livelock**.
	- There are solutions to the livelock problem, too: for example, one could add a random delay before looping back and trying the entire thing over again, thus decreasing the odds of repeated interference among competing threads.
```C
top:
	pthread_mutex_lock(L1);
	if (pthread_mutex_trylock(L2) != 0) {
		pthread_mutex_unlock(L1);
		goto top;
	}
```
4. **Mutual Exclusion**
	- One could design various data structures without locks at all. 
	- The idea behind these **lock-free** (and related **wait-free**) approaches here is simple: using powerful hardware instructions, you can build data structures in a manner that does not require explicit locking.
	- As a simple example, let us assume we have a [[0x18_Locks#Compare-And-Swap|compare-and-swap]] instruction, instead of acquiring a lock, doing the update, and then releasing it, we have instead built an approach that repeatedly tries to update the value to the new amount and uses the compare-and-swap to do so.
	- In this manner, no lock is acquired, and no deadlock can arise (though **livelock** is still a possibility).
	- Let us consider a slightly more complex example: list insertion. let us try to perform this insertion in a lock-free manner simply using the compare-and-swap instruction.
	- The code here updates the next pointer to point to the current head, and then tries to swap the newly-created node into position as the new head of the list. 
	- However, this will fail if some other thread successfully swapped in a new head in the meanwhile, causing this thread to retry again with the new head.
```C
void insert(int value) {
	node_t *n = malloc(sizeof(node_t));
	assert(n != NULL);
	n->value = value;
	do {
		n->next = head;
	} while (CompareAndSwap(&head, n->next, n) == 0);
}
```
##### Deadlock Avoidance via Scheduling
- Avoidance requires some global knowledge of which locks various threads might grab during their execution, and subsequently schedules said threads in a way as to guarantee no deadlock can occur.
- For example, assume we have two processors and four threads which must be scheduled upon them. Assume further we know that Thread 1 (T1) grabs locks L1 and L2 (in some order, at some point during its execution), T2 grabs L1 and L2 as well, T3 grabs just L2, and T4 grabs no locks at all. ![[locks and threads.png]]
- A smart scheduler could thus compute that as long as T1 and T2 are not run at the same time, no deadlock could ever arise. ![[scheduler no deadlock.png]]
# Sources
- Operating Systems: Three Easy Pieces - Chapter 32.
- [Lecture 9 - part 1](https://youtu.be/Fnp_K63ss44)