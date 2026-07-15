# Move All Zeroes to End

---

## 1. Problem Statement

Given an integer array `nums`, move all `0`'s to the end of it while maintaining the relative order of the non-zero elements. This operation must be performed in-place without making a copy of the array.

*   **Input:** An integer array `nums` of size $N$.
*   **Output:** The same array `nums` modified in-place.
*   **Constraints:**
    *   $1 \le N \le 10^4$
    *   $-2^{31} \le nums[i] \le 2^{31} - 1$
*   **Observations:**
    *   Minimize the total number of operations (writes).
    *   Maintain the original relative ordering of all non-zero elements.
    *   Perform this in $O(1)$ auxiliary space.

---

## 2. Interview Clarification

Before coding, clarify the following with the interviewer:
1.  **Should the function return anything?**
    *   *Answer:* Usually no, the function should return `void` since the modification must be in-place.
2.  **What if the array is already sorted or contains no zeroes?**
    *   *Answer:* The array should remain unchanged.
3.  **Does the relative order of non-zero elements matter?**
    *   *Answer:* Yes, maintaining the relative order of non-zero elements is a strict requirement.
4.  **Can we use extra memory?**
    *   *Answer:* The problem specifies doing it in-place with $O(1)$ auxiliary space.

---

## 3. Thinking Process

*   **First Instinct (Brute Force):**
    *   Create a temporary array of the same size.
    *   Iterate through the original array and copy all non-zero elements into the temporary array.
    *   Fill the remaining slots of the temporary array with zeroes.
    *   Copy all elements back to the original array.
    *   This takes $O(N)$ time and $O(N)$ space.
*   **Optimizing Space (Two-Pointer Approach):**
    *   How can we do this without extra space?
    *   We can use two pointers: `writePointer` and `readPointer`.
    *   `writePointer` points to the position where the next non-zero element should be written.
    *   `readPointer` iterates through the array.
    *   Whenever `readPointer` encounters a non-zero element:
        *   We copy `nums[readPointer]` to `nums[writePointer]`.
        *   Increment `writePointer`.
    *   After the loop finishes, all non-zero elements have been shifted to the front of the array.
    *   Then, we can simply fill all indices from `writePointer` to `N - 1` with `0`.
    *   This takes $O(N)$ time and $O(1)$ space.
*   **Further Optimization (One-Pass Swap):**
    *   Instead of writing non-zeroes first and then writing zeroes in a separate loop, can we do it in a single pass?
    *   Yes, we can swap elements instead of copying them!
    *   Let `i` be the iterator and `insertPos` (or `j`) track the position of the first zero.
    *   Iterate `i` through the array. Whenever `nums[i]` is non-zero:
        *   Swap `nums[i]` and `nums[insertPos]`.
        *   Increment `insertPos`.
    *   By swapping, the zeroes are naturally pushed to the back as non-zero elements are moved forward.
    *   This also takes $O(N)$ time and $O(1)$ space but minimizes writes.

---

## 4. Pattern Recognition

*   **Topic:** Arrays.
*   **Pattern:** **Two Pointers (Read/Write pointer or Fast/Slow pointer)**.
*   *Why this pattern?* We need to modify elements in-place while traversing. By keeping one pointer to track the insertion point of the valid elements and another to scan the array, we can safely overwrite or swap elements without losing information.

---

## 5. Brute Force Solution

### Algorithm
1.  Initialize a temporary array `temp` of size $N$ with all zeroes.
2.  Initialize a pointer `tempIdx = 0`.
3.  Iterate `i` from `0` to `N - 1`:
    *   If `nums[i] != 0`, assign `temp[tempIdx] = nums[i]` and increment `tempIdx`.
4.  Copy the elements of `temp` back into `nums`.

### Dry Run
Input: `nums = [0, 1, 0, 3, 12]`
1.  `temp = [0, 0, 0, 0, 0]`, `tempIdx = 0`
2.  Scan `nums`:
    *   `nums[0] = 0` $\to$ skip.
    *   `nums[1] = 1` $\to$ `temp[0] = 1`, `tempIdx = 1`.
    *   `nums[2] = 0` $\to$ skip.
    *   `nums[3] = 3` $\to$ `temp[1] = 3`, `tempIdx = 2`.
    *   `nums[4] = 12` $\to$ `temp[2] = 12`, `tempIdx = 3`.
3.  `temp` is now `[1, 3, 12, 0, 0]`.
4.  Copy `temp` back to `nums`.

### Complexity
*   **Time Complexity:** $O(N)$ — We traverse the array twice.
*   **Space Complexity:** $O(N)$ — We create an auxiliary array of size $N$.

### Code (C++17)
```cpp
#include <vector>

class SolutionBrute {
public:
    void moveZeroes(std::vector<int>& nums) {
        std::vector<int> temp(nums.size(), 0);
        int tempIdx = 0;
        
        for (int num : nums) {
            if (num != 0) {
                temp[tempIdx++] = num;
            }
        }
        
        nums = temp;
    }
};
```

---

## 6. Better Solution (Two-Pass O(1) Space)

### Algorithm
1.  Initialize `writePos = 0`.
2.  Iterate `readPos` from `0` to `N - 1`:
    *   If `nums[readPos] != 0`, set `nums[writePos] = nums[readPos]` and increment `writePos`.
3.  After the loop, run a second loop from `writePos` to `N - 1` and set `nums[i] = 0`.

### Dry Run
Input: `nums = [0, 1, 0, 3, 12]`
1.  `writePos = 0`.
2.  First pass:
    *   `readPos = 0`: `nums[0]` is 0 $\to$ skip.
    *   `readPos = 1`: `nums[1]` is 1 $\to$ `nums[writePos] = 1` $\to$ `nums = [1, 1, 0, 3, 12]`, `writePos = 1`.
    *   `readPos = 2`: `nums[2]` is 0 $\to$ skip.
    *   `readPos = 3`: `nums[3]` is 3 $\to$ `nums[writePos] = 3` $\to$ `nums = [1, 3, 0, 3, 12]`, `writePos = 2`.
    *   `readPos = 4`: `nums[4]` is 12 $\to$ `nums[writePos] = 12` $\to$ `nums = [1, 3, 12, 3, 12]`, `writePos = 3`.
3.  Second pass:
    *   Fill with `0` from index `3` to `4`.
    *   `nums[3] = 0`, `nums[4] = 0`.
    *   Final: `[1, 3, 12, 0, 0]`.

### Complexity
*   **Time Complexity:** $O(N)$ — Two passes.
*   **Space Complexity:** $O(1)$ — Modified in-place.

### Code (C++17)
```cpp
#include <vector>

class SolutionBetter {
public:
    void moveZeroes(std::vector<int>& nums) {
        int n = nums.size();
        int writePos = 0;
        
        // Phase 1: Move all non-zero elements forward
        for (int readPos = 0; readPos < n; ++readPos) {
            if (nums[readPos] != 0) {
                nums[writePos++] = nums[readPos];
            }
        }
        
        // Phase 2: Fill the rest with zeroes
        for (int i = writePos; i < n; ++i) {
            nums[i] = 0;
        }
    }
};
```

---

## 7. Optimal Solution (One-Pass Swapping)

### Algorithm
1.  Initialize `insertPos = 0` (this pointer tracks where the next non-zero element should go).
2.  Iterate `i` from `0` to `N - 1`:
    *   If `nums[i]` is not `0`:
        *   Swap `nums[i]` and `nums[insertPos]`.
        *   Increment `insertPos`.

### Why Optimal
*   It performs the entire shifting in a single pass.
*   It minimizes write operations. For example, if there are no zeroes, no unnecessary assignments are made (or we can add a check `if (i != insertPos)` to prevent self-swaps).
*   Space complexity is strictly $O(1)$.

---

### Beautiful C++17 Code

```cpp
#include <vector>
#include <algorithm>

class SolutionOptimal {
public:
    void moveZeroes(std::vector<int>& nums) {
        int n = nums.size();
        int insertPos = 0;
        
        for (int i = 0; i < n; ++i) {
            if (nums[i] != 0) {
                // Avoid redundant swaps with itself
                if (i != insertPos) {
                    std::swap(nums[i], nums[insertPos]);
                }
                insertPos++;
            }
        }
    }
};
```

---

## 8. Code Walkthrough

1.  `int insertPos = 0;`
    Points to the first zero element position encountered, which is ready to receive a swapped non-zero element.
2.  `for (int i = 0; i < n; ++i) { ... }`
    Main loop traversing the array left-to-right.
3.  `if (nums[i] != 0) { ... }`
    Triggered when we find a non-zero element.
4.  `if (i != insertPos) { std::swap(nums[i], nums[insertPos]); }`
    If the current index `i` is different from `insertPos`, we swap the elements. This brings the non-zero element forward and pushes the zero element backward. If `i == insertPos`, it means we haven't encountered any zeroes yet, so we avoid redundant self-swapping.
5.  `insertPos++;`
    Moves the insertion boundary forward.

---

## 9. Dry Run

Given `nums = [0, 1, 0, 3, 12]`, $N = 5$

| Step | Index `i` | Value `nums[i]` | Condition: `nums[i] != 0` | `insertPos` | Action / Array State after step |
| :--- | :--- | :--- | :--- | :--- | :--- |
| — | — | — | — | `0` | Initial: `[0, 1, 0, 3, 12]` |
| 1 | `0` | `0` | False | `0` | Skip. Array: `[0, 1, 0, 3, 12]` |
| 2 | `1` | `1` | True | `0` | Swap `nums[1]` and `nums[0]`. `insertPos` becomes 1. Array: `[1, 0, 0, 3, 12]` |
| 3 | `2` | `0` | False | `1` | Skip. Array: `[1, 0, 0, 3, 12]` |
| 4 | `3` | `3` | True | `1` | Swap `nums[3]` and `nums[1]`. `insertPos` becomes 2. Array: `[1, 3, 0, 0, 12]` |
| 5 | `4` | `12` | True | `2` | Swap `nums[4]` and `nums[2]`. `insertPos` becomes 3. Array: `[1, 3, 12, 0, 0]` |

---

## 10. Complexity Analysis

| Approach | Time Complexity (Best/Avg/Worst) | Space Complexity | Reasoning |
| :--- | :--- | :--- | :--- |
| **Brute Force** | $O(N)$ | $O(N)$ | Uses an auxiliary array of size $N$. |
| **Better (Two-Pass)** | $O(N)$ | $O(1)$ | Two sequential scans. Max writes is $N$. |
| **Optimal (One-Pass)** | $O(N)$ | $O(1)$ | Single pass. Number of swap operations is equal to the count of non-zero elements. |

---

## 11. Common Mistakes

1.  **Losing relative order:** Sorting the array to push zeroes to the end. This violates the relative order requirement of non-zero elements (e.g. `[0, 2, 1]` becomes `[1, 2, 0]` instead of `[2, 1, 0]`).
2.  **Using nested loops:** Attempting to shift zeroes by bubbling them one-by-one, resulting in $O(N^2)$ time.
3.  **Redundant swapping:** Swapping elements when `i == insertPos` which does unnecessary work.

---

## 12. Edge Cases

| Edge Case | Input Example | Expected Output | Why It Matters / How Code Handles It |
| :--- | :--- | :--- | :--- |
| **No Zeroes** | `[1, 2, 3]` | `[1, 2, 3]` | `i` is always equal to `insertPos`. No swaps occur. |
| **All Zeroes** | `[0, 0, 0]` | `[0, 0, 0]` | Condition is never met. No swaps occur. |
| **Zeroes at the End** | `[1, 2, 0, 0]` | `[1, 2, 0, 0]` | Standard traversal keeps non-zeroes in place. |
| **Zeroes at the Front** | `[0, 0, 1, 2]` | `[1, 2, 0, 0]` | Zeroes are immediately swapped with non-zero elements. |
| **Single Element Array** | `[0]` or `[1]` | `[0]` or `[1]` | Loops once, executes correctly. |

---

## 13. Interview Explanation

> "To move all zeroes to the end of an array while maintaining the order of non-zero elements, we want to perform this modification in-place.
> A naive solution is to use a temporary array, copy all non-zero elements first, fill the rest with zeroes, and copy it back. This takes linear space.
> We can optimize this to $O(1)$ space using a two-pointer approach.
> The optimal method is a single-pass swapping algorithm. We maintain a slow pointer, `insertPos`, which tracks where the next non-zero element should go.
> We then iterate through the array with a fast pointer `i`. When we encounter a non-zero element, we swap it with the element at `insertPos` and increment `insertPos`. This naturally pushes all zeroes to the end of the array without needing a second pass or extra space, taking $O(N)$ time and $O(1)$ space."

---

## 14. Follow-up Questions

1.  **Can we minimize the number of writes even further?**
    *   *Answer:* The two-pass method actually does fewer writes than swapping if the array has a large number of non-zero elements already in place (since swaps perform 3 writes). However, the swap approach is generally preferred because it is a single-pass solution. We can optimize swapping by checking `if (i != insertPos)`.
2.  **What if the array is read-only?**
    *   *Answer:* If the array is read-only, we are forced to allocate extra space, yielding $O(N)$ space complexity.

---

## 15. Variations

*   **Remove Element (LeetCode 27):**
    *   *Difference:* Instead of zeroes, move any target value `val` to the end, and return the count of non-target elements.
*   **Sort Colors / 3-Way Partitioning (LeetCode 75):**
    *   *Difference:* Categorize three different elements (0s, 1s, 2s) in-place instead of just two categories (zeroes and non-zeroes).

---

## 16. Revision Notes

*   **Slow & Fast Pointer Pattern:** Use a slow pointer to track the insertion boundary and a fast pointer to scan for active values.
*   **Self-Swap Prevention:** Check `i != insertPos` before swapping to minimize register modifications.

---

## 17. Memorization Version

```cpp
void moveZeroes(vector<int>& nums) {
    int insertPos = 0;
    for (int i = 0; i < nums.size(); ++i) {
        if (nums[i] != 0) {
            if (i != insertPos) {
                swap(nums[i], nums[insertPos]);
            }
            insertPos++;
        }
    }
}
```
*Key reminder: Swap non-zeroes forward, increment insertPos.*
