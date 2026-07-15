# Array Rotation – Juggling Algorithm

## 1. Problem Statement

Given an array of integers `arr` of size `N`, rotate the array to the left by `d` positions in-place.

### Input
- An integer array `arr` of size `N`.
- An integer `d` ($0 \le d \le 10^5$).

### Output
- The array `arr` rotated left by `d` positions in-place.

### Constraints
- $1 \le N \le 10^5$
- $0 \le d \le 10^5$
- $-10^9 \le arr[i] \le 10^9$

### Observations
1. Rotating left by `d` positions means moving the first `d` elements to the back.
2. Rotating by `d` is equivalent to rotating by `d % N`. If `d` is a multiple of `N`, the array remains unchanged.
3. The rotation must be done in-place with $O(1)$ auxiliary space.

---

## 2. Interview Clarification

- **Q:** Can `d` be greater than `N`?
  - **A:** Yes. We handle this by setting `d = d % N`.
- **Q:** Is the rotation left or right?
  - **A:** Left rotation.
- **Q:** What is the target space complexity?
  - **A:** $O(1)$ auxiliary space (in-place).
- **Q:** Can we use standard library utilities like `std::rotate`?
  - **A:** We should implement the underlying algorithm ourselves to show our understanding of cycle-based rotation.

---

## 3. Thinking Process

### First Instinct (Brute Force)
We can rotate the array left by 1 position, and repeat this process `d` times.
- To rotate left by 1: save `arr[0]` in a temporary variable, shift `arr[1...N-1]` to `arr[0...N-2]`, and place the saved element at `arr[N-1]`.
- Repeating this `d` times takes $O(N \cdot d)$ time and $O(1)$ space. For large $N$ and $d$, this is too slow.

### Using Extra Space (Better Space-Time Tradeoff)
We can copy the first `d` elements into a temporary array, shift the remaining `N - d` elements to the front, and then copy the temporary array back to the end of the original array.
- This takes $O(N)$ time and $O(d)$ auxiliary space. 
- While fast, it violates the $O(1)$ auxiliary space constraint.

### The Reversal Algorithm (Common & Elegant)
A highly popular $O(N)$ time, $O(1)$ space approach is the **Reversal Algorithm**:
1. Reverse the first `d` elements: `reverse(arr, 0, d - 1)`.
2. Reverse the remaining `N - d` elements: `reverse(arr, d, N - 1)`.
3. Reverse the entire array: `reverse(arr, 0, N - 1)`.

This requires three reverse passes, resulting in $O(N)$ time and $O(1)$ space. It is simple to implement but performs approximately $2N$ write operations.

### The Juggling Algorithm (Optimal Single-Pass Cycle Shifting)
Can we move each element directly to its final position in a single pass?
Yes! This is known as the **Juggling Algorithm** (or Cycle Leader Algorithm). It decomposes the rotation into cyclic shifts.
- If we shift elements by `d` positions, the elements will form cycles of indices:
  $$i \rightarrow (i + d) \bmod N \rightarrow (i + 2d) \bmod N \dots$$
- The number of independent, non-overlapping cycles is exactly equal to the greatest common divisor of $N$ and $d$:
  $$\text{Number of Cycles} = \text{GCD}(N, d)$$
- For each cycle starting at index $i \in [0, \text{GCD}(N, d) - 1]$:
  1. Save `temp = arr[i]`.
  2. Move the element at $(j + d) \bmod N$ to index $j$, repeating this step along the cycle path.
  3. When the cycle wraps back to the starting index $i$, place `temp` into the final vacant slot.
- This approach visits and writes each element of the array exactly once (excluding the cycle leaders stored in `temp`), making it highly write-efficient.

---

## 4. Pattern Recognition

- **Pattern:** Permutations and Cyclic Decomposition / Greatest Common Divisor (GCD).
- **Why this topic?** Array rotation is a cyclic shift permutation. Any permutation can be decomposed into disjoint cycles. Knowing that the number of cycles is $GCD(N, d)$ allows us to process each cycle independently in-place.

---

## 5. Brute Force Solution

### Algorithm
1. Set `d = d % N`.
2. Loop `d` times:
   - Save `temp = arr[0]`.
   - Shift elements left: loop `j` from `0` to `N-2`: `arr[j] = arr[j+1]`.
   - Place `arr[N-1] = temp`.

### Complexity Analysis
- **Time Complexity:** $O(N \cdot d)$ (if $d \approx N$, it becomes $O(N^2)$).
- **Space Complexity:** $O(1)$ auxiliary space.

### Brute Force C++17 Code
```cpp
#include <vector>

class Solution {
public:
    void rotateBrute(std::vector<int>& arr, int d) {
        int n = arr.size();
        if (n == 0) return;
        d = d % n;
        if (d == 0) return;
        
        for (int i = 0; i < d; ++i) {
            int temp = arr[0];
            for (int j = 0; j < n - 1; ++j) {
                arr[j] = arr[j + 1];
            }
            arr[n - 1] = temp;
        }
    }
};
```

---

## 6. Better Solution (Reversal Algorithm)

### Algorithm
1. Set `d = d % N`.
2. Reverse the prefix `arr[0...d-1]`.
3. Reverse the suffix `arr[d...N-1]`.
4. Reverse the entire array `arr[0...N-1]`.

### Complexity Analysis
- **Time Complexity:** $O(N)$ (every element is swapped twice, meaning $\approx 2N$ moves).
- **Space Complexity:** $O(1)$ auxiliary space.

### Reversal C++17 Code
```cpp
#include <vector>
#include <algorithm>

class Solution {
private:
    void reverseArray(std::vector<int>& arr, int start, int end) {
        while (start < end) {
            std::swap(arr[start], arr[end]);
            start++;
            end--;
        }
    }
public:
    void rotateReversal(std::vector<int>& arr, int d) {
        int n = arr.size();
        if (n == 0) return;
        d = d % n;
        if (d == 0) return;
        
        reverseArray(arr, 0, d - 1);
        reverseArray(arr, d, n - 1);
        reverseArray(arr, 0, n - 1);
    }
};
```

---

## 7. Optimal Solution (Juggling Algorithm)

### Algorithm
1. Set `d = d % N`.
2. Compute `g = gcd(N, d)`.
3. Loop `i` from `0` to `g - 1` (for each cycle):
   - Store `temp = arr[i]`.
   - Set `j = i`.
   - Inside an infinite loop:
     - Compute the next index: `k = j + d`.
     - If `k >= N`, wrap it around: `k = k - N`.
     - If `k == i`, we have completed the cycle. Break the loop.
     - Shift the element: `arr[j] = arr[k]`.
     - Move the pointer: `j = k`.
   - Place the stored element back into the final vacancy: `arr[j] = temp`.

### Why this is Optimal
It achieves $O(N)$ time complexity and $O(1)$ space complexity. More importantly, it writes to each array slot exactly once. In hardware architectures where memory write operations are slower or consume more energy (such as flash memory), this algorithm is superior to the Reversal Algorithm because it performs half the write operations.

### C++17 Optimal Code
```cpp
#include <vector>
#include <numeric>

class Solution {
private:
    // Helper function to find greatest common divisor
    int getGCD(int a, int b) {
        if (b == 0) return a;
        return getGCD(b, a % b);
    }
public:
    void rotateJuggling(std::vector<int>& arr, int d) {
        int n = arr.size();
        if (n == 0) return;
        
        d = d % n;
        if (d == 0) return;
        
        // Find GCD of N and d to determine the number of cycles
        int gcd_val = getGCD(n, d);
        
        for (int i = 0; i < gcd_val; ++i) {
            // Save the starting element of the current cycle
            int temp = arr[i];
            int j = i;
            
            while (true) {
                int k = j + d;
                if (k >= n) {
                    k = k - n; // Wrap around index
                }
                
                // If we are back at the start, the cycle is complete
                if (k == i) {
                    break;
                }
                
                // Move elements
                arr[j] = arr[k];
                j = k;
            }
            // Put the saved element in its final position
            arr[j] = temp;
        }
    }
};
```

---

## 8. Code Walkthrough

1. **GCD Calculation**: `int gcd_val = getGCD(n, d);` computes the number of independent shifting cycles.
2. **Cycle Iteration**: We loop from `i = 0` to `gcd_val - 1`. Each loop represents one cycle starting with the leader index `i`.
3. **Inner Cycle Traversal**:
   - `temp = arr[i]` holds the leader element.
   - `int j = i` acts as the vacant slot index.
   - `int k = j + d` points to the element that must fill the vacant slot `j`. If `k >= n`, we adjust it by subtracting `n` (which is faster than modulo `%`).
   - If `k == i`, it means the element at index `k` was already saved in `temp`. We break and place `temp` at index `j`.
   - Otherwise, we perform the shift `arr[j] = arr[k]` and update `j = k` to represent the new vacant slot.

---

## 9. Dry Run

Let's dry run the optimal solution on `arr = [1, 2, 3, 4, 5, 6]`, `N = 6`, `d = 2`.
- `d = 2 % 6 = 2`.
- `gcd(6, 2) = 2`. There will be 2 cycles.

### Cycle 0 (`i = 0`)
- `temp = arr[0] = 1`, `j = 0`
- **Step 1:** `k = j + d = 0 + 2 = 2`.
  - Is `k == i`? `2 == 0` (False).
  - Shift: `arr[0] = arr[2]` $\rightarrow$ `arr[0] = 3`.
  - Update: `j = 2`. Array: `[3, 2, 3, 4, 5, 6]`.
- **Step 2:** `k = j + d = 2 + 2 = 4`.
  - Is `k == i`? `4 == 0` (False).
  - Shift: `arr[2] = arr[4]` $\rightarrow$ `arr[2] = 5`.
  - Update: `j = 4`. Array: `[3, 2, 5, 4, 5, 6]`.
- **Step 3:** `k = j + d = 4 + 2 = 6 >= 6` $\rightarrow$ `k = 6 - 6 = 0`.
  - Is `k == i`? `0 == 0` (True).
  - Break loop.
- **End of Cycle:** `arr[j] = temp` $\rightarrow$ `arr[4] = 1`. Array: `[3, 2, 5, 4, 1, 6]`.

### Cycle 1 (`i = 1`)
- `temp = arr[1] = 2`, `j = 1`
- **Step 1:** `k = j + d = 1 + 2 = 3`.
  - Is `k == i`? `3 == 1` (False).
  - Shift: `arr[1] = arr[3]` $\rightarrow$ `arr[1] = 4`.
  - Update: `j = 3`. Array: `[3, 4, 5, 4, 1, 6]`.
- **Step 2:** `k = j + d = 3 + 2 = 5`.
  - Is `k == i`? `5 == 1` (False).
  - Shift: `arr[3] = arr[5]` $\rightarrow$ `arr[3] = 6`.
  - Update: `j = 5`. Array: `[3, 4, 5, 6, 1, 6]`.
- **Step 3:** `k = j + d = 5 + 2 = 7 >= 6` $\rightarrow$ `k = 7 - 6 = 1`.
  - Is `k == i`? `1 == 1` (True).
  - Break loop.
- **End of Cycle:** `arr[j] = temp` $\rightarrow$ `arr[5] = 2`. Array: `[3, 4, 5, 6, 1, 2]`.

**Final Array:** `[3, 4, 5, 6, 1, 2]`

---

## 10. Complexity Analysis

- **Time Complexity:** 
  - **Best/Average/Worst Case:** $O(N)$ because each element is visited and shifted exactly once.
- **Space Complexity:**
  - **Auxiliary Space:** $O(1)$ since only a few temporary integers are used.

---

## 11. Common Mistakes

1. **Not applying modulo operator**: If `d >= N`, accessing indices like `j + d` directly can lead to out-of-bounds errors. Always do `d = d % N` first.
2. **Incorrect modulo wrap-around logic**: Writing `k = (j + d) % n` inside the inner loop is mathematically correct but slower than `k = j + d; if (k >= n) k -= n;`.
3. **Wrong number of cycles**: Shifting all elements starting from index 0 without stopping can cause infinite loops or corrupt the array if $GCD(N, d) > 1$. You must divide the process into exactly $GCD(N, d)$ outer cycles.

---

## 12. Edge Cases

| Edge Case | Input Example | Expected Output | Why it matters |
|:---|:---|:---|:---|
| **d = 0** | `[1, 2, 3]`, `d = 0` | `[1, 2, 3]` | No rotation needed. |
| **d = N** | `[1, 2, 3]`, `d = 3` | `[1, 2, 3]` | Modulo reduces `d` to 0. |
| **GCD(N, d) = 1** | `[1, 2, 3, 4, 5]`, `d = 2` | `[3, 4, 5, 1, 2]` | Single cycle processes entire array. |
| **GCD(N, d) = d** | `[1, 2, 3, 4]`, `d = 2` | `[3, 4, 1, 2]` | Array is split into $d$ cycles. |

---

## 13. Interview Explanation

"To rotate an array left by `d` positions in-place, the simplest optimal solution is the **Reversal Algorithm**, which reverses the first `d` elements, the remaining `N - d` elements, and then the entire array. This takes $O(N)$ time and $O(1)$ space, but requires roughly $2N$ element modifications because every element is swapped twice.

For a more write-efficient solution, we can use the **Juggling Algorithm**. It shifts elements cyclicly to their final positions. We calculate $g = \text{GCD}(N, d)$, which tells us the number of independent, non-overlapping cycles in our rotation permutation. We run $g$ cycles. In each cycle starting at index `i`, we save `arr[i]` in a temporary variable, shift elements from their respective offsets `(j + d) % N` to `j`, and finally place our saved value in the last open spot. 

This Juggling Algorithm also runs in $O(N)$ time and $O(1)$ space, but only performs $N$ write operations, making it twice as fast as the Reversal Algorithm in terms of memory writes."

---

## 14. Follow-up Questions

- **Q:** How would you modify the Juggling Algorithm to rotate the array to the **right**?
  - **A:** To rotate right by `d`, we can convert it to a left rotation by `N - d` positions: `d_left = (N - (d % N)) % N`. Alternatively, we can adjust the cycle index calculations to traverse backward by subtracting `d` instead of adding it.
- **Q:** What is the runtime of calculating GCD?
  - **A:** Finding GCD of $N$ and $d$ using Euclidean algorithm takes $O(\log(\min(N, d)))$ time, which is negligible compared to the $O(N)$ array scan.

---

## 15. Variations

- **Rotate Linked List:** Instead of index swapping, we locate the $(N-d)$-th node, make it the new head, and link the original end to the original head.
- **Rotate a 2D Matrix:** Requires rotating layer-by-layer or performing transpose followed by row reversals.

---

## 16. Revision Notes

- **Core concept:** Permutations are decomposed into $GCD(N, d)$ disjoint cycles.
- **Next index formula:** `k = j + d; if (k >= n) k -= n;`.
- **Memory efficiency:** Juggling is highly write-efficient (only $N$ writes), whereas Reversal writes $\approx 2N$ times.

---

## 17. Memorization Version

- **Step 1:** Set `d = d % N`, find `g = gcd(N, d)`.
- **Step 2:** Loop `i` from `0` to `g - 1`:
  - `temp = arr[i]`, `j = i`
  - Loop:
    - `k = j + d`. If `k >= n` $\rightarrow$ `k -= n`.
    - If `k == i` $\rightarrow$ Break.
    - `arr[j] = arr[k]`, `j = k`.
  - `arr[j] = temp`.
- **Complexity:** $O(N)$ time, $O(1)$ space.
