# Binary Search – Standard

---

## 1. Problem Statement

Given a sorted array of integers `nums` and an integer `target`, write a function to search for `target` in `nums`. If `target` exists, then return its index. Otherwise, return `-1`.

You must write both **iterative** and **recursive** implementations.

### Input
- `nums`: A sorted array of integers (non-decreasing order).
- `target`: An integer to search for.

### Output
- The 0-based index of `target` in the array if it exists, otherwise `-1`.

### Constraints
- $1 \le \text{nums.length} \le 10^4$
- $-10^4 < \text{nums}[i], \text{target} < 10^4$
- All integers in `nums` are unique and sorted in ascending order.

### Key Observations
1. The input array is **already sorted**. This is the single most critical precondition for Binary Search.
2. Elements are unique, meaning there is at most one correct index.
3. Linear search takes $O(n)$ time. The sorted property allows us to discard half of the search space at each step, yielding $O(\log n)$ complexity.

---

## 2. Interview Clarification

Before writing any code, a candidate should clarify the following points with the interviewer:
1. **Is the array sorted in ascending or descending order?** (Assuming ascending as per standard, but good to clarify).
2. **Can there be duplicate elements?** If there are duplicates, do we return any index, the first occurrence, or the last occurrence? (Here, we assume unique elements, so any match is sufficient).
3. **What is the expected size of the input?** If the size fits standard memory constraints, standard pointers/indices are fine.
4. **Should the solution handle potential overflow of indices?** (Yes, standard binary search can suffer from overflow if the sum of low and high exceeds the maximum integer limit).
5. **Are we returning `-1` if the target is not present, or should we return the insertion position?** (Return `-1`).

---

## 3. Thinking Process

### First Instinct
If I need to find an element in an array, I can look at every element from left to right. This is **Linear Search**. It is simple to implement but does not exploit the sorted nature of the array. The worst-case complexity is $O(n)$ where we might search the entire array.

### Observations & Patterns
Since the array is sorted, comparing the target with the middle element gives us deep structural information:
- If the middle element is exactly the target, we are done!
- If the middle element is **greater** than the target, then because the array is sorted, every element to the right of the middle element must also be greater than the target. Thus, the target can only lie in the left half. We can safely ignore the right half.
- If the middle element is **less** than the target, every element to the left of the middle element must be less than the target. The target can only lie in the right half. We ignore the left half.

By repeatedly checking the middle element and halving the search space, we perform a binary search.

### Optimal Insight: Avoiding Mid Overflow
Traditionally, the middle index is computed as:
$$\text{mid} = \frac{\text{low} + \text{high}}{2}$$

However, in languages like C++ where integers have fixed bit widths (like 32-bit signed integers, which max out at $2,147,483,647$), if both `low` and `high` are large (e.g., greater than $10^9$), their sum `low + high` can exceed $2^{31}-1$, resulting in integer overflow (leading to negative indices and undefined behavior).

To prevent this, we calculate `mid` using subtraction first:
$$\text{mid} = \text{low} + \frac{\text{high} - \text{low}}{2}$$

Since `high >= low`, `high - low` is always non-negative and smaller than `high`. Adding this positive offset to `low` guarantees that the value never exceeds `high` and avoids any potential overflow.

---

## 4. Pattern Recognition

This problem belongs to the **Binary Search** pattern.
- **Why this topic?** The input space is ordered (sorted array), and we want to find a specific target.
- **When to apply?** Whenever you can divide the search space into two halves based on a condition (binary decision) and discard one half completely.

---

## 5. Brute Force Solution

### Algorithm
Perform a linear scan from index `0` to `n-1`. Compare each element with the target.

### Dry Run
Input: `nums = [-1, 0, 3, 5, 9, 12]`, `target = 9`
- Index `0`: `nums[0] = -1` (Not target)
- Index `1`: `nums[1] = 0` (Not target)
- Index `2`: `nums[2] = 3` (Not target)
- Index `3`: `nums[3] = 5` (Not target)
- Index `4`: `nums[4] = 9` (Target found! Return index `4`)

### Complexity Analysis
- **Time Complexity:** $O(n)$ in the worst case (target is at the end or not present).
- **Space Complexity:** $O(1)$ auxiliary space.

### Pros/Cons
- **Pros:** Extremely simple; works on unsorted arrays.
- **Cons:** Inefficient for large sorted arrays.

### Commented C++17 Code
```cpp
#include <vector>

class BruteForceSearch {
public:
    int search(const std::vector<int>& nums, int target) {
        int n = nums.size();
        for (int i = 0; i < n; ++i) {
            // Compare each element with target
            if (nums[i] == target) {
                return i; // Return index immediately if found
            }
        }
        return -1; // Target not found
    }
};
```

---

## 6. Better Solution

For this problem, there is no intermediate "better" solution between the $O(n)$ linear scan and the $O(\log n)$ optimal binary search. Therefore, the brute force solution jumps directly to the optimal solution.

---

## 7. Optimal Solution

The optimal solution is **Binary Search** implemented both **iteratively** and **recursively**.

### Algorithm (Iterative)
1. Initialize two pointers: `low = 0` and `high = nums.size() - 1`.
2. While `low <= high`:
   - Compute `mid = low + (high - low) / 2`.
   - If `nums[mid] == target`, return `mid`.
   - If `nums[mid] < target`, discard the left half by setting `low = mid + 1`.
   - If `nums[mid] > target`, discard the right half by setting `high = mid - 1`.
3. If the loop terminates without finding the target, return `-1`.

### Algorithm (Recursive)
1. Define a helper function `binarySearchHelper(nums, low, high, target)`.
2. Base case: If `low > high`, target is not present. Return `-1`.
3. Calculate `mid = low + (high - low) / 2`.
4. If `nums[mid] == target`, return `mid`.
5. If `nums[mid] < target`, recursively search the right half: `binarySearchHelper(nums, mid + 1, high, target)`.
6. If `nums[mid] > target`, recursively search the left half: `binarySearchHelper(nums, low, mid - 1, target)`.

### Full Dry Run
Input: `nums = [-1, 0, 3, 5, 9, 12]`, `target = 9`

#### Iterative Dry Run:
- **Initialization:** `low = 0`, `high = 5`
- **Iteration 1:**
  - `low = 0`, `high = 5`
  - `mid = 0 + (5 - 0) / 2 = 2`
  - `nums[mid] = nums[2] = 3`
  - Since `3 < 9`, target must be in the right half. Update `low = mid + 1 = 3`.
- **Iteration 2:**
  - `low = 3`, `high = 5`
  - `mid = 3 + (5 - 3) / 2 = 4`
  - `nums[mid] = nums[4] = 9`
  - Target found! Return `4`.

### Why Optimal?
Each comparison halves the remaining search space. This log-scale division means the maximum number of iterations is $\approx \log_2(n)$. This is mathematically optimal for comparison-based searching in a sorted array.

### Beautiful C++17 Code

```cpp
#include <vector>

class BinarySearch {
public:
    // 1. Iterative Implementation (Standard & Preferred for Space Complexity)
    int searchIterative(const std::vector<int>& nums, int target) {
        int low = 0;
        int high = static_cast<int>(nums.size()) - 1;

        while (low <= high) {
            // Avoid (low + high)/2 to prevent integer overflow
            int mid = low + (high - low) / 2;

            if (nums[mid] == target) {
                return mid; // Target found
            } else if (nums[mid] < target) {
                low = mid + 1; // Discard left half
            } else {
                high = mid - 1; // Discard right half
            }
        }

        return -1; // Target not found
    }

    // 2. Recursive Implementation
    int searchRecursive(const std::vector<int>& nums, int target) {
        return recursiveHelper(nums, 0, static_cast<int>(nums.size()) - 1, target);
    }

private:
    int recursiveHelper(const std::vector<int>& nums, int low, int high, int target) {
        // Base case: Search space is exhausted
        if (low > high) {
            return -1;
        }

        // Avoid integer overflow
        int mid = low + (high - low) / 2;

        if (nums[mid] == target) {
            return mid;
        } else if (nums[mid] < target) {
            // Tail recursion: Search right half
            return recursiveHelper(nums, mid + 1, high, target);
        } else {
            // Tail recursion: Search left half
            return recursiveHelper(nums, low, mid - 1, target);
        }
    }
};
```

---

## 8. Code Walkthrough

### Iterative Version Details:
- **`low` and `high` pointers:** Keep track of the active search range. We cast `nums.size()` to `int` because `size()` returns `size_t` (unsigned), and subtracting `1` from `0` on an unsigned type yields a massive positive number (underflow), causing infinite loops.
- **`low <= high` loop condition:** The loop continues as long as there is at least one element to check. If `low == high`, we check that final single element.
- **`low + (high - low) / 2`:** Computes the mathematical midpoint without triggering signed integer overflow.
- **Bound updates:** We use `mid + 1` and `mid - 1` rather than `mid` to guarantee the search space strictly shrinks at every step, avoiding infinite loops.

---

## 9. Dry Run

Let's trace `nums = [-1, 0, 3, 5, 9, 12]`, `target = 2` (not present):

| Step | `low` | `high` | `mid` | `nums[mid]` | Relation | Action |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| Start | 0 | 5 | - | - | - | - |
| 1 | 0 | 5 | 2 | 3 | `3 > 2` | `high = mid - 1` $\rightarrow$ 1 |
| 2 | 0 | 1 | 0 | -1 | `-1 < 2` | `low = mid + 1` $\rightarrow$ 1 |
| 3 | 1 | 1 | 1 | 0 | `0 < 2` | `low = mid + 1` $\rightarrow$ 2 |
| End | 2 | 1 | - | - | `low > high` | Loop terminates, returns `-1` |

---

## 10. Complexity Analysis

| Metric | Iterative | Recursive | Reasoning |
| :--- | :--- | :--- | :--- |
| **Best Case Time** | $O(1)$ | $O(1)$ | Target is exactly at the initial `mid` position. |
| **Average Case Time** | $O(\log n)$ | $O(\log n)$ | On average, the search space is halved $\log_2 n$ times. |
| **Worst Case Time** | $O(\log n)$ | $O(\log n)$ | When target is at the boundary or absent. |
| **Space Complexity** | $O(1)$ | $O(\log n)$ | Iterative uses constant memory. Recursive uses $O(\log n)$ call stack space. |

---

## 11. Common Mistakes

1. **Unsigned underflow on empty vectors:**
   ```cpp
   int high = nums.size() - 1; // If size is 0, nums.size() - 1 becomes 18446744073709551615 (underflow)
   ```
   *Correction:* Check if the vector is empty first or cast to signed `int`.
2. **Integer overflow on mid calculation:**
   Using `(low + high) / 2` instead of `low + (high - low) / 2`.
3. **Incorrect loop condition (`low < high` instead of `low <= high`):**
   Using `<` will skip checking the final remaining element when `low == high`, leading to incorrect results if that single element is the target.
4. **Infinite loop by not shrinking bounds:**
   Updating `low = mid` or `high = mid` instead of `low = mid + 1` and `high = mid - 1`.

---

## 12. Edge Cases

| Edge Case | Input Example | Expected Output | Rationale |
| :--- | :--- | :--- | :--- |
| **Single Element - Match** | `nums = [5]`, `target = 5` | `0` | Loop runs once with `low = high = 0`, finds match. |
| **Single Element - No Match** | `nums = [5]`, `target = 2` | `-1` | First check fails, pointers cross, exits. |
| **Two Elements** | `nums = [3, 5]`, `target = 5` | `1` | Verifies fractional division of `mid`. |
| **Target smaller than all** | `nums = [1, 2, 3]`, `target = 0` | `-1` | `high` pointer moves out of bounds (`high = -1`). |
| **Target larger than all** | `nums = [1, 2, 3]`, `target = 5` | `-1` | `low` pointer moves out of bounds (`low = 3`). |

---

## 13. Interview Explanation

> "To search for a target in a sorted array efficiently, we use Binary Search. Instead of scanning linearly, we compare the target with the middle element of the array. Since the array is sorted, this single comparison lets us discard an entire half of the search space. If the target is smaller than the middle element, we look in the left half; if it's larger, we look in the right half. We repeat this process, updating our search boundaries `low` and `high`. To avoid integer overflow in languages like C++, we calculate the middle index as `low + (high - low) / 2` instead of the naive `(low + high) / 2`. This reduces our search time from $O(n)$ to $O(\log n)$."

---

## 14. Follow-up Questions

1. **How would you modify this to find the first occurrence of a duplicate element?**
   - *Answer:* When `nums[mid] == target`, instead of returning immediately, save `mid` as a potential answer and continue searching in the left half (`high = mid - 1`).
2. **Can you apply binary search to a linked list?**
   - *Answer:* Yes, but access time to the middle node is $O(n)$, making the overall time complexity $O(n)$ anyway, which negates the advantage of binary search.
3. **What is the C++ STL equivalent of standard binary search?**
   - *Answer:* `std::binary_search` (returns boolean), `std::lower_bound` (returns iterator to first element $\ge$ target), and `std::upper_bound` (returns iterator to first element $>$ target).

---

## 15. Variations

1. **Search in a 2D Matrix:** Treat the 2D matrix as a flat 1D array and perform standard binary search, translating `mid` back to `row` and `col` via `/` and `%`.
2. **Search in Rotated Sorted Array:** Find the rotation pivot first or adjust binary search conditions to identify which half of the array is sorted.

---

## 16. Revision Notes

- **Binary Search Template:**
  ```cpp
  int low = 0, high = n - 1;
  while(low <= high) {
      int mid = low + (high - low)/2;
      if(nums[mid] == target) return mid;
      if(nums[mid] < target) low = mid + 1;
      else high = mid - 1;
  }
  ```
- **Mnemonic:** *"Divide, check, and conquer. Keep mid clean of overflow."*
- **Worst-case checks:** $\approx \log_2(10^6) \approx 20$ operations.

---

## 17. Memorization Version

```cpp
int search(vector<int>& nums, int target) {
    int low = 0, high = nums.size() - 1;
    while (low <= high) {
        int mid = low + (high - low) / 2;
        if (nums[mid] == target) return mid;
        else if (nums[mid] < target) low = mid + 1;
        else high = mid - 1;
    }
    return -1;
}
```
*Time:* $O(\log n)$, *Space:* $O(1)$. Prevent overflow with `low + (high - low) / 2`. Ensure signed casts for size arithmetic.
