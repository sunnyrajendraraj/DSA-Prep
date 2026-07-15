# Rotate Array by K Positions (Right)

---

## 1. Problem Statement

Given an integer array `nums` and a non-negative integer `k`, rotate the array to the right by `k` steps, where `k` is non-negative. Rotating an array to the right means shifting each element to the right, and the elements at the end wrap around to the front.

### Input
- An integer array `nums` of size `n` ($1 \le n \le 10^5$).
- An integer `k` ($0 \le k \le 10^5$).

### Output
- The array `nums` modified in-place (or returned depending on the platform) representing the rotated array.

### Constraints
- $1 \le n \le 10^5$
- $-2^{31} \le nums[i] \le 2^{31} - 1$
- $0 \le k \le 10^5$

### Observations
1. **Modulo Arithmetic:** If $k \ge n$, rotating the array $n$ times returns it to its original state. Therefore, rotating by $k$ is equivalent to rotating by $k \pmod n$.
2. **Direction:** Rotating to the right by $k$ is equivalent to rotating to the left by $n - k$.
3. **In-place:** The optimal solution should perform the rotation without allocating a new array (i.e., $O(1)$ auxiliary space).

---

## 2. Interview Clarification

Before writing any code, a candidate should clarify the following with the interviewer:
1. **Can $k$ be larger than the size of the array?**
   - *Answer:* Yes, $k$ can be greater than $n$. In that case, we should take $k = k \pmod n$.
2. **Can $k$ be negative?**
   - *Answer:* For this problem, assume $k \ge 0$. If $k$ could be negative, rotating right by $-k$ is equivalent to rotating left by $k$ (or right by $n - k$).
3. **Should we modify the array in-place or return a new array?**
   - *Answer:* In-place modification is highly preferred to maintain space efficiency.
4. **What if the array is empty or has only one element?**
   - *Answer:* If $n \le 1$, no rotation is needed. Return immediately.

---

## 3. Thinking Process

Let's think aloud:
* **Initial thoughts:** If I have an array `[1, 2, 3, 4, 5]` and $k = 2$, the result should be `[4, 5, 1, 2, 3]`. The last $k$ elements `[4, 5]` move to the front, and the first $n-k$ elements `[1, 2, 3]` shift to the back.
* **First Instinct (Extra Array):** What if I create a temporary array of the same size? I can copy elements at index `i` to index `(i + k) % n` in the temp array, and then copy the temp array back to `nums`. This is straightforward but uses $O(n)$ extra memory.
* **Second Instinct (Rotate One-by-One):** What if I shift elements one position to the right, $k$ times?
  - Shift 1: `[5, 1, 2, 3, 4]`
  - Shift 2: `[4, 5, 1, 2, 3]`
  This takes $O(1)$ space, but for each of the $k$ rotations, we shift $n$ elements. Total time complexity: $O(n \times k)$. If $k \approx n \approx 10^5$, this will lead to a Time Limit Exceeded (TLE) error.
* **Optimal Insight (The Reversal Pattern):** How can we achieve $O(n)$ time complexity and $O(1)$ space? Let's observe the structure:
  - Input: `[1, 2, 3, 4, 5, 6, 7]`, $k = 3$. Output: `[5, 6, 7, 1, 2, 3, 4]`.
  - Let's split the array into two parts: the first $n - k$ elements `[1, 2, 3, 4]` and the last $k$ elements `[5, 6, 7]`.
  - If we reverse the first part: `[4, 3, 2, 1, 5, 6, 7]`
  - If we reverse the second part: `[4, 3, 2, 1, 7, 6, 5]`
  - If we reverse the entire array: `[5, 6, 7, 1, 2, 3, 4]`.
  - Wow! This matches the output exactly! Let's double check if we can do this in reverse order:
    1. Reverse the entire array: `[7, 6, 5, 4, 3, 2, 1]`
    2. Reverse the first $k$ elements: `[5, 6, 7, 4, 3, 2, 1]`
    3. Reverse the remaining $n - k$ elements: `[5, 6, 7, 1, 2, 3, 4]`.
    Yes, this also works and is mathematically equivalent. It only requires reversing subsegments of the array, which takes $O(n)$ time and $O(1)$ auxiliary space.

---

## 4. Pattern Recognition

This problem belongs to the **Array Manipulation** and **In-place Transformation** patterns.
* **Why this topic?** Reordering elements within a sequential container is a fundamental array manipulation task.
* **Which pattern?** The **Reversal Algorithm** is a classic technique used to rotate blocks of elements. By reversing sub-segments, we change the relative order of blocks without requiring temporary storage.

---

## 5. Brute Force Solution

The brute force approach uses an extra array to temporarily store the elements in their new rotated positions.

### Algorithm
1. Retrieve the size of the array $n$.
2. Calculate the effective rotation index $k = k \pmod n$. If $k == 0$ or $n \le 1$, return immediately.
3. Instantiate a temporary array `temp` of size $n$.
4. Iterate through the array `nums` from index $0$ to $n-1$.
5. Place each element `nums[i]` at its new position in the temporary array: `temp[(i + k) % n] = nums[i]`.
6. Copy all elements from `temp` back into `nums`.

### Dry Run
Input: `nums = [1, 2, 3, 4, 5]`, $k = 2$, $n = 5$.
1. Effective $k = 2 \pmod 5 = 2$.
2. Temp array of size 5 created: `temp = [0, 0, 0, 0, 0]`.
3. Iteration:
   - $i = 0$: `temp[(0+2)%5] = nums[0]` $\rightarrow$ `temp[2] = 1`.
   - $i = 1$: `temp[(1+2)%5] = nums[1]` $\rightarrow$ `temp[3] = 2`.
   - $i = 2$: `temp[(2+2)%5] = nums[2]` $\rightarrow$ `temp[4] = 3`.
   - $i = 3$: `temp[(3+2)%5] = nums[3]` $\rightarrow$ `temp[0] = 4`.
   - $i = 4$: `temp[(4+2)%5] = nums[4]` $\rightarrow$ `temp[1] = 5`.
4. Copy `temp` (`[4, 5, 1, 2, 3]`) back to `nums`.

### Complexity Analysis
- **Time Complexity:** $O(n)$ — We iterate through the array twice (once to copy to `temp`, once to copy back).
- **Space Complexity:** $O(n)$ — Required to store the temporary array of size $n$.

### Pros/Cons
- **Pros:** Extremely easy to understand; requires no complex pointer arithmetic.
- **Cons:** High memory footprint. Memory allocation can be slow for very large arrays.

### C++17 Code
```cpp
#include <vector>

class Solution {
public:
    void rotate(std::vector<int>& nums, int k) {
        int n = nums.size();
        if (n <= 1) return;
        
        // Handle k greater than array size
        k = k % n;
        if (k == 0) return;
        
        // Allocate temporary array
        std::vector<int> temp(n);
        
        // Copy to rotated positions
        for (int i = 0; i < n; ++i) {
            temp[(i + k) % n] = nums[i];
        }
        
        // Copy back to original array
        nums = temp;
    }
};
```

---

## 6. Better Solution

We can rotate the array one-by-one, $k$ times. This avoids allocating a full temporary array of size $n$, resulting in $O(1)$ auxiliary space.

### Algorithm
1. Calculate effective $k = k \pmod n$. If $k == 0$ or $n \le 1$, return.
2. Repeat $k$ times:
   a. Store the last element of the array in a temporary variable `last`.
   b. Shift all elements from index $n-2$ down to $0$ one position to the right.
   c. Put `last` at index $0$.

### Dry Run
Input: `nums = [1, 2, 3, 4, 5]`, $k = 2$.
- **First rotation:**
  - `last = nums[4] = 5`.
  - Shift elements right: `nums` becomes `[1, 1, 2, 3, 4]`.
  - Assign `nums[0] = 5`. Array is now `[5, 1, 2, 3, 4]`.
- **Second rotation:**
  - `last = nums[4] = 4`.
  - Shift elements right: `nums` becomes `[5, 5, 1, 2, 3]`.
  - Assign `nums[0] = 4`. Array is now `[4, 5, 1, 2, 3]`.

### Complexity Analysis
- **Time Complexity:** $O(n \times k)$ — For each of the $k$ rotations, we shift $n-1$ elements.
- **Space Complexity:** $O(1)$ — Only a single variable `last` is used.

### Pros/Cons
- **Pros:** In-place, $O(1)$ space.
- **Cons:** Very slow for large $n$ and $k$. If $n = 10^5$ and $k = 10^5$, it requires $10^{10}$ operations, causing a timeout.

### C++17 Code
```cpp
#include <vector>

class Solution {
public:
    void rotate(std::vector<int>& nums, int k) {
        int n = nums.size();
        if (n <= 1) return;
        
        k = k % n;
        if (k == 0) return;
        
        // Perform k individual rotations
        for (int step = 0; step < k; ++step) {
            int last_element = nums[n - 1];
            // Shift elements right by 1
            for (int i = n - 1; i > 0; --i) {
                nums[i] = nums[i - 1];
            }
            nums[0] = last_element;
        }
    }
};
```

---

## 7. Optimal Solution

The optimal approach uses the **Reversal Algorithm**. It rotates the array in-place using three reverse operations, yielding $O(n)$ time and $O(1)$ space.

### Algorithm
1. Calculate effective $k = k \pmod n$. If $k == 0$ or $n \le 1$, return.
2. Reverse the first $n - k$ elements (from index $0$ to $n - k - 1$).
3. Reverse the last $k$ elements (from index $n - k$ to $n - 1$).
4. Reverse the entire array (from index $0$ to $n - 1$).

*(Alternatively: Reverse the entire array first, then reverse the first $k$ elements, and then the remaining $n - k$ elements. Both approaches yield the same result.)*

### Dry Run
Input: `nums = [1, 2, 3, 4, 5, 6, 7]`, $k = 3$, $n = 7$.
1. Effective $k = 3 \pmod 7 = 3$.
2. Reverse first $n - k = 4$ elements (`nums[0...3]`):
   - Before: `[1, 2, 3, 4, 5, 6, 7]`
   - After: `[4, 3, 2, 1, 5, 6, 7]`
3. Reverse last $k = 3$ elements (`nums[4...6]`):
   - Before: `[4, 3, 2, 1, 5, 6, 7]`
   - After: `[4, 3, 2, 1, 7, 6, 5]`
4. Reverse the whole array (`nums[0...6]`):
   - Before: `[4, 3, 2, 1, 7, 6, 5]`
   - After: `[5, 6, 7, 1, 2, 3, 4]`

The array is successfully rotated.

### C++17 Code
```cpp
#include <vector>
#include <algorithm> // For std::reverse

class Solution {
private:
    // Helper function to reverse subarray in place
    void reverseSubarray(std::vector<int>& nums, int start, int end) {
        while (start < end) {
            std::swap(nums[start], nums[end]);
            start++;
            end--;
        }
    }

public:
    void rotate(std::vector<int>& nums, int k) {
        int n = nums.size();
        if (n <= 1) return;
        
        // Handle k > n
        k = k % n;
        if (k == 0) return;
        
        // Step 1: Reverse first n - k elements
        reverseSubarray(nums, 0, n - k - 1);
        
        // Step 2: Reverse last k elements
        reverseSubarray(nums, n - k, n - 1);
        
        // Step 3: Reverse the entire array
        reverseSubarray(nums, 0, n - 1);
    }
};
```

---

## 8. Code Walkthrough

Let's dissect the optimal solution:
* `k = k % n;`: This operation handles cases where $k$ is larger than $n$. A rotation of size $n$ returns the array to its original position, so we only need the remainder.
* `reverseSubarray(nums, 0, n - k - 1);`: Reverses the left subarray containing elements that will shift to the end.
* `reverseSubarray(nums, n - k, n - 1);`: Reverses the right subarray containing elements that will wrap around to the front.
* `reverseSubarray(nums, 0, n - 1);`: Reversing the whole array restores the proper relative order of the elements inside each block, while swapping their locations.

---

## 9. Dry Run

Let's trace `nums = [1, 2, 3, 4]`, $k = 6$, $n = 4$.

| Step | Operation / Variable State | Array State | Explanation |
| :--- | :--- | :--- | :--- |
| 1 | `n = nums.size()` | `[1, 2, 3, 4]` | Array size $n = 4$ |
| 2 | `k = k % n` $\rightarrow$ `6 % 4 = 2` | `[1, 2, 3, 4]` | $k$ becomes 2 |
| 3 | Reverse `nums[0...1]` ($n-k-1 = 1$) | `[2, 1, 3, 4]` | Reverse first 2 elements |
| 4 | Reverse `nums[2...3]` ($n-k = 2$) | `[2, 1, 4, 3]` | Reverse last 2 elements |
| 5 | Reverse `nums[0...3]` | `[3, 4, 1, 2]` | Reverse entire array |

---

## 10. Complexity Analysis

| Approach | Time Complexity | Space Complexity | Notes |
| :--- | :--- | :--- | :--- |
| **Brute Force** | $O(n)$ | $O(n)$ | Allocates a full copy of the array. |
| **Better** | $O(n \times k)$ | $O(1)$ | Rotates elements one-by-one; prone to TLE. |
| **Optimal** | $O(n)$ | $O(1)$ | Three passes of array reversal; very efficient. |

* **Time Complexity Reason (Optimal):** We reverse three segments. The total elements swapped across the steps is $2 \times n$, which simplifies to $O(n)$.
* **Space Complexity Reason (Optimal):** We swap elements in-place using temporary variables, so no dynamic memory is allocated.

---

## 11. Common Mistakes

1. **Forgetting $k \ge n$:** Not applying modulo `k % n` leads to an out-of-bounds array access when dividing the array segments.
2. **Incorrect Indices:** Getting the boundaries of the subsegment reversals wrong (e.g., reversing from `0` to `n - k` instead of `n - k - 1`).
3. **Array size 1:** Forgetting to return early when $n = 1$, which can cause division-by-zero or negative boundaries.

---

## 12. Edge Cases

| Edge Case | Input Example | Expected Output | Why it matters |
| :--- | :--- | :--- | :--- |
| **$k = 0$** | `nums=[1,2]`, $k=0$ | `[1,2]` | Modulo yields 0, must return early. |
| **$k$ is a multiple of $n$** | `nums=[1,2,3]`, $k=6$ | `[1,2,3]` | Effective rotation is 0; must not error. |
| **$n = 1$** | `nums=[5]`, $k=3$ | `[5]` | Single element doesn't change; avoids negative bounds. |
| **$k > n$** | `nums=[1,2]`, $k=3$ | `[2,1]` | Must wrap around correctly ($k \pmod n = 1$). |

---

## 13. Interview Explanation

"To rotate an array of size $n$ to the right by $k$ positions, my first observation is that rotating $n$ times yields the same array. Hence, we can simplify $k$ to $k \pmod n$.
A naive approach would use an extra array of size $n$, placing each element at `(i + k) % n`. This takes linear time but requires $O(n)$ auxiliary space.
To do this in-place with $O(1)$ space, we can use the **Reversal Algorithm**. If we split the array into two parts: the first $n - k$ elements and the last $k$ elements, we can reverse both parts individually, and then reverse the entire array. This swaps the two blocks and restores their original ordering, completing the rotation in $O(n)$ time and $O(1)$ auxiliary space."

---

## 14. Follow-up Questions

1. **Can you rotate the array to the left instead of the right?**
   * *Answer:* Yes. Rotating left by $k$ is equivalent to rotating right by $n - k$. We would reverse the first $k$ elements, then the remaining $n - k$ elements, and then the entire array.
2. **What if we want to rotate a linked list instead?**
   * *Answer:* For a linked list, we can find the length, make the list circular by connecting the tail to the head, and then break the connection at the $(n - k)$-th node.

---

## 15. Variations

1. **Rotate Array (Left):** Rotate the array to the left by $k$ positions.
   - *Formula:* Reverse `0` to `k-1`, reverse `k` to `n-1`, then reverse `0` to `n-1`.
2. **Rotate Image / Matrix:** Rotating a 2D grid by 90 degrees.
   - *Approach:* Transpose the matrix first, then reverse each row.

---

## 16. Revision Notes

* **Trick:** Reversing parts swaps positions of blocks while keeping internal elements correct once fully reversed.
* **Modulo Formula:** Always use `k = k % n`.
* **Sub-array bounds:**
  - Left segment: `0` to `n - k - 1`
  - Right segment: `n - k` to `n - 1`
  - Full: `0` to `n - 1`

---

## 17. Memorization Version

```cpp
// Rotate right by k:
// k = k % n;
// reverse(nums.begin(), nums.end() - k);
// reverse(nums.end() - k, nums.end());
// reverse(nums.begin(), nums.end());
```
* **Steps:** Reverse First $n-k$ $\rightarrow$ Reverse Last $k$ $\rightarrow$ Reverse All.
* **Time:** $O(n)$, **Space:** $O(1)$.
