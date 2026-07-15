# Arrays — Q01: Find Second Largest Element

---

## 1. Problem Statement

**Input:** An integer array `arr` of size `n`.  
**Output:** The second largest distinct element in the array.  
**Constraints:**
- `n >= 2`
- Array may contain duplicates
- Elements can be negative, zero, or positive

**Important observations:**
- "Second largest" means the second distinct value when elements are sorted in descending order.
- If all elements are the same, there is no second largest — handle this as an edge case.

---

## 2. Interview Clarification

Before writing a single line of code, ask the interviewer:

1. **Duplicates?** — If `arr = [5, 5, 5, 3]`, is the second largest `3` or is `5` itself (just occurring again)?
   - Standard interpretation: second largest **distinct** value → answer is `3`.
2. **Can all elements be equal?** → Then there's no second largest; clarify expected return (e.g., `-1` or `INT_MIN`).
3. **Negative numbers?** → Array may have all negatives; ensure logic handles that.
4. **Size guarantee?** → Is `n >= 2` guaranteed?
5. **Can I sort the array?** → Sorting changes the original array — confirm if that's allowed.
6. **Expected time complexity?** → O(n log n)? O(n)?

---

## 3. Thinking Process

- First instinct: **Sort** the array in descending order and return the second unique element. Simple, but O(n log n).
- Can I do better? I only need **two values** — the largest and the second largest. I don't need full sorting.
- Observation: If I scan once and track the **maximum** so far, can I simultaneously track the **second maximum**?
  - Yes! Every time I find a new maximum, my old maximum becomes a candidate for second largest.
  - Every time I find something less than max but greater than second max, update second max.
- This gives me an **O(n) single-pass** solution.

---

## 4. Pattern Recognition

**Topic:** Arrays  
**Pattern:** Single-pass linear scan with two tracking variables.

This is a classic "track top-K elements" problem where K=2. The same mental model extends to Kth largest element problems (using a min-heap for larger K).

---

## 5. Brute Force Solution

### Why this comes first
The most natural idea: sort the array, then find the second distinct element from the end.

### Algorithm
1. Sort the array in ascending order.
2. Start from the second-to-last element and move left until you find an element different from the last (largest) element.
3. That element is the second largest.

### Dry Run
```
arr = [12, 35, 1, 10, 34, 1]
After sorting: [1, 1, 10, 12, 34, 35]
Largest = 35 (last element)
Scan leftward: 34 ≠ 35 → Second Largest = 34
```

### Time Complexity
- **O(n log n)** — dominated by sorting

### Space Complexity
- **O(1)** if in-place sort (like `std::sort`), **O(n)** if extra array is made

### Pros
- Simple to write and understand
- No tricky logic

### Cons
- Sorting is unnecessary — we don't need the full sorted order
- Modifies the array

### C++ Code (Brute Force)

```cpp
#include <bits/stdc++.h>
using namespace std;

int findSecondLargestBrute(vector<int>& arr) {
    int n = arr.size();
    
    // Sort array in ascending order
    sort(arr.begin(), arr.end());
    
    // The largest element is arr[n-1]
    // Scan leftward to find the first element different from the largest
    for (int i = n - 2; i >= 0; i--) {
        if (arr[i] != arr[n - 1]) {
            return arr[i];  // This is the second largest distinct element
        }
    }
    
    // If no second distinct element exists (all elements are equal)
    return -1;
}
```

---

## 6. Better Solution

*There is no meaningful intermediate solution here — the brute force (O(n log n)) jumps directly to optimal (O(n)). The key insight is that we can track two variables in a single pass.*

---

## 7. Optimal Solution

### Why this is optimal
We only need to make **one pass** through the array. We track two variables:
- `largest`: the maximum element seen so far
- `secondLargest`: the maximum element seen so far that is **strictly less than `largest`**

Every element is visited exactly once → **O(n) time, O(1) space** — this is the best possible for an unsorted array.

### Algorithm
1. Initialize `largest = INT_MIN`, `secondLargest = INT_MIN`.
2. For each element `x` in the array:
   - If `x > largest`: update `secondLargest = largest`, then `largest = x`
   - Else if `x > secondLargest` AND `x != largest`: update `secondLargest = x`
3. If `secondLargest == INT_MIN`, no second largest exists → return -1.
4. Return `secondLargest`.

### Complete Dry Run
```
arr = [12, 35, 1, 10, 34, 1]
Initialize: largest = INT_MIN, secondLargest = INT_MIN

Step 1: x = 12
  12 > INT_MIN (largest) → secondLargest = INT_MIN, largest = 12
  State: largest=12, secondLargest=INT_MIN

Step 2: x = 35
  35 > 12 (largest) → secondLargest = 12, largest = 35
  State: largest=35, secondLargest=12

Step 3: x = 1
  1 < 35, 1 < 12 → no update
  State: largest=35, secondLargest=12

Step 4: x = 10
  10 < 35, 10 < 12 → no update
  State: largest=35, secondLargest=12

Step 5: x = 34
  34 < 35, 34 > 12 → secondLargest = 34
  State: largest=35, secondLargest=34

Step 6: x = 1 (duplicate)
  1 < 35, 1 < 34 → no update

Final: secondLargest = 34
```

### Why this is the best approach
- **O(n) time** — single pass, cannot be faster for unsorted arrays
- **O(1) space** — only two variables
- Handles duplicates correctly
- Handles negative numbers correctly

### C++ Code (Optimal)

```cpp
#include <bits/stdc++.h>
using namespace std;

int findSecondLargest(vector<int>& arr) {
    int n = arr.size();
    
    // Initialize both tracking variables to the smallest possible integer
    int largest = INT_MIN;
    int secondLargest = INT_MIN;
    
    for (int i = 0; i < n; i++) {
        int currentElement = arr[i];
        
        if (currentElement > largest) {
            // Current element is the new maximum
            // The previous maximum becomes a candidate for second largest
            secondLargest = largest;
            largest = currentElement;
        } else if (currentElement > secondLargest && currentElement != largest) {
            // Current element is not the largest, but it's better than second largest
            // The condition currentElement != largest handles duplicates of the maximum
            secondLargest = currentElement;
        }
    }
    
    // If secondLargest was never updated, no second distinct element exists
    if (secondLargest == INT_MIN) {
        return -1;  // Signal: no second largest exists
    }
    
    return secondLargest;
}
```

---

## 8. Code Walkthrough

- `largest` and `secondLargest` start at `INT_MIN`. This ensures any element in the array will be larger on the first comparison.
- When we find a new largest element: `secondLargest = largest; largest = currentElement;` (demote the old maximum).
- The `else if` block handles cases where the element is between the two values. The check `currentElement != largest` is critical to ignore duplicate maximums.
- Finally, if `secondLargest` remains `INT_MIN`, it means no second distinct element was found (e.g., all elements are identical), so we return `-1`.

---

## 9. Dry Run

| Step | `currentElement` | `currentElement > largest?` | `currentElement > secondLargest?` | `largest` | `secondLargest` |
|------|-----------------|-----------------------------|------------------------------------|-----------|-----------------|
| Init | —               | —                           | —                                  | INT_MIN   | INT_MIN         |
| 1    | 12              | Yes → promote               | —                                  | 12        | INT_MIN         |
| 2    | 35              | Yes → promote               | —                                  | 35        | 12              |
| 3    | 1               | No                          | No (1 < 12)                        | 35        | 12              |
| 4    | 10              | No                          | No (10 < 12)                       | 35        | 12              |
| 5    | 34              | No                          | Yes (34 > 12, 34 ≠ 35)            | 35        | 34              |
| 6    | 1               | No                          | No                                 | 35        | 34              |

**Result:** `secondLargest = 34`

---

## 10. Complexity Analysis

- **Time Complexity:** **O(n)**. We visit each element of the array exactly once.
- **Space Complexity:** **O(1)**. We only use two integer variables.

---

## 11. Common Mistakes

- Not checking `currentElement != largest` which leads to duplicate maximum values being selected as second largest.
- Initializing variables with `0` instead of `INT_MIN`, which fails if the array contains only negative numbers.
- Returning `arr[n-2]` directly after sorting, without accounting for duplicates.

---

## 12. Edge Cases

| Edge Case | Example Input | Expected Output | Reason |
|-----------|---------------|-----------------|--------|
| All elements equal | `[5, 5, 5]` | `-1` | No second distinct element |
| Array of size 2 (equal) | `[4, 4]` | `-1` | No second distinct element |
| Array of size 2 (distinct) | `[4, 7]` | `4` | Basic minimum size |
| Negative values | `[-10, -5, -20]` | `-10` | Handles negative integers |

---

## 13. Interview Explanation

"To find the second largest distinct element in an array, I perform a single pass through the array. I maintain two variables: `largest` and `secondLargest`, both initialized to `INT_MIN`. For each element, if it's greater than `largest`, I update `secondLargest` to be `largest`, and update `largest` to the current element. If it's not greater than `largest` but is greater than `secondLargest` (and not equal to `largest`), I update `secondLargest` to the current element. This ensures that we correctly identify the second largest distinct value in O(n) time and O(1) auxiliary space."

---

## 14. Follow-up Questions

- **Can you find the K-th largest element?** Yes, by using a Min-Heap of size K which takes O(n log K) time, or using QuickSelect which takes O(n) average time.
- **What if the array is sorted?** If sorted, we can start from the end and find the first element different from `arr[n-1]`.

---

## 15. Variations

- **Find the second smallest element:** Track `smallest` and `secondSmallest`, updating them with opposite comparison logic.

---

## 16. Revision Notes

- **Trick:** Champion and runner-up analogy.
- **Condition:** `currentElement != largest` is crucial.
- **Complexity:** Time: O(n), Space: O(1).

---

## 17. Memorization Version

```cpp
int largest = INT_MIN, secondLargest = INT_MIN;
for (int x : arr) {
    if (x > largest) {
        secondLargest = largest;
        largest = x;
    } else if (x > secondLargest && x != largest) {
        secondLargest = x;
    }
}
return (secondLargest == INT_MIN) ? -1 : secondLargest;
```
