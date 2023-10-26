# Lab03

## Parallel Sorting and Searching Algorithms

### Odd-Even Transposition Sort (OETS)

`Odd-Even Transposition Sort` is a parallel sorting algorithm. It is based on the `Bubble Sort` technique, which compares every two consecutive numbers in the array and swaps them if the first is greater than the second to get an ascending order array. It consists of two phases – the odd phase and even phase:
    - Odd phase: Every odd indexed element is compared with the next even indexed element(considering 1-based indexing).
    - Even phase: Every even indexed element is compared with the next odd indexed element.


