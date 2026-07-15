# Chocolate Distribution Problem

## 1. Problem Statement

Given an array `arr` of $n$ integers, where each value represents the number of chocolates in a packet. There are $m$ students. The task is to distribute chocolate packets such that:
1. Each student gets exactly one packet.
2. The difference between the maximum number of chocolates given to a student and the minimum number of chocolates given to a student is minimized.

### Input
- An array of integers `arr` of size `n` representing packet sizes.
- An integer `m` representing the number of students.

### Output
- An integer representing the minimum possible difference between the maximum and minimum chocolate packets distributed.

### Constraints
- $1 \le n \le 10^5$
- $1 \le arr[i] \le 10^9$
- $1 \le m \le n$

### Observations
1. If we choose a subset of size $m$ from the array, the difference between the max and min of this subset needs to be minimized.
2. If the packets are sorted in ascending order, any contiguous subarray of size $m$ represents a selection of packets where the minimum is at the start of the subarray, and the maximum is at the end of the subarray.

---

## 2. Interview Clarification

Before writing code, clarify the following with the interviewer:
1. **Can $m$ be greater than $n$?**
   - *If $m > n$, we cannot distribute packets to all students (since each student gets exactly one packet). In this case, return -1 or throw an error.*
2. **Can the array contain duplicate elements?**
   - *Yes, multiple packets can have the same number of chocolates.*
3. **What is the range of chocolate packet sizes?**
   - *Chocolate packet sizes can be up to $10^9$, so the differences fit in a standard 32-bit signed integer, but we should make sure we don't overflow during comparisons if there are negative numbers (which there shouldn't be for packet sizes).*

---

## 3. Thinking Process

Let's think step-by-step:
- **First Instinct (Brute Force):**
  We want to choose a subset of $m$ packets from the $n$ packets.
  The total number of ways to choose $m$ packets is given by the combination formula: $\binom{n}{m} = \frac{n!}{m!(n-m)!}$.
  For each combination, we can find the maximum element and the minimum element, and calculate their difference.
  The minimum difference among all combinations would be our answer.
  - For example, if $n = 5$ and $m = 3$, there are $\binom{5}{3} = 10$ combinations. For large $n$, this gets extremely slow (factorial/exponential complexity).

- **Optimizing by Sorting:**
  If we sort the array, say `arr` becomes sorted in ascending order:
  `arr[0] <= arr[1] <= arr[2] <= ... <= arr[n-1]`
  Suppose we choose any $m$ packets. To minimize the difference between the max and min of these $m$ packets, it makes intuitive sense that they should be as close to each other as possible.
  In a sorted array, the elements that are closest to each other are adjacent.
  Thus, any group of $m$ packets with the minimum difference between their maximum and minimum must be contiguous in the sorted array!
  So, after sorting, we only need to look at contiguous windows of size $m$.
  For a window starting at index `i` and ending at index `i + m - 1`:
  - The minimum element in this sorted window is `arr[i]`.
  - The maximum element is `arr[i + m - 1]`.
  - The difference is `arr[i + m - 1] - arr[i]`.
  We slide this window of size $m$ from `i = 0` to `i = n - m` and find the minimum difference.

---

## 4. Pattern Recognition

This problem fits the **Sliding Window** pattern combined with **Sorting**.
- **Why this topic?** We need to select a group of $m$ elements from $n$ elements to minimize the range (max - min).
- **Pattern:**
  - Sorting rearranges elements so that the range of any selection is minimized when the selection is contiguous.
  - Once sorted, the contiguous selection of size $m$ becomes a fixed-size sliding window of size $m$.

---

## 5. Brute Force Solution

### Algorithm
1. Generate all possible subsets of size $m$ using recursion (combinations).
2. For each subset, find the maximum element and the minimum element.
3. Compute the difference `max - min`.
4. Keep track of the minimum difference found across all subsets.
5. Return this minimum difference.

### Dry Run
Let `arr = [7, 3, 2, 4]`, `m = 3`.
- Subsets of size 3:
  - `{7, 3, 2}`: min = 2, max = 7. Difference = 5.
  - `{7, 3, 4}`: min = 3, max = 7. Difference = 4.
  - `{7, 2, 4}`: min = 2, max = 7. Difference = 5.
  - `{3, 2, 4}`: min = 2, max = 4. Difference = 2.
- Minimum difference is `2`.

### Complexity Analysis
- **Time Complexity:** $O(\binom{n}{m} \cdot m)$ because we generate all subsets of size $m$, and for each, we find the min and max in $O(m)$ time.
- **Space Complexity:** $O(m)$ for recursion stack.

### Pros and Cons
- **Pros:** Simple recursive logic; correct results for very small datasets.
- **Cons:** Factorial/Exponential time complexity; will crash or time out for $n > 20$.

### C++17 Code
```cpp
#include <vector>
#include <algorithm>
#include <climits>
#include <iostream>

class SolutionBrute {
private:
    void getMinDiffSubsets(const std::vector<int>& arr, int m, int index, 
                           std::vector<int>& currentSubset, int& minDiff) {
        if (currentSubset.size() == static_cast<size_t>(m)) {
            int currentMin = *std::min_element(currentSubset.begin(), currentSubset.end());
            int currentMax = *std::max_element(currentSubset.begin(), currentSubset.end());
            minDiff = std::min(minDiff, currentMax - currentMin);
            return;
        }
        
        if (index == static_cast<int>(arr.size())) {
            return;
        }
        
        // Option 1: Include current element
        currentSubset.push_back(arr[index]);
        getMinDiffSubsets(arr, m, index + 1, currentSubset, minDiff);
        currentSubset.pop_back(); // Backtrack
        
        // Option 2: Exclude current element
        getMinDiffSubsets(arr, m, index + 1, currentSubset, minDiff);
    }

public:
    int findMinDiff(const std::vector<int>& arr, int m) {
        int n = arr.size();
        if (m == 0 || n == 0 || m > n) return -1;
        
        int minDiff = INT_MAX;
        std::vector<int> currentSubset;
        getMinDiffSubsets(arr, m, 0, currentSubset, minDiff);
        return minDiff;
    }
};
```

---

## 6. Better Solution

> [!NOTE]
> Since sorting immediately improves the time complexity from exponential $O(\binom{n}{m})$ to $O(n \log n)$, there is no intermediate "better" solution. We jump directly to the optimal sorting and sliding window approach.

---

## 7. Optimal Solution

### Algorithm
1. If $m > n$ or $n = 0$, return $-1$.
2. Sort the chocolate packets array `arr` in ascending order.
3. Initialize `minDiff` to a very large value (`INT_MAX`).
4. Slide a window of size `m` from index `0` to `n - m`:
   - Let `i` be the start of the window.
   - The end of the window is at index `i + m - 1`.
   - Calculate the difference: `diff = arr[i + m - 1] - arr[i]`.
   - Update `minDiff = min(minDiff, diff)`.
5. Return `minDiff`.

### C++17 Code
```cpp
#include <vector>
#include <algorithm>
#include <climits>

class SolutionOptimal {
public:
    int findMinDiff(std::vector<int>& arr, int m) {
        int n = arr.size();
        
        // Edge Case: If there are fewer packets than students, distribution is impossible
        if (n < m || n == 0 || m == 0) {
            return -1; 
        }
        
        // Step 1: Sort the packets in ascending order
        std::sort(arr.begin(), arr.end());
        
        int minDifference = INT_MAX;
        
        // Step 2: Slide a window of size m
        for (int i = 0; i <= n - m; ++i) {
            // The maximum elements in this window is at arr[i + m - 1]
            // The minimum elements in this window is at arr[i]
            int currentDifference = arr[i + m - 1] - arr[i];
            
            // Track the minimum difference
            if (currentDifference < minDifference) {
                minDifference = currentDifference;
            }
        }
        
        return minDifference;
    }
};
```

---

## 8. Code Walkthrough

Let's break down the optimal solution:
1. `if (n < m || n == 0 || m == 0) return -1;`
   - Handles safety checks. If there are fewer packets than students, or packets/students count is invalid, we return `-1`.
2. `std::sort(arr.begin(), arr.end());`
   - Rearranges the chocolate packets. For example, `[7, 3, 2, 4, 9, 12, 56]` becomes `[2, 3, 4, 7, 9, 12, 56]`.
3. `int minDifference = INT_MAX;`
   - Initializes our tracking variable with the maximum possible integer value.
4. `for (int i = 0; i <= n - m; ++i)`
   - Iterates through the array. The condition `i <= n - m` ensures that the window of size `m` (from `i` to `i + m - 1`) fits within the bounds of the array.
5. `int currentDifference = arr[i + m - 1] - arr[i];`
   - Since the array is sorted, `arr[i]` is the minimum value in the window `[i ... i + m - 1]` and `arr[i + m - 1]` is the maximum value.
6. `minDifference = std::min(minDifference, currentDifference);`
   - Keeps track of the smallest difference we have seen so far.
7. `return minDifference;`
   - Returns the final minimized difference.

---

## 9. Dry Run

Let `arr = [12, 4, 7, 9, 2, 23, 25, 41, 30, 40, 28, 42, 30, 44, 48, 50]` and `m = 7`.
- **Step 1: Sort `arr`**
  `arr` becomes: `[2, 4, 7, 9, 12, 23, 25, 28, 30, 30, 40, 41, 42, 44, 48, 50]`
  $n = 16$, $m = 7$. Sliding window limit: $n - m = 9$.

| Iteration `i` | Window Start Index | Window End Index (`i+6`) | Min Packet (`arr[start]`) | Max Packet (`arr[end]`) | Difference | `minDifference` |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Initial** | — | — | — | — | — | `INT_MAX` |
| **i = 0** | 0 | 6 | 2 | 25 | $25 - 2 = 23$ | 23 |
| **i = 1** | 1 | 7 | 4 | 28 | $28 - 4 = 24$ | 23 |
| **i = 2** | 2 | 8 | 7 | 30 | $30 - 7 = 23$ | 23 |
| **i = 3** | 3 | 9 | 9 | 30 | $30 - 9 = 21$ | 21 |
| **i = 4** | 4 | 10 | 12 | 40 | $40 - 12 = 28$ | 21 |
| **i = 5** | 5 | 11 | 23 | 41 | $41 - 23 = 18$ | 18 |
| **i = 6** | 6 | 12 | 25 | 42 | $42 - 25 = 17$ | **17** |
| **i = 7** | 7 | 13 | 28 | 44 | $44 - 28 = 16$ | **16** |
| **i = 8** | 8 | 14 | 30 | 48 | $48 - 30 = 18$ | 16 |
| **i = 9** | 9 | 15 | 30 | 50 | $50 - 30 = 20$ | 16 |

**Final Output:** `16`.

---

## 10. Complexity Analysis

| Metric | Complexity | Reasoning |
| :--- | :--- | :--- |
| **Time Complexity** | $O(n \log n)$ | Sorting the array takes $O(n \log n)$ time. The subsequent sliding window loop runs $(n - m + 1)$ times, each taking $O(1)$ time, yielding $O(n)$ time. Thus, the sorting step dominates. |
| **Space Complexity** | $O(1)$ or $O(n)$ | Depending on the sorting algorithm implementation. C++ `std::sort` generally uses $O(\log n)$ auxiliary stack space. If we sort in-place, the auxiliary space is $O(1)$. |

---

## 11. Common Mistakes

1. **Incorrect Window End Index**:
   - Using `i + m` instead of `i + m - 1` as the end index of the window. Remember that a window starting at `i` with size `m` ends at index `i + m - 1`.
2. **Missing Edge Cases**:
   - Forgetting to check if $m > n$. If not checked, `i + m - 1` will result in out-of-bound errors.
3. **Array Indexing in the Loop Limit**:
   - Running the loop until `i < n` instead of `i <= n - m`, which also leads to out-of-bound array access.

---

## 12. Edge Cases

| Edge Case | Input | Expected Output | Why It Matters |
| :--- | :--- | :--- | :--- |
| $m > n$ (More students than packets) | `arr = [5, 6], m = 3` | `-1` | Prevents out-of-bounds array access |
| $m = n$ (Equal students and packets) | `arr = [3, 8, 9], m = 3` | `6` (9 - 3) | Window is the entire array |
| $m = 1$ (Single student) | `arr = [5, 10, 15], m = 1` | `0` | Difference is always 0 since max and min of 1 packet is the packet itself |
| Array contains duplicates | `arr = [5, 5, 5, 5], m = 2` | `0` | Difference between identical values is 0 |

---

## 13. Interview Explanation

*"To solve the Chocolate Distribution Problem, the goal is to choose $m$ packets from $n$ packets such that the difference between the largest and smallest packet sizes in our choice is minimized.
If we sort the packets in ascending order, the values that are closest to each other will be positioned adjacent to one another. Thus, any optimal group of $m$ packets must form a contiguous subarray of size $m$ in the sorted array.
Therefore, the optimal approach is to first sort the array. Then, we use a sliding window of size $m$. For each window starting at index $i$, the minimum value is at $arr[i]$ and the maximum value is at $arr[i + m - 1]$. We calculate the difference for all possible windows and return the minimum difference. This takes $O(n \log n)$ time due to sorting and $O(1)$ extra space."*

---

## 14. Follow-up Questions

1. **What if the packet sizes are too large to fit in integer types?**
   - *Use `long long` in C++ to prevent overflow, although for packet count, subtraction of positive integers will not overflow standard integer limits if they are positive and fit in 64-bit.*
2. **Can we solve this in $O(n)$ if the values are bounded?**
   - *Yes, if the packet sizes have a small range, we could use Counting Sort or Radix Sort to sort the array in $O(n)$ time, reducing the overall time complexity to $O(n)$.*

---

## 15. Variations

1. **Min Diff between Max and Min of K Elements**: Exactly the same problem, parameterized with $k$ instead of $m$.
2. **Fairness in Distribution**: Minimizing other metrics such as standard deviation or variance of the distributed values (would require prefix sums or dynamic programming).

---

## 16. Revision Notes

- **Sorting Key**: Contiguous subarray of size $m$ contains the optimal selection.
- **Window boundaries**: `arr[i]` (min) and `arr[i + m - 1]` (max).
- **Time Complexity**: dominated by sorting ($O(n \log n)$).
- **Mnemonic**: "Sort, slide size $m$, find min gap."

---

## 17. Memorization Version

```cpp
// Optimal Chocolate Distribution
int findMinDiff(vector<int>& arr, int m) {
    int n = arr.size();
    if (n < m) return -1;
    sort(arr.begin(), arr.end());
    int minDiff = INT_MAX;
    for (int i = 0; i <= n - m; ++i) {
        minDiff = min(minDiff, arr[i + m - 1] - arr[i]);
    }
    return minDiff;
}
```
*Time: $O(n \log n)$, Space: $O(1)$*
