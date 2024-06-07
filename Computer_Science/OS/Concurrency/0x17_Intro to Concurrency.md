# Explanation
- We introduce a new abstraction for a single running process: that of a **thread**. Instead of our classic view of a single point of execution within a program (i.e., a single PC where instructions are being fetched from and executed), a **multi-threaded** program has more than one point of execution (i.e., multiple PCs, each of which is being fetched and executed from). 
- Perhaps another way to think of this is that each thread is very much like a separate process, except for one difference: they **share the same address space** and thus can access the same data.
- Each thread has: 
	- A program counter (PC) that tracks where the program is fetching instructions from.
	- Its own private set of registers it uses for computation; 
- Thus, if there are two threads that are running on a single processor, when switching from running one (T1) to running the other (T2), a **context switch** must take place.
	- There is one major difference, though, in the context switch we perform between threads as compared to processes: the address space remains the same (i.e., there is no need to switch which page table we are using).
- With processes, we saved state to a **[[0x01_Process#^5bb4f2|process control block (PCB)]]**; now, we’ll need one or more **thread control blocks (TCBs)** to store the state of each thread of a process.
- The address space of a classic process (which we can now call a **single-threaded** process), there is a single stack, usually residing at the bottom of the address space 
- However, in a multi-threaded process, each thread runs independently and instead of a single stack in the address space, there will be **one per thread** which is sometimes called **thread-local** storage.
	 ![[Single-Threaded And Multi-Threaded.png]]
##### Why Use Threads?
- There are at least two major reasons you should use threads:
	1. **Parallelism:**
		- Imagine a program that performs operations on very large arrays, for example, adding two large arrays together.
		- If you are running on just a single processor, the task is straightforward: just perform each operation and be done.
		- However, if you are executing the program on a system with multiple processors, you have the potential of speeding up this process considerably by using the processors to each perform a portion of the work. 
		- The task of transforming your standard single-threaded program into a program that does this sort of work on multiple CPUs is called **parallelization**.
	2. **Avoid blocking program progress due to slow I/O:**
		- Imagine that you are writing a program that performs different types of I/O: either waiting to send or receive a message, for an explicit disk I/O to complete, or even (implicitly) for a page fault to finish. 
		- Instead of waiting, your program may wish to do something else, including utilizing the CPU to perform computation, or even issuing further I/O requests.
		- Using threads is a natural way to avoid getting stuck; while one thread in your program waits, the CPU scheduler can switch to other threads, which are ready to run and do something useful.
	- Threading enables **overlap** of I/O with other activities within a single program, much like **multiprogramming** did for processes across programs.
##### Thread Creation
```C
#include <stdio.h>
#include <assert.h>
#include <pthread.h>

void *mythread(void *arg) {
	printf("%s\n", (char *) arg);
	return NULL;
}

int main(int argc, char *argv[]) {
	pthread_t p1, p2;
	printf("main: begin\n");
	pthread_create(&p1, NULL, mythread, "A");
	pthread_create(&p2, NULL, mythread, "B");
	// join waits for the threads to finish
	pthread_join(p1, NULL);
	pthread_join(p2, NULL);
	printf("main: end\n");
	return 0;
}
```
- One way to think about thread creation is that it is a bit like making a function call.
- However, instead of first executing the function and then returning to the caller, the system instead creates a new thread of execution for the routine that is being called, and it runs independently of the caller, perhaps before returning from the create, but perhaps much later.
- What runs next is determined by the OS **scheduler**, and although the scheduler likely implements some sensible algorithm, it is hard to know what will run at any given moment in time.
##### Why It Gets Worse: Shared Data
```c
#include <stdio.h>
#include <pthread.h>

static volatile int counter = 0;

// mythread()
//
// Simply adds 1 to counter repeatedly, in a loop
// No, this is not how you would add 10,000,000 to
// a counter, but it shows the problem nicely.
//
void *mythread(void *arg) {
	printf("%s: begin\n", (char *) arg);
	int i;
	for (i = 0; i < 1e7; i++) {
		counter = counter + 1;
	}
	printf("%s: done\n", (char *) arg);
	return NULL;
}

// main()
//
// Just launches two threads (pthread_create)
// and then waits for them (pthread_join)
//
int main(int argc, char *argv[]) {
	pthread_t p1, p2;
	printf("main: begin (counter = %d)\n", counter);
	pthread_create(&p1, NULL, mythread, "A");
	pthread_create(&p2, NULL, mythread, "B");
	// join waits for the threads to finish
	pthread_join(p1, NULL);
	pthread_join(p2, NULL);
	printf("main: done with both (counter = %d)\n", counter);
	return 0;
}
```
Unfortunately, when we run this code, even on a single processor, we
don’t necessarily get the desired result. Sometimes, we get:
- `main: done with both (counter = 19345221)`
- `main: done with both (counter = 19221041)`
##### The Heart Of The Problem: Uncontrolled Scheduling
- The code sequence for incrementing counter by 1 might look something like this (in x86):
	```Assembly
	mov 0x8049a1c, %eax
	add $0x1, %eax
	mov %eax, 0x8049a1c
	```
- suppose one of our two threads (Thread 1) enters this region of code, and is thus about to increment counter by one. 
	- It loads the value of counter (let’s say it’s 50 to begin with) into its register eax. Thus, eax=50 for Thread 1. Then it adds one to the register; thus eax=51.
	- Now, something unfortunate happens: a timer interrupt goes off; thus, the OS saves the state of the currently running thread (its PC, its registers including eax, etc.) to the thread’s TCB.
- Now something worse happens: Thread 2 is chosen to run, and it enters this same piece of code. 
	- It also executes the first instruction, getting the value of counter and putting it into its eax (remember: each thread when running has its own private registers; the registers are **virtualized** by the context-switch code that saves and restores them).
	- The value of counter is still 50 at this point, and thus Thread 2 has eax=50. 
	- Let’s then assume that Thread 2 executes the next two instructions, incrementing eax by 1, and then saving the contents of eax into counter. 
	- Thus, the global variable counter now has the value 51.
- Finally, another context switch occurs, and Thread 1 resumes running.
	- Recall that it had just executed the `mov` and `add`, and is now about to perform the final `mov` instruction. Recall also that eax=51. 
	- Thus, the final `mov` instruction executes, and saves the value to memory; the counter is set to 51 again.
- ![[problem.png]]
- What we have demonstrated here is called a **race condition** (or, more specifically, a **data race**):the results depend on the timing of the code’s execution.
- Because multiple threads executing this code can result in a race condition, we call this code a **critical section**.
	- A critical section is a piece of code that accesses a shared variable (or more generally, a shared resource) and must not be concurrently executed by more than one thread.
- What we really want for this code is what we call **mutual exclusion**.
	- This property guarantees that if one thread is executing within the critical section, the others will be prevented from doing so.
##### The Wish For Atomicity
- One way to solve this problem would be to have more powerful instructions that, in a single step, did exactly whatever we needed done and thus removed the possibility of an untimely interrupt. 
- For example, what if we had a super instruction that looked like this: 
	- `memory-add 0x8049a1c, $0x1`
	- It could not be interrupted mid-instruction, because that is precisely the guarantee we receive from the hardware: when an interrupt occurs, either the instruction has not run at all, or it has run to completion; there is no in-between state.
- What we will instead do is ask the hardware for a few useful instructions upon which we can build a general set of what we call **synchronization primitives**. 
- By using this hardware support, in combination with some help from the operating system, we will be able to build multi-threaded code that accesses critical sections in a synchronized and controlled manner.
# Sources
- Operating Systems: Three Easy Pieces - Chapter 26.
- [Lecture 6 - part 1](https://www.youtube.com/watch?v=4PghlMdp9cU)
- [Lecture 6 - part 2](https://www.youtube.com/watch?v=hivv8F-LjzY)