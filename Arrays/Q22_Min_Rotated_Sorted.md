# Find Minimum in Rotated Sorted Array

## 1. Problem Statement

Suppose an array of length $N$ sorted in ascending order is rotated between $1$ and $N$ times. For example, the array `nums = [0, 1, 2, 4, 5, 6, 7]` might become:
- `[4, 5, 6, 7, 0, 1, 2]` if it was rotated 4 times.
- `[0, 1, 2, 4, 5, 6, 7]` if it was rotated 7 times.

Given the sorted rotated array `nums` of **unique** elements, return the *minimum element* of this array.

### Examples
- **Input:** `nums = [3, 4, 5, 1, 2]`
- **Output:** 1
  *(Explanation: The original array was `[1, 2, 3, 4, 5]` rotated 3 times.)*

- **Input:** `nums = [4, 5, 6, 7, 0, 1, 2]`
- **Output:** 0
  *(Explanation: The minimum element is 0.)*

- **Input:** `nums = [11, 13, 15, 17]`
- **Output:** 11
  *(Explanation: The array was rotated 4 times (which is equivalent to 0 rotations).)*

### Constraints
- $1 \le N \le 5000$
- $-5000 \le nums[i] \le 5000$
- All integers in `nums` are **unique**.

---

## 2. Interview Clarification Questions

Before writing any code, a candidate should clarify:
1. **Q:** Can the array contain duplicate elements?
   * **A:** No, assume all elements are unique. *(Note: If duplicates exist, it is a different variation of the problem, LeetCode Hard).*
2. **Q:** Is it possible that the array is not rotated at all?
   * **A:** Yes, a rotation of $N$ times results in the original sorted array.
3. **Q:** What is the expected time complexity?
   * **A:** $O(\log N)$ using binary search. A linear search of $O(N)$ is a brute force solution.

---

## 3. Thinking Process

### Step 1: First Instinct (Brute Force)
Since the minimum element is present in the array, we can simply scan the entire array from index $0$ to $N-1$ and keep track of the minimum element found.
- Initialize `minVal = nums[0]`.
- Loop through all elements, if `nums[i] < minVal`, update `minVal = nums[i]`.
- This runs in $O(N)$ time and requires $O(1)$ space.

### Step 2: Binary Search (Optimal Insight)
Since the array is sorted (but rotated), we should look for a logarithmic time solution $O(\log N)$. This suggests **Binary Search**.

In a rotated sorted array:
- If we divide the array in half at index `mid`, **at least one of the halves will always be sorted**.
- We can determine which half is sorted by comparing the boundary elements.
- Let's compare `nums[mid]` with `nums[high]`:
  - **Case 1:** `nums[mid] > nums[high]`
    - Since `nums[mid]` is larger than the element at the end of the range, the rotation pivot (and therefore the minimum element) must lie in the **right half** of the array.
    - Thus, we can safely eliminate the left half: `low = mid + 1`.
  - **Case 2:** `nums[mid] < nums[high]`
    - Since the middle element is smaller than the rightmost element, the right half from `mid` to `high` is sorted in ascending order.
    - The minimum element must be in the **left half** (which includes `mid` itself, as `nums[mid]` could be the minimum).
    - Thus, we can safely eliminate the right half (excluding `mid`'s left side but keeping `mid`): `high = mid`.
- We repeat this division until `low == high`. At this point, `nums[low]` will be the minimum element.

---

## 4. Pattern Recognition

This problem fits the **Modified Binary Search** pattern.
- **Why?** The input array is sorted (possessing monotonicity) but partitioned by a rotation pivot. Because we can determine which half is sorted by checking boundary elements in $O(1)$ time, we can discard half of the search space at each step.

---

## 5. Brute Force Solution

### Algorithm
1. Initialize `minVal = nums[0]`.
2. Iterate through the array starting from index 1.
3. Update `minVal = min(minVal, nums[i])`.
4. Return `minVal`.

### Complexity
- **Time Complexity:** $O(N)$ because we inspect all $N$ elements.
- **Space Complexity:** $O(1)$ auxiliary space.

### Pros and Cons
- **Pros:** Extremely simple to implement; works even if there are duplicates.
- **Cons:** Inefficient; fails to exploit the sorted structure of the array.

### C++17 Code
```cpp
#include <vector>
#include <algorithm>

int findMinBrute(const std::vector<int>& nums) {
    int minVal = nums[0];
    for (int num : nums) {
        minVal = std::min(minVal, num);
    }
    return minVal;
}
```

---

## 6. Better Solution

There is no intermediate "better" solution for this problem. The brute force solution is $O(N)$ and the optimal solution is $O(\log N)$. We jump directly to the optimal solution.

---

## 7. Optimal Solution (Binary Search)

### Algorithm
1. Initialize two pointers: `low = 0` and `high = nums.size() - 1`.
2. While `low < high`:
   - Calculate the midpoint: `mid = low + (high - low) / 2`.
   - Compare `nums[mid]` with `nums[high]`:
     - If `nums[mid] > nums[high]`, set `low = mid + 1` (the minimum is in the right half).
     - If `nums[mid] < nums[high]`, set `high = mid` (the minimum is at `mid` or in the left half).
3. Return `nums[low]`.

### C++17 Code

```cpp
#include <vector>

int findMin(const std::vector<int>& nums) {
    int low = 0;
    int high = nums.size() - 1;
    
    // Binary search loop
    while (low < high) {
        int mid = low + (high - low) / 2;
        
        // If mid element is greater than the high element,
        // the minimum element must be in the right unsorted subarray.
        if (nums[mid] > nums[high]) {
            low = mid + 1;
        } 
        // Otherwise, the right subarray is sorted, meaning the minimum
        // is in the left subarray (including mid itself).
        else {
            high = mid;
        }
    }
    
    // low and high converge to the index of the minimum element
    return nums[low];
}
```

---

## 8. Code Walkthrough

1. **Pointers setup:** `low` is set to `0` and `high` is set to `nums.size() - 1` representing the active search boundaries.
2. **Loop Condition:** `while (low < high)` runs until the search space shrinks to a single element. Note we use `<` and not `<=` because when `low == high`, we have identified the minimum index.
3. **Midpoint calculation:** `low + (high - low) / 2` prevents integer overflow.
4. **Partition check:**
   - `if (nums[mid] > nums[high])`: The value at `mid` is too large to be part of a sorted segment extending to `high`. This indicates the inflection point (pivot) lies to the right of `mid`. We shift `low` to `mid + 1`.
   - `else`: `nums[mid] <= nums[high]`. This means the subarray from `mid` to `high` is sorted. The smallest element in this segment is `nums[mid]`. Any element to the right of `mid` is larger than `nums[mid]`, so they cannot be the absolute minimum. Thus, we discard everything to the right of `mid` by setting `high = mid`.
5. **Return:** When `low == high`, the loop terminates. `nums[low]` points to the minimum element.

---

## 9. Dry Run

Input: `nums = [4, 5, 6, 7, 0, 1, 2]`

### Initialization:
- `low = 0`, `high = 6`

### Iterations:

| Step | `low` | `high` | `mid` | `nums[mid]` | `nums[high]` | Condition | Action |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **1** | 0 | 6 | 3 | `nums[3] = 7` | `nums[6] = 2` | `7 > 2` (True) | `low = 3 + 1 = 4` |
| **2** | 4 | 6 | 5 | `nums[5] = 1` | `nums[6] = 2` | `1 > 2` (False) | `high = 5` |
| **3** | 4 | 5 | 4 | `nums[4] = 0` | `nums[5] = 1` | `0 > 1` (False) | `high = 4` |
| **4** | 4 | 4 | - | - | - | `low < high` is False | Exit loop |

Return: `nums[4] = 0`.

---

## 10. Complexity Analysis

- **Time Complexity:** $O(\log N)$. At each iteration, we compare the middle element and reduce the search space by half.
- **Space Complexity:** $O(1)$ as we only use a few pointer variables.

---

## 11. Common Mistakes

1. **Incorrect Loop Condition:** Using `while (low <= high)` instead of `while (low < high)`. This can lead to an infinite loop because `high = mid` will keep re-evaluating the same elements.
2. **Incorrectly updating `high`:** Setting `high = mid - 1` instead of `high = mid`. If `nums[mid]` is the actual minimum, setting `high = mid - 1` excludes it from the search space, yielding an incorrect answer.
3. **Comparing `nums[mid]` with `nums[low]`:** It is safer to compare `nums[mid]` with `nums[high]`. Comparing with `nums[low]` requires extra handling for when the array is already sorted (unrotated).

---

## 12. Edge Cases

| Edge Case | Input Example | Why it matters | Correct Output |
| :--- | :--- | :--- | :--- |
| **Not Rotated (Fully Sorted)** | `[1, 2, 3]` | `nums[mid] < nums[high]` always matches, pushing `high` left until `low == 0`. | `1` |
| **Single Element** | `[5]` | Loop condition `low < high` (0 < 0) is false immediately. | `5` |
| **Two Elements** | `[2, 1]` | `mid` is 0. `nums[0] > nums[1]`, shifts `low` to 1. | `1` |
| **Two Elements (Sorted)** | `[1, 2]` | `mid` is 0. `nums[0] < nums[1]`, shifts `high` to 0. | `1` |

---

## 13. Interview Explanation

"To find the minimum element in a rotated sorted array of unique elements, the brute force approach is to scan the array linearly, which takes $O(N)$ time. However, because the array was originally sorted, we can use binary search to locate the minimum in $O(\log N)$ time.

The key observation is that if we split the array in half, one of the halves will always be sorted. We compare the middle element `nums[mid]` with the rightmost element `nums[high]`. 
If `nums[mid]` is greater than `nums[high]`, it tells us that the rotation point (the drop-off point where the minimum element resides) is in the right half. Thus, we search the right half by setting `low = mid + 1`. 
Otherwise, if `nums[mid]` is less than or equal to `nums[high]`, the right half is sorted, meaning the minimum must lie in the left half (including `mid` itself). So we search the left half by setting `high = mid`. We repeat this until our pointers converge."

---

## 14. Follow-up Questions

1. **Q: What if the array contains duplicates? (e.g., `[1, 0, 1, 1, 1]`)**
   * *A:* If duplicates are allowed, we cannot decide which half to discard if `nums[mid] == nums[high]`. In this specific case, we can only safely decrement `high` by 1 (`high--`) to shrink the search space, which degrades the worst-case time complexity to $O(N)$.
2. **Q: How can we find how many times the sorted array was rotated?**
   * *A:* The number of rotations is equal to the index of the minimum element. Once we find the minimum element, its index tells us the rotation count.

---

## 15. Variations

1. **Search in Rotated Sorted Array:**
   - Instead of finding the minimum, search for a target element.
2. **Find Minimum in Rotated Sorted Array with Duplicates:**
   - Handles the case where `nums[mid] == nums[high]` by decrementing the `high` pointer.

---

## 16. Revision Notes

- **Pivot Property:** The minimum element is the only element that is smaller than its left neighbor.
- **Mid vs High:** 
  - `nums[mid] > nums[high] \to` Go Right (`low = mid + 1`)
  - `nums[mid] <= nums[high] \to` Go Left (`high = mid`)
- **Complexity:** $O(\log N)$ Time, $O(1)$ Space.

---

## 17. Memorization Version

```cpp
// 1-min Review Version
int findMin(vector<int>& nums) {
    int low = 0, high = nums.size() - 1;
    while (low < high) {
        int mid = low + (high - low) / 2;
        if (nums[mid] > nums[high]) {
            low = mid + 1;
        } else {
            high = mid;
        }
    }
    return nums[low];
}
```
*Complexity:* $O(\log N)$ Time, $O(1)$ Space.
