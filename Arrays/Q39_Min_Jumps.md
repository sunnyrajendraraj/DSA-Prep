# Minimum Number of Jumps to Reach End

## 1. Problem Statement

Given an array of non-negative integers `arr` of size `n`, where each element represents the maximum number of steps that can be made forward from that element. Write a program to return the minimum number of jumps to reach the last index of the array (starting from the first index). If an element is 0, then you cannot move through that element. If it is impossible to reach the end, return -1.

### Input
- An array of integers `arr` of size `n`.

### Output
- An integer representing the minimum number of jumps to reach index `n - 1`. If the end is unreachable, return `-1`.

### Constraints
- $1 \le n \le 10^6$
- $0 \le arr[i] \le 10^5$

### Observations
1. At any index `i`, we can jump to any index `j` in the range `i + 1` to `i + arr[i]`.
2. This is a optimization problem of finding the shortest path in a directed graph where edges exist from `i` to all indices reachable from `i`.
3. Since we want to find the *minimum* number of jumps, it fits standard shortest-path BFS or greedy structures.

---

## 2. Interview Clarification

Before writing any code, clarify these points with the interviewer:
1. **Can the array contain zeros?**
   - *Yes, `arr[i] = 0` means you cannot jump forward from index `i`.*
2. **What should we return if the destination is unreachable?**
   - *Return `-1`.*
3. **What if the array has only 1 element?**
   - *If `n = 1`, we are already at the end. The answer is `0` jumps.*
4. **Is it guaranteed that we start at index 0?**
   - *Yes, we always start at index 0.*

---

## 3. Thinking Process

Let's think about how to solve this:
- **First Instinct (Brute Force Recursion):**
  From index `0`, we can jump to any index `j` from `1` to `arr[0]`.
  The minimum jumps to reach the end from index `i` is $1 + \min(\text{jumps to reach end from } j)$ for all $j \in [i+1, i+arr[i]]$.
  This recursive tree has a branching factor up to $n$, leading to exponential time complexity: $O(n^n)$ or $O(2^n)$.

- **Improving to Dynamic Programming (Better Solution):**
  We can compute this bottom-up. Create a `dp` array of size `n` where `dp[i]` represents the minimum jumps required to reach index `i` from index `0`.
  - Initialize `dp[0] = 0`, and `dp[i] = INT_MAX` for $i > 0$.
  - For each index `i` from 0 to `n-1`, if `dp[i]` is reachable (i.e. not `INT_MAX`), we can transition to all indices `j` that we can reach from `i`.
  - For $j$ from $i+1$ to $\min(n-1, i + arr[i])$, update `dp[j] = min(dp[j], dp[i] + 1)`.
  This is a bottom-up DP that takes $O(n^2)$ time and $O(n)$ space. While correct, for $n = 10^6$ (as per typical constraints), $O(n^2)$ is too slow.

- **Optimal Insight (Greedy Approach):**
  Instead of evaluating every single jump combination, we can think of this as a Breadth-First Search (BFS) in terms of levels.
  Each "level" represents the range of indices reachable with a certain number of jumps.
  Let's track three variables:
  1. `maxReach`: The farthest index we can reach so far from *any* of the indices visited in the current or previous steps.
  2. `currentEnd`: The boundary of the current jump "level". Once we reach this index, we MUST make another jump to move forward.
  3. `jumps`: The count of jumps taken.

  As we iterate through the array from `i = 0` to `n-2`:
  - Update `maxReach = max(maxReach, i + arr[i])`.
  - If we reach the end of the current jump level (`i == currentEnd`):
    - We must make a jump. Increement `jumps`.
    - Update the next level boundary: `currentEnd = maxReach`.
    - If `currentEnd <= i`, it means we cannot move forward anymore (we hit a 0 or got stuck), so we return `-1`.
  - If at any point `maxReach >= n - 1`, we know we can reach the end on the next jump, but we can also just let the loop run to get the exact jump boundary updates.

  This greedy strategy only scans the array once, giving an $O(n)$ time and $O(1)$ space solution!

---

## 4. Pattern Recognition

This problem fits the **Greedy / BFS Level Partitioning** pattern.
- **Why this topic?** We want to find the shortest path in a 1D array where jumping range is given.
- **Pattern:** 
  - Instead of standard BFS with a queue, we track the boundaries of each level (jumps) using `currentEnd` and `maxReach`.
  - This allows us to simulate BFS in $O(1)$ space.

---

## 5. Brute Force Solution

### Algorithm
1. Define a recursive function `minJumpsRecursion(index, arr)` that returns the min jumps to reach the end from the current `index`.
2. If `index >= n - 1`, return 0 (we are already at or past the end).
3. If `arr[index] == 0`, return `INT_MAX` (we cannot move from here).
4. Loop through all possible jump sizes `j` from 1 to `arr[index]`:
   - Recursively calculate jumps needed from `index + j`.
   - If that path is valid, update the minimum jumps.
5. Return $1 + \min\_jumps\_found$.

### Complexity Analysis
- **Time Complexity:** $O(2^n)$ or $O(n^n)$ in the worst case because we explore every branch.
- **Space Complexity:** $O(n)$ for the recursion stack.

### Pros and Cons
- **Pros:** Simple to think of recursively.
- **Cons:** Triggers TLE even for $n > 20$.

### C++17 Code
```cpp
#include <vector>
#include <algorithm>
#include <climits>

class SolutionBrute {
private:
    int getMinJumps(int index, const std::vector<int>& arr) {
        int n = arr.size();
        if (index >= n - 1) return 0;
        if (arr[index] == 0) return INT_MAX;
        
        int minJumps = INT_MAX;
        for (int j = 1; j <= arr[index]; ++j) {
            int result = getMinJumps(index + j, arr);
            if (result != INT_MAX) {
                minJumps = std::min(minJumps, 1 + result);
            }
        }
        
        return minJumps;
    }

public:
    int minJumps(const std::vector<int>& arr) {
        int result = getMinJumps(0, arr);
        return (result == INT_MAX) ? -1 : result;
    }
};
```

---

## 6. Better Solution

### Algorithm
This is the **Dynamic Programming (Tabulation)** approach.
1. Create a `dp` array of size `n` initialized to `INT_MAX`.
2. Set `dp[0] = 0` (0 jumps to reach index 0).
3. For each index `i` from `0` to `n-2`:
   - If `dp[i] == INT_MAX`, skip (index `i` is unreachable).
   - Otherwise, for each step `j` from `1` to `arr[i]`:
     - If `i + j < n`, update `dp[i + j] = min(dp[i + j], dp[i] + 1)`.
4. Return `dp[n-1]`. If it is still `INT_MAX`, return `-1`.

### Complexity Analysis
- **Time Complexity:** $O(n^2)$ since for each element `i` we could jump up to $n$ elements.
- **Space Complexity:** $O(n)$ for the `dp` array.

### Pros and Cons
- **Pros:** Avoids recursive call stack, correct for $n \le 10^4$.
- **Cons:** Too slow for larger arrays ($n \ge 10^5$).

### C++17 Code
```cpp
#include <vector>
#include <algorithm>
#include <climits>

class SolutionBetter {
public:
    int minJumps(const std::vector<int>& arr) {
        int n = arr.size();
        if (n <= 1) return 0;
        
        std::vector<int> dp(n, INT_MAX);
        dp[0] = 0;
        
        for (int i = 0; i < n; ++i) {
            if (dp[i] == INT_MAX) continue; // Unreachable
            
            int max_jump = arr[i];
            for (int j = 1; j <= max_jump && i + j < n; ++j) {
                dp[i + j] = std::min(dp[i + j], dp[i] + 1);
            }
        }
        
        return (dp[n - 1] == INT_MAX) ? -1 : dp[n - 1];
    }
};
```

---

## 7. Optimal Solution

### Algorithm
We use a **Greedy Single-pass BFS Simulation**.
1. If $n \le 1$, return `0`.
2. If `arr[0] == 0`, return `-1` (we cannot make even the first step).
3. Initialize:
   - `maxReach = arr[0]`: Farthest index we can reach from all positions visited in the current jump level.
   - `currentEnd = arr[0]`: The end of our current jump level.
   - `jumps = 1`: We must make at least 1 jump from the start.
4. Loop through the array from `i = 1` to `n - 2`:
   - Update `maxReach = max(maxReach, i + arr[i])`.
   - If we reach the end of the current jump level (`i == currentEnd`):
     - Increment `jumps`.
     - Update the jump level boundary: `currentEnd = maxReach`.
     - If `currentEnd <= i`, it means we cannot advance further, so return `-1`.
5. After the loop, return `jumps` if `maxReach >= n - 1` else `-1`.

### C++17 Code
```cpp
#include <vector>
#include <algorithm>

class SolutionOptimal {
public:
    int minJumps(const std::vector<int>& arr) {
        int n = arr.size();
        
        // Base case: If array size is 1, we are already at the destination
        if (n <= 1) {
            return 0;
        }
        
        // If the first element is 0, we can never move anywhere
        if (arr[0] == 0) {
            return -1;
        }
        
        // maxReach stores the maximum index we can reach from any index visited so far
        int maxReach = arr[0];
        
        // currentEnd stores the end of the window of the current jump level
        int currentEnd = arr[0];
        
        // jumps stores the number of jumps taken
        int jumps = 1;
        
        // We only loop up to n - 2 because once we reach or pass n - 2,
        // we can always reach the last element on our next step if maxReach >= n - 1.
        for (int i = 1; i < n - 1; ++i) {
            // Update the furthest we can reach
            maxReach = std::max(maxReach, i + arr[i]);
            
            // If we have reached the end of the current jump level
            if (i == currentEnd) {
                jumps++; // Must jump again
                currentEnd = maxReach; // Update the boundary to the new max reach
                
                // If the new boundary cannot advance beyond the current index, we are stuck
                if (currentEnd <= i) {
                    return -1;
                }
            }
        }
        
        // If we can reach the last index, return the jumps count, else -1
        if (maxReach >= n - 1) {
            return jumps;
        }
        return -1;
    }
};
```

---

## 8. Code Walkthrough

Let's break down the optimal solution logic:
1. `if (n <= 1) return 0;`
   - Zero jumps needed to reach the end if we are already there.
2. `if (arr[0] == 0) return -1;`
   - We are stuck at the start.
3. `int maxReach = arr[0]; int currentEnd = arr[0]; int jumps = 1;`
   - We start at index 0. The first jump allows us to reach any index up to `arr[0]`. Thus, our level boundary `currentEnd` is `arr[0]`, the furthest index we can reach is `maxReach = arr[0]`, and we've used 1 jump.
4. `for (int i = 1; i < n - 1; ++i)`
   - We iterate from `1` up to `n - 2`. We don't iterate to `n - 1` because we don't need to jump *from* the last index.
5. `maxReach = std::max(maxReach, i + arr[i]);`
   - As we scan, we find the maximum index we could reach from all visited indices in the current jump window.
6. `if (i == currentEnd)`
   - Once we reach the end of the current window (`currentEnd`), we must commit to another jump. So we increment `jumps`.
   - The new window limit becomes the maximum distance we could have reached (`currentEnd = maxReach`).
   - If `currentEnd <= i`, it means we could not find any index that could jump beyond our current position `i`. We are stuck; return `-1`.
7. `if (maxReach >= n - 1) return jumps;`
   - Finally, check if the furthest we can reach is at least the last index. If so, return `jumps`, otherwise return `-1`.

---

## 9. Dry Run

Let's trace `arr = [1, 3, 5, 8, 9, 2, 6, 7, 6, 8, 9]`.
$n = 11$. Target index = 10.
- **Initial state:** `maxReach = arr[0] = 1`, `currentEnd = arr[0] = 1`, `jumps = 1`.
- Loop runs from `i = 1` to `9`.

| `i` | `arr[i]` | `maxReach` (max(maxReach, i+arr[i])) | `i == currentEnd`? | Action | New `currentEnd` | `jumps` |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Initial** | — | 1 | — | — | 1 | 1 |
| **i = 1** | 3 | $\max(1, 1+3) = 4$ | Yes ($1 == 1$) | Increment jumps, update end | 4 | 2 |
| **i = 2** | 5 | $\max(4, 2+5) = 7$ | No | None | 4 | 2 |
| **i = 3** | 8 | $\max(7, 3+8) = 11$ | No | None | 4 | 2 |
| **i = 4** | 9 | $\max(11, 4+9) = 13$ | Yes ($4 == 4$) | Increment jumps, update end | 13 | 3 |
| **i = 5** | 2 | $\max(13, 5+2) = 13$ | No | None | 13 | 3 |
| **i = 6** | 6 | $\max(13, 6+6) = 12$ | No | None | 13 | 3 |
| **i = 7** | 7 | $\max(13, 7+7) = 14$ | No | None | 13 | 3 |
| **i = 8** | 6 | $\max(14, 8+6) = 14$ | No | None | 13 | 3 |
| **i = 9** | 8 | $\max(14, 9+8) = 17$ | No | None | 13 | 3 |

- **End of Loop check:** `maxReach = 17` which is $\ge n-1$ ($10$).
- **Output:** `3`.

---

## 10. Complexity Analysis

| Approach | Time Complexity | Space Complexity | Notes |
| :--- | :--- | :--- | :--- |
| **Brute Force** | $O(2^n)$ | $O(n)$ | Call stack depth |
| **Better DP** | $O(n^2)$ | $O(n)$ | $N$ states, each checks up to $N$ jumps |
| **Optimal Greedy** | $O(n)$ | $O(1)$ | Single pass scan, constant extra space |

---

## 11. Common Mistakes

1. **Looping up to `n - 1` instead of `n - 2`**:
   - If we loop up to `n - 1`, we might increment the jump count unnecessarily when `i == currentEnd` at the very last index.
2. **Not Handling Stuck Case**:
   - Forgetting to check if `currentEnd <= i`. If `arr = [3, 2, 1, 0, 4]`, at index 3, `currentEnd` becomes 3, and we can never progress. Without checking, the loop will run into an infinite loop or wrong counts.
3. **Wrong Base Case**:
   - Returning `1` instead of `0` when `n = 1`.

---

## 12. Edge Cases

| Edge Case | Input | Expected Output | Why It Matters |
| :--- | :--- | :--- | :--- |
| Single element | `[0]` | `0` | Already at the end, 0 jumps |
| First element is 0 | `[0, 2, 3]` | `-1` | Cannot start moving |
| Unreachable middle 0 | `[2, 0, 0, 1]` | `-1` | Checks if code correctly detects stagnation |
| Reachable in 1 jump | `[5, 1, 1, 1]` | `1` | Checks if we don't count extra jumps |

---

## 13. Interview Explanation

*"To find the minimum number of jumps to reach the end of the array, we can use a greedy BFS-like approach.
Instead of trying every path, which takes exponential time, or using dynamic programming which takes $O(n^2)$ time, we can group indices by the number of jumps it takes to reach them.
We maintain the maximum index we can reach so far (`maxReach`), the boundary of our current jump level (`currentEnd`), and the number of `jumps`.
We iterate through the array. At each element, we update `maxReach`. When our loop index reaches `currentEnd` (meaning we've exhausted all options from our previous jump level), we must make a new jump, so we increment our jump count and update `currentEnd` to `maxReach`. If we ever find that the boundary cannot move past our current position, we return -1. This allows us to find the minimum jumps in $O(n)$ time and $O(1)$ space."*

---

## 14. Follow-up Questions

1. **Can we print the actual path of indices visited to reach the end?**
   - *Yes, but this requires an $O(n)$ space complexity. We can store the parent/previous index of each index that updated its max reach, then reconstruct the path.*
2. **What if the jump lengths can be negative?**
   - *This turns into a general Shortest Path problem on a graph. We would need Dijkstra's algorithm, as simple greedy single-pass only works when jumps are strictly forward (directed acyclic structure).*

---

## 15. Variations

1. **Jump Game I (LeetCode 55)**: Determine if you can reach the last index (returns a boolean).
2. **Jump Game III (LeetCode 1306)**: You can jump either left or right by `arr[i]` steps; find if you can reach any index with value 0 (requires standard BFS/DFS).

---

## 16. Revision Notes

- **Level Boundary Mnemonic**: `currentEnd` represents the end of the current level. Once reached, increment jumps and set `currentEnd = maxReach`.
- **Optimal Complexity**: $O(n)$ time, $O(1)$ space.
- **Loop End**: Terminate loop at `n - 2`.

---

## 17. Memorization Version

```cpp
// Optimal Greedy Min Jumps (Jump Game II)
int minJumps(vector<int>& arr) {
    int n = arr.size();
    if (n <= 1) return 0;
    if (arr[0] == 0) return -1;
    
    int maxReach = arr[0], currentEnd = arr[0], jumps = 1;
    for (int i = 1; i < n - 1; ++i) {
        maxReach = max(maxReach, i + arr[i]);
        if (i == currentEnd) {
            jumps++;
            currentEnd = maxReach;
            if (currentEnd <= i) return -1;
        }
    }
    return (maxReach >= n - 1) ? jumps : -1;
}
```
*Time: $O(n)$, Space: $O(1)$*
