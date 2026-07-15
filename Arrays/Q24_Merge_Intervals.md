# Merge Intervals

## 1. Problem Statement

Given an array of `intervals` where `intervals[i] = [start_i, end_i]`, merge all overlapping intervals, and return an array of the non-overlapping intervals that cover all the input intervals.

### Examples
- **Input:** `intervals = [[1, 3], [2, 6], [8, 10], [15, 18]]`
- **Output:** `[[1, 6], [8, 10], [15, 18]]`
  *(Explanation: Since intervals `[1, 3]` and `[2, 6]` overlap, they are merged into `[1, 6]`.)*

- **Input:** `intervals = [[1, 4], [4, 5]]`
- **Output:** `[[1, 5]]`
  *(Explanation: Intervals `[1, 4]` and `[4, 5]` are considered overlapping because they touch at `4`.)*

### Constraints
- $1 \le intervals.length \le 10^4$
- `intervals[i].length == 2`
- $0 \le start_i \le end_i \le 10^4$

---

## 2. Interview Clarification Questions

Before writing any code, a candidate should clarify:
1. **Q:** Are the intervals sorted initially?
   * **A:** No, they can be in any order.
2. **Q:** What is considered overlapping? If two intervals touch at a point (e.g., `[1, 4]` and `[4, 5]`), do we merge them?
   * **A:** Yes, they overlap if the start of one is less than or equal to the end of the other.
3. **Q:** Can we modify the input array?
   * **A:** Yes, merging in-place inside the input array is highly encouraged to optimize space.
4. **Q:** Can the intervals have negative bounds?
   * **A:** The constraints state values are non-negative, but the same logic would hold for negative bounds.

---

## 3. Thinking Process

### Step 1: First Instinct (Brute Force)
Without sorting, we would have to check every interval against all other intervals to see if they overlap.
- If they overlap, we merge them into a single interval and remove the old ones.
- We would need to repeat this process until no more merges can be made.
- Since we have to continually search the array for overlapping elements, this nested searching leads to an $O(N^2)$ or $O(N^3)$ time complexity.

### Step 2: Sorting for Simplification (Optimal Insight)
How can we check for overlaps in a single pass?
- If we sort the intervals by their start times, any overlapping intervals will become adjacent in the array.
- For example, if we have `[[2, 6], [1, 3], [8, 10]]`, sorting by start time gives `[[1, 3], [2, 6], [8, 10]]`.
- Once sorted, we can use a **single-pass sweep** (linear scan):
  - Start with the first interval as our "current active interval".
  - Iterate through the remaining intervals:
    - If the next interval's start time is **less than or equal to** the active interval's end time, they overlap. We merge them by updating the active interval's end time to `max(active.end, next.end)`.
    - If the next interval's start time is **greater than** the active interval's end time, they do not overlap. The active interval is complete. We save it, and make the next interval the new active interval.

### Step 3: Space Optimization (In-Place vs. New List)
- **New List Approach:** We push merged intervals into a new vector. This is easier to implement but uses $O(N)$ extra space.
- **In-Place Approach:** We can reuse the input vector by maintaining a write pointer `writeIdx`. 
  - We write merged intervals back to the input array at `intervals[writeIdx]`.
  - At the end, we resize the input vector to `writeIdx + 1`. This reduces auxiliary space to $O(1)$ (excluding sorting stack space).

---

## 4. Pattern Recognition

This problem fits the **Interval Sorting / Sweep Line** pattern.
- **Why?** Working with ranges or intervals almost always requires sorting by start times. Sorting reduces the comparison space from all pairs $O(N^2)$ to just adjacent pairs $O(N)$.

---

## 5. Brute Force Solution

### Algorithm
1. Keep track of which intervals have been merged using a boolean array `visited`.
2. Compare each interval `i` with every other interval `j`:
   - If they overlap and neither is visited:
     - Merge them by setting `start_i = min(start_i, start_j)` and `end_i = max(end_i, end_j)`.
     - Mark `j` as visited.
     - Reset the search to check the newly expanded interval against all others.
3. Push all unvisited (or newly formed) intervals into a result list.

### Complexity
- **Time Complexity:** $O(N^2)$ due to recursive merges and searching.
- **Space Complexity:** $O(N)$ for the `visited` array.

### C++17 Code
```cpp
#include <vector>
#include <algorithm>

std::vector<std::vector<int>> mergeBrute(std::vector<std::vector<int>>& intervals) {
    int n = intervals.size();
    if (n <= 1) return intervals;
    
    std::vector<bool> merged(n, false);
    
    for (int i = 0; i < n; ++i) {
        if (merged[i]) continue;
        bool hasMerge = true;
        while (hasMerge) {
            hasMerge = false;
            for (int j = 0; j < n; ++j) {
                if (i != j && !merged[j]) {
                    // Check overlap
                    if (std::max(intervals[i][0], intervals[j][0]) <= std::min(intervals[i][1], intervals[j][1])) {
                        intervals[i][0] = std::min(intervals[i][0], intervals[j][0]);
                        intervals[i][1] = std::max(intervals[i][1], intervals[j][1]);
                        merged[j] = true;
                        hasMerge = true; // Restart check with updated interval
                    }
                }
            }
        }
    }
    
    std::vector<std::vector<int>> result;
    for (int i = 0; i < n; ++i) {
        if (!merged[i]) {
            result.push_back(intervals[i]);
        }
    }
    return result;
}
```

---

## 6. Better Solution

Sorting the intervals first is the primary optimization. We can write a simple version that pushes elements into a new vector. This runs in $O(N \log N)$ time and uses $O(N)$ space.

Since this is the standard approach, we will detail it and the in-place $O(1)$ space optimization in the **Optimal Solution** section.

---

## 7. Optimal Solution

We present two implementations:
1. **Standard Optimal:** Uses a new list, highly readable.
2. **In-place Optimal:** Modifies the input vector directly to achieve $O(1)$ auxiliary space.

### Algorithm (In-Place Sweep)
1. Sort the `intervals` array in ascending order of start times: `intervals[i][0]`.
2. Initialize a write index `writeIdx = 0`.
3. Iterate `i` from `1` to `N - 1`:
   - If the current interval `intervals[i]` overlaps with the active merged interval at `intervals[writeIdx]`:
     - Update the end of the active interval: `intervals[writeIdx][1] = max(intervals[writeIdx][1], intervals[i][1])`.
   - If they do not overlap:
     - Increment `writeIdx`.
     - Copy `intervals[i]` to `intervals[writeIdx]`.
4. Resize the `intervals` vector to `writeIdx + 1` elements.
5. Return the modified `intervals` vector.

### C++17 Code

```cpp
#include <vector>
#include <algorithm>

// Version 1: Standard Optimal (using extra result vector)
std::vector<std::vector<int>> merge(std::vector<std::vector<int>>& intervals) {
    if (intervals.empty()) return {};
    
    // Sort by start times
    std::sort(intervals.begin(), intervals.end(), [](const auto& a, const auto& b) {
        return a[0] < b[0];
    });
    
    std::vector<std::vector<int>> merged;
    merged.push_back(intervals[0]);
    
    for (size_t i = 1; i < intervals.size(); ++i) {
        // If current interval overlaps with the last merged interval
        if (intervals[i][0] <= merged.back()[1]) {
            merged.back()[1] = std::max(merged.back()[1], intervals[i][1]);
        } else {
            // No overlap, append current interval
            merged.push_back(intervals[i]);
        }
    }
    return merged;
}

// Version 2: In-place Space Optimized (O(1) auxiliary space)
std::vector<std::vector<int>> mergeInPlace(std::vector<std::vector<int>>& intervals) {
    if (intervals.size() <= 1) return intervals;
    
    // Sort by start times
    std::sort(intervals.begin(), intervals.end(), [](const auto& a, const auto& b) {
        return a[0] < b[0];
    });
    
    int writeIdx = 0;
    
    for (size_t i = 1; i < intervals.size(); ++i) {
        // If current interval overlaps with the interval at writeIdx
        if (intervals[i][0] <= intervals[writeIdx][1]) {
            intervals[writeIdx][1] = std::max(intervals[writeIdx][1], intervals[i][1]);
        } else {
            // Move write pointer and copy the non-overlapping interval
            writeIdx++;
            intervals[writeIdx] = intervals[i];
        }
    }
    
    // Shrink vector to keep only the merged elements
    intervals.resize(writeIdx + 1);
    return intervals;
}
```

---

## 8. Code Walkthrough (In-Place Version)

1. **Sort:** `std::sort` arranges the intervals. The lambda `[](const auto& a, const auto& b) { return a[0] < b[0]; }` ensures sorting is based on the start index.
2. **Pointers:** `writeIdx = 0` points to the last finalized merged interval.
3. **Loop:** We start scanning from index `1`.
4. **Overlap Check:** `intervals[i][0] <= intervals[writeIdx][1]` checks if the current interval's start is within the active interval.
   - If true, they overlap. We update `intervals[writeIdx][1]` to the maximum end value.
5. **No-Overlap Handling:** If false, the active interval is finalized. We increment `writeIdx` and overwrite `intervals[writeIdx]` with the current interval `intervals[i]`.
6. **Resize:** `intervals.resize(writeIdx + 1)` deletes the garbage values beyond the merged count.

---

## 9. Dry Run

Input: `intervals = [[2, 6], [1, 3], [8, 10], [15, 18]]`

### Sort Step:
Sorted: `[[1, 3], [2, 6], [8, 10], [15, 18]]`

### Iterations:
`writeIdx = 0`, active interval is `intervals[0] = [1, 3]`

| Loop Index `i` | Interval `intervals[i]` | Overlap Check (`start_i <= end_active`) | Action | Updated `intervals` | `writeIdx` |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1** | `[2, 6]` | `2 <= 3` (True) | Merge: `end_active = max(3, 6) = 6` | `[[1, 6], [2, 6], [8, 10], [15, 18]]` | 0 |
| **2** | `[8, 10]` | `8 <= 6` (False) | New active: `writeIdx = 1`, Copy to index 1 | `[[1, 6], [8, 10], [8, 10], [15, 18]]` | 1 |
| **3** | `[15, 18]` | `15 <= 10` (False) | New active: `writeIdx = 2`, Copy to index 2 | `[[1, 6], [8, 10], [15, 18], [15, 18]]` | 2 |

### Resizing:
Resize to `writeIdx + 1 = 3`. 
Resulting Array: `[[1, 6], [8, 10], [15, 18]]`.

---

## 10. Complexity Analysis

- **Time Complexity:** $O(N \log N)$ where $N$ is the number of intervals. Sorting takes $O(N \log N)$ time, and the linear sweep takes $O(N)$ time.
- **Space Complexity:**
  - **Standard Version:** $O(N)$ to store the merged intervals.
  - **In-Place Version:** $O(1)$ auxiliary space (ignoring sorting recursion stack space which is $O(\log N)$).

---

## 11. Common Mistakes

1. **Forgetting to sort:** If the intervals are not sorted, the sweep line fails to detect overlaps between non-adjacent inputs.
2. **Incorrect sorting comparator:** Sorting by end times instead of start times. While possible, sorting by end times makes the merge logic more complicated.
3. **Missing boundary touch conditions:** Using `<` instead of `<=` when checking for overlaps. For example, `[1, 2]` and `[2, 3]` touch at `2` and must be merged.

---

## 12. Edge Cases

| Edge Case | Input Example | Why it matters | Correct Output |
| :--- | :--- | :--- | :--- |
| **Single Interval** | `[[1, 4]]` | Loop doesn't run; returns input. | `[[1, 4]]` |
| **Identical Intervals** | `[[1, 2], [1, 2]]` | Merges into one. | `[[1, 2]]` |
| **Fully Nested Intervals** | `[[1, 10], [2, 3]]` | Inner interval is completely absorbed. | `[[1, 10]]` |
| **No Overlaps** | `[[1, 2], [3, 4]]` | Returns unchanged sorted intervals. | `[[1, 2], [3, 4]]` |

---

## 13. Interview Explanation

"To merge overlapping intervals, the brute force approach is to compare every interval with all other intervals, which takes $O(N^2)$ time. 

We can optimize this to $O(N \log N)$ time by first sorting the intervals based on their start times. Once sorted, any intervals that can be merged will be adjacent to each other. 

We can then perform a linear sweep. We start with the first interval as our active interval. For each subsequent interval, we check if its start time is less than or equal to the end time of our active interval. If it is, they overlap, and we merge them by extending the active interval's end time to the maximum of the two. If it isn't, they don't overlap, so we save the active interval and start a new active interval with the current one. 

To optimize auxiliary space to $O(1)$, we can perform this merge in-place by writing the merged intervals back into the input vector and resizing it at the end."

---

## 14. Follow-up Questions

1. **Q: How do we handle this if the intervals are arriving dynamically as a stream?**
   * *A:* This is the "Insert Interval" problem. If we have a set of non-overlapping intervals, and a new interval is added, we can find its insertion position in $O(\log N)$ using binary search, and then merge overlapping intervals in $O(N)$ time.
2. **Q: Can we sort in $O(N)$ time if the start coordinates are within a small range?**
   * *A:* Yes, if start coordinates are small integers, we can use Radix Sort or Counting Sort to sort in $O(N)$ time, reducing the overall complexity to $O(N)$.

---

## 15. Variations

1. **Insert Interval:** Insert a new interval into a sorted list of non-overlapping intervals and merge if necessary.
2. **Meeting Rooms:** Given array of meeting time intervals, determine if a person could attend all meetings (returns true if no overlaps exist).

---

## 16. Revision Notes

- **Sorting Key:** Always sort by start times: `intervals[i][0]`.
- **Merge Check:** `current_start <= previous_end`.
- **In-place technique:** Reuse input array using a write pointer and `vector::resize()`.

---

## 17. Memorization Version

```cpp
// 1-min Review Version: In-place
vector<vector<int>> merge(vector<vector<int>>& intervals) {
    if (intervals.size() <= 1) return intervals;
    sort(intervals.begin(), intervals.end());
    int writeIdx = 0;
    for (int i = 1; i < intervals.size(); ++i) {
        if (intervals[i][0] <= intervals[writeIdx][1]) {
            intervals[writeIdx][1] = max(intervals[writeIdx][1], intervals[i][1]);
        } else {
            intervals[++writeIdx] = intervals[i];
        }
    }
    intervals.resize(writeIdx + 1);
    return intervals;
}
```
*Complexity:* $O(N \log N)$ Time, $O(1)$ Auxiliary Space.
