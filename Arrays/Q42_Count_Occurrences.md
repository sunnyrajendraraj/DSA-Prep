# Count Occurrences of Element in Sorted Array

## 1. Problem Statement

Given a sorted array of integers `arr` of size `n` and a target value `x`, count the number of occurrences of `x` in `arr`.
If `x` is not present in the array, return `0`.

### Input
- A sorted array of integers `arr` of size `n`.
- An integer `target`.

### Output
- An integer representing the count of occurrences of `target` in `arr`.

### Constraints
- $1 \le n \le 10^5$
- $-10^9 \le arr[i], target \le 10^9$
- `arr` is sorted in non-decreasing order.

### Observations
1. The array is already sorted.
2. If `target` is present, all occurrences of `target` will be contiguous in the array.
3. The count of occurrences is given by the difference between the index of the last occurrence and the index of the first occurrence, plus 1: `(lastIndex - firstIndex + 1)`.

---

## 2. Interview Clarification

Before writing code, clarify these points with the interviewer:
1. **Should the solution utilize the sorted nature of the array?**
   - *Yes, we should aim for a solution faster than the linear $O(n)$ scan.*
2. **What if the target is not present in the array?**
   - *Return `0`.*
3. **What if the array is empty?**
   - *The constraints say $n \ge 1$, but if it can be empty, we should return `0`. Always write code to handle $n = 0$ safely.*

---

## 3. Thinking Process

Let's build the intuition step-by-step:
- **First Instinct (Brute Force):**
  We can traverse the entire array and keep a running counter. Whenever `arr[i] == target`, we increment the count.
  - This takes $O(n)$ time and $O(1)$ space. It is extremely simple but does not leverage the fact that the array is sorted.

- **Improving (Better Solution):**
  Since the array is sorted, we can search for the target using a standard binary search.
  - If we don't find it, return `0`.
  - If we find it at index `mid`, we can expand outwards:
    - Initialize `count = 1`.
    - Iterate leftwards from `mid - 1` while elements equal the target, incrementing the count.
    - Iterate rightwards from `mid + 1` while elements equal the target, incrementing the count.
  - *Bottleneck:* If the array consists entirely of the target (e.g. `[3, 3, 3, 3, 3]`, target = 3), the expansion scan will look at all elements, taking $O(n)$ time.

- **Optimal Insight (Two Binary Searches):**
  We can compute the exact boundaries of the target range without doing any linear scanning.
  We know that:
  $\text{Total Count} = \text{lastPosition} - \text{firstPosition} + 1$
  
  We can find `firstPosition` and `lastPosition` using the optimal binary search functions from the previous problem (Q41):
  - Find the first occurrence of the target (`firstPosition`).
  - If `firstPosition == -1` (target not found), we immediately return `0` occurrences.
  - Else, find the last occurrence of the target (`lastPosition`).
  - Return `lastPosition - firstPosition + 1`.
  
  Since both searches take $O(\log n)$ time, the total complexity is $O(\log n)$ in all cases, which is optimal.

---

## 4. Pattern Recognition

This problem fits the **Sorted Array Search / Boundary Range** pattern.
- **Why this topic?** The input is sorted, and we need to count contiguous identical elements.
- **Pattern:**
  - Finding a range boundary using binary search is a standard way to solve range counting problems in logarithmic time.

---

## 5. Brute Force Solution

### Algorithm
1. Initialize `count = 0`.
2. For each element in the array:
   - If the element is equal to the target, increment `count`.
3. Return `count`.

### Complexity Analysis
- **Time Complexity:** $O(n)$ as we visit all elements in the array.
- **Space Complexity:** $O(1)$ auxiliary space.

### Pros and Cons
- **Pros:** Simple, requires no sorted guarantees to work.
- **Cons:** Inefficient; ignores the sorted property of the array.

### C++17 Code
```cpp
#include <vector>

class SolutionBrute {
public:
    int countOccurrences(const std::vector<int>& arr, int target) {
        int count = 0;
        for (int num : arr) {
            if (num == target) {
                count++;
            }
        }
        return count;
    }
};
```

---

## 6. Better Solution

### Algorithm
1. Use Binary Search to find *any* index `mid` where `arr[mid] == target`.
2. If target is not found, return `0`.
3. Initialize `count = 1`.
4. Scan left from `mid - 1` to `0`:
   - If `arr[i] == target`, increment `count`.
   - Else, break early.
5. Scan right from `mid + 1` to `n - 1`:
   - If `arr[i] == target`, increment `count`.
   - Else, break early.
6. Return `count`.

### Complexity Analysis
- **Time Complexity:** 
  - **Average Case:** $O(\log n)$ if the number of duplicates is small.
  - **Worst Case:** $O(n)$ if all elements in the array are equal to the target.
- **Space Complexity:** $O(1)$ auxiliary space.

### Pros and Cons
- **Pros:** Faster than brute force on average; can break early.
- **Cons:** Worst-case complexity is still linear.

### C++17 Code
```cpp
#include <vector>

class SolutionBetter {
public:
    int countOccurrences(const std::vector<int>& arr, int target) {
        int n = arr.size();
        if (n == 0) return 0;
        
        int low = 0;
        int high = n - 1;
        int foundIndex = -1;
        
        while (low <= high) {
            int mid = low + (high - low) / 2;
            if (arr[mid] == target) {
                foundIndex = mid;
                break;
            } else if (arr[mid] < target) {
                low = mid + 1;
            } else {
                high = mid - 1;
            }
        }
        
        if (foundIndex == -1) {
            return 0; // Target not found
        }
        
        int count = 1;
        
        // Scan left
        int left = foundIndex - 1;
        while (left >= 0 && arr[left] == target) {
            count++;
            left--;
        }
        
        // Scan right
        int right = foundIndex + 1;
        while (right < n && arr[right] == target) {
            count++;
            right++;
        }
        
        return count;
    }
};
```

---

## 7. Optimal Solution

### Algorithm
We reuse the two binary search concept to find the exact boundaries:
1. Define a helper function `findBound(arr, target, isFirst)`:
   - Perform binary search. If target matches, save the index, and redirect the search space left (`high = mid - 1`) if `isFirst` is true, or right (`low = mid + 1`) if `isFirst` is false.
2. Find the first occurrence index: `first = findBound(arr, target, true)`.
3. If `first == -1`, the element doesn't exist, so return `0`.
4. Find the last occurrence index: `last = findBound(arr, target, false)`.
5. Return `last - first + 1`.

### C++17 Code
```cpp
#include <vector>

class SolutionOptimal {
private:
    // Helper function to find the first or last occurrence index of the target
    int findBound(const std::vector<int>& arr, int target, bool isFirst) {
        int n = arr.size();
        int low = 0;
        int high = n - 1;
        int resultIndex = -1;
        
        while (low <= high) {
            int mid = low + (high - low) / 2;
            
            if (arr[mid] == target) {
                resultIndex = mid; // Record the index
                
                if (isFirst) {
                    high = mid - 1; // Keep searching in the left half
                } else {
                    low = mid + 1;  // Keep searching in the right half
                }
            } else if (arr[mid] < target) {
                low = mid + 1;
            } else {
                high = mid - 1;
            }
        }
        
        return resultIndex;
    }

public:
    int countOccurrences(const std::vector<int>& arr, int target) {
        int firstIndex = findBound(arr, target, true);
        
        // If the target is not present, count is 0
        if (firstIndex == -1) {
            return 0;
        }
        
        int lastIndex = findBound(arr, target, false);
        
        // Return the count based on indices
        return lastIndex - firstIndex + 1;
    }
};
```

---

## 8. Code Walkthrough

Let's break down the optimal execution:
1. `int firstIndex = findBound(arr, target, true);`
   - Finds the very first occurrence of `target` using binary search.
2. `if (firstIndex == -1) return 0;`
   - Optimizes by skipping the second binary search if the target is not even in the array.
3. `int lastIndex = findBound(arr, target, false);`
   - Finds the very last occurrence of `target`.
4. `return lastIndex - firstIndex + 1;`
   - Calculates the length of the window containing the target. For example, if target is at indices `3, 4, 5`, then `firstIndex = 3` and `lastIndex = 5`. The count is $5 - 3 + 1 = 3$.

---

## 9. Dry Run

Let `arr = [1, 1, 2, 2, 2, 2, 3]`, `target = 2`.
- **First Search: `findBound(arr, 2, true)` (Leftmost occurrence)**

| Iteration | `low` | `high` | `mid` | `arr[mid]` | Action | `resultIndex` |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **1** | 0 | 6 | 3 | 2 | $2 == 2 \implies$ set `resultIndex = 3`, `high = 2` | 3 |
| **2** | 0 | 2 | 1 | 1 | $1 < 2 \implies$ set `low = 2` | 3 |
| **3** | 2 | 2 | 2 | 2 | $2 == 2 \implies$ set `resultIndex = 2`, `high = 1` | **2** |
| **Loop Ends** | 2 | 1 | — | — | `low > high` (2 > 1), terminate | 2 |

`firstIndex = 2`.

- **Second Search: `findBound(arr, 2, false)` (Rightmost occurrence)**

| Iteration | `low` | `high` | `mid` | `arr[mid]` | Action | `resultIndex` |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **1** | 0 | 6 | 3 | 2 | $2 == 2 \implies$ set `resultIndex = 3`, `low = 4` | 3 |
| **2** | 4 | 6 | 5 | 2 | $2 == 2 \implies$ set `resultIndex = 5`, `low = 6` | 5 |
| **3** | 6 | 6 | 6 | 3 | $3 > 2 \implies$ set `high = 5` | 5 |
| **Loop Ends** | 6 | 5 | — | — | `low > high` (6 > 5), terminate | **5** |

`lastIndex = 5`.

- **Calculation:**
  $\text{Count} = lastIndex - firstIndex + 1 = 5 - 2 + 1 = 4$.

**Final Output:** `4`.

---

## 10. Complexity Analysis

| Approach | Time Complexity | Space Complexity | Notes |
| :--- | :--- | :--- | :--- |
| **Brute Force** | $O(n)$ | $O(1)$ | Full scan |
| **Better (Expand)** | $O(\log n)$ average / $O(n)$ worst | $O(1)$ | Degrades when all elements are duplicates |
| **Optimal** | $O(\log n)$ | $O(1)$ | Two independent binary searches |

---

## 11. Common Mistakes

1. **Forgetting to check `firstIndex == -1`**:
   - If target is not present, `firstIndex` is `-1`. If we blindly call `findBound` for `lastIndex` as well, it will also return `-1`. The output calculation would be $-1 - (-1) + 1 = 1$, which is wrong (should be 0). Always return 0 early if `firstIndex == -1`.
2. **Infinite Binary Search Loops**:
   - Setting boundaries to `mid` instead of `mid + 1` or `mid - 1`. Always shrink the search space correctly.
3. **Array Index Out-of-Bounds in Better Solution**:
   - Forgetting to check boundaries (`left >= 0` and `right < n`) when expanding from `mid`.

---

## 12. Edge Cases

| Edge Case | Input | Expected Output | Why It Matters |
| :--- | :--- | :--- | :--- |
| Target not present | `arr = [1, 2, 3], target = 4` | `0` | Returns 0 early |
| Single element matching | `arr = [2], target = 2` | `1` | Minimal sized array |
| Single element mismatch | `arr = [2], target = 1` | `0` | Returns 0 early |
| All elements match | `arr = [3, 3, 3, 3], target = 3` | `4` | Tests maximum occurrence |

---

## 13. Interview Explanation

*"To count occurrences of a target element in a sorted array, the most optimal way is to use binary search.
Since the array is sorted, all occurrences of the target will be contiguous. This means we can find the range of these occurrences.
We can run two binary searches: one to find the first occurrence and another to find the last occurrence of the target.
If the first binary search returns -1, it means the target does not exist, so we return 0.
Otherwise, we locate both the first and last indices, and the count of occurrences is simply `(lastIndex - firstIndex + 1)`.
This gives a guaranteed time complexity of $O(\log n)$ with $O(1)$ extra space, which is far better than a linear scan that takes $O(n)$."*

---

## 14. Follow-up Questions

1. **Can we implement this using C++ STL `std::equal_range`?**
   - *Yes. `std::equal_range` returns a pair of iterators representing the range of elements that are equal to the target. The count is `std::distance(range.first, range.second)`.*
2. **What if the target appears in a non-sorted array?**
   - *Binary search is not applicable. We would have to use a hash map to count frequencies in $O(n)$ time and space, or a linear search in $O(n)$ time and $O(1)$ space.*

---

## 15. Variations

1. **Count occurrences in a sorted 2D matrix**: Find occurrences of target in a 2D sorted grid (row-wise and column-wise).
2. **Find majority element**: Check if a target element appears more than $N/2$ times in a sorted array (can be solved by checking if `arr[firstIndex + N/2] == target`).

---

## 16. Revision Notes

- **Counting Formula**: `lastIndex - firstIndex + 1`.
- **Early Exit**: If `firstIndex == -1`, count is immediately 0.
- **Time Complexity**: $O(\log n)$ worst case.

---

## 17. Memorization Version

```cpp
// Optimal Occurrence Counting in Sorted Array
int findBound(const vector<int>& arr, int target, bool isFirst) {
    int low = 0, high = arr.size() - 1, ans = -1;
    while (low <= high) {
        int mid = low + (high - low) / 2;
        if (arr[mid] == target) {
            ans = mid;
            if (isFirst) high = mid - 1;
            else low = mid + 1;
        } else if (arr[mid] < target) low = mid + 1;
        else high = mid - 1;
    }
    return ans;
}
int countOccurrences(vector<int>& arr, int target) {
    int first = findBound(arr, target, true);
    if (first == -1) return 0;
    int last = findBound(arr, target, false);
    return last - first + 1;
}
```
*Time: $O(\log n)$, Space: $O(1)$*
