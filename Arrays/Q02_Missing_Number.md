# Find Missing Number (1 to N)

---

## 1. Problem Statement

Given an array containing $N-1$ distinct integers in the range $[1, N]$, find the single missing number.

*   **Input:** An integer array `nums` of size $N-1$, containing distinct integers in the range $[1, N]$.
*   **Output:** The single integer from the range $[1, N]$ that does not appear in the array.
*   **Constraints:**
    *   $N \ge 1$
    *   `nums` contains unique integers.
    *   $1 \le nums[i] \le N$
*   **Observations:**
    *   Since the array is of size $N-1$ and values are from $1$ to $N$, exactly one number is missing.
    *   The array is not necessarily sorted.
    *   Since all elements are distinct and positive, we can leverage mathematical properties or bitwise operations.

---

## 2. Interview Clarification

Before writing code, a candidate should clarify these assumptions with the interviewer:
1.  **Can the input array be empty?**
    *   *Answer:* If $N=1$, the array size is $0$. The missing number is $1$.
2.  **Can there be multiple missing numbers?**
    *   *Answer:* No, exactly one number is missing.
3.  **Is the array sorted?**
    *   *Answer:* No, it can be in any order.
4.  **Can the array contain duplicates?**
    *   *Answer:* No, all elements are unique.
5.  **Is integer overflow a concern?**
    *   *Answer:* Yes, if $N$ is very large (e.g., $N = 10^5$ or $10^6$), the sum of numbers from $1$ to $N$ can exceed the limit of a standard 32-bit signed integer. We should use 64-bit integers (`long long` in C++) to prevent overflow.

---

## 3. Thinking Process

*   **First Instinct:** Sort the array. Once sorted, we can scan from left to right. If `nums[i] != i + 1`, then `i + 1` is our missing number. If we reach the end without finding it, the missing number is $N$.
*   **Observation & Optimization:** Sorting takes $O(N \log N)$ time. Can we do better? Since we know the range is exactly $[1, N]$, we can use a hash set or a boolean frequency array to mark visited elements. This takes $O(N)$ time but uses $O(N)$ extra space.
*   **Mathematical Insight (Sum Formula):** The sum of first $N$ natural numbers is given by the formula:
    $$\text{Expected Sum} = \frac{N \times (N + 1)}{2}$$
    If we sum up all elements in the given array, the difference between the expected sum and the actual sum will be the missing number.
    $$\text{Missing Number} = \text{Expected Sum} - \text{Actual Sum}$$
    This reduces the time complexity to $O(N)$ and space complexity to $O(1)$. However, if $N$ is very large, $\text{Expected Sum}$ can overflow.
*   **Bitwise Insight (XOR):** We can avoid the overflow issue using the XOR bitwise operator.
    *   Property of XOR:
        *   $A \oplus A = 0$
        *   $A \oplus 0 = A$
    *   If we XOR all numbers from $1$ to $N$, and then XOR all elements in the array, the elements that appear in both will pair up and cancel each other out ($A \oplus A = 0$), leaving only the missing number.
    *   This has $O(N)$ time and $O(1)$ space complexity, and does not suffer from integer overflow.

---

## 4. Pattern Recognition

*   **Topic:** Arrays, Bit Manipulation, Math.
*   **Pattern:** **Math / Accumulation** or **Bitwise XOR**.
*   *Why this pattern?* Since the elements are bounded in a strict range $[1, N]$, we can represent the aggregate properties of the full set (either via sum or XOR) and find the outlier by calculating the difference or the XOR residue.

---

## 5. Brute Force Solution (Sorting)

### Algorithm
1.  Sort the input array in ascending order.
2.  Iterate through the array from index `0` to `N-2`.
3.  If `nums[i]` is not equal to `i + 1`, return `i + 1`.
4.  If the loop completes without returning, it means all elements from $1$ to $N-1$ are present. Return $N$.

### Dry Run
Input: `nums = [3, 1, 4]`, $N=4$
1.  Sort: `nums = [1, 3, 4]`
2.  `i = 0`: `nums[0] = 1`, matches `i + 1 = 1`.
3.  `i = 1`: `nums[1] = 3`, does not match `i + 1 = 2`.
4.  Return `2`.

### Complexity
*   **Time Complexity:** $O(N \log N)$ due to sorting.
*   **Space Complexity:** $O(1)$ (or $O(N)$ depending on the sorting algorithm implementation, but generally in-place).

### Pros/Cons
*   **Pros:** Very simple to understand.
*   **Cons:** Unnecessarily slow due to sorting. Modifies the input array.

### Code (C++17)
```cpp
#include <vector>
#include <algorithm>

class SolutionBrute {
public:
    int findMissingNumber(std::vector<int>& nums) {
        int n = nums.size() + 1; // Array size is N-1, so N is nums.size() + 1
        std::sort(nums.begin(), nums.end());
        
        for (int i = 0; i < n - 1; ++i) {
            if (nums[i] != i + 1) {
                return i + 1;
            }
        }
        return n;
    }
};
```

---

## 6. Better Solution (Hashing)

### Algorithm
1.  Initialize a boolean tracking array or hash set of size $N+1$ with `false`.
2.  Iterate through the input array and mark each visited number as `true` in the tracking array.
3.  Iterate from $1$ to $N$. The first number whose tracking value remains `false` is the missing number.

### Dry Run
Input: `nums = [3, 1, 4]`, $N = 4$
1.  Create track array: `visited = [false, false, false, false, false]` (indices 0 to 4).
2.  Mark elements:
    *   `nums[0] = 3` $\to$ `visited[3] = true`
    *   `nums[1] = 1` $\to$ `visited[1] = true`
    *   `nums[2] = 4` $\to$ `visited[4] = true`
3.  Scan `visited` from index 1 to 4:
    *   `visited[1] = true`
    *   `visited[2] = false` $\to$ Return `2`.

### Complexity
*   **Time Complexity:** $O(N)$ — One pass to populate the hash set/visited array, and one pass to find the missing element.
*   **Space Complexity:** $O(N)$ — Extra storage of size $N+1$ for tracking.

### Pros/Cons
*   **Pros:** Faster than sorting, $O(N)$ time. Does not modify input.
*   **Cons:** Uses $O(N)$ auxiliary memory.

### Code (C++17)
```cpp
#include <vector>

class SolutionBetter {
public:
    int findMissingNumber(const std::vector<int>& nums) {
        int n = nums.size() + 1;
        std::vector<bool> visited(n + 1, false);
        
        for (int num : nums) {
            visited[num] = true;
        }
        
        for (int i = 1; i <= n; ++i) {
            if (!visited[i]) {
                return i;
            }
        }
        return -1;
    }
};
```

---

## 7. Optimal Solutions (Math & XOR)

We present two optimal approaches: the **Sum Formula** and the **XOR Method**.

### Approach A: Sum Formula
*   Sum of first $N$ natural numbers: $S_N = \frac{N(N+1)}{2}$.
*   Sum of array elements: $S_{arr} = \sum nums[i]$.
*   Missing number = $S_N - S_{arr}$.

### Approach B: XOR Method
*   XOR all numbers from $1$ to $N$: $X_1 = 1 \oplus 2 \oplus \dots \oplus N$.
*   XOR all elements in the array: $X_2 = nums[0] \oplus nums[1] \dots \oplus nums[N-2]$.
*   Missing number = $X_1 \oplus X_2$.

### Dry Run (XOR Method)
Input: `nums = [3, 1, 4]`, $N = 4$
1.  Initialize `xorSum = 0`.
2.  XOR all numbers from $1$ to $N$:
    `xorSum = 0 ^ 1 ^ 2 ^ 3 ^ 4`
3.  XOR all numbers in the array:
    `xorSum = (0 ^ 1 ^ 2 ^ 3 ^ 4) ^ (3 ^ 1 ^ 4)`
4.  Rearrange:
    `xorSum = (1 ^ 1) ^ (3 ^ 3) ^ (4 ^ 4) ^ 2`
    `xorSum = 0 ^ 0 ^ 0 ^ 2 = 2`
5.  Return `2`.

### Why Optimal
Both methods achieve $O(N)$ time with $O(1)$ auxiliary space. The XOR method is preferred in production because it prevents any risk of integer overflow.

---

### Beautiful C++17 Code

```cpp
#include <vector>
#include <numeric>

class SolutionOptimal {
public:
    // Optimal 1: XOR Approach (Safe from overflow)
    int findMissingNumberXOR(const std::vector<int>& nums) {
        int n = nums.size() + 1;
        int xorSum = 0;
        
        // XOR all numbers from 1 to N
        for (int i = 1; i <= n; ++i) {
            xorSum ^= i;
        }
        
        // XOR all numbers in the array
        for (int num : nums) {
            xorSum ^= num;
        }
        
        return xorSum;
    }

    // Optimal 2: Sum Approach (Uses long long to prevent overflow)
    int findMissingNumberSum(const std::vector<int>& nums) {
        long long n = nums.size() + 1;
        long long expectedSum = (n * (n + 1)) / 2;
        
        long long actualSum = 0;
        for (int num : nums) {
            actualSum += num;
        }
        
        return static_cast<int>(expectedSum - actualSum);
    }
};
```

---

## 8. Code Walkthrough

### XOR Approach Walkthrough
1.  `int n = nums.size() + 1;`
    Determines $N$, which is 1 larger than the array size.
2.  `int xorSum = 0;`
    Initializes our XOR accumulator.
3.  `for (int i = 1; i <= n; ++i) { xorSum ^= i; }`
    Computes $1 \oplus 2 \oplus \dots \oplus N$.
4.  `for (int num : nums) { xorSum ^= num; }`
    XORs every element of the input array. Since duplicate elements cancel out (e.g., $3 \oplus 3 = 0$), the only surviving value is the missing number.
5.  `return xorSum;`
    Returns the accumulated result.

---

## 9. Dry Run (XOR Method)

Given `nums = [1, 2, 4]`, $N = 4$ (expected: `3`)

| Step | Operation / Code Line | Variable State |
| :--- | :--- | :--- |
| 1 | `int n = nums.size() + 1;` | `n = 4` |
| 2 | `int xorSum = 0;` | `xorSum = 0` |
| 3 | Loop 1: `i = 1` | `xorSum = 0 ^ 1 = 1` |
| 4 | Loop 1: `i = 2` | `xorSum = 1 ^ 2 = 3` |
| 5 | Loop 1: `i = 3` | `xorSum = 3 ^ 3 = 0` |
| 6 | Loop 1: `i = 4` | `xorSum = 0 ^ 4 = 4` |
| 7 | Loop 2: `num = nums[0]` (1) | `xorSum = 4 ^ 1 = 5` |
| 8 | Loop 2: `num = nums[1]` (2) | `xorSum = 5 ^ 2 = 7` |
| 9 | Loop 2: `num = nums[2]` (4) | `xorSum = 7 ^ 4 = 3` |
| 10 | Return `xorSum` | Returns `3` |

---

## 10. Complexity Analysis

| Approach | Time Complexity (Best/Avg/Worst) | Space Complexity | Reasoning |
| :--- | :--- | :--- | :--- |
| **Brute Force (Sorting)** | $O(N \log N)$ | $O(1)$ | Sorting takes $O(N \log N)$ time. Iteration takes $O(N)$. |
| **Better (Hashing)** | $O(N)$ | $O(N)$ | Extra array of size $N+1$ requires linear auxiliary space. |
| **Optimal (Sum)** | $O(N)$ | $O(1)$ | Single pass to sum up array elements. |
| **Optimal (XOR)** | $O(N)$ | $O(1)$ | Two simple loops of sizes $N$ and $N-1$ with constant space. |

---

## 11. Common Mistakes

1.  **Integer Overflow in Sum Formula:** Summing up to $10^5$ or $10^6$ in standard 32-bit signed integers leads to integer overflow.
    *   *Fix:* Use `long long` for calculations.
2.  **Incorrect Upper Bound for $N$:** Using `nums.size()` as $N$ instead of `nums.size() + 1`.
3.  **1-based vs 0-based Indexing Confusion:** Not matching elements to indices properly when using the sorting method.

---

## 12. Edge Cases

| Edge Case | Input Example | Expected Output | Why It Matters / How Code Handles It |
| :--- | :--- | :--- | :--- |
| **Smallest Array ($N=1$)** | `nums = []` | `1` | Size is 0. Expected sum formula gives 1. XOR gives 1. |
| **Missing First Element** | `nums = [2, 3, 4]` | `1` | Correctly computed because 1 was never XORed/summed. |
| **Missing Last Element** | `nums = [1, 2, 3]` | `4` | Correctly identified because $N$ is not present. |
| **Large N** | Array of size $99,999$ | Missing number | Verifies code safety against overflow (Sum method) and efficiency. |

---

## 13. Interview Explanation

> "To find the missing number from $1$ to $N$ in a given array of size $N-1$, we can implement multiple approaches.
> A simple brute-force method is to sort the array and look for any mismatch between the value and its expected position, which takes $O(N \log N)$ time.
> A faster approach is to use a hash set or tracking array to record seen values in $O(N)$ time, but this requires $O(N)$ extra space.
> To solve this in $O(N)$ time and $O(1)$ space, we can use either the mathematical sum formula or bitwise XOR.
> Using the sum formula, we calculate the expected sum of the first $N$ numbers as $\frac{N(N+1)}{2}$ and subtract the sum of the array elements. To prevent integer overflow for large $N$, we calculate using 64-bit integers.
> Alternatively, the XOR method is completely safe from overflow. We XOR all numbers from $1$ to $N$ and all numbers in the array. Since matching pairs cancel out to zero, the final XOR result is exactly the missing number."

---

## 14. Follow-up Questions

1.  **What if the array is already sorted?**
    *   *Answer:* If the array is sorted, we can find the missing number in $O(\log N)$ time using **Binary Search**. We can look for the first index $i$ where `nums[i] != i + 1`.
2.  **What if there are two missing numbers?**
    *   *Answer:* We can find them in $O(N)$ time and $O(1)$ space using a modified XOR method (partitioning the numbers based on the rightmost set bit of the combined XOR result) or by using a system of two linear equations.

---

## 15. Variations

*   **Single Number (LeetCode 136):** Given an array where every element appears twice except for one.
    *   *Difference:* No sequential range. Simply XOR all array elements together.
*   **Find all Numbers Disappeared in an Array (LeetCode 448):**
    *   *Difference:* Multiple numbers can be missing, array size is $N$. Can be solved in-place by marking visited elements as negative.

---

## 16. Revision Notes

*   **XOR Trick:** $x \oplus x = 0$ and $x \oplus 0 = x$.
*   **Sum Trick:** $\sum_{i=1}^{N} i = \frac{N(N+1)}{2}$. Always use `long long` for sums.
*   **Complexity Summary:** Optimal is $O(N)$ time and $O(1)$ space.

---

## 17. Memorization Version

```cpp
// 1. Math Sum Formula O(N) / O(1)
int missingSum(vector<int>& nums) {
    long long n = nums.size() + 1;
    long long total = n * (n + 1) / 2;
    for (int x : nums) total -= x;
    return total;
}

// 2. XOR Method O(N) / O(1)
int missingXOR(vector<int>& nums) {
    int res = 0, n = nums.size();
    for (int i = 1; i <= n + 1; ++i) res ^= i;
    for (int x : nums) res ^= x;
    return res;
}
```
*Remember: XOR is overflow-safe.*
