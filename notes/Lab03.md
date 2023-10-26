# Lab03

## Parallel Sorting and Searching Algorithms

### Odd-Even Transposition Sort (OETS)

`Odd-Even Transposition Sort` is a parallel sorting algorithm. It is based on the `Bubble Sort` technique, which compares every two consecutive numbers in the array and swaps them if the first is greater than the second to get an ascending order array. It consists of two phases – the odd phase and even phase:
    - `Odd phase`: Every odd indexed element is compared with the next even indexed element(considering 1-based indexing). Basically,  we compare all the odd-indexed elements with their immediate 
                   successors in the array and swap them if they’re out of order. 
    - `Even phase`: Every even indexed element is compared with the next odd indexed element. Basically, we repeat the process above but for all even-indexed elements and their successors.

Below is the traditional bubble sort algorithm:
```C
function bubbleSort(list) {
  sorted = false;
  while (!sorted) {
    sorted = true;
    for (var i = 0; i < list.length - 1; i++) {
      if (list[i] > list[i + 1]) {
        swap(list[i], list[i + 1]);
        sorted = false;
      }
    }
  }
}
```

and below is the Odd-Even Transposition Sort algorithm:

```C
function oddEvenSort(list) {
  for (var k = 0; k < list.length; k++) {
    for (i = 0; i < list.length - 1; i += 2) {
      if (list[i] > list[i + 1]) {
        swap(list[i], list[i + 1]);
      }
    }
    for (i = 1; i < list.length - 1; i += 2) {
      if (list[i] > list[i + 1]) {
        swap(list[i], list[i + 1]);
      }
    }
  }
}
```
As we see above, the Odd-Even Transposition Sort Algorithm allows us to parallelize the comparision and swapping of each phase without a race condition. i.e we can parallelly compute the comparision and swapping of **even index elements with their neighbours on the right side** and then **odd index elements with their neighbours on the right side**.
Example:
Let's say we have a list `[4, 3, 2, 1]`
#### First Iteration
    ##### `Odd Phase`:
        - Compare 4 (index 0) and 3 (index 1). Swap to get: [3, 4, 2, 1].
        - Simultaneously, compare 2 (index 2) and 1 (index 3). Swap to get: [3, 4, 1, 2].

`Result after Odd Phase: [3, 4, 1, 2]`

    ##### `Even Phase`:
        - Compare 3 (index 0) and 4 (index 1). No swap needed.
        - Simultaneously, compare 1 (index 2) and 2 (index 3). No swap needed.

`Result after Even Phase: [3, 4, 1, 2]`

#### Second Iteration
    ##### `Odd Phase`:
        - Compare 3 (index 0) and 4 (index 1). No swap needed.
        - Simultaneously, compare 1 (index 2) and 2 (index 3). No swap needed.

`Result after Odd Phase: [3, 4, 1, 2]`

    ##### `Even Phase`:
        - Compare 3 (index 0) and 4 (index 1). No swap needed.
        - Simultaneously, compare 1 (index 2) and 2 (index 3). No swap needed.

`Result after Even Phase: [3, 4, 1, 2]`
T
#### Third Iteration

    ##### `Odd Phase`:
        - Compare 3 (index 0) and 4 (index 1). No swap needed.
        - Simultaneously, compare 1 (index 2) and 2 (index 3). No swap needed.

`Result after Odd Phase: [3, 4, 1, 2]`

    ##### `Even Phase`:
        - Compare 3 (index 0) and 1 (index 2). Swap to get: [1, 4, 3, 2].
        - Simultaneously, compare 4 (index 1) and 2 (index 3). Swap to get: [1, 2, 3, 4].

`Result after Even Phase: [1, 2, 3, 4]`

Now the list is sorted!

