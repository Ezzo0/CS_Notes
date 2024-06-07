# Explanation
- There are many cases where a [[0x17_Intro to Concurrency|thread]] wishes to check whether a condition is true before continuing its execution. For example, a parent thread might wish to check whether a child thread has completed before continuing.
- We could try using a shared variable, but it is hugely inefficient as the parent spins and wastes CPU time.
##### Definition and Routines
- A **condition variable** is an explicit queue that threads can put themselves on when some state of execution (i.e., some **condition**) is not as desired (by **waiting** on the condition).
- Some other thread, when it changes said state, can then wake one (or more) of those waiting threads and thus allow them to continue (by **signaling** on the condition).
- To declare such a condition variable, one simply writes something like this: `pthread cond t c;`, which declares c as a condition variable. 
- A condition variable has two operations associated with it: `pthread_cond_wait()` and `pthread_cond_signal()`.
	- `pthread_cond_wait(pthread_cond_t *c, pthread_mutex_t *m);` is executed when a thread wishes to put itself to sleep. 
	- `pthread_cond_signal(pthread_cond_t *c);` is executed when a thread has changed something in the program and thus wants to wake a sleeping thread waiting on this condition.
- `wait()` call is that it also takes a [[0x18_Locks#^f3ec79|mutex]] as a parameter; it assumes that this [[0x18_Locks#^f3ec79|mutex]] is locked when `wait()` is called. 
- The responsibility of `wait()` is to release the lock and put the calling thread to sleep (atomically); when the thread wakes up, it must re-acquire the lock before returning to the caller.
```C
int done = 0;
pthread_mutex_t m = PTHREAD_MUTEX_INITIALIZER;
pthread_cond_t c = PTHREAD_COND_INITIALIZER;

void *child(void *arg) {
	printf("child\n");
	
	Pthread_mutex_lock(&m);
	done = 1;
	Pthread_cond_signal(&c);
	Pthread_mutex_unlock(&m);
	
	return NULL;
}

int main(int argc, char *argv[]) {
	printf("parent: begin\n");
	pthread_t p;
	Pthread_create(&p, NULL, child, NULL);
	
	Pthread_mutex_lock(&m);
	while (done == 0)
		Pthread_cond_wait(&c, &m); // Sleep & release the lock
								   // Then, returning from wait()
								   // with the lock held
	Pthread_mutex_unlock(&m);
	
	printf("parent: end\n");
	return 0;
}
```
- you might be wondering if we need the state variable `done`. What if the code looked like the example below?
```C
void *child(void *arg) {
	printf("child\n");
	
	Pthread_mutex_lock(&m);
	Pthread_cond_signal(&c);
	Pthread_mutex_unlock(&m);
	
	return NULL;
}

int main(int argc, char *argv[]) {
	printf("parent: begin\n");
	pthread_t p;
	Pthread_create(&p, NULL, child, NULL);
	
	Pthread_mutex_lock(&m);
	Pthread_cond_wait(&c, &m);
	Pthread_mutex_unlock(&m);
	
	printf("parent: end\n");
	return 0;
}
```
- Unfortunately this approach is broken. Imagine the case where the child runs immediately; in this case, the child will signal, but there is no thread asleep on the condition. 
- When the parent runs, it will simply call `wait` and be stuck; no thread will ever wake it.
##### The Producer/Consumer (Bounded Buffer) Problem
- Imagine one or more producer threads and one or more consumer threads. Producers generate data items and place them in a buffer; consumers grab said items from the buffer and consume them in some way.
- Because the bounded buffer is a shared resource, we must of course require synchronized access to it, lest a race condition arise.
```C
int buffer[MAX];
int fill_ptr = 0;
int use_ptr  = 0;
int count    = 0;

int loops; // must initialize somewhere...
cond_t cond;
mutex_t mutex;

void put(int value) {
	buffer[fill_ptr] = value;
	fill_ptr = (fill_ptr + 1) % MAX;
	count++;
}

int get() {
	int tmp = buffer[use_ptr];
	use_ptr = (use_ptr + 1) % MAX;
	count--;
	return tmp;
}

void *producer(void *arg) {
	int i;
	for (i = 0; i < loops; i++) {
		Pthread_mutex_lock(&mutex);           // p1
		if (count == MAX)                     // p2
			Pthread_cond_wait(&cond, &mutex); // p3
		put(i);                               // p4
		Pthread_cond_signal(&cond);           // p5
		Pthread_mutex_unlock(&mutex);         // p6
	}
}

void *consumer(void *arg) {
	int i;
	for (i = 0; i < loops; i++) {
		Pthread_mutex_lock(&mutex);           // c1
		if (count == 0)                       // c2
			Pthread_cond_wait(&cond, &mutex); // c3
		int tmp = get();                      // c4
		Pthread_cond_signal(&cond);           // c5
		Pthread_mutex_unlock(&mutex);         // c6
		printf("%d\n", tmp);
	}
}
```
- When a producer wants to fill the buffer, it waits for it to be empty (p1–p3). 
- The consumer has the exact same logic, but waits for a different condition: fullness (c1–c3).
- However, if we have more than one of these threads (e.g., two consumers), the solution has two critical problems. 
	1. Assume there are two consumers (Tc1 and Tc2 ) and one producer (Tp).
		- First, a consumer (Tc1 ) runs; it acquires the lock (c1), checks if any buffers are ready for consumption (c2), and finding that none are, waits (c3) (which releases the lock). 
		- Then the producer (Tp ) runs. It acquires the lock (p1), checks if all buffers are full (p2), and finding that not to be the case, goes ahead and fills the buffer (p4).
		- The producer then signals that a buffer has been filled (p5). Critically, this moves the first consumer (Tc1 ) from sleeping on a condition variable to the ready queue; Tc1 is now able to run (but not yet running). 
		- The producer then continues until realizing the buffer is full, at which point it sleeps (p6, p1–p3).
		- another consumer (Tc2 ) sneaks in and consumes the one existing value in the buffer (c1, c2, c4, c5, c6, skipping the wait at c3 because the buffer is full). 
		- Now assume Tc1 runs; just before returning from the wait, it reacquires the lock and then returns. It then calls `get()` (c4), but there are no buffers to consume! An assertion triggers, and the code has not functioned as desired. 
		- Clearly, we should have somehow prevented (Tc1) from trying to consume because (Tc2) snuck in and consumed the one value in the buffer that had been produced.
		- This interpretation of what a signal means is often referred to as **Mesa semantics**![[Thread Trace_Broken Solution (v1).png]]
	2.  change the `if` to a `while`. 
		- Now consumer Tc1 wakes up and (with the lock held) rechecks the state of the shared variable (c2). If the buffer is empty at that point, the consumer simply goes back to sleep (c3).
		- However, this code still has a bug, the problem occurs when two consumers run first (Tc1 and Tc2 ) and both go to sleep (c3).
		- Then, the producer runs, puts a value in the buffer, and wakes one of the consumers (say Tc1 ). 
		- The producer then loops back and tries to put more data in the buffer; because the buffer is full, the producer instead waits on the condition (thus sleeping). 
		- Now, one consumer is ready to run (Tc1 ), and two threads are sleeping on a condition (Tc2 and Tp ).
		- The consumer Tc1 then wakes by returning from `wait()` (c3), rechecks the condition (c2), and finding the buffer full, consumes the value (c4).
		- This consumer then, signals on the condition (c5), waking only one thread that is sleeping. However, which thread should it wake?
		- Because the consumer has emptied the buffer, it clearly should wake the producer. However, if it wakes the consumer Tc2 (which is definitely possible, depending on how the wait queue is managed), we have a problem.
		- Specifically, the consumer Tc2 will wake up and find the buffer empty (c2), and go back to sleep (c3). The producer Tp , which has a value to put into the buffer, is left sleeping. The other consumer thread, Tc1, also goes back to sleep. All three threads are left sleeping, a clear bug.
		 ![[Thread Trace_Broken Solution (v2).png]]

```C
int loops; // must initialize somewhere...
cond_t cond;
mutex_t mutex;

void *producer(void *arg) {
	int i;
	for (i = 0; i < loops; i++) {
		Pthread_mutex_lock(&mutex);           // p1
		while (count == MAX)                  // p2
			Pthread_cond_wait(&cond, &mutex); // p3
		put(i);                               // p4
		Pthread_cond_signal(&cond);           // p5
		Pthread_mutex_unlock(&mutex);         // p6
	}
}

void *consumer(void *arg) {
	int i;
	for (i = 0; i < loops; i++) {
		Pthread_mutex_lock(&mutex);           // c1
		while (count == 0)                    // c2
			Pthread_cond_wait(&cond, &mutex); // c3
		int tmp = get();                      // c4
		Pthread_cond_signal(&cond);           // c5
		Pthread_mutex_unlock(&mutex);         // c6
		printf("%d\n", tmp);
	}
}
```
- The solution here is using two condition variables, instead of one, in order to properly signal which type of thread should wake up when the state of the system changes.
- In the code, producer threads wait on the condition **empty**, and signals **fill**.
- Conversely, consumer threads wait on **fill** and signal **empty**. 
- By doing so, the second problem above is avoided by design: a consumer can never accidentally wake a consumer, and a producer can never accidentally wake a producer
```C
cond_t empty, fill;
mutex_t mutex;

void *producer(void *arg) {
	int i;
	for (i = 0; i < loops; i++) {
		Pthread_mutex_lock(&mutex);
		while (count == MAX)
			Pthread_cond_wait(&empty, &mutex);
		put(i);
		Pthread_cond_signal(&fill);
		Pthread_mutex_unlock(&mutex);
	}
}

void *consumer(void *arg) {
	int i;
	for (i = 0; i < loops; i++) {
		Pthread_mutex_lock(&mutex);
		while (count == 0)
			Pthread_cond_wait(&fill, &mutex);
		int tmp = get();
		Pthread_cond_signal(&empty);
		Pthread_mutex_unlock(&mutex);
		printf("%d\n", tmp);
	}
}
```
##### Covering Conditions
```C
// how many bytes of the heap are free?
int bytesLeft = MAX_HEAP_SIZE;

// need lock and condition too
cond_t c;
mutex_t m;

void *allocate(int size) {
	Pthread_mutex_lock(&m);
	while (bytesLeft < size)
		Pthread_cond_wait(&c, &m);
	void *ptr = ...; // get mem from heap
	bytesLeft -= size;
	Pthread_mutex_unlock(&m);
	return ptr;
}

void free(void *ptr, int size) {
	Pthread_mutex_lock(&m);
	bytesLeft += size;
	Pthread_cond_signal(&c); // whom to signal??
	Pthread_mutex_unlock(&m);
}
```
- As you might see in the code, when a thread calls into the memory allocation code, it might have to wait in order for more memory to become free. 
- Conversely, when a thread frees memory, it signals that more memory is free. However, the code above has a problem: which waiting thread (there can be more than one) should be woken up?
- Consider the following scenario. Assume there are zero bytes free; thread Ta calls `allocate(100)`, followed by thread Tb which asks for less memory by calling `allocate(10)`. 
- Both Ta and Tb thus wait on the condition and go to sleep; there aren’t enough free bytes to satisfy either of these requests.
- At that point, assume a third thread, Tc , calls `free(50)`. Unfortunately, when it calls signal to wake a waiting thread, it might not wake the correct waiting thread, Tb , which is waiting for only 10 bytes to be freed; Ta should remain waiting, as not enough memory is yet free. 
- Thus, the code in the figure does not work, as the thread waking other threads does not know which thread (or threads) to wake up.
- The solution: replace the `pthread_cond_signal()` call in the code above with a call to `pthread_cond_broadcast()`, which wakes up all waiting threads.
- The downside, of course, can be a negative performance impact, as we might needlessly wake up many other waiting threads that shouldn’t (yet) be awake.
- This condition is called a **covering condition**, as it covers all the cases where a thread needs to wake up (conservatively).
# Sources
- Operating Systems: Three Easy Pieces - Chapter 30.
- [Lecture 7 - part 2](https://www.youtube.com/watch?v=X5clCyJ4uuk)
- [Lecture 7 - part 3](https://youtu.be/RN8A9EvKBdY)
- [Lecture 8 - part 1](https://youtu.be/U1LfmL7f1h8)
- [Lecture 8 - part 2](https://youtu.be/cuY8r8RXqAY)