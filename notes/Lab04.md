# Lab04

## Introduction to Java Multithreading

Unlike the C language, where implementing a thread and synchronization mechanisms depends on a library specific to a particular operating system (Linux, Windows, MacOSX), Java provides built-in support for working with threads directly within its SDK.

### Implementing a New Thread
In Java, there are two ways to implement a new thread.

1. Creating a class that implements the `Runnable` interface, which contains the `void run()` method. The code representing the logic of the new thread will be placed inside this method.

Example:
```Java
public class Task implements Runnable {
    public void run() {
        System.out.println("Hello from my new thread!");
    }
}
```

2. Creating a class that extends the `Thread class` and overrides the `void run()` method within it. Similar to the first case, the logic of the new thread will be implemented within this method.

Example:
```Java
public class MyThread extends Thread {
    public void run() {
        System.out.println("Hello from my new thread!");
    }
}
```

### Running the New Thread in Parallel

I. In the case where the mechanism of implementing the `Runnable interface` has been used, to create a new thread that contains the logic defined in the `Task class`, an instance of the `Task class` will be created and provided as a parameter to the constructor of the `Thread class`. To run the new thread created using the constructor in parallel, you will call its `public void start()` method.

Example:
```Java
public class Main {
    public static void main(String[] args) {
        Thread t = new Thread(new Task());
        t.start();
    }
}
```
II. If the `Thread class` has been extended to implement a new type of thread, a new thread can be created by directly instantiating the `MyThread class`. To start the parallel execution of this thread, the `public void start() `method inherited from the `Thread` class should be called.

Example:
```Java
public class Main {
    public static void main(String[] args) {
        MyThread t = new MyThread();
        t.start();
    }
}
```

> Attention‼️
> 
> There is a crucial distinction between the `start()` and `run()` methods of the Thread class! When the `run()` method is invoked, the code within it will execute sequentially within the thread that called
>
> it. When the `start()` method is invoked, the Java Virtual Machine (JVM) creates a new thread that will execute the instructions within the `run()` method concurrently with the thread that called the
>
> `start()` method.
>

### Waiting for the Termination of a Thread's Execution

To wait for the termination of a thread's execution, Java provides us with the `public final void join()` method of the `Thread class`. It's important to note that this method can throw exceptions like `InterruptedException`.

Example:
```Java
public class Main {
    public static void main(String[] args) {
        MyThread t = new MyThread();
        t.start();
 
        try {
            t.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

### Sending Parameters to a Thread and Obtaining Results from It

To send parameters to a thread, the constructor of the class encapsulating the thread logic will be used, regardless of the implementation method (either through inheritance or interface implementation). To obtain a result from a thread after it has finished execution (when the `join()` method call has returned), we can use getter methods that return the result or directly access the result field if it is defined as `public`.

```Java
public class Task extends Thread {
 
    private int id;
    private int result;
 
    public Task(int id) {
        this.id = id;
    }
 
    public void run() {
        result = id * id;
    }
 
    public int getResult() {
        return result;
    }
 
}
```

```Java
public class Main {
 
    public static void main(String[] args) {
        int NUMBER_OF_THREADS = 4;
        Thread[] t = new Thread[NUMBER_OF_THREADS];
 
        for (int i = 0; i < NUMBER_OF_THREADS; ++i) {
            t[i] = new Task(i);
            t[i].start();
        }
 
        for (int i = 0; i < NUMBER_OF_THREADS; ++i) {
            try {
                t[i].join();
                System.out.println("Thread " + i + " computed result " + ((Task)t[i]).getResult() + ".");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```

## The "synchronized" Keyword
The reserved keyword `synchronized` is used to define blocks of code and methods that represent **critical sections/regions**.

Example:
```
public class MyConcurrentArray<T> {
 
    private static int numberOfInstances = 0;
    private T[] content;
 
    public MyConcurrentArray(int size) {
        if (size > 0) {
            content = new T[size];
        } else {
          throw new RuntimeException("Negative size provided for MyConcurrentArray instantiation.");
        }
 
        synchronized(MyConcurrentArray.class) {
            ++numberOfInstances;
        }
    }
 
    // Synchronized method.
    public synchronized T get(int index) {
        if (index < content.length) {
            return content[index];
        }
        throw new IndexOutOfBoundsException(index + " is out of bounds for MyConcurrentArray of size " + content.length);
    }
 
    public void set(int index, T newT) {
        // Synchronized code block using the current instance (this) as a lock.
        synchronized(this) {
            if (index < content.length) {
                content[index] = newT;
            }
            throw new IndexOutOfBoundsException(index + " is out of bounds for MyConcurrentArray of size " + content.length);
        }
    }
 
    // Static synchronized method.
    public static synchronized int getNumberOfInstances(){
        return numberOfInstances;
    }
 
    public void size() {
        return content.length;
    }
}
```
Note that the `get` method is defined as `synchronized`. When a thread calls this method on an instance of the `MyConcurrentArray class`, it first needs to obtain the `monitor` associated with this object in order to execute the method body. If the `monitor` is not held by any other thread, the calling thread can execute the method's instructions. Otherwise, it will be `blocked (waiting)` until the monitor becomes available. After executing the method body, the method releases access to the monitor.

In the case of the `set` method, there is a `synchronized` block of instructions. It uses the `monitor` designated within the parentheses to provide exclusive access to the current thread within the critical region. In our example, it utilizes `the current instance of the object (this)`. The mechanism for entering and exiting the critical section is the same as described above for synchronized methods.

For `synchronized static` methods, an attempt is made to obtain the `monitor` associated with the class to execute their code. This happens because a static method belongs to the class, not to any instance of the class. Therefore, when exclusive access to a static field in a class is needed, the class is used in the header of the `synchronized` block (e.g., `MyClass.class` as presented in the constructor of the example).

> Attention‼️
> 
> Synchronized methods and code blocks in Java are reentrant. If a thread has acquired the monitor of an object, it can enter any other synchronized block or method associated with that object (implicitly
>
> with that monitor). This behavior is not activated by default for pthread_mutex_t defined in C (it can be achieved by specifying attributes at creation: PTHREAD_MUTEX_RECURSIVE).
>


## CyclicBarrier Class

A cyclic barrier is a mechanism for `(re)synchronizing` multiple threads, designed to block a specified number of threads and only allow them to proceed `synchronously` after all of them have invoked its resynchronization method. In Java, this mechanism is represented by the `CyclicBarrier class`. Upon instantiation, the number of threads to be resynchronized is specified. This thread synchronization mechanism is useful for `parallelized iterative algorithms` that require a synchronization step for threads before moving on to the next iteration of computation. It's important to note that invoking the `await()` method on the cyclic barrier can throw exceptions such as `BrokenBarrierException` or `InterruptedException`.

Examples:

```Java
public class Task extends Thread {
 
    private int id;
 
    public Task(int id) {
        this.id = id;
    }
 
    public void run() {
        while (!solved) {
            executeAlgorithmStep();
 
            try {
                // Resynchronizing threads for the next step of the algorithm.
                Main.barrier.await();
            } catch (BrokenBarrierException | InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```

```Java
public class Main {
 
    public static CyclicBarrier barrier;
 
    public static void main(String[] args) {
        int NUMBER_OF_THREADS = 4;
        barrier = new CyclicBarrier(NUMBER_OF_THREADS);
        Task[] t = new Task[NUMBER_OF_THREADS];
 
        for (int i = 0; i < NUMBER_OF_THREADS; ++i) {
            t[i] = new Task(id);
            t[i].start();
        }
 
        for (int i = 0; i < NUMBER_OF_THREADS; ++i) {
            try {
                t[i].join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```

## The "volatile" Keyword

The `volatile` keyword associated with a specific variable indicates that this variable cannot be optimized (placed in a register or excluded from conditions based on `compile-time invariants`). Every read or write operation associated with this variable will work directly with the RAM memory. This is useful in preventing `incompatible optimizations` when combined with modifying the value from another thread or reading outdated data from the core's register associated with the thread when the variable has received a new value from another thread.

```Java
public static volatile int counter = 0;
```
> Attention‼️
> 
> It is important to mention that volatile variables increase the program's execution time because every modification made to them must be propagated to other threads, while normal variables can be cached
>
> up to the processor's register level at some point (in the case of a loop counter).
> 

## Atomic Variables
In multithreading, the shared entity mostly leads to a problem when `concurrency` is incorporated. A shared entity such as, `mutable object` or `variable`, might be changed, which may result in the inconsistency of the program or database. So, it becomes crucial to deal with the shared entity while accessed concurrently. An atomic variable can be one of the alternatives in such a scenario. 

Java provides atomic classes such as `AtomicInteger`, `AtomicLong`, `AtomicBoolean` and `AtomicReference`. Objects of these classes represent the atomic variable of `int`, `long`, `boolea`n, and `object reference` respectively. These classes contain the following methods.
    - `set(int value)`: Sets to the given value
    - `get()`: Gets the current value
    - `lazySet(int value)`: Eventually sets to the given value
    - `compareAndSet(int expect, int update)`: Atomically sets the value to the given updated value if the current value == the expected value
    - `addAndGet(int delta)`: Atomically adds the given value to the current value
    - `decrementAndGet()`: Atomically decrements by one the current value

Example:

```Java
class Counter extends Thread {

	// Counter Variable
	int count = 0;

	// method which would be called upon
	// the start of execution of a thread
	public void run()
	{
		int max = 1_000_00_000;

		// incrementing counter
		// total of max times
		for (int i = 0; i < max; i++) {
			count++;
		}
	}
}

public class UnSafeCounter {
	public static void main(String[] args)
		throws InterruptedException
	{
		// Instance of Counter Class
		Counter c = new Counter();

		// Defining Two different threads
		Thread first = new Thread(c, "First");
		Thread second = new Thread(c, "Second");

		// Threads start executing
		first.start();
		second.start();

		// main thread will wait for
		// both threads to get completed
		first.join();
		second.join();

		// Printing final value of count variable
		System.out.println(c.count);
	}
}

```

Output:

`
137754082
`
In a `single thread environment`, the above-mentioned class will give the expected result only. But when a `multithreaded environment` is concerned, it may lead to `inconsistent` results. It happens because updating “var” is done in three steps: `Reading`, `updating`, and `writing`. If two or more threads try to update the value at the same time, then it may not update properly.

This issue can be solved using `Lock and Synchronization`, but not efficiently. 

1. `Using lock analogy or synchronization`: Synchronization or Locking can solve our problem, but it compromises time efficiency or performance. First, it mandates resource and thread scheduler to control lock. Second, when multiple threads attempt to acquire a lock, only one of them wins, rest are suspended or blocked. Suspending or blocking of threads can have a huge impact on performance.

```Java
import java.io.*;
import java.util.concurrent.locks.*;

class Counter extends Thread {

	// Counter Variable
	int count = 0;

	// method which would be called upon
	// the start of execution of a thread
	public synchronized void run()
	{

		int max = 1_000_00_000;

		// incrementing counter total of max times
		for (int i = 0; i < max; i++) {
			count++;
		}
	}
}

public class SynchronizedCounter {
	public static void main(String[] args)
		throws InterruptedException
	{
		// Instance of Counter Class
		Counter c = new Counter();

		// Defining Two different threads
		Thread first = new Thread(c, "First");
		Thread second = new Thread(c, "Second");

		// Threads start executing
		first.start();
		second.start();

		// main thread will wait for both
		// threads to complete execution
		first.join();
		second.join();

		// Printing final value of count variable
		System.out.println(c.count);
	}
}

```
Output: 

`
200000000
`

2. `Using Atomic variable`

```Java
import java.util.concurrent.atomic.AtomicInteger;

class Counter extends Thread {

	// Atomic counter Variable
	AtomicInteger count;

	// Constructor of class
	Counter()
	{
		count = new AtomicInteger();
	}

	// method which would be called upon
	// the start of execution of a thread
	public void run()
	{

		int max = 1_000_00_000;

		// incrementing counter total of max times
		for (int i = 0; i < max; i++) {
			count.addAndGet(1);
		}
	}
}

public class AtomicCounter {
	public static void main(String[] args)
		throws InterruptedException
	{
		// Instance of Counter Class
		Counter c = new Counter();

		// Defining Two different threads
		Thread first = new Thread(c, "First");
		Thread second = new Thread(c, "Second");

		// Threads start executing
		first.start();
		second.start();

		// main thread will wait for both
		// threads to complete execution
		first.join();
		second.join();

		// Printing final value of count variable
		System.out.println(c.count);
	}
}

```

Output:

`
200000000
`














