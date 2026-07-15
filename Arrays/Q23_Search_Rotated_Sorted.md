# Search in Rotated Sorted Array

## 1. Problem Statement

There is an integer array `nums` sorted in ascending order (with **distinct** values). Prior to being passed to your function, `nums` is possibly rotated at an unknown pivot index `k` ($1 \le k < nums.length$) such that the resulting array is `[nums[k], nums[k+1], ..., nums[n-1], nums[0], nums[1], ..., nums[k-1]]` (0-indexed).

For example, `[0, 1, 2, 4, 5, 6, 7]` might be rotated at pivot index `3` and become `[4, 5, 6, 7, 0, 1, 2]`.

Given the array `nums` **after** the possible rotation and an integer `target`, return the *index* of `target` if it is in `nums`, or `-1` if it is not in `nums`.

You must write an algorithm with $O(\log N)$ runtime complexity.

### Examples
- **Input:** `nums = [4, 5, 6, 7, 0, 1, 2]`, `target = 0`
- **Output:** 4

- **Input:** `nums = [4, 5, 6, 7, 0, 1, 2]`, `target = 3`
- **Output:** -1

- **Input:** `nums = [1]`, `target = 0`
- **Output:** -1

### Constraints
- $1 \le N \le 5000$
- $-10^4 \le nums[i] \le 10^4$
- All values of `nums` are **unique**.
- `nums` is an ascending array that is rotated.
- $-10^4 \le target \le 10^4$

---

## 2. Interview Clarification Questions

Before writing any code, a candidate should clarify:
1. **Q:** Can there be duplicate values in the array?
   * **A:** No, all values are distinct. *(Note: If duplicates are present, it is a different problem: Search in Rotated Sorted Array II).*
2. **Q:** What should we return if the target is not found?
   * **A:** Return `-1`.
3. **Q:** Can the array be empty?
   * **A:** According to the constraints, $N \ge 1$, so it will have at least one element.

---

## 3. Thinking Process

### Step 1: First Instinct (Brute Force)
The simplest solution is to perform a linear search:
- Iterate through `nums` from index $0$ to $N-1$.
- Check if `nums[i] == target`.
- If found, return `i`.
- If the loop finishes without finding the target, return `-1`.
- This runs in $O(N)$ time complexity and $O(1)$ space.

### Step 2: Binary Search (Optimal Insight)
The problem explicitly requests an $O(\log N)$ solution, pointing directly to **Binary Search**.
- In standard binary search, we split the array and compare `nums[mid]` with `target`.
- In a rotated sorted array, the sorted order is broken at the rotation pivot.
- **Key Observation:** If we split the array at any index `mid`, **at least one of the halves (left or right) will always be sorted**.
  - Let's determine which half is sorted by comparing the boundary values:
    - If `nums[low] <= nums[mid]`, then the left half `[low...mid]` is sorted.
    - Otherwise, the right half `[mid...high]` must be sorted.
  
- Once we identify the sorted half, we check if the `target` lies within its boundaries:
  - **If the left half is sorted:**
    - If `nums[low] <= target && target < nums[mid]`, then the target must lie in this left sorted portion. We search here by setting `high = mid - 1`.
    - Otherwise, the target is in the right half. We set `low = mid + 1`.
  - **If the right half is sorted:**
    - If `nums[mid] < target && target <= nums[high]`, the target must lie in this right sorted portion. We search here by setting `low = mid + 1`.
    - Otherwise, the target is in the left half. We set `high = mid - 1`.

---

## 4. Pattern Recognition

This problem fits the **Modified Binary Search** pattern.
- **Why?** The array is partitioned into sorted subsets. In binary search, we look for sorted ranges because we can make definitive decisions about target inclusion in $O(1)$ boundary checks, enabling us to eliminate half of the search space.

---

## 5. Brute Force Solution

### Algorithm
1. Loop through `i` from `0` to `n - 1`.
2. If `nums[i] == target`, return `i`.
3. Return `-1` after the loop.

### Complexity
- **Time Complexity:** $O(N)$ since we may scan the entire array.
- **Space Complexity:** $O(1)$ auxiliary space.

### Pros and Cons
- **Pros:** Simple, handles duplicates automatically.
- **Cons:** Too slow, does not satisfy the $O(\log N)$ constraint.

### C++17 Code
```cpp
#include <vector>

int searchBrute(const std::vector<int>& nums, int target) {
    for (size_t i = 0; i < nums.size(); ++i) {
        if (nums[i] == target) {
            return static_cast<int>(i);
        }
    }
    return -1;
}
```

---

## 6. Better Solution

There is no intermediate "better" solution for this problem. The brute force solution is $O(N)$ and the optimal solution is $O(\log N)$. We jump directly to the optimal solution.

---

## 7. Optimal Solution (Binary Search)

### Algorithm
1. Initialize two pointers: `low = 0` and `high = nums.size() - 1`.
2. While `low <= high`:
   - Calculate midpoint: `mid = low + (high - low) / 2`.
   - If `nums[mid] == target`, return `mid`.
   - **Identify the sorted half:**
     - **If Left Half is Sorted (`nums[low] <= nums[mid]`):**
       - Check if target is within the left boundaries: `nums[low] <= target && target < nums[mid]`.
       - If yes, narrow search to left: `high = mid - 1`.
       - If no, search right: `low = mid + 1`.
     - **If Right Half is Sorted:**
       - Check if target is within the right boundaries: `nums[mid] < target && target <= nums[high]`.
       - If yes, narrow search to right: `low = mid + 1`.
       - If no, search left: `high = mid - 1`.
3. If the loop ends, return `-1`.

### C++17 Code

```cpp
#include <vector>

int search(const std::vector<int>& nums, int target) {
    int low = 0;
    int high = nums.size() - 1;
    
    while (low <= high) {
        int mid = low + (high - low) / 2;
        
        if (nums[mid] == target) {
            return mid;
        }
        
        // Check if the left half is sorted
        if (nums[low] <= nums[mid]) {
            // Check if target lies in the sorted left half
            if (nums[low] <= target && target < nums[mid]) {
                high = mid - 1; // Search left
            } else {
                low = mid + 1;  // Search right
            }
        } 
        // Otherwise, the right half must be sorted
        else {
            // Check if target lies in the sorted right half
            if (nums[mid] < target && target <= nums[high]) {
                low = mid + 1;  // Search right
            } else {
                high = mid - 1; // Search left
            }
        }
    }
    
    return -1; // Target not found
}
```

---

## 8. Code Walkthrough

1. **Initialization:** `low = 0`, `high = nums.size() - 1`.
2. **Standard Loop Condition:** `while (low <= high)` allows searching down to a single element.
3. **Midpoint & Match Check:** `mid` is computed, and we immediately return it if `nums[mid] == target`.
4. **Branch 1 (Left Sorted):** `nums[low] <= nums[mid]` detects that the left side has no rotation boundary.
   - We then check if the target lies within the range `[nums[low], nums[mid])`. If it does, we shift `high = mid - 1`. Otherwise, we search the right side.
5. **Branch 2 (Right Sorted):** If the left side is not sorted, the right side must be sorted.
   - We check if the target lies within the range `(nums[mid], nums[high]]`. If it does, we shift `low = mid + 1`. Otherwise, we search the left side.

---

## 9. Dry Run

Input: `nums = [4, 5, 6, 7, 0, 1, 2]`, `target = 0`

### Initialization:
- `low = 0`, `high = 6`

### Iterations:

| Step | `low` | `high` | `mid` | `nums[mid]` | Sorted Half | Range Check for Target (`0`) | Action |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **1** | 0 | 6 | 3 | 7 | Left is sorted (`nums[0] <= nums[3]` is `4 <= 7`) | Is `4 <= 0 < 7`? (False) | `low = 3 + 1 = 4` |
| **2** | 4 | 6 | 5 | 1 | Right is sorted (`nums[4] <= nums[5]` is `0 <= 1`) | Is `0 <= 0 < 1`? (True) | `high = 5 - 1 = 4` |
| **3** | 4 | 4 | 4 | 0 | Left is sorted (`nums[4] <= nums[4]` is `0 <= 0`) | `nums[4] == target` (Match!) | Return `4` |

Return value: `4`.

---

## 10. Complexity Analysis

- **Time Complexity:** $O(\log N)$. At each iteration, we eliminate half of the search space, performing a constant number of comparisons.
- **Space Complexity:** $O(1)$ because we only use a few pointer variables.

---

## 11. Common Mistakes

1. **Incorrect comparisons:** Writing `nums[low] < target` instead of `nums[low] <= target`. If the target is equal to the element at `low`, it will be incorrectly skipped.
2. **Mismatching sorting condition:** Confusing `nums[low] <= nums[mid]` with `nums[mid] >= nums[high]`. Stick to comparing the middle with one end consistently.
3. **Calculating mid incorrectly:** Using `(low + high) / 2` which can cause integer overflow for large indices.

---

## 12. Edge Cases

| Edge Case | Input Example | Why it matters | Correct Output |
| :--- | :--- | :--- | :--- |
| **Target at low boundary** | `[3, 1]`, `target = 3` | Verifies the `<=` condition on boundary checks. | `0` |
| **Target at high boundary** | `[3, 1]`, `target = 1` | Verifies target matching at high index. | `1` |
| **Single Element (Match)** | `[5]`, `target = 5` | Checks single iteration convergence. | `0` |
| **Single Element (Mismatch)** | `[5]`, `target = 2` | Handled correctly by terminating loop. | `-1` |
| **Array Not Rotated** | `[1, 2, 3]`, `target = 2` | Handled properly since left half is always sorted. | `1` |

---

## 13. Interview Explanation

"To search for a target in a rotated sorted array in $O(\log N)$ time, we modify standard binary search. 

At any given point in a rotated sorted array, when we divide the array in half using a middle element, one of the two halves is guaranteed to be sorted. We can check if the left half is sorted by verifying if the left element is less than or equal to the middle element. 

If the left half is sorted, we check if the target lies within the left boundaries. If it does, we narrow our search to the left half; otherwise, we search the right half. 

If the left half is not sorted, it means the right half must be sorted. We check if the target lies within the right boundaries. If it does, we search the right half; otherwise, we search the left half. We repeat this process, cutting the search space in half at each step."

---

## 14. Follow-up Questions

1. **Q: What if duplicates are allowed? (e.g., `nums = [2, 5, 6, 0, 0, 1, 2]`)**
   * *A:* If duplicates are present, we can encounter a state where `nums[low] == nums[mid] == nums[high]`. When this happens, we cannot tell which half is sorted. We must increment `low` and decrement `high` to shrink the window, which makes the worst-case time complexity $O(N)$.
2. **Q: How can we find the index of the pivot (the point where rotation occurs)?**
   * *A:* The pivot point is the index of the minimum element. We can find the minimum element in $O(\log N)$ time using the algorithm from the previous module.

---

## 15. Variations

1. **Search in Rotated Sorted Array II (with duplicates):**
   - Requires handling the `nums[low] == nums[mid] == nums[high]` condition.
2. **Find Rotation Count:**
   - Find how many times the array was rotated (equal to the index of the minimum element).

---

## 16. Revision Notes

- **Pivot Invariant:** At least one half is always sorted.
- **Sorted Checks:**
  - Left sorted: `nums[low] <= nums[mid]`
  - Right sorted: `nums[mid] <= nums[high]` (implicitly handled if left check fails)
- **Target Checks:** Include boundary checks (`<=` and `>=`) when comparing targets with endpoints.

---

## 17. Memorization Version

```cpp
// 1-min Review Version
int search(vector<int>& nums, int target) {
    int low = 0, high = nums.size() - 1;
    while (low <= high) {
        int mid = low + (high - low) / 2;
        if (nums[mid] == target) return mid;
        if (nums[low] <= nums[mid]) {
            if (nums[low] <= target && target < nums[mid]) high = mid - 1;
            else low = mid + 1;
        } else {
            if (nums[mid] < target && target <= nums[high]) low = mid + 1;
            else high = mid - 1;
        }
    }
    return -1;
}
```
*Complexity:* $O(\log N)$ Time, $O(1)$ Space.
