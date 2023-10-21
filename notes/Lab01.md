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

## Pthreads (POSIX Threads)




