# Maximum Sum of Non-Adjacent Elements (House Robber)

## 1. Problem Statement

Given an array of integers (which can contain positive, negative, or zero elements, though the classic version focuses on non-negative integers), find the maximum sum of elements such that no two elements in the sum are adjacent to each other in the original array.

### Input
- An array of integers `nums` of size `n`.

### Output
- A single integer representing the maximum possible sum of non-adjacent elements.

### Constraints
- $1 \le n \le 10^5$
- $0 \le nums[i] \le 10^4$ (For the classic House Robber problem, values are non-negative. If negatives are allowed, we can choose not to pick any negative numbers if the sum becomes smaller, but usually elements are non-negative).

### Observations
1. If we select an element at index `i`, we cannot select elements at index `i-1` and `i+1`.
2. For each element, we make a binary choice: **Include** it in our sum (and skip the adjacent element) or **Exclude** it (and consider the adjacent element).
3. This is a optimization problem with overlapping subproblems, making it a classic candidate for Dynamic Programming.

---

## 2. Interview Clarification

Before writing any code, a candidate should clarify the following points with the interviewer:
1. **Can the array contain negative numbers?** 
   - *Typically no, but if it does, can the maximum sum be 0 (by choosing nothing)? Yes, usually if all are negative, the answer is 0 (or the maximum single negative number if we must choose at least one). Clarify if we can choose an empty set of elements.*
2. **Can the array be empty?**
   - *Clarify constraints: if $n = 0$, we return 0.*
3. **What is the maximum size of the array?**
   - *This determines if an $O(n^2)$ solution will pass ($n \le 1000$) or if we need $O(n)$ ($n \ge 10^5$).*
4. **Is the array circular? (i.e., is the first element adjacent to the last element?)**
   - *This is the House Robber II variation. For this standard problem, we assume the array is linear.*

---

## 3. Thinking Process

Let's build the intuition from scratch:
- **First Instinct (Brute Force):**
  For each element, we have two choices:
  - Take it: If we take `nums[i]`, we must jump to `nums[i-2]`.
  - Skip it: If we skip `nums[i]`, we can move to `nums[i-1]`.
  This binary choice at every step leads to a decision tree. Let's write a recursive function `solve(i)` which returns the max sum from index `0` to `i`.
  - Formula: `solve(i) = max(nums[i] + solve(i-2), solve(i-1))`
  - Base cases: 
    - `i < 0`: return 0
    - `i == 0`: return `nums[0]`

- **Identifying the Bottleneck:**
  The recursive approach recalculates the same subproblems multiple times. For example, to solve `solve(5)`, we need `solve(4)` and `solve(3)`. To solve `solve(4)`, we need `solve(3)` and `solve(2)`. Here, `solve(3)` is computed twice. This redundancy leads to $O(2^n)$ time complexity.

- **Improving with Memoization (Top-down DP):**
  We can store the results of each state `solve(i)` in a DP array of size `n` initialized to `-1`. If `dp[i] != -1`, return the stored value. This reduces the time complexity to $O(n)$ but requires $O(n)$ space for the recursion stack and the DP array.

- **Eliminating Recursion (Bottom-up DP):**
  We can fill the DP table iteratively:
  - `dp[0] = nums[0]`
  - `dp[1] = max(nums[0], nums[1])`
  - For $i \ge 2$, `dp[i] = max(dp[i-1], nums[i] + dp[i-2])`
  This eliminates the recursion overhead, keeping $O(n)$ time and $O(n)$ space.

- **Optimal Insight (Space Optimization):**
  Look closely at the transition: `dp[i] = max(dp[i-1], nums[i] + dp[i-2])`.
  To compute `dp[i]`, we only need the values of the two preceding states: `dp[i-1]` and `dp[i-2]`.
  Instead of maintaining an entire array, we can use two variables:
  - `prev2` representing `dp[i-2]`
  - `prev1` representing `dp[i-1]`
  At each step:
  - `current = max(prev1, nums[i] + prev2)`
  - Update for the next iteration: `prev2 = prev1`, `prev1 = current`.
  This achieves $O(1)$ auxiliary space while keeping $O(n)$ time!

---

## 4. Pattern Recognition

This problem fits the **Dynamic Programming: Deciding/State Transition** pattern.
- **Why this topic?** It requires choosing a subset of elements to maximize a value under adjacency constraints.
- **Pattern:** 
  - Choice-based state transition: at each step, you either *include* or *exclude* the current element.
  - Overlapping subproblems and optimal substructure make DP the optimal choice.

---

## 5. Brute Force Solution

### Algorithm
1. Define a recursive helper function `solve(index, nums)` that returns the maximum sum possible from index `0` up to `index`.
2. If `index < 0`, return 0.
3. If `index == 0`, return `nums[0]`.
4. Calculate two possibilities:
   - `pick = nums[index] + solve(index - 2, nums)`
   - `notPick = 0 + solve(index - 1, nums)`
5. Return the maximum of `pick` and `notPick`.

### Dry Run
Let `nums = [2, 1, 4, 9]`.
- Call `solve(3)`:
  - `pick = 9 + solve(1)`
    - `solve(1) = max(nums[1] + solve(-1), solve(0)) = max(1 + 0, 2) = 2`
    - So `pick = 9 + 2 = 11`
  - `notPick = solve(2)`
    - `solve(2) = max(nums[2] + solve(0), solve(1))`
      - `solve(0) = 2`
      - `solve(1) = 2` (computed above)
      - `solve(2) = max(4 + 2, 2) = 6`
    - So `notPick = 6`
  - `solve(3) = max(11, 6) = 11`

### Complexity Analysis
- **Time Complexity:** $O(2^n)$ because each recursive step branches into two subproblems.
- **Space Complexity:** $O(n)$ due to the recursion stack depth.

### Pros and Cons
- **Pros:** Extremely simple to understand and implement.
- **Cons:** Extremely slow; triggers Time Limit Exceeded (TLE) for $n > 30$.

### C++17 Code
```cpp
#include <vector>
#include <algorithm>
#include <iostream>

class SolutionBrute {
private:
    int solveRecursively(int index, const std::vector<int>& nums) {
        // Base cases
        if (index < 0) {
            return 0;
        }
        if (index == 0) {
            return nums[0];
        }
        
        // Pick the current element, move to index - 2
        int pick = nums[index] + solveRecursively(index - 2, nums);
        
        // Skip the current element, move to index - 1
        int notPick = solveRecursively(index - 1, nums);
        
        return std::max(pick, notPick);
    }

public:
    int getMaxSum(const std::vector<int>& nums) {
        if (nums.empty()) return 0;
        return solveRecursively(nums.size() - 1, nums);
    }
};
```

---

## 6. Better Solution

### Algorithm
This is the **Dynamic Programming with Memoization (Top-down)** or **Tabulation (Bottom-up)** approach.
We will show the bottom-up Tabulation method as it avoids recursion stack overhead.
1. Create a `dp` array of size `n`.
2. Initialize `dp[0] = nums[0]`.
3. If $n > 1$, initialize `dp[1] = max(nums[0], nums[1])`.
4. Loop from $i = 2$ to $n-1$:
   - `dp[i] = max(dp[i-1], nums[i] + dp[i-2])`
5. Return `dp[n-1]`.

### Dry Run
Let `nums = [2, 1, 4, 9]`. `dp` array size 4.
- `dp[0] = nums[0] = 2`
- `dp[1] = max(nums[0], nums[1]) = max(2, 1) = 2`
- For `i = 2`:
  - `pick = nums[2] + dp[0] = 4 + 2 = 6`
  - `notPick = dp[1] = 2`
  - `dp[2] = max(6, 2) = 6`
- For `i = 3`:
  - `pick = nums[3] + dp[1] = 9 + 2 = 11`
  - `notPick = dp[2] = 6`
  - `dp[3] = max(11, 6) = 11`
- Result is `dp[3] = 11`.

### Complexity Analysis
- **Time Complexity:** $O(n)$ as we loop through the array exactly once.
- **Space Complexity:** $O(n)$ to store the DP table.

### Pros and Cons
- **Pros:** Linear time complexity, completely eliminates redundant calculations.
- **Cons:** Still uses $O(n)$ auxiliary space which can be optimized further.

### C++17 Code
```cpp
#include <vector>
#include <algorithm>

class SolutionBetter {
public:
    int getMaxSum(const std::vector<int>& nums) {
        int n = nums.size();
        if (n == 0) return 0;
        if (n == 1) return nums[0];
        
        std::vector<int> dp(n, 0);
        dp[0] = nums[0];
        dp[1] = std::max(nums[0], nums[1]);
        
        for (int i = 2; i < n; ++i) {
            int pick = nums[i] + dp[i - 2];
            int notPick = dp[i - 1];
            dp[i] = std::max(pick, notPick);
        }
        
        return dp[n - 1];
    }
};
```

---

## 7. Optimal Solution

### Algorithm
We optimize the Space Complexity of the Tabulation DP to $O(1)$ by noting that we only ever reference the previous two states: `dp[i-1]` and `dp[i-2]`.
1. Let `prev2` represent the max sum up to `i-2` (initially 0).
2. Let `prev1` represent the max sum up to `i-1` (initially `nums[0]`).
3. Iterate from `i = 1` to `n-1`:
   - `take = nums[i] + (i > 1 ? prev2 : 0)` (if $i=1$, we cannot add `prev2` as there is no element before `nums[0]`).
   - `skip = prev1`
   - `current_max = max(take, skip)`
   - Update state: `prev2 = prev1`, `prev1 = current_max`.
4. Return `prev1`.

### C++17 Code
```cpp
#include <vector>
#include <algorithm>

class SolutionOptimal {
public:
    int getMaxSum(const std::vector<int>& nums) {
        int n = nums.size();
        if (n == 0) return 0;
        
        // prev1 represents dp[i-1], initialized to the first element
        int prev1 = nums[0];
        // prev2 represents dp[i-2], initialized to 0
        int prev2 = 0;
        
        for (int i = 1; i < n; ++i) {
            // Decide whether to pick the current element or skip it
            int pick = nums[i];
            if (i > 1) {
                pick += prev2; // Add max sum from 2 steps back
            }
            int notPick = prev1; // Carry forward max sum from 1 step back
            
            int current = std::max(pick, notPick);
            
            // Move our pointers forward
            prev2 = prev1;
            prev1 = current;
        }
        
        return prev1;
    }
};
```

---

## 8. Code Walkthrough

Let's break down the optimal solution block-by-block:
1. `if (n == 0) return 0;`
   - Checks if the input array is empty. If it is, the max sum is obviously 0.
2. `int prev1 = nums[0];`
   - Represents the maximum sum we can get considering only the first element.
3. `int prev2 = 0;`
   - Represents the maximum sum we can get for a sequence of length 0 (elements before index 0).
4. `for (int i = 1; i < n; ++i)`
   - Loops from index 1 to `n-1`.
5. `int pick = nums[i]; if (i > 1) pick += prev2;`
   - If we pick `nums[i]`, we can also add the best sum from two steps back (`prev2`). This guard `i > 1` is necessary because at `i = 1`, `prev2` corresponds to a non-existent index `-1`.
6. `int notPick = prev1;`
   - If we don't pick `nums[i]`, the max sum remains the same as the step before (`prev1`).
7. `int current = std::max(pick, notPick);`
   - We choose the best of both decisions.
8. `prev2 = prev1; prev1 = current;`
   - Shift our states forward for the next iteration. `prev2` becomes the old `prev1`, and `prev1` becomes the newly computed `current`.

---

## 9. Dry Run

Let's trace `nums = [2, 1, 4, 9]` with the optimal solution.

| Iteration `i` | `nums[i]` | `prev2` | `prev1` | `pick` (nums[i] + prev2) | `notPick` (prev1) | `current` (max) | New `prev2` | New `prev1` |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Initial** | — | — | 2 | — | — | — | 0 | 2 |
| **i = 1** | 1 | 0 | 2 | $1 + 0 = 1$ | 2 | 2 | 2 | 2 |
| **i = 2** | 4 | 2 | 2 | $4 + 2 = 6$ | 2 | 6 | 2 | 6 |
| **i = 3** | 9 | 2 | 6 | $9 + 2 = 11$ | 6 | 11 | 6 | 11 |

**Final Output:** `prev1 = 11`.

---

## 10. Complexity Analysis

| Approach | Time Complexity | Space Complexity | Notes |
| :--- | :--- | :--- | :--- |
| **Brute Force** | $O(2^n)$ | $O(n)$ | Call stack depth is $O(n)$ |
| **Better (Tabulation DP)** | $O(n)$ | $O(n)$ | Linear scan, uses $O(n)$ array |
| **Optimal (Variables DP)** | $O(n)$ | $O(1)$ | No extra array or recursion |

---

## 11. Common Mistakes

1. **Incorrect Initialization of `prev2`**: 
   - Initializing `prev2` to `nums[0]` is a common error. It must be initialized to `0` because it represents the state at `i < 0`.
2. **Accessing Index `-1`**:
   - In recursive solutions, forgetting to handle `index < 0` leading to out-of-bound errors.
3. **Integer Overflow**:
   - If the elements are very large ($10^9$) and the array is large, the maximum sum can exceed standard integer limits. Although typical constraints fits in a standard 32-bit signed int, always check if `long long` is needed.

---

## 12. Edge Cases

| Edge Case | Input | Expected Output | Why It Matters |
| :--- | :--- | :--- | :--- |
| Single element | `[5]` | `5` | Loop does not run, returns `nums[0]` |
| Two elements | `[3, 10]` | `10` | Loop runs once, chooses the maximum of the two |
| Decreasing array | `[10, 8, 5, 2]` | `15` | Tests if it correctly skips adjacent elements (`10 + 5 = 15`) |
| All zeros | `[0, 0, 0]` | `0` | Sum should be 0 |

---

## 13. Interview Explanation

*"To solve the 'Maximum Sum of Non-Adjacent Elements' problem, my initial approach is to use recursion by exploring all possibilities: at each element, we either pick it (and jump two steps back) or skip it (and move one step back). However, this recursive brute force takes exponential $O(2^n)$ time because it computes the same subproblems repeatedly.
To optimize, we observe that the decision at index `i` depends only on the results of the subproblems at `i-1` and `i-2`. Thus, we can build a Dynamic Programming solution. By keeping track of only two variables—`prev1` for the max sum up to the previous element, and `prev2` for the max sum up to two elements back— we can iteratively calculate the maximum sum for each position in $O(n)$ time. This requires only $O(1)$ extra space, which is optimal."*

---

## 14. Follow-up Questions

1. **What if the houses are arranged in a circle? (House Robber II)**
   - *In a circular array, the first and last elements are adjacent. To solve this, we run our optimal linear DP twice: once on `nums[0 ... n-2]` and once on `nums[1 ... n-1]`. The answer is the maximum of these two runs.*
2. **What if we want to print the actual elements that yield the maximum sum?**
   - *We would need to maintain the choice array/history (e.g. tracking whether we picked or skipped at each step) using a back-pointer array, which requires $O(n)$ space.*

---

## 15. Variations

1. **House Robber II (LeetCode 213)**: The array is circular.
2. **House Robber III (LeetCode 337)**: The houses are arranged in a binary tree. You cannot rob two directly linked parent-child houses.
3. **Delete and Earn (LeetCode 740)**: Can be transformed into the House Robber problem by grouping values and mapping their sums to adjacent index slots.

---

## 16. Revision Notes

- **Core Formula**: `current = max(nums[i] + prev2, prev1)`
- **Pointers Update**: `prev2 = prev1; prev1 = current;`
- **Mnemonic**: "Pick current + two steps back vs. Skip current (take one step back)"
- **Complexity**: $Time = O(n)$, $Space = O(1)$

---

## 17. Memorization Version

```cpp
// Optimal Space-Optimized DP (House Robber)
int rob(vector<int>& nums) {
    int prev1 = nums[0], prev2 = 0;
    for (size_t i = 1; i < nums.size(); ++i) {
        int pick = nums[i] + (i > 1 ? prev2 : 0);
        int notPick = prev1;
        int current = max(pick, notPick);
        prev2 = prev1;
        prev1 = current;
    }
    return prev1;
}
```
*Time: $O(n)$, Space: $O(1)$*
