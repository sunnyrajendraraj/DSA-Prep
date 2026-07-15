# Find Union of Two Arrays

---

## 1. Problem Statement

Given two integer arrays `nums1` and `nums2`, find their union. The union of two arrays is defined as a collection of all **distinct** elements present in both arrays combined. The output can be returned in any order (unless specified as sorted).

### Input
- `nums1`: An integer array of size `m` ($1 \le m \le 10^5$).
- `nums2`: An integer array of size `n` ($1 \le n \le 10^5$).

### Output
- An integer array containing the distinct elements representing the union of `nums1` and `nums2`.

### Constraints
- $1 \le m, n \le 10^5$
- $-10^9 \le nums1[i], nums2[j] \le 10^9$

### Observations
1. **Uniqueness:** The key constraint is that the output must not contain duplicate values, even if a value appears multiple times in `nums1` or `nums2`.
2. **Order:** Union is mathematically unordered, but sorting the inputs can help optimize the identification of duplicates.

---

## 2. Interview Clarification

Before writing code, clarify these points with the interviewer:
1. **Does the union array need to be sorted?**
   - *Answer:* Not strictly, but returning it sorted is a common requirement. Let's design the optimal two-pointer solution to output sorted union elements.
2. **Can the input arrays contain duplicates within themselves?**
   - *Answer:* Yes, and they must be filtered out in the union.
3. **What is the expected behavior for large inputs?**
   - *Answer:* The algorithm should handle up to $10^5$ elements efficiently, which makes a quadratic time complexity unacceptable.

---

## 3. Thinking Process

Let's think aloud:
* **Initial thoughts:** We need to combine all elements from `nums1` and `nums2` while removing any duplicates.
* **Brute Force (Concatenate, Sort, and Unique):**
  - Combine both arrays into a single large array.
  - Sort the combined array of size $m + n$.
  - Traverse the sorted array and copy only unique elements (i.e. elements that are different from their predecessor) to our output.
  - Time Complexity: $O((m+n) \log(m+n))$ due to sorting. Space Complexity: $O(m + n)$ to store the concatenated array.
* **Better (Hash Set):**
  - Hash Sets are designed to automatically filter out duplicate values.
  - Create a `std::unordered_set<int> unionSet`.
  - Insert all elements of `nums1` into `unionSet`.
  - Insert all elements of `nums2` into `unionSet`.
  - Convert `unionSet` to a vector.
  - Time Complexity: $O(m + n)$ average time. Space Complexity: $O(m + n)$ to store the set.
* **Optimal (Two Pointers on Sorted Arrays):**
  - If we want to achieve $O(1)$ auxiliary space (excluding the output array) or if the inputs are already sorted, we can use the two-pointer technique.
  - First, sort both `nums1` and `nums2` (if they are not already sorted). Sorting takes $O(m \log m + n \log n)$.
  - Set pointer `i = 0` (for `nums1`) and pointer `j = 0` (for `nums2`).
  - Compare `nums1[i]` and `nums2[j]`.
    - If `nums1[i] < nums2[j]`:
      - Add `nums1[i]` to the result (if it's not a duplicate of the last added element).
      - Increment `i`.
    - If `nums1[i] > nums2[j]`:
      - Add `nums2[j]` to the result (if it's not a duplicate of the last added element).
      - Increment `j`.
    - If `nums1[i] == nums2[j]`:
      - Add `nums1[i]` to the result (if not duplicate).
      - Increment both `i` and `j`.
  - After the loop, if there are remaining elements in `nums1` or `nums2`, add them while checking for duplicates.
  - Time Complexity: $O(m \log m + n \log n)$ due to sorting. Traversal takes $O(m+n)$. Space Complexity: $O(1)$ auxiliary space if sorted in-place.

---

## 4. Pattern Recognition

This problem belongs to the **Hashing** and **Two Pointers** patterns.
* **Why this topic?** Collecting unique items across multiple groups.
* **Which pattern?**
  - **Two Pointers (Parallel Scan):** Merging sorted sequences by incrementing the pointer pointing to the smaller element, while checking for duplicates against the last written result.
  - **Hash Set Unique Filtering:** Adding elements to a hash set for automatic deduplication.

---

## 5. Brute Force Solution

Concatenate the arrays, sort the combination, and extract unique elements.

### Algorithm
1. Create a vector `combined` of size $m+n$.
2. Copy all elements of `nums1` and `nums2` into `combined`.
3. Sort `combined` in ascending order.
4. Create an empty vector `result`.
5. Iterate through `combined`:
   - If `result` is empty or the current element is different from the last element in `result`, append the current element to `result`.
6. Return `result`.

### Dry Run
Input: `nums1 = [3, 1, 3]`, `nums2 = [2, 1]`.
1. `combined = [3, 1, 3, 2, 1]`.
2. Sorted: `combined = [1, 1, 2, 3, 3]`.
3. Unique extraction:
   - Item 1: `1` $\rightarrow$ Push (result is empty) $\rightarrow$ `result = [1]`.
   - Item 2: `1` $\rightarrow$ Skip (matches `result.back()`).
   - Item 3: `2` $\rightarrow$ Push (doesn't match `1`) $\rightarrow$ `result = [1, 2]`.
   - Item 4: `3` $\rightarrow$ Push (doesn't match `2`) $\rightarrow$ `result = [1, 2, 3]`.
   - Item 5: `3` $\rightarrow$ Skip.
4. Return `[1, 2, 3]`.

### Complexity Analysis
- **Time Complexity:** $O((m+n) \log(m+n))$ — Due to sorting the combined array.
- **Space Complexity:** $O(m + n)$ — To store the concatenated array before sorting.

### C++17 Code
```cpp
#include <vector>
#include <algorithm>

class Solution {
public:
    std::vector<int> makeUnion(std::vector<int>& nums1, std::vector<int>& nums2) {
        std::vector<int> combined;
        combined.reserve(nums1.size() + nums2.size());
        
        // Step 1: Concatenate both arrays
        combined.insert(combined.end(), nums1.begin(), nums1.end());
        combined.insert(combined.end(), nums2.begin(), nums2.end());
        
        // Step 2: Sort
        std::sort(combined.begin(), combined.end());
        
        // Step 3: Extract unique elements
        std::vector<int> result;
        for (int val : combined) {
            if (result.empty() || result.back() != val) {
                result.push_back(val);
            }
        }
        
        return result;
    }
};
```

---

## 6. Better Solution (Hash Set)

Use a Hash Set to automatically filter out duplicate values.

### Algorithm
1. Create a `std::unordered_set<int> unionSet`.
2. Iterate through `nums1` and insert every element into `unionSet`.
3. Iterate through `nums2` and insert every element into `unionSet`.
4. Convert `unionSet` to a vector `result` and return.

### Complexity Analysis
- **Time Complexity:** $O(m + n)$ — Inserting $m+n$ elements into a hash set takes linear time on average.
- **Space Complexity:** $O(m + n)$ — Required to store elements in the hash set.

### C++17 Code
```cpp
#include <vector>
#include <unordered_set>

class Solution {
public:
    std::vector<int> makeUnion(std::vector<int>& nums1, std::vector<int>& nums2) {
        // Hash set handles deduplication
        std::unordered_set<int> unionSet;
        
        for (int val : nums1) {
            unionSet.insert(val);
        }
        
        for (int val : nums2) {
            unionSet.insert(val);
        }
        
        // Convert set to vector
        return std::vector<int>(unionSet.begin(), unionSet.end());
    }
};
```

---

## 7. Optimal Solution (Sorting + Two Pointers)

If the inputs are already sorted, or if we want to run in-place without $O(m+n)$ auxiliary set storage, we sort both arrays and use the Two-Pointer approach.

### Algorithm
1. Sort both `nums1` and `nums2`.
2. Initialize pointers `i = 0` (for `nums1`) and `j = 0` (for `nums2`).
3. Create an empty result vector `result`.
4. While `i < m` and `j < n`:
   - If `nums1[i] < nums2[j]`:
     - If `result` is empty or `result.back() != nums1[i]`, push `nums1[i]`.
     - Increment `i`.
   - Else if `nums1[i] > nums2[j]`:
     - If `result` is empty or `result.back() != nums2[j]`, push `nums2[j]`.
     - Increment `j`.
   - Else (i.e. `nums1[i] == nums2[j]`):
     - If `result` is empty or `result.back() != nums1[i]`, push `nums1[i]`.
     - Increment both `i` and `j`.
5. While `i < m` (flush remaining elements in `nums1`):
   - If `result.empty() || result.back() != nums1[i]`, push `nums1[i]`.
   - Increment `i`.
6. While `j < n` (flush remaining elements in `nums2`):
   - If `result.empty() || result.back() != nums2[j]`, push `nums2[j]`.
   - Increment `j`.
7. Return `result`.

### C++17 Code
```cpp
#include <vector>
#include <algorithm>

class Solution {
private:
    // Helper function to push unique elements into the result vector
    void pushUnique(std::vector<int>& result, int val) {
        if (result.empty() || result.back() != val) {
            result.push_back(val);
        }
    }

public:
    std::vector<int> makeUnion(std::vector<int>& nums1, std::vector<int>& nums2) {
        int m = nums1.size();
        int n = nums2.size();
        
        // Step 1: Sort both arrays
        std::sort(nums1.begin(), nums1.end());
        std::sort(nums2.begin(), nums2.end());
        
        std::vector<int> result;
        int i = 0;
        int j = 0;
        
        // Step 2: Parallel scan
        while (i < m && j < n) {
            if (nums1[i] < nums2[j]) {
                pushUnique(result, nums1[i]);
                i++;
            } 
            else if (nums2[j] < nums1[i]) {
                pushUnique(result, nums2[j]);
                j++;
            } 
            else {
                pushUnique(result, nums1[i]);
                i++;
                j++;
            }
        }
        
        // Step 3: Flush remaining elements of nums1
        while (i < m) {
            pushUnique(result, nums1[i]);
            i++;
        }
        
        // Step 4: Flush remaining elements of nums2
        while (j < n) {
            pushUnique(result, nums2[j]);
            j++;
        }
        
        return result;
    }
};
```

---

## 8. Code Walkthrough

* `std::sort(...)`: Orders both arrays to group duplicates together and align elements by relative value size.
* `pushUnique(...)`: Avoids repetitive code by wrapping the duplicate check (`result.empty() || result.back() != val`) inside a separate helper function.
* `while (i < m && j < n)`: Standard merge-style iteration. We always select the smaller of the two elements to place in the union first, keeping the union sorted.
* `while (i < m)` and `while (j < n)`: Once one of the arrays is exhausted, we push the remaining elements of the other array. Duplicates are still filtered out during this phase.

---

## 9. Dry Run

Input: `nums1 = [1, 2, 2, 3]`, `nums2 = [2, 4]`.
(Already sorted for simplicity). $m = 4$, $n = 2$.
Pointers: `i = 0`, `j = 0`.

| Step | `i` (`nums1[i]`) | `j` (`nums2[j]`) | Comparison | Result Vector | Action |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1** | 0 (1) | 0 (2) | $1 < 2$ | `[1]` | Push `1`, `i++` (1) |
| **2** | 1 (2) | 0 (2) | $2 == 2$ | `[1, 2]` | Push `2`, `i++` (2), `j++` (1) |
| **3** | 2 (2) | 1 (4) | $2 < 4$ | `[1, 2]` | Skip duplicate, push fails, `i++` (3) |
| **4** | 3 (3) | 1 (4) | $3 < 4$ | `[1, 2, 3]` | Push `3`, `i++` (4) |
| **Flush**| 4 (done) | 1 (4) | `i` exhausted | `[1, 2, 3, 4]` | Push remaining `nums2[1]` (4), `j++` (2) |

Loop finishes. Output: `[1, 2, 3, 4]`.

---

## 10. Complexity Analysis

| Approach | Time Complexity | Space Complexity | Notes |
| :--- | :--- | :--- | :--- |
| **Brute Force** | $O((m+n) \log(m+n))$ | $O(m + n)$ | Concat + sort. Easy but memory heavy. |
| **Hash Set** | $O(m + n)$ | $O(m + n)$ | Fastest when inputs are unsorted. |
| **Two Pointer** | $O(m \log m + n \log n)$| $O(1)$ auxiliary | Best if arrays are already sorted. |

* **Time Complexity (Two Pointer):** Sorting takes $O(m \log m + n \log n)$. Scanning both arrays takes $O(m + n)$. The sorting step dominates.
* **Space Complexity (Two Pointer):** $O(1)$ auxiliary space if sorted in-place. If the output array is included, space is $O(m + n)$.

---

## 11. Common Mistakes

1. **Forgetting to flush remaining elements:** Ending the process once the first pointer reaches the end, missing elements in the other array.
2. **Missing duplicate check on flush:** Neglecting to check for duplicates in the tail flush loops, resulting in duplicates at the end of the union array.
3. **Array index bounds:** Using `result.back()` without checking if `result.empty()` is true.

---

## 12. Edge Cases

| Edge Case | Input Example | Expected Output | Why it matters |
| :--- | :--- | :--- | :--- |
| **One empty array** | `[]`, `[1, 2]` | `[1, 2]` | Loop condition fails immediately; flush handles the rest. |
| **Completely overlapping** | `[1, 2]`, `[1, 2]` | `[1, 2]` | Checks duplicate elimination when values are identical. |
| **Distinct ranges** | `[1, 2]`, `[3, 4]` | `[1, 2, 3, 4]` | Verifies correct ordering and transition. |

---

## 13. Interview Explanation

"To find the union of two arrays, we want to collect all unique elements from both arrays.
A simple way is to use a Hash Set. We insert all elements of both arrays into the set, which automatically removes duplicates. This takes $O(m + n)$ time and $O(m + n)$ space.
Alternatively, if we want to save space or if the arrays are already sorted, we can use the Two-Pointer approach. We sort both arrays first, then place pointers at the beginning of each array. We compare the elements and add the smaller one to our result while skipping duplicates, and increment that pointer. If they match, we add the value once and increment both pointers. Finally, we flush any remaining elements from either array into the result. This takes $O(m \log m + n \log n)$ time and $O(1)$ auxiliary space."

---

## 14. Follow-up Questions

1. **What if the arrays are sorted and we want to optimize for space?**
   * *Answer:* The Two-Pointer approach is already optimal, requiring only $O(1)$ auxiliary space.
2. **How does standard library `std::set_union` compare?**
   * *Answer:* C++ provides `std::set_union` in the `<algorithm>` library. It works on sorted ranges using the same two-pointer logic we implemented.

---

## 15. Variations

1. **Intersection of Two Arrays:** Finding elements that appear in both arrays instead of all elements.
2. **Difference of Two Arrays (`nums1 - nums2`):** Find all distinct elements present in `nums1` but not in `nums2`.

---

## 16. Revision Notes

* **Flush rule:** Never forget the remaining elements loops: `while (i < m)` and `while (j < n)`.
* **Push check helper:** Standardizing `result.empty() || result.back() != val` avoids repeated code blocks.
* **Complexity:** Unsorted inputs $\rightarrow$ Hash Set is fastest ($O(m+n)$). Sorted inputs $\rightarrow$ Two Pointers is best ($O(m+n)$ time, $O(1)$ space).

---

## 17. Memorization Version

```cpp
// Sorted Union
int i = 0, j = 0;
while (i < m && j < n) {
    if (A[i] < B[j]) {
        if (res.empty() || res.back() != A[i]) res.push_back(A[i]);
        i++;
    } else if (B[j] < A[i]) {
        if (res.empty() || res.back() != B[j]) res.push_back(B[j]);
        j++;
    } else {
        if (res.empty() || res.back() != A[i]) res.push_back(A[i]);
        i++; j++;
    }
}
// Flush A and B with duplicate checks
```
* **Time:** $O(m \log m + n \log n)$, **Space:** $O(1)$ auxiliary.
