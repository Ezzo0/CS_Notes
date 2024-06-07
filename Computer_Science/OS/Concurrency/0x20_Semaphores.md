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
- 
# Sources
- Operating Systems: Three Easy Pieces - Chapter 31.
- [Lecture 8 - part 2](https://youtu.be/cuY8r8RXqAY)