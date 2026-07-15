# Search in Nearly Sorted Array

## 1. Problem Statement
Given a sorted array of $N$ integers where each element is at most 1 position away from its sorted position. That is, the element at index $i$ in the sorted array could be present at index $i-1$, $i$, or $i+1$ in the given nearly sorted array. 
Find the index of a target element in this nearly sorted array. If the target is not present, return `-1`.

### Input
- `arr`: An array of $N$ integers representing the nearly sorted array.
- `target`: The value to search for.

### Output
- Return the 0-based index of the target if found, otherwise return `-1`.

### Constraints
- $1 \le N \le 10^5$
- $-10^9 \le \text{arr}[i] \le 10^9$
- The array elements are distinct.

### Observations
1. In a fully sorted array, the element at index `mid` divides the array. 
2. In a nearly sorted array, the element that *should* be at `mid` could be at index `mid - 1`, `mid`, or `mid + 1`.
3. Thus, during binary search, we can inspect three indices (`mid - 1`, `mid`, `mid + 1`) to see if any of them contains the target.
4. If the target is not found at those three positions, we can determine which half of the array to search next, similar to standard binary search, but adjusting our next step to avoid re-checking elements.

---

## 2. Interview Clarification
Before coding, a candidate should clarify:
1. **Q:** Can elements be out of bounds of the array when checking `mid - 1` and `mid + 1`?
   - **A:** Yes. We must perform boundary checks to ensure we do not access indices $< 0$ or $\ge N$.
2. **Q:** Can the array contain duplicates?
   - **A:** The problem description states elements are distinct. If duplicates were allowed, we could return any valid index.
3. **Q:** What does "nearly sorted" mean mathematically?
   - **A:** If the fully sorted version of the array is $S$, then for any index $i$, $\text{arr}[i]$ is either $S[i-1]$, $S[i]$, or $S[i+1]$.

---

## 3. Thinking Process
### First Instinct
A simple linear search can find the target in $O(N)$ time.
- However, this does not make use of the sorted property of the array.

### Can we do better?
Since the array is mostly sorted, we want to run a modified Binary Search:
- Let's define the search space using `low = 0` and `high = N - 1`.
- Calculate `mid = low + (high - low) / 2`.
- Check if the target is at:
  - `mid`: if `arr[mid] == target`, return `mid`.
  - `mid - 1` (if `mid - 1 >= low`): if `arr[mid - 1] == target`, return `mid - 1`.
  - `mid + 1` (if `mid + 1 <= high`): if `arr[mid + 1] == target`, return `mid + 1`.
- If the target is not at any of these three positions, how do we decide where to search next?
  - If `target < arr[mid]`, since the elements are mostly sorted and `target` is not at `mid - 1`, it must be in the left subarray. We can search in the range `[low, mid - 2]`. (We use `mid - 2` instead of `mid - 1` because we already checked `mid - 1`).
  - If `target > arr[mid]`, since the elements are mostly sorted and `target` is not at `mid + 1`, it must be in the right subarray. We can search in the range `[mid + 2, high]`. (We use `mid + 2` because we already checked `mid + 1`).
- This reduces the search space by half in each step, maintaining the logarithmic time complexity of binary search.

---

## 4. Pattern Recognition
This problem belongs to the **Modified Binary Search** pattern.
- Why? The structure of the search space is close to sorted, allowing us to make decisions to discard half of the array at each step, but requires checking adjacent neighbors of the midpoint to handle the local disorder.

---

## 5. Brute Force Solution
### Algorithm
1. Traverse the array from index $0$ to $N - 1$.
2. If `arr[i] == target`, return `i`.
3. If we loop through the entire array without finding the target, return `-1`.

### Complexity Analysis
- **Time Complexity:** $O(N)$ because we might inspect every element in the worst case.
- **Space Complexity:** $O(1)$ auxiliary space.

### Pros and Cons
- **Pros:** Extremely simple to implement; works even if the array is completely unsorted.
- **Cons:** Inefficient; ignores the nearly sorted property.

### C++17 Code
```cpp
#include <vector>

class Solution {
public:
    int searchBruteForce(const std::vector<int>& arr, int target) {
        int n = arr.size();
        for (int i = 0; i < n; ++i) {
            if (arr[i] == target) {
                return i;
            }
        }
        return -1;
    }
};
```

---

## 6. Better Solution
*Note: For this specific problem, there is no intermediate "Better" solution that sits between the brute force (linear search) and the optimal modified Binary Search. We jump directly from Brute Force to the Optimal Solution.*

---

## 7. Optimal Solution
### Algorithm
1. Initialize `low = 0` and `high = N - 1`.
2. While `low <= high`:
   - Calculate `mid = low + (high - low) / 2`.
   - Check if `arr[mid] == target`. If so, return `mid`.
   - Check if `mid - 1 >= low` and `arr[mid - 1] == target`. If so, return `mid - 1`.
   - Check if `mid + 1 <= high` and `arr[mid + 1] == target`. If so, return `mid + 1`.
   - If the target is not found:
     - If `target < arr[mid]`, search the left side: `high = mid - 2`.
     - Else, search the right side: `low = mid + 2`.
3. If the loop ends and the target is not found, return `-1`.

### Dry Run
`arr` = `[10, 3, 40, 20, 50, 80, 70]`, `target = 40`.
$N = 7$.
`low = 0`, `high = 6`.

**Iteration 1:**
- `mid = 0 + (6 - 0) / 2 = 3`
- Check `mid` indices:
  - `arr[mid] = arr[3] = 20` (not equal to 40)
  - `mid - 1 = 2` $\rightarrow$ `arr[2] = 40` (equal to 40!)
- Match found! Return index `2`.

Let's try another dry run where `target = 90` (not in array).
`low = 0`, `high = 6`.

**Iteration 1:**
- `mid = 3`.
- `arr[3] = 20`, `arr[2] = 40`, `arr[4] = 50`. No match.
- Since `target (90) > arr[mid] (20)`, we set `low = mid + 2 = 5`.

**Iteration 2:**
- `low = 5`, `high = 6`.
- `mid = 5 + (6 - 5) / 2 = 5`.
- Check `mid` indices:
  - `arr[5] = 80` (not equal to 90)
  - `mid - 1 = 4` $\rightarrow$ outside `[low, high]` (not checked)
  - `mid + 1 = 6` $\rightarrow$ `arr[6] = 70` (not equal to 90)
- Since `target (90) > arr[mid] (80)`, we set `low = mid + 2 = 7`.

**Loop Ends:**
- `low` (7) > `high` (6). Loop terminates.
- Return `-1`.

### C++17 Code
```cpp
#include <vector>

class Solution {
public:
    int searchNearlySorted(const std::vector<int>& arr, int target) {
        int n = arr.size();
        int low = 0;
        int high = n - 1;

        while (low <= high) {
            int mid = low + (high - low) / 2;

            // 1. Check mid
            if (arr[mid] == target) {
                return mid;
            }

            // 2. Check mid - 1 (ensure within search space bounds)
            if (mid - 1 >= low && arr[mid - 1] == target) {
                return mid - 1;
            }

            // 3. Check mid + 1 (ensure within search space bounds)
            if (mid + 1 <= high && arr[mid + 1] == target) {
                return mid + 1;
            }

            // 4. Decide search direction.
            // Since we checked mid-1, mid, and mid+1, we can skip them
            if (target < arr[mid]) {
                high = mid - 2;
            } else {
                low = mid + 2;
            }
        }

        return -1;
    }
};
```

---

## 8. Code Walkthrough
- `int low = 0; int high = n - 1;`: Sets up boundaries for binary search.
- `int mid = low + (high - low) / 2;`: Standard overflow-safe midpoint calculation.
- `if (arr[mid] == target) return mid;`: If the middle element is the target, return its index.
- `if (mid - 1 >= low && arr[mid - 1] == target) return mid - 1;`: Checks the left neighbor of the midpoint. The condition `mid - 1 >= low` ensures we do not cross the boundary of the current search space or access a negative index.
- `if (mid + 1 <= high && arr[mid + 1] == target) return mid + 1;`: Checks the right neighbor of the midpoint. The condition `mid + 1 <= high` prevents out-of-bounds access.
- `if (target < arr[mid]) high = mid - 2;`: If the target is smaller than `arr[mid]`, it must lie in the left part. Since we already checked `mid - 1`, we jump directly to `mid - 2`.
- `else low = mid + 2;`: If the target is larger, it must lie in the right part. Since we already checked `mid + 1`, we jump directly to `mid + 2`.

---

## 9. Dry Run
Using `arr` = `[10, 3, 40, 20, 50, 80, 70]`, `target = 70`.

| Step | low | high | mid | Check `mid-1` | Check `mid` | Check `mid+1` | Action |
|---|---|---|---|---|---|---|---|
| 1 | 0 | 6 | 3 | `arr[2]=40` (No) | `arr[3]=20` (No) | `arr[4]=50` (No) | `target (70) > arr[3] (20)` $\rightarrow$ `low = 5` |
| 2 | 5 | 6 | 5 | `arr[4]` (out of bound) | `arr[5]=80` (No) | `arr[6]=70` (Yes!) | Match found at `mid + 1 = 6`. Return `6` |

---

## 10. Complexity Analysis
- **Time Complexity:**
  - In each step of the binary search, we perform a constant number of comparisons ($O(1)$) and reduce the search space size by half (from $N$ to $N/2 - 1$).
  - **Total Time Complexity:** $O(\log N)$ in the worst case, same as standard binary search.
- **Space Complexity:**
  - $O(1)$ auxiliary space.

---

## 11. Common Mistakes
1. **Out-of-Bounds Indexing:** Accessing `arr[mid - 1]` or `arr[mid + 1]` without verifying that they lie within `[low, high]` (or at least `[0, N-1]`). This can cause access violations or undefined behavior.
2. **Incorrect Step Size:** Shifting bounds to `high = mid - 1` or `low = mid + 1` instead of `mid - 2` or `mid + 2`. While shifting by 1 is functionally correct, it performs redundant checks because `mid - 1` and `mid + 1` have already been verified in the current iteration.
3. **Loop Termination Failure:** If `low` and `high` boundaries are updated incorrectly, it can result in infinite loops.

---

## 12. Edge Cases
| Edge Case | Example | Why it matters | Solution |
|---|---|---|---|
| Single element array | `arr=[10]`, `target=10` | Midpoint is 0. Neighbors `mid-1` and `mid+1` are out of bounds. | Handled by bounds checks. Target matched at `mid`. |
| Target is at first index | `arr=[10, 3, 40]`, `target=10` | Mid is 1. Target is at `mid - 1 = 0`. | Handled by `mid - 1` check. |
| Target is at last index | `arr=[10, 3, 40]`, `target=40` | Mid is 1. Target is at `mid + 1 = 2`. | Handled by `mid + 1` check. |
| Target not present | `arr=[10, 20, 30]`, `target=25` | Search should terminate and return -1. | `low` exceeds `high` and returns -1. |

---

## 13. Interview Explanation
"In a nearly sorted array, each element is at most 1 position away from its fully sorted position. This means the element that should reside at index `mid` can only be at `mid - 1`, `mid`, or `mid + 1`. 
We can modify the standard binary search to check all three positions: `mid - 1`, `mid`, and `mid + 1` at each step. 
If we find a match at any of these three, we return its index. 
If not, we can safely discard these three elements and decide which half to search. If the target is smaller than `arr[mid]`, we search the left half by setting `high = mid - 2`. If the target is larger, we search the right half by setting `low = mid + 2`. 
This allows us to search the array in $O(\log N)$ time and $O(1)$ space."

---

## 14. Follow-up Questions
1. **Q: What if the elements are at most $K$ positions away from their sorted position?**
   - **A:** If they are at most $K$ positions away, we would check all elements from index `mid - K` to `mid + K`. If no match is found, we shift the search range: `high = mid - (K + 1)` or `low = mid + (K + 1)`. The time complexity would be $O(K \log N)$. If $K$ is small, this is still very efficient.
2. **Q: Can we sort the nearly sorted array first?**
   - **A:** Sorting a nearly sorted array where elements are at most 1 position away can be done in $O(N)$ time (e.g. using insertion sort or a min-heap of size 3). However, sorting first takes $O(N)$ time, whereas our binary search takes $O(\log N)$ time, which is much faster.

---

## 15. Variations
1. **Sort a K-Sorted Array:**
   - Instead of searching, sort the array. This can be solved optimally using a Min-Heap of size $K + 1$ in $O(N \log K)$ time.
2. **Search in a Rotated Sorted Array:**
   - Another modified binary search problem where the search space is divided into sorted segments.

---

## 16. Revision Notes
- **Offset limit:** Max displacement is 1.
- **Checked elements:** `mid - 1`, `mid`, `mid + 1`.
- **Search skip:** Jump by 2: `mid - 2` or `mid + 2`.
- **Safety check:** Always clamp/validate indices with `low` and `high` bounds.

---

## 17. Memorization Version
- Set `low = 0`, `high = N - 1`.
- While `low <= high`:
  - `mid = low + (high - low) / 2`.
  - Check `mid`, `mid - 1` (if $\ge$ `low`), and `mid + 1` (if $\le$ `high`). If match, return index.
  - If `target < arr[mid]`, `high = mid - 2`.
  - Else `low = mid + 2`.
- Return `-1`.
