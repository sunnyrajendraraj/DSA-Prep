# Peak Element in Array

---

## 1. Problem Statement

A peak element is an element that is strictly greater than its neighbors.

Given a 0-indexed integer array `nums`, find a peak element, and return its index. If the array contains multiple peaks, return the index to **any of the peaks**.

You may imagine that `nums[-1] = nums[n] = -∞`. In other words, an element is always considered to be strictly greater than a neighbor that is outside the array.

You must write an algorithm that runs in $O(\log n)$ time.

### Input
- `nums`: An integer array.

### Output
- The 0-based index of any peak element in `nums`.

### Constraints
- $1 \le \text{nums.length} \le 1000$
- $-2^{31} \le \text{nums}[i] \le 2^{31} - 1$
- `nums[i] != nums[i + 1]` for all valid `i`. (No two adjacent elements are equal).

### Key Observations
1. **Implicit Boundaries:** The virtual boundaries at indices `-1` and `n` are $-\infty$. This guarantees that a peak element **always exists** in any non-empty array.
2. **No Adjacent Duplicates:** Because `nums[i] != nums[i+1]`, we will never encounter flat plateaus. Every step is either strictly ascending or strictly descending.
3. **Local Property vs. Global Sorting:** Even though the array is **not sorted**, we can still apply Binary Search. This is a critical concept: binary search does not strictly require a sorted array; it only requires a way to decide which half contains the target.

---

## 2. Interview Clarification

Before coding, clarify these details:
1. **Can there be multiple peaks?** (Yes. We can return the index of any peak).
2. **What if the array is sorted in ascending order?** (The last element is the peak, since `nums[n] = -∞`).
3. **What if the array is sorted in descending order?** (The first element is the peak, since `nums[-1] = -∞`).
4. **Is it guaranteed that adjacent elements are never equal?** (Yes, as per standard constraints. If duplicates were allowed, finding a peak in $O(\log n)$ would be mathematically impossible in the worst case, e.g., `[1, 1, 1, 1, 1, 2, 1]`, and we would need $O(n)$ time).

---

## 3. Thinking Process

### First Instinct (Brute Force)
Since a peak is strictly greater than its neighbors, we can scan the array from left to right.
- For index `0`, we compare it with `nums[1]`. If `nums[0] > nums[1]`, then `0` is a peak (since the left neighbor is $-\infty$).
- For any index `i`, if `nums[i] > nums[i-1]` and `nums[i] > nums[i+1]`, it is a peak.
- For index `n-1`, if `nums[n-1] > nums[n-2]`, it is a peak.
This linear scan runs in $O(n)$ time.

### Optimal Insight (Binary Search on Slopes)
Can we find a peak in $O(\log n)$? Let's analyze the properties of a middle element `nums[mid]` and its right neighbor `nums[mid+1]`.

Since adjacent elements are unequal, there are only two possibilities:
1. **`nums[mid] < nums[mid+1]` (Ascending Slope):**
   - The sequence is rising from `mid` to `mid+1`.
   - Since the rightmost boundary `nums[n]` drops to $-\infty$, the sequence must eventually drop.
   - Therefore, a peak **must** exist somewhere to the right of `mid` (in the range `[mid + 1, n-1]`). We can safely discard the left half.
2. **`nums[mid] > nums[mid+1]` (Descending Slope):**
   - The sequence is falling from `mid` to `mid+1`.
   - Since the leftmost boundary `nums[-1]` is $-\infty$, the sequence must have risen to at least `nums[mid]` from the left.
   - Therefore, a peak **must** exist at `mid` or somewhere to the left of `mid` (in the range `[0, mid]`). We can safely discard the right half.

By checking the relationship between `nums[mid]` and `nums[mid+1]`, we can eliminate half of the search space at each step!

---

## 4. Pattern Recognition

This problem belongs to the **Binary Search on Unsorted Array (Slope Method)** pattern.
- **Why this topic?** We can establish a binary decision boundary at every index based on whether we are on a rising slope or a falling slope.
- **Key Lesson:** Sorted order is *not* a prerequisite for Binary Search. The only prerequisite is the ability to determine which half of the search space can be safely discarded.

---

## 5. Brute Force Solution

### Algorithm
Perform a linear scan. The first element that is greater than its right neighbor is a peak.
*Proof:* If we iterate from left to right, we know `nums[i] > nums[i-1]` is true for all visited elements (otherwise we would have stopped at `i-1` since it would be greater than `nums[i]`). Thus, we only need to check if `nums[i] > nums[i+1]`.

### Dry Run
`nums = [1, 2, 1, 3, 5, 6, 4]`
- `i = 0`: `nums[0] = 1`. `nums[1] = 2`. `1 < 2`, not a peak.
- `i = 1`: `nums[1] = 2`. `nums[2] = 1`. `2 > 1` (And we know `2 > nums[0]` since we reached here). Return index `1`.

### Complexity Analysis
- **Time Complexity:** $O(n)$ in the worst case (e.g., strictly increasing array, where peak is at the end).
- **Space Complexity:** $O(1)$ auxiliary space.

### Pros/Cons
- **Pros:** Simple, works with duplicate adjacent elements if we adapt it.
- **Cons:** Inefficient for large arrays; does not meet the $O(\log n)$ requirement.

### Commented C++17 Code
```cpp
#include <vector>

class BruteForce {
public:
    int findPeakElement(const std::vector<int>& nums) {
        int n = nums.size();
        for (int i = 0; i < n; ++i) {
            // Check boundary conditions along with neighbors
            bool left_ok = (i == 0 || nums[i] > nums[i - 1]);
            bool right_ok = (i == n - 1 || nums[i] > nums[i + 1]);
            
            if (left_ok && right_ok) {
                return i;
            }
        }
        return -1;
    }
};
```

---

## 6. Better Solution

For this problem, there is no intermediate solution between the linear $O(n)$ scan and the optimal $O(\log n)$ binary search.

---

## 7. Optimal Solution

The optimal solution is **Binary Search** using a convergence template.

### Algorithm
1. Initialize `low = 0` and `high = nums.size() - 1`.
2. While `low < high`:
   - Compute `mid = low + (high - low) / 2`.
   - Compare `nums[mid]` with `nums[mid + 1]`:
     - If `nums[mid] < nums[mid + 1]`, we are on an ascending slope. The peak is on the right side. Set `low = mid + 1`.
     - If `nums[mid] > nums[mid + 1]`, we are on a descending slope. The peak is on the left side (including `mid`). Set `high = mid`.
3. When the loop terminates, `low == high` and both point to a peak element. Return `low`.

### Full Dry Run
`nums = [1, 2, 1, 3, 5, 6, 4]`
- `low = 0`, `high = 6`

#### Iteration 1:
- `mid = 0 + (6 - 0) / 2 = 3`
- `nums[mid] = nums[3] = 3`
- Compare with `nums[mid+1] = nums[4] = 5`.
- Since `3 < 5` (ascending slope), peak must be on the right.
- Update `low = mid + 1 = 4`.

#### Iteration 2:
- `low = 4`, `high = 6`
- `mid = 4 + (6 - 4) / 2 = 5`
- `nums[mid] = nums[5] = 6`
- Compare with `nums[mid+1] = nums[6] = 4`.
- Since `6 > 4` (descending slope), peak must be on the left (including `mid`).
- Update `high = mid = 5`.

#### Iteration 3:
- `low = 4`, `high = 5`
- `mid = 4 + (5 - 4) / 2 = 4`
- `nums[mid] = nums[4] = 5`
- Compare with `nums[mid+1] = nums[5] = 6`.
- Since `5 < 6` (ascending slope), peak must be on the right.
- Update `low = mid + 1 = 5`.

- **Termination:** `low = 5`, `high = 5`. Loop terminates since `low < high` is false.
- Return `low = 5` (which points to element `6`, a valid peak).

### Why Optimal?
Each step evaluates the slope at `mid` and shrinks the search space by half. The check is local ($O(1)$) and avoids accessing out-of-bounds indices because `mid` can never equal `high` when `low < high`, ensuring `mid + 1` is always a valid index. This guarantees finding a peak in $O(\log n)$ time.

### Beautiful C++17 Code

```cpp
#include <vector>

class Solution {
public:
    int findPeakElement(const std::vector<int>& nums) {
        int low = 0;
        int high = static_cast<int>(nums.size()) - 1;

        // Use low < high because we compare mid with mid + 1.
        // This prevents mid + 1 from going out of bounds.
        while (low < high) {
            int mid = low + (high - low) / 2;

            if (nums[mid] < nums[mid + 1]) {
                // Ascending slope: peak lies strictly to the right
                low = mid + 1;
            } else {
                // Descending slope: peak lies to the left (including mid)
                high = mid;
            }
        }

        // low and high converge to the peak index
        return low;
    }
};
```

---

## 8. Code Walkthrough

- **`low < high` loop condition:** Since we look at `nums[mid + 1]`, we must ensure that `mid + 1` is a valid index. If we used `low <= high`, then when `low == high`, `mid` would be equal to `high`, and checking `mid + 1` would access an out-of-bounds element.
- **`low = mid + 1`:** When `nums[mid] < nums[mid+1]`, `mid` itself cannot be a peak because its right neighbor is strictly greater. Thus, we exclude `mid` and search `[mid + 1, high]`.
- **`high = mid`:** When `nums[mid] > nums[mid+1]`, `mid` could be the peak, or the peak could be to its left. We cannot exclude `mid` yet, so we shrink our window to `[low, mid]`.

---

## 9. Dry Run

Let's dry run `nums = [3, 2, 1]`:

| Step | `low` | `high` | `mid` | `nums[mid]` | `nums[mid+1]` | Slope Direction | Action |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| Init | 0 | 2 | - | - | - | - | - |
| 1 | 0 | 2 | 1 | 2 | 1 | Descending (`2 > 1`) | `high = mid` $\rightarrow$ 1 |
| 2 | 0 | 1 | 0 | 3 | 2 | Descending (`3 > 2`) | `high = mid` $\rightarrow$ 0 |
| End | 0 | 0 | - | - | - | Pointers met | Return `low = 0` |

---

## 10. Complexity Analysis

- **Time Complexity:**
  - **Best Case:** $O(1)$ if the array has only 1 element.
  - **Average Case:** $O(\log n)$ as we halve the search space at each iteration.
  - **Worst Case:** $O(\log n)$ when the peak is at either extreme end of the array.
- **Space Complexity:**
  - $O(1)$ auxiliary space since we only store two pointer variables.

---

## 11. Common Mistakes

1. **Out of bounds access on `mid + 1`:**
   Using `low <= high` with `nums[mid + 1]`. When `low == high`, `mid` equals `high`, and `nums[mid + 1]` triggers an out-of-bounds index access.
2. **Incorrect slope analysis:**
   Confusing the direction. If `nums[mid] < nums[mid+1]`, it means the curve is rising, so the peak is to the right.
3. **Thinking Binary Search requires sorted arrays:**
   Candidates often get stuck trying to sort the array first. Sorting takes $O(n \log n)$, which defeats the purpose. The problem must be solved directly in $O(\log n)$ time.

---

## 12. Edge Cases

| Edge Case | Input | Expected Output | Rationale |
| :--- | :--- | :--- | :--- |
| **Single Element** | `nums = [1]` | `0` | Loop does not run. Returns `0`. Correct since neighbors are $-\infty$. |
| **Strictly Increasing** | `nums = [1, 2, 3]` | `2` | Peak is the last element. |
| **Strictly Decreasing** | `nums = [3, 2, 1]` | `0` | Peak is the first element. |
| **V-Shape (Valley)** | `nums = [5, 2, 6]` | `0` or `2` | Both `5` and `6` are valid peaks. Pointers will converge to one of them. |

---

## 13. Interview Explanation

> "To find a peak element in an unsorted array in $O(\log n)$ time, we use a binary search approach based on slopes. Even though the array is not sorted, any element is either part of an ascending slope or a descending slope. 
> We check the midpoint `mid` and its right neighbor `mid + 1`. If `nums[mid] < nums[mid + 1]`, we are on a rising slope, meaning there must be a peak to the right of `mid`. We shift our search space to `low = mid + 1`. 
> Otherwise, if `nums[mid] > nums[mid + 1]`, we are on a falling slope, meaning a peak must exist at `mid` or to its left. We shift our search space to `high = mid`. 
> Our search space halves each time, and when `low` and `high` converge, they point to a peak. This takes $O(\log n)$ time and $O(1)$ space."

---

## 14. Follow-up Questions

1. **What if duplicate adjacent elements are allowed?**
   - *Answer:* If duplicates are allowed (e.g. `[1, 1, 1, 2, 1, 1]`), we cannot determine the slope direction at `mid` if `nums[mid] == nums[mid+1]`. We would have to check both sides or fall back to linear scan, making the worst-case time complexity $O(n)$.
2. **How would you find a peak in a 2D matrix?**
   - *Answer:* Use 2D peak finding. We can select the middle column, find the maximum element in that column, and compare it with its left and right neighbors. This reduces the problem to a 1D peak search, running in $O(r \log c)$ time, where $r$ is rows and $c$ is columns.

---

## 15. Variations

1. **Find Minimum in Rotated Sorted Array (LeetCode 153):** Similar logic of comparing elements to find the unsorted/inflection point.
2. **Find Peak Index in a Mountain Array (LeetCode 852):** A special case of this problem where the array is guaranteed to be strictly increasing up to a peak and then strictly decreasing.

---

## 16. Revision Notes

- **Loop Condition:** `low < high`
- **Decision:**
  - If `nums[mid] < nums[mid+1]` $\rightarrow$ `low = mid + 1`
  - Else $\rightarrow$ `high = mid`
- **Mnemonic:** *"Follow the rising slope; it must lead to a peak."*

---

## 17. Memorization Version

```cpp
int findPeakElement(vector<int>& nums) {
    int low = 0, high = nums.size() - 1;
    while (low < high) {
        int mid = low + (high - low) / 2;
        if (nums[mid] < nums[mid + 1]) {
            low = mid + 1;
        } else {
            high = mid;
        }
    }
    return low;
}
```
*Time:* $O(\log n)$, *Space:* $O(1)$. Compare `mid` and `mid + 1` to find ascending/descending slope. Loop condition is `low < high`.
