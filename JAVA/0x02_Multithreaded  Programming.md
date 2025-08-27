# Explanation
- A multithreaded program contains **two or more parts** that can run **concurrently**. Each part of such a program is called a [[0x17_Intro to Concurrency#Explanation|thread]], and each thread defines a separate path of execution. Thus, multithreading is a specialized form of multitasking.
- There are two distinct types of multitasking: 
	- **Process-based:** 
		- A [[0x01_Process|process]] is, in essence, a **program** that is executing. Thus, process-based multitasking is the feature that allows your computer to run two or more programs concurrently.
		- In process-based multitasking, a program is the smallest unit of code that can be dispatched by the scheduler.
	- **Thread-based:**
		- In a thread-based multitasking environment, the **thread** is the smallest unit of dispatchable code. 
		- This means that a single program can perform two or more tasks **simultaneously** as long as these two actions are being **performed by two or more separate threads**.
- Multitasking threads require less overhead than multitasking processes:
	- **Multitasking processes**:
		- Processes are _heavyweight_ tasks that require their own _separate_ address spaces. 
		- _Interprocess communication_ is _expensive_ and limited. 
		- _Context switching_ from one process to another is also _costly_.
	- **Multitasking threads:**
		- Threads, on the other hand, are _lighter_ weight. 
		- They share the _same_ address space and cooperatively share the same heavyweight process. 
		- _Interthread communication_ is _inexpensive_, and _context switching_ from one thread to the next is _lower_ in cost.
- Process-based multitasking is not under Java’s direct control. However, multithreaded multitasking is.
- Multithreading enables you to write efficient programs that make maximum use of the processing power available in the system.
- One important way multithreading achieves this is by keeping idle time to a minimum. This is especially important for the interactive, networked environment in which Java operates because idle time is common.
### The Java Thread Model
- Threads exist in several states. A thread can be _running_. It can be ready to run as soon as it gets CPU time. 
- A running thread can be _suspended_, which temporarily halts its activity. 
- A suspended thread can then be _resumed_, allowing it to pick up where it left off. 
- A thread can be _blocked_ when waiting for a resource. 
- At any time, a thread can be _terminated_, which halts its execution immediately. Once **terminated**, a thread cannot be **resumed**.
##### Thread Priorities
- Thread priorities are _integers_ that specify the relative priority of one thread to another that are used to decide when to switch from one running thread to the next. This is called a _context switch_.
- The rules that determine when a context switch takes place are simple:
	- **A thread can voluntarily relinquish control**. This occurs when explicitly yielding, sleeping, or when blocked. In this scenario, all other threads are examined, and the highest-priority thread that is ready to run is given the CPU.
	- **A thread can be preempted by a higher-priority thread**. In this case, a lower-priority thread that does not yield the processor is simply preempted—no matter what it is doing—by a higher-priority thread. Basically, as soon as a higher-priority thread wants to run, it does. This is called _preemptive multitasking_.
##### Synchronization
- The **monitor** is a control mechanism that can hold only one thread. Once a thread **enters** a monitor, all other threads must **wait** until that thread exits the monitor. 
- In this way, a monitor can be used to protect a shared asset from being manipulated by more than one thread at a time.
- In Java, there is no class “_Monitor_”; instead, each object has its own **implicit** monitor that is **automatically entered** when one of the object’s **synchronized methods** is called. 
- Once a thread is **inside** a synchronized method, **no other thread** can call any other synchronized method on the **same** object.
##### Messaging
- Java provides a clean, low-cost way for two or more threads to talk to each other, via calls to predefined methods that all objects have. 
- Java’s messaging system allows a thread to enter a synchronized method on an object, and then wait there until some other thread explicitly notifies it to come out.
##### The Thread Class and the Runnable Interface
- Java’s multithreading system is built upon the **Thread** class, its methods, and its companion interface, **Runnable**. 
- To create a new thread, your program will either extend **Thread** or implement the **Runnable** interface. The **Thread** class defines several methods that help manage threads. Several of those used in this chapter are shown here:
	 ![[threadClassMethods.PNG|800]]
### The Main Thread
- When a Java program starts up, one thread begins running immediately. This is usually called the _main thread_ of your program, because it is the one that is executed when your program begins.
- The main thread is important for two reasons:
	- It is the thread from which other “**child**” threads will be spawned.
	- Often, it must be the last thread to **finish** execution because it performs various **shutdown** actions.
- Although the main thread is created automatically when your program is started, it can be controlled through a **Thread** object. To do so, you must obtain a reference to it by calling the method `currentThread( )`, which is a **public static** member of **Thread**.
- In this program, the [[0x01_Exception Handling#Using try and catch|try/catch]] block is around the loop because `sleep( )` method in **Thread** might throw an **InterruptedException**. This would happen if some other thread wanted to interrupt this sleeping one.
```Java
// Controlling the main Thread.
class CurrentThreadDemo {
    public static void main(String[] args) {
        Thread t = Thread.currentThread();
        System.out.println("Current thread: " + t);
        
        // Change the name of the thread
        t.setName("My Thread");
        System.out.println("After name change: " + t);
        
        try {
            for (int n = 5; n > 0; n--) {
                System.out.println(n);
                Thread.sleep(1000);
            }
        } catch (InterruptedException e) {
            System.out.println("Main thread interrupted");
        }
    }
}
```
- When `t` is used as an argument to `println()`. This displays, in order: the **name of the thread, its priority, and the name of its group** like the following:
```
Current thread: Thread[main,5,main]
After name change: Thread[My Thread,5,main]
```
- A _thread group_ is a data structure that controls the state of a collection of threads as a whole.
### Creating a Thread
- In the most general sense, you create a thread by instantiating an object of type Thread. Java defines two ways in which this can be accomplished:
	- You can implement the **Runnable** interface.
	- You can extend the **Thread** class, itself.
##### Implementing Runnable
- To implement Runnable, a class need only implement a single method called run( ), which is declared like this: `public void run()`.
- Inside `run()`, you will define the code that constitutes the new thread. It is important to understand that `run()` can call other methods, use other classes, and declare variables, just like the main thread can.
- The only difference is that `run()` establishes the entry point for another, concurrent thread of execution within your program. This thread will end when `run()` returns.
- After you create a class that implements Runnable, you will instantiate an object of type **Thread** from within that class. **Thread** defines several constructors. The one that we will use is shown here: `Thread(Runnable threadOb, String threadName)`.
- In this constructor, `threadOb` is an instance of a class that implements the **Runnable** interface. This defines where execution of the thread will begin. The name of the new thread is specified by `threadName`.
- After the new thread is created, it will not start running until you call its `start()` method, which is declared within **Thread**. In essence, `start()` initiates a call to `run()`. The `start()` method is shown here: `void start()`.
- Here is an example that creates a new thread and starts it running:
```Java
// Create a second thread.
class NewThread implements Runnable {
    Thread t;
    
    NewThread() {
        // Create a new, second thread
        t = new Thread(this, "Demo Thread");
        System.out.println("Child thread: " + t);
    }
    
    // This is the entry point for the second thread.
    public void run() {
        try {
            for(int i = 5; i > 0; i--) {
                System.out.println("Child Thread: " + i);
                Thread.sleep(500);
            }
        } catch (InterruptedException e) {
            System.out.println("Child interrupted.");
        }
        System.out.println("Exiting child thread.");
    }
}

class ThreadDemo {
    public static void main(String[] args) {
        NewThread nt = new NewThread(); // Create a new thread
        nt.t.start(); // Start the thread
        
        try {
            for (int i = 5; i > 0; i--) {
                System.out.println("Main Thread: " + i);
                Thread.sleep(1000);
            }
        } catch (InterruptedException e) {
            System.out.println("Main thread interrupted.");
        }
        
        System.out.println("Main thread exiting.");
    }
}
```
- Inside **NewThread**’s constructor, a new **Thread** object is created by the following statement: `t = new Thread(this, "Demo Thread");`.
- Passing `this` as the first argument indicates that you want the new thread to call the `run()` method on this object. Inside `main()`, `start()` is called, which starts the thread of execution beginning at the `run()` method. This causes the child thread’s **for** loop to begin. Next the main thread enters its **for** loop.
##### Extending Thread
- The extending class must override the `run()` method, which is the entry point for the new thread. As before, a call to `start()` begins execution of the new thread. Here is the preceding program rewritten to extend **Thread**:
```Java
// Create a second thread by extending Thread
class NewThread extends Thread {
    NewThread() {
        // Create a new, second thread
        super("Demo Thread");
        System.out.println("Child thread: " + this);
    }

    // This is the entry point for the second thread.
    public void run() {
        try {
            for (int i = 5; i > 0; i--) {
                System.out.println("Child Thread: " + i);
                Thread.sleep(500);
            }
        } catch (InterruptedException e) {
            System.out.println("Child interrupted.");
        }
        System.out.println("Exiting child thread.");
    }
}

class ExtendThread {
    public static void main(String[] args) {
        NewThread nt = new NewThread(); // Create a new thread
        nt.start(); // Start the thread

        try {
            for (int i = 5; i > 0; i--) {
                System.out.println("Main Thread: " + i);
                Thread.sleep(1000);
            }
        } catch (InterruptedException e) {
            System.out.println("Main thread interrupted.");
        }
        System.out.println("Main thread exiting.");
    }
}
```
### Using isAlive( ) and join( )
- Two ways exist to determine whether a thread has finished. First, you can call `isAlive()` on the thread. This method is defined by **Thread**, and its general form is shown here: `final boolean isAlive( )`.
- While `isAlive()` is occasionally useful, the method that you will more commonly use to wait for a thread to finish is called `join()`, shown here: 
	 `final void join( ) throws InterruptedException`.
- This method waits until the thread on which it is called **terminates**. Its name comes from the concept of the calling thread waiting until the specified thread joins it. Additional forms of `join()` allow you to specify a **maximum amount of time** that you want to wait for the specified thread to terminate.
- Here is an improved version of the preceding example that uses `join()` to ensure that the main thread is the last to stop.
```Java
// Using join() to wait for threads to finish.
class NewThread implements Runnable {
    String name; // name of thread
    Thread t;

    NewThread(String threadname) {
        name = threadname;
        t = new Thread(this, name);
        System.out.println("New thread: " + t);
    }

    // This is the entry point for thread.
    public void run() {
        try {
            for (int i = 5; i > 0; i--) {
                System.out.println(name + ": " + i);
                Thread.sleep(1000);
            }
        } catch (InterruptedException e) {
            System.out.println(name + " interrupted.");
        }
        System.out.println(name + " exiting.");
    }
}

class DemoJoin {
    public static void main(String[] args) {
        NewThread nt1 = new NewThread("One");
        NewThread nt2 = new NewThread("Two");
        NewThread nt3 = new NewThread("Three");

        // Start the threads.
        nt1.t.start();
        nt2.t.start();
        nt3.t.start();

        System.out.println("Thread One is alive: " + nt1.t.isAlive());
        System.out.println("Thread Two is alive: " + nt2.t.isAlive());
        System.out.println("Thread Three is alive: " + nt3.t.isAlive());

        // wait for threads to finish
        try {
            System.out.println("Waiting for threads to finish.");
            nt1.t.join();
            nt2.t.join();
            nt3.t.join();
        } catch (InterruptedException e) {
            System.out.println("Main thread Interrupted");
        }

        System.out.println("Thread One is alive: " + nt1.t.isAlive());
        System.out.println("Thread Two is alive: " + nt2.t.isAlive());
        System.out.println("Thread Three is alive: " + nt3.t.isAlive());
        System.out.println("Main thread exiting.");
    }
}
```
- Sample output from this program is shown here.
```
New thread: Thread[One,5,main] New thread: Thread[Two,5,main] 
New thread: Thread[Three,5,main]
Thread One is alive: true
Thread Two is alive: true
Thread Three is alive: true
Waiting for threads to finish.
One: 5
Two: 5
Three: 5
One: 4
Two: 4
Three: 4
One: 3 
Two: 3 
Three: 3 
One: 2 
Two: 2 
Three: 2 
One: 1 
Two: 1 
Three: 1 
Two exiting. 
Three exiting. 
One exiting.
Thread One is alive: false 
Thread Two is alive: false 
Thread Three is alive: false 
Main thread exiting.
```
### Thread Priorities
- To set a thread’s priority, use the `setPriority()` method, which is a member of **Thread**. This is its general form: `final void setPriority(int level)`.
- The value of `level` must be within the range `MIN_PRIORITY` and `MAX_PRIORITY`. Currently, these values are 1 and 10, respectively. To return a thread to default priority, specify `NORM_PRIORITY`, which is currently 5. These priorities are defined as **static final** variables within **Thread**.
- You can obtain the current priority setting by calling the `getPriority()` method of **Thread**, shown here: `final int getPriority()`.
### Synchronization
- When two or more threads need access to a shared resource, they need some way to ensure that the resource will be used by only **one thread** at a time. The process by which this is achieved is called _synchronization_.
- Key to synchronization is the concept of the **monitor**. A monitor is an object that is used as a [[0x18_Locks|mutually exclusive lock]]. Only one thread can **own** a monitor at a **given time**.
- When a thread acquires a lock, it is said to have _entered_ the monitor. All other threads attempting to enter the locked monitor will be suspended until the first thread _exits_ the monitor.
- These other threads are said to be _waiting_ for the monitor. A thread that owns a monitor can reenter the same monitor if it so desires.
- You can synchronize your code in either of two ways.
	- **Synchronized Methods**.
	- **The synchronized Statement**.
##### Using Synchronized Methods
- All objects have their own **implicit** monitor associated with them. To **enter** an object’s monitor, just **call** a method that has been modified with the `synchronized` keyword.
- While a thread is **inside** a synchronized method, all other threads that try to call it (or any other synchronized method) on the same instance **have to wait**.
- To exit the monitor and relinquish control of the object to the next waiting thread, the owner of the monitor simply returns from the synchronized method.
- To restrict access to only one thread at a time, you simply need to precede method's definition with the keyword `synchronized`, as shown here: 
	`synchronized MethodType method()`. This prevents other threads from entering the method while another thread is using it.
##### The synchronized Statement
- While creating synchronized methods within classes that you create is an easy and effective means of achieving synchronization, it will not work in all cases.
- Imagine that you want to synchronize access to objects of a class that was not designed for multithreaded access. That is, the class does not use **synchronized** methods.
- Fortunately, the solution to this problem is quite easy: You simply put calls to the methods defined by this class inside a **synchronized** block.
- This is the general form of the synchronized statement:
```Java
synchronized(objRef) { 
	// statements to be synchronized
}
```
- Here, `objRef` is a reference to the **object being synchronized**. A synchronized block ensures that a call to a synchronized method that is a member of `objRef`’s class occurs only after the current thread has successfully entered `objRef`’s monitor.
- Here is an example, using a synchronized block within the `run()` method:
```Java
// This program uses a synchronized block.
class Callme {
    void call(String msg) {
        System.out.print("[" + msg);
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            System.out.println("Interrupted");
        }
        System.out.println("]");
    }
}

class Caller implements Runnable {
    String msg;
    Callme target;
    Thread t;

    public Caller(Callme targ, String s) {
        target = targ;
        msg = s;
        t = new Thread(this);
    }

    // synchronize calls to call()
    public void run() {
        synchronized (target) { // synchronized block
            target.call(msg);
        }
    }
}

class Synch1 {
    public static void main(String[] args) {
        Callme target = new Callme();
        Caller ob1 = new Caller(target, "Hello");
        Caller ob2 = new Caller(target, "Synchronized");
        Caller ob3 = new Caller(target, "World");

        // Start the threads.
        ob1.t.start();
        ob2.t.start();
        ob3.t.start();

        // wait for threads to end
        try {
            ob1.t.join();
            ob2.t.join();
            ob3.t.join();
        } catch (InterruptedException e) {
            System.out.println("Interrupted");
        }
    }
}
```
### Interthread Communication
- To avoid polling, Java includes an elegant interprocess communication mechanism via the `wait()`, `notify()`, and `notifyAll()` methods. These methods are implemented as **final** methods in **Object**, so all classes have them. All three methods can be called only from within a **synchronized** context.
- The rules for using these methods are:
	- `wait()` tells the calling thread to give up the monitor and go to sleep until some other thread enters the same monitor and calls `notify()` or `notifyAll()`.
	- `notify()` wakes up a thread that called `wait()` on the same object.
	- `notifyAll()` wakes up all the threads that called `wait()` on the same object. One of the threads will be granted access.
- Although `wait()` normally waits until `notify()` or `notifyAll()` is called, there is a possibility that in very rare cases the waiting thread could be awakened due to a _spurious wakeup_. In this case, a waiting thread resumes without `notify()` or `notifyAll()` having been called.
- Because of this remote possibility, the Java API documentation recommends that calls to `wait()` should take place within a loop that checks the condition on which the thread is waiting.
- Consider the following sample program that incorrectly implements a simple form of the [[0x19_Condition Variables#The Producer/Consumer (Bounded Buffer) Problem|producer/ consumer]] problem.
```Java
// An incorrect implementation of a producer and consumer.
class Q {
    int n;

    synchronized int get() {
        System.out.println("Got: " + n);
        return n;
    }

    synchronized void put(int n) {
        this.n = n;
        System.out.println("Put: " + n);
    }
}

class Producer implements Runnable {
    Q q;
    Thread t;

    Producer(Q q) {
        this.q = q;
        t = new Thread(this, "Producer");
    }

    public void run() {
        int i = 0;
        while (true) {
            q.put(i++);
        }
    }
}

class Consumer implements Runnable {
    Q q;
    Thread t;

    Consumer(Q q) {
        this.q = q;
        t = new Thread(this, "Consumer");
    }

    public void run() {
        while (true) {
            q.get();
        }
    }
}

class PC {
    public static void main(String[] args) {
        Q q = new Q();
        Producer p = new Producer(q);
        Consumer c = new Consumer(q);

        // Start the threads.
        p.t.start();
        c.t.start();
        System.out.println("Press Control-C to stop.");
    }
}
```
- Although the `put()` and `get()` methods on `Q` are synchronized, nothing stops the producer from **overrunning** the consumer, nor will anything stop the consumer from **consuming** the same queue value **twice**.
```
Put: 1 
Got: 1 
Got: 1 
Got: 1 
Got: 1 
Got: 1 
Put: 2 
Put: 3 
Put: 4 
Put: 5 
Put: 6 
Put: 7 
Got: 7
```
- The proper way to write this program in Java is to use `wait()` and `notify()` to signal in both directions, as shown here:
```Java
// Corrected implementation of producer and consumer with proper synchronization
class Q {
    int n;
    boolean valueSet = false;

    synchronized int get() {
        while (!valueSet) {
            try {
                wait();
            } catch (InterruptedException e) {
                System.out.println("Interrupted");
            }
        }
        System.out.println("Got: " + n);
        valueSet = false;
        notify();
        return n;
    }

    synchronized void put(int n) {
        while (valueSet) {
            try {
                wait();
            } catch (InterruptedException e) {
                System.out.println("Interrupted");
            }
        }
        this.n = n;
        valueSet = true;
        System.out.println("Put: " + n);
        notify();
    }
}

class Producer implements Runnable {
    Q q;
    Thread t;

    Producer(Q q) {
        this.q = q;
        t = new Thread(this, "Producer");
    }

    public void run() {
        int i = 0;
        while (true) {
            q.put(i++);
        }
    }
}

class Consumer implements Runnable {
    Q q;
    Thread t;

    Consumer(Q q) {
        this.q = q;
        t = new Thread(this, "Consumer");
    }

    public void run() {
        while (true) {
            q.get();
        }
    }
}

class PC {
    public static void main(String[] args) {
        Q q = new Q();
        Producer p = new Producer(q);
        Consumer c = new Consumer(q);

        p.t.start();
        c.t.start();
        System.out.println("Press Control-C to stop.");
    }
}
```
- Inside `get()`, `wait()` is called. This causes its execution to suspend until `Producer` notifies you that some data is ready. When this happens, execution inside `get()` resumes. 
- After the data has been obtained, `get()` calls `notify()`. This tells `Producer` that it is okay to put more data in the queue. 
- Inside `put()`, `wait()` suspends execution until `Consumer` has removed the item from the queue. When execution resumes, the next item of data is put in the queue, and `notify()` is called. This tells `Consumer` that it should now remove it.
### Suspending, Resuming, and Stopping Threads
- A thread must be designed so that the `run()` method **periodically checks** to determine whether that thread should suspend, resume, or stop its own execution.
- Typically, this is accomplished by establishing a **flag** variable that indicates the execution state of the thread.
- As long as this flag is set to “_running_,” the `run()` method must continue to let the thread execute. If this variable is set to “_suspend_,” the thread must pause. If it is set to “_stop_,” the thread must terminate.
- The following example illustrates how the `wait()` and `notify()` methods that are inherited from **Object** can be used to control the execution of a thread.
```Java
// Suspending and resuming a thread the modern way.
class NewThread implements Runnable {
    String name; // name of thread
    Thread t;
    boolean suspendFlag;

    NewThread(String threadname) {
        name = threadname;
        t = new Thread(this, name);
        System.out.println("New thread: " + t);
        suspendFlag = false;
    }

    // This is the entry point for thread.
    public void run() {
        try {
            for (int i = 15; i > 0; i--) {
                System.out.println(name + ": " + i);
                Thread.sleep(200);
                synchronized (this) {
                    while (suspendFlag) {
                        wait();
                    }
                }
            }
        } catch (InterruptedException e) {
            System.out.println(name + " interrupted.");
        }
        System.out.println(name + " exiting.");
    }

    synchronized void mysuspend() {
        suspendFlag = true;
    }

    synchronized void myresume() {
        suspendFlag = false;
        notify();
    }
}

class SuspendResume {
    public static void main(String[] args) {
        NewThread ob1 = new NewThread("One");
        NewThread ob2 = new NewThread("Two");

        ob1.t.start(); // Start the thread
        ob2.t.start(); // Start the thread

        try {
            Thread.sleep(1000);
            ob1.mysuspend();
            System.out.println("Suspending thread One");
            Thread.sleep(1000);
            ob1.myresume();
            System.out.println("Resuming thread One");
            ob2.mysuspend();
            System.out.println("Suspending thread Two");
            Thread.sleep(1000);
            ob2.myresume();
            System.out.println("Resuming thread Two");
        } catch (InterruptedException e) {
            System.out.println("Main thread Interrupted");
        }

        // wait for threads to finish
        try {
            System.out.println("Waiting for threads to finish.");
            ob1.t.join();
            ob2.t.join();
        } catch (InterruptedException e) {
            System.out.println("Main thread Interrupted");
        }

        System.out.println("Main thread exiting.");
    }
}
```
### Obtaining a Thread’s State
- A thread can exist in a number of different states. You can obtain the current state of a thread by calling the `getState()` method defined by **Thread**. It is shown here:`Thread.State getState()`.
- It returns a value of type `Thread.State` that indicates the state of the thread at the time at which the call was made. `State` is an **enumeration** defined by **Thread**.
- Here are the values that can be returned by `getState()`:
	 ![[Thread states.PNG|750]]
	- ![[thread states relations.PNG]]
- It is important to understand that a thread’s state **may change after the call** to `getState()`. Thus, depending on the circumstances, the state obtained by calling `getState()` may _not reflect_ the actual state of the thread only a moment later. 
- For this reason, `getState()` is not intended to provide a means of synchronizing threads. It’s primarily used for _debugging or for profiling_ a thread’s run-time characteristics.
### Using a Factory Method to Create and Start a Thread
- In some cases, it is not necessary to separate the creation of a thread from the start of its execution. One way to do this is to use a **static factory method**.
- A _factory method_ is a method that returns an object of a class. Typically, factory methods are static methods of a class. 
- As it relates to creating and starting a thread, a factory method will create the thread, call `start()` on the thread, and then return a reference to the thread.
```Java
// A factory method that creates and starts a thread. 
public static NewThread createAndStart() {
	NewThread myThrd = new NewThread(); 
	myThrd.t.start(); 
	return myThrd;
}
```
- In cases in which you don’t need to keep a reference to the executing thread, you can sometimes create and start a thread with one line of code, without the use of a factory method.
```Java
new NewThread().t.start();
```
# Sources
- Java The Complete Reference - Chapter 11.
- [Java Concurrency and Multithreading](https://www.youtube.com/playlist?list=PLL8woMHwr36EDxjUoCzboZjedsnhLP1j4).