# Maximum Length Bitonic Subarray - Comprehensive Interview Preparation Module

## 1. Problem Statement
Given an array `arr` of $N$ integers, find the length of the **longest bitonic subarray**. 

A subarray is **bitonic** if it is first sorted in non-decreasing order, then sorted in non-increasing order. 
*Note:* A subarray that is only non-decreasing or only non-increasing is also considered bitonic (with the decreasing or increasing part having length 0).

### Input
- An integer array `arr` of size $N$ ($1 \le N \le 10^6$).
- Element values: $-10^9 \le arr[i] \le 10^9$.

### Output
- An integer representing the length of the longest bitonic subarray.

### Constraints
- Time Complexity: $O(N)$
- Auxiliary Space: $O(N)$ (Better: optimize to $O(1)$ space).

### Observations
1. A bitonic subarray has a single peak element at some index `i`.
2. To the left of `i` (within the subarray), elements must be non-decreasing: $arr[k-1] \le arr[k]$ for all $k \le i$.
3. To the right of `i` (within the subarray), elements must be non-increasing: $arr[k] \ge arr[k+1]$ for all $k \ge i$.
4. The peak `i` is counted in both the increasing part and the decreasing part.

---

## 2. Interview Clarification
Before coding, a candidate should clarify:
1. **Are the inequalities strict?** (i.e. strictly increasing then strictly decreasing)
   - *Answer:* The standard definition of a bitonic subarray is non-decreasing followed by non-increasing. So duplicate adjacent elements are allowed to continue the sequence.
2. **Can a purely increasing or purely decreasing subarray be bitonic?**
   - *Answer:* Yes, a subarray that is entirely increasing or entirely decreasing is considered bitonic.
3. **What is the minimum possible return value?**
   - *Answer:* For any non-empty array, the minimum length of a bitonic subarray is 1 (a single element).

---

## 3. Thinking Process
Let's build the intuition step-by-step:
1. **First Instinct (Brute Force):**
   - For every element $i$ in the array, let's treat it as the peak.
   - Expand leftwards to find how far the non-decreasing sequence goes.
   - Expand rightwards to find how far the non-increasing sequence goes.
   - The length for peak $i$ is `left_len + right_len + 1`.
   - Take the maximum length over all $i$.
   - This takes $O(N^2)$ time in the worst case (e.g. if the array itself is sorted or bitonic).

2. **Optimal Intuition (Precomputation / DP):**
   - In the brute force, we re-evaluate the increasing/decreasing lengths from scratch for each index. We can optimize this by precomputing these lengths.
   - Let's define two arrays:
     - `inc[i]`: Length of the longest non-decreasing subarray ending at index `i`.
     - `dec[i]`: Length of the longest non-increasing subarray starting at index `i`.
   - We can build `inc` from left to right:
     - `inc[0] = 1`
     - For $i > 0$: `inc[i] = (arr[i] >= arr[i-1]) ? inc[i-1] + 1 : 1`
   - We can build `dec` from right to left:
     - `dec[N-1] = 1`
     - For $i < N-1$: `dec[i] = (arr[i] >= arr[i+1]) ? dec[i+1] + 1 : 1`
   - Once both are computed, the longest bitonic subarray peaking at index $i$ has length:
     $$\text{Length}(i) = inc[i] + dec[i] - 1$$
   - The overall answer is $\max_{0 \le i < N} (inc[i] + dec[i] - 1)$.
   - This approach runs in $O(N)$ time and uses $O(N)$ space.

3. **Can we optimize Space to $O(1)$?**
   - Yes! Instead of storing full `inc` and `dec` arrays, we can scan the array and compute the bitonic subarray lengths on-the-fly.
   - We can maintain a rolling peak and track how long we've been increasing and decreasing. We'll detail this in the Optimal Solution section.

---

## 4. Pattern Recognition
- **Topic:** Dynamic Programming / Precomputation / Two Pointers
- **Pattern:** Peak-Valley analysis / Subarray Properties
- **Why this pattern?** A bitonic subarray is composed of two independent properties (increasing from left, decreasing from right) that meet at a common peak index. Breaking the problem down to analyze the peak enables precomputation of both properties.

---

## 5. Brute Force Solution

### Algorithm
1. Initialize `max_len = 1`.
2. Loop through each index $i$ from $0$ to $N-1$:
   - Set `len = 1`.
   - Traverse left from $i$ (using index $j = i-1$ down to 0) as long as `arr[j] <= arr[j+1]`. Increment `len`.
   - Traverse right from $i$ (using index $k = i+1$ up to $N-1$) as long as `arr[k] <= arr[k-1]`. Increment `len`.
   - Update `max_len = max(max_len, len)`.
3. Return `max_len`.

### Complexity Analysis
- **Time Complexity:** $O(N^2)$ in the worst case (e.g. `[1, 2, 3, 4, 5]`).
- **Space Complexity:** $O(1)$ auxiliary space.

### C++17 Brute Force Code
```cpp
#include <vector>
#include <algorithm>

class BitonicSubarrayBrute {
public:
    int maxBitonicLength(const std::vector<int>& arr) {
        int n = arr.size();
        if (n == 0) return 0;
        
        int max_len = 1;
        
        for (int i = 0; i < n; ++i) {
            int len = 1;
            
            // Expand left
            int j = i - 1;
            while (j >= 0 && arr[j] <= arr[j + 1]) {
                len++;
                j--;
            }
            
            // Expand right
            int k = i + 1;
            while (k < n && arr[k] <= arr[k - 1]) {
                len++;
                k++;
            }
            
            max_len = std::max(max_len, len);
        }
        
        return max_len;
    }
};
```

---

## 6. Better Solution (Precomputation - $O(N)$ Time, $O(N)$ Space)

### Algorithm
1. Initialize arrays `inc` and `dec` of size $N$ with 1s.
2. Traverse from left to right (from $1$ to $N-1$):
   - If `arr[i] >= arr[i-1]`, set `inc[i] = inc[i-1] + 1`.
3. Traverse from right to left (from $N-2$ down to 0):
   - If `arr[i] >= arr[i+1]`, set `dec[i] = dec[i+1] + 1`.
4. Traverse the array and compute `inc[i] + dec[i] - 1` for each index.
5. Return the maximum value found.

### Dry Run
Input: `arr = [12, 4, 78, 90, 45, 23]`

- `inc` array construction:
  - `inc[0] = 1`
  - `arr[1] (4) < arr[0] (12)` $\to$ `inc[1] = 1`
  - `arr[2] (78) >= arr[1] (4)` $\to$ `inc[2] = inc[1] + 1 = 2`
  - `arr[3] (90) >= arr[2] (78)` $\to$ `inc[3] = inc[2] + 1 = 3`
  - `arr[4] (45) < arr[3] (90)` $\to$ `inc[4] = 1`
  - `arr[5] (23) < arr[4] (45)` $\to$ `inc[5] = 1`
  - `inc = [1, 1, 2, 3, 1, 1]`

- `dec` array construction:
  - `dec[5] = 1`
  - `arr[4] (45) >= arr[5] (23)` $\to$ `dec[4] = dec[5] + 1 = 2`
  - `arr[3] (90) >= arr[4] (45)` $\to$ `dec[3] = dec[4] + 1 = 3`
  - `arr[2] (78) < arr[3] (90)` $\to$ `dec[2] = 1`
  - `arr[1] (4) < arr[2] (78)` $\to$ `dec[1] = 1`
  - `arr[0] (12) >= arr[1] (4)` $\to$ `dec[0] = dec[1] + 1 = 2`
  - `dec = [2, 1, 1, 3, 2, 1]`

- Calculate Bitonic Length `inc[i] + dec[i] - 1`:
  - `i = 0`: $1 + 2 - 1 = 2$
  - `i = 1`: $1 + 1 - 1 = 1$
  - `i = 2`: $2 + 1 - 1 = 2$
  - `i = 3`: $3 + 3 - 1 = 5$
  - `i = 4`: $1 + 2 - 1 = 2$
  - `i = 5`: $1 + 1 - 1 = 1$
- Max value is 5 (for subarray `[4, 78, 90, 45, 23]`).

### C++17 Better Code
```cpp
#include <vector>
#include <algorithm>

class BitonicSubarrayBetter {
public:
    int maxBitonicLength(const std::vector<int>& arr) {
        int n = arr.size();
        if (n == 0) return 0;
        
        std::vector<int> inc(n, 1);
        std::vector<int> dec(n, 1);
        
        // Build increasing subarray lengths
        for (int i = 1; i < n; ++i) {
            if (arr[i] >= arr[i - 1]) {
                inc[i] = inc[i - 1] + 1;
            }
        }
        
        // Build decreasing subarray lengths
        for (int i = n - 2; i >= 0; --i) {
            if (arr[i] >= arr[i + 1]) {
                dec[i] = dec[i + 1] + 1;
            }
        }
        
        // Find max length
        int max_len = 1;
        for (int i = 0; i < n; ++i) {
            max_len = std::max(max_len, inc[i] + dec[i] - 1);
        }
        
        return max_len;
    }
};
```

---

## 7. Optimal Solution (Single Pass - $O(N)$ Time, $O(1)$ Space)

### Algorithm
We can scan the array and count lengths on the fly:
1. Initialize `max_len = 1`, `inc_len = 1`, `dec_len = 0`.
2. We iterate through the array from $i = 1$ to $N-1$:
   - If `arr[i] > arr[i-1]`:
     - If we were in a decreasing phase (`dec_len > 0`), it means the previous bitonic subarray has ended. We calculate its length: `inc_len + dec_len`, update `max_len`, and reset `inc_len` to the count of the last increasing elements (which is 1 + the number of equal elements at the transition, but if it is strictly greater, it's just 1). Wait, how do we handle duplicates?
     - Let's track:
       - `inc_run`: length of current increasing run.
       - `dec_run`: length of current decreasing run.
       - `equal_run`: count of identical values at the peak or transition.
     - Let's use a simpler state machine to avoid complexity:
       - Keep track of `start` of the current bitonic subarray.
       - As we traverse, we can identify peak transitions.
       - Alternatively, we can find all local peak candidates in a single pass.
       - Let's design the one-pass algorithm using a direct pointer method:
         - A bitonic subarray is partitioned by peaks.
         - Let's traverse the array with a index pointer `i = 0`.
         - While `i < n-1`:
           - Find the end of the non-decreasing segment. This is the peak candidate.
           - Find the end of the non-increasing segment. This is the end of the bitonic subarray.
           - Calculate length. Update `max_len`.
           - The next bitonic subarray starts at the end of the decreasing segment? No, it could start earlier if the decreasing segment contains increasing subsegments.
           - Wait! The transition point from decreasing back to increasing is a valley. The next bitonic subarray can start at this valley.
           - So, if we find a valley, we start the next bitonic subarray from that valley.
           - Let's trace this:
             - An transition from decreasing to increasing occurs when `arr[i] < arr[i+1]` after we've been decreasing.
             - So the start of the next bitonic subarray is indeed that valley!
             - Let's write this transition logic clearly.

### Single-Pass Two-Pointer State Machine
1. Set `i = 0`, `max_len = 1`.
2. While `i < N-1`:
   - Keep track of the starting index: `start = i`.
   - **Step A (Non-decreasing phase):** Traverse as long as `arr[i] <= arr[i+1]`.
     - `i++`
   - **Step B (Non-increasing phase):** Traverse as long as `arr[i] >= arr[i+1]`.
     - Maintain a boolean `decreased = false` to know if we actually had a decreasing phase.
     - While `arr[i] >= arr[i+1]`, increment `i` and set `decreased = true`.
   - Update `max_len = max(max_len, i - start + 1)`.
   - If we finished a step where we did not decrease (e.g. we reached end of array or we hit a flat part), we must advance `i` to avoid infinite loop. But wait! If we didn't decrease, `i` has moved forward during Step A.
   - What is the starting point of the next bitonic subarray?
     - If we did a decreasing phase, the next subarray could start at the valley. The valley is exactly at the index `i` we reached! So the next iteration starts with `start = i` (meaning we don't change `i`).
     - But what if there are duplicates?
       - If `arr[i] == arr[i+1]`, it is allowed in both non-decreasing and non-increasing.
       - Let's test this logic on `[10, 20, 30, 40]`:
         - `start = 0`.
         - Step A: `i` increments to 3 (`arr[3] = 40`).
         - Step B: `arr[3] >= arr[4]` is false. Loop doesn't run.
         - `max_len = max(1, 3 - 0 + 1) = 4`.
         - Loop ends since `i = 3` (which is `n-1`). Correct!
       - Let's test on `[40, 30, 20, 10]`:
         - `start = 0`.
         - Step A: `arr[0] <= arr[1]` false. Loop doesn't run.
         - Step B: `i` increments to 3.
         - `max_len = max(1, 3 - 0 + 1) = 4`. Correct!
       - Let's test on `[10, 20, 30, 20, 10, 40, 30]`:
         - `start = 0`.
         - Step A: `i` goes to 2 (`arr[2] = 30`).
         - Step B: `i` goes to 4 (`arr[4] = 10`).
         - `max_len = max(1, 4 - 0 + 1) = 5`.
         - Next loop starts at `i = 4`.
         - `start = 4`.
         - Step A: `i` goes to 5 (`arr[5] = 40`).
         - Step B: `i` goes to 6 (`arr[6] = 30`).
         - `max_len = max(5, 6 - 4 + 1) = 5`. Correct!
       - Wait! What if there is a flat plateau? E.g., `[10, 20, 20, 10]`
         - `start = 0`.
         - Step A: `i` goes to 2 (`arr[2] = 20`).
         - Step B: `arr[2] >= arr[3]` (20 >= 10) $\to$ `i` goes to 3 (`arr[3] = 10`).
         - `max_len = max(1, 4) = 4`. Correct!
       - What if `[10, 20, 10, 10, 20]`?
         - `start = 0`.
         - Step A: `i` goes to 1.
         - Step B: `i` goes to 3.
         - `max_len = 4` (`[10, 20, 10, 10]`).
         - Next loop starts at `i = 3`.
         - `start = 3`.
         - Step A: `i` goes to 4.
         - `max_len = max(4, 4 - 3 + 1) = 4`. Correct!

Wait, is there any case where the next bitonic subarray should start *before* the current `i`?
Suppose `arr = [10, 22, 9, 33, 21, 50, 41, 60]`.
- Iteration 1:
  - `start = 0`.
  - Step A: `i` goes to 1 (`22`).
  - Step B: `i` goes to 2 (`9`).
  - `max_len = 3` (`[10, 22, 9]`).
  - Next search starts at `i = 2` (`9`).
- Iteration 2:
  - `start = 2`.
  - Step A: `i` goes to 3 (`33`).
  - Step B: `i` goes to 4 (`21`).
  - `max_len = max(3, 3) = 3`.
  - Next search starts at `i = 4` (`21`).
- Iteration 3:
  - `start = 4`.
  - Step A: `i` goes to 5 (`50`).
  - Step B: `i` goes to 6 (`41`).
  - `max_len = 3`.
  - Next search starts at `i = 6` (`41`).
- Iteration 4:
  - `start = 6`.
  - Step A: `i` goes to 7 (`60`).
  - Step B: no run.
  - `max_len = 3`.
All bitonic subarrays found are size 3. The logic is completely correct and runs in $O(N)$ time with $O(1)$ auxiliary space!

Let's double check if we need to do anything when `arr[i] > arr[i+1]` is not met in Step B.
Wait, if `arr[i] < arr[i+1]` at the end of Step B, we don't increment `i` manually because `i` has already advanced during Step A or Step B.
But what if `i` didn't advance at all?
If `arr[i] > arr[i+1]` at the start (meaning we cannot increase), then Step A doesn't run, but Step B will run and increment `i`.
What if `arr[i] == arr[i+1]`?
Then Step A runs and increments `i`.
Thus, `i` is guaranteed to advance by at least 1 in every outer loop iteration, preventing infinite loops.

Let's write this beautiful $O(1)$ space solution.

### C++17 Optimal Code
```cpp
#include <vector>
#include <algorithm>

class BitonicSubarrayOptimal {
public:
    int maxBitonicLength(const std::vector<int>& arr) {
        int n = arr.size();
        if (n <= 1) return n;
        
        int max_len = 1;
        int i = 0;
        
        while (i < n - 1) {
            int start = i;
            
            // Step A: Run through non-decreasing elements
            while (i < n - 1 && arr[i] <= arr[i + 1]) {
                i++;
            }
            
            // Step B: Run through non-increasing elements
            while (i < n - 1 && arr[i] >= arr[i + 1]) {
                i++;
            }
            
            // Update the maximum length found so far
            max_len = std::max(max_len, i - start + 1);
        }
        
        return max_len;
    }
};
```

---

## 8. Code Walkthrough
- **Line 7:** Handle edge cases where $N \le 1$.
- **Line 12:** Outer loop runs while the index `i` is less than `n - 1`.
- **Line 13:** Remember the start index of the current bitonic subarray candidate.
- **Line 16-18:** Traverse the non-decreasing phase of the wave. The index `i` stops at the local peak.
- **Line 21-23:** Traverse the non-increasing phase of the wave. The index `i` stops at the local valley.
- **Line 26:** The length of this bitonic subarray is `i - start + 1`. We update `max_len` with it.
- Since we don't decrement `i` and only advance, the time complexity is strictly linear.

---

## 9. Dry Run
Input: `[12, 4, 78, 90, 45, 23]`

| Iteration | `start` | Loop A condition (non-decreasing) | `i` after A | Loop B condition (non-increasing) | `i` after B | `max_len` update |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **1** | 0 | `12 <= 4` (False) | 0 | `12 >= 4` (True) $\to$ `i` increments | 1 | $\max(1, 1 - 0 + 1) = 2$ |
| **2** | 1 | `4 <= 78` (True) $\to$ `78 <= 90` (True) $\to$ `90 <= 45` (False) | 3 | `90 >= 45` (True) $\to$ `45 >= 23` (True) $\to$ end of array | 5 | $\max(2, 5 - 1 + 1) = 5$ |

Output: 5.

---

## 10. Complexity Analysis
| Approach | Time Complexity | Auxiliary Space Complexity | Reasoning |
| :--- | :--- | :--- | :--- |
| **Brute Force** | $O(N^2)$ | $O(1)$ | Expands left and right for every element in the worst case. |
| **Precomputation (Better)** | $O(N)$ | $O(N)$ | Uses two extra arrays of size $N$ for precomputing counts. |
| **Two-Pointer (Optimal)** | $O(N)$ | $O(1)$ | Uses a single pass with index tracking, no extra arrays. |

---

## 11. Common Mistakes
1. **Wrong peak transition:** Assuming that the next bitonic subarray must start at the peak `i` instead of the valley. Valley-to-valley is correct.
2. **Infinite loop on duplicates:** Not ensuring the index `i` increments at least once per loop when elements are duplicate. The outer loop guard and inner logic handle this.
3. **Parity of inequalities:** Using strict `<` and `>` when the problem definition allows non-decreasing and non-increasing ($\le$ and $\ge$).

---

## 12. Edge Cases
| Edge Case | Input Example | Expected Output | Why it Matters |
| :--- | :--- | :--- | :--- |
| **Strictly increasing** | `[1, 2, 3]` | `3` | Step A executes completely; Step B doesn't. |
| **Strictly decreasing** | `[3, 2, 1]` | `3` | Step A doesn't execute; Step B executes completely. |
| **All equal elements** | `[5, 5, 5]` | `3` | Equal elements count as bitonic. |
| **Alternating peaks/valleys** | `[1, 3, 2, 4, 3]` | `3` | Verifies multiple sub-waves are captured. |

---

## 13. Interview Explanation
> "To find the maximum length bitonic subarray in linear time and constant space, we can scan the array from left to right using a two-pointer approach.
> A bitonic subarray consists of a non-decreasing phase followed by a non-increasing phase.
> We start at index `0` and find the end of the non-decreasing segment. This gives us the local peak. From this peak, we then find the end of the non-increasing segment, which gives us the local valley.
> The length of this bitonic subarray is the difference between the valley and start indices plus one. We record the maximum length.
> Since the next bitonic subarray can start at this valley, we set the next starting point to the valley index and repeat the process. This scans the array in a single pass, resulting in $O(N)$ time complexity and $O(1)$ auxiliary space."

---

## 14. Follow-up Questions
1. **What if the subarray must be strictly increasing and then strictly decreasing?**
   - *Answer:* We modify the conditions in Step A to `arr[i] < arr[i+1]` and Step B to `arr[i] > arr[i+1]`.
2. **Can we find the longest bitonic subsequence instead of subarray?**
   - *Answer:* Yes, but that is a different problem solved using Dynamic Programming (Longest Increasing Subsequence from left and right) in $O(N \log N)$ or $O(N^2)$ time.

---

## 15. Variations
1. **Longest Mountain Subarray:**
   - Similar to bitonic, but requires strict increase followed by strict decrease, and the peak cannot be at the boundaries (both increasing and decreasing lengths must be $\ge 1$). We will cover this in Q48.
2. **Minimum Swaps to make Array Bitonic:**
   - Involves finding inversion counts or DP, which is a much harder problem.

---

## 16. Revision Notes
- **Peak-Valley scan:** Find peak, then find valley, calculate length, start next scan from valley.
- **Complexity:** $O(N)$ time, $O(1)$ space.
- **Non-decreasing/Non-increasing:** Use `<=` and `>=`.

---

## 17. Memorization Version
- Start at `i = 0`.
- While `i < n-1`:
  - `start = i`.
  - While `arr[i] <= arr[i+1]`, `i++`.
  - While `arr[i] >= arr[i+1]`, `i++`.
  - `max_len = max(max_len, i - start + 1)`.
- Time: $O(N)$, Space: $O(1)$.
