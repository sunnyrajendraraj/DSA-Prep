# Equilibrium Index of Array

---

## 1. Problem Statement

An **Equilibrium Index** of an array is an index such that the sum of elements at lower indices is equal to the sum of elements at higher indices. More formally, an index $i$ (0-based) is an equilibrium index if:

$$\sum_{j=0}^{i-1} \text{arr}[j] = \sum_{k=i+1}^{N-1} \text{arr}[k]$$

If $i = 0$, the left sum is assumed to be `0`. 
If $i = N-1$, the right sum is assumed to be `0`.

### Input
- An array of integers `arr` of size $N$.

### Output
- The first equilibrium index (0-based index) if it exists, otherwise return `-1`.

### Constraints
- $1 \le N \le 10^6$
- $-10^5 \le \text{arr}[i] \le 10^5$

### Observations
- Elements can be positive, negative, or zero.
- The equilibrium index itself is not included in either the left sum or the right sum.
- Since elements can be negative, the cumulative sum does not strictly increase or decrease, meaning multiple equilibrium indices are possible.

---

## 2. Interview Clarification

Here are questions a candidate should ask the interviewer:
1. **What should we return if no such index exists?**
   - *Answer:* Return `-1`.
2. **If there are multiple equilibrium indices, which one should we return?**
   - *Answer:* Return the lowest (first) equilibrium index.
3. **Can the array contain negative numbers?**
   - *Answer:* Yes, elements can be negative, positive, or zero.
4. **Could the sum of elements overflow the standard 32-bit signed integer?**
   - *Answer:* Since $N \le 10^6$ and $\text{arr}[i] \le 10^5$, the maximum absolute sum can be $10^6 \times 10^5 = 10^{11}$, which exceeds the range of a standard 32-bit signed integer ($\approx \pm 2 \times 10^9$). We must use 64-bit integers (`long long` in C++) for sums.

---

## 3. Thinking Process

1. **First Instinct (Brute Force):**
   - For every possible index $i$ from $0$ to $N-1$, calculate the sum of elements from index $0$ to $i-1$ (left sum), and then calculate the sum of elements from $i+1$ to $N-1$ (right sum). If they match, we return $i$.
   - This requires running two loops inside a loop, resulting in a time complexity of $O(N^2)$.
2. **Optimizing Sum Calculations (Better):**
   - The primary bottleneck in the brute force approach is recalculating the left and right sums from scratch for every index.
   - We can precompute the sums. An auxiliary array containing prefix sums can tell us the sum of any range in $O(1)$ time. 
   - Let `prefixSum[i]` store the sum of elements from index `0` to `i`.
   - Left sum at $i$ is `prefixSum[i-1]` (or 0 if $i=0$).
   - Right sum at $i$ is `prefixSum[N-1] - prefixSum[i]`.
   - This reduces time complexity to $O(N)$ but requires $O(N)$ extra space.
3. **Space Optimization (Optimal):**
   - Can we avoid the extra $O(N)$ space?
   - Notice that if we know the total sum of the array, then at any index $i$:
     $$\text{rightSum} = \text{totalSum} - \text{leftSum} - \text{arr}[i]$$
   - We can initialize `leftSum` to `0` and first compute the `totalSum` of the array in a single pass.
   - In a second pass, as we move from left to right, we can dynamically check if `leftSum == rightSum`. If they are equal, we found our equilibrium index. Otherwise, we add `arr[i]` to `leftSum` and proceed.
   - This achieves $O(N)$ time complexity with $O(1)$ auxiliary space.

---

## 4. Pattern Recognition

- **Pattern:** Running Prefix Sum / Accumulation.
- **Why this topic?** The problem requires comparing the left partition and right partition sums of an array. Since partitions shift by exactly one element at each step, we can maintain the running sums incrementally in $O(1)$ rather than recomputing them.

---

## 5. Brute Force Solution

### Algorithm
1. Loop through each index $i$ from $0$ to $N-1$.
2. Initialize `leftSum = 0` and `rightSum = 0`.
3. Compute `leftSum` by summing all elements from index $0$ to $i-1$.
4. Compute `rightSum` by summing all elements from index $i+1$ to $N-1$.
5. If `leftSum == rightSum`, return $i$.
6. If the loop completes without finding any such index, return `-1`.

### Dry Run
Input: `arr = [1, 7, 3, 6, 5, 6]`
- $i=0$: `leftSum` = 0, `rightSum` = 7+3+6+5+6 = 27. (Not equal)
- $i=1$: `leftSum` = 1, `rightSum` = 3+6+5+6 = 20. (Not equal)
- $i=2$: `leftSum` = 1+7 = 8, `rightSum` = 6+5+6 = 17. (Not equal)
- $i=3$: `leftSum` = 1+7+3 = 11, `rightSum` = 5+6 = 11. (Equal! Return index 3)

### Complexity
- **Time Complexity:** $O(N^2)$ because for each element, we traverse the rest of the array to calculate the sums.
- **Space Complexity:** $O(1)$ auxiliary space.

### Pros and Cons
- **Pros:** Extremely easy to understand and implement.
- **Cons:** Too slow for large inputs ($N \ge 10^5$ will time out).

### C++17 Code
```cpp
#include <vector>
#include <numeric>

class Solution {
public:
    int findEquilibriumIndexBrute(const std::vector<int>& arr) {
        int n = arr.size();
        for (int i = 0; i < n; ++i) {
            long long leftSum = 0;
            long long rightSum = 0;
            
            // Calculate left sum
            for (int j = 0; j < i; ++j) {
                leftSum += arr[j];
            }
            
            // Calculate right sum
            for (int k = i + 1; k < n; ++k) {
                rightSum += arr[k];
            }
            
            if (leftSum == rightSum) {
                return i; // First equilibrium index found
            }
        }
        return -1; // No equilibrium index found
    }
};
```

---

## 6. Better Solution

### Improvement over Brute
Instead of recalculating sums repeatedly, we precompute prefix sums in an auxiliary array.

### Algorithm
1. Create a `prefixSum` array of size $N$ where `prefixSum[i]` stores the sum of elements from $0$ to $i$.
2. For any index $i$:
   - `leftSum` = (i == 0) ? 0 : `prefixSum[i-1]`
   - `rightSum` = `prefixSum[N-1] - prefixSum[i]`
3. Iterate $i$ from $0$ to $N-1$, compare `leftSum` and `rightSum`, and return the first matching index.

### Dry Run
Input: `arr = [1, 7, 3, 6, 5, 6]`
- `prefixSum` = `[1, 8, 11, 17, 22, 28]`
- $i=0$: `leftSum` = 0, `rightSum` = 28 - 1 = 27. (Not equal)
- $i=1$: `leftSum` = 1, `rightSum` = 28 - 8 = 20. (Not equal)
- $i=2$: `leftSum` = 8, `rightSum` = 28 - 11 = 17. (Not equal)
- $i=3$: `leftSum` = 11, `rightSum` = 28 - 17 = 11. (Equal! Return index 3)

### Complexity
- **Time Complexity:** $O(N)$ - One pass to compute the prefix sums and one pass to find the index.
- **Space Complexity:** $O(N)$ to store the `prefixSum` array.

### C++17 Code
```cpp
#include <vector>

class Solution {
public:
    int findEquilibriumIndexBetter(const std::vector<int>& arr) {
        int n = arr.size();
        if (n == 0) return -1;
        
        std::vector<long long> prefixSum(n, 0);
        prefixSum[0] = arr[0];
        for (int i = 1; i < n; ++i) {
            prefixSum[i] = prefixSum[i - 1] + arr[i];
        }
        
        for (int i = 0; i < n; ++i) {
            long long leftSum = (i == 0) ? 0 : prefixSum[i - 1];
            long long rightSum = prefixSum[n - 1] - prefixSum[i];
            
            if (leftSum == rightSum) {
                return i;
            }
        }
        return -1;
    }
};
```

---

## 7. Optimal Solution

### Best Approach
We can optimize the space of the "Better Solution" by using a single variable for `totalSum` and updating a running `leftSum` on the fly. 

### Algorithm
1. Compute the `totalSum` of all elements in the array.
2. Initialize `leftSum = 0`.
3. Loop through the array from $i = 0$ to $N-1$:
   - Calculate `rightSum = totalSum - leftSum - arr[i]`.
   - If `leftSum == rightSum`, then $i$ is the equilibrium index. Return $i$.
   - Add `arr[i]` to `leftSum`.
4. If the loop ends, return `-1`.

### Full Dry Run
Input: `arr = [1, 7, 3, 6, 5, 6]`
- `totalSum` = $1+7+3+6+5+6 = 28$.
- `leftSum` = $0$.
- Step-by-step iteration details are shown in the **Dry Run** section below.

### Why Optimal?
- We only visit each element twice (once for total sum, once to find the index). Time is $O(N)$.
- We only store a few scalar variables (`totalSum`, `leftSum`, `rightSum`). Space is $O(1)$.
- We cannot do better than $O(N)$ time because we must examine every element at least once to compute the sum.

### Beautiful C++17 Code
```cpp
#include <vector>
#include <numeric>

class Solution {
public:
    int pivotIndex(const std::vector<int>& nums) {
        // totalSum stores the sum of all elements in the array
        long long totalSum = 0;
        for (int num : nums) {
            totalSum += num;
        }
        
        // leftSum keeps track of the running sum of elements to the left of index i
        long long leftSum = 0;
        
        for (size_t i = 0; i < nums.size(); ++i) {
            // rightSum is the total sum minus the left sum and the current element
            long long rightSum = totalSum - leftSum - nums[i];
            
            if (leftSum == rightSum) {
                return i; // First equilibrium index
            }
            
            // Add current element to leftSum for the next iteration
            leftSum += nums[i];
        }
        
        return -1; // No equilibrium index exists
    }
};
```

---

## 8. Code Walkthrough

1. **Calculate Total Sum:**
   ```cpp
   long long totalSum = 0;
   for (int num : nums) {
       totalSum += num;
   }
   ```
   We traverse the array once to sum all elements. We use `long long` to prevent potential integer overflow when $N$ is large.

2. **Track Running Left Sum:**
   ```cpp
   long long leftSum = 0;
   ```
   We initialize `leftSum` to `0` because there are no elements to the left of index `0`.

3. **Check Equilibrium Condition:**
   ```cpp
   long long rightSum = totalSum - leftSum - nums[i];
   if (leftSum == rightSum) {
       return i;
   }
   ```
   At each index `i`, the sum of elements to the right is the total sum minus the current element and the sum of elements to its left. If `leftSum` equals `rightSum`, we immediately return `i`.

4. **Update Running Sum:**
   ```cpp
   leftSum += nums[i];
   ```
   If the condition is not met, the current element becomes part of the left sum for the next index.

---

## 9. Dry Run

Input: `nums = [1, 7, 3, 6, 5, 6]`, `totalSum = 28`

| Iteration | Index `i` | `nums[i]` | `leftSum` (Before Check) | `rightSum = totalSum - leftSum - nums[i]` | `leftSum == rightSum`? | Action / Update `leftSum` |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| 1 | 0 | 1 | 0 | $28 - 0 - 1 = 27$ | $0 == 27$ (False) | `leftSum` becomes $0 + 1 = 1$ |
| 2 | 1 | 7 | 1 | $28 - 1 - 7 = 20$ | $1 == 20$ (False) | `leftSum` becomes $1 + 7 = 8$ |
| 3 | 2 | 3 | 8 | $28 - 8 - 3 = 17$ | $8 == 17$ (False) | `leftSum` becomes $8 + 3 = 11$ |
| 4 | 3 | 6 | 11 | $28 - 11 - 6 = 11$ | $11 == 11$ (True) | **Return 3** |

---

## 10. Complexity Analysis

### Comparison Table

| Approach | Time Complexity | Space Complexity | Overflow Safety |
|:---|:---:|:---:|:---:|
| **Brute Force** | $O(N^2)$ | $O(1)$ | No |
| **Better (Prefix Array)** | $O(N)$ | $O(N)$ | Yes (using `long long`) |
| **Optimal (Two Variables)** | $O(N)$ | $O(1)$ | Yes (using `long long`) |

### Optimal Space-Time Complexity
- **Time Complexity:**
  - **Best Case:** $O(N)$ (e.g., equilibrium is at index 0, but we still need one pass to calculate the total sum).
  - **Average Case:** $O(N)$
  - **Worst Case:** $O(N)$
- **Space Complexity:**
  - $O(1)$ auxiliary space as we only use variables to store the sums.

---

## 11. Common Mistakes

1. **Off-by-One at Boundaries:**
   - Forgetting that the left sum at index `0` is `0`, or that the right sum at index `N-1` is `0`.
2. **Updating `leftSum` prematurely:**
   - Adding `nums[i]` to `leftSum` *before* calculating `rightSum` or performing the equality check. The current element `nums[i]` should not be part of either `leftSum` or `rightSum`.
3. **Integer Overflow:**
   - Using standard 32-bit `int` for summing large arrays. Always use `long long` when the constraints allow the sum to exceed $2 \times 10^9$.

---

## 12. Edge Cases

| Edge Case | Input Example | Expected Output | Why it Matters |
|:---|:---|:---:|:---|
| **Single Element** | `[5]` | `0` | Left sum = 0, Right sum = 0. Handled correctly. |
| **Equilibrium at First Index** | `[2, -1, 1]` | `0` | Left sum at index 0 is 0. Right sum is $-1 + 1 = 0$. |
| **Equilibrium at Last Index** | `[-1, 1, 2]` | `2` | Left sum at index 2 is $-1 + 1 = 0$. Right sum is 0. |
| **All Negative Elements** | `[-1, -2, -3, -2, -1]` | `2` | Negative sum values must match correctly. |
| **No Equilibrium Index** | `[1, 2, 3]` | `-1` | Loop must terminate and return `-1`. |
| **Multiple Equilibrium Indices** | `[1, 2, 0, 3]` | `2` | Returns the first one (index 2: left = 3, right = 3). |

---

## 13. Interview Explanation

**Speaker Script:**
> "To solve the Equilibrium Index problem efficiently, my primary goal is to find an index where the sum of elements to its left equals the sum of elements to its right. 
> A naive approach would calculate these sums from scratch for every index, which takes $O(N^2)$ time.
> We can optimize this to $O(N)$ time and $O(1)$ space by using prefix sum math. We first compute the total sum of the entire array. Then, we traverse the array from left to right while keeping track of a running `leftSum`. 
> At any current index `i`, we can compute the sum of elements to its right in $O(1)$ time using the formula: `rightSum = totalSum - leftSum - arr[i]`. 
> If `leftSum` equals `rightSum`, we return `i` as the equilibrium index. If not, we add the current element to `leftSum` and check the next index. If the loop completes without a match, we return `-1`."

---

## 14. Follow-up Questions

1. **What if the array is modified dynamically? (Updates and queries)**
   - *Answer:* We can use a Fenwick Tree (Binary Indexed Tree) or Segment Tree. This allows updates in $O(\log N)$ and finding the equilibrium index in $O(\log N)$ or $O(N)$ depending on search strategy.
2. **What if the elements are too large to fit in memory?**
   - *Answer:* If processing a stream of elements, we can do a two-pass stream or file read. First pass computes the total sum. Second pass scans elements again to find the equilibrium index.

---

## 15. Variations

1. **Find Pivot Index (LeetCode 724):** Exactly the same problem.
2. **Find Pivot Integer (LeetCode 2485):** Find an integer $x$ such that the sum of all elements from $1$ to $x$ is equal to the sum of all elements from $x$ to $n$.
3. **Partition Array Into Three Parts With Equal Sum (LeetCode 1013):** Check if we can partition the array into three parts with equal sums.

---

## 16. Revision Notes

- **Equilibrium Formula:** `rightSum = totalSum - leftSum - arr[i]`
- **First-pass:** Calculate total sum.
- **Second-pass:** Compare `leftSum` with `rightSum`, then update `leftSum`.
- **Constraint Tip:** Check if total sum overflows 32-bit integer limits.
- **Complexity:** $O(N)$ Time, $O(1)$ Space.

---

## 17. Memorization Version

```cpp
// Time: O(N), Space: O(1)
int pivotIndex(vector<int>& nums) {
    long long total = accumulate(nums.begin(), nums.end(), 0LL);
    long long left = 0;
    for (int i = 0; i < nums.size(); ++i) {
        if (left == total - left - nums[i]) return i;
        left += nums[i];
    }
    return -1;
}
```
