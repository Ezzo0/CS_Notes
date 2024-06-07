# Explanation
- Programmers annotate source code with **locks**, putting them around [[0x17_Intro to Concurrency#The Heart Of The Problem Uncontrolled Scheduling|critical sections]], and thus ensure that any such critical section executes as if it were a single atomic instruction.
##### Locks: The Basic Idea
- As an example, assume our critical section looks like this, the canonical update of a shared variable: `balance = balance + 1;`
- To use a lock, we add some code around the critical section like this:
```C
lock_t mutex; // some globally-allocated lock ’mutex’
...
lock(&mutex);
balance = balance + 1;
unlock(&mutex)
```
 - A lock is just a variable, and thus to use one, you must declare a **lock variable** of some kind (such as mutex above).  ^f3ec79
 - This lock variable holds the state of the lock at any instant in time. 
 - It is either **available** (or **unlocked** or **free**) and thus no thread holds the lock, or **acquired** (or **locked** or **held**), and thus exactly one thread holds the lock and presumably is in a critical section.
 - Other information could be stored in the data type as well, such as which thread holds the lock, or a queue for ordering lock acquisition, but information like that is hidden from the user of the lock.
 - The semantics of the `lock()` and `unlock()` routines are simple.
	 - Calling the routine `lock()` tries to acquire the lock; if no other thread holds the lock, the thread will acquire the lock and enter the critical section; this thread is sometimes said to be the **owner** of the lock. 
	 - If another thread then calls `lock()` on that same lock variable, it will not return while the lock is held by another thread.
##### Pthread Locks
```C
pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER;

Pthread_mutex_lock(&lock); // wrapper; exits on failure
balance = balance + 1;
Pthread_mutex_unlock(&lock);
```
- The POSIX version passes a variable to lock and unlock, as we may be using different locks to protect different variables. 
- Doing so can increase concurrency: instead of one big lock that is used any time any critical section is accessed (a **coarse-grained** locking strategy), one will often protect different data and data structures with different locks, thus allowing more threads to be in locked code at once (a more **fine-grained** approach).
##### Controlling Interrupts
- One of the earliest solutions used to provide mutual exclusion was to disable interrupts for critical sections; this solution was invented for single-processor systems.
- The main positive of this approach is its simplicity, but The negatives, unfortunately, are many:
	- First, this approach requires to allow any calling thread to perform a **privileged** operation (turning interrupts on and off), and thus trust that this facility is not abused.
	- Second, if **multiple threads** are running on different CPUs, and each try to enter the same critical section, it does not matter whether interrupts are disabled; threads will be able to **run on other processors**, and thus could enter the critical section.
	- Third, turning off interrupts for extended periods of time can lead to **interrupts becoming lost**, which can lead to serious systems problems. Imagine, for example, if the CPU missed the fact that a disk device has finished a read request. How will the OS know to wake the process waiting for said read?
##### A Failed Attempt: Just Using Loads/Stores
```C
typedef struct __lock_t { int flag; } lock_t;

void init(lock_t *mutex) {
// 0 -> lock is available, 1 -> held
mutex->flag = 0;
}

void unlock(lock_t *mutex) {
mutex->flag = 0;
}
```
- The idea is using a simple flag to indicate whether some thread has possession of a lock.
- The first thread that enters the critical section will call `lock()`, which **tests** whether the flag is equal to 1 (in this case, it is not), and then **sets** the flag to 1 to indicate that the thread now **holds** the lock. 
- When finished with the critical section, the thread calls `unlock()` and clears the flag, thus indicating that the lock is **no longer** held.
- If another thread happens to call `lock()` while that first thread is in the critical section, it will simply **spin-wait** in the while loop for that thread to call `unlock()` and clear the flag.
- Unfortunately, the code has two problems: one of correctness, and another of performance.
	1. The correctness problem:
		- With interrupts, a case is produced where both threads set the flag to 1 and both threads are thus able to enter the critical section.
		 ![[No Mutual Exclusion.png]]
	2. The performance problem:
		- Spin-waiting wastes time waiting for another thread to release a lock. 
		- The waste is exceptionally high on a uniprocessor, where the thread that the waiter is waiting for cannot even run (at least, until a context switch occurs).
##### Building Working Spin Locks with Test-And-Set 
- The simplest bit of hardware support to understand is known as a **test-and-set** (or **atomic exchange**) instruction. We define what the test- and-set instruction does via the following C code snippet:
```C
int TestAndSet(int *old_ptr, int new) {
int old = *old_ptr; // fetch old value at old_ptr
*old_ptr = new;     // store ’new’ into old_ptr
return old;         // return the old value

/*********** This was in lecture ***********/
/*
int old;
asm volatile("lock; xchgl %0, %1" : "+m" (*old_ptr), "=a" (old) : "1" (new) : "cc")
return old;
*/
}
```
- `TestAndSet` returns the old value pointed to by the `old_ptr`, and simultaneously updates said value to `new`. The key, of course, is that this sequence of operations is performed **atomically**.
- As it turns out, this slightly more powerful instruction is enough to build a simple **spin lock**:
```C
typedef struct __lock_t {
	int flag;
} lock_t;

void init(lock_t *lock) {
	// 0: lock is available, 1: lock is held
	lock->flag = 0;
}

void lock(lock_t *lock) {
	while (TestAndSet(&lock->flag, 1) == 1)
		; // spin-wait (do nothing)
}

void unlock(lock_t *lock) {
	lock->flag = 0;
}
```
- Unfortunately, spin locks don’t provide any fairness guarantees. Indeed, a thread spinning may spin **forever**, under contention. Simple spin locks (as discussed thus far) are not fair and may lead to **starvation**.
- For spin locks, in the single CPU case, performance overheads can be quite painful.
	- imagine the case where the thread holding the lock is preempted within a critical section. 
	- The scheduler might then run every other thread (imagine there are N − 1 others), each of which tries to acquire the lock. In this case, each of those threads will spin for the duration of a time slice before giving up the CPU, a waste of CPU cycles.
##### Compare-And-Swap
- Another hardware primitive that some systems provide is known as the **compare-and-swap** instruction (as it is called on SPARC, for example), or **compare-and-exchange** (as it called on x86).
```C
char compare_and_swap(int *ptr, int old, int new) {
    unsigned char ret;
    // Note that sete sets a ’byte’ not the word
    __asm__ __volatile__ (
	" lock\n"
	" cmpxchgl %2,%1\n"
	" sete %0\n"
	: "=q" (ret), "=m" (*ptr)
	: "r" (new), "m" (*ptr), "a" (old)
	: "memory");
    return ret;
}
```
- The basic idea is for compare-and-swap to test whether the value at the address specified by `ptr` is equal to expected; 
	- if so, update the memory location pointed to by `ptr` with the new value. 
	- If not, do nothing. 
	- In either case, return the original value at that memory location, thus allowing the code calling compare-and-swap to know whether it succeeded or not.
##### Load-Linked and Store-Conditional
- The load-linked operates much like a typical load instruction, and simply fetches a value from memory and places it in a register. 
- The key difference comes with the store-conditional, which only succeeds (and updates the value stored at the address just load-linked from) if no intervening store to the address has taken place. 
- In the case of success, the store-conditional returns 1 and updates the value at `ptr` to `value`; if it fails, the value at `ptr` is not updated and 0 is returned.
- The C pseudocode for these instructions is:
```pseudocode
int LoadLinked(int *ptr) {
	return *ptr;
}
int StoreConditional(int *ptr, int value) {
	if (no update to *ptr since LL to this addr) {
	*ptr = value;
	return 1; // success!
	} else {
		return 0; // failed to update
	}
}
```
- Build a lock using load-linked and store-conditional:
```C
void lock(lock_t *lock) {
	while (LoadLinked(&lock->flag) || !StoreConditional(&lock->flag, 1))
		; // spin
}

void unlock(lock_t *lock) {
	lock->flag = 0;
}
```
##### Fetch-And-Add
- One final hardware primitive is the **fetch-and-add** instruction, which atomically increments a value while returning the old value at a particular address. 
- The C pseudocode for the fetch-and-add instruction looks like this:
```pseudocode
int FetchAndAdd(int *ptr) {
	int old = *ptr;
	*ptr = old + 1;
	return old;
}
```
- Fetch-and-add is used to build a **ticket lock**
```C
typedef struct __lock_t {
	int ticket;
	int turn;
} lock_t;

void lock_init(lock_t *lock) {
	lock->ticket = 0;
	lock->turn = 0;
}

void lock(lock_t *lock) {
	int myturn = FetchAndAdd(&lock->ticket);
	while (lock->turn != myturn)
		; // spin
}

void unlock(lock_t *lock) {
	lock->turn = lock->turn + 1;
}
```
- Instead of a single value, this solution uses a ticket and turn variable in combination to build a lock. 
- The basic operation is pretty simple: when a thread wishes to acquire a lock, it first does an atomic fetch-and-add on the ticket value; that value is now considered this thread’s “turn” (`myturn`). 
- The globally shared `lock->turn` is then used to determine which thread’s turn it is; when (`myturn == turn`) for a given thread, it is that thread’s turn to enter the critical section.
- Unlock is accomplished simply by incrementing the turn such that the next waiting thread (if there is one) can now enter the critical section.
- This ensures progress for **all threads**. Once a thread is assigned its ticket value, it will be scheduled at some point in the future (once those in front of it have passed through the critical section and released the lock).
#####  A Simple Approach: Just Yield, Baby
- Hardware support got us pretty far: working locks, and even fairness in lock acquisition.
- However, we still have a problem: what to do when a context switch occurs in a critical section, and threads start to spin endlessly, waiting for the interrupted (lock-holding) thread to be run again?
- The first basic idea is when you are going to spin, instead **give up** the CPU to another thread.
```C
void init() {
	flag = 0;
}

void lock() {
	while (TestAndSet(&flag, 1) == 1)
		yield(); // give up the CPU
}

void unlock() {
	flag = 0;
}
```
- In this approach, we assume an operating system primitive `yield()` which a thread can call when it wants to give up the CPU and let another thread run.
- `yield` is simply a system call that moves the caller from the **running** state to the **ready** state (**deschedules** itself), and thus promotes another thread to running.
- Let us now consider the case where there are many threads (say 100) contending for a lock repeatedly. 
- In this case, if one thread acquires the lock and is preempted before releasing it, the other 99 will each call `lock()`, find the lock held, and yield the CPU. 
- Assuming some kind of [[0x03_Introduction to scheduling#Round Robin|round-robin]] scheduler, each of the 99 will execute this run-and-yield pattern before the thread holding the lock gets to run again. 
- While better than our spinning approach (which would waste 99 time slices spinning), this approach is still costly; the cost of a context switch can be substantial, and there is thus plenty of waste.
- Worse, this approach does not address **starvation**. A thread may get caught in an **endless yield** loop while other threads repeatedly enter and exit the critical section.
##### Using Queues: Sleeping Instead Of Spinning
- The scheduler determines which thread runs next; if the scheduler makes a bad choice, a thread that runs must either spin waiting for the lock (our first approach), or yield the CPU immediately (our second approach). Either way, there is potential for waste and no prevention of starvation.
- We will use the support provided by Solaris, in terms of two calls: `park()` to put a calling **thread to sleep**, and `unpark(threadID)` to **wake** a particular thread as designated by threadID. 
- These two routines can be used in tandem to build a lock that puts a caller to sleep if it tries to acquire a held lock and wakes it when the lock is free.
```C
typedef struct __lock_t {
	int flag;
	int guard;
	queue_t *q;
} lock_t;

void lock_init(lock_t *m) {
	m->flag = 0;
	m->guard = 0;
	queue_init(m->q);
}

void lock(lock_t *m) {
	while (TestAndSet(&m->guard, 1) == 1)
		; //acquire guard lock by spinning
	if (m->flag == 0) {
		m->flag = 1; // lock is acquired
		m->guard = 0;
	} else {
		queue_add(m->q, gettid());
		setpark(); 
		m->guard = 0;
		park();
	}
}

void unlock(lock_t *m) {
	while (TestAndSet(&m->guard, 1) == 1)
		; //acquire guard lock by spinning
	if (queue_empty(m->q))
		m->flag = 0; // let go of lock; no one wants it
	else
		unpark(queue_remove(m->q)); // hold lock (for next thread!)
	
	m->guard = 0;
}
```
- By calling `setpark()` routine, a thread can indicate it is **about to** park. 
- If it then happens to be interrupted and another thread calls `unpark` before `park` is actually called, the subsequent `park` returns immediately instead of sleeping.
##### Different OS, Different Support
- Linux provides a **futex** which is similar to the Solaris interface but provides more in-kernel functionality. 
- Specifically, each **futex** has associated with it a specific physical memory location, as well as a per-futex in-kernel queue.
- The call to `futex_wait(address, expected)` puts the calling thread to sleep, assuming the value at the address `address` is equal to `expected`.
- If it is not equal, the call returns immediately.
- The call to the routine `futex_wake(address)` wakes one thread that is waiting on the queue.
```C
void mutex_lock (int *mutex) {
	int v;
	// Bit 31 was clear, we got the mutex (fastpath)
	if (atomic_bit_test_set (mutex, 31) == 0)
		return;
	atomic_increment (mutex);
	while (1) {
		if (atomic_bit_test_set (mutex, 31) == 0) {
			atomic_decrement (mutex);
			return;
		}
		// Have to waitFirst to make sure futex value
		// we are monitoring is negative (locked).
		v = *mutex;
		if (v >= 0)
			continue;
		futex_wait (mutex, v);
	}
}

void mutex_unlock (int *mutex) {
	// Adding 0x80000000 to counter results in 0 if and
	// only if there are not other interested threads
	if (atomic_add_zero (mutex, 0x80000000))
		return;
	// There are other threads waiting for this mutex,
	// wake one of them up.
	futex_wake (mutex);
}
```
- Code uses a single integer to track both whether the lock is held or not (the high bit of the integer) and the number of waiters on the lock (all the other bits).
- Thus, if the lock is negative, it is held (because the high bit is set and that bit determines the sign of the integer).
##### Two-Phase Locks
- A two-phase lock realizes that spinning can be useful, particularly if the lock is about to be released. 
- So in the first phase, the lock spins for a while, hoping that it can acquire the lock.
- However, if the lock is not acquired during the first spin phase, a second phase is entered, where the caller is put to sleep, and only woken up when the lock becomes free later. 
- The Linux lock above is a form of such a lock, but it only spins once; a generalization of this could spin in a loop for a fixed amount of time before using futex support to sleep.
# Sources
- Operating Systems: Three Easy Pieces - Chapter 28.
- [Lecture 6 - part 3](https://youtu.be/BoLYvNp2Lc4)
- [Lecture 7 - part 1](https://www.youtube.com/watch?v=G95w4ghn42A)
- [Lecture 7 - part 2](https://www.youtube.com/watch?v=X5clCyJ4uuk)