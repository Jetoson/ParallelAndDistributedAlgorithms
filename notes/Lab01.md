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



