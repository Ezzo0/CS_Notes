# Explanation
- A semaphore is an object with an integer value that we can manipulate with two routines; in the POSIX standard, these routines are `sem_wait()`and `sem_post()`.
- Because the initial value of the semaphore determines its behavior, before calling any other routine to interact with the semaphore, we must first initialize it to some value.
```C
#include <semaphore.h>
sem_t s;
sem_init(&s, 0, 1);
```
- A semaphore `s` is declared and initialized to the value 1
- The second argument to `sem_init()` is be set to 0 ; this indicates that the semaphore is shared between [[0x17_Intro to Concurrency|threads]] in the same process.
- `sem_wait()` behavior is waiting if value of semaphore `s` is 0 or negative and decrementing the value of semaphore `s` by one.
- `sem_post` behavior is incrementing the value of semaphore `s` by one and if there are one or more threads waiting, wake one.
```C
// Behavior demonstration FROM LECTURE HANDOUT
sem_init(sem_t *s, int value) {
	s->value = value;
}
sem_wait(sem_t *s) {
	while (s->value <= 0)
		put_self_to_sleep();
	s->value--;
}
sem_post(sem_t *s) {
	s->value++;
	wake_one_waiting_thread();
}
// Each routine executes ATOMICALLY
```
##### Binary Semaphores ([[0x18_Locks|Locks]])
```C
sem_t m;
sem_init(&m, 0, X); // init to X; what should X be?

sem_wait(&m);
// critical section here
sem_post(&m);
```
- Critical to making this work, though, is the initial value of the semaphore `m`.
- Looking back at definition of the `sem_wait()` and `sem_post()` routines above, we can see that the initial value should be 1.
- To make this clear, let’s imagine a scenario with two threads: 
	- The first thread (Thread 0) calls `sem_wait()`; it will first decrement the value of the semaphore, changing it to 0. 
	- Then, it will wait only if the value is not greater than or equal to 0. Because the value is 0, `sem_wait()` will simply return and the calling thread will continue; Thread 0 is now free to enter the critical section. 
	- If no other thread tries to acquire the lock while Thread 0 is inside the critical section, when it calls `sem_post()`, it will simply restore the value of the semaphore to 1 (and not wake a waiting thread, because there are none).
	 ![[Single Thread Using A Semaphore.png]]
- when Thread 0 “holds the lock”, and another thread (Thread 1) tries to enter the critical section by calling `sem_wait()`.
	- In this case, Thread 1 will decrement the value of the semaphore to -1, and thus wait (putting itself to sleep and relinquishing the processor). 
	- When Thread 0 runs again, it will eventually call `sem_post()`, incrementing the value of the semaphore back to zero, and then wake the waiting thread (Thread 1), which will then be able to acquire the lock for itself. 
	- When Thread 1 finishes, it will again increment the value of the semaphore, restoring it to 1 again.
	 ![[Two Threads Using A Semaphore.png]]
##### Semaphores For Ordering
- If one thread waiting for something to happen, and another thread making that something happen and then signaling that it has happened, then we are using the semaphore as an **ordering** primitive (similar to our use of **[[0x19_Condition Variables|condition variables]]** earlier).
- Imagine a thread creates another thread and then wants to wait for it to complete its execution:
```C
sem_t s;

void *child(void *arg) {
	printf("child\n");
	sem_post(&s); // signal here: child is done
	return NULL;
}

int main(int argc, char *argv[]) {
	sem_init(&s, 0, X); // what should X be?
	printf("parent: begin\n");
	pthread_t c;
	Pthread_create(&c, NULL, child, NULL);
	sem_wait(&s); // wait here for child
	printf("parent: end\n");
	return 0;
}
```
- As you can see in the code, the parent simply calls `sem_wait()` and the child `sem_post()` to wait for the condition of the child finishing its execution to become true. But, what should the initial value of this semaphore be?
- The answer is that the value of the semaphore should be set to is 0.
- There are two cases to consider: 
	1. First, let us assume that the parent creates the child but the child has not run yet. 
		-  In this case, the parent will call `sem_wait()` before the child has called `sem_post()`; we’d like the parent to wait for the child to run. 
		- The only way this will happen is if the value of the semaphore is not greater than 0; hence, 0 is the initial value.
		- The parent runs, decrements the semaphore (to -1), then waits (sleeping). 
		- When the child finally runs, it will call `sem_post()`, increment the value of the semaphore to 0, and wake the parent, which will then return from `sem_wait()` and finish the program.
		 ![[Parent Waiting For Child (Case 1).png]]
	2.  The second case occurs when the child runs to completion before the parent gets a chance to call `sem_wait()`.
		-  In this case, the child will first call `sem_post()`, thus incrementing the value of the semaphore from 0 to 1. 
		- When the parent then gets a chance to run, it will call `sem_wait()` and find the value of the semaphore to be 1; the parent will thus decrement the value (to 0) and return from `sem_wait()` without waiting, also achieving the desired effect.
		 ![[Parent Waiting For Child (Case 2).png]]
##### [[0x19_Condition Variables#The Producer/Consumer (Bounded Buffer) Problem|The Producer/Consumer (Bounded Buffer) Problem]]
- **First Attempt**
	- Our first attempt at solving the problem introduces two semaphores, `empty` and `full`, which the threads will use to indicate when a buffer entry has been emptied or filled, respectively.
	- Let us first imagine that `MAX` = 1 (there is only one buffer in the array), and see if this works.
	- Assume the consumer gets to run first. Thus, the consumer will hit Line C1, calling `sem_wait(&full)`. Because full was initialized to the value 0, the call will decrement full (to -1), block the consumer, and wait for another thread to call `sem_post()` on full.
	- Assume the producer then runs. It will hit Line P1, thus calling the `sem_wait(&empty)` routine. Unlike the consumer, the producer will continue through this line. 
	- Thus, empty will be decremented to 0 and the producer will put a data value into the first entry of buffer (Line P2). 
	- The producer will then continue on to P3 and call `sem_post(&full)`, changing the value of the full semaphore from -1 to 0 and waking the consumer.
	- Let us now imagine that MAX is greater than 1 (say MAX=10). We now have a problem: a race condition.
	- Imagine two producers (Pa and Pb) both calling into `put()` at roughly the same time. Assume producer Pa gets to run first, and just starts to fill the first buffer entry (fill=0 at Line F1).
	- Before Pa gets a chance to increment the fill counter to 1, it is interrupted. Producer Pb starts to run, and at Line F1 it also puts its data into the 0th element of buffer, which means that the old data there is overwritten!
```C
int buffer[MAX];
int fill = 0;
int use = 0;

void put(int value) {
	buffer[fill] = value;    // Line F1
	fill = (fill + 1) % MAX; // Line F2
}

int get() {
	int tmp = buffer[use];   // Line G1
	use = (use + 1) % MAX;   // Line G2
	return tmp;
}

sem_t empty;
sem_t full;

void *producer(void *arg) {
	int i;
	for (i = 0; i < loops; i++) {
		sem_wait(&empty);       // Line P1
		put(i);                 // Line P2
		sem_post(&full);        // Line P3
	}
}

void *consumer(void *arg) {
	int tmp = 0;
	while (tmp != -1) {
		sem_wait(&full);        // Line C1
		tmp = get();            // Line C2
		sem_post(&empty);       // Line C3
		printf("%d\n", tmp);
	}
}

int main(int argc, char *argv[]) {
	// ...
	sem_init(&empty, 0, MAX); // MAX are empty
	sem_init(&full, 0, 0);    // 0 are full
	// ...
}
```
- **A Solution: Adding Mutual Exclusion**
	- The filling of a buffer and incrementing of the index into the buffer is a critical section, and thus must be guarded carefully. So let’s use binary semaphore and add some locks.
	- That seems like the right idea, but it also doesn’t work. Why? **[[0x21_Common Concurrency Problems#Deadlock Bugs|Deadlock]]**.
	- Imagine two threads, one producer and one consumer. The consumer gets to run first. 
	- It acquires the `mutex` (Line C0), and then calls `sem_wait()` on the full semaphore (Line C1); because there is no data yet, this call causes the consumer to block and thus yield the CPU; importantly, though, the consumer still holds the lock.
	- A producer then runs and it has data to produce. The first thing it does is call `sem_wait()`on the binary `mutex` semaphore (Line P0). The lock is already held. Hence, the producer is now stuck waiting too.
	- There is a simple cycle here. The consumer holds the `mutex` and is waiting for the someone to signal full. The producer could signal full but is waiting for the `mutex`. Thus, the producer and consumer are each stuck waiting for each other: **a classic [[0x21_Common Concurrency Problems#Deadlock Bugs|Deadlock]]**.
```C
void *producer(void *arg) {
	sem_wait(&mutex);
	int i;
	for (i = 0; i < loops; i++) {
		sem_wait(&empty);       // Line P1
		put(i);                 // Line P2
		sem_post(&full);        // Line P3
	}
	sem_post(&mutex);
}

void *consumer(void *arg) {
	sem_wait(&mutex);
	int i;
	for (i = 0; i < loops; i++) {
		sem_wait(&full);        // Line C1
		int tmp = get();        // Line C2
		sem_post(&empty);       // Line C3
		printf("%d\n", tmp);
	}
	sem_post(&mutex);
}
```
- **At Last, A Working Solution**
	- we simply move the `mutex` acquire and release to be just around the critical section; the full and empty wait and signal code is left outside.
```C
void *producer(void *arg) {
	int i;
	for (i = 0; i < loops; i++) {
		sem_wait(&empty);       // Line P1
		sem_wait(&mutex);       // Line P1.5 (Lock)
		put(i);                 // Line P2
		sem_post(&mutex);       // Line P2.5 (Unlock)
		sem_post(&full);        // Line P3
	}
}

void *consumer(void *arg) {
	int i;
	for (i = 0; i < loops; i++) {
		sem_wait(&full);        // Line C1
		sem_wait(&mutex);       // Line C1.5 (lock)
		int tmp = get();        // Line C2
		sem_post(&mutex);       // Line C2.5 (unlock)
		sem_post(&empty);       // Line C3
		printf("%d\n", tmp);
	}
}
```
##### Reader-Writer Locks
- Imagine a number of concurrent list operations, including inserts and simple lookups. While inserts change the state of the list (and thus a traditional critical section makes sense), lookups simply read the data structure; as long as we can guarantee that no insert is on-going, we can allow many lookups to proceed concurrently. 
- The special type of lock we will now develop to support this type of operation is known as a **reader-writer lock**.
- If some thread wants to update the data structure, it should call the new pair of synchronization operations: `rwlock_acquire_writelock()`, to acquire a write lock, and `rwlock_release_writelock()`, to release it. 
- Internally, these simply use the `writelock` semaphore to ensure that only a single writer can acquire the lock and thus enter the critical section to update the data structure.
- When acquiring a read lock, the reader first acquires `lock` and then increments the `readers` variable to track how many readers are currently inside the data structure.
- When the first reader acquires the lock, the reader also acquires the write lock by calling `sem_wait()` on the `writelock` semaphore, and then releasing the `lock` by calling `sem_post()`.
- Thus, once a reader has acquired a read lock, more readers will be allowed to acquire the read lock too; however, any thread that wishes to acquire the write lock will have to wait until all readers are finished; the last one to exit the critical section calls `sem_post()` on “writelock” and thus enables a waiting writer to acquire the lock.
- This approach works (as desired), but does have some negatives, especially when it comes to fairness. In particular, it would be relatively easy for readers to starve writers.
```C
typedef struct _rwlock_t {
sem_t lock;        // binary semaphore (basic lock)
sem_t writelock;   // allow ONE writer/MANY readers
int readers;       // #readers in critical section
} rwlock_t;

void rwlock_init(rwlock_t *rw) {
	rw->readers = 0;
	sem_init(&rw->lock, 0, 1);
	sem_init(&rw->writelock, 0, 1);
}

void rwlock_acquire_readlock(rwlock_t *rw) {
	sem_wait(&rw->lock);
	rw->readers++;
	if (rw->readers == 1) // first reader gets writelock
		sem_wait(&rw->writelock);
	sem_post(&rw->lock);
}

void rwlock_release_readlock(rwlock_t *rw) {
	sem_wait(&rw->lock);
	rw->readers--;
	if (rw->readers == 0) // last reader lets it go
		sem_post(&rw->writelock);
	sem_post(&rw->lock);
}

void rwlock_acquire_writelock(rwlock_t *rw) {
	sem_wait(&rw->writelock);
}

void rwlock_release_writelock(rwlock_t *rw) {
	sem_post(&rw->writelock);
}
```
##### The Dining Philosophers
- The basic setup for the problem is: 
	- Assume there are five “philosophers” sitting around a table. Between each pair of philosophers is a single fork (and thus, five total). 
	- The philosophers each have times where they think, and don’t need any forks, and times where they eat. 
	- In order to eat, a philosopher needs two forks, both the one on their left and the one on their right.
	 ![[The Dining Philosophers.png|350]]
- Here is the basic loop of each philosopher, assuming each has a unique thread identifier p from 0 to 4 (inclusive):
```C
while (1) {
	think();
	get_forks(p);
	eat();
	put_forks(p);
}
```
- The key challenge, then, is to write the routines `get_forks()` and `put_forks()` such that there is no deadlock, no philosopher starves and never gets to eat, and concurrency is high (i.e., as many philosophers can eat at the same time as possible).
- We’ll use a few helper functions to get us towards a solution. They are:
	- `int left(int p) { return p; }`.
	- `int right(int p) { return (p + 1) % 5; }`.
- When philosopher p wishes to refer to the fork on their left, they call `left(p)`.
- Similarly, the fork on the right of a philosopher `p` is referred to by calling `right(p)`.
- We’ll also need some semaphores to solve this problem. Let us assume we have five, one for each fork: `sem_t forks[5]`.
- **Broken Solution**
	- Assume we initialize each semaphore (in the `forks` array) to a value of 1.
	- Assume also that each philosopher knows its own number (`p`). We can thus write the `get_forks()` and `put_forks()` routine as below.
	- The problem is **deadlock**. If each philosopher happens to grab the fork on their left before any philosopher can grab the fork on their right, each will be stuck holding one fork and waiting for another, forever.
```C
void get_forks(int p) {
	sem_wait(&forks[left(p)]);
	sem_wait(&forks[right(p)]);
}

void put_forks(int p) {
	sem_post(&forks[left(p)]);
	sem_post(&forks[right(p)]);
}
```
- **A Solution: Breaking The Dependency**
	- The simplest way to attack this problem is to change how forks are acquired by at least one of the philosophers.
	- Specifically, let’s assume that philosopher 4 (the highest numbered one) gets the forks in a _different_ order than the others; the `put_forks()` code remains the same.
```C
void get_forks(int p) {
	if (p == 4) {
		sem_wait(&forks[right(p)]);
		sem_wait(&forks[left(p)]);
	} else {
		sem_wait(&forks[left(p)]);
		sem_wait(&forks[right(p)]);
	}
}
```
##### Thread Throttling
- The specific problem is this: how can a programmer prevent “too many” threads from doing something at once and bogging the system down? 
- Answer: decide upon a threshold for “too many”, and then use a semaphore to limit the number of threads concurrently executing the piece of code in question. We call this approach **throttling**, and consider it a form of **admission control**.
- Imagine that you create hundreds of threads to work on some problem in parallel. 
	- However, in a certain part of the code, each thread acquires a large amount of memory to perform part of the computation; let’s call this part of the code the memory-intensive region. 
	- If all of the threads enter the memory-intensive region at the same time, the sum of all the memory allocation requests will exceed the amount of physical memory on the machine. 
	- As a result, the machine will start thrashing (i.e., swapping pages to and from the disk), and the entire computation will slow to a crawl.
- A simple semaphore can solve this problem. By initializing the value of the semaphore to the maximum number of threads you wish to enter the memory-intensive region at once, and then putting a `sem_wait()` and `sem_post()` around the region, a semaphore can naturally throttle the number of threads that are ever concurrently in the dangerous region of the code.
##### How To Implement Semaphores
- One subtle difference between our `Zemaphore` and pure semaphore as defined by Dijkstra is that we don’t maintain the invariant that the value of the semaphore, when negative, reflects the number of waiting threads; indeed, the value will never be lower than zero.
```C
typedef struct __Zem_t {
	int value;
	pthread_cond_t cond;
	pthread_mutex_t lock;
} Zem_t;

// only one thread can call this
void Zem_init(Zem_t *s, int value) {
	s->value = value;
	Cond_init(&s->cond);
	Mutex_init(&s->lock);
}

void Zem_wait(Zem_t *s) {
	Mutex_lock(&s->lock);
	while (s->value <= 0)
		Cond_wait(&s->cond, &s->lock);
	s->value--;
	Mutex_unlock(&s->lock);
}

void Zem_post(Zem_t *s) {
	Mutex_lock(&s->lock);
	s->value++;
	Cond_signal(&s->cond);
	Mutex_unlock(&s->lock);
}
```
# Sources
- Operating Systems: Three Easy Pieces - Chapter 31.
- [Lecture 8 - part 2](https://youtu.be/cuY8r8RXqAY)