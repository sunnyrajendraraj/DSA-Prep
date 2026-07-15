# Find Duplicates in an Array

---

## 1. Problem Statement

Given an integer array `nums` of size $N$ where all the integers in the array are in the range $[1, N]$, and each integer appears at most twice, find and return an array of all the integers that appear twice.

*   **Input:** An integer array `nums` of size $N$.
*   **Output:** A list containing all numbers that appear more than once (exactly twice).
*   **Constraints:**
    *   $N \ge 1$
    *   $1 \le nums[i] \le N$
    *   Each integer appears once or twice.
*   **Observations:**
    *   Since elements are bounded by the size of the array ($1 \le nums[i] \le N$), we can map values directly to array indices.
    *   We want to achieve $O(N)$ time complexity and $O(1)$ auxiliary space.

---

## 2. Interview Clarification

Before coding, clarify the following with the interviewer:
1.  **Can we modify the input array?**
    *   *Answer:* Yes, but we should restore it to its original state if requested. The optimal solution uses sign negation which modifies the array in-place.
2.  **In what order should the duplicates be returned?**
    *   *Answer:* Any order is fine.
3.  **What if there are no duplicates?**
    *   *Answer:* Return an empty array.
4.  **Can numbers appear more than twice?**
    *   *Answer:* The problem specifies they appear at most twice. (If they can appear more than twice, index negation still works, but we must avoid adding the duplicate multiple times by only adding it to the result when we transition the state for the first time).

---

## 3. Thinking Process

*   **First Instinct (Brute Force):**
    *   Compare every element with all other elements.
    *   Use two loops. The outer loop selects an element at index `i`, and the inner loop checks if the same element exists at any index `j > i`.
    *   This takes $O(N^2)$ time and $O(1)$ space.
*   **Better Solution (Hashing / Set):**
    *   We can use a hash set or a boolean array of size $N+1$ to keep track of visited numbers.
    *   We iterate through the array. If we see a number already in our hash set, we add it to our duplicate list. Otherwise, we add it to the hash set.
    *   This takes $O(N)$ time and $O(N)$ auxiliary space.
*   **Optimal Insight (Array Index Negation):**
    *   Since the numbers in the array are in the range $[1, N]$, their values map naturally to array indices $0$ to $N - 1$.
    *   We can use the array itself as a hash table!
    *   For each element `num = abs(nums[i])`:
        *   The index it maps to is `idx = num - 1`.
        *   If `nums[idx]` is negative, it means we have visited this index before. Therefore, `num` is a duplicate.
        *   If `nums[idx]` is positive, we make it negative to mark that we have visited `num`.
    *   This approach is $O(N)$ in time and $O(1)$ in auxiliary space.

---

## 4. Pattern Recognition

*   **Topic:** Arrays.
*   **Pattern:** **Array as Hash Map (Index Negation / Cyclic Sort / Sign Marking)**.
*   *Why this pattern?* When elements of an array of size $N$ are constrained in the range $[0, N]$ or $[1, N]$, the indices themselves represent a zero-cost hash map. We can encode state information (like "visited") by changing the sign of the numbers at specific indices.

---

## 5. Brute Force Solution

### Algorithm
1.  Initialize an empty list `duplicates`.
2.  Iterate `i` from `0` to `N - 1`:
    *   Iterate `j` from `i + 1` to `N - 1`:
        *   If `nums[i] == nums[j]`, check if it's already in the duplicates list. If not, add `nums[i]` to `duplicates`.
3.  Return `duplicates`.

### Complexity
*   **Time Complexity:** $O(N^2)$ due to nested loops.
*   **Space Complexity:** $O(1)$ auxiliary space.

### Code (C++17)
```cpp
#include <vector>
#include <algorithm>

class SolutionBrute {
public:
    std::vector<int> findDuplicates(const std::vector<int>& nums) {
        std::vector<int> duplicates;
        int n = nums.size();
        
        for (int i = 0; i < n; ++i) {
            for (int j = i + 1; j < n; ++j) {
                if (nums[i] == nums[j]) {
                    // Check if duplicate already exists in results
                    if (std::find(duplicates.begin(), duplicates.end(), nums[i]) == duplicates.end()) {
                        duplicates.push_back(nums[i]);
                    }
                }
            }
        }
        return duplicates;
    }
};
```

---

## 6. Better Solution (Hashing)

### Algorithm
1.  Initialize an empty list `duplicates` and a hash set (or a frequency/visited array of size $N+1$).
2.  For each number `num` in `nums`:
    *   If `visited[num]` is true:
        *   Add `num` to `duplicates`.
    *   Else:
        *   Set `visited[num] = true`.
3.  Return `duplicates`.

### Complexity
*   **Time Complexity:** $O(N)$ — Single pass to scan the array.
*   **Space Complexity:** $O(N)$ — Auxiliary space for the hash set / visited array.

### Code (C++17)
```cpp
#include <vector>

class SolutionBetter {
public:
    std::vector<int> findDuplicates(const std::vector<int>& nums) {
        std::vector<int> duplicates;
        int n = nums.size();
        std::vector<bool> visited(n + 1, false);
        
        for (int num : nums) {
            if (visited[num]) {
                duplicates.push_back(num);
            } else {
                visited[num] = true;
            }
        }
        return duplicates;
    }
};
```

---

## 7. Optimal Solution (Index Negation Technique)

### Algorithm
1.  Initialize an empty list `duplicates`.
2.  Iterate through the array using index `i` from `0` to `N - 1`:
    *   Find the target index: `val = abs(nums[i])` and `idx = val - 1`.
    *   Check the sign of `nums[idx]`:
        *   If `nums[idx] < 0`, it means we have seen `val` before. Add `val` to `duplicates`.
        *   If `nums[idx] > 0`, negate it: `nums[idx] = -nums[idx]`.
3.  Optionally, restore the original array by taking the absolute value of all elements.
4.  Return `duplicates`.

### Dry Run
Input: `nums = [4, 3, 2, 7, 8, 2, 3, 1]`
*   `i = 0`: `val = 4`, `idx = 3`. `nums[3] = 7` (positive). Negate it: `nums[3] = -7`.
    Array: `[4, 3, 2, -7, 8, 2, 3, 1]`
*   `i = 1`: `val = 3`, `idx = 2`. `nums[2] = 2` (positive). Negate it: `nums[2] = -2`.
    Array: `[4, 3, -2, -7, 8, 2, 3, 1]`
*   `i = 2`: `val = 2`, `idx = 1`. `nums[1] = 3` (positive). Negate it: `nums[1] = -3`.
    Array: `[4, -3, -2, -7, 8, 2, 3, 1]`
*   `i = 3`: `val = 7`, `idx = 6`. `nums[6] = 3` (positive). Negate it: `nums[6] = -3`.
    Array: `[4, -3, -2, -7, 8, 2, -3, 1]`
*   `i = 4`: `val = 8`, `idx = 7`. `nums[7] = 1` (positive). Negate it: `nums[7] = -1`.
    Array: `[4, -3, -2, -7, -8, 2, -3, -1]` (Wait, idx for 8 is 7. `nums[4]` is 8, so `nums[7]` negated to `-1`).
*   `i = 5`: `val = 2`, `idx = 1`. `nums[1] = -3` (negative!). Duplicate found: `2`.
    Array: `[4, -3, -2, -7, -8, 2, -3, -1]`
*   `i = 6`: `val = 3` (obtained from `abs(-3)`), `idx = 2`. `nums[2] = -2` (negative!). Duplicate found: `3`.
    Array: `[4, -3, -2, -7, -8, 2, -3, -1]`
*   `i = 7`: `val = 1`, `idx = 0`. `nums[0] = 4` (positive). Negate it: `nums[0] = -4`.
*   Duplicates returned: `[2, 3]`.

### Why Optimal
*   We use the input array to store visited flags, eliminating auxiliary space requirements.
*   Time complexity is linear $O(N)$ with a single pass.
*   We can easily restore the input array back to its original values if needed.

---

### Beautiful C++17 Code

```cpp
#include <vector>
#include <cmath>

class SolutionOptimal {
public:
    std::vector<int> findDuplicates(std::vector<int>& nums) {
        std::vector<int> duplicates;
        
        for (int i = 0; i < nums.size(); ++i) {
            // Use absolute value to get original element value
            int val = std::abs(nums[i]);
            int idx = val - 1;
            
            // If the element at index is already negative, we've seen 'val' before
            if (nums[idx] < 0) {
                duplicates.push_back(val);
            } else {
                nums[idx] = -nums[idx]; // Mark as visited
            }
        }
        
        // Optional: Restore original values in the array
        for (int& num : nums) {
            num = std::abs(num);
        }
        
        return duplicates;
    }
};
```

---

## 8. Code Walkthrough

1.  `std::vector<int> duplicates;`
    Container to store all identified duplicate elements.
2.  `for (int i = 0; i < nums.size(); ++i) { ... }`
    Loop to inspect each element of the array.
3.  `int val = std::abs(nums[i]);`
    Get the original value of the current element. Since elements could have been negated during previous steps, we take the absolute value.
4.  `int idx = val - 1;`
    Convert the 1-based number value to a 0-based index mapping.
5.  `if (nums[idx] < 0) { duplicates.push_back(val); }`
    If `nums[idx]` is negative, it indicates that a previous element also mapped to this same index. Thus, `val` is a duplicate.
6.  `else { nums[idx] = -nums[idx]; }`
    If the index mapping points to a positive number, we mark it as visited by negating its value.
7.  `for (int& num : nums) { num = std::abs(num); }`
    Clean up phase: Restore the array's state by mapping all negated values back to their positive counterparts.

---

## 9. Dry Run

Given `nums = [2, 1, 2]`, $N = 3$ (expected duplicate: `2`)

| Step | Index `i` | Value `nums[i]` | Target Index `idx` | Target Value `nums[idx]` | Condition: `nums[idx] < 0` | Action / Array State |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| — | — | — | — | — | — | `[2, 1, 2]` |
| 1 | `0` | `2` | `2 - 1 = 1` | `1` | False | Negate `nums[1]`. Array: `[2, -1, 2]` |
| 2 | `1` | `-1` $\to$ `abs = 1` | `1 - 1 = 0` | `2` | False | Negate `nums[0]`. Array: `[-2, -1, 2]` |
| 3 | `2` | `2` | `2 - 1 = 1` | `-1` | True | Duplicate found! Add `2` to result. Array: `[-2, -1, 2]` |
| 4 | Clean up | — | — | — | — | Restore. Array: `[2, 1, 2]` |

---

## 10. Complexity Analysis

| Approach | Time Complexity | Space Complexity (Auxiliary) | Reasoning |
| :--- | :--- | :--- | :--- |
| **Brute Force** | $O(N^2)$ | $O(1)$ | Double loops scan for matches. |
| **Better (Hashing)** | $O(N)$ | $O(N)$ | Uses a visited array/hash set. |
| **Optimal (Index Negation)**| $O(N)$ | $O(1)$ | In-place marking utilizing the sign bit. Restore phase runs in linear time. |

---

## 11. Common Mistakes

1.  **Forgetting to use `abs` when retrieving current values:** `int val = nums[i];` instead of `std::abs(nums[i]);`. Since we negate numbers, we can encounter a negative value, yielding an out-of-bounds array index error.
2.  **Incorrect mapping:** Mapping `val` to `val` instead of `val - 1` (0-based indexing), causing out-of-bounds errors on the upper limit $N$.
3.  **Not restoring the array:** Leaving the array elements negated when returning. (Check with the interviewer if the input is allowed to be modified permanently).

---

## 12. Edge Cases

| Edge Case | Input Example | Expected Output | Why It Matters / How Code Handles It |
| :--- | :--- | :--- | :--- |
| **No Duplicates** | `[1, 2, 3]` | `[]` | No indices are visited twice. Return empty vector. |
| **All Duplicates** | `[1, 1, 2, 2]` | `[1, 2]` | Correctly maps and finds both duplicates. |
| **Minimal Input size** | `[1, 1]` | `[1]` | Smallest possible array size. Handles boundary condition of indexing. |
| **Elements out of order** | `[2, 1, 1]` | `[1]` | Validates lookup correctness. |

---

## 13. Interview Explanation

> "To find all duplicates in an array of size $N$ with elements ranging from $1$ to $N$, we can use three approaches.
> A brute-force nested search compares all pairs, which takes $O(N^2)$ time.
> A hash map or frequency array allows us to find duplicates in $O(N)$ time but requires $O(N)$ extra space.
> To achieve $O(N)$ time complexity and $O(1)$ auxiliary space, we can treat the input array itself as a hash table.
> Because the elements are in the range $[1, N]$, we can map each element `val` to the index `val - 1`. As we iterate through the array, we look at the element at this mapped index. If the element is positive, we make it negative to mark that we have visited the value. If it is already negative, we know we have encountered this value before, making it a duplicate.
> Finally, we restore the array to its original positive values and return our list of duplicates."

---

## 14. Follow-up Questions

1.  **What if the array is read-only and we cannot use extra space?**
    *   *Answer:* If the array is read-only and we cannot modify it, we can use **Floyd's Cycle Detection Algorithm (Tortoise and Hare)** if there is exactly one duplicate. If there are multiple duplicates, we are forced to either sort (modifying it) or use binary search on the range $[1, N]$ in $O(N \log N)$ time and $O(1)$ space.
2.  **Can we use the Cyclic Sort pattern?**
    *   *Answer:* Yes, we can try to place each number at its correct index (i.e. put `nums[i]` at index `nums[i] - 1`) through swapping. After sorting, any element `nums[i] != i + 1` is a duplicate. This also takes $O(N)$ time and $O(1)$ auxiliary space but modifies the array heavily.

---

## 15. Variations

*   **Find the Duplicate Number (Leetcode 287):**
    *   *Difference:* Exactly one duplicate number exists, but it can be repeated multiple times. The array is read-only. This is solved using Floyd's Cycle Detection Algorithm.
*   **First Missing Positive (Leetcode 41):**
    *   *Difference:* Find the smallest positive missing integer in an unsorted array. This uses a very similar cyclic sort / index-marking strategy.

---

## 16. Revision Notes

*   **Sign-flipping Trick:** Useful when integers are bounded: $1 \le nums[i] \le N$ with array size $N$.
*   **Zero-Based Adjustment:** Always subtract 1 (`val - 1`) to map values to indices.
*   **Safety:** Always take `abs` when referencing the current element.

---

## 17. Memorization Version

```cpp
vector<int> findDuplicates(vector<int>& nums) {
    vector<int> duplicates;
    for (int i = 0; i < nums.size(); ++i) {
        int idx = abs(nums[i]) - 1;
        if (nums[idx] < 0) {
            duplicates.push_back(abs(nums[i]));
        } else {
            nums[idx] = -nums[idx];
        }
    }
    for (int& x : nums) x = abs(x); // Restore original
    return duplicates;
}
```
*Key reminder: Map to `abs(num) - 1`, negate visited, verify if negative.*
