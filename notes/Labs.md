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


![Mutex](https://iq.opengenus.org/content/images/2023/07/A351_Mutex-and-Semaphore-in-OS2.png)

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
























