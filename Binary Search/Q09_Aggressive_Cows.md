# Aggressive Cows (Maximum Minimum Distance)

## 1. Problem Statement
Given an array `stalls` of $N$ integers representing the coordinates of stalls on a straight line. There are $C$ cows which need to be placed in these stalls such that the **minimum distance between any two of them is as large as possible**.
Return this maximum minimum distance.

### Input
- `stalls`: An array of $N$ integers representing stall positions.
- `C`: Number of cows.

### Output
- Return an integer representing the maximum possible value of the minimum distance between any two cows.

### Constraints
- $2 \le N \le 10^5$
- $0 \le \text{stalls}[i] \le 10^9$
- $2 \le C \le N$

### Observations
1. If we sort the stall positions, we can easily calculate the distance between any two stalls.
2. The minimum possible distance between any two cows is `1` (if two stalls are adjacent).
3. The maximum possible distance between any two cows cannot exceed the distance between the first and last stall: `stalls[N-1] - stalls[0]` (after sorting).
4. As the target minimum distance $D$ increases, it becomes harder (or impossible) to place all $C$ cows in the stalls. If a distance $D$ is possible, then any distance $< D$ is also possible. This represents a monotonic search space: `[Possible, Possible, ..., Possible, Impossible, Impossible]`.

---

## 2. Interview Clarification
Before coding, a candidate should clarify:
1. **Q:** Are the stall positions sorted in the input?
   - **A:** No, stall positions can be in any random order. Sorting them first is a necessary preprocessing step.
2. **Q:** Can multiple cows be placed in the same stall?
   - **A:** No, at most one cow can be placed in any stall.
3. **Q:** What if $C > N$?
   - **A:** It is impossible to place the cows since we can only place one cow per stall. However, the constraints state $C \le N$.
4. **Q:** Can stall coordinates be negative?
   - **A:** Stall positions are non-negative integers. If they were negative, sorting still handles it correctly.

---

## 3. Thinking Process
### First Instinct
To find the configuration that maximizes the minimum distance, we could try to generate all combinations of choosing $C$ stalls out of $N$. For each combination, we compute the minimum distance between adjacent chosen stalls and keep track of the maximum.
- Time complexity would be $O(\binom{N}{C})$, which is extremely slow and will fail for $N = 10^5$.

### Can we do better?
Let's think about the answer directly. The answer must lie in the range $[1, \text{stalls}[N-1] - \text{stalls}[0]]$ (assuming sorted).
Let's call a distance limit $D$. We want to check if it's possible to place $C$ cows such that every pair of cows is separated by a distance of at least $D$:
- We place the first cow in the first stall (`stalls[0]`) to maximize the remaining space.
- Then, we iterate through the subsequent stalls and place the next cow at the first stall whose distance from the last placed cow is $\ge D$.
- If we can place all $C$ cows this way, then the distance $D$ is **feasible**.
- If we run out of stalls before placing all $C$ cows, then $D$ is **infeasible**.

Since the feasibility function `canPlace(D)` is monotonic (i.e. if $D$ is feasible, $D-1$ is also feasible; if $D$ is infeasible, $D+1$ is also infeasible), we can use **Binary Search on the Answer**.

---

## 4. Pattern Recognition
This problem belongs to the **Binary Search on Answer (Maximize the Minimum)** pattern.
- Why? We are asked to "maximize the minimum distance". 
- The search space of possible distances is contiguous: $[1, \text{max\_distance}]$.
- The feasibility function `canPlace(D)` divides the search space into two contiguous halves: `[True, True, ..., True, False, False]`. We want to find the largest value for which `canPlace(D)` is `True`.

---

## 5. Brute Force Solution
### Algorithm
1. Sort the `stalls` array.
2. The search range of distance starts at $1$ and goes up to $high = \text{stalls}[N-1] - \text{stalls}[0]$.
3. Iterate $D$ from $1$ to $high$:
   - Check if we can place $C$ cows with a minimum distance of $D$ using a greedy placement function.
   - If we can, update the answer to $D$.
   - If we cannot, return the last successful $D$ (which is $D-1$).

### Complexity Analysis
- **Time Complexity:** 
  - Sorting: $O(N \log N)$
  - Search loop: $O(N \times \text{stalls}[N-1])$ in the worst case, which can be up to $10^{14}$ operations and will TLE.
- **Space Complexity:** $O(1)$ auxiliary space.

### Pros and Cons
- **Pros:** Simple, sequential search.
- **Cons:** Very slow for large coordinates (e.g. coordinates up to $10^9$).

### C++17 Code
```cpp
#include <vector>
#include <algorithm>

class Solution {
private:
    bool canPlaceCows(const std::vector<int>& stalls, int n, int cows, int minDist) {
        int count = 1; // Place the first cow at stalls[0]
        int lastPosition = stalls[0];

        for (int i = 1; i < n; ++i) {
            if (stalls[i] - lastPosition >= minDist) {
                count++;
                lastPosition = stalls[i];
                if (count == cows) return true;
            }
        }
        return false;
    }

public:
    int aggressiveCowsBruteForce(std::vector<int>& stalls, int cows) {
        int n = stalls.size();
        std::sort(stalls.begin(), stalls.end());

        int maxDist = stalls[n - 1] - stalls[0];
        int ans = 1;

        for (int d = 1; d <= maxDist; ++d) {
            if (canPlaceCows(stalls, n, cows, d)) {
                ans = d;
            } else {
                break;
            }
        }
        return ans;
    }
};
```

---

## 6. Better Solution
*Note: For this specific problem, there is no intermediate "Better" solution that sits between the brute force (linear search on answer) and the optimal Binary Search on the answer. We jump directly from Brute Force to the Optimal Binary Search.*

---

## 7. Optimal Solution
### Algorithm
1. Sort the `stalls` array in non-decreasing order.
2. Initialize `low = 1` (the minimum possible distance) and `high = stalls[N-1] - stalls[0]` (the maximum possible distance).
3. Initialize `ans = 1`.
4. Perform Binary Search on the range `[low, high]`:
   - Calculate `mid = low + (high - low) / 2`.
   - Check if we can place $C$ cows with a minimum separation of `mid` using `canPlaceCows(stalls, N, C, mid)`.
   - If `canPlaceCows` is true:
     - This means `mid` is a valid minimum distance. We record `ans = mid`.
     - Since we want to maximize the minimum distance, we try to find a larger valid distance by searching the right half: `low = mid + 1`.
   - If `canPlaceCows` is false:
     - This means `mid` is too large to place $C$ cows. We must search the left half: `high = mid - 1`.
5. Return `ans`.

### Dry Run
`stalls` = `[1, 2, 8, 4, 9]`, $C = 3$.
1. Sort `stalls` $\rightarrow$ `[1, 2, 4, 8, 9]`.
2. $N = 5$.
3. `low = 1`, `high = 9 - 1 = 8`.
4. Let `ans = 1`.

**Iteration 1:**
- `mid = 1 + (8 - 1) / 2 = 4`
- Check `canPlaceCows(stalls, 5, 3, 4)`:
  - Cow 1: placed at `1`.
  - Next stall $\ge 1 + 4 = 5$. First stall matching is `8`. Cow 2 placed at `8`.
  - Next stall $\ge 8 + 4 = 12$. No stalls matching.
  - Cows placed = 2. We need 3. Return **false**.
- Update: `high = mid - 1 = 3`.

**Iteration 2:**
- `low = 1`, `high = 3`.
- `mid = 1 + (3 - 1) / 2 = 2`
- Check `canPlaceCows(stalls, 5, 3, 2)`:
  - Cow 1: placed at `1`.
  - Next stall $\ge 1 + 2 = 3$. First stall matching is `4`. Cow 2 placed at `4`.
  - Next stall $\ge 4 + 2 = 6$. First stall matching is `8`. Cow 3 placed at `8`.
  - Cows placed = 3. Return **true**.
- Update: `ans = 2`, `low = mid + 1 = 3`.

**Iteration 3:**
- `low = 3`, `high = 3`.
- `mid = 3 + (3 - 3) / 2 = 3`
- Check `canPlaceCows(stalls, 5, 3, 3)`:
  - Cow 1: placed at `1`.
  - Next stall $\ge 1 + 3 = 4$. First stall matching is `4`. Cow 2 placed at `4`.
  - Next stall $\ge 4 + 3 = 7$. First stall matching is `8`. Cow 3 placed at `8`.
  - Cows placed = 3. Return **true**.
- Update: `ans = 3`, `low = mid + 1 = 4`.

**Loop Ends:**
- `low` (4) > `high` (3). Loop terminates.
- Return `ans = 3`.

### C++17 Code
```cpp
#include <vector>
#include <algorithm>

class Solution {
private:
    // Helper function to check if we can place 'cows' cows in 'stalls'
    // such that the minimum distance between any two cows is at least 'minDist'
    bool canPlaceCows(const std::vector<int>& stalls, int n, int cows, int minDist) {
        int count = 1; // Place the first cow in the first stall
        int lastPosition = stalls[0];

        for (int i = 1; i < n; ++i) {
            if (stalls[i] - lastPosition >= minDist) {
                count++;
                lastPosition = stalls[i]; // Update the position of the last placed cow
                
                if (count == cows) {
                    return true;
                }
            }
        }
        return false;
    }

public:
    int getMaximumMinDistance(std::vector<int>& stalls, int cows) {
        int n = stalls.size();
        
        // Step 1: Sort the stall locations
        std::sort(stalls.begin(), stalls.end());

        // Step 2: Define binary search boundaries
        int low = 1;                                  // Minimum possible distance
        int high = stalls[n - 1] - stalls[0];         // Maximum possible distance
        int ans = 1;

        // Step 3: Perform Binary Search on the distance range
        while (low <= high) {
            int mid = low + (high - low) / 2;

            if (canPlaceCows(stalls, n, cows, mid)) {
                ans = mid;          // Record the candidate distance
                low = mid + 1;      // Try to find a larger minimum distance
            } else {
                high = mid - 1;     // Reduce the distance limit
            }
        }

        return ans;
    }
};
```

---

## 8. Code Walkthrough
- `std::sort(stalls.begin(), stalls.end());`: Sorts the stall coordinate positions. This allows a greedy placement in a single linear pass.
- `int low = 1;`: Sets the minimum possible distance to 1.
- `int high = stalls[n - 1] - stalls[0];`: Sets the maximum possible distance to the span of the furthest stalls.
- `while (low <= high)`: Runs binary search.
- `canPlaceCows(stalls, n, cows, mid)`: Evaluates if we can place all cows such that no two cows are closer than `mid`.
  - We place the first cow at `stalls[0]`.
  - For each subsequent stall, we check if `stalls[i] - lastPosition >= minDist`. If it is, we place a cow there, update `lastPosition = stalls[i]`, and increment the cow count.
  - If we place all `cows`, return `true`. If we reach the end of the stalls without placing all cows, return `false`.
- `ans = mid; low = mid + 1;`: If `mid` distance is possible, record it and try to find a larger distance by moving the lower bound up.
- `high = mid - 1`: If `mid` is not possible, search in the smaller distance range.

---

## 9. Dry Run
Using `stalls` = `[1, 2, 4, 8, 9]`, $C = 3$.

| Step | low | high | mid | Cows Placed (Positions) | Valid? | Action | ans |
|---|---|---|---|---|---|---|---|
| 1 | 1 | 8 | 4 | 2 (1, 8) | No | `high = 3` | 1 |
| 2 | 1 | 3 | 2 | 3 (1, 4, 8) | Yes | `low = 3` | 2 |
| 3 | 3 | 3 | 3 | 3 (1, 4, 8) | Yes | `low = 4` | 3 |

Loop terminates. Return `3`.

---

## 10. Complexity Analysis
- **Time Complexity:**
  - Sorting the array: $O(N \log N)$.
  - The binary search takes $\log(\text{max\_dist})$ iterations where $\text{max\_dist} = \text{stalls}[N-1] - \text{stalls}[0] \le 10^9$. $\log(10^9) \approx 30$.
  - Inside each step of the binary search, we iterate through the stalls array of size $N$, taking $O(N)$ time.
  - **Total Time Complexity:** $O(N \log N + N \log(\text{max\_dist}))$. For $N = 10^5$ and coordinates up to $10^9$, this executes around $3 \times 10^6$ operations, which is extremely fast.
- **Space Complexity:**
  - $O(1)$ auxiliary space if sorting is done in-place.

---

## 11. Common Mistakes
1. **Forgetting to Sort:** The greedy checker `canPlaceCows` works only if the stall locations are in sorted order. Forgetting to sort results in incorrect greedy selections.
2. **Incorrect search space bounds:** Setting `low = stalls[0]` and `high = stalls[N-1]` is a common mistake. The range is for *distance*, not coordinate values. The minimum distance is $1$, and the maximum distance is $\text{stalls}[N-1] - \text{stalls}[0]$.
3. **Off-by-one during greedy placement:** Starting the cow count at 0 instead of 1. Since we always place the first cow at `stalls[0]`, `count` must be initialized to 1.

---

## 12. Edge Cases
| Edge Case | Example | Why it matters | Solution |
|---|---|---|---|
| 2 Cows | `stalls=[1, 9]`, $C=2$ | Only two cows can be placed. | Handled correctly. Returns the distance between the two stalls (`8`). |
| Consecutive Stalls | `stalls=[1, 2, 3]`, $C=3$ | Min distance is 1. | Handled correctly, returns 1. |
| Large Stall Coordinates | Stalls up to $10^9$ | Risk of integer overflow during difference calculations. | Storing coordinates in standard `int` is fine since $10^9 < 2 \times 10^9$, but check ranges. |
| $C = N$ | `stalls=[1, 5, 8]`, $C=3$ | Must place a cow in every stall. | Returns the minimum difference between adjacent stalls. |

---

## 13. Interview Explanation
"The problem asks us to place $C$ cows in $N$ stalls such that the minimum distance between any two cows is maximized. This is a classic 'Maximize the Minimum' problem. 
If we sort the stall positions, the range of possible answers for the minimum distance must lie between $1$ and the total span of the stalls, which is `stalls[N-1] - stalls[0]`.
Since the viability of a minimum distance $D$ is monotonic (if a distance of $4$ is possible, then $3$ and $2$ are also possible), we can perform binary search on this distance range. 
For each candidate distance `mid`, we write a greedy helper function `canPlaceCows` which places the first cow at the first stall, and then places each subsequent cow in the next available stall that is at least `mid` distance away. If we can place all $C$ cows, then `mid` is possible, so we record it and try for a larger distance by searching the right half. Otherwise, we search the left half.
The time complexity is dominated by sorting, $O(N \log N)$, and the binary search, $O(N \log(\text{max\_dist}))$. The space complexity is $O(1)$."

---

## 14. Follow-up Questions
1. **Q: What if the stalls are arranged in a circular track instead of a linear line?**
   - **A:** If it is a circle, we must consider the wrap-around distance. We would need to select a starting stall, and the distance between the last cow and the first cow must also be $\ge D$. We can try placing the first cow in different stalls or optimize using search logic.
2. **Q: Why does placing the first cow at `stalls[0]` always yield the optimal configuration?**
   - **A:** Placing the first cow as far left as possible (i.e. at `stalls[0]`) maximizes the remaining space available for the rest of the cows, which never hurts our chances of placing them.

---

## 15. Variations
1. **Magnetic Force Between Two Balls (LeetCode 1552):**
   - Identical problem with a different description (balls and buckets instead of cows and stalls).
2. **Reduce Array Size to The Half (Conceptually similar greedy constraint checks):**
   - General binary search on answer where validity is checked via greedy.

---

## 16. Revision Notes
- **Problem Pattern:** Maximize the minimum value.
- **Search Space:** `low = 1`, `high = stalls[N-1] - stalls[0]`.
- **Greedy Check:** Place cow 1 at `stalls[0]`. Next cow placed at stall where `stalls[i] - last >= mid`.
- **Monotonicity:** `[True, True, ..., True, False, False]`. Find last `True`.

---

## 17. Memorization Version
- Sort `stalls`.
- Range: `low = 1`, `high = stalls[N-1] - stalls[0]`.
- Loop: `mid = low + (high - low) / 2`.
- Checker: Greedily place cows. First cow at `stalls[0]`. If `stalls[i] - last >= mid`, place next cow. If total cows placed $\ge C$, return true.
- If true: `ans = mid`, `low = mid + 1`. Else: `high = mid - 1`.
- Return `ans`.
