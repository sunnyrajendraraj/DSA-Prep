# Kadane's Algorithm – Maximum Subarray Sum

---

## 1. Problem Statement

Given an integer array `nums`, find the contiguous subarray (containing at least one number) which has the largest sum and return its sum. Additionally, determine the start and end indices of the subarray to print the actual elements.

*   **Input:** An integer array `nums` of size $N$.
*   **Output:** The maximum subarray sum (integer) and the actual subarray.
*   **Constraints:**
    *   $1 \le N \le 10^5$
    *   $-10^4 \le nums[i] \le 10^4$
*   **Observations:**
    *   If all numbers are positive, the maximum subarray is the entire array.
    *   If all numbers are negative, the maximum subarray is the single largest (least negative) element.
    *   The subarray must be contiguous (elements must be adjacent).

---

## 2. Interview Clarification

Before writing code, clarify the following details with the interviewer:
1.  **Can the array contain all negative numbers?**
    *   *Answer:* Yes. In this case, the maximum sum is the value of the single maximum element (e.g., for `[-3, -1, -2]`, the maximum subarray sum is `-1`).
2.  **How should we handle empty input?**
    *   *Answer:* The constraints state $N \ge 1$, so the array will not be empty.
3.  **If there are multiple subarrays with the same maximum sum, which one should we return/print?**
    *   *Answer:* Any of them is acceptable.
4.  **Are we allowed to modify the original array?**
    *   *Answer:* Kadane's Algorithm can be implemented without modifying the input array.

---

## 3. Thinking Process

*   **First Instinct (Brute Force):** Generate all possible contiguous subarrays.
    *   A subarray is defined by a start index `i` and an end index `j`.
    *   For each pair `(i, j)`, calculate the sum of elements from `i` to `j`.
    *   Track the maximum sum seen so far.
    *   This takes $O(N^3)$ time because we have $O(N^2)$ pairs and computing each sum takes $O(N)$ time.
*   **Improvement (Better Solution):**
    *   Instead of recalculating the sum from scratch for each `(i, j)` pair, we can compute the sum incrementally.
    *   `Sum(i, j) = Sum(i, j - 1) + nums[j]`.
    *   This eliminates the inner loop for summation, reducing the time complexity to $O(N^2)$.
*   **Optimal Insight (Kadane's Algorithm):**
    *   Can we solve this in a single pass? Let's analyze the contribution of a number to a subarray.
    *   As we iterate through the array, we accumulate a running sum.
    *   If at any point the running sum becomes negative, it is always better to discard the accumulated subarray and start a new subarray from the next element. Why? Because adding a negative sum to any subsequent sequence will only decrease the potential maximum sum.
    *   Thus, at each step `i`, we decide whether to add `nums[i]` to our current subarray sum, or start a new subarray beginning at `nums[i]`.
    *   This logic can be formulated as:
        `currentSum = max(nums[i], currentSum + nums[i])`
    *   We update our `maxSum` at each step:
        `maxSum = max(maxSum, currentSum)`
    *   To print the actual subarray, we can track the starting index of the current subarray and update the global start and end indices whenever we find a new maximum sum.

---

## 4. Pattern Recognition

*   **Topic:** Arrays, Dynamic Programming.
*   **Pattern:** **Kadane's Algorithm / Sliding Window / Greedy**.
*   *Why this pattern?* We want to find a contiguous block with a maximum property. By greedily dropping the prefix of our subarray when its sum drops below $0$, we maintain the optimal contiguous sequence.

---

## 5. Brute Force Solution

### Algorithm
1.  Initialize `maxSum` to the lowest possible integer (`INT_MIN`).
2.  Run a loop `i` from `0` to `N - 1` (start of subarray).
3.  Run a nested loop `j` from `i` to `N - 1` (end of subarray).
4.  Run a third loop `k` from `i` to `j` to compute the sum of `nums[i...j]`.
5.  If `currentSum > maxSum`, update `maxSum = currentSum`.
6.  Return `maxSum`.

### Dry Run
Input: `nums = [-2, 1, -3]`
*   Subarrays:
    *   `[-2]` $\to$ sum = `-2`
    *   `[-2, 1]` $\to$ sum = `-1`
    *   `[-2, 1, -3]` $\to$ sum = `-4`
    *   `[1]` $\to$ sum = `1`
    *   `[1, -3]` $\to$ sum = `-2`
    *   `[-3]` $\to$ sum = `-3`
*   Max sum is `1`.

### Complexity
*   **Time Complexity:** $O(N^3)$ due to three nested loops.
*   **Space Complexity:** $O(1)$ auxiliary space.

### Code (C++17)
```cpp
#include <vector>
#include <climits>
#include <algorithm>

class SolutionBrute {
public:
    int maxSubArray(const std::vector<int>& nums) {
        int n = nums.size();
        int maxSum = INT_MIN;
        
        for (int i = 0; i < n; ++i) {
            for (int j = i; j < n; ++j) {
                int currentSum = 0;
                for (int k = i; k <= j; ++k) {
                    currentSum += nums[k];
                }
                maxSum = std::max(maxSum, currentSum);
            }
        }
        return maxSum;
    }
};
```

---

## 6. Better Solution (O(N^2))

### Algorithm
1.  Initialize `maxSum` to `INT_MIN`.
2.  Iterate `i` from `0` to `N - 1` (start of subarray).
3.  Initialize `currentSum = 0`.
4.  Iterate `j` from `i` to `N - 1` (end of subarray):
    *   Add `nums[j]` to `currentSum`.
    *   Update `maxSum = max(maxSum, currentSum)`.
5.  Return `maxSum`.

### Complexity
*   **Time Complexity:** $O(N^2)$ with two nested loops.
*   **Space Complexity:** $O(1)$ auxiliary space.

### Code (C++17)
```cpp
#include <vector>
#include <climits>
#include <algorithm>

class SolutionBetter {
public:
    int maxSubArray(const std::vector<int>& nums) {
        int n = nums.size();
        int maxSum = INT_MIN;
        
        for (int i = 0; i < n; ++i) {
            int currentSum = 0;
            for (int j = i; j < n; ++j) {
                currentSum += nums[j];
                maxSum = std::max(maxSum, currentSum);
            }
        }
        return maxSum;
    }
};
```

---

## 7. Optimal Solution (Kadane's Algorithm)

### Algorithm
1.  Initialize `maxSum` to `INT_MIN` and `currentSum` to `0`.
2.  To track the subarray indices:
    *   Initialize `start = 0`, `tempStart = 0`, and `end = 0`.
3.  Iterate `i` from `0` to `N - 1`:
    *   Add `nums[i]` to `currentSum`.
    *   If `currentSum > maxSum`:
        *   Update `maxSum = currentSum`.
        *   Update `start = tempStart`.
        *   Update `end = i`.
    *   If `currentSum < 0`:
        *   Reset `currentSum = 0`.
        *   Set `tempStart = i + 1` (propose starting the next subarray from index `i + 1`).
4.  Print/Return the maximum sum and the elements from index `start` to `end`.

### Dry Run
Input: `nums = [-2, 1, -3, 4, -1, 2, 1, -5, 4]`
*   `i = 0` (`-2`): `currentSum = -2`. `maxSum` becomes `-2`. `start = 0, end = 0`. Since `currentSum < 0`, reset `currentSum = 0`, `tempStart = 1`.
*   `i = 1` (`1`): `currentSum = 1`. `maxSum` becomes `1`. `start = 1, end = 1`.
*   `i = 2` (`-3`): `currentSum = -2`. `currentSum < 0`, reset `currentSum = 0`, `tempStart = 3`.
*   `i = 3` (`4`): `currentSum = 4`. `maxSum` becomes `4`. `start = 3, end = 3`.
*   `i = 4` (`-1`): `currentSum = 3`.
*   `i = 5` (`2`): `currentSum = 5`. `maxSum` becomes `5`. `start = 3, end = 5`.
*   `i = 6` (`1`): `currentSum = 6`. `maxSum` becomes `6`. `start = 3, end = 6`. Subarray is `[4, -1, 2, 1]`.
*   `i = 7` (`-5`): `currentSum = 1`.
*   `i = 8` (`4`): `currentSum = 5`.
*   Final maximum sum is `6`, subarray is `[4, -1, 2, 1]`.

---

### Beautiful C++17 Code

```cpp
#include <vector>
#include <climits>
#include <iostream>
#include <algorithm>

struct SubarrayResult {
    int maxSum;
    std::vector<int> subarray;
};

class SolutionOptimal {
public:
    SubarrayResult maxSubArray(const std::vector<int>& nums) {
        int n = nums.size();
        int maxSum = INT_MIN;
        int currentSum = 0;
        
        int start = 0;
        int end = 0;
        int tempStart = 0;
        
        for (int i = 0; i < n; ++i) {
            currentSum += nums[i];
            
            if (currentSum > maxSum) {
                maxSum = currentSum;
                start = tempStart;
                end = i;
            }
            
            // If sum becomes negative, discard the prefix
            if (currentSum < 0) {
                currentSum = 0;
                tempStart = i + 1;
            }
        }
        
        // Extract the subarray elements
        std::vector<int> sub(nums.begin() + start, nums.begin() + end + 1);
        
        return {maxSum, sub};
    }
};
```

---

## 8. Code Walkthrough

1.  `int maxSum = INT_MIN;`
    We initialize `maxSum` to the minimum possible integer to correctly handle arrays consisting entirely of negative numbers.
2.  `int currentSum = 0;`
    Accumulator for the current subarray sum.
3.  `int start = 0, end = 0, tempStart = 0;`
    *   `tempStart` keeps track of the active starting index of the candidate subarray.
    *   `start` and `end` record the boundaries of the best subarray found so far.
4.  `currentSum += nums[i];`
    Add the current element to the running sum.
5.  `if (currentSum > maxSum) { maxSum = currentSum; start = tempStart; end = i; }`
    If the current subarray sum exceeds the global maximum, update `maxSum` and commit the current boundaries.
6.  `if (currentSum < 0) { currentSum = 0; tempStart = i + 1; }`
    If the running sum goes below zero, resetting it to zero is optimal because a negative prefix will only reduce the sum of any future subarray starting from `tempStart = i + 1`.

---

## 9. Dry Run

Given `nums = [-2, 1, -3, 4]`, $N = 4$

| Index `i` | Value `nums[i]` | `currentSum` | `maxSum` | `tempStart` | `start` | `end` | Action |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| — | — | `0` | `INT_MIN` | `0` | `0` | `0` | Initialization |
| `0` | `-2` | `-2` | `-2` | `0` | `0` | `0` | `currentSum > maxSum` $\to$ update `maxSum`, `currentSum < 0` $\to$ reset to 0, `tempStart = 1` |
| `1` | `1` | `1` | `1` | `1` | `1` | `1` | `currentSum > maxSum` $\to$ update `maxSum`, `start = 1`, `end = 1` |
| `2` | `-3` | `-2` | `1` | `1` | `1` | `1` | `currentSum < 0` $\to$ reset to 0, `tempStart = 3` |
| `3` | `4` | `4` | `4` | `3` | `3` | `3` | `currentSum > maxSum` $\to$ update `maxSum`, `start = 3`, `end = 3` |

Final Subarray: `nums[3...3] = [4]`, Max Sum: `4`

---

## 10. Complexity Analysis

| Approach | Time Complexity | Space Complexity | Reasoning |
| :--- | :--- | :--- | :--- |
| **Brute Force** | $O(N^3)$ | $O(1)$ | Three nested loops to calculate all subarray sums. |
| **Better** | $O(N^2)$ | $O(1)$ | Two nested loops, utilizing cumulative sums. |
| **Optimal (Kadane's)** | $O(N)$ | $O(1)$ | Single pass over the array. Vector slicing at the end takes $O(\text{length of subarray})$ time. |

---

## 11. Common Mistakes

1.  **Initializing `maxSum` to `0`:** If the array contains only negative numbers (e.g., `[-5, -2, -3]`), the program will return `0` instead of `-2`.
    *   *Correction:* Always initialize `maxSum` to `INT_MIN` (or `nums[0]`).
2.  **Resetting `currentSum` before checking `maxSum`:** If `nums = [-1]`, resetting `currentSum` to `0` first will cause `maxSum` to become `0` instead of `-1`.
    *   *Correction:* Always compare and update `maxSum` first, then check if `currentSum` should be reset.
3.  **Incorrect tracking of starting index:** Setting `start = i` instead of using `tempStart` when updating `maxSum`.

---

## 12. Edge Cases

| Edge Case | Input Example | Expected Output | Why It Matters / How Code Handles It |
| :--- | :--- | :--- | :--- |
| **Single Element Array** | `[5]` | Sum: `5`, Subarray: `[5]` | Loops once; correctly returns the element. |
| **All Negative Numbers** | `[-3, -8, -2, -5]` | Sum: `-2`, Subarray: `[-2]` | `maxSum` updates to `-2`, then resets `currentSum` to 0. Correct. |
| **All Positive Numbers** | `[1, 2, 3]` | Sum: `6`, Subarray: `[1, 2, 3]` | `currentSum` never drops below 0. Correctly returns entire array. |
| **Alternating Signs** | `[1, -1, 1, -1]` | Sum: `1`, Subarray: `[1]` | Correctly tracks maximum without cumulative decay. |

---

## 13. Interview Explanation

> "To find the maximum contiguous subarray sum, we can explore three approaches.
> The brute-force method generates all possible subarrays and calculates their sums, which takes $O(N^3)$ time. We can improve this to $O(N^2)$ by calculating the sums incrementally.
> The optimal solution is Kadane's Algorithm, which runs in $O(N)$ time and uses $O(1)$ auxiliary space.
> The core intuition of Kadane's algorithm is that we scan the array left-to-right, maintaining a running sum. If the running sum ever becomes negative, we discard the subarray accumulated so far by resetting the running sum to zero, since a negative prefix will only reduce the sum of any subsequent elements. We track the maximum sum seen during the iteration.
> To reconstruct the actual subarray, we keep a temporary start index and update the final start and end indices whenever we find a new maximum sum."

---

## 14. Follow-up Questions

1.  **Can you solve this using Divide and Conquer?**
    *   *Answer:* Yes, we can divide the array in half. The maximum subarray sum is the maximum of:
        1. Maximum subarray sum in the left half.
        2. Maximum subarray sum in the right half.
        3. Maximum subarray sum that crosses the midpoint.
        This takes $O(N \log N)$ time and $O(\log N)$ stack space.
2.  **How would you handle a circular array?**
    *   *Answer:* For a circular array, the maximum subarray can wrap around the boundary. The maximum sum is either the normal maximum subarray sum (via Kadane's) or the total sum minus the minimum subarray sum (also found via Kadane's).

---

## 15. Variations

*   **Maximum Product Subarray (LeetCode 152):**
    *   *Difference:* We must track both the maximum and minimum products at each step, because multiplying two negative numbers can yield a positive maximum.
*   **Maximum Sum Circular Subarray (LeetCode 918):**
    *   *Difference:* Handles wrap-around subarrays.

---

## 16. Revision Notes

*   **Kadane's Rule:** Discard running sum if it falls below 0.
*   **Initialization Rule:** Initialize `maxSum = INT_MIN` to handle negative numbers.
*   **Subarray Reconstruction:** Use a `tempStart` pointer that moves to `i + 1` whenever `currentSum` resets.

---

## 17. Memorization Version

```cpp
int maxSubArray(vector<int>& nums) {
    int max_sum = INT_MIN, cur_sum = 0;
    for (int x : nums) {
        cur_sum += x;
        max_sum = max(max_sum, cur_sum);
        if (cur_sum < 0) cur_sum = 0;
    }
    return max_sum;
}
```
*Tip: Update `max_sum` before resetting `cur_sum` to zero.*
