# Maximum Product Subarray

---

## 1. Problem Statement

Given an integer array `nums`, find a contiguous non-empty subarray that has the largest product, and return the product.

### Input
- An array of integers `nums` of size $N$.

### Output
- An integer representing the maximum product of a contiguous subarray.

### Constraints
- $1 \le N \le 2 \times 10^4$
- $-10 \le \text{nums}[i] \le 10$
- The product of any prefix or suffix of `nums` is guaranteed to fit in a 64-bit integer (and usually a 32-bit integer for standard test suites).

### Observations
- If the array contains only positive numbers, the maximum product is the product of the entire array.
- If there are negative numbers:
  - An even number of negative numbers results in a positive product.
  - An odd number of negative numbers requires us to exclude at least one negative number (and its adjacent elements on one side) to get the maximum positive product.
  - A negative number multiplied by a negative number becomes positive. This means a very small negative product (`min_so_far`) could suddenly become a very large positive product (`max_so_far`) when multiplied by another negative number.
- Zeros reset the product to 0. A subarray cannot "extend" through a zero without its product becoming 0. Hence, zeros effectively divide the array into smaller independent subproblems.

---

## 2. Interview Clarification

1. **Can the array contain zeros?**
   - *Answer:* Yes, and multiplying by zero results in a product of zero.
2. **Can the array contain negative numbers?**
   - *Answer:* Yes.
3. **What is the minimum size of the array?**
   - *Answer:* The array will have at least 1 element.
4. **Does a single element count as a subarray?**
   - *Answer:* Yes, a subarray is a contiguous part of the array, so a single element is a valid subarray.

---

## 3. Thinking Process

1. **First Instinct (Brute Force):**
   - We can generate all possible subarrays, calculate their products, and keep track of the maximum.
   - To do this, we use two nested loops: the outer loop defines the start of the subarray, and the inner loop extends the subarray and calculates the running product.
   - Time complexity: $O(N^2)$, space complexity: $O(1)$. Too slow for $N = 2 \times 10^4$.

2. **Developing the Dynamic Programming Intuition (Kadane's Variation):**
   - In Kadane's algorithm for maximum subarray sum, we track `max_ending_here`. For products, this isn't sufficient because of signs.
   - If we have `nums[i] = -5` and the maximum product ending at `i-1` is `10`, the new product is `-50`. However, if the *minimum* product ending at `i-1` was `-20`, then multiplying it by `-5` yields `+100`.
   - Therefore, at each index `i`, we must track both:
     - `max_so_far`: The maximum product ending at index `i`.
     - `min_so_far`: The minimum product ending at index `i`.
   - At index `i`, the new candidates for `max_so_far` and `min_so_far` are:
     1. The element itself: `nums[i]` (starts a new subarray).
     2. `nums[i] * max_so_far_prev` (extends the previous max product).
     3. `nums[i] * min_so_far_prev` (extends the previous min product, which could flip sign).
   - Thus:
     - $\text{max\_so\_far} = \max(\text{nums}[i], \max(\text{nums}[i] \times \text{max\_so\_far\_prev}, \text{nums}[i] \times \text{min\_so\_far\_prev}))$
     - $\text{min\_so\_far} = \min(\text{nums}[i], \min(\text{nums}[i] \times \text{max\_so\_far\_prev}, \text{nums}[i] \times \text{min\_so\_far\_prev}))$

3. **Alternative Optimal Approach (Prefix and Suffix Products):**
   - Another elegant approach relies on the observation that if we traverse the array from left to right (prefix product) and right to left (suffix product), the maximum product subarray must be either a prefix product or a suffix product of some subarray between zeros.
   - If we encounter a zero, we reset the running product to `1` (representing starting a new subarray).
   - We maintain a global maximum of all prefix and suffix products.
   - This approach is extremely simple to write but requires clear explanation of why it works. We will focus on the Kadane-based approach as the standard optimal solution since it generalizes well to other DP problems, but prefix-suffix is a great alternative.

---

## 4. Pattern Recognition

- **Pattern:** Dynamic Programming / Kadane's Algorithm modification.
- **Why this topic?** The optimal decision at step `i` depends on both the maximum and minimum subproblem solutions from step `i-1` due to the sign-flipping property of multiplication.

---

## 5. Brute Force Solution

### Algorithm
1. Initialize `maxProduct` to `nums[0]` (or negative infinity).
2. For each start index `i` from `0` to $N-1$:
   - Initialize `currentProduct = 1`.
   - For each end index `j` from `i` to $N-1$:
     - Multiply `currentProduct` by `nums[j]`.
     - Update `maxProduct = max(maxProduct, currentProduct)`.
3. Return `maxProduct`.

### Dry Run
Input: `nums = [2, 3, -2, 4]`
- `i = 0`:
  - `j = 0`: `currentProduct = 2`. `maxProduct = 2`.
  - `j = 1`: `currentProduct = 6`. `maxProduct = 6`.
  - `j = 2`: `currentProduct = -12`. `maxProduct = 6`.
  - `j = 3`: `currentProduct = -48`. `maxProduct = 6`.
- `i = 1`:
  - `j = 1`: `currentProduct = 3`. `maxProduct = 6`.
  - `j = 2`: `currentProduct = -6`. `maxProduct = 6`.
  ... and so on.
Return `6`.

### Complexity
- **Time Complexity:** $O(N^2)$ due to nested loops.
- **Space Complexity:** $O(1)$ auxiliary space.

### Pros and Cons
- **Pros:** Simple, handles zeros and negatives naturally.
- **Cons:** Inefficient. TLE for $N \ge 10^4$.

### C++17 Code
```cpp
#include <vector>
#include <algorithm>

class Solution {
public:
    int maxProductBrute(const std::vector<int>& nums) {
        int n = nums.size();
        int maxProduct = nums[0];
        
        for (int i = 0; i < n; ++i) {
            int currentProduct = 1;
            for (int j = i; j < n; ++j) {
                currentProduct *= nums[j];
                maxProduct = std::max(maxProduct, currentProduct);
            }
        }
        
        return maxProduct;
    }
};
```

---

## 6. Better Solution

### DP with State Arrays
Instead of tracking variables, we can represent this explicitly as a Dynamic Programming problem using two DP tables: `dpMax` and `dpMin`.

### Algorithm
1. Create `dpMax` and `dpMin` arrays of size $N$.
2. Initialize `dpMax[0] = nums[0]` and `dpMin[0] = nums[0]`.
3. Loop `i` from 1 to $N-1$:
   - `dpMax[i] = max({ (long long)nums[i], nums[i] * dpMax[i-1], nums[i] * dpMin[i-1] })`
   - `dpMin[i] = min({ (long long)nums[i], nums[i] * dpMax[i-1], nums[i] * dpMin[i-1] })`
4. The answer is the maximum element in `dpMax`.

### Complexity
- **Time Complexity:** $O(N)$ because we traverse the array once.
- **Space Complexity:** $O(N)$ to store the DP arrays.

### C++17 Code
```cpp
#include <vector>
#include <algorithm>

class Solution {
public:
    int maxProductBetter(const std::vector<int>& nums) {
        int n = nums.size();
        if (n == 0) return 0;
        
        std::vector<long long> dpMax(n);
        std::vector<long long> dpMin(n);
        
        dpMax[0] = nums[0];
        dpMin[0] = nums[0];
        long long ans = nums[0];
        
        for (int i = 1; i < n; ++i) {
            long long val = nums[i];
            dpMax[i] = std::max({val, val * dpMax[i - 1], val * dpMin[i - 1]});
            dpMin[i] = std::min({val, val * dpMax[i - 1], val * dpMin[i - 1]});
            ans = std::max(ans, dpMax[i]);
        }
        
        return static_cast<int>(ans);
    }
};
```

---

## 7. Optimal Solution

### Space-Optimized Dynamic Programming (Kadane's Variation)
Since `dpMax[i]` and `dpMin[i]` only depend on `dpMax[i-1]` and `dpMin[i-1]`, we can replace the arrays with two scalar variables: `max_so_far` and `min_so_far`.

### Algorithm
1. Initialize `max_so_far = nums[0]`, `min_so_far = nums[0]`, and `ans = nums[0]`.
2. Iterate `i` from `1` to $N-1$:
   - If `nums[i]` is negative, swap `max_so_far` and `min_so_far`. (Multiplying by a negative number flips the inequality, so the previous minimum product becomes the candidate for the new maximum, and vice versa).
   - Update `max_so_far = max(nums[i], max_so_far * nums[i])`.
   - Update `min_so_far = min(nums[i], min_so_far * nums[i])`.
   - Update `ans = max(ans, max_so_far)`.
3. Return `ans`.

### Why Optimal?
- It processes each element in a single pass: $O(N)$ time.
- It uses only three variables for tracking the state: $O(1)$ auxiliary space.
- We cannot achieve better than $O(N)$ time because any element could potentially be part of the optimal subarray.

### Beautiful C++17 Code
```cpp
#include <vector>
#include <algorithm>

class Solution {
public:
    int maxProduct(const std::vector<int>& nums) {
        if (nums.empty()) return 0;
        
        // Track the max and min products ending at the current position
        long long maxSoFar = nums[0];
        long long minSoFar = nums[0];
        long long globalMax = nums[0];
        
        for (size_t i = 1; i < nums.size(); ++i) {
            long long current = nums[i];
            
            // If the current element is negative, multiplying by it will
            // swap the roles of maximum and minimum products.
            if (current < 0) {
                std::swap(maxSoFar, minSoFar);
            }
            
            // The candidate for max product is either the current element itself
            // or the current element multiplied by the previous max
            maxSoFar = std::max(current, maxSoFar * current);
            
            // The candidate for min product is either the current element itself
            // or the current element multiplied by the previous min
            minSoFar = std::min(current, minSoFar * current);
            
            // Update the overall maximum product found so far
            globalMax = std::max(globalMax, maxSoFar);
        }
        
        return static_cast<int>(globalMax);
    }
};
```

---

## 8. Code Walkthrough

1. **State Initialization:**
   ```cpp
   long long maxSoFar = nums[0];
   long long minSoFar = nums[0];
   long long globalMax = nums[0];
   ```
   We start with the first element of the array as our initial candidate for maximum and minimum.

2. **Negative Number Handling:**
   ```cpp
   if (current < 0) {
       std::swap(maxSoFar, minSoFar);
   }
   ```
   If the current number is negative, `maxSoFar * current` will become small/negative, and `minSoFar * current` will become large/positive. Swapping them before calculations simplifies our update step.

3. **Updating States:**
   ```cpp
   maxSoFar = std::max(current, maxSoFar * current);
   minSoFar = std::min(current, minSoFar * current);
   ```
   For `maxSoFar`, we decide whether to start a new subarray at `current` or extend the previous maximum product. Same logic applies to `minSoFar`.

4. **Global Max Update:**
   ```cpp
   globalMax = std::max(globalMax, maxSoFar);
   ```
   We update our overall answer with the maximum product ending at the current index.

---

## 9. Dry Run

Input: `nums = [2, 3, -2, 4]`

- **Initial State:** `maxSoFar = 2`, `minSoFar = 2`, `globalMax = 2`.

| Iteration | Index `i` | `current` | Swap? | `maxSoFar` Update | `minSoFar` Update | `globalMax` |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| 1 | 1 | 3 | No | $\max(3, 2 \times 3) = 6$ | $\min(3, 2 \times 3) = 3$ | $\max(2, 6) = 6$ |
| 2 | 2 | -2 | Yes (`maxSoFar=3`, `minSoFar=6`) | $\max(-2, 3 \times -2) = -2$ | $\min(-2, 6 \times -2) = -12$ | $\max(6, -2) = 6$ |
| 3 | 3 | 4 | No | $\max(4, -2 \times 4) = 4$ | $\min(4, -12 \times 4) = -48$ | $\max(6, 4) = 6$ |

Output: `6` (from subarray `[2, 3]`).

---

## 10. Complexity Analysis

### Comparison Table

| Approach | Time Complexity | Space Complexity | Extra Space Usage |
|:---|:---:|:---:|:---|
| **Brute Force** | $O(N^2)$ | $O(1)$ | No extra arrays |
| **DP with Arrays** | $O(N)$ | $O(N)$ | Two DP arrays of size $N$ |
| **Optimal (Space-Optimized DP)** | $O(N)$ | $O(1)$ | Only 3 variables |

### Optimal Space-Time Complexity
- **Time Complexity:**
  - **Best Case:** $O(N)$
  - **Average Case:** $O(N)$
  - **Worst Case:** $O(N)$
- **Space Complexity:**
  - $O(1)$ auxiliary space as we only maintain state variables.

---

## 11. Common Mistakes

1. **Incorrect Swap Ordering:**
   - Updating `maxSoFar` and then using the updated `maxSoFar` to compute `minSoFar` without saving the original value. Swapping them when `current < 0` completely avoids this bug.
2. **Not Handling Zeros:**
   - If the array contains zeros, the product resets. The formula handles this because `std::max(current, ...)` will reset `maxSoFar` to `0` when `current` is `0`, and then to the next element in the next iteration.
3. **Overflowing standard integer types:**
   - Even if the final answer fits in a 32-bit integer, intermediate products might overflow. Using `long long` for internal DP variables is highly recommended.

---

## 12. Edge Cases

| Edge Case | Input Example | Expected Output | Why it Matters |
|:---|:---|:---:|:---|
| **Single Element** | `[-3]` | `-3` | Standard logic should return the element itself. |
| **All Negative Elements** | `[-2, -3, -4]` | `6` | Subarray `[-2, -3]` gives product 6. |
| **Contains Zeros** | `[-2, 0, -1]` | `0` | Standard logic must handle zero resets correctly. |
| **Alternating Signs** | `[2, -3, 2, -3]` | `36` | Product of the entire array is $2 \times -3 \times 2 \times -3 = 36$. |
| **Single Zero** | `[0]` | `0` | Returns 0. |

---

## 13. Interview Explanation

**Speaker Script:**
> "To find the maximum product subarray in $O(N)$ time and $O(1)$ space, we modify Kadane’s algorithm. Unlike the sum variant, a minimum product (which could be a large negative number) can suddenly become a maximum product when multiplied by a negative number.
> Thus, at each index, we keep track of both the maximum and minimum product ending at that position.
> When we encounter a negative number, the maximum and minimum values swap potential roles, so we swap our tracking variables. We then update our running maximum and minimum by comparing the current element itself with the product of the current element and the previous tracking variable.
> By updating a global maximum at each step, we can find the maximum product subarray in a single pass."

---

## 14. Follow-up Questions

1. **How would you return the actual subarray that yields the maximum product?**
   - *Answer:* We can track the starting and ending indices of the current maximum product window. When `globalMax` is updated, we update the best start and end indices. If `maxSoFar` resets to `current`, we reset the temp start index to the current index.
2. **Can we solve this using prefix and suffix products?**
   - *Answer:* Yes. We calculate prefix products and suffix products. If a zero is encountered, we reset the running product to 1. The maximum value encountered in either pass is the answer. This is also $O(N)$ time and $O(1)$ space.

---

## 15. Variations

1. **Maximum Subarray Sum (LeetCode 53 - Kadane's Algorithm)**
2. **Subarray Product Less Than K (LeetCode 713)**
3. **House Robber (LeetCode 198 - similar DP state tracking)**

---

## 16. Revision Notes

- **Key concept:** Negatives flip maximums and minimums.
- **Trick:** Swap `maxSoFar` and `minSoFar` if `current < 0`.
- **Reset condition:** `maxSoFar = max(current, maxSoFar * current)`.
- **Complexity:** $O(N)$ Time, $O(1)$ Space.

---

## 17. Memorization Version

```cpp
// Time: O(N), Space: O(1)
int maxProduct(vector<int>& nums) {
    int maxVal = nums[0], minVal = nums[0], res = nums[0];
    for (size_t i = 1; i < nums.size(); ++i) {
        if (nums[i] < 0) swap(maxVal, minVal);
        maxVal = max(nums[i], maxVal * nums[i]);
        minVal = min(nums[i], minVal * nums[i]);
        res = max(res, maxVal);
    }
    return res;
}
```
