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
















