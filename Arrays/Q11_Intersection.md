# Find Intersection of Two Arrays

---

## 1. Problem Statement

Given two integer arrays `nums1` and `nums2`, return an array of their intersection.

There are two common variations of this problem:
* **Variant A (Unique Intersection):** Each element in the result must be **unique**, and you may return the result in any order.
* **Variant B (Frequency-Preserving Intersection):** Each element in the result must appear as many times as it shows in **both** arrays, and you may return the result in any order.

We will explain how to handle **both variants**, focusing on Variant A as the baseline and showing how Variant B differs.

### Input
- `nums1`: An integer array of size `m` ($1 \le m \le 1000$).
- `nums2`: An integer array of size `n` ($1 \le n \le 1000$).

### Output
- An integer array representing the intersection of `nums1` and `nums2`.

### Constraints
- $1 \le nums1.length, nums2.length \le 1000$
- $0 \le nums1[i], nums2[i] \le 1000$

---

## 2. Interview Clarification

Before writing any code, clarify these details with the interviewer:
1. **Should the intersection return only unique elements, or should it preserve duplicates?**
   - *Answer:* Let's implement unique elements (Variant A) first, then discuss preserving duplicate counts (Variant B).
2. **Are the input arrays sorted?**
   - *Answer:* No, they are unsorted. If they were sorted, we could optimize our search.
3. **Can we modify the input arrays?**
   - *Answer:* Yes, you can sort them in-place if needed.

---

## 3. Thinking Process

Let's think aloud:
* **Initial thoughts:** We want to find elements that are present in both `nums1` and `nums2`.
* **Brute Force (Nested Loops):**
  - For each element in `nums1`, search if it exists in `nums2`.
  - If it exists, add it to our output list (checking that it isn't already added to avoid duplicates for Variant A).
  - Time Complexity: $O(m \times n)$ because of the nested search. Space Complexity: $O(\min(m, n))$ to store the unique outputs.
* **Better (Hash Set):**
  - Hash tables allow $O(1)$ lookup times.
  - Insert all elements of `nums1` into a hash set `set1`. This takes $O(m)$ time.
  - Traverse `nums2` and check if each element exists in `set1`. If it does, add it to the output set (to ensure uniqueness) and then convert that set to a vector.
  - Time Complexity: $O(m + n)$. Space Complexity: $O(m + \min(m,n))$ to store the sets.
  - *For Variant B (Duplicates preserved):* Instead of a hash set, we use a hash map (frequency map) for `nums1` containing `value -> count`. For each element in `nums2`, check if it exists in the map with a count $> 0$. If so, add it to the output, decrement the count in the map.
* **Optimal (Sorting + Two Pointers):**
  - What if space is constrained and we cannot allocate hash tables? We can sort both arrays first.
  - Sorting takes $O(m \log m + n \log n)$.
  - Set pointers `i = 0` (for `nums1`) and `j = 0` (for `nums2`).
  - Compare `nums1[i]` and `nums2[j]`.
    - If `nums1[i] == nums2[j]`: We found an intersection element! Add to output.
      - **Unique Intersection (Variant A):** Skip duplicates by incrementing `i` and `j` past any identical values.
      - **Duplicate-preserving (Variant B):** Simply add it and increment both pointers by 1.
    - If `nums1[i] < nums2[j]`: Since the arrays are sorted, `nums1[i]` is too small to match anything further right in `nums2`, so increment `i`.
    - If `nums1[i] > nums2[j]`: `nums2[j]` is too small, so increment `j`.
  - Time Complexity: $O(m \log m + n \log n)$ for sorting, and $O(m + n)$ for pointer scanning. Space Complexity: $O(1)$ auxiliary space if sorted in-place.

---

## 4. Pattern Recognition

This problem belongs to the **Hashing** and **Two Pointers** patterns.
* **Why this topic?** Identifying matching items across collections.
* **Which pattern?**
  - **Two Pointers on Sorted Arrays:** Moving indices based on relative value size to scan elements linearly.
  - **Frequency Hashing:** Tracking item occurrences to match exact overlap counts.

---

## 5. Brute Force Solution

Use nested loops to check if each element of `nums1` is present in `nums2`.

### Algorithm (Variant A)
1. Initialize an empty result vector `result`.
2. Iterate $i$ from $0$ to $m-1$:
   - Check if `nums1[i]` is already in `result` (to avoid duplicates). If so, skip.
   - Iterate $j$ from $0$ to $n-1$:
     - If `nums1[i] == nums2[j]`:
       - Append `nums1[i]` to `result`.
       - Break the inner loop.
3. Return `result`.

### Complexity Analysis
- **Time Complexity:** $O(m \times n)$ — For each element in `nums1`, we scan `nums2` linearly.
- **Space Complexity:** $O(\min(m, n))$ — To store the output array.

### C++17 Code
```cpp
#include <vector>
#include <algorithm>

class Solution {
public:
    std::vector<int> intersection(std::vector<int>& nums1, std::vector<int>& nums2) {
        std::vector<int> result;
        
        for (int i = 0; i < nums1.size(); ++i) {
            // Check if nums1[i] is already added to result
            if (std::find(result.begin(), result.end(), nums1[i]) != result.end()) {
                continue;
            }
            
            // Search in nums2
            for (int j = 0; j < nums2.size(); ++j) {
                if (nums1[i] == nums2[j]) {
                    result.push_back(nums1[i]);
                    break; // Found match, stop searching for this element
                }
            }
        }
        
        return result;
    }
};
```

---

## 6. Better Solution (Hash Set / Hash Map)

Use Hash Set for unique intersection, and Hash Map for duplicate-preserving intersection.

### Algorithm (Variant A - Unique Intersection)
1. Insert all elements of `nums1` into a `std::unordered_set<int> set1`.
2. Create an empty `std::unordered_set<int> resultSet` to store the output.
3. Traverse `nums2`:
   - If an element is found in `set1`, insert it into `resultSet`.
4. Convert `resultSet` to a vector and return.

### Algorithm (Variant B - Duplicate-Preserving Intersection)
1. Create a `std::unordered_map<int, int> frequencies`.
2. Populate the map with counts of elements in `nums1`.
3. Create a result vector `result`.
4. Traverse `nums2`:
   - If the element exists in `frequencies` and its count is $> 0$:
     - Append it to `result`.
     - Decrement the count in `frequencies`.
5. Return `result`.

### Complexity Analysis (Variant A)
- **Time Complexity:** $O(m + n)$ — Inserting $m$ elements into a hash set takes $O(m)$, and scanning $n$ elements takes $O(n)$ average time.
- **Space Complexity:** $O(m + \min(m, n))$ — For the hash sets.

### C++17 Code (Variant A: Unique)
```cpp
#include <vector>
#include <unordered_set>

class SolutionSetUnique {
public:
    std::vector<int> intersection(std::vector<int>& nums1, std::vector<int>& nums2) {
        std::unordered_set<int> set1(nums1.begin(), nums1.end());
        std::unordered_set<int> resultSet;
        
        for (int val : nums2) {
            if (set1.count(val)) {
                resultSet.insert(val);
            }
        }
        
        return std::vector<int>(resultSet.begin(), resultSet.end());
    }
};
```

### C++17 Code (Variant B: Duplicate Preserving)
```cpp
#include <vector>
#include <unordered_map>

class SolutionSetDuplicate {
public:
    std::vector<int> intersect(std::vector<int>& nums1, std::vector<int>& nums2) {
        std::unordered_map<int, int> frequencies;
        for (int num : nums1) {
            frequencies[num]++;
        }
        
        std::vector<int> result;
        for (int num : nums2) {
            if (frequencies[num] > 0) {
                result.push_back(num);
                frequencies[num]--; // Consume one instance
            }
        }
        
        return result;
    }
};
```

---

## 7. Optimal Solution (Sorting + Two Pointers)

If memory is constrained (we want $O(1)$ auxiliary space), sorting both arrays first allows using the two-pointer technique.

### Algorithm (Variant A - Unique Intersection)
1. Sort both `nums1` and `nums2`.
2. Initialize pointers `i = 0`, `j = 0`.
3. While `i < nums1.size()` and `j < nums2.size()`:
   - If `nums1[i] == nums2[j]`:
     - If the result vector is empty or `nums1[i]` is not equal to `result.back()` (to avoid duplicates), add it to `result`.
     - Increment both `i` and `j`.
   - If `nums1[i] < nums2[j]`: increment `i`.
   - If `nums1[i] > nums2[j]`: increment `j`.
4. Return `result`.

### Algorithm (Variant B - Duplicate-Preserving)
* Same as above, but delete the duplicate check (`result.back() != nums1[i]`). Simply push `nums1[i]` every time they match, and increment both pointers by 1.

### C++17 Code (Variant A: Unique)
```cpp
#include <vector>
#include <algorithm>

class Solution {
public:
    std::vector<int> intersection(std::vector<int>& nums1, std::vector<int>& nums2) {
        // Step 1: Sort both arrays
        std::sort(nums1.begin(), nums1.end());
        std::sort(nums2.begin(), nums2.end());
        
        std::vector<int> result;
        int i = 0;
        int j = 0;
        
        // Step 2: Two-pointer scan
        while (i < nums1.size() && j < nums2.size()) {
            if (nums1[i] == nums2[j]) {
                // To maintain unique values in the result
                if (result.empty() || result.back() != nums1[i]) {
                    result.push_back(nums1[i]);
                }
                i++;
                j++;
            } 
            else if (nums1[i] < nums2[j]) {
                i++;
            } 
            else {
                j++;
            }
        }
        
        return result;
    }
};
```

---

## 8. Code Walkthrough

* `std::sort(...)`: Sorts both input vectors in ascending order.
* `while (i < nums1.size() && j < nums2.size())`: Traverses both arrays.
* `if (result.empty() || result.back() != nums1[i])`: This prevents duplicates from entering the output vector. Since the inputs are sorted, any duplicates will be adjacent, so we only need to verify if the last added element is the same as the current match.
* `else if (nums1[i] < nums2[j]) i++;`: If the element in `nums1` is smaller, we increment `i` to find a larger value that might match `nums2[j]`.

---

## 9. Dry Run

Input (Variant A): `nums1 = [4, 9, 5]`, `nums2 = [9, 4, 9, 8, 4]`.
1. Sort `nums1` $\rightarrow$ `[4, 5, 9]`.
2. Sort `nums2` $\rightarrow$ `[4, 4, 8, 9, 9]`.
3. Pointers: `i = 0`, `j = 0`.

| Step | `i` (`nums1[i]`) | `j` (`nums2[j]`) | Comparison | Result Vector | Action |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1** | 0 (4) | 0 (4) | $4 == 4$ | `[4]` | Push `4`, `i++` (1), `j++` (1) |
| **2** | 1 (5) | 1 (4) | $5 > 4$ | `[4]` | `j++` (2) |
| **3** | 1 (5) | 2 (8) | $5 < 8$ | `[4]` | `i++` (2) |
| **4** | 2 (9) | 2 (8) | $9 > 8$ | `[4]` | `j++` (3) |
| **5** | 2 (9) | 3 (9) | $9 == 9$ | `[4, 9]` | Push `9`, `i++` (3), `j++` (4) |

Loop ends because `i = 3` is equal to `nums1.size()`.
Output: `[4, 9]`.

---

## 10. Complexity Analysis

| Variant / Approach | Time Complexity | Space Complexity | Notes |
| :--- | :--- | :--- | :--- |
| **Brute Force** | $O(m \times n)$ | $O(\min(m, n))$ | Slow nested comparisons. |
| **Hash Set (Variant A)**| $O(m + n)$ | $O(m + \min(m, n))$ | Fast, but uses auxiliary hash tables. |
| **Hash Map (Variant B)**| $O(m + n)$ | $O(m + \min(m, n))$ | Preserves exact duplicate occurrence counts. |
| **Two Pointer (Sorted)**| $O(m \log m + n \log n)$| $O(1)$ | No extra space needed (in-place sorting). |

* **Time Complexity (Two Pointer):** Sorting takes $O(m \log m + n \log n)$ time. The linear traversal takes $O(m + n)$. The sorting step dominates, so total time is $O(m \log m + n \log n)$.
* **Space Complexity (Two Pointer):** $O(1)$ auxiliary space if sorted in-place (ignoring recursive stack frame allocations during sorting, which take $O(\log m + \log n)$ space).

---

## 11. Common Mistakes

1. **Forgetting to skip duplicates (Variant A):** Adding the same matching element multiple times.
2. **Infinite loops:** Forgetting to increment BOTH pointers `i++` and `j++` when a match is found.
3. **Out of bounds access:** Using `result.back()` without verifying if `result.empty()` is true.

---

## 12. Edge Cases

| Edge Case | Input Example | Expected Output | Why it matters |
| :--- | :--- | :--- | :--- |
| **No intersection** | `[1, 2]`, `[3, 4]` | `[]` | Pointers should run to completion without finding matches. |
| **Identical arrays** | `[1, 1, 2]`, `[1, 1, 2]` | `[1, 2]` (Unique) | Must filter duplicates properly. |
| **One empty array** | `[]`, `[1, 2]` | `[]` | Loop condition `i < size` must fail immediately. |

---

## 13. Interview Explanation

"To find the intersection of two arrays, we have two common requirements: returning only unique common elements or preserving the frequency of common elements.
For unique elements, a brute force approach uses nested loops in $O(m \times n)$ time. We can optimize this to $O(m + n)$ time using a Hash Set by storing elements of the first array and checking them against the second.
If we want to avoid extra memory usage, we can sort both arrays first and use a two-pointer approach. We start at the beginning of both sorted arrays and move pointers inward based on element comparisons. If the elements match, we add them to the result (handling duplicates if we need unique values) and increment both pointers. This takes $O(m \log m + n \log n)$ time and $O(1)$ auxiliary space."

---

## 14. Follow-up Questions

1. **Which approach is better if one array is extremely small and the other is very large?**
   * *Answer:* If `nums1` is small ($m \ll n$), storing `nums1` in a Hash Set is very efficient because the set size is small ($O(m)$ space) and lookup takes $O(n)$ time. Alternatively, we can sort the large array and perform binary search for each element of the small array in $O(m \log n)$ time and $O(1)$ space.
2. **What if the arrays are already sorted?**
   * *Answer:* The Two-Pointer approach becomes the most optimal choice immediately, yielding $O(m + n)$ time and $O(1)$ space.

---

## 15. Variations

1. **Intersection of Three Sorted Arrays:** Use three pointers and increment the pointers pointing to smaller elements.
2. **Union of Two Arrays:** Find all distinct elements from both arrays combined.

---

## 16. Revision Notes

* **Unique Check Trick:** `if (result.empty() || result.back() != nums1[i])` works on sorted outputs.
* **Freq Map Trick (Variant B):** Decrement frequency counts to consume matched items.
* **Pointers:** Move pointer of the smaller element forward to find a match.

---

## 17. Memorization Version

```cpp
// Unique Intersection (Sorted)
sort(A.begin(), A.end());
sort(B.begin(), B.end());
int i = 0, j = 0;
while (i < A.size() && j < B.size()) {
    if (A[i] == B[j]) {
        if (res.empty() || res.back() != A[i]) res.push_back(A[i]);
        i++; j++;
    } else if (A[i] < B[j]) i++;
    else j++;
}
```
* **Time:** $O(m \log m + n \log n)$, **Space:** $O(1)$ auxiliary.
