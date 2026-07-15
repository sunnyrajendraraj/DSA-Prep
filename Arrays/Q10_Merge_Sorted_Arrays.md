# Merge Two Sorted Arrays

---

## 1. Problem Statement

Given two sorted integer arrays `nums1` and `nums2` of size `m` and `n` respectively, merge them into a single sorted array.

There are two primary variants of this problem:
1. **With Extra Space:** Merge `nums1` and `nums2` into a new sorted array of size `m + n`.
2. **In-place (LeetCode style):** `nums1` has a size of `m + n`, where the first `m` elements denote the elements that should be merged, and the last `n` elements are set to `0` and should be ignored. `nums2` has a size of `n`. Merge `nums2` into `nums1` in-place.
3. **In-place (Without Extra Space - Shell/Gap Method):** `nums1` of size `m` and `nums2` of size `n` are individual arrays. Merge them such that the first `m` smallest elements are in `nums1` (sorted) and the remaining `n` elements are in `nums2` (sorted), without using any extra space.

We will focus on the **LeetCode style In-place Merge** as the main optimal version and the **Gap Method** as the advanced in-place merge without extra space.

### Input
- `nums1` of size `m + n` (first `m` elements are sorted).
- `nums2` of size `n` (sorted).

### Output
- `nums1` is modified in-place to contain all `m + n` elements in sorted order.

### Constraints
- $nums1.length == m + n$
- $nums2.length == n$
- $0 \le m, n \le 200$
- $1 \le m + n \le 200$
- $-10^9 \le nums1[i], nums2[j] \le 10^9$

---

## 2. Interview Clarification

Before writing any code, clarify these points with the interviewer:
1. **Do we merge into a new array or modify one of the inputs?**
   - *Answer:* Modify `nums1` in-place since it contains extra padding of size `n` at the end.
2. **Can there be duplicate values between the two arrays?**
   - *Answer:* Yes, duplicates are allowed and should be preserved.
3. **What if one of the arrays is empty?**
   - *Answer:* If `nums2` is empty ($n = 0$), `nums1` is already sorted and we do not need to do anything. If $m = 0$, copy `nums2` to `nums1`.

---

## 3. Thinking Process

Let's think aloud:
* **Initial thoughts:** We have two sorted arrays. We need to merge them.
* **Brute Force (Concatenate and Sort):**
  - Simply copy all elements of `nums2` into the tail of `nums1`.
  - Sort the entire `nums1` array of size `m + n`.
  - Time Complexity: $O((m+n) \log(m+n))$ due to sorting. Space Complexity: $O(1)$ if in-place sort is used.
* **Optimal (Two Pointers - Back to Front):**
  - Standard merge sort uses two pointers starting from the beginning (index 0). But if we start from the beginning of `nums1`, we might overwrite elements of `nums1` that haven't been merged yet, requiring us to copy `nums1` first ($O(m)$ extra space).
  - *Observation:* The end of `nums1` contains empty padding (zeros). Why not start merging from the **back**?
  - Place pointer `i = m - 1` at the end of the initialized part of `nums1`.
  - Place pointer `j = n - 1` at the end of `nums2`.
  - Place pointer `write_idx = m + n - 1` at the end of the total space in `nums1`.
  - Compare `nums1[i]` and `nums2[j]`. Whichever is larger gets placed at `nums1[write_idx]`.
  - Decrement the pointer of the larger element and `write_idx`.
  - If we run out of elements in `nums1` (i.e., `i < 0`), copy the remaining elements of `nums2` to the front of `nums1`.
  - If we run out of elements in `nums2` (i.e., `j < 0`), the remaining elements in `nums1` are already in their correct places.
* **Advanced Variant: In-place Merge (No Extra Space - Gap Method):**
  - Suppose we have `nums1` of size `m` and `nums2` of size `n`, and we cannot resize them or use any extra array.
  - We can use the **Gap Method** (based on Shell Sort).
  - Let $gap = \lceil (m+n)/2 \rceil$.
  - We compare elements at a distance of $gap$. The first pointer `p1` starts at $0$, and the second pointer `p2` starts at $gap$.
  - If `p2` falls within `nums1` ($p2 < m$), we compare `nums1[p1]` and `nums1[p2]`. If `nums1[p1] > nums1[p2]`, we swap.
  - If `p2` spans across both arrays ($p1 < m$ and $p2 \ge m$), we compare `nums1[p1]` and `nums2[p2 - m]`. If `nums1[p1] > nums2[p2 - m]`, we swap.
  - If both pointers are in `nums2` ($p1 \ge m$), we compare `nums2[p1 - m]` and `nums2[p2 - m]`. Swap if necessary.
  - Increment `p1` and `p2` by 1 until `p2` reaches the end of the second array.
  - After completing one pass, update the gap: $gap = \lceil gap / 2 \rceil$.
  - Repeat the process until $gap == 0$ (or after the pass where $gap == 1$ is finished).
  - Time Complexity: $O((m+n) \log(m+n))$, Space Complexity: $O(1)$. This is highly optimal for in-place merging of distinct arrays without auxiliary space.

---

## 4. Pattern Recognition

This problem belongs to the **Two Pointers** and **Divide and Conquer (Sorting)** patterns.
* **Why this topic?** Merging two sorted sequences is a classic linear scan problem.
* **Which pattern?**
  - **Backwards Two Pointers:** Eliminates overwriting issues when reusing space.
  - **Shell Sort / Gap Method:** Merging distinct arrays in-place with fractional steps.

---

## 5. Brute Force Solution

Concatenate the elements of the second array into the end of the first array, then sort.

### Algorithm
1. Loop $i$ from $0$ to $n-1$.
2. Copy `nums2[i]` to `nums1[m + i]`.
3. Sort `nums1` from index $0$ to $m + n - 1$.

### Dry Run
Input: `nums1 = [1, 2, 3, 0, 0, 0]`, $m = 3$, `nums2 = [2, 5, 6]`, $n = 3$.
1. Copying:
   - `nums1[3] = nums2[0]` $\rightarrow$ `nums1[3] = 2`
   - `nums1[4] = nums2[1]` $\rightarrow$ `nums1[4] = 5`
   - `nums1[5] = nums2[2]` $\rightarrow$ `nums1[5] = 6`
   - Array becomes: `nums1 = [1, 2, 3, 2, 5, 6]`
2. Sorting:
   - Sort `[1, 2, 3, 2, 5, 6]` $\rightarrow$ `[1, 2, 2, 3, 5, 6]`.

### Complexity Analysis
- **Time Complexity:** $O((m+n) \log(m+n))$ — Due to sorting the merged array of size $m+n$.
- **Space Complexity:** $O(1)$ — If using in-place sort (like heap sort or introsort).

### Pros/Cons
- **Pros:** Simple, few lines of code.
- **Cons:** Does not take advantage of the fact that the individual arrays are already sorted. Slow for large values.

### C++17 Code
```cpp
#include <vector>
#include <algorithm>

class Solution {
public:
    void merge(std::vector<int>& nums1, int m, std::vector<int>& nums2, int n) {
        // Concatenate nums2 into nums1's empty space
        for (int i = 0; i < n; ++i) {
            nums1[m + i] = nums2[i];
        }
        
        // Sort the entire array
        std::sort(nums1.begin(), nums1.end());
    }
};
```

---

## 6. Better Solution (Advanced: Gap Method)

If we want to merge two separate arrays `nums1` (size $m$) and `nums2` (size $n$) in-place *without* resizing either array (so no extra padding is provided).

### Algorithm
1. Initialize $gap = \lceil (m + n) / 2.0 \rceil$.
2. While $gap > 0$:
   - Initialize pointers: `left = 0`, `right = gap`.
   - While `right < m + n`:
     - Compare and swap elements at `left` and `right`.
     - Handle indices:
       - If `left < m` and `right < m`: compare `nums1[left]` and `nums1[right]`.
       - If `left < m` and `right >= m`: compare `nums1[left]` and `nums2[right - m]`.
       - If `left >= m` and `right >= m`: compare `nums2[left - m]` and `nums2[right - m]`.
     - Increment `left` and `right` by 1.
   - Update $gap$: If $gap == 1$, break. Otherwise, $gap = \lceil gap / 2.0 \rceil$.

### Complexity Analysis
- **Time Complexity:** $O((m+n) \log(m+n))$ — $O(\log(m+n))$ gap steps, each step runs a linear scan of size $m+n$.
- **Space Complexity:** $O(1)$ — In-place element swapping.

### C++17 Code
```cpp
#include <vector>
#include <cmath>
#include <algorithm>

class GapMergeSolution {
private:
    void swapIfGreater(std::vector<int>& nums1, std::vector<int>& nums2, int idx1, int idx2, int m) {
        if (idx1 < m && idx2 < m) {
            if (nums1[idx1] > nums1[idx2]) {
                std::swap(nums1[idx1], nums1[idx2]);
            }
        } else if (idx1 < m && idx2 >= m) {
            if (nums1[idx1] > nums2[idx2 - m]) {
                std::swap(nums1[idx1], nums2[idx2 - m]);
            }
        } else {
            if (nums2[idx1 - m] > nums2[idx2 - m]) {
                std::swap(nums2[idx1 - m], nums2[idx2 - m]);
            }
        }
    }

public:
    void merge(std::vector<int>& nums1, int m, std::vector<int>& nums2, int n) {
        int len = m + n;
        int gap = std::ceil(len / 2.0);
        
        while (gap > 0) {
            int left = 0;
            int right = gap;
            
            while (right < len) {
                swapIfGreater(nums1, nums2, left, right, m);
                left++;
                right++;
            }
            
            if (gap == 1) break;
            gap = std::ceil(gap / 2.0);
        }
    }
};
```

---

## 7. Optimal Solution (LeetCode Standard: Backwards Merge)

Since the target array `nums1` has pre-allocated space at the end, we can merge elements from back to front in linear time and constant space.

### Algorithm
1. Initialize pointers:
   - `i = m - 1` (last active element in `nums1`).
   - `j = n - 1` (last element in `nums2`).
   - `write_idx = m + n - 1` (last position of `nums1`).
2. While `j >= 0` (as long as there are elements left in `nums2` to merge):
   - If `i >= 0` and `nums1[i] > nums2[j]`:
     - Set `nums1[write_idx] = nums1[i]`.
     - Decrement `i`.
   - Else:
     - Set `nums1[write_idx] = nums2[j]`.
     - Decrement `j`.
   - Decrement `write_idx`.
3. If `i < 0` and `j >= 0`, the loop will copy the remaining elements of `nums2` to the front of `nums1`. If `j < 0` first, the remaining elements of `nums1` are already in their correct places.

### Dry Run
Input: `nums1 = [1, 3, 5, 0, 0, 0]`, $m = 3$, `nums2 = [2, 4, 6]`, $n = 3$.
- `i = 2` (val: 5), `j = 2` (val: 6), `write_idx = 5`.
- Compare `5` and `6`. $6 > 5 \rightarrow$ `nums1[5] = 6`. `j` becomes 1, `write_idx` becomes 4.
- `i = 2` (val: 5), `j = 1` (val: 4). Compare `5` and `4`. $5 > 4 \rightarrow$ `nums1[4] = 5`. `i` becomes 1, `write_idx` becomes 3.
- `i = 1` (val: 3), `j = 1` (val: 4). Compare `3` and `4`. $4 > 3 \rightarrow$ `nums1[3] = 4`. `j` becomes 0, `write_idx` becomes 2.
- `i = 1` (val: 3), `j = 0` (val: 2). Compare `3` and `2`. $3 > 2 \rightarrow$ `nums1[2] = 3`. `i` becomes 0, `write_idx` becomes 1.
- `i = 0` (val: 1), `j = 0` (val: 2). Compare `1` and `2`. $2 > 1 \rightarrow$ `nums1[1] = 2`. `j` becomes -1, `write_idx` becomes 0.
- Loop terminates because `j < 0`. Array is `[1, 2, 3, 4, 5, 6]`.

### C++17 Code
```cpp
#include <vector>

class Solution {
public:
    void merge(std::vector<int>& nums1, int m, std::vector<int>& nums2, int n) {
        // Pointers for nums1, nums2, and write target
        int i = m - 1;
        int j = n - 1;
        int write_idx = m + n - 1;
        
        // Merge starting from the back
        while (j >= 0) {
            // If nums1 has elements left and its current element is larger
            if (i >= 0 && nums1[i] > nums2[j]) {
                nums1[write_idx] = nums1[i];
                i--;
            } else {
                nums1[write_idx] = nums2[j];
                j--;
            }
            write_idx--;
        }
    }
};
```

---

## 8. Code Walkthrough

* `int i = m - 1, j = n - 1, write_idx = m + n - 1;`: Sets up indices. `write_idx` represents the location where the next largest element will be placed.
* `while (j >= 0)`: The loop only needs to run until we have placed all elements of `nums2`. If `nums2` is exhausted first, the remaining elements in `nums1` are already sorted and in their correct final positions.
* `if (i >= 0 && nums1[i] > nums2[j])`: If `nums1` has unplaced elements and its current element is larger than the current element in `nums2`, we write `nums1[i]` to `write_idx` and decrement `i`.
* `else`: Otherwise, we write `nums2[j]` and decrement `j`.

---

## 9. Dry Run

Input: `nums1 = [0]`, $m = 0$, `nums2 = [1]`, $n = 1$.

| Step | Pointer `i` | Pointer `j` | `write_idx` | Comparison | Action | `nums1` State |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Start**| -1 | 0 | 0 | — | — | `[0]` |
| **1** | -1 | 0 | 0 | `i >= 0` is False | `nums1[0] = nums2[0] = 1`, `j--` | `[1]` |
| **End** | -1 | -1 | -1 | Loop Ends (`j < 0`) | Done | `[1]` |

---

## 10. Complexity Analysis

| Approach | Time Complexity | Space Complexity | Notes |
| :--- | :--- | :--- | :--- |
| **Brute Force** | $O((m+n) \log(m+n))$ | $O(1)$ | Simple copy + sort. |
| **Gap Method** | $O((m+n) \log(m+n))$ | $O(1)$ | Used when arrays are separate and cannot be resized. |
| **Optimal (Backwards)**| $O(m + n)$ | $O(1)$ | Linear time, in-place, zero auxiliary space. |

* **Time Complexity (Optimal):** $O(m + n)$ since we make at most $m + n$ comparisons and writes in a single pass.
* **Space Complexity (Optimal):** $O(1)$ auxiliary space because we reuse the allocated padding in `nums1`.

---

## 11. Common Mistakes

1. **Merging from front:** Trying to merge from index 0. This overwrites the existing elements in `nums1` before they can be compared, requiring a copy of `nums1`.
2. **Incorrect while loop condition:** Writing `while (i >= 0 && j >= 0)` and forgetting to flush the remaining elements of `nums2` when `nums1` runs out first.
3. **Array index bounds:** Accessing `nums1[i]` when `i < 0`, leading to segmentation faults.

---

## 12. Edge Cases

| Edge Case | Input Example | Expected Output | Why it matters |
| :--- | :--- | :--- | :--- |
| **$m = 0$ (nums1 empty)** | `nums1=[0]`, $m=0$, `nums2=[5]`, $n=1$ | `[5]` | Verifies copy of `nums2` when `nums1` has no initial elements. |
| **$n = 0$ (nums2 empty)** | `nums1=[1]`, $m=1$, `nums2=[]`, $n=0$ | `[1]` | Checks early exit when `nums2` has nothing to merge. |
| **All elements in `nums2` smaller**| `nums1=[4,5,6,0,0,0]`, `nums2=[1,2,3]`| `[1,2,3,4,5,6]`| `i` runs out first, remaining `nums2` copied. |

---

## 13. Interview Explanation

"To merge two sorted arrays where the first array has extra space at the end to hold the result, a naive approach would be to append the second array to the first and then sort it. This takes $O((m+n) \log(m+n))$ time.
However, we can optimize this to $O(m+n)$ time and $O(1)$ auxiliary space by merging from the back. Since the end of `nums1` has empty spaces, we place two pointers at the ends of the initialized parts of both arrays and a write pointer at the end of the total space. By comparing elements from the back and copying the larger one to the write position, we guarantee that we never overwrite any elements in `nums1` that haven't been processed yet. If `nums1` is exhausted first, we copy the remaining elements of `nums2` into place."

---

## 14. Follow-up Questions

1. **What if the two arrays are separate, cannot be modified, and we must return a new array?**
   * *Answer:* We use the standard two-pointer approach from the front, copying the smaller element into a new array. This takes $O(m+n)$ time and $O(m+n)$ space.
2. **What is the mathematical justification of the Gap Method?**
   * *Answer:* The Gap Method is a variant of Shell Sort. By comparing elements at a distance of $gap$ and reducing the gap by half in each pass, we keep the elements partially sorted. When the gap becomes 1, it performs a final bubble/insertion pass on almost sorted data, which is highly efficient.

---

## 15. Variations

1. **Merge Intervals:** Merge overlapping intervals.
   - *Difference:* Requires sorting the intervals based on start times and then merging adjacent intervals.
2. **Intersection of Two Sorted Arrays:** Find common elements in two sorted arrays.
   - *Difference:* Instead of merging, we move pointers inward and only save elements when they match.

---

## 16. Revision Notes

* **Trick:** Merge from right to left (back to front) to avoid overwriting.
* **Exit condition:** The loop only needs to check `while (j >= 0)` because once `nums2` is finished, `nums1` elements are already in their correct places.
* **Shell Sort Gap formula:** $gap = \lceil gap / 2 \rceil$.

---

## 17. Memorization Version

```cpp
// Backwards Merge Sorted Arrays
int i = m - 1, j = n - 1, write_idx = m + n - 1;
while (j >= 0) {
    if (i >= 0 && nums1[i] > nums2[j]) {
        nums1[write_idx--] = nums1[i--];
    } else {
        nums1[write_idx--] = nums2[j--];
    }
}
```
* **Time:** $O(m+n)$, **Space:** $O(1)$ in-place.
