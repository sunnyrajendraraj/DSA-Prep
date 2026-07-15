# Trapping Rain Water

## 1. Problem Statement

Given `N` non-negative integers representing an elevation map where the width of each bar is `1`, compute how much water it can trap after raining.

### Input
- An integer array `height` of size `N`.

### Output
- An integer representing the total volume of trapped rainwater.

### Constraints
- $1 \le N \le 10^5$
- $0 \le height[i] \le 10^4$

### Observations
1. Water cannot be trapped at the first bar (index `0`) and the last bar (index `N-1`) because there is no boundary on one side to contain the water.
2. The water trapped on top of any bar `i` depends on the height of the tallest bar to its left and the height of the tallest bar to its right.
3. Specifically, for any bar `i`:
   - Let `L_max` be the maximum height in `height[0...i]`.
   - Let `R_max` be the maximum height in `height[i...N-1]`.
   - The water level above bar `i` is determined by `min(L_max, R_max)`.
   - The water trapped at bar `i` is `min(L_max, R_max) - height[i]`.

```
       Water Level = min(L_max, R_max)
       ~~~~~~~|~~~~~~~
       |      |      |
       |L_max |      | R_max
       |      |bar i |
_______|______|_height|_______
```

---

## 2. Interview Clarification

- **Q:** What should we return if the array size is less than 3?
  - **A:** Return `0` since we need at least 3 bars (left wall, middle basin, right wall) to trap any water.
- **Q:** Can the height of the bars be negative?
  - **A:** No, heights are always non-negative.
- **Q:** Can we modify the input array?
  - **A:** Yes, but we should aim for an optimal solution that uses $O(1)$ extra space without modifying the input array.

---

## 3. Thinking Process

### First Instinct (Brute Force)
For each bar in the array, we look left to find the tallest bar, and look right to find the tallest bar.
- Iterate `i` from `0` to `N - 1`.
- For each `i`, run a loop to the left to find `left_max = max(height[0...i])`.
- Run another loop to the right to find `right_max = max(height[i...N-1])`.
- Add `min(left_max, right_max) - height[i]` to our total water.
- Since we do this for every element, the time complexity is $O(N^2)$.

### Precalculating Boundaries (Better Solution)
In the brute force approach, we recalculate the left and right maximums repeatedly. We can optimize this by precomputing them:
- Create a `left_max` array where `left_max[i]` stores the max height from index `0` to `i`. We can populate this in one left-to-right pass: `left_max[i] = max(left_max[i-1], height[i])`.
- Create a `right_max` array where `right_max[i]` stores the max height from index `i` to `N-1`. We can populate this in one right-to-left pass: `right_max[i] = max(right_max[i+1], height[i])`.
- In a third pass, compute the water trapped at each index: `water += min(left_max[i], right_max[i]) - height[i]`.
- This reduces the time complexity to $O(N)$, but requires $O(N)$ auxiliary space.

### Eliminating Extra Space (Optimal Insight)
Can we do this in $O(1)$ space?
Instead of precomputing the entire prefix and suffix max arrays, we can maintain the boundaries on the fly using two pointers, `left` at `0` and `right` at `N - 1`.
- We maintain `left_max` (the maximum height seen so far from the left) and `right_max` (the maximum height seen so far from the right).
- Since water level is capped by the **minimum** of the two walls (`min(left_max, right_max)`), we only need to know the smaller side at any point.
- If `height[left] <= height[right]`:
  - The height at `left` is guaranteed to be smaller than or equal to some height at the right side.
  - Thus, the bottleneck is on the left. The water trapped at `left` is solely determined by `left_max`.
  - If `height[left] >= left_max`, we update `left_max = height[left]` (no water trapped).
  - Else, we trap `left_max - height[left]` water.
  - Move the `left` pointer to the right (`left++`).
- If `height[left] > height[right]`:
  - The height at `right` is smaller than the height at `left`.
  - Thus, the bottleneck is on the right. The water trapped at `right` is determined by `right_max`.
  - If `height[right] >= right_max`, we update `right_max = height[right]` (no water trapped).
  - Else, we trap `right_max - height[right]` water.
  - Move the `right` pointer to the left (`right--`).

This elegant two-pointer approach eliminates the need for extra arrays, reducing space complexity to $O(1)$ while retaining $O(N)$ time complexity.

---

## 4. Pattern Recognition

- **Pattern:** Two Pointers.
- **Why this topic?** The problem involves boundaries on both ends of a sequence. A two-pointer approach starting from both ends of the array allows us to process elements by shrinking the window from the side that has the smaller current boundary.

---

## 5. Brute Force Solution

### Algorithm
1. Initialize `total_water = 0`.
2. Loop `i` from `0` to `N-1`:
   - Initialize `left_max = 0`, `right_max = 0`.
   - Loop `j` from `i` down to `0`: `left_max = max(left_max, height[j])`.
   - Loop `j` from `i` to `N-1`: `right_max = max(right_max, height[j])`.
   - Add `min(left_max, right_max) - height[i]` to `total_water`.
3. Return `total_water`.

### Complexity Analysis
- **Time Complexity:** $O(N^2)$ because of nested scans for each index.
- **Space Complexity:** $O(1)$ auxiliary space.

### Brute Force C++17 Code
```cpp
#include <vector>
#include <algorithm>

class Solution {
public:
    int trapBrute(const std::vector<int>& height) {
        int n = height.size();
        int total_water = 0;
        
        for (int i = 0; i < n; ++i) {
            int left_max = 0;
            int right_max = 0;
            
            // Find the maximum element to the left of i
            for (int j = i; j >= 0; --j) {
                left_max = std::max(left_max, height[j]);
            }
            
            // Find the maximum element to the right of i
            for (int j = i; j < n; ++j) {
                right_max = std::max(right_max, height[j]);
            }
            
            // Water trapped at bar i
            total_water += std::min(left_max, right_max) - height[i];
        }
        return total_water;
    }
};
```

---

## 6. Better Solution (Precomputed Prefix/Suffix Max)

### Algorithm
1. Initialize two vectors: `left_max` and `right_max` of size `N`.
2. Fill `left_max`:
   - `left_max[0] = height[0]`.
   - Loop `i` from `1` to `N-1`: `left_max[i] = max(left_max[i-1], height[i])`.
3. Fill `right_max`:
   - `right_max[N-1] = height[N-1]`.
   - Loop `i` from `N-2` down to `0`: `right_max[i] = max(right_max[i+1], height[i])`.
4. Calculate water trapped:
   - Loop `i` from `0` to `N-1`: `total_water += min(left_max[i], right_max[i]) - height[i]`.
5. Return `total_water`.

### Complexity Analysis
- **Time Complexity:** $O(N)$ because we traverse the array three times.
- **Space Complexity:** $O(N)$ to store the `left_max` and `right_max` arrays.

### Better C++17 Code
```cpp
#include <vector>
#include <algorithm>

class Solution {
public:
    int trapBetter(const std::vector<int>& height) {
        int n = height.size();
        if (n < 3) return 0;
        
        std::vector<int> left_max(n);
        std::vector<int> right_max(n);
        
        // Fill left_max array
        left_max[0] = height[0];
        for (int i = 1; i < n; ++i) {
            left_max[i] = std::max(left_max[i - 1], height[i]);
        }
        
        // Fill right_max array
        right_max[n - 1] = height[n - 1];
        for (int i = n - 2; i >= 0; --i) {
            right_max[i] = std::max(right_max[i + 1], height[i]);
        }
        
        // Calculate trapped water
        int total_water = 0;
        for (int i = 0; i < n; ++i) {
            total_water += std::min(left_max[i], right_max[i]) - height[i];
        }
        
        return total_water;
    }
};
```

---

## 7. Optimal Solution (Two-Pointer Approach)

### Algorithm
1. Initialize pointers `left = 0`, `right = N - 1`.
2. Initialize variables `left_max = 0`, `right_max = 0`, and `total_water = 0`.
3. While `left < right`:
   - If `height[left] <= height[right]`:
     - If `height[left] >= left_max`, update `left_max = height[left]`.
     - Else, add `left_max - height[left]` to `total_water`.
     - Increment `left`.
   - Else:
     - If `height[right] >= right_max`, update `right_max = height[right]`.
     - Else, add `right_max - height[right]` to `total_water`.
     - Decrement `right`.
4. Return `total_water`.

### Why this is Optimal
It processes the array in a single pass ($O(N)$ time) and requires no extra arrays ($O(1)$ space). It operates on the principle that the water height at any bar is restricted by the shorter of its two bounding walls. By comparing `height[left]` and `height[right]`, we always proceed from the side that is guaranteed to be bounded by the other.

### C++17 Optimal Code
```cpp
#include <vector>
#include <algorithm>

class Solution {
public:
    int trap(const std::vector<int>& height) {
        int n = height.size();
        if (n < 3) return 0;
        
        int left = 0;
        int right = n - 1;
        
        int left_max = 0;
        int right_max = 0;
        
        int total_water = 0;
        
        while (left < right) {
            // The water level is limited by the shorter height
            if (height[left] <= height[right]) {
                if (height[left] >= left_max) {
                    // Update the left boundary
                    left_max = height[left];
                } else {
                    // Trap water above the left bar
                    total_water += left_max - height[left];
                }
                ++left;
            } else {
                if (height[right] >= right_max) {
                    // Update the right boundary
                    right_max = height[right];
                } else {
                    // Trap water above the right bar
                    total_water += right_max - height[right];
                }
                --right;
            }
        }
        
        return total_water;
    }
};
```

---

## 8. Code Walkthrough

1. **Safety Check**: `if (n < 3) return 0;` handles cases where water trapping is impossible.
2. **Pointer Initialization**: `left` starts at 0, `right` starts at the last index.
3. **Loop Condition**: `while (left < right)` ensures the two pointers don't overlap, which would result in double-counting.
4. **Branching**:
   - `if (height[left] <= height[right])`: We know that the right side has a bar that is at least as high as `height[left]`. Thus, the water level at `left` is bounded by `left_max`.
     - Update `left_max` if the current height is greater.
     - Otherwise, compute water: `left_max - height[left]`.
     - Shift `left` pointer rightward.
   - `else`: The left side has a bar taller than `height[right]`. Thus, the water level at `right` is bounded by `right_max`.
     - Update `right_max` if the current height is greater.
     - Otherwise, compute water: `right_max - height[right]`.
     - Shift `right` pointer leftward.

---

## 9. Dry Run

Let's dry run the optimal solution on `height = [0, 1, 0, 2, 1, 0, 1, 3]`.

- Init: `left = 0`, `right = 7`, `left_max = 0`, `right_max = 0`, `total_water = 0`

| Step | `left` | `right` | `height[left]` | `height[right]` | Decision | `left_max` | `right_max` | Water Added | `total_water` |
|:---:|:---:|:---:|:---:|:---:|:---|:---:|:---:|:---:|:---:|
| 1 | 0 | 7 | 0 | 3 | `height[left] <= height[right]` | `max(0,0)=0` | 0 | 0 | 0 |
| 2 | 1 | 7 | 1 | 3 | `height[left] <= height[right]` | `max(0,1)=1` | 0 | 0 | 0 |
| 3 | 2 | 7 | 0 | 3 | `height[left] <= height[right]` | 1 | 0 | `1 - 0 = 1` | 1 |
| 4 | 3 | 7 | 2 | 3 | `height[left] <= height[right]` | `max(1,2)=2` | 0 | 0 | 1 |
| 5 | 4 | 7 | 1 | 3 | `height[left] <= height[right]` | 2 | 0 | `2 - 1 = 1` | 2 |
| 6 | 5 | 7 | 0 | 3 | `height[left] <= height[right]` | 2 | 0 | `2 - 0 = 2` | 4 |
| 7 | 6 | 7 | 1 | 3 | `height[left] <= height[right]` | 2 | 0 | `2 - 1 = 1` | 5 |
| 8 | 7 | 7 | - | - | Loop terminates (`left == right`) | - | - | - | **5** |

**Final Result:** `5`

---

## 10. Complexity Analysis

- **Time Complexity:** 
  - **Best Case:** $O(N)$ (no matter what, the two pointers meet at the center, visiting each element once).
  - **Average Case:** $O(N)$.
  - **Worst Case:** $O(N)$.
- **Space Complexity:**
  - **Auxiliary Space:** $O(1)$ since only a few integer variables are kept in memory.

---

## 11. Common Mistakes

1. **Incorrect pointer updates**: Forgetting to increment `left` or decrement `right` inside the conditional branches, which leads to infinite loops.
2. **Comparing boundary values incorrectly**: Branching based on `left_max <= right_max` instead of `height[left] <= height[right]`. Comparing the actual heights at the pointers is correct because it guarantees the existence of a taller bounding bar on the opposite side.
3. **Double counting the meeting element**: Continuing the loop when `left == right`. The loop condition must be strictly `left < right`.

---

## 12. Edge Cases

| Edge Case | Input Example | Expected Output | Why it matters |
|:---|:---|:---|:---|
| **Fewer than 3 elements** | `[2, 3]` | `0` | No container can be formed. |
| **Strictly Decreasing** | `[3, 2, 1, 0]` | `0` | Water simply runs off the edge. |
| **Strictly Increasing** | `[0, 1, 2, 3]` | `0` | Water runs off the edge. |
| **Flat elevation** | `[2, 2, 2]` | `0` | No valleys to trap water. |
| **U-Shaped / Valley** | `[3, 0, 3]` | `3` | Validates that a single large basin traps water correctly. |

---

## 13. Interview Explanation

"To solve the Trapping Rain Water problem, we must compute how much water can be trapped above each bar. The water trapped on top of a bar at index `i` is determined by the formula: `min(tallest bar to its left, tallest bar to its right) - height[i]`. 

We can solve this using three different approaches:
1. **Brute Force**: For each bar, search left and right for the maximum heights. This takes $O(N^2)$ time.
2. **Prefix/Suffix Max Arrays**: Precompute the maximum heights to the left and right in two arrays. This reduces the time complexity to $O(N)$ but requires $O(N)$ auxiliary space.
3. **Two-Pointer Approach (Optimal)**: We maintain two pointers, `left` at the beginning and `right` at the end of the array, along with `left_max` and `right_max`. Because the water level is bounded by the shorter wall, we can compare `height[left]` and `height[right]`. If `height[left]` is smaller, we know the bottleneck is on the left, so we compute the water trapped at `left` using `left_max` and move the `left` pointer inward. If `height[right]` is smaller, we do the same on the right. This achieves $O(N)$ time complexity and reduces auxiliary space to $O(1)$."

---

## 14. Follow-up Questions

- **Q:** Can we solve this using a Stack?
  - **A:** Yes, we can use a **Monotonic Decreasing Stack** storing indices. When we find a bar taller than the stack's top, it acts as a right boundary. We pop the top of the stack as the bottom of the basin, and the new top of the stack is the left boundary. The trapped water is calculated horizontally instead of vertically.
- **Q:** How does this problem change if the width of each bar varies?
  - **A:** The height calculation remains the same, but when adding to the total water, we multiply the trapped height by the width of the bar.

---

## 15. Variations

- **Container With Most Water:** Find two lines that together with the x-axis forms a container such that the container contains the most water (no intermediate bars).
- **Trapping Rain Water II (3D):** Finding trapped water on a 2D grid of bars. Resolved using a Min-Heap (Priority Queue) starting from the boundary cells (Dijkstra-like boundary shrinkage).

---

## 16. Revision Notes

- **Basic Formula:** `water[i] = min(left_max, right_max) - height[i]`.
- **Optimal Strategy:** Two-pointer scan starting at `0` and `N-1`.
- **Pointers Update Logic:** Compare `height[left]` and `height[right]`. Move the smaller pointer inward to resolve the bottleneck.
- **Complexity:** $O(N)$ Time, $O(1)$ Space.

---

## 17. Memorization Version

- **Pointers:** `left = 0`, `right = n - 1`, `left_max = 0`, `right_max = 0`.
- **Loop** `while (left < right)`:
  - If `height[left] <= height[right]`:
    - `height[left] >= left_max ? left_max = height[left] : ans += left_max - height[left]`
    - `left++`
  - Else:
    - `height[right] >= right_max ? right_max = height[right] : ans += right_max - height[right]`
    - `right--`
- **Complexity:** $O(N)$ time, $O(1)$ space.
