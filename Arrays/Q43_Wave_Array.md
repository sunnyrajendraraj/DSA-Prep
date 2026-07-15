# Wave Array - Comprehensive Interview Preparation Module

## 1. Problem Statement
Given an unsorted array of integers `arr`, rearrange the elements in-place into a **wave-like array** such that:
$$arr[0] \ge arr[1] \le arr[2] \ge arr[3] \le arr[4] \dots$$

In other words, the elements at even indices ($0, 2, 4, \dots$) must be greater than or equal to their immediate neighboring elements, while elements at odd indices ($1, 3, 5, \dots$) must be less than or equal to their neighbors.

### Input
- An integer array `arr` of size $N$ ($1 \le N \le 10^6$).
- Element values: $-10^9 \le arr[i] \le 10^9$.

### Output
- The array rearranged in-place to satisfy the wave property. (If there are multiple valid wave arrays, any is acceptable).

### Constraints
- Time Complexity: Optimal $O(N)$
- Auxiliary Space: $O(1)$ in-place

### Observations
1. In a wave array, elements alternate between "local maxima" (peaks) at even indices and "local minima" (valleys) at odd indices.
2. The problem does not require a sorted wave (like lexicographically smallest). Any wave arrangement is acceptable.
3. Swapping adjacent elements is a key operation to fix local violations of the wave property.

---

## 2. Interview Clarification
Before coding, a candidate should clarify the following with the interviewer:
1. **Can the array contain duplicate elements?**
   - *Answer:* Yes, duplicates are allowed. The relations are non-strict ($\ge$ and $\le$), so duplicates can sit next to each other.
2. **Is there a preference for which wave form is required?** (i.e., $arr[0] \le arr[1] \ge arr[2]$ vs $arr[0] \ge arr[1] \le arr[2]$)
   - *Answer:* The problem specifies $arr[0] \ge arr[1] \le arr[2] \ge arr[3] \dots$ (Peaks at even indices, valleys at odd indices).
3. **Do we need to return the lexicographically smallest wave array?**
   - *Answer:* No, any wave-form arrangement is acceptable. If lexicographically smallest were required, sorting first would be mandatory.
4. **Is it acceptable to modify the input array in-place?**
   - *Answer:* Yes, in-place modification with $O(1)$ auxiliary space is expected.

---

## 3. Thinking Process
Let's build the intuition step-by-step:
1. **First Instinct (Sorting):** 
   - If we sort the array in ascending order, say `[1, 2, 3, 4, 5, 6]`.
   - To create a wave $arr[0] \ge arr[1] \le arr[2] \ge arr[3] \dots$, let's look at pairs:
     - Swap `arr[0]` and `arr[1]` $\to$ `[2, 1, 3, 4, 5, 6]`
     - Swap `arr[2]` and `arr[3]` $\to$ `[2, 1, 4, 3, 5, 6]`
     - Swap `arr[4]` and `arr[5]` $\to$ `[2, 1, 4, 3, 6, 5]`
   - Let's check: $2 \ge 1 \le 4 \ge 3 \le 6 \ge 5$. This is a valid wave!
   - Sorting takes $O(N \log N)$ time. Can we do better?

2. **Optimal Insight (Single Pass / Greedy Local Correction):**
   - Sorting is overkill because we don't need a global ordering; we only need a local relationship between adjacent elements.
   - Let's focus on even indices ($i = 0, 2, 4, \dots$). These elements must be peaks:
     - $arr[i] \ge arr[i-1]$ (if $i > 0$)
     - $arr[i] \ge arr[i+1]$ (if $i < N-1$)
   - If we traverse the array focusing only on even indices:
     - If the previous element is greater than the current element ($arr[i] < arr[i-1]$), we swap them.
     - If the next element is greater than the current element ($arr[i] < arr[i+1]$), we swap them.
   - Will this swap break the previously established wave properties? Let's check with a mathematical proof.

### Correctness Proof
Suppose we are at an even index $i$.
- We ensure $arr[i] \ge arr[i-1]$ by swapping them if $arr[i] < arr[i-1]$.
- Does this swap violate the relationship established at the previous even index $i-2$?
  - Prior to visiting $i$, the wave condition at $i-2$ was satisfied, meaning $arr[i-2] \ge arr[i-1]$.
  - If we swap $arr[i]$ and $arr[i-1]$ because $arr[i] < arr[i-1]$:
    - The new value at $arr[i-1]$ is the old $arr[i]$, which is strictly smaller than the old $arr[i-1]$.
    - Since $arr[i-2] \ge \text{old } arr[i-1]$ and $\text{old } arr[i-1] > \text{new } arr[i-1]$, it transitive-wise guarantees that $arr[i-2] \ge \text{new } arr[i-1]$.
  - Therefore, the wave property at $i-2$ is **never violated** by swapping elements at $i$ and $i-1$.
- Similarly, swapping $arr[i]$ and $arr[i+1]$ will only make $arr[i+1]$ smaller, which is safe for the subsequent step.
- This allows us to solve the problem in a single pass of $O(N)$ time and $O(1)$ space.

---

## 4. Pattern Recognition
- **Topic:** Array Rearrangement / Greedy Algorithms
- **Pattern:** Local Adjustment / Neighbor Comparison
- **Why this pattern?** The constraint of the wave form is entirely localized (each element only cares about its left and right neighbors). In such problems, resolving local violations sequentially from left to right without looking back often yields the global optimal structure.

---

## 5. Brute Force Solution

### Algorithm
1. Sort the input array in ascending order.
2. Traverse the array with a step of 2 (i.e., $i = 0, 2, 4, \dots$).
3. Swap each element at index $i$ with its adjacent element at $i+1$ (if $i+1 < N$).

### Dry Run
Input: `arr = [3, 6, 5, 10, 7, 20]`
1. Sort the array: `arr = [3, 5, 6, 7, 10, 20]`
2. Step 2 iteration:
   - $i = 0$: Swap `arr[0]` (3) and `arr[1]` (5) $\to$ `[5, 3, 6, 7, 10, 20]`
   - $i = 2$: Swap `arr[2]` (6) and `arr[3]` (7) $\to$ `[5, 3, 7, 6, 10, 20]`
   - $i = 4$: Swap `arr[4]` (10) and `arr[5]` (20) $\to$ `[5, 3, 7, 6, 20, 10]`
3. Output: `[5, 3, 7, 6, 20, 10]`
   - Checking: $5 \ge 3 \le 7 \ge 6 \le 20 \ge 10$. Valid wave!

### Complexity Analysis
- **Time Complexity:** $O(N \log N)$ due to sorting.
- **Space Complexity:** $O(1)$ (or $O(\log N)$ for sorting stack space) if sorted in-place.

### Pros and Cons
- **Pros:** Extremely simple to implement; works correctly even if the interviewer demands the lexicographically smallest wave array (assuming sorted input swap style).
- **Cons:** Sub-optimal time complexity of $O(N \log N)$.

### C++17 Brute Force Code
```cpp
#include <vector>
#include <algorithm>

class WaveArrayBrute {
public:
    void convertToWave(std::vector<int>& arr) {
        int n = arr.size();
        // Step 1: Sort the array
        std::sort(arr.begin(), arr.end());
        
        // Step 2: Swap adjacent elements at even positions
        for (int i = 0; i < n - 1; i += 2) {
            std::swap(arr[i], arr[i + 1]);
        }
    }
};
```

---

## 6. Better Solution
*Note: For this problem, there is no intermediate "better" solution. The transition goes directly from the $O(N \log N)$ sorting approach to the optimal $O(N)$ single-pass greedy approach.*

---

## 7. Optimal Solution

### Algorithm
1. Traverse the array at even positions: $i = 0, 2, 4, \dots, N-1$.
2. For each even index $i$:
   - If there is a left neighbor ($i > 0$) and $arr[i] < arr[i-1]$, swap $arr[i]$ and $arr[i-1]$.
   - If there is a right neighbor ($i < N-1$) and $arr[i] < arr[i+1]$, swap $arr[i]$ and $arr[i+1]$.
3. Since we only visit even indices, the step size is 2, ensuring $O(N)$ time.

### Dry Run
Input: `arr = [10, 90, 49, 2, 1, 5, 23]`

| Step | Index `i` | Current Array | Action / Logic |
| :--- | :--- | :--- | :--- |
| Start | - | `[10, 90, 49, 2, 1, 5, 23]` | Initial state |
| 1 | `i = 0` | `[10, 90, 49, 2, 1, 5, 23]` | Check right: `arr[0] (10) < arr[1] (90)`. Swap! |
| | | `[90, 10, 49, 2, 1, 5, 23]` | Array updated. |
| 2 | `i = 2` | `[90, 10, 49, 2, 1, 5, 23]` | Check left: `arr[2] (49) >= arr[1] (10)`. No swap. <br> Check right: `arr[2] (49) >= arr[3] (2)`. No swap. |
| 3 | `i = 4` | `[90, 10, 49, 2, 1, 5, 23]` | Check left: `arr[4] (1) < arr[3] (2)`. Swap! |
| | | `[90, 10, 49, 1, 2, 5, 23]` | Array updated. <br> Check right: `arr[4] (2) < arr[5] (5)`. Swap! |
| | | `[90, 10, 49, 1, 5, 2, 23]` | Array updated. |
| 4 | `i = 6` | `[90, 10, 49, 1, 5, 2, 23]` | Check left: `arr[6] (23) >= arr[5] (2)`. No swap. |

Final Output: `[90, 10, 49, 1, 5, 2, 23]`
Checking relations: $90 \ge 10 \le 49 \ge 1 \le 5 \ge 2 \le 23$. Valid wave!

### Why Optimal
- We visit each element a constant number of times (at most twice: once as a center, and potentially once as a neighbor).
- No additional space is allocated.
- Lower bound for checking an array of size $N$ is $\Omega(N)$ because every element must be inspected at least once. Hence, $O(N)$ is optimal.

### C++17 Optimal Code
```cpp
#include <vector>
#include <utility>

class WaveArrayOptimal {
public:
    void convertToWave(std::vector<int>& arr) {
        int n = arr.size();
        
        // Traverse all even indices
        for (int i = 0; i < n; i += 2) {
            // If current even element is smaller than its left neighbor, swap them
            if (i > 0 && arr[i] < arr[i - 1]) {
                std::swap(arr[i], arr[i - 1]);
            }
            
            // If current even element is smaller than its right neighbor, swap them
            if (i < n - 1 && arr[i] < arr[i + 1]) {
                std::swap(arr[i], arr[i + 1]);
            }
        }
    }
};
```

---

## 8. Code Walkthrough
- **Line 7:** Loop iterates from `i = 0` up to `n-1` with step `i += 2`, ensuring we only process even indices (the peaks).
- **Line 9-11:** If `i > 0` (left neighbor exists) and the peak element `arr[i]` is less than its left neighbor `arr[i-1]`, swap them to restore the property `arr[i] >= arr[i-1]`.
- **Line 14-16:** If `i < n-1` (right neighbor exists) and the peak element `arr[i]` is less than its right neighbor `arr[i+1]`, swap them to restore the property `arr[i] >= arr[i+1]`.
- All modifications happen in-place using standard utility function `std::swap`.

---

## 9. Dry Run
Input: `[4, 3, 2, 7, 8, 9]`, Size $N = 6$.

| Step | `i` | Condition `i > 0 && arr[i] < arr[i-1]` | Condition `i < n-1 && arr[i] < arr[i+1]` | Array State |
| :--- | :--- | :--- | :--- | :--- |
| Start | - | - | - | `[4, 3, 2, 7, 8, 9]` |
| 1 | 0 | False (`i == 0`) | False (`4 >= 3`) | `[4, 3, 2, 7, 8, 9]` |
| 2 | 2 | True (`2 < 3`) $\to$ Swap(2, 3) | - | `[4, 2, 3, 7, 8, 9]` |
| | | - | True (`3 < 7`) $\to$ Swap(3, 7) | `[4, 2, 7, 3, 8, 9]` |
| 3 | 4 | True (`8 < 3` is False) | True (`8 < 9`) $\to$ Swap(8, 9) | `[4, 2, 7, 3, 9, 8]` |

Final Output: `[4, 2, 7, 3, 9, 8]`
Verification: $4 \ge 2 \le 7 \ge 3 \le 9 \ge 8$. Valid!

---

## 10. Complexity Analysis
| Approach | Time Complexity | Auxiliary Space Complexity | Reasoning |
| :--- | :--- | :--- | :--- |
| **Brute Force (Sort)** | $O(N \log N)$ | $O(1)$ | Sorting the array takes $O(N \log N)$ time, swapping takes $O(N)$. |
| **Optimal (Greedy)** | $O(N)$ | $O(1)$ | Single pass over even indices. Constant number of comparisons/swaps per index. |

---

## 11. Common Mistakes
1. **Wrong step size:** Incrementing by `i += 1` instead of `i += 2`. Doing a single step checks every index as a peak, which ruins the valley condition at odd indices.
2. **Out of bounds access:** Forgetting to check if `i > 0` before accessing `arr[i-1]` or `i < n-1` before accessing `arr[i+1]`.
3. **Lexicographical confusion:** Assuming the output must be sorted in some way. Always clarify if sorting is needed. If the output needs to be the lexicographically smallest wave array, the sorting approach is mandatory.

---

## 12. Edge Cases
| Edge Case | Input Example | Expected Output | Why it Matters |
| :--- | :--- | :--- | :--- |
| **Single element** | `[1]` | `[1]` | Loop doesn't execute; terminates immediately without crash. |
| **Two elements** | `[1, 5]` | `[5, 1]` | `i = 0` checks `arr[0] < arr[1]` and swaps. |
| **Already sorted (ascending)** | `[1, 2, 3, 4]` | `[2, 1, 4, 3]` | Tests correctness of peak swaps. |
| **Already sorted (descending)** | `[4, 3, 2, 1]` | `[4, 3, 2, 1]` | No swaps needed; validates logic. |
| **All elements equal** | `[2, 2, 2]` | `[2, 2, 2]` | Non-strict comparison ($\ge, \le$) prevents infinite loops or needless swaps. |

---

## 13. Interview Explanation
> "To convert an array into a wave array where even index elements are peaks and odd index elements are valleys, we don't need a fully sorted array. We can solve this in a single linear scan.
> We iterate through all the even indices of the array. At each even index `i`, we check if it is smaller than its left neighbor or its right neighbor. If it is smaller than either of them, we swap the elements to make `i` the local maximum.
> This local correction is guaranteed to produce a globally valid wave because swapping a peak element with its neighbor only decreases the value at the neighbor's position, which preserves the wave condition at any previously processed elements. This greedy approach runs in $O(N)$ time and uses $O(1)$ auxiliary space."

---

## 14. Follow-up Questions
1. **What if we want $arr[0] \le arr[1] \ge arr[2] \le arr[3] \dots$ instead?**
   - *Answer:* Simply run the same algorithm focusing on odd indices as peaks instead of even indices, or swap relations.
2. **How to find the lexicographically smallest wave array?**
   - *Answer:* Sort the array first, then swap adjacent elements at even positions (the Brute Force method).
3. **Can we do this if we are reading a stream of numbers where we can't look ahead?**
   - *Answer:* Yes, we can buffer 3 elements at a time to determine the local peak and output them as a wave.

---

## 15. Variations
1. **Wave Array II (Strict inequalities):**
   - If the elements are unique and we need strict inequalities, the same algorithm works since elements are swapped to establish strict ordering. If duplicates exist, strict inequalities might be mathematically impossible (e.g., `[2, 2, 2]`).
2. **Sort in wave form with minimum swaps:**
   - Finding the minimum number of swaps to achieve a wave form from an arbitrary unsorted array. This uses dynamic programming or coordinate compression with BIT/Segment trees.

---

## 16. Revision Notes
- **Peak Rule:** Focus on even indices and ensure they are larger than their neighbors.
- **Transitivity:** Swapping `arr[i]` with `arr[i-1]` reduces `arr[i-1]`, preserving the peak at `arr[i-2]`.
- **Complexity:** $O(N)$ time, $O(1)$ space.

---

## 17. Memorization Version
- Loop through the array with `i` from `0` to `n-1` incrementing by `2`.
- If `i > 0` and `arr[i] < arr[i-1]`, swap.
- If `i < n-1` and `arr[i] < arr[i+1]`, swap.
- Time: $O(N)$, Space: $O(1)$. Correctness proven by transitivity of inequalities.
