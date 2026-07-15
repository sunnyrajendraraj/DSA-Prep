# Sort Array of 0s, 1s, and 2s (Dutch National Flag)

## 1. Problem Statement

Given an array `nums` with `n` objects colored red, white, or blue, sort them **in-place** so that objects of the same color are adjacent, with the colors in the order red, white, and blue.

We will use the integers `0`, `1`, and `2` to represent the color red, white, and blue, respectively.

You must solve this problem without using the library's sort function.

### Input
* `nums`: A vector of integers containing only `0`, `1`, and `2` where $1 \le \text{nums.length} \le 300$.

### Output
* The array is sorted in-place. No value is returned.

### Constraints
* `nums[i]` is either `0`, `1`, or `2`.
* Could you come up with a one-pass algorithm using only constant extra space?

---

## 2. Interview Clarification

Before writing any code, clarify these points with your interviewer:
1. **Can we use counting sort?**
   * *Answer*: Counting sort is a good two-pass solution, but we should aim for a one-pass solution.
2. **Is it acceptable to modify the array in-place?**
   * *Answer*: Yes, in-place sorting is required.
3. **What is the size limit of the array?**
   * *Answer*: The array size is small ($N \le 300$ in basic constraints), but your solution should scale efficiently to larger arrays.

---

## 3. Thinking Process

### First Instinct (Brute Force)
* Sort the array using a standard comparison-based sorting algorithm (e.g., QuickSort or MergeSort).
* This takes $O(n \log n)$ time and does not take advantage of the fact that there are only three unique values.

### Better Instinct (Counting Sort - Two Passes)
* Since there are only three unique numbers (`0`, `1`, and `2`), we can count their frequencies.
* **Pass 1**: Count the occurrences of `0`s, `1`s, and `2`s in the array.
* **Pass 2**: Overwrite the array with the corresponding number of `0`s, then `1`s, and finally `2`s.
* This takes $O(n)$ time (two passes) and $O(1)$ space.

### Optimal Insight (Dutch National Flag Algorithm - One Pass)
* Can we do this in a single pass? Yes, using three pointers: `low`, `mid`, and `high`.
* **Concept**: Divide the array into four regions:
  1. `nums[0 ... low-1]` contains only `0`s.
  2. `nums[low ... mid-1]` contains only `1`s.
  3. `nums[mid ... high]` is the unsorted/unexamined region.
  4. `nums[high+1 ... n-1]` contains only `2`s.
* We initialize `low = 0`, `mid = 0`, and `high = n - 1`.
* While `mid <= high`, we inspect `nums[mid]`:
  * **Case 1**: `nums[mid] == 0`
    * Swap `nums[mid]` and `nums[low]`.
    * Increment both `low` and `mid`. (Since the element swapped from `low` is guaranteed to be a `1`, it is safe to advance `mid`).
  * **Case 2**: `nums[mid] == 1`
    * It is already in the correct place (middle region).
    * Increment `mid`.
  * **Case 3**: `nums[mid] == 2`
    * Swap `nums[mid]` and `nums[high]`.
    * Decrement `high`. Do NOT increment `mid` because the new element swapped from `high` has not been examined yet and could be a `0`, `1`, or `2`.
* This sorts the array in a single pass using $O(n)$ time and $O(1)$ auxiliary space.

---

## 4. Pattern Recognition

* **Topic**: Sorting / Three Pointers.
* **Pattern**: Dutch National Flag Partitioning.
* **Why this works**: By grouping elements from the ends towards the middle, we maintain invariants for each partition. Pointers act as boundary markers for each group.

---

## 5. Brute Force Solution

### Algorithm (Standard Sort)
* Call `std::sort` on the array.

### Complexity Analysis
* **Time Complexity**: $O(n \log n)$
* **Space Complexity**: $O(1)$ or $O(\log n)$ depending on the implementation of `std::sort`.

### C++17 Code
```cpp
#include <vector>
#include <algorithm>

class SolutionBrute {
public:
    void sortColors(std::vector<int>& nums) {
        std::sort(nums.begin(), nums.end());
    }
};
```

---

## 6. Better Solution

### Algorithm (Counting Sort)
1. Initialize counts: `count0 = 0`, `count1 = 0`, `count2 = 0`.
2. Traverse the array once to increment the counts of `0`, `1`, and `2`.
3. Overwrite the array:
   * Fill first `count0` positions with `0`.
   * Fill next `count1` positions with `1`.
   * Fill remaining `count2` positions with `2`.

### Complexity Analysis
* **Time Complexity**: $O(n)$ (two passes: one to count, one to overwrite).
* **Space Complexity**: $O(1)$ extra space.

### C++17 Code
```cpp
#include <vector>

class SolutionBetter {
public:
    void sortColors(std::vector<int>& nums) {
        int count0 = 0, count1 = 0, count2 = 0;
        
        // Pass 1: Count frequencies
        for (int num : nums) {
            if (num == 0) count0++;
            else if (num == 1) count1++;
            else if (num == 2) count2++;
        }
        
        // Pass 2: Overwrite values
        int idx = 0;
        while (count0 > 0) {
            nums[idx++] = 0;
            count0--;
        }
        while (count1 > 0) {
            nums[idx++] = 1;
            count1--;
        }
        while (count2 > 0) {
            nums[idx++] = 2;
            count2--;
        }
    }
};
```

---

## 7. Optimal Solution

### Algorithm (Dutch National Flag)
1. Initialize `low = 0`, `mid = 0`, `high = nums.size() - 1`.
2. While `mid <= high`:
   * If `nums[mid] == 0`:
     * `std::swap(nums[low], nums[mid])`
     * `low++`, `mid++`
   * Else if `nums[mid] == 1`:
     * `mid++`
   * Else (if `nums[mid] == 2`):
     * `std::swap(nums[mid], nums[high])`
     * `high--`

### Why this is Optimal
* It makes a single pass through the array.
* Only swaps elements when necessary.
* Does not require extra memory space beyond three index pointers.

### C++17 Code
```cpp
#include <vector>
#include <utility>

class SolutionOptimal {
public:
    void sortColors(std::vector<int>& nums) {
        int low = 0;
        int mid = 0;
        int high = static_cast<int>(nums.size()) - 1;
        
        while (mid <= high) {
            if (nums[mid] == 0) {
                std::swap(nums[low], nums[mid]);
                low++;
                mid++;
            } 
            else if (nums[mid] == 1) {
                mid++;
            } 
            else { // nums[mid] == 2
                std::swap(nums[mid], nums[high]);
                high--;
            }
        }
    }
};
```

---

## 8. Code Walkthrough

* **Line 7**: `int low = 0;`
  Pointer `low` marks the boundary where the next `0` should be placed.
* **Line 8**: `int mid = 0;`
  Pointer `mid` is the active scanner moving from left to right.
* **Line 9**: `int high = static_cast<int>(nums.size()) - 1;`
  Pointer `high` marks the boundary where the next `2` should be placed.
* **Line 11**: `while (mid <= high)`
  The loop runs as long as there are unexamined elements between `mid` and `high`.
* **Line 12-16**: `if (nums[mid] == 0)`
  We swap `nums[mid]` with `nums[low]`. Since we know that elements before `mid` (up to `low`) are already processed and must be `1`s (or equal to `mid`), the swapped element at `mid` is safe to be skipped, so both pointers increment.
* **Line 17-19**: `else if (nums[mid] == 1)`
  A `1` is in its correct relative position. We simply advance `mid`.
* **Line 20-23**: `else` (value is 2)
  We swap `nums[mid]` with `nums[high]`. Because the element swapped from `high` to `mid` is unknown and unexamined, we do NOT advance `mid`. We only decrement `high` to contract the unplaced `2` boundary.

---

## 9. Dry Run

Input: `nums = [2, 0, 2, 1, 1, 0]`
Initial state: `low = 0`, `mid = 0`, `high = 5`

| Step | `low` | `mid` | `high` | Array state | `nums[mid]` | Action |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **0** | 0 | 0 | 5 | `[2, 0, 2, 1, 1, 0]` | 2 | Swap `nums[mid]` & `nums[high]`. `high--` |
| **1** | 0 | 0 | 4 | `[0, 0, 2, 1, 1, 2]` | 0 | Swap `nums[low]` & `nums[mid]`. `low++`, `mid++` |
| **2** | 1 | 1 | 4 | `[0, 0, 2, 1, 1, 2]` | 0 | Swap `nums[low]` & `nums[mid]`. `low++`, `mid++` |
| **3** | 2 | 2 | 4 | `[0, 0, 2, 1, 1, 2]` | 2 | Swap `nums[mid]` & `nums[high]`. `high--` |
| **4** | 2 | 2 | 3 | `[0, 0, 1, 1, 2, 2]` | 1 | `nums[mid] == 1`. `mid++` |
| **5** | 2 | 3 | 3 | `[0, 0, 1, 1, 2, 2]` | 1 | `nums[mid] == 1`. `mid++` |
| **End** | 2 | 4 | 3 | `[0, 0, 1, 1, 2, 2]` | - | Loop terminates as `mid > high` ($4 > 3$) |

Output: `[0, 0, 1, 1, 2, 2]`

---

## 10. Complexity Analysis

| Approach | Time Complexity | Space Complexity | Notes |
| :--- | :--- | :--- | :--- |
| Brute Force (std::sort) | $O(n \log n)$ | $O(1)$ or $O(\log n)$ | General sorting |
| Counting Sort | $O(n)$ | $O(1)$ | Two passes over the array |
| **Dutch National Flag** | **$O(n)$** | **$O(1)$** | **Single pass, in-place swaps** |

---

## 11. Common Mistakes

1. **Incrementing `mid` when swapping with `high`**:
   * This is the most common bug. If you swap `nums[mid]` with `nums[high]`, the value that comes from `high` is unexamined. If you increment `mid`, you might skip a `0` or another `2` without placing it correctly.
2. **Loop condition `mid < high` instead of `mid <= high`**:
   * If you use `mid < high`, the element at the index `high` might not get sorted when `mid == high`. You must process the element when `mid == high`.

---

## 12. Edge Cases

| Edge Case | Example | Why it matters | Solution Behavior |
| :--- | :--- | :--- | :--- |
| All elements are identical | `[1, 1, 1]` | Pointers should traverse without infinite loop. | Handles correctly via `mid++` (Correct) |
| Already sorted | `[0, 1, 2]` | Standard sorted behavior. | No swaps, just traverses (Correct) |
| Reverse sorted | `[2, 1, 0]` | Maximum swaps required. | Swaps boundaries. (Correct) |
| Missing a category | `[0, 2, 0, 2]` | Missing `1`s in the array. | Low and High swap and group. (Correct) |

---

## 13. Interview Explanation

"To sort an array containing only `0`s, `1`s, and `2`s in-place in a single pass, we use the Dutch National Flag algorithm. This algorithm uses three pointers: `low`, `mid`, and `high`.

* `low` tracks the boundary where `0`s should end.
* `high` tracks the boundary where `2`s should start.
* `mid` scans the array from index 0 to `high`.

We start with `low = 0`, `mid = 0`, and `high = n - 1`. As we iterate:
1. If the current element `nums[mid]` is `0`, we swap it with `nums[low]` and increment both `low` and `mid`.
2. If it is `1`, it is already in the middle, so we simply increment `mid`.
3. If it is `2`, we swap it with `nums[high]` and decrement `high`, but we do not increment `mid` because the new element at `mid` needs to be processed.
The algorithm terminates when `mid` crosses `high`, sorting the array in $O(n)$ time and $O(1)$ space."

---

## 14. Follow-up Questions

1. **What if we have 4 colors instead of 3? (e.g., 0, 1, 2, 3)**
   * *Answer*: With 4 colors, we cannot use a simple 3-pointer partition. We would have to perform two rounds of partitioning (similar to QuickSort partitioning), or use counting sort which scales easily to $k$ colors with $O(k)$ extra space.
2. **Is this sorting algorithm stable?**
   * *Answer*: No, swapping elements across pointers makes the Dutch National Flag algorithm unstable.

---

## 15. Variations

1. **Move Zeros**: Move all zeros to the end of the array (Two pointers: `read` and `write`).
2. **Sort Array By Parity**: Group even numbers at the beginning and odd numbers at the end. (Simplified two-pointer partition).

---

## 16. Revision Notes

* **Pointer Roles**:
  * `0`s go to left of `low`.
  * `1`s stay between `low` and `mid-1`.
  * `2`s go to right of `high`.
* **Important rule**: Do NOT increment `mid` after swapping `nums[mid]` with `nums[high]`.
* **Complexity**: $O(n)$ Time, $O(1)$ Space.

---

## 17. Memorization Version

```cpp
// O(n) Time, O(1) Space. One Pass Dutch National Flag.
void sortColors(vector<int>& nums) {
    int low = 0, mid = 0, high = nums.size() - 1;
    while (mid <= high) {
        if (nums[mid] == 0) {
            swap(nums[low++], nums[mid++]);
        } else if (nums[mid] == 1) {
            mid++;
        } else {
            swap(nums[mid], nums[high--]);
        }
    }
}
```
