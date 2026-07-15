# Find Position of Element in Infinite Sorted Array

## 1. Problem Statement
Given an infinite sorted array (an array of sorted integers with no defined size or length), find the index of a target element. If the target element is not present in the array, return `-1`.
Since the array is infinite, you cannot access its size, and trying to access an element out of bounds will lead to an error or return a special sentinel value (e.g., `2^31 - 1` or `INT_MAX`). 
We are given an interface/object `ArrayReader` which has a method `int get(int index)` that returns the element at the specified index.

### Input
- `reader`: An instance of `ArrayReader` containing the sorted sequence.
- `target`: The integer value we need to find.

### Output
- Return the 0-based index of the `target` if found, otherwise return `-1`.

### Constraints
- $1 \le \text{target} \le 10^9$
- The array elements are sorted in non-decreasing order.
- The index of the target (if present) is within bounds of a standard 32-bit integer.

### Observations
1. In a typical array, we perform binary search within the bounds of `[0, size - 1]`.
2. Since this array is infinite, we do not know the upper bound.
3. However, since the array is sorted, the elements keep increasing. We can search for the target by first defining a reasonable bounding box `[low, high]` and then performing a standard binary search within that box.

---

## 2. Interview Clarification
Before coding, a candidate should clarify:
1. **Q:** What does `reader.get(index)` return if the index is out of bounds (beyond the actual elements of the array)?
   - **A:** It returns $2^{31} - 1$ (`INT_MAX`).
2. **Q:** Can the array contain duplicate elements?
   - **A:** Yes, it can contain duplicates. If duplicates exist, finding any instance of the target or the first occurrence should be clarified. (We assume finding any index of the target is sufficient).
3. **Q:** Can the target be negative?
   - **A:** Yes, the elements can be negative, but since it is sorted and infinite, they will eventually grow positive.

---

## 3. Thinking Process
### First Instinct
Since the size is unknown, we could do a linear search starting from index 0:
- Check `reader.get(0)`, `reader.get(1)`, `reader.get(2)`, ...
- Stop when `reader.get(i) >= target` or `reader.get(i) == INT_MAX`.
- If `reader.get(i) == target`, return `i`. Else return `-1`.
- While this works, it takes $O(N)$ time, where $N$ is the actual index of the target. This is too slow if the target is located at a very large index (e.g., $10^6$).

### Can we do better?
Since the array is sorted, we want to use binary search. To use binary search, we need a left boundary `low` and a right boundary `high`.
- We can initialize our window size to 2: `low = 0`, `high = 1`.
- If the target is greater than the element at `high` (`reader.get(high) < target`), it means the target lies beyond our current window.
- We can exponentially increase our search space. We double the window size:
  - The new `low` becomes the previous `high + 1`.
  - The new `high` becomes `previous_high + (current_window_size * 2)`.
  - In practice, we can just do: `low = high + 1` and `high = high * 2`.
- We repeat this doubling process until we find a `high` index where `reader.get(high) >= target`.
- Once we find such a `high`, the target is guaranteed to lie within the range `[low, high]`.
- We then perform a standard binary search in the range `[low, high]`.

### Why is this optimal?
- Doubling the index exponentially narrows down the range. We will find the correct range in $O(\log N)$ steps, where $N$ is the target's index.
- Once the range is found, the size of the range `[low, high]` is at most $N$. Performing binary search within this range takes $O(\log N)$ steps.
- Total time complexity is $O(\log N) + O(\log N) = O(\log N)$.

---

## 4. Pattern Recognition
This problem belongs to the **Exponential Search / Unbounded Binary Search** pattern.
- Why? We are searching in an infinite/unbounded sorted list.
- The standard strategy is to first find the boundaries of the search space using exponential steps (multiplying the index by 2 at each step), and then apply classic binary search within those boundaries.

---

## 5. Brute Force Solution
### Algorithm
1. Start at index `i = 0`.
2. Loop indefinitely:
   - Get value at index `i`.
   - If value equals `target`, return `i`.
   - If value exceeds `target` or equals `INT_MAX`, return `-1`.
   - Increment `i` by 1.

### Complexity Analysis
- **Time Complexity:** $O(N)$ where $N$ is the index of the target.
- **Space Complexity:** $O(1)$ auxiliary space.

### Pros and Cons
- **Pros:** Easy to write, no overflow risks with index doubling.
- **Cons:** Extremely slow if the target is at index $10^6$ or larger.

### C++17 Mock Interface and Code
```cpp
#include <climits>

// Mock ArrayReader class for compilation purposes
class ArrayReader {
public:
    int get(int index) const {
        // Returns element at index, or INT_MAX if out of bounds
        // Actual implementation provided by the system
        return index; 
    }
};

class Solution {
public:
    int searchBruteForce(const ArrayReader& reader, int target) {
        int i = 0;
        while (true) {
            int val = reader.get(i);
            if (val == target) {
                return i;
            }
            if (val > target || val == INT_MAX) {
                break;
            }
            i++;
        }
        return -1;
    }
};
```

---

## 6. Better Solution
*Note: For this specific problem, there is no intermediate "Better" solution that sits between the brute force (linear search) and the optimal Exponential Search. We jump directly from Brute Force to the Optimal Solution.*

---

## 7. Optimal Solution
### Algorithm
1. Initialize the boundaries: `low = 0` and `high = 1`.
2. Expand the boundaries while `reader.get(high) < target`:
   - Store `low` in a temporary variable if needed, or simply update:
   - `low = high + 1`.
   - `high = high * 2` (exponential expansion).
3. Once the loop terminates, the target must be within `[low, high]`.
4. Perform binary search in the range `[low, high]`:
   - While `low <= high`:
     - Calculate `mid = low + (high - low) / 2`.
     - Get value `val = reader.get(mid)`.
     - If `val == target`, return `mid`.
     - If `val > target`, search in the left half: `high = mid - 1`.
     - If `val < target`, search in the right half: `low = mid + 1`.
5. If the loop ends and we haven't found the target, return `-1`.

### Dry Run
Suppose the infinite array is `[3, 5, 7, 9, 10, 90, 100, 130, 140, 160, 170, ...]`, and `target = 10`.

**Step 1: Finding bounds**
- Start: `low = 0`, `high = 1`.
  - `reader.get(high)` = `reader.get(1)` = 5.
  - Since $5 < 10$, we expand bounds.
  - `low = 2`, `high = 1 * 2 = 2`.
- Check:
  - `reader.get(high)` = `reader.get(2)` = 7.
  - Since $7 < 10$, we expand bounds.
  - `low = 3`, `high = 2 * 2 = 4`.
- Check:
  - `reader.get(high)` = `reader.get(4)` = 10.
  - Since $10 \not< 10$, loop terminates.
- Final boundaries: `low = 3`, `high = 4`.

**Step 2: Binary Search in range `[3, 4]`**
- `low = 3`, `high = 4`.
- `mid = 3 + (4 - 3) / 2 = 3`.
- `reader.get(3)` = 9.
- Since $9 < 10$, we set `low = mid + 1 = 4`.
- Next iteration: `low = 4`, `high = 4`.
- `mid = 4 + (4 - 4) / 2 = 4`.
- `reader.get(4)` = 10.
- Since $10 == 10$, we return index `4`.

### C++17 Code
```cpp
#include <climits>

// Mock ArrayReader class for compilation purposes
class ArrayReader {
public:
    int get(int index) const {
        return index; // Placeholder
    }
};

class Solution {
public:
    int search(const ArrayReader& reader, int target) {
        // Step 1: Find the bounds where target can exist
        int low = 0;
        int high = 1;

        // Exponentially increase high bound until we overshoot target
        while (reader.get(high) < target) {
            low = high + 1;
            // Prevent integer overflow when doubling high
            if (high > INT_MAX / 2) {
                high = INT_MAX;
                break;
            }
            high = high * 2;
        }

        // Step 2: Perform standard binary search in [low, high]
        while (low <= high) {
            int mid = low + (high - low) / 2;
            int midValue = reader.get(mid);

            if (midValue == target) {
                return mid;
            } else if (midValue > target) {
                // If midValue is greater than target (or is INT_MAX),
                // search in the left half
                high = mid - 1;
            } else {
                // Search in the right half
                low = mid + 1;
            }
        }

        return -1;
    }
};
```

---

## 8. Code Walkthrough
- `int low = 0; int high = 1;`: Sets up initial sliding window.
- `while (reader.get(high) < target)`: Keeps expanding the search space exponentially. If the value at `high` is less than `target`, the target must reside at a larger index.
- `low = high + 1;`: Shifts `low` up to optimize the search space.
- `if (high > INT_MAX / 2)`: Crucial check to avoid signed integer overflow when performing `high * 2`. If `high` is close to the limits, we set `high = INT_MAX` and stop.
- `high = high * 2;`: Doubles the upper bound index.
- `while (low <= high)`: Executes standard binary search within the bounded search space.
- `int midValue = reader.get(mid);`: Reads the candidate value. If it matches target, return `mid`.
- `high = mid - 1` and `low = mid + 1`: Standard binary search narrowing.

---

## 9. Dry Run
Target = 90.
Array = `[3, 5, 7, 9, 10, 90, 100, 130, 140, 160]`.

| Step | low | high | Value at high | Action |
|---|---|---|---|---|
| Init | 0 | 1 | 5 | $5 < 90$ $\rightarrow$ Double |
| 1 | 2 | 2 | 7 | $7 < 90$ $\rightarrow$ Double |
| 2 | 3 | 4 | 10 | $10 < 90$ $\rightarrow$ Double |
| 3 | 5 | 8 | 140 | $140 \ge 90$ $\rightarrow$ Stop range search |

Now, Binary Search in `[5, 8]`:
- `low = 5`, `high = 8`.
- `mid = 5 + (8 - 5) / 2 = 6`. Value at index 6 = 100.
- Since $100 > 90$, `high = mid - 1 = 5`.
- Next iteration: `low = 5`, `high = 5`.
- `mid = 5`. Value at index 5 = 90.
- Match! Return `5`.

---

## 10. Complexity Analysis
- **Time Complexity:**
  - **Finding Range:** The index of the high pointer is doubled at each step: $1, 2, 4, 8, \dots, 2^k$. If the target is at index $P$, the loop runs $k$ times where $2^k \ge P$, which means $k = O(\log P)$.
  - **Binary Search:** The binary search is performed on the range `[low, high]` where the length of the range is at most $2^{k-1}$. The binary search takes $O(\log(2^{k-1})) = O(k) = O(\log P)$ steps.
  - **Total Time Complexity:** $O(\log P)$ where $P$ is the position of the target in the infinite array.
- **Space Complexity:**
  - $O(1)$ auxiliary space.

---

## 11. Common Mistakes
1. **Integer Overflow:** Doubling `high` without safety checks can exceed `INT_MAX`, resulting in negative values and an infinite loop or segmentation fault.
2. **Slow Bound Expansion:** Incrementing `high` linearly (`high++`) instead of exponentially (`high = high * 2`). This defeats the purpose of binary search and results in $O(N)$ time complexity.
3. **Wrong Range Initialization:** Initializing `low = 0` and not shifting `low` inside the expansion loop. While it is correct to keep `low = 0`, updating `low = previous_high + 1` reduces the binary search space size significantly.

---

## 12. Edge Cases
| Edge Case | Example | Why it matters | Solution |
|---|---|---|---|
| Target is at index 0 | `target = 3`, first element is 3 | Loop condition `reader.get(high) < target` fails immediately. | Returns index 0 correctly. |
| Target is not in array | `target = 6` in `[3, 5, 7, ...]` | Binary search should terminate and return -1. | `low` and `high` overlap, standard binary search ends, returns -1. |
| Target is very large | `target` is at index $10^9$ | Risk of overflowing the `high` index. | Check `high > INT_MAX / 2` before doubling. |
| Target is smaller than first element | `target = 1` in `[3, 5, 7, ...]` | Loop terminates at start, binary search returns -1. | Handled correctly. |

---

## 13. Interview Explanation
"To find the position of an element in an infinite sorted array, we cannot use the standard binary search directly because the size of the array is unknown. 
Instead, we can use Exponential Search. We start with a small window `[0, 1]`. If the target is larger than the element at the `high` index, we shift `low` to `high + 1` and double the `high` index. We repeat this doubling until we find an index `high` where the element is greater than or equal to the target. 
Once we find this boundary, the target is guaranteed to be within the range `[low, high]`. We can then perform a standard binary search in this range. 
Since we double the index at each step, we find the range in $O(\log P)$ steps, and the subsequent binary search also takes $O(\log P)$ steps, where $P$ is the position of the target. This gives an optimal time complexity of $O(\log P)$ and uses $O(1)$ space."

---

## 14. Follow-up Questions
1. **Q: What if the API reader is slow or has latency?**
   - **A:** If reading is slow, we want to minimize the number of queries. Our method does at most $2 \log P$ queries. We could tweak the expansion factor (e.g. triple or quadruple the range) but doubling is mathematically balanced.
2. **Q: Can we use this approach to find the boundary of 0s and 1s in an infinite array of 0s followed by 1s?**
   - **A:** Yes. This is exactly the same pattern. We find the index where the value becomes 1 using exponential search, and then binary search for the transition index.

---

## 15. Variations
1. **Search in a Sorted Array of Unknown Size (LeetCode 702):**
   - Exact same problem setup, matching this template.
2. **Find the first transition point in an infinite binary array:**
   - Find the first index containing `1` where the array format is `[0, 0, 0, ..., 0, 1, 1, 1, ...]`.

---

## 16. Revision Notes
- **Strategy:** Find boundaries first, then binary search.
- **Bound expansion:** `low = high + 1`, `high = high * 2`.
- **Time Complexity:** $O(\log P)$ where $P$ is target's actual position.
- **Overflow check:** Always make sure `high` doesn't overflow `INT_MAX`.

---

## 17. Memorization Version
- Start `low = 0`, `high = 1`.
- While `reader.get(high) < target`:
  - `low = high + 1`
  - `high = min(high * 2, INT_MAX)` (exit if already `INT_MAX`).
- Binary search in `[low, high]`.
- Return index if found, else `-1`.
