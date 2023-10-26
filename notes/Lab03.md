# Lab03

## Parallel Sorting and Searching Algorithms

### Odd-Even Transposition Sort (OETS)

`Odd-Even Transposition Sort` is a parallel sorting algorithm. It is based on the `Bubble Sort` technique, which compares every two consecutive numbers in the array and swaps them if the first is greater than the second to get an ascending order array. It consists of two phases – the odd phase and even phase:
    - `Odd phase`: Every odd indexed element is compared with the next even indexed element(considering 1-based indexing). Basically,  we compare all the odd-indexed elements with their immediate 
                   successors in the array and swap them if they’re out of order. 
    - `Even phase`: Every even indexed element is compared with the next odd indexed element. Basically, we repeat the process above but for all even-indexed elements and their successors.

The complexity of `Bubble Sort algorithm` is O(N^2^), as it finishes in at most N iterations of the array (where N is the number of elements in the array to be sorted). But given `P` threads of execution, the complexity of `OETS` algorithm will be `O(N/P*N)`, or `O(N)` for `P=N`.

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
Let's say we have a list `[4, 3, 2, 1]` and let's sort it using OETS algorithm.
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

N.B. The odd-even transposition sort algorithm was originally designed to run on processor arrays, where each processor holds a single value from the array to be sorted and can only communicate with the processor to the left and the one to the right.

### Shear sort (also known as row-column sort or snake-order sort)
The algorithm assumes that we are working with processors connected in a `matrix-like form`. In this setup, a processor can communicate with neighbors to the `left`, `right`, `above`, and `below`. If we imagine that processors are arranged in a matrix, the two phases of the shear sort algorithm are as follows:
    - Sort the matrix rows so that even rows have values sorted in ascending order, and odd rows have values sorted in descending order.
    - Sort the columns in ascending order.
    
It is guaranteed that the algorithm will sort the numbers in at most sup(log2N) + 1 phases, where N is the number of elements to be sorted. For this reason, the algorithm has a complexity of O(Nlog2N). The pseudocode of the algorithm is presented below.

```C
function shearSort(matrix) {
  for (k = 0; k < ceil(log2(matrix.lines * matrix.columns)) + 1; k++) {
    for (i = 0; i < matrix.lines; i += 2) {
      sortAscendingLine(i);
    }
    for (i = 1; i < matrix.lines; i += 2) {
      sortDescendingLine(i);
    }
    for (i = 0; i < matrix.columns; i++) {
      sortAscendingColumn(i);
    }
  }
}
```


