# Leaders in an Array

---

## 1. Problem Statement

Given an integer array `arr` of size $N$, find all the "leaders" in the array. An element is a leader if it is strictly greater than all the elements to its right side. The rightmost element is always a leader.

*   **Input:** An integer array `arr` of size $N$.
*   **Output:** A list containing all the leaders in the array.
*   **Constraints:**
    *   $1 \le N \le 10^6$
    *   $-10^9 \le arr[i] \le 10^9$
*   **Observations:**
    *   The last element has no elements to its right, so it is always a leader.
    *   If an element is a leader, it must be larger than the maximum element to its right.
    *   The leaders do not have to be in any specific sorted order, but they should generally be returned in either their original order of appearance (left-to-right) or reversed order (right-to-left), as specified by the interviewer.

---

## 2. Interview Clarification

Before coding, clarify the following with your interviewer:
1.  **Should the leaders be returned in the order they appear in the original array (left-to-right) or right-to-left?**
    *   *Answer:* Usually left-to-right (original order). If our optimal solution scans from right to left, we should reverse the results before returning.
2.  **What if there are duplicates? For example, in `[6, 6, 3]`, is the first `6` a leader?**
    *   *Answer:* The definition says "strictly greater than all elements to its right". So the first `6` is not a leader because it is not strictly greater than the second `6`.
3.  **Can the array contain negative numbers?**
    *   *Answer:* Yes, elements can be negative.
4.  **Can the array be empty?**
    *   *Answer:* Constraints state $N \ge 1$. If $N = 1$, that single element is the leader.

---

## 3. Thinking Process

*   **First Instinct (Brute Force):** For every element at index `i`, scan all elements from index `i + 1` to `N - 1`. If we find any element that is greater than or equal to `arr[i]`, then `arr[i]` is not a leader. If we reach the end of the array without finding any such element, then `arr[i]` is a leader. This takes $O(N^2)$ time.
*   **Optimization (Scanning from Right):**
    *   Can we avoid scanning the right side repeatedly?
    *   If we scan from the rightmost element (index `N - 1`) to the left (index `0`), we can maintain the maximum value seen so far.
    *   Let `max_from_right` represent the maximum element to the right of the current index.
    *   For the current element `arr[i]`:
        *   If `arr[i] > max_from_right`, then `arr[i]` is strictly greater than all elements to its right. Thus, it is a leader. We add it to our leaders list and update `max_from_right = arr[i]`.
        *   Otherwise, it is not a leader.
    *   By doing a single pass from right to left, we solve the problem in $O(N)$ time with $O(1)$ auxiliary space.
    *   Since we collect leaders from right to left, we just reverse the collection at the end to restore the original left-to-right order.

---

## 4. Pattern Recognition

*   **Topic:** Arrays.
*   **Pattern:** **Scan from Right / Suffixed Accumulation**.
*   *Why this pattern?* The property of being a "leader" depends entirely on elements to the right of the current position. Processing elements from right-to-left allows us to build the suffix information (in this case, the suffix maximum) incrementally.

---

## 5. Brute Force Solution

### Algorithm
1.  Initialize an empty list/vector `leaders`.
2.  Iterate `i` from `0` to `N - 1`:
    *   Set a boolean flag `isLeader = true`.
    *   Iterate `j` from `i + 1` to `N - 1`:
        *   If `arr[i] <= arr[j]`, set `isLeader = false` and break the inner loop.
    *   If `isLeader` is true, add `arr[i]` to `leaders`.
3.  Return `leaders`.

### Dry Run
Input: `arr = [16, 17, 4, 3, 5, 2]`, $N = 6$
*   `i = 0` (`16`): Compare with `17, 4, 3, 5, 2`. Since `16 <= 17`, `isLeader = false`.
*   `i = 1` (`17`): Compare with `4, 3, 5, 2`. All are smaller. `isLeader = true`. Add `17`.
*   `i = 2` (`4`): Compare with `3, 5, 2`. Since `4 <= 5`, `isLeader = false`.
*   `i = 3` (`3`): Compare with `5, 2`. Since `3 <= 5`, `isLeader = false`.
*   `i = 4` (`5`): Compare with `2`. `5 > 2`. `isLeader = true`. Add `5`.
*   `i = 5` (`2`): No elements to right. `isLeader = true`. Add `2`.
*   Output: `[17, 5, 2]`

### Complexity
*   **Time Complexity:** $O(N^2)$ in the worst case (e.g., sorted in ascending order or flat array).
*   **Space Complexity:** $O(1)$ auxiliary space (excluding the output list).

### Pros/Cons
*   **Pros:** Straightforward to implement, requires no extra thinking.
*   **Cons:** Inefficient. TLE (Time Limit Exceeded) for $N \ge 10^5$.

### Code (C++17)
```cpp
#include <vector>

class SolutionBrute {
public:
    std::vector<int> getLeaders(const std::vector<int>& arr) {
        std::vector<int> leaders;
        int n = arr.size();
        
        for (int i = 0; i < n; ++i) {
            bool isLeader = true;
            for (int j = i + 1; j < n; ++j) {
                if (arr[i] <= arr[j]) {
                    isLeader = false;
                    break;
                }
            }
            if (isLeader) {
                leaders.push_back(arr[i]);
            }
        }
        return leaders;
    }
};
```

---

## 6. Better Solution

For this problem, there is no intermediate "better" solution between the $O(N^2)$ brute force and the $O(N)$ optimal scan from right. Thus, we jump directly to the optimal solution.

---

## 7. Optimal Solution

### Algorithm
1.  Initialize an empty list `leaders`.
2.  Set `max_from_right` to a very small value, or initialize it with the last element `arr[N - 1]`.
3.  Add `arr[N - 1]` to `leaders` (the rightmost element is always a leader).
4.  Iterate `i` from `N - 2` down to `0`:
    *   If `arr[i] > max_from_right`, then `arr[i]` is a leader:
        *   Add `arr[i]` to `leaders`.
        *   Update `max_from_right = arr[i]`.
5.  Since we processed the array from right to left, reverse the `leaders` list to match the original left-to-right order.
6.  Return the reversed list.

### Dry Run
Input: `arr = [16, 17, 4, 3, 5, 2]`, $N = 6$
*   Initialize: `max_from_right = arr[5] = 2`. Add `2` to `leaders`. `leaders = [2]`.
*   `i = 4` (`arr[4] = 5`): Is `5 > 2`? Yes. Add `5` to `leaders`. Update `max_from_right = 5`. `leaders = [2, 5]`.
*   `i = 3` (`arr[3] = 3`): Is `3 > 5`? No.
*   `i = 2` (`arr[2] = 4`): Is `4 > 5`? No.
*   `i = 1` (`arr[1] = 17`): Is `17 > 5`? Yes. Add `17` to `leaders`. Update `max_from_right = 17`. `leaders = [2, 5, 17]`.
*   `i = 0` (`arr[0] = 16`): Is `16 > 17`? No.
*   Reversed: `[17, 5, 2]`. Return this array.

### Why Optimal
*   We only visit each element exactly once.
*   We only need $O(1)$ space to track the maximum element seen so far.
*   It is impossible to find leaders in less than $O(N)$ time because we must examine every element at least once.

---

### Beautiful C++17 Code

```cpp
#include <vector>
#include <algorithm>

class SolutionOptimal {
public:
    std::vector<int> getLeaders(const std::vector<int>& arr) {
        std::vector<int> leaders;
        int n = arr.size();
        if (n == 0) return leaders;
        
        // Rightmost element is always a leader
        int maxFromRight = arr[n - 1];
        leaders.push_back(maxFromRight);
        
        // Scan the array from right to left
        for (int i = n - 2; i >= 0; --i) {
            if (arr[i] > maxFromRight) {
                leaders.push_back(arr[i]);
                maxFromRight = arr[i]; // Update the maximum seen so far
            }
        }
        
        // Reverse the leaders vector to restore original left-to-right order
        std::reverse(leaders.begin(), leaders.end());
        return leaders;
    }
};
```

---

## 8. Code Walkthrough

1.  `if (n == 0) return leaders;`
    Edge case handling for empty input.
2.  `int maxFromRight = arr[n - 1];`
    Initializes the tracking variable with the rightmost element, as there are no elements to its right.
3.  `leaders.push_back(maxFromRight);`
    Directly adds the rightmost element to the leaders list.
4.  `for (int i = n - 2; i >= 0; --i) { ... }`
    Loop starts from the second to last element and moves leftwards.
5.  `if (arr[i] > maxFromRight) { leaders.push_back(arr[i]); maxFromRight = arr[i]; }`
    If the current element is strictly greater than the maximum element found to its right, it qualifies as a leader. We record it and update the suffix maximum.
6.  `std::reverse(leaders.begin(), leaders.end());`
    Reverses the list to match the original array order.

---

## 9. Dry Run

Given `arr = [10, 22, 12, 3, 0, 6]`, $N = 6$ (expected: `[22, 12, 6]`)

| Step | Index `i` | Value `arr[i]` | Condition: `arr[i] > maxFromRight` | `maxFromRight` | `leaders` (before reverse) |
| :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | `5` | `6` | (Rightmost, always added) | `6` | `[6]` |
| 2 | `4` | `0` | `0 > 6` (False) | `6` | `[6]` |
| 3 | `3` | `3` | `3 > 6` (False) | `6` | `[6]` |
| 4 | `2` | `12` | `12 > 6` (True) | `12` | `[6, 12]` |
| 5 | `1` | `22` | `22 > 12` (True) | `22` | `[6, 12, 22]` |
| 6 | `0` | `10` | `10 > 22` (False) | `22` | `[6, 12, 22]` |
| 7 | Reverse | — | — | — | `[22, 12, 6]` |

---

## 10. Complexity Analysis

| Approach | Time Complexity (Best/Avg/Worst) | Space Complexity (Auxiliary) | Reasoning |
| :--- | :--- | :--- | :--- |
| **Brute Force** | $O(N^2)$ | $O(1)$ | Double nested loops scan the suffix for every element. |
| **Optimal** | $O(N)$ | $O(1)$ | Single pass from right-to-left. Reversing the output takes $O(N)$ time. |

---

## 11. Common Mistakes

1.  **Handling Duplicate Elements incorrectly:** Letting non-strictly greater elements become leaders (e.g., classifying `[5, 5, 2]` as having two `5` leaders).
    *   *Correction:* Ensure strictly greater logic `arr[i] > maxFromRight` is used rather than `arr[i] >= maxFromRight`.
2.  **Forgetting to reverse the output:** Returning leaders in right-to-left order when the interviewer expects them in original index order.
3.  **Out-of-bounds index on small inputs:** Doing `arr[n - 2]` without checking if `n >= 2`.

---

## 12. Edge Cases

| Edge Case | Input Example | Expected Output | Why It Matters / How Code Handles It |
| :--- | :--- | :--- | :--- |
| **Single Element Array** | `[5]` | `[5]` | Loop doesn't execute; correctly returns the single element. |
| **Strictly Decreasing** | `[5, 4, 3, 2, 1]` | `[5, 4, 3, 2, 1]` | Every element is greater than its right neighbor. |
| **Strictly Increasing** | `[1, 2, 3, 4, 5]` | `[5]` | Only the rightmost element is a leader. |
| **All Duplicates** | `[4, 4, 4, 4]` | `[4]` | Only the last `4` is a leader since others are not strictly greater. |
| **Negative Values** | `[-5, -2, -8, -10]` | `[-2, -8, -10]` | Ensure comparisons work properly with negative numbers. |

---

## 13. Interview Explanation

> "To find all leaders in an array, we must identify elements that are strictly greater than all elements to their right.
> A naive approach involves checking all elements to the right of each element using nested loops. This takes $O(N^2)$ time.
> A much more efficient approach is to start scanning the array from the rightmost element and move towards the left. Since the rightmost element has nothing to its right, it is always a leader. We initialize our running maximum with this element. As we traverse leftwards, we compare each element with our running maximum. If the element is strictly greater, it must be a leader. We then add it to our list and update our running maximum.
> After finishing the pass, we reverse our list of leaders to restore the original left-to-right order. This optimal method executes in $O(N)$ time and uses $O(1)$ auxiliary space."

---

## 14. Follow-up Questions

1.  **Can we solve this without reversing the array at the end?**
    *   *Answer:* Yes, we can scan from left to right, but it is harder to maintain the suffix maximum. Alternatively, we can use a **Monotonic Stack** to keep track of leaders, but this uses $O(N)$ auxiliary space.
2.  **How would you modify the solution to find the leaders if leaders were defined as elements greater than all elements to their left?**
    *   *Answer:* We would scan from left to right, keeping track of the running maximum from the left. The first element is always a leader. We would not need to reverse the result list at the end.

---

## 15. Variations

*   **Next Greater Element (LeetCode 496):**
    *   *Difference:* Instead of finding if an element is greater than *all* elements to its right, find the immediate next element to its right that is greater. This requires a Monotonic Stack.
*   **Dominant Index (LeetCode 747):**
    *   *Difference:* Find if the largest element in the array is at least twice as large as every other element.

---

## 16. Revision Notes

*   **Suffix Max Technique:** Scanning from right-to-left allows you to keep track of suffix properties in a single pass.
*   **Strict Comparison:** Pay close attention to "strictly greater" vs "greater than or equal".
*   **Time Complexity:** $O(N)$ with $O(1)$ auxiliary space (excluding the output container).

---

## 17. Memorization Version

```cpp
vector<int> leaders(vector<int>& arr) {
    vector<int> ans;
    int n = arr.size();
    if (n == 0) return ans;
    
    int maxSoFar = arr[n - 1];
    ans.push_back(maxSoFar);
    
    for (int i = n - 2; i >= 0; --i) {
        if (arr[i] > maxSoFar) {
            maxSoFar = arr[i];
            ans.push_back(maxSoFar);
        }
    }
    reverse(ans.begin(), ans.end());
    return ans;
}
```
*Key reminder: Scan from right, track max, reverse result.*
