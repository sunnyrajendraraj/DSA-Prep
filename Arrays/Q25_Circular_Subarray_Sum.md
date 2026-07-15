# Maximum Circular Subarray Sum

## 1. Problem Statement

Given a circular integer array `arr` of size `N`, find the maximum possible sum of a non-empty subarray in the circular array.

A circular array means the end of the array connects to the beginning of the array. Formally, the next element of `arr[N-1]` is `arr[0]`. A subarray here can be a normal subarray (contiguous block without wrapping) or a circular subarray (wrapping from the end of the array back to the beginning). Note that a subarray cannot contain any element of the original array more than once (i.e., its maximum length is `N`).

### Input
- An integer array `arr` of size `N`.

### Output
- An integer representing the maximum subarray sum.

### Constraints
- $1 \le N \le 10^5$
- $-10^4 \le arr[i] \le 10^4$

### Observations
1. If the array is NOT circular, this is the classic **Maximum Subarray Sum** problem solvable in $O(N)$ time using **Kadane's Algorithm**.
2. If the maximum subarray wraps around, it contains a prefix of the array and a suffix of the array.
3. The elements that are *not* included in this wrapping subarray must form a contiguous subarray in the middle.
4. To maximize the sum of the wrapping prefix and suffix, we must minimize the sum of the excluded contiguous middle subarray.
5. $\text{Circular Subarray Sum} = \text{Total Sum} - \text{Minimum Subarray Sum}$.
6. If all elements in the array are negative, the minimum subarray sum will equal the total sum, making the circular sum 0 (representing an empty subarray). However, since the problem specifies a *non-empty* subarray, we must handle this edge case.

---

## 2. Interview Clarification

- **Q:** Can the subarray be empty?
  - **A:** No, the problem specifies a non-empty subarray.
- **Q:** Can the circular subarray span more than $N$ elements?
  - **A:** No, a subarray cannot contain duplicate indices of the original array. Its maximum length is $N$.
- **Q:** What should we return if all elements are negative?
  - **A:** We should return the maximum single element (i.e., the largest negative value), which is the maximum non-empty subarray sum.
- **Q:** Can we modify the input array?
  - **A:** Yes, unless specified otherwise, but achieving $O(1)$ auxiliary space without permanent modifications is preferred.

---

## 3. Thinking Process

### First Instinct (Brute Force)
To find the maximum subarray sum, we could generate all possible subarrays. For a circular array, a subarray can start at any index $i$ and have a length $L$ from $1$ to $N$. 
- For each start index $i \in [0, N-1]$:
  - For each length $L \in [1, N]$:
    - Compute the sum of the elements from index $i$ to $(i + L - 1) \bmod N$.
    - Keep track of the maximum sum seen.
- This approach requires $O(N^2)$ time to evaluate all subarrays and is too slow for $N = 10^5$.

### Developing the Optimal Insight
Let's analyze the two possible cases for the maximum circular subarray:

#### Case 1: The maximum subarray does not wrap around.
- The subarray lies completely within the array boundaries $[0, N-1]$.
- We can find its sum directly using the standard **Kadane's Algorithm**.
- Example: `[1, -2, 4, -1, 2]` $\rightarrow$ Maximum subarray is `[4, -1, 2]` with sum $5$.

```
[ 1, -2, 4, -1, 2 ]
        |-------| -> Max Subarray (No wrap)
```

#### Case 2: The maximum subarray wraps around.
- The subarray starts towards the end of the array, wraps around, and ends towards the beginning.
- It looks like: `[Prefix] + [Suffix]` where the middle elements are excluded.
- Let the total sum of the array be `total_sum`.
- Let the middle excluded elements form a contiguous subarray. To make the sum of the prefix and suffix as large as possible, the sum of this middle subarray must be as small as possible.
- $\text{Max Circular Sum} = \text{total\_sum} - \text{Min Subarray Sum}$.
- Example: `[8, -8, 9, -9, 10]`
  - `total_sum` = $8 + (-8) + 9 + (-9) + 10 = 10$.
  - The minimum subarray is `[-8, 9, -9]` or just `[-9]`. Let's find the absolute minimum contiguous subarray: it is `[-8, 9, -9]` with sum $-8$.
  - $\text{Max Circular Sum} = 10 - (-8) = 18$ (which corresponds to `[10]` wrapping around to `[8]`, sum = $18$).

```
[ 8,  -8,  9,  -9,  10 ]
|--|  |---------|  |---| -> Circular Subarray (Wraps around)
       Excluded
       Middle
```

### The All-Negatives Edge Case
What if the array is `[-3, -2, -3]`?
- Standard Kadane's Max = $-2$.
- Min Subarray Sum = $-8$ (the entire array).
- `total_sum` = $-8$.
- $\text{Max Circular Sum} = \text{total\_sum} - \text{Min Subarray Sum} = -8 - (-8) = 0$.
- Since $0 > -2$, our formula would return $0$, which corresponds to an empty subarray. But we must return a non-empty subarray, so the correct answer is $-2$.
- **Rule:** If `total_sum == min_subarray_sum`, it means all elements are negative. In this case, the maximum circular subarray sum is simply the maximum non-empty subarray sum (which is `max_subarray_sum` found by Kadane's).

---

## 4. Pattern Recognition

- **Pattern:** Kadane's Algorithm / Complementary Thinking.
- **Why this topic?** We need to find optimal contiguous ranges. Whenever a problem asks for maximum contiguous subarray sums, Kadane's is the first pattern to consider.
- **Why Circular?** By looking at the circular property as the complement of the non-circular property (Total - Min), we reduce the circular problem to two instances of the linear Kadane's problem.

---

## 5. Brute Force Solution

### Algorithm
1. Initialize `max_sum = INT_MIN`.
2. Loop through every starting index `i` from `0` to `N - 1`.
3. For each start index, initialize `current_sum = 0`.
4. Loop through lengths `L` from `1` to `N`.
5. Add the element at index `(i + L - 1) % N` to `current_sum`.
6. Update `max_sum = max(max_sum, current_sum)`.
7. Return `max_sum`.

### Dry Run
Input: `arr = [1, -2, 3]`
- `i = 0`:
  - `L = 1`: sum = `arr[0] = 1`. `max_sum` = 1.
  - `L = 2`: sum = `1 + arr[1] = 1 - 2 = -1`.
  - `L = 3`: sum = `-1 + arr[2] = 2`. `max_sum` = 2.
- `i = 1`:
  - `L = 1`: sum = `arr[1] = -2`.
  - `L = 2`: sum = `-2 + arr[2] = 1`.
  - `L = 3`: sum = `1 + arr[0] = 2`.
- `i = 2`:
  - `L = 1`: sum = `arr[2] = 3`. `max_sum` = 3.
  - `L = 2`: sum = `3 + arr[0] = 4`. `max_sum` = 4.
  - `L = 3`: sum = `4 + arr[1] = 2`.
Output: `4`

### Complexity Analysis
- **Time Complexity:** $O(N^2)$ because we have nested loops of size $N$.
- **Space Complexity:** $O(1)$ auxiliary space.

### Pros and Cons
- **Pros:** Extremely simple to understand and implement.
- **Cons:** TLE (Time Limit Exceeded) for $N \ge 10^5$.

### Brute Force C++17 Code
```cpp
#include <vector>
#include <algorithm>
#include <climits>

class Solution {
public:
    int maxSubarraySumCircularBrute(const std::vector<int>& arr) {
        int n = arr.size();
        int max_sum = INT_MIN;

        // Try every starting position
        for (int i = 0; i < n; ++i) {
            int current_sum = 0;
            // Try every subarray length from 1 to n
            for (int len = 1; len <= n; ++len) {
                int idx = (i + len - 1) % n;
                current_sum += arr[idx];
                max_sum = std::max(max_sum, current_sum);
            }
        }
        return max_sum;
    }
};
```

---

## 6. Better Solution

> [!NOTE]
> Since the brute force is $O(N^2)$ and the optimal solution is $O(N)$ with $O(1)$ space using a direct adaptation of Kadane's algorithm, there is no distinct intermediate "Better" solution. The transition goes directly from Brute Force to the Optimal Solution.

---

## 7. Optimal Solution (Dual Kadane's Algorithm)

### Algorithm
1. Compute the maximum subarray sum of the array using standard Kadane's algorithm. Let this be `max_normal`.
2. Compute the minimum subarray sum of the array using a modified Kadane's algorithm. Let this be `min_normal`.
3. Compute the sum of all elements in the array. Let this be `total_sum`.
4. If `total_sum == min_normal`, all elements are negative. Return `max_normal`.
5. Otherwise, return `max(max_normal, total_sum - min_normal)`.

### Why this is Optimal
It runs in a single pass (or two simple linear passes) over the array. Since we must read every element at least once to determine the maximum sum, $O(N)$ is the absolute lower bound for time complexity. We achieve this with $O(1)$ extra space.

### C++17 Optimal Code
```cpp
#include <vector>
#include <algorithm>
#include <numeric>

class Solution {
public:
    int maxSubarraySumCircular(const std::vector<int>& arr) {
        if (arr.empty()) return 0;

        int total_sum = 0;
        
        // Variables for finding maximum subarray sum (Standard Kadane)
        int max_so_far = arr[0];
        int curr_max = arr[0];
        
        // Variables for finding minimum subarray sum (Modified Kadane)
        int min_so_far = arr[0];
        int curr_min = arr[0];
        
        total_sum += arr[0];

        for (size_t i = 1; i < arr.size(); ++i) {
            total_sum += arr[i];
            
            // Standard Kadane to find maximum subarray sum
            curr_max = std::max(arr[i], curr_max + arr[i]);
            max_so_far = std::max(max_so_far, curr_max);
            
            // Modified Kadane to find minimum subarray sum
            curr_min = std::min(arr[i], curr_min + arr[i]);
            min_so_far = std::min(min_so_far, curr_min);
        }
        
        // If all numbers are negative, total_sum will equal min_so_far.
        // In this case, total_sum - min_so_far = 0, which corresponds to an empty subarray.
        // We must return the maximum single element (max_so_far).
        if (total_sum == min_so_far) {
            return max_so_far;
        }
        
        // Return the maximum of the non-wrapped and wrapped subarray sum
        return std::max(max_so_far, total_sum - min_so_far);
    }
};
```

---

## 8. Code Walkthrough

1. **Base Case Check**: `if (arr.empty()) return 0;` handles any potential empty input safely.
2. **Initialization**:
   - `total_sum` tracks the total sum of all elements.
   - `curr_max` and `max_so_far` are initialized to `arr[0]` to track the maximum contiguous subarray.
   - `curr_min` and `min_so_far` are initialized to `arr[0]` to track the minimum contiguous subarray.
3. **Iterative Pass**:
   - We loop from `i = 1` to `N - 1`.
   - Update `total_sum` by adding `arr[i]`.
   - For `curr_max`: decide whether to start a new subarray at `arr[i]` or extend the previous subarray (`curr_max + arr[i]`). Update `max_so_far` accordingly.
   - For `curr_min`: decide whether to start a new subarray at `arr[i]` or extend the previous subarray (`curr_min + arr[i]`). Update `min_so_far` accordingly.
4. **All-Negative Check**: If `total_sum == min_so_far`, we return `max_so_far` because wrapping around would require excluding all elements (leaving an empty subarray).
5. **Output**: We return `std::max(max_so_far, total_sum - min_so_far)`.

---

## 9. Dry Run

Let's dry run the optimal solution with `arr = [5, -3, 5]`.

### Variable Initialization
- `total_sum = 5`
- `curr_max = 5`, `max_so_far = 5`
- `curr_min = 5`, `min_so_far = 5`

### Step-by-Step Trace

| Step | `i` | `arr[i]` | `total_sum` | `curr_max` = `max(arr[i], curr_max + arr[i])` | `max_so_far` | `curr_min` = `min(arr[i], curr_min + arr[i])` | `min_so_far` |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| Init | - | - | 5 | 5 | 5 | 5 | 5 |
| 1 | 1 | -3 | 2 | `max(-3, 5-3) = 2` | `max(5, 2) = 5` | `min(-3, 5-3) = -3` | `min(5, -3) = -3` |
| 2 | 2 | 5 | 7 | `max(5, 2+5) = 7` | `max(5, 7) = 7` | `min(5, -3+5) = 2` | `min(-3, 2) = -3` |

### Final Computations
- `total_sum` = 7
- `max_so_far` = 7
- `min_so_far` = -3
- Is `total_sum == min_so_far`? `7 == -3` is `False`.
- Return `max(max_so_far, total_sum - min_so_far)` $\rightarrow$ `max(7, 7 - (-3))` $\rightarrow$ `max(7, 10)` $\rightarrow$ **`10`**.
- The circular subarray with sum 10 is `[5]` (index 2) wrapping to `[5]` (index 0).

---

## 10. Complexity Analysis

- **Time Complexity:** 
  - **Best Case:** $O(N)$ when the array is processed in a single pass.
  - **Average Case:** $O(N)$.
  - **Worst Case:** $O(N)$ as we only perform constant-time operations for each element.
- **Space Complexity:**
  - **Auxiliary Space:** $O(1)$ because we only use a few tracking variables.

---

## 11. Common Mistakes

1. **Incorrectly initializing Kadane's variables to 0**: If the array is `[-3, -2, -3]`, initializing `max_so_far` to 0 will output 0 instead of -2. Always initialize to `arr[0]`.
2. **Missing the all-negative numbers case**: If all elements are negative, `total_sum` will equal `min_so_far`, resulting in `total_sum - min_so_far = 0`. Since all elements are negative, 0 is greater than any element, but it represents an empty subarray which is invalid.
3. **Double counting elements**: Assuming wrapping around can include elements multiple times (e.g., circular subarray of length $> N$).

---

## 12. Edge Cases

| Edge Case | Input Example | Expected Output | Why it matters |
|:---|:---|:---|:---|
| **All Negative Elements** | `[-3, -2, -3]` | `-2` | Prevents returning 0 (empty subarray). |
| **All Positive Elements** | `[1, 2, 3]` | `6` | `total_sum - min` equals `max_so_far`. |
| **Single Element** | `[-5]` | `-5` | Code must not crash or compute out-of-bounds indices. |
| **Alternating Elements** | `[3, -2, 3, -2]` | `4` | Tests whether partial wrap-around is correctly optimized. |

---

## 13. Interview Explanation

"To solve the Maximum Circular Subarray Sum problem efficiently, we recognize that the maximum subarray can either be normal (lying fully within array boundaries) or circular (wrapping from the end to the start). 

If it's a normal subarray, we can find its maximum sum using standard **Kadane's Algorithm**. 

If it wraps around, the subarray is composed of a prefix and a suffix. The elements not included in this wrap-around subarray must form a contiguous subarray in the middle. To maximize the wrap-around sum, we must minimize this excluded middle subarray sum. Thus, the circular maximum sum is the `Total Sum of the array` minus the `Minimum Subarray Sum` (which we can find using a modified Kadane's algorithm).

Finally, we return the maximum of these two cases. The only edge case is when all elements are negative. In that case, the minimum subarray sum equals the total sum, which would yield a sum of 0 (an empty subarray). Since the subarray must be non-empty, we simply return the maximum element in the array."

---

## 14. Follow-up Questions

- **Q:** How would you modify the optimal algorithm if we also need to output the start and end indices of the maximum circular subarray?
  - **A:** We track the starting and ending indices during both the standard Kadane (for `max_so_far`) and the modified Kadane (for `min_so_far`). If the wrapped sum is larger, the start index of the circular subarray is `(end_idx_of_min_subarray + 1) % N` and the end index is `(start_idx_of_min_subarray - 1 + N) % N`.
- **Q:** Can we solve this if the array elements are extremely large and overflow standard 32-bit integers?
  - **A:** We would use 64-bit integers (`long long` in C++) for sums.

---

## 15. Variations

- **Maximum Subarray Sum (Linear):** Classic Kadane's.
- **K-Concatenation Maximum Sum:** Find the maximum subarray sum of an array concatenated $K$ times. If $K=1$, it is Kadane's. If $K \ge 2$, it is similar to the circular case where we check if the total sum is positive and add it $K-2$ times to the maximum circular sum.

---

## 16. Revision Notes

- **Core Formula:** `ans = max(max_normal, total_sum - min_normal)`
- **Key Exception:** If `total_sum == min_normal`, return `max_normal`.
- **Kadane Min Subarray:** `curr_min = min(arr[i], curr_min + arr[i])`.
- **Complexity:** $O(N)$ Time, $O(1)$ Space.

---

## 17. Memorization Version

- **Step 1:** Run standard Kadane to find `max_normal`.
- **Step 2:** Run modified Kadane to find `min_normal`.
- **Step 3:** Calculate `total_sum` of the array.
- **Step 4:** If `total_sum == min_normal` $\rightarrow$ Return `max_normal`.
- **Step 5:** Else $\rightarrow$ Return `max(max_normal, total_sum - min_normal)`.
- **Complexity:** $O(N)$ time, $O(1)$ space. Appropriate for quick implementation.
