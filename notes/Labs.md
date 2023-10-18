# Lab01




















# Lab02


## Synchronization elements in Pthreads

### Introduction
When two or more threads can access shared data and they try to change it at the same time, **Race Condition** occurs. Because the thread scheduling algorithm can swap between threads at any time, 
you don't know **the order** in which the threads will attempt to access the shared data. Therefore, the result of the change in data is dependent on the thread scheduling algorithm, i.e. both 
threads are **racing** to access/change the data. 

Problems often occur when one thread does a **check-then-act** (e.g. **check** if the value is X, then **act** to do something that depends on the value being X) and another thread does something to 
the value in between the **check** and the "act". E.g:

```C
if (x == 5) // The "Check"
{
   y = x * 2; // The "Act"

   // If another thread changed x in between "if (x == 5)" and "y = x * 2" above,
   // y will not be equal to 10.
}
```

**y** could be 10, or it could be **anything**, depending on whether another thread changed x in between the check and act. We have no real way of knowing.
In order to prevent race conditions from occurring, we use techniques as **mutex** and **Barrier**.

### Mutex (short for Mutual Exclusion)
Is a synchronization primitive that allows us to protect access to data when we have (potentially) **concurrent writes**. It functions as a **lock** that safeguards access to shared resources.
A mutex is used to define a **critical region**, which is a section of the program where at most one thread can be at any given time. If a thread T1 attempts to enter a critical region when another thread T0 is already inside, T1 will be blocked until T0 exits the critical region.

#### Best Mutex Analogy from Stackoverflow:

> When I am having a big heated discussion at work, I use a rubber chicken which I keep in my desk for just such occasions. The person holding the chicken is the only person who is allowed to talk.
> If you don't hold the chicken you cannot speak. You can only indicate that you want the chicken and wait until you get it before you speak. Once you have finished speaking, you can hand the chicken
> back to the moderator who will hand it to the next person to speak. This ensures that people do not speak over each other, and also have their own space to talk.
> 
> Replace Chicken with Mutex and person with thread and you basically have the concept of a mutex.
>
> Of course, there is no such thing as a rubber mutex. Only rubber chicken. My cats once had a rubber mouse, but they ate it.
>
> Of course, before you use the rubber chicken, you need to ask yourself whether you actually need 5 people in one room and would it not just be easier with one person in the room on their own doing
>  all the work. Actually, this is just extending the analogy, but you get the idea.


![Mutex](https://www.ictdemy.com/images/5689/threading/barrier.png)

A mutex has two primary operations: **locking (lock)** and **unlocking (unlock)**. Through locking, a thread marks entry into the critical region, indicating that any other thread attempting to perform a locking operation will have to wait. Unlocking signifies the exit from the critical region, granting permission for another thread to enter the critical region.

In Pthreads, a typical sequence of using a mutex looks like this:
   1. Create and initialize a mutex variable.
   2. Multiple threads attempt to lock the mutex (i.e., enter the critical region).
   3. Only one of them succeeds and holds the mutex (i.e., is inside the critical region).
   4. The thread inside the critical region performs various operations on the protected data.
   5. The thread holding the mutex unlocks it, exiting the critical region.
   6. Another thread enters the critical region and repeats the process.
   7. Finally, the mutex variable is destroyed.

#### Mutex usage example:
```C
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>

#define NUM_THREADS 5

// Shared counter
int counter = 0;

// Mutex for counter
pthread_mutex_t counter_mutex;

void* increment(void* arg) {
    for (int i = 0; i < 100000; i++) {
        // Lock the mutex
        pthread_mutex_lock(&counter_mutex);

        // Increment the shared counter
        counter++;

        // Unlock the mutex
        pthread_mutex_unlock(&counter_mutex);
    }
    return NULL;
}

int main() {
    pthread_t threads[NUM_THREADS];

    // Initialize the mutex
    if (pthread_mutex_init(&counter_mutex, NULL) != 0) {
        printf("Error initializing mutex.\n");
        return 1;
    }

    // Create multiple threads
    for (int i = 0; i < NUM_THREADS; i++) {
        if (pthread_create(&threads[i], NULL, increment, NULL) != 0) {
            printf("Error creating thread %d.\n", i);
            return 1;
        }
    }

    // Wait for all threads to finish
    for (int i = 0; i < NUM_THREADS; i++) {
        pthread_join(threads[i], NULL);
    }

    printf("Final counter value: %d\n", counter);

    // Destroy the mutex
    pthread_mutex_destroy(&counter_mutex);

    return 0;
}

```

I. In Pthreads, a mutex is represented by a variable of type pthread_mutex_t, and it is initialized using the following function:
```C
int pthread_mutex_init(pthread_mutex_t *mutex, const pthread_mutexattr_t *attr);
```
The first parameter represents a reference to the mutex variable, and the second parameter specifies the attributes of the newly created mutex (if default behavior is desired, the attr parameter can be left as NULL).

II. To deallocate a mutex, the following function is used, which takes a pointer to the mutex to be destroyed as a parameter:
```C
int pthread_mutex_destroy(pthread_mutex_t *mutex);
```

III. To lock a mutex, the following function is used, which takes the mutex as its parameter:
```C
int pthread_mutex_lock(pthread_mutex_t *mutex);
```

IV. The reverse operation, which specifies the exit from a critical region (i.e., unlocking the mutex), is performed using the following function:
```C
int pthread_mutex_unlock(pthread_mutex_t *mutex);
```
All four mutex functions return 0 if executed successfully or an error code otherwise.

⚠️**Attention**⚠️

If we want to protect a section of our program using a mutex, then every thread that accesses that section must both lock and unlock the same mutex variable. Additionally, if a thread attempts to unlock a mutex it does not hold (has not previously locked), it will result in undefined behavior.

### Barrier

Another primitive synchronization technique which ensures that no thread can proceed beyond the point where it is placed until all threads managed by the barrier reach that point. An example of usage is when we distribute a computation across multiple threads and want to proceed with the program execution only when each thread has finished its own calculations.In parallel computing, a barrier is a type of synchronization method where it enables multiple threads to wait until all threads have reached a particular point of execution(barrier) before any thread continues. Synchronization barriers can be shared across different threads and processes.

#### Best Analogy from Medium:
>Think of it like being out for a hike with some friends. You agree to wait for each other at the top of each hill (and you make a mental note how many are in your group). Say you’re the first one to reach > the top of the first hill. You’ll wait there at the top for your friends. One by one, they’ll arrive at the top, but nobody will continue until the last person in your group arrives. Once they do, you’ll > all proceed. Barrier Synchronization works in same way.

[Barrier Image](https://miro.medium.com/v2/resize:fit:4800/format:webp/1*dLShG53xSIHE1MAqVZptnw@2x.jpeg)

I. In Pthreads, a barrier is represented by the pthread_barrier_t type and initialized using the following function:
```C
int pthread_barrier_init(pthread_barrier_t *barrier, const pthread_barrierattr_t *attr, unsigned count);
```
The first parameter represents a reference to the barrier, the second parameter can be used to set barrier attributes (similar to mutex), and the last parameter denotes the number of threads that must reach the barrier for it to be released. This means that the barrier has an internal counter that counts the threads waiting for its release. When the counter reaches the number set during the barrier initialization, the threads can resume their parallel execution.

II. To deallocate a barrier, the following function is used:
```C
int pthread_barrier_destroy(pthread_barrier_t *barrier);
```
To make a thread wait at a barrier (to "set a barrier" in code), the following function is used:
```C
int pthread_barrier_wait(pthread_barrier_t *barrier);
```
The function above will return **PTHREAD_BARRIER_SERIAL_THREAD** for a single arbitrary thread from the barrier and 0 for all others. If the function encounters any errors, it will return an error code.

⚠️**Attention**⚠️

Every thread that needs to wait at the barrier will call the above function on the same variable of type pthread_barrier_t. If the number of threads calling pthread_barrier_wait is less than the parameter with which the barrier was initialized, it will never be unblocked.


















