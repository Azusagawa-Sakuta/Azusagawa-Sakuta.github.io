---
title: Monotonic Stack and Queue
date: 2025-03-28 16:29:57
tags: 
  - Algorithm
  - C++
categories:
  - Codes
  - Algorithms
---

## Sliding Window

### Explanation
The **Sliding Window** problem is a common algorithmic challenge that requires finding the minimum or maximum value within a fixed-length window in an array. A straightforward approach would be to compute the min/max of the entire array and check every possible window, but this results in a time complexity of `O(nlogn)`. However, a more efficient approach involves using a **monotonic queue or stack**.

A useful analogy is comparing this problem to **observing mountains from an airplane window**. Imagine you’re flying and want to determine the tallest mountains visible within a given distance. Initially, the highest mountain in the first window dominates the view, obscuring smaller peaks in front of it. As the window moves forward, the once-dominant peak might exit the range, revealing previously hidden mountains that now become relevant. This means we need a data structure to efficiently track the largest values while dynamically updating the window boundaries.

By using a **monotonic queue**, we can efficiently maintain the highest values within the window, ensuring that each element is processed only once, leading to an optimal `O(n)` time complexity.

### Example by text

Here is an example for the questions above. We want to get the maximum of a given array in every 3 element.

``` Plaintext
arr = {6, 10, 3, 4, 0, 8}

[{6], 10, 3, 4, 0, 8} -> max = 6
[{6, 10], 3, 4, 0, 8} -> max = 10
{[6, 10, 3], 4, 0, 8} -> max = 10
{6, [10, 3, 4], 0, 8} -> max = 10
{6, 10, [3, 4, 0], 8} -> max = 4
{6, 10, 3, [4, 0, 8]} -> max = 8
{6, 10, 3, 4, [0, 8}] -> max = 8
{6, 10, 3, 4, 0, [8}] -> max = 8
```

Where the **monotonic queue** was realized like:

``` Plaintext
6 (Push the first element)
10 (10 > 6, pop the 6, push next)
10 3 (3 < 10, push w/o poping)
10 4 (4 < 10, but 4 > 3, pop 3, push 4)
4 0 (10 is poped because the window slided, 0 < 4, push)
8 (8 > 4 and 8 > 0, pop all, push)
8 (nothing happened)
8 (nothing happened)
```
It's obvious that the first elements are the required values of the question.

### Example by C++ program

Here is a `C++` program. Notice that when we are realizing this project, we often store the index in monotonic queue, which can simplify the way to access values in array. We haven't used STL here. You can try use STL yourself.
``` cpp
// Realize the question with deque (w/o STL here)
T dq[ARRAY_SIZE];
T arr[ARRAY_SIZE];
T window_max[ARRAY_SIZE];
size_t head = 0;
size_t tail = 0;

for (size_t i = 1; i <= n; i++) {
    // When the back boundary is out of range, update the head pointer to follow the window. 
    while (head < tail && dq[head] >= i - window_size) head++;
    // Maximum of windows is the first element of deque.
    window_max[i] = arr[dq[head]];
    // If the value pushing is larger than the tail, pop the tail.
    while (head < tail && arr[dq[tail - 1]] < arr[i]) tail--;
    // Push the value.
    dq[tail++] = i;
}
```
**CAUTION**:
1. The second step and the third and fourth steps can exchange the position, because the update of tail will not affect the maximum. However, when you exchange these steps, the maximum value you get is **NOT** at the same window as the provided code.
2. In this code, tail is pointing at **(position of the last element) + 1**, so we use `dq[tail - 1]` to get the index of last element. And also, `dq[tail++]` to update the value of last element.

## TO-BE-CONTINUED