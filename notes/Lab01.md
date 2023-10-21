# Introduction to Parallel Programming with Pthreads

## Introduction 

In the world of **Serial Computation**, a problem at hand is divided in to a set of discret instructions which will be executed by a **single** processor one after the other. At any moment in time one instruction will be executed. But in the world of **Parallel Computation** things are a bit different. Multiple computation resources can be deployed to solve a problem in a relatively small period of time.

### To achive parallelism:
1. The problem must first be divided into discrete components that can be solved concurrently.
2. Eeach such component must be further divided into a set of instructions.
3. Instructions from different components of the problem can be executed simultaneously on different processors.
To accomplish this process, a **mechanism** for coordinating the execution of different components of the problem is required.

### To efficiently parallelize a problem:
1. It needs to be logically divisible into separate components that can be executed simultaneously
2. The parallel execution time of these components should be shorter when multiple computing resources are available compared to when only a single resource is available
3. We need either a multi-processor/multi-core machine or an arbitrary number of such computing machines connected through a network (extending the concept of parallel programming to distributed programming).

 ### Design considerations:
- How to partition the problem?
- How to balance the workload?
- How to handle communication between parallel-running components?
- What data dependencies exist?
- How to synchronize the parallel components of the program?
- How much effort is required to parallelize a problem?

## Threads

A thread is defined as an independent stream of instructions that can be scheduled by the operating system. It really is an execution context, which is all the information a CPU needs to execute a stream of instructions.

Difference among **process** and **thread** from Stackoverflow:
> A thread is an execution context, which is all the information a CPU needs to execute a stream of instructions.
> Suppose you're reading a book, and you want to take a break right now, but you want to be able to come back and resume reading from the exact point where you stopped. One way to achieve that is by jotting > down the page number, line number, and word number. So your execution context for reading a book is these 3 numbers.
>
> If you have a roommate, and she's using the same technique, she can take the book while you're not using it, and resume reading from where she stopped. Then you can take it back, and resume it from
> where you were.
>
> Threads work in the same way. A CPU is giving you the illusion that it's doing multiple computations at the same time. It does that by spending a bit of time on each computation. It can do that because it > has an execution context for each computation. Just like you can share a book with your friend, many tasks can share a CPU.
>
> On a more technical level, an execution context (therefore a thread) consists of the values of the CPU's registers.
>
> Last: threads are different from processes. A thread is a context of execution, while a process is a bunch of resources associated with a computation. A process can have one or many threads.
> 
> Clarification: the resources associated with a process include memory pages (all the threads in a process have the same view of the memory), file descriptors (e.g., open sockets), and security credentials > (e.g., the ID of the user who started the process).


> A thread is an independent set of values for the processor registers (for a single core). Since this includes the Instruction Pointer (aka Program Counter), it controls what executes in what order. It also > includes the Stack Pointer, which had better point to a unique area of memory for each thread or else they will interfere with each other.
>
> Threads are the software unit affected by control flow (function call, loop, goto), because those instructions operate on the Instruction Pointer, and that belongs to a particular thread. Threads are often > scheduled according to some prioritization scheme (although it's possible to design a system with one thread per processor core, in which case every thread is always running and no scheduling is needed).
>
> In fact the value of the Instruction Pointer and the instruction stored at that location is sufficient to determine a new value for the Instruction Pointer. For most instructions, this simply advances the > IP by the size of the instruction, but control flow instructions change the IP in other, predictable ways. The sequence of values the IP takes on forms a path of execution weaving through the program code, > giving rise to the name "thread".


![Thread](https://randu.org/tutorials/threads/images/process.png)

N.B Threads contain only necessary information, such as a stack(for local variables, function arguments, return values), a copy of the registers, program counter and any thread specific data to allow them to be scheduled individually. Othe data is shared within the process between all threads.

### Analogy:
> Isn't that something you put through an eye of a sewing needle?
>
>   Yes.
>
> How does it relate to programming then?
> 
> Think of sewing needles as the processors and the threads in a program as the thread fiber. If you had two needles but only one thread, it would take longer to finish the job (as one needle is idle)
>  than if you split the thread into two and used both needles at the same time. Taking this analogy a little further, if one needle had to sew on a button (blocking I/O), the other needle could
> continue doing other useful work even if the other needle took 1 hour to sew on a single button. If you only used one needle, you would be ~1 hour behind! 


### Traits of programs that can benefit from a multi-threaded implementation:
- They contain computational components that can run in parallel
- They have data that can be operated on in parallel
- They occasionally block while waiting for I/O
- They need to respond to asynchronous events
- Certain execution components have higher priority than others.

### Terminology
- **Lightweight Process (LWP)** can be thought of as a virtual CPU where the number of LWPs is usually greater than the number of CPUs in the system. Thread libraries communicate with LWPs to schedule threads. LWPs are also sometimes referred to as kernel threads.
- **X-to-Y model.** The mapping between LWPs and Threads. Depending upon the operating system implementation and/or user-level thread library in use, this can vary from 1:1, X:1, or X:Y. Linux, some BSD kernels, and some Windows versions use the 1:1 model. User-level threading libraries are commonly in the X:1 class as the underlying kernel does not have any knowledge of the user-level threads. The X:Y model is used in Windows 7.
- **Contention Scope** is how threads compete for system resources (i.e. scheduling).
- **Bound threads** have system-wide contention scope, in other words, these threads contend with other processes on the entire system.
- **Unbound threads** have process contention scope.
- **Thread-safe** means that the program protects shared data, possibly through the use of mutual exclusion.
- **Reentrant** code means that a program can have more than one thread executing concurrently.
- **Async-safe** means that a function is reentrant while handling a signal (i.e. can be called from a signal handler).
- **Concurrency vs. Parallelism** - They are not the same! Parallelism implies simultaneous running of code (which is not possible, in the strict sense, on uniprocessor machines) while concurrency implies that many tasks can run in any order and possibly in parallel.

### Amdahl's Law and the Pareto Principle
 Threads can provide benefits... for the right applications! Don't waste your time multithreading a portion of code or an entire program that isn't worth multithreading.

**Gene Amdahl** argued the theoretical maximum improvement that is possible for a computer program that is parallelized, under the premise that the program is strongly scaled (i.e. the program operates on **a fixed problem size**). His claim is a well known assertion known as **Amdahl's Law**. Essentially, Amdahl's law states that: 
> the speedup of a program due to parallelization can be no larger than the inverse of the portion of the program that is immutably sequential.
For example, if 50% of your program is not parallelizable, then you can only expect a maximum speedup of 2x, regardless the number of processors you throw at the problem. Of course many problems and data sets that parallel programs process are not of fixed size or the serial portion can be very close to zero. What is important here, is to understand that most interesting problems that are solved by computer programs tend to have some limitations in the amount of parallelism that can be effectively expressed (or introduced by the very mechanism to parallelize) and exploited as threads or some other parallel construct.

It must be underscored how important it is to understand the problem the computer program is trying to solve first, before simply jumping in head first. Careful planning and consideration of not only what the program must attack in a parallel fashion and the means to do so by way of the algorithms employed and the vehicle for which they are delivered must be performed.

There is a common saying: 
> "90% of processor cycles are spent in 10% of the code."
This is more formally known as the **Pareto Principle**. Carefully analyze your code or your design plan; don't spend all of your time optimizing/parallelizing the 90% of the code that doesn't matter much!

## Pthreads (POSIX Threads)
The POSIX thread libraries are a C/C++ thread API based on standards. It enables the creation of a new concurrent process flow. It works well on multi-processor or multi-core systems, where the process flow may be scheduled to execute on another processor, increasing speed through parallel or distributed processing.

From a programmer's perspective, Pthreads are defined as a set of types and functions for the C language, implemented in a header file named pthread.h and a library named pthread (although, in some Pthreads implementations, this library may be included in another library).

N.B. Remember the following basics when working on threads

- Thread operations include thread **creation**, **termination**, **synchronization (joins,blocking)**, **scheduling**, **data management** and **process interaction**.
- A thread does not maintain a list of created threads, nor does it know the thread that created it.
- All threads within a process share the same address space.
- Threads in the same process share:
        - Process instructions
        - Most data
        - open files (descriptors)
        - signals and signal handlers
        - current working directory
        - User and group id
- Each thread has a unique:
        - Thread ID
        - set of registers, stack pointer
        - stack for local variables, return addresses
        - signal mask
        - priority
        - Return value: errno
- pthread functions return "0" if OK.


### Compiling running in C
```C
$ gcc -o program program.c -lpthread
$ ./program
```

### Thread Creation and Termination:
In a C program with Pthreads, initially, there is a single thread of execution called the **main thread**. Any other thread must be explicitly created and started by the programmer, and this is done through the **pthread_create** function, which can be called as many times as needed and from anywhere in the code. When we call **pthread_create**, the newly created thread will run in parallel with the main thread. This means that all the code after the **pthread_create** function call will execute concurrently with the code of the new thread. This function has the following signature:
```C
int pthread_create(pthread_t * thread, 
                       const pthread_attr_t * attr,
                       void * (*start_routine)(void *), 
                       void *arg);
    
```
Arguments:
- **Thread** - returns the thread id. (unsigned long int defined in bits/pthreadtypes.h)
- **attr** - Set to NULL if default thread attributes are used. (else define members of the struct pthread_attr_t defined in bits/pthreadtypes.h) Attributes include:
        - detached state (joinable? Default: PTHREAD_CREATE_JOINABLE. Other option: PTHREAD_CREATE_DETACHED)
        - scheduling policy (real-time? PTHREAD_INHERIT_SCHED,PTHREAD_EXPLICIT_SCHED,SCHED_OTHER)
        - scheduling parameter
        - inheritsched attribute (Default: PTHREAD_EXPLICIT_SCHED Inherit from parent thread: PTHREAD_INHERIT_SCHED)
        - scope (Kernel threads: PTHREAD_SCOPE_SYSTEM User threads: PTHREAD_SCOPE_PROCESS Pick one or the other not both.)
        - guard size
        - stack address (See unistd.h and bits/posix_opt.h _POSIX_THREAD_ATTR_STACKADDR)
        - stack size (default minimum PTHREAD_STACK_SIZE set in pthread.h),
- **void \* (\*start_routine)** - pointer to the function to be threaded. Function has a single argument: pointer to void.
- **\*arg** - pointer to argument of function. To pass multiple arguments, send a pointer to a structure.

Signature of **pthread_exit** function: 
```C
    void pthread_exit(void *retval);
```
Arguments:
    **retval** - Return value of thread.
This routine kills the thread. The **pthread_exit** function never returns. If the thread is not detached, the thread id and return value may be examined from another thread by using pthread_join.
Note: the return pointer *retval, must not be of local scope otherwise it would cease to exist once the thread terminates. 

```C
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>

void *print_message_function( void *ptr );

main()
{
     pthread_t thread1, thread2;
     char *message1 = "Thread 1";
     char *message2 = "Thread 2";
     int  iret1, iret2;

    /* Create independent threads each of which will execute function */

     iret1 = pthread_create( &thread1, NULL, print_message_function, (void*) message1);
     iret2 = pthread_create( &thread2, NULL, print_message_function, (void*) message2);

     /* Wait till threads are complete before main continues. Unless we  */
     /* wait we run the risk of executing an exit which will terminate   */
     /* the process and all threads before the threads have completed.   */

     pthread_join( thread1, NULL);
     pthread_join( thread2, NULL); 

     printf("Thread 1 returns: %d\n",iret1);
     printf("Thread 2 returns: %d\n",iret2);
     exit(0);
}

void *print_message_function( void *ptr )
{
     char *message;
     message = (char *) ptr;
     printf("%s \n", message);
}
```

N.B. 
The maximum number of threads that can be created by a process depends on the **Pthreads** implementation, but generally, it is not recommended to have more threads than the number of CPU cores on the machine you are running on, due to overhead.

### Thread Synchronization:
 The threads library provides three synchronization mechanisms:
    1. **mutexes** - Mutual exclusion lock: Block access to variables by other threads. This enforces exclusive access by a thread to a variable or set of variables.
    2. **joins** - Make a thread wait till others are complete (terminated).
    3. **condition variables** - data type **pthread_cond_t**
    
A **join** is performed when one wants to wait for a thread to finish. A thread calling routine(i.e. The main thread) may launch multiple threads then wait for them to finish to get the results. One wait for the completion of the threads with a join. By calling **pthread_join** function, we can ensure that the calling thread blocks until the other thread finishes its processing (i.e., completes the thread function).  If one of the secondary threads has already completed its execution before the **pthread_join** call, the function will return instantly.The **pthread_join** function has the following signature:
```C
int pthread_join(pthread_t thread, void **retval);
```
The **thread** parameter represents the identifier of the thread we are waiting for, and **retval** represents the return value of the expected thread function (and can be set to **NULL** if we don't need this information). The function returns 0 in case of success or an error code otherwise.

Example C Program:
```C
#include <stdio.h>
#include <pthread.h>

#define NTHREADS 10
void *thread_function(void *);
pthread_mutex_t mutex1 = PTHREAD_MUTEX_INITIALIZER;
int  counter = 0;

main()
{
   pthread_t thread_id[NTHREADS];
   int i, j;

   for(i=0; i < NTHREADS; i++)
   {
      pthread_create( &thread_id[i], NULL, thread_function, NULL );
   }

   for(j=0; j < NTHREADS; j++)
   {
      pthread_join( thread_id[j], NULL); 
   }
  
   /* Now that all threads are complete I can print the final result.     */
   /* Without the join I could be printing a value before all the threads */
   /* have been completed.                                                */

   printf("Final counter value: %d\n", counter);
}

void *thread_function(void *dummyPtr)
{
   printf("Thread number %ld\n", pthread_self());
   pthread_mutex_lock( &mutex1 );
   counter++;
   pthread_mutex_unlock( &mutex1 );
}
```

![CheatSheet](https://imgur.com/a/xJe8caU)




