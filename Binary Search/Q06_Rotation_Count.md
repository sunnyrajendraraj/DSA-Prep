# Find Rotation Count in Rotated Sorted Array

---

## 1. Problem Statement

Given an ascending sorted array of distinct integers `nums` that has been rotated (shifted) to the right by some unknown number of times, find the number of times the array was rotated.

For example, if the original sorted array was `[1, 2, 3, 4, 5]`:
- Rotated 1 time: `[5, 1, 2, 3, 4]`
- Rotated 2 times: `[4, 5, 1, 2, 3]`
- Rotated 0 times: `[1, 2, 3, 4, 5]`

### Input
- `nums`: A rotated sorted array of distinct integers.

### Output
- An integer representing the rotation count.

### Constraints
- $1 \le \text{nums.length} \le 10^5$
- $-10^9 \le \text{nums}[i] \le 10^9$
- All elements in the array are **distinct**.

### Key Observations
1. **Relation to Minimum Element:** The rotation count is exactly equal to the 0-based index of the **minimum element** in the array.
   - For `[4, 5, 1, 2, 3]`, the minimum element is `1` at index `2`. The array was rotated 2 times.
   - For `[1, 2, 3]`, the minimum element is `1` at index `0`. The array was rotated 0 times.
2. **Rotated Sorted Properties:** A rotated sorted array consists of two sorted sub-segments (e.g., `[4, 5]` and `[1, 2, 3]`). The boundary between these two segments is the minimum element (inflection point).
3. **Binary Search Applicability:** We can find the index of the minimum element in $O(\log n)$ time using binary search.

---

## 2. Interview Clarification

Before coding, clarify these details:
1. **Are all elements distinct?** (Yes, standard version assumes distinct elements. If duplicates are present, the time complexity can degrade to $O(n)$ in the worst case, e.g., `[1, 1, 1, 0, 1]`).
2. **Is the rotation direction always to the right (clockwise)?** (Yes, standard clockwise rotation).
3. **What should we return if the array is not rotated?** (Return `0`, which is the index of the minimum element).

---

## 3. Thinking Process

### First Instinct (Brute Force)
Since the array is sorted except at the inflection point, we can scan the array linearly. The minimum element is either the first element (if the array is not rotated) or the element that is smaller than its predecessor (`nums[i] < nums[i - 1]`). The index of this element is our rotation count.
This linear scan runs in $O(n)$ time.

### Optimal Insight (Binary Search for Minimum)
Can we find the minimum element in $O(\log n)$ time? 
Let's look at `nums[mid]` and compare it with the last element `nums[high]`:
1. **`nums[mid] > nums[high]` (Pivot is on the right):**
   - Since `nums[mid]` is larger than `nums[high]`, the inflection point (minimum element) must lie strictly to the right of `mid`.
   - Why? In a normal sorted array, elements increase from left to right. If `mid` is greater than `high`, the sequence must have wrapped around (dropped) somewhere to the right of `mid`.
   - Thus, we update `low = mid + 1`.
2. **`nums[mid] <= nums[high]` (Pivot is on the left or is mid):**
   - The elements from `mid` to `high` are in non-decreasing order. This means the right side is sorted, so the minimum element cannot be in the range `[mid + 1, high]`.
   - The minimum element must be `nums[mid]` itself or lie to the left of `mid`.
   - Thus, we update `high = mid`.

By repeatedly applying this logic, we halve the search space at each step and converge on the index of the minimum element.

---

## 4. Pattern Recognition

This problem belongs to the **Binary Search on Rotated Sorted Array** pattern.
- **Why this topic?** The input is structured in two sorted parts. Comparing the middle element with the boundary element tells us which part of the array contains the inflection point (minimum).
- **Key Mnemonic:** *"Rotation count is the index of the min element."*

---

## 5. Brute Force Solution

### Algorithm
Scan the array from left to right. Keep track of the minimum value and its index. Return the index of the minimum.

### Dry Run
`nums = [4, 5, 1, 2, 3]`
- Start: `min_val = nums[0] = 4`, `min_idx = 0`
- `i = 1`: `nums[1] = 5`. Not smaller.
- `i = 2`: `nums[2] = 1 < 4`. Update `min_val = 1`, `min_idx = 2`.
- `i = 3`: `nums[3] = 2`. Not smaller.
- `i = 4`: `nums[4] = 3`. Not smaller.
- Return `min_idx = 2`.

### Complexity Analysis
- **Time Complexity:** $O(n)$ because we inspect every element once.
- **Space Complexity:** $O(1)$ auxiliary space.

### Pros/Cons
- **Pros:** Simple, handles duplicate values without changes.
- **Cons:** Inefficient; fails to meet the $O(\log n)$ requirement.

### Commented C++17 Code
```cpp
#include <vector>

class BruteForce {
public:
    int findRotationCount(const std::vector<int>& nums) {
        int n = nums.size();
        int min_val = nums[0];
        int min_idx = 0;

        for (int i = 1; i < n; ++i) {
            // Keep track of the smallest element's index
            if (nums[i] < min_val) {
                min_val = nums[i];
                min_idx = i;
            }
        }
        return min_idx;
    }
};
```

---

## 6. Better Solution

For this problem, there is no intermediate solution between the linear $O(n)$ scan and the optimal $O(\log n)$ binary search.

---

## 7. Optimal Solution

The optimal solution is to find the index of the minimum element using binary search.

### Algorithm
1. Initialize `low = 0` and `high = nums.size() - 1`.
2. While `low < high`:
   - Compute `mid = low + (high - low) / 2`.
   - If `nums[mid] > nums[high]`, it means the right half is unsorted and contains the minimum element. Set `low = mid + 1`.
   - Else (if `nums[mid] <= nums[high]`), it means the right half is sorted, and the minimum element is either at `mid` or to its left. Set `high = mid`.
3. When the loop terminates, `low == high`. Return `low`.

### Full Dry Run
`nums = [4, 5, 1, 2, 3]`
- `low = 0`, `high = 4`

#### Iteration 1:
- `mid = 0 + (4 - 0) / 2 = 2`
- `nums[mid] = nums[2] = 1`, `nums[high] = nums[4] = 3`.
- Compare: `1 <= 3`. The right side is sorted.
- Update `high = mid = 2`.

#### Iteration 2:
- `low = 0`, `high = 2`
- `mid = 0 + (2 - 0) / 2 = 1`
- `nums[mid] = nums[1] = 5`, `nums[high] = nums[2] = 1`.
- Compare: `5 > 1`. The minimum must be to the right of `mid`.
- Update `low = mid + 1 = 2`.

- **Termination:** `low = 2`, `high = 2`. Loop terminates because `low < high` is false.
- Return `low = 2`.

### Why Optimal?
We reduce our search space by half at each step by comparing `nums[mid]` with `nums[high]`. This yields a logarithmic time complexity of $O(\log n)$ with $O(1)$ space.

### Beautiful C++17 Code

```cpp
#include <vector>

class Solution {
public:
    int findRotationCount(const std::vector<int>& nums) {
        if (nums.empty()) return 0;

        int low = 0;
        int high = static_cast<int>(nums.size()) - 1;

        // Use low < high to avoid infinite loop when high is updated to mid
        while (low < high) {
            int mid = low + (high - low) / 2;

            if (nums[mid] > nums[high]) {
                // Inflection point (minimum) is in the right half
                low = mid + 1;
            } else {
                // Minimum is mid or lies in the left half
                high = mid;
            }
        }

        // low and high converge to the index of the minimum element
        return low;
    }
};
```

---

## 8. Code Walkthrough

- **`low < high` loop condition:** Since we update `high = mid`, if we used `low <= high`, the loop would get stuck in an infinite cycle when `low == high` because `mid` would equal `low`, and updating `high = mid` would not change the pointers.
- **`nums[mid] > nums[high]` check:** Comparing with `nums[high]` is the most robust way to determine if the right half contains the transition point.
  - If true, `mid` cannot be the minimum (since `nums[high]` is smaller than it). Hence, `low = mid + 1`.
  - If false, the right half is sorted, so the minimum is at most `nums[mid]`. Hence, `high = mid`.

---

## 9. Dry Run

Let's dry run a non-rotated array `nums = [1, 2, 3, 4]`:

| Step | `low` | `high` | `mid` | `nums[mid]` | `nums[high]` | Comparison | Action |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| Init | 0 | 3 | - | - | - | - | - |
| 1 | 0 | 3 | 1 | 2 | 4 | `2 <= 4` | `high = mid` $\rightarrow$ 1 |
| 2 | 0 | 1 | 0 | 1 | 2 | `1 <= 2` | `high = mid` $\rightarrow$ 0 |
| End | 0 | 0 | - | - | - | Pointers met | Return `low = 0` |

---

## 10. Complexity Analysis

| Case | Time Complexity | Space Complexity | Reason |
| :--- | :--- | :--- | :--- |
| **Best Case** | $O(1)$ | $O(1)$ | Array of size 1 (loop doesn't run). |
| **Average Case** | $O(\log n)$ | $O(1)$ | Standard binary search space reduction. |
| **Worst Case** | $O(\log n)$ | $O(1)$ | Inflection point requires full search path. |

---

## 11. Common Mistakes

1. **Comparing `nums[mid]` with `nums[low]` instead of `nums[high]`:**
   Comparing with `low` can lead to bugs. For example, in `[1, 2, 3]`, comparing `2` with `1` (`nums[mid] > nums[low]`) suggests the right half is unsorted, which is incorrect. Comparing with `high` is always safe for finding the minimum.
2. **Infinite Loops:**
   Using `low <= high` loop condition with `high = mid`. When `low = 0, high = 1`, `mid = 0`. If `nums[0] <= nums[1]`, we do `high = mid` $\rightarrow$ `high = 0`. The next loop iteration has `low = 0, high = 0`, leading to `mid = 0`, and the cycle repeats infinitely.
3. **Not recognizing that rotation count equals index of min element:**
   Trying to track pivot elements using complex neighbor comparison rules which have many edge cases. Simply finding the minimum element is cleaner and bug-free.

---

## 12. Edge Cases

| Edge Case | Input | Expected Output | Rationale |
| :--- | :--- | :--- | :--- |
| **No Rotation** | `nums = [1, 2, 3]` | `0` | Minimum is `1` at index `0`. |
| **Single Element** | `nums = [5]` | `0` | Returns `0` directly. |
| **Max Rotations** | `nums = [2, 3, 4, 1]` | `3` | Minimum is `1` at index `3` (rotated 3 times). |

---

## 13. Interview Explanation

> "To find the rotation count of a rotated sorted array of distinct integers, we utilize the key observation that the rotation count is equal to the index of the minimum element in the array. 
> We can find this minimum element's index in $O(\log n)$ time using binary search. We compare the middle element `nums[mid]` with the last element `nums[high]`. 
> If `nums[mid] > nums[high]`, it indicates that the inflection point (where elements drop) must be in the right half of the array, so we search the right side by setting `low = mid + 1`. 
> Otherwise, the right half is sorted, meaning the minimum element must be `nums[mid]` or reside in the left half, so we search the left side by setting `high = mid`. 
> When the pointers converge, they point directly to the minimum element's index, which is our rotation count."

---

## 14. Follow-up Questions

1. **What if duplicate elements are allowed?**
   - *Answer:* If duplicates are allowed, `nums[mid] == nums[high]` can occur (e.g. `[1, 0, 1, 1, 1]`). In this case, we cannot tell which half contains the minimum. We can only shrink the search space safely by decrementing `high` by 1 (`high--`). This degrades the worst-case time complexity to $O(n)$.
   - Duplicate-handling template adjustment:
     ```cpp
     if (nums[mid] > nums[high]) {
         low = mid + 1;
     } else if (nums[mid] < nums[high]) {
         high = mid;
     } else {
         high--; // Shrink search space by 1
     }
     ```

---

## 15. Variations

1. **Search in Rotated Sorted Array (LeetCode 33):** Instead of returning the minimum index, search for a target value in a rotated sorted array.
2. **Find Minimum in Rotated Sorted Array (LeetCode 153):** This is exactly the same problem, but we return the value `nums[low]` instead of the index `low`.

---

## 16. Revision Notes

- **Equivalence:** Rotation Count = Index of Minimum Element.
- **Comparison rule:** Compare `nums[mid]` with `nums[high]`.
- **Bound updates:**
  - If `nums[mid] > nums[high]` $\rightarrow$ `low = mid + 1`
  - Else $\rightarrow$ `high = mid`
- **Mnemonic:** *"Compare mid to high. If mid is higher, step right. Else, slide left."*

---

## 17. Memorization Version

```cpp
int findRotationCount(vector<int>& nums) {
    int low = 0, high = nums.size() - 1;
    while (low < high) {
        int mid = low + (high - low) / 2;
        if (nums[mid] > nums[high]) {
            low = mid + 1;
        } else {
            high = mid;
        }
    }
    return low;
}
```
*Time:* $O(\log n)$, *Space:* $O(1)$. Min element index is the rotation count. Compare `mid` with `high`, loop is `low < high`.
