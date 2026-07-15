# Longest Mountain Subarray - Comprehensive Interview Preparation Module

## 1. Problem Statement
Given an integer array `arr`, return the length of the **longest subarray** that is a **mountain**. If no mountain subarray exists, return `0`.

A subarray `arr[i...j]` (of length $\ge 3$) is a **mountain** if:
1. There exists some index `k` with $i < k < j$ such that:
   - $arr[i] < arr[i+1] < \dots < arr[k-1] < arr[k]$ (strictly increasing)
   - $arr[k] > arr[k+1] > \dots > arr[j-1] > arr[j]$ (strictly decreasing)

### Input
- An integer array `arr` of size $N$ ($1 \le N \le 10^4$).
- Element values: $0 \le arr[i] \le 10^4$.

### Output
- An integer representing the length of the longest mountain subarray.

### Constraints
- Time Complexity: $O(N)$
- Auxiliary Space: $O(1)$ optimal ($O(N)$ acceptable as intermediate).

### Observations
1. A mountain must have at least 3 elements.
2. The peak of the mountain cannot be the first or last element of the subarray (i.e. we need both a non-empty increasing phase and a non-empty decreasing phase).
3. The transitions must be **strictly** increasing and **strictly** decreasing (no duplicates allowed inside the slopes).

---

## 2. Interview Clarification
Before coding, a candidate should clarify:
1. **Can mountain slopes contain equal adjacent elements?**
   - *Answer:* No, the problem specifies *strictly* increasing and decreasing. Any flat surface (e.g. `[1, 2, 2, 1]`) breaks the mountain.
2. **What if the array is entirely sorted?**
   - *Answer:* Since we need both an increasing and a decreasing phase, an entirely sorted array (ascending or descending) has no mountains and should return `0`.
3. **What is the minimum size of a valid mountain?**
   - *Answer:* The minimum size is `3` (e.g., `[1, 2, 1]`).

---

## 3. Thinking Process
Let's build the intuition step-by-step:

### Method 1: Two-Pass Dynamic Programming ($O(N)$ Time, $O(N)$ Space)
- For every index $i$ in the array, we can check if it can serve as a mountain peak.
- A peak at $i$ needs:
  - An increasing run ending at $i$. Let's precompute `left[i]`: length of strictly increasing subarray ending at `i`.
  - A decreasing run starting at $i$. Let's precompute `right[i]`: length of strictly decreasing subarray starting at `i`.
- We can populate `left` from left to right:
  - `left[0] = 0`
  - `left[i] = (arr[i] > arr[i-1]) ? left[i-1] + 1 : 0`
- We can populate `right` from right to left:
  - `right[N-1] = 0`
  - `right[i] = (arr[i] > arr[i+1]) ? right[i+1] + 1 : 0`
- A valid peak is any index $i$ where `left[i] > 0` and `right[i] > 0`.
- The mountain length peaking at $i$ is `left[i] + right[i] + 1`.
- We take the maximum of this length.

### Method 2: Two-Pointer / Sliding Window ($O(N)$ Time, $O(1)$ Space)
- Instead of using $O(N)$ extra space to store the prefix and suffix lengths, we can identify mountains on-the-fly.
- We start a pointer `base = 0`.
- While `base` is a valid starting point for a mountain:
  - We set `end = base`.
  - **Check for increasing phase:** If `end + 1 < N` and `arr[end] < arr[end+1]`:
    - Move `end` forward as long as `arr[end] < arr[end+1]`.
    - Once we stop, `end` is at a candidate peak.
    - **Check for decreasing phase:** If `end + 1 < N` and `arr[end] > arr[end+1]` (valid peak check):
      - Move `end` forward as long as `arr[end] > arr[end+1]`.
      - Now `end` represents the valley at the end of the mountain.
      - Calculate the mountain length: `end - base + 1`. Update `max_len`.
      - Since the next mountain can start at this valley, we set `base = end`.
    - If there was no decreasing phase, `end` was not a valid peak. We can set `base = end` (since any new mountain starting before `end` would have to cross this peak, which is impossible as the slope didn't decrease).
  - If we couldn't even start an increasing phase (i.e. `arr[base] >= arr[base+1]`), we increment `base` by 1.

This state machine checks every element at most twice (once as part of ascending, once as part of descending), yielding $O(N)$ time and $O(1)$ space.

---

## 4. Pattern Recognition
- **Topic:** Two Pointers / Dynamic Programming
- **Pattern:** Peak-Valley Identification / State Machine
- **Why this pattern?** A mountain has distinct stages (Ascend $\to$ Peak $\to$ Descend). Tracking these stages using pointers or DP allows us to split the global search into local state transitions.

---

## 5. Brute Force Solution

### Algorithm
1. Initialize `max_len = 0`.
2. For each element $i$ from $1$ to $N-2$:
   - Check if $i$ is a peak: `arr[i] > arr[i-1] && arr[i] > arr[i+1]`.
   - If it is, expand leftwards as long as `arr[left] > arr[left-1]` and rightwards as long as `arr[right] > arr[right+1]`.
   - Length of mountain $= right - left + 1$.
   - Update `max_len = max(max_len, length)`.
3. Return `max_len`.

### Complexity Analysis
- **Time Complexity:** $O(N^2)$ in the worst case (e.g. alternating zig-zag array).
- **Space Complexity:** $O(1)$ auxiliary space.

### C++17 Brute Force Code
```cpp
#include <vector>
#include <algorithm>

class MountainBrute {
public:
    int longestMountain(std::vector<int>& arr) {
        int n = arr.size();
        int max_len = 0;
        
        for (int i = 1; i < n - 1; ++i) {
            // Check if arr[i] is a peak
            if (arr[i] > arr[i - 1] && arr[i] > arr[i + 1]) {
                int left = i;
                int right = i;
                
                // Expand left
                while (left > 0 && arr[left] > arr[left - 1]) {
                    left--;
                }
                
                // Expand right
                while (right < n - 1 && arr[right] > arr[right + 1]) {
                    right++;
                }
                
                max_len = std::max(max_len, right - left + 1);
            }
        }
        
        return max_len;
    }
};
```

---

## 6. Better Solution (Two-Pass DP - $O(N)$ Time, $O(N)$ Space)

### Algorithm
1. Initialize `left` and `right` arrays of size $N$ with 0.
2. Build `left` array from index $1$ to $N-1$:
   - If `arr[i] > arr[i-1]`, set `left[i] = left[i-1] + 1`.
3. Build `right` array from index $N-2$ down to 0:
   - If `arr[i] > arr[i+1]`, set `right[i] = right[i+1] + 1`.
4. Scan all indices $i$ from $1$ to $N-2$:
   - If `left[i] > 0` and `right[i] > 0`, calculate `left[i] + right[i] + 1` and update `max_len`.
5. Return `max_len`.

### C++17 Better Code
```cpp
#include <vector>
#include <algorithm>

class MountainBetter {
public:
    int longestMountain(std::vector<int>& arr) {
        int n = arr.size();
        if (n < 3) return 0;
        
        std::vector<int> left(n, 0);
        std::vector<int> right(n, 0);
        
        // Build increasing slope counts ending at i
        for (int i = 1; i < n; ++i) {
            if (arr[i] > arr[i - 1]) {
                left[i] = left[i - 1] + 1;
            }
        }
        
        // Build decreasing slope counts starting at i
        for (int i = n - 2; i >= 0; --i) {
            if (arr[i] > arr[i + 1]) {
                right[i] = right[i + 1] + 1;
            }
        }
        
        int max_len = 0;
        for (int i = 1; i < n - 1; ++i) {
            if (left[i] > 0 && right[i] > 0) {
                max_len = std::max(max_len, left[i] + right[i] + 1);
            }
        }
        
        return max_len;
    }
};
```

---

## 7. Optimal Solution (Two-Pointer Single Pass - $O(N)$ Time, $O(1)$ Space)

### Algorithm
1. Initialize `max_len = 0` and `base = 0`.
2. While `base < N - 1`:
   - Set `end = base`.
   - **Step A:** If `end + 1 < N` and `arr[end] < arr[end + 1]`:
     - Move `end` forward while `end + 1 < N` and `arr[end] < arr[end + 1]`.
     - (We have now climbed to a peak).
     - **Step B:** If `end + 1 < N` and `arr[end] > arr[end + 1]`:
       - Move `end` forward while `end + 1 < N` and `arr[end] > arr[end + 1]`.
       - (We have now descended to the valley).
       - Update `max_len = max(max_len, end - base + 1)`.
       - Set `base = end` (the next mountain can start at the current valley).
     - Else:
       - If we didn't descend, `end` is not a valid peak. Set `base = end` (or `base = end + 1` if they are equal). To be safe, we can just set `base = end` and if it is equal, the next check will fail to increase and increment `base` by 1.
   - If we couldn't ascend at all (i.e. `arr[base] >= arr[base+1]`), increment `base` by 1.
3. Return `max_len`.

### Dry Run
Input: `arr = [2, 1, 4, 7, 3, 2, 5]`

| Iteration | `base` | Step A (Ascend) | Peak Index | Step B (Descend) | Valley Index | `max_len` Update | New `base` |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **1** | 0 | `2 < 1` (False) | - | - | - | - | `base++` $\to$ 1 |
| **2** | 1 | `1 < 4 < 7` | 3 | `7 > 3 > 2` | 5 | $\max(0, 5 - 1 + 1) = 5$ | `base = 5` |
| **3** | 5 | `2 < 5` | 6 | `5 > (end of array)` (False) | - | - | `base = 6` |
| **4** | 6 | `base = n - 1` $\to$ Loop Terminated | - | - | - | - | - |

Output: `5` (Subarray: `[1, 4, 7, 3, 2]`).

### Why Optimal
- We visit each element at most twice (once climbing up, once climbing down).
- No auxiliary memory is used.
- Lower bound is $\Omega(N)$ since we must examine all heights. Hence, $O(N)$ time and $O(1)$ space is optimal.

### C++17 Optimal Code
```cpp
#include <vector>
#include <algorithm>

class MountainOptimal {
public:
    int longestMountain(std::vector<int>& arr) {
        int n = arr.size();
        int max_len = 0;
        int base = 0;
        
        while (base < n - 1) {
            int end = base;
            
            // Step A: Climb the mountain (Strictly Increasing)
            if (end + 1 < n && arr[end] < arr[end + 1]) {
                while (end + 1 < n && arr[end] < arr[end + 1]) {
                    end++;
                }
                
                // Step B: Descend the mountain (Strictly Decreasing)
                if (end + 1 < n && arr[end] > arr[end + 1]) {
                    while (end + 1 < n && arr[end] > arr[end + 1]) {
                        end++;
                    }
                    // Valid mountain found! Update max_len
                    max_len = std::max(max_len, end - base + 1);
                    
                    // Next mountain can start from the valley
                    base = end;
                    continue;
                }
            }
            
            // If not a valid mountain, shift base forward
            base = std::max(base + 1, end);
        }
        
        return max_len;
    }
};
```

---

## 8. Code Walkthrough
- **Line 7:** Initialize `max_len = 0` and `base = 0`.
- **Line 9:** Run loop while the start of candidate mountain `base` is within bounds.
- **Line 13:** If `arr[base] < arr[base+1]`, start climbing.
- **Line 14-16:** Climb as high as possible. `end` stops at the peak.
- **Line 19:** If peak is higher than next element, start descending.
- **Line 20-22:** Descend as low as possible. `end` stops at the valley.
- **Line 24-27:** Record mountain length `end - base + 1` and jump `base` to `end`. We use `continue` to avoid the fallback increment.
- **Line 32:** If a valid mountain was not found, we increment `base` by either 1 or `end` to bypass the flat / invalid region.

---

## 9. Dry Run
Input: `[2, 2, 2]`, Size $N = 3$.
- `base = 0`: `arr[0] < arr[1]` (2 < 2) is False.
  - Falls to Line 32: `base = max(1, 0) = 1`.
- `base = 1`: `arr[1] < arr[2]` (2 < 2) is False.
  - Falls to Line 32: `base = max(2, 1) = 2`.
- Loop terminates since `base = 2` is not `< n-1` (which is 2).
Output: `0`. Correct!

---

## 10. Complexity Analysis
| Approach | Time Complexity | Auxiliary Space Complexity | Reasoning |
| :--- | :--- | :--- | :--- |
| **Brute Force** | $O(N^2)$ | $O(1)$ | Expands left and right from every potential peak. |
| **Two-Pass DP** | $O(N)$ | $O(N)$ | Uses two extra arrays of size $N$. |
| **Two-Pointer (Optimal)** | $O(N)$ | $O(1)$ | Single pass over array using sliding base pointer. |

---

## 11. Common Mistakes
1. **Allowing flat portions:** Not using strict inequality, which incorrectly classifies `[1, 2, 2, 1]` as a mountain.
2. **Missing decreasing phase check:** Forgetting that a mountain must go *down* as well as *up*. An array like `[1, 2, 3]` is not a mountain.
3. **Invalid peak boundaries:** Forgetting to check if `end + 1 < n` during pointer increments, leading to out-of-bounds errors.

---

## 12. Edge Cases
| Edge Case | Input Example | Expected Output | Why it Matters |
| :--- | :--- | :--- | :--- |
| **No peak (Increasing only)**| `[1, 2, 3]` | `0` | No decreasing slope. |
| **No peak (Decreasing only)**| `[3, 2, 1]` | `0` | No increasing slope. |
| **Flat plateaus** | `[1, 2, 2, 1]` | `0` | Strict inequalities prevent false peaks. |
| **Multiple mountains** | `[1, 3, 1, 4, 1]`| `3` | Validates that multiple peaks are compared. |
| **Small Array ($N < 3$)** | `[1, 2]` | `0` | Immediate rejection without crashing. |

---

## 13. Interview Explanation
> "To find the longest mountain subarray in $O(N)$ time and $O(1)$ space, we can use a two-pointer sliding window approach. 
> We traverse the array from left to right using a `base` pointer. At each position, we check if we can climb up, meaning the next element is strictly larger.
> If we can, we move our `end` pointer to climb to the local peak.
> From the peak, we check if we can descend, meaning the next element is strictly smaller.
> If both phases are valid, we have found a mountain. We measure its length and update our maximum. We then set our next `base` starting position to the valley we reached.
> If no valid mountain was formed, we slide our `base` pointer forward. This ensures we scan the array in a single pass without using any extra memory."

---

## 14. Follow-up Questions
1. **What if the array represents a circular terrain?**
   - *Answer:* We can concatenate the array with itself (or use modulo arithmetic) and find the mountain of max length $\le N$.
2. **Can we count the total number of mountains in the array?**
   - *Answer:* Yes, each peak with valid left and right slopes forms a unique maximal mountain, so we can count them during our single-pass traversal.

---

## 15. Variations
1. **Maximum Length Bitonic Subarray:**
   - As covered in Q46, allows duplicates and does not require peak to be strictly inside (only increasing or only decreasing is valid).
2. **Find Peak Element:**
   - Find any peak element in an unsorted array in $O(\log N)$ time.

---

## 16. Revision Notes
- **Mountain Criteria:** Strict increase, strict decrease, size $\ge 3$.
- **State transitions:** Ascend $\to$ Descend $\to$ Update $\to$ Jump.
- **Space Complexity:** $O(1)$ space is easily achievable with state-machine scanning.

---

## 17. Memorization Version
- Loop `base` from `0` to `n-2`.
- If `arr[base] < arr[base+1]`:
  - `end = base`.
  - While `arr[end] < arr[end+1]`, `end++`.
  - If `arr[end] > arr[end+1]`:
    - While `arr[end] > arr[end+1]`, `end++`.
    - `max_len = max(max_len, end - base + 1)`.
    - `base = end; continue;`
- Else `base = max(base+1, end)`.
- Time: $O(N)$, Space: $O(1)$.
