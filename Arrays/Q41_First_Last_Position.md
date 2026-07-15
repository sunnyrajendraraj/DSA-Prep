# Find First and Last Position of Element in Sorted Array

## 1. Problem Statement

Given an array of integers `nums` sorted in non-decreasing order, find the starting and ending position of a given `target` value.
If `target` is not found in the array, return `[-1, -1]`.

### Input
- A sorted array of integers `nums`.
- An integer `target`.

### Output
- A vector/array of size 2: `[first_position, last_position]`.

### Constraints
- $0 \le nums.size() \le 10^5$
- $-10^9 \le nums[i], target \le 10^9$
- `nums` is sorted in non-decreasing order.

### Observations
1. The array is already sorted, which strongly hints at using Binary Search to find elements in $O(\log n)$ time.
2. The target can appear multiple times. We need to find the boundary occurrences (leftmost and rightmost).

---

## 2. Interview Clarification

Before writing code, clarify these points with the interviewer:
1. **Can the array contain duplicates?**
   - *Yes, this is the core of the problem: finding the range of duplicates.*
2. **What if the target is not in the array?**
   - *Return `[-1, -1]`.*
3. **What if the array is empty?**
   - *Return `[-1, -1]`.*
4. **Should the solution be optimal in terms of time complexity?**
   - *Yes, we should aim for $O(\log n)$ complexity.*

---

## 3. Thinking Process

Let's build the intuition step-by-step:
- **First Instinct (Brute Force):**
  We can perform a linear scan from left to right.
  - The first index where `nums[i] == target` is the starting position.
  - The last index where `nums[i] == target` is the ending position.
  This runs in $O(n)$ time. It does not utilize the fact that the array is sorted.

- **Improving (Better Solution):**
  We can use standard Binary Search to find *any* occurrence of the target.
  Once we locate it at index `mid`, we can perform a linear scan:
  - Scan leftwards from `mid` until we find an element not equal to `target`. The index just after that is the first position.
  - Scan rightwards from `mid` until we find an element not equal to `target`. The index just before that is the last position.
  - *Bottleneck:* If the array consists entirely of the target (e.g., `[5, 5, 5, 5, 5]`, target = 5), the linear scans will traverse the entire array, degrading the time complexity to $O(n)$ in the worst case.

- **Optimal Insight (Two Binary Searches):**
  To maintain a guaranteed $O(\log n)$ time complexity even in the worst case, we can run two distinct binary searches:
  1. **First Binary Search (Leftmost Bound)**:
     - Standard binary search. When we find `nums[mid] == target`, instead of stopping, we record `mid` as a potential first occurrence. Since we want the *first* occurrence, we continue searching in the left half by updating our search space to `[low, mid - 1]`.
  2. **Second Binary Search (Rightmost Bound)**:
     - Standard binary search. When we find `nums[mid] == target`, we record `mid` as a potential last occurrence and continue searching in the right half by updating our search space to `[mid + 1, high]`.

  By running these two searches independently (or as a helper function), we get both indices in $O(\log n)$ time.

---

## 4. Pattern Recognition

This problem fits the **Binary Search: Boundary Finding** pattern.
- **Why this topic?** The array is sorted, and we need to locate specific boundaries of target intervals.
- **Pattern:**
  - Standard binary search is modified to find the "first" or "last" occurrence by continuing the search in the left/right subtree when the target is found.

---

## 5. Brute Force Solution

### Algorithm
1. Initialize `first = -1` and `last = -1`.
2. Iterate through the array from `i = 0` to `n - 1`:
   - If `nums[i] == target`:
     - If `first == -1`, set `first = i`.
     - Update `last = i`.
3. Return `[first, last]`.

### Complexity Analysis
- **Time Complexity:** $O(n)$ since we scan the entire array of size $n$.
- **Space Complexity:** $O(1)$ auxiliary space.

### Pros and Cons
- **Pros:** Extremely simple to implement.
- **Cons:** Very inefficient for large arrays; doesn't utilize sorting.

### C++17 Code
```cpp
#include <vector>

class SolutionBrute {
public:
    std::vector<int> searchRange(const std::vector<int>& nums, int target) {
        int first = -1;
        int last = -1;
        int n = nums.size();
        
        for (int i = 0; i < n; ++i) {
            if (nums[i] == target) {
                if (first == -1) {
                    first = i;
                }
                last = i;
            }
        }
        
        return {first, last};
    }
};
```

---

## 6. Better Solution

### Algorithm
1. Perform standard binary search to find the target.
2. If target is found at index `mid`:
   - Initialize `first = mid`, `last = mid`.
   - While `first > 0` and `nums[first - 1] == target`, decrement `first`.
   - While `last < n - 1` and `nums[last + 1] == target`, increment `last`.
   - Return `[first, last]`.
3. If standard binary search returns `-1`, return `[-1, -1]`.

### Complexity Analysis
- **Time Complexity:** 
  - **Average Case:** $O(\log n)$ if duplicate size is small.
  - **Worst Case:** $O(n)$ if the array is filled with the target.
- **Space Complexity:** $O(1)$ auxiliary space.

### Pros and Cons
- **Pros:** Faster than brute force for typical arrays.
- **Cons:** Worst-case time complexity is still $O(n)$.

### C++17 Code
```cpp
#include <vector>

class SolutionBetter {
public:
    std::vector<int> searchRange(const std::vector<int>& nums, int target) {
        int n = nums.size();
        int low = 0, high = n - 1;
        int foundIdx = -1;
        
        // Find any occurrence
        while (low <= high) {
            int mid = low + (high - low) / 2;
            if (nums[mid] == target) {
                foundIdx = mid;
                break;
            } else if (nums[mid] < target) {
                low = mid + 1;
            } else {
                high = mid - 1;
            }
        }
        
        if (foundIdx == -1) {
            return {-1, -1};
        }
        
        // Expand outward
        int first = foundIdx;
        while (first > 0 && nums[first - 1] == target) {
            first--;
        }
        
        int last = foundIdx;
        while (last < n - 1 && nums[last + 1] == target) {
            last++;
        }
        
        return {first, last};
    }
};
```

---

## 7. Optimal Solution

### Algorithm
We write a helper binary search function `findBound(nums, target, isFirst)` which returns the index of the boundary.
- If `isFirst` is `true`, search for the first (leftmost) occurrence.
- If `isFirst` is `false`, search for the last (rightmost) occurrence.

Inside `findBound`:
1. Initialize `low = 0`, `high = n - 1`, and `ans = -1`.
2. While `low <= high`:
   - Compute `mid = low + (high - low) / 2`.
   - If `nums[mid] == target`:
     - Set `ans = mid`.
     - If `isFirst` is `true`, search left: set `high = mid - 1`.
     - Else, search right: set `low = mid + 1`.
   - Else if `nums[mid] < target`, set `low = mid + 1`.
   - Else, set `high = mid - 1`.
3. Return `ans`.

Our main function simply returns `{findBound(nums, target, true), findBound(nums, target, false)}`.

### C++17 Code
```cpp
#include <vector>

class SolutionOptimal {
private:
    // Helper function to find the first or last index of the target
    int findBound(const std::vector<int>& nums, int target, bool isFirst) {
        int n = nums.size();
        int low = 0;
        int high = n - 1;
        int resultIndex = -1;
        
        while (low <= high) {
            int mid = low + (high - low) / 2;
            
            if (nums[mid] == target) {
                resultIndex = mid; // Record potential answer
                
                if (isFirst) {
                    // Search left to find an earlier occurrence
                    high = mid - 1;
                } else {
                    // Search right to find a later occurrence
                    low = mid + 1;
                }
            } else if (nums[mid] < target) {
                low = mid + 1; // Target is in the right half
            } else {
                high = mid - 1; // Target is in the left half
            }
        }
        
        return resultIndex;
    }

public:
    std::vector<int> searchRange(const std::vector<int>& nums, int target) {
        int first = findBound(nums, target, true);
        int last = findBound(nums, target, false);
        return {first, last};
    }
};
```

---

## 8. Code Walkthrough

Let's break down the helper function `findBound(nums, target, isFirst)`:
1. `int low = 0; int high = n - 1; int resultIndex = -1;`
   - Sets up search boundaries and initializes result index to `-1` (default if target is not found).
2. `int mid = low + (high - low) / 2;`
   - Finds the midpoint to avoid potential overflow compared to `(low + high) / 2`.
3. `if (nums[mid] == target)`
   - Target found!
   - `resultIndex = mid;` matches current target.
   - If `isFirst` is true (leftmost bound requested):
     - `high = mid - 1;` we shrink the search window to the left side, checking if there is another matching target at a smaller index.
   - If `isFirst` is false (rightmost bound requested):
     - `low = mid + 1;` we shrink the search window to the right side, checking if there is another matching target at a larger index.
4. `else if (nums[mid] < target)` and `else { high = mid - 1; }`
   - Adjusts search space normally if the element at `mid` is not equal to target.

---

## 9. Dry Run

Let `nums = [5, 7, 7, 8, 8, 10]`, `target = 8`.
- **First Search: `findBound(nums, 8, true)` (Looking for first position)**

| Iteration | `low` | `high` | `mid` | `nums[mid]` | Action | `resultIndex` |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **1** | 0 | 5 | 2 | 7 | $7 < 8 \implies$ set `low = 3` | -1 |
| **2** | 3 | 5 | 4 | 8 | $8 == 8 \implies$ set `resultIndex = 4`, `high = 3` | 4 |
| **3** | 3 | 3 | 3 | 8 | $8 == 8 \implies$ set `resultIndex = 3`, `high = 2` | **3** |
| **Loop Ends** | 3 | 2 | — | — | `low > high` (3 > 2), terminate | 3 |

First position = `3`.

- **Second Search: `findBound(nums, 8, false)` (Looking for last position)**

| Iteration | `low` | `high` | `mid` | `nums[mid]` | Action | `resultIndex` |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **1** | 0 | 5 | 2 | 7 | $7 < 8 \implies$ set `low = 3` | -1 |
| **2** | 3 | 5 | 4 | 8 | $8 == 8 \implies$ set `resultIndex = 4`, `low = 5` | 4 |
| **3** | 5 | 5 | 5 | 10 | $10 > 8 \implies$ set `high = 4` | 4 |
| **Loop Ends** | 5 | 4 | — | — | `low > high` (5 > 4), terminate | **4** |

Last position = `4`.

**Final Output:** `[3, 4]`.

---

## 10. Complexity Analysis

| Metric | Complexity | Reasoning |
| :--- | :--- | :--- |
| **Time Complexity** | $O(\log n)$ | We run two independent binary searches. Each binary search splits the search space in half at every step, taking $O(\log n)$ time. Total time is $2 \cdot O(\log n) = O(\log n)$. |
| **Space Complexity** | $O(1)$ | No dynamic memory allocation; only a few variables are stored in the stack frame. |

---

## 11. Common Mistakes

1. **Incorrect Update of Boundaries**:
   - Forgetting to shift `low` or `high` when the element is found. If you set `high = mid` (instead of `mid - 1`) or `low = mid` (instead of `mid + 1`), the loop might run infinitely when `low == high`.
2. **Mixing up First and Last bounds search space redirection**:
   - Directing the search to the right side when looking for the first index, or directing to the left when looking for the last. Remember: **First $\implies$ search left (`high = mid - 1`)**; **Last $\implies$ search right (`low = mid + 1`)**.
3. **Overflow on Midpoint calculation**:
   - Writing `(low + high) / 2`. If `low` and `high` are very large, their sum can exceed standard integer range. Always use `low + (high - low) / 2`.

---

## 12. Edge Cases

| Edge Case | Input | Expected Output | Why It Matters |
| :--- | :--- | :--- | :--- |
| Target not present | `nums = [1, 2, 4, 5], target = 3` | `[-1, -1]` | Verify no false matches |
| Empty array | `nums = [], target = 0` | `[-1, -1]` | Boundary check to prevent segmentation fault |
| Single element (matching) | `nums = [5], target = 5` | `[0, 0]` | Standard base case |
| Single element (non-matching) | `nums = [5], target = 3` | `[-1, -1]` | Standard base case |
| All elements match target | `nums = [2, 2, 2, 2], target = 2` | `[0, 3]` | Tests boundary search over large duplicates |

---

## 13. Interview Explanation

*"To find the first and last position of a target in a sorted array, we can use binary search to achieve logarithmic time complexity.
If we do a simple binary search and expand outward, the worst case where all elements match the target will degrade to $O(n)$ time.
Instead, we perform two independent binary searches.
For the first binary search, when we find the target, we record its index as a candidate for the first occurrence, and then we continue searching in the left sub-segment by updating our `high` pointer to `mid - 1`. This finds the leftmost bound.
For the second binary search, when we find the target, we record its index as a candidate for the last occurrence, and then we continue searching in the right sub-segment by updating our `low` pointer to `mid + 1`. This finds the rightmost bound.
Each search takes $O(\log n)$ time, resulting in an optimal $O(\log n)$ time and $O(1)$ space solution."*

---

## 14. Follow-up Questions

1. **Can we implement this using C++ STL standard functions?**
   - *Yes. We can use `std::lower_bound` to find the first index and `std::upper_bound` to find the index after the last. The index of the first occurrence is `it1 - nums.begin()`. If the element is found, the last occurrence index is `(it2 - nums.begin()) - 1`.*
2. **What happens if target values are floating point numbers?**
   - *The logic remains exactly the same; comparison operator types change to `double` or `float`.*

---

## 15. Variations

1. **Count Occurrences of a Target**: Discussed in the next problem. We find `last - first + 1`.
2. **Search Insertion Position (LeetCode 35)**: Find where the target should be inserted if it's not present (equivalent to finding the lower bound).

---

## 16. Revision Notes

- **Leftmost search**: If `nums[mid] == target`, update `ans = mid` and `high = mid - 1`.
- **Rightmost search**: If `nums[mid] == target`, update `ans = mid` and `low = mid + 1`.
- **Optimal Time**: $O(\log n)$, **Space**: $O(1)$.
- **STL Alternative**: `lower_bound` and `upper_bound`.

---

## 17. Memorization Version

```cpp
// Optimal First and Last Position Binary Search
int findBound(vector<int>& nums, int target, bool isFirst) {
    int low = 0, high = nums.size() - 1, ans = -1;
    while (low <= high) {
        int mid = low + (high - low) / 2;
        if (nums[mid] == target) {
            ans = mid;
            if (isFirst) high = mid - 1;
            else low = mid + 1;
        } else if (nums[mid] < target) low = mid + 1;
        else high = mid - 1;
    }
    return ans;
}
vector<int> searchRange(vector<int>& nums, int target) {
    return {findBound(nums, target, true), findBound(nums, target, false)};
}
```
*Time: $O(\log n)$, Space: $O(1)$*
