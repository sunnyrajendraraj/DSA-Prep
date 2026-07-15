# Binary Search on Answer – Minimum Max Subarray

---

## 1. Problem Statement

Given an array of integers `nums` and an integer `k`, you need to partition the array into `k` contiguous subarrays such that the **maximum sum of any subarray is minimized**. Return this minimized maximum sum.

This problem is also classically known as the **Book Allocation Problem** or the **Painter's Partition Problem**.

### Input
- `nums`: A vector of positive integers.
- `k`: An integer representing the number of contiguous subarrays to partition the array into.

### Output
- An integer representing the minimized maximum subarray sum.

### Constraints
- $1 \le \text{nums.length} \le 10^5$
- $1 \le \text{nums}[i] \le 10^5$
- $1 \le k \le \text{nums.length}$

### Key Observations
1. **Contiguous Subarrays:** The division must be sequential (no sorting or reordering of elements allowed).
2. **Monotonic Decision Space:**
   - If it is possible to partition the array such that no subarray exceeds a sum $S$, then it is definitely possible for any sum greater than $S$.
   - If it is impossible for sum $S$, then it is also impossible for any sum less than $S$.
   - This monotonic property enables **Binary Search on the Answer**.
3. **Subarray Limit ($k$):** If the array can be partitioned into $M$ subarrays (where $M \le k$) each having a sum $\le S$, then it can also be partitioned into exactly $k$ subarrays (since we can always split any existing subarray further without increasing the maximum sum).

---

## 2. Interview Clarification

Before coding, clarify these details:
1. **Are all elements in `nums` positive?** (Yes, standard versions assume positive numbers. If negative numbers exist, the monotonic property breaks down, and standard binary search on answer does not work).
2. **Can `k` be greater than the length of the array?** (Usually no, as we cannot partition $N$ elements into $>N$ non-empty contiguous subarrays. If $k > N$, return error or handle it as invalid).
3. **What is the range of individual elements and the sum?** (Sum can exceed the 32-bit signed integer limit, so use `long long` for sum calculations).
4. **Must the subarrays be non-empty?** (Yes).

---

## 3. Thinking Process

### First Instinct
To find the partition that minimizes the maximum subarray sum, we could try generating all possible partitions of the array into `k` subarrays, calculate the maximum subarray sum for each, and find the minimum. However, partition generation is a combinatorial problem requiring $O\left(\binom{N-1}{k-1}\right)$ time, which is completely infeasible for $N = 10^5$.

### Observations
Let's think about the possible value of our answer (let's call it $x$, representing the maximum subarray sum):
1. **What is the absolute minimum value $x$ could possibly take?**
   Even if $k = N$ (each element is in its own subarray), the maximum subarray sum will be the value of the largest single element in the array. Thus, $x \ge \max(\text{nums})$.
2. **What is the absolute maximum value $x$ could possibly take?**
   If $k = 1$ (the entire array is one subarray), the maximum subarray sum is the sum of all elements in the array. Thus, $x \le \sum(\text{nums})$.

So, the optimal answer $x$ must lie in the range:
$$[\max(\text{nums}), \sum(\text{nums})]$$

### The Binary Search on Answer Pattern
Since the search range is bounded and sorted (from $\max(\text{nums})$ to $\sum(\text{nums})$), we can binary search within this range!
For a candidate maximum sum `mid`:
- Write a helper function `isValid(mid)` to check: *Can we partition `nums` into $\le k$ subarrays such that no subarray sum exceeds `mid`?*
- If **yes** (`isValid(mid) == true`):
  - `mid` is a potential answer.
  - Since we want to **minimize** the maximum sum, we try to find an even smaller valid sum. We search the left half: `high = mid - 1`.
- If **no** (`isValid(mid) == false`):
  - `mid` is too small. We cannot partition the array under this limit.
  - We must allow a larger sum. We search the right half: `low = mid + 1`.

---

## 4. Pattern Recognition

This is a classic **Binary Search on Answer** problem.
- **Why this topic?**
  1. The target to find is not an element in the array, but a value from a continuous range of possible answers.
  2. Finding a direct partition is hard, but checking if a candidate answer is valid is easy (greedy simulation in $O(n)$ time).
  3. The viability of answers is monotonic: `[No, No, No, Yes, Yes, Yes]`. We want to find the transition point (the first `Yes`).

---

## 5. Brute Force Solution

### Algorithm
Linear scan through every possible answer from $\max(\text{nums})$ to $\sum(\text{nums})$. For each value, use the greedy check to see if it can form $\le k$ subarrays. The first value that succeeds is our minimum maximum subarray sum.

### Dry Run
`nums = [7, 2, 5, 10, 8]`, `k = 2`
- $\max(\text{nums}) = 10$, $\sum(\text{nums}) = 32$.
- Search range: $[10, 32]$
- Try $x = 10$:
  - Subarrays: `[7, 2]`, `[5]`, `[10]`, `[8]`. Total subarrays = 4. Since $4 > 2$, $x = 10$ is invalid.
- Try $x = 11, 12, \dots$
- Try $x = 18$:
  - Subarrays: `[7, 2, 5]` (sum 14), `[10, 8]` (sum 18). Total subarrays = 2. Valid! Return 18.

### Complexity Analysis
- **Time Complexity:** $O((\sum(\text{nums}) - \max(\text{nums})) \times n)$. In the worst case, this can be extremely slow if the sum is large.
- **Space Complexity:** $O(1)$ auxiliary space.

### Pros/Cons
- **Pros:** Conceptually simple.
- **Cons:** Too slow; times out for large array sums.

### Commented C++17 Code
```cpp
#include <vector>
#include <numeric>
#include <algorithm>

class BruteForce {
private:
    // Helper to check if a limit is feasible with <= k subarrays
    bool isValid(const std::vector<int>& nums, long long limit, int k) {
        long long current_sum = 0;
        int subarrays = 1;
        for (int num : nums) {
            if (current_sum + num > limit) {
                // Start a new subarray
                subarrays++;
                current_sum = num;
                if (subarrays > k) return false;
            } else {
                current_sum += num;
            }
        }
        return true;
    }

public:
    int splitArray(const std::vector<int>& nums, int k) {
        long long low = *std::max_element(nums.begin(), nums.end());
        long long high = std::accumulate(nums.begin(), nums.end(), 0LL);

        // Linear scan of the entire range
        for (long long limit = low; limit <= high; ++limit) {
            if (isValid(nums, limit, k)) {
                return static_cast<int>(limit);
            }
        }
        return -1;
    }
};
```

---

## 6. Better Solution

There is no intermediate "better" solution. The transition goes directly from brute-force search space scanning to optimal binary searching.

---

## 7. Optimal Solution

The optimal solution is to apply binary search on the range of answers.

### Algorithm
1. Set `low = max(nums)` and `high = sum(nums)`.
2. Initialize `ans = high`.
3. While `low <= high`:
   - Compute `mid = low + (high - low) / 2`.
   - Run `isValid(nums, mid, k)`.
   - If `isValid` returns `true`:
     - Update `ans = mid` (candidate answer).
     - Try to find a smaller maximum sum: `high = mid - 1`.
   - If `isValid` returns `false`:
     - Limit is too tight. Increase allowed sum: `low = mid + 1`.
4. Return `ans`.

### Helper Function `isValid(nums, limit, k)`
- Track `current_sum = 0` and `subarrays_count = 1`.
- For each `num` in `nums`:
  - If `current_sum + num > limit`:
    - Increment `subarrays_count`.
    - Reset `current_sum = num`.
  - Else:
    - Add `num` to `current_sum`.
- Return `subarrays_count <= k`.

### Full Dry Run
`nums = [7, 2, 5, 10, 8]`, `k = 2`
- `low = 10` (max element), `high = 32` (sum of elements).

#### Iteration 1:
- `low = 10`, `high = 32`, `mid = 21`
- Check `isValid(21)`:
  - Accumulate: `7 + 2 + 5 = 14 <= 21`
  - Next element `10`: `14 + 10 = 24 > 21` $\rightarrow$ Start new subarray. count = 2. current = 10.
  - Next element `8`: `10 + 8 = 18 <= 21`.
  - End of array. Subarrays count = 2 $\le$ 2. Valid!
- Update `ans = 21`, `high = 20`.

#### Iteration 2:
- `low = 10`, `high = 20`, `mid = 15`
- Check `isValid(15)`:
  - Accumulate: `7 + 2 + 5 = 14 <= 15`
  - Next element `10`: `14 + 10 = 24 > 15` $\rightarrow$ Start new subarray. count = 2. current = 10.
  - Next element `8`: `10 + 8 = 18 > 15` $\rightarrow$ Start new subarray. count = 3. current = 8.
  - Subarrays count = 3 > 2. Invalid!
- Update `low = 16`.

#### Iteration 3:
- `low = 16`, `high = 20`, `mid = 18`
- Check `isValid(18)`:
  - Accumulate: `7 + 2 + 5 = 14 <= 18`
  - Next element `10`: `14 + 10 = 24 > 18` $\rightarrow$ Start new subarray. count = 2. current = 10.
  - Next element `8`: `10 + 8 = 18 <= 18`.
  - Subarrays count = 2 $\le$ 2. Valid!
- Update `ans = 18`, `high = 17`.

#### Iteration 4:
- `low = 16`, `high = 17`, `mid = 16`
- Check `isValid(16)`:
  - Accumulate: `7 + 2 + 5 = 14 <= 16`
  - Next element `10`: `14 + 10 = 24 > 16` $\rightarrow$ Start new subarray. count = 2. current = 10.
  - Next element `8`: `10 + 8 = 18 > 16` $\rightarrow$ Start new subarray. count = 3. current = 8.
  - Subarrays count = 3 > 2. Invalid!
- Update `low = 17`.

#### Iteration 5:
- `low = 17`, `high = 17`, `mid = 17`
- Check `isValid(17)`:
  - Accumulate: `7 + 2 + 5 = 14 <= 17`
  - Next element `10`: `14 + 10 = 24 > 17` $\rightarrow$ Start new subarray. count = 2. current = 10.
  - Next element `8`: `10 + 8 = 18 > 17` $\rightarrow$ Start new subarray. count = 3. current = 8.
  - Subarrays count = 3 > 2. Invalid!
- Update `low = 18`.
- Loop terminates since `low (18) > high (17)`.
- Final answer: `18`.

### Why Optimal?
Binary search divides the search space of size $S$ (where $S = \sum(\text{nums}) - \max(\text{nums})$) by half each time. The validation helper runs in $O(n)$ time. The resulting total time complexity is $O(n \log(\sum(\text{nums})))$, which is extremely efficient and easily runs within limits for $N = 10^5$ and sum $\approx 10^{10}$.

### Beautiful C++17 Code

```cpp
#include <vector>
#include <numeric>
#include <algorithm>

class Solution {
private:
    // Greedily checks if it's possible to split nums into <= k subarrays
    // such that the maximum sum of any subarray does not exceed 'limit'.
    bool isValid(const std::vector<int>& nums, long long limit, int k) {
        long long current_sum = 0;
        int subarrays_count = 1;

        for (int num : nums) {
            // If a single element exceeds limit, it's impossible.
            // (Though low is initialized to max_element, so this is safe)
            if (num > limit) {
                return false;
            }

            if (current_sum + num > limit) {
                // Assign current element to a new subarray
                subarrays_count++;
                current_sum = num;

                // If number of subarrays exceeds allowed k, limit is invalid
                if (subarrays_count > k) {
                    return false;
                }
            } else {
                current_sum += num;
            }
        }
        return true;
    }

public:
    int splitArray(const std::vector<int>& nums, int k) {
        if (nums.empty()) return 0;

        // The answer range is between the maximum single element (min possible max sum)
        // and the sum of all elements (max possible max sum).
        long long low = *std::max_element(nums.begin(), nums.end());
        long long high = std::accumulate(nums.begin(), nums.end(), 0LL);
        long long ans = high;

        while (low <= high) {
            long long mid = low + (high - low) / 2;

            if (isValid(nums, mid, k)) {
                ans = mid;         // Record potential valid minimized sum
                high = mid - 1;    // Try to find a smaller maximum sum
            } else {
                low = mid + 1;     // Limit too small, search right side
            }
        }

        return static_cast<int>(ans);
    }
};
```

---

## 8. Code Walkthrough

### `splitArray` function:
- **`low` Initialization:** `*max_element` identifies the largest element. No maximum sum can be smaller than this element.
- **`high` Initialization:** `accumulate(..., 0LL)` computes the total sum using a `long long` literal `0LL` to avoid integer overflow during summation.
- **`while(low <= high)`:** Classic binary search loop.
- **`ans = mid`:** We cache `mid` because we are trying to find the minimum value.

### `isValid` helper:
- **`subarrays_count = 1`:** We start with one subarray.
- **`num > limit` check:** Safely handles cases where an element is larger than the candidate limit (though our search start bound `low` prevents this).
- **`current_sum + num > limit`:** If adding the current number to the active subarray pushes its sum past the limit, we close the active subarray and start a new one with the current number.

---

## 9. Dry Run

Let's trace `nums = [1, 2, 3, 4, 5]`, `k = 3`:
- `low = 5`
- `high = 15`

| Step | `low` | `high` | `mid` | `isValid(mid)` Output | Action | Updated `ans` |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| Init | 5 | 15 | - | - | - | 15 |
| 1 | 5 | 15 | 10 | True (`[1,2,3,4]`, `[5]`) -> 2 subs | `high = mid - 1` | 10 |
| 2 | 5 | 9 | 7 | True (`[1,2,3]`, `[4]`, `[5]`) -> 3 subs | `high = mid - 1` | 7 |
| 3 | 5 | 6 | 5 | False (`[1,2]`, `[3]`, `[4]`, `[5]`) -> 4 subs | `low = mid + 1` | 7 |
| 4 | 6 | 6 | 6 | True (`[1,2,3]`, `[4]`, `[5]`) -> 3 subs | `high = mid - 1` | 6 |
| End | 6 | 5 | - | Pointers crossed (`low > high`) | Terminate | **6** |

---

## 10. Complexity Analysis

- **Time Complexity:**
  - **Initialization:** Finding the maximum element takes $O(n)$ time. Summing elements takes $O(n)$ time.
  - **Binary Search Loop:** The search range is $R = \sum(\text{nums}) - \max(\text{nums})$. The loop runs $\log_2(R)$ times.
  - **Validation Check:** Each call to `isValid` takes $O(n)$ time.
  - **Total Time Complexity:** $O(n \log(\sum(\text{nums})))$. For $N = 10^5$ and sum $= 10^{10}$, $\log_2(10^{10}) \approx 34$ iterations. $34 \times 10^5 \approx 3.4 \times 10^6$ operations, which easily finishes in < 10ms.
- **Space Complexity:**
  - $O(1)$ auxiliary space as we only use a few tracking variables.

---

## 11. Common Mistakes

1. **Incorrect Search Space Boundaries:**
   - Setting `low = 0` or `low = 1`. If `low` is less than `max(nums)`, the greedy checker will fail because a single element cannot be split, and we cannot assign it to any valid subarray.
2. **Signed 32-bit Integer Overflow:**
   - Summing elements using `int` instead of `long long`. If `nums` has $10^5$ elements each equal to $10^5$, the sum is $10^{10}$, which overflows standard 32-bit signed `int` (max $\approx 2 \times 10^9$).
3. **Off-by-One subarray count initiation:**
   - Initializing `subarrays_count = 0` instead of `1`. We must start with at least 1 subarray.

---

## 12. Edge Cases

| Edge Case | Input Example | Expected Output | Rationale |
| :--- | :--- | :--- | :--- |
| **k = 1** | `nums = [1, 2, 3]`, `k = 1` | `6` | Only 1 subarray is allowed. The answer must be the sum of all elements. |
| **k = nums.length** | `nums = [1, 2, 3]`, `k = 3` | `3` | Each element gets its own subarray. The answer is the maximum element. |
| **All elements equal** | `nums = [4, 4, 4, 4]`, `k = 2` | `8` | Best split is `[4, 4]` and `[4, 4]`. |
| **Single element** | `nums = [10]`, `k = 1` | `10` | Single element, single partition. |

---

## 13. Interview Explanation

> "To solve the problem of partitioning an array into $k$ contiguous subarrays while minimizing the maximum subarray sum, we notice that the answer space is monotonic. The minimum possible answer is the value of the largest single element, and the maximum possible answer is the sum of all elements. 
> Since this range is sorted, we can binary search for the correct maximum sum. For each candidate sum `mid`, we write a greedy function that iterates through the array and counts how many subarrays are needed to ensure no subarray exceeds `mid`. If the count is less than or equal to $k$, it means `mid` is a feasible maximum sum. We record `mid` as a candidate answer and check if a smaller sum is possible by searching the left half. If not, we search the right half. This approach runs in $O(n \log(\text{sum}))$ time, which is highly optimal."

---

## 14. Follow-up Questions

1. **How does this problem relate to the Book Allocation / Painter's Partition problem?**
   - *Answer:* They are identical. In Book Allocation, $N$ books with pages `nums[i]` must be allocated to $k$ students. In Painter's Partition, $N$ boards of length `nums[i]` must be painted by $k$ painters. Both minimize the maximum work allocated.
2. **What if the subarrays do not need to be contiguous?**
   - *Answer:* The problem changes completely. It becomes the bin packing / job scheduling problem, which is NP-complete. We would need dynamic programming or backtracking instead of binary search.

---

## 15. Variations

1. **Capacity To Ship Packages Within D Days (LeetCode 1011):** Find the minimum weight capacity of a ship to transport packages within $D$ days. (Identical concept).
2. **Aggressive Cows (Spoj):** Instead of minimizing the maximum distance, we maximize the minimum distance. The structure is the same, but the greedy logic and boundary updates are inverted.

---

## 16. Revision Notes

- **Answer Range:** `[max_element, sum_of_elements]`
- **Greedy Check:** Accumulate elements; if sum exceeds limit, increment count and reset sum.
- **Binary Search Direction:**
  - If valid: `high = mid - 1` (minimize)
  - If invalid: `low = mid + 1` (allow larger sum)
- **Key Mnemonic:** *"Range is Max to Sum. Greedy check determines validity. Binary Search narrows down the sweet spot."*

---

## 17. Memorization Version

```cpp
bool isValid(vector<int>& nums, long long mid, int k) {
    long long sum = 0; int count = 1;
    for (int x : nums) {
        if (sum + x > mid) { count++; sum = x; }
        else sum += x;
    }
    return count <= k;
}
int splitArray(vector<int>& nums, int k) {
    long long low = *max_element(nums.begin(), nums.end());
    long long high = accumulate(nums.begin(), nums.end(), 0LL);
    long long ans = high;
    while (low <= high) {
        long long mid = low + (high - low) / 2;
        if (isValid(nums, mid, k)) { ans = mid; high = mid - 1; }
        else low = mid + 1;
    }
    return ans;
}
```
*Time:* $O(n \log(\text{sum}))$, *Space:* $O(1)$. Bounded search range sorted dynamically.
