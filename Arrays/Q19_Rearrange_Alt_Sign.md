# Rearrange Array in Alternating Positive and Negative Sign

## 1. Problem Statement

Given an array of integers `nums`, rearrange the elements such that they follow an alternating pattern of positive and negative numbers. 

### Case 1: LeetCode Version (Equal Counts)
- **Input:** An array `nums` of even length $N$, containing exactly $N/2$ positive integers and $N/2$ negative integers.
- **Output:** Rearranged array starting with a positive integer, followed by a negative integer, and alternating thereafter. The relative order of same-sign integers must be preserved (stable rearrangement).
- **Constraints:**
  - $2 \le N \le 2 \times 10^5$
  - `nums` length is even.
  - Exactly $N/2$ positive and $N/2$ negative integers.
  - $-10^5 \le nums[i] \le 10^5$ (no zeros).

### Case 2: General Version (Unequal Counts)
- **Input:** An array `nums` of length $N$ containing any number of positive and negative integers.
- **Output:** Rearranged array starting with a positive integer, followed by a negative integer, and alternating thereafter. If one sign runs out of elements, the remaining elements of the other sign should be appended to the end of the array while maintaining their relative order.
- **Constraints:**
  - $1 \le N \le 2 \times 10^5$
  - $-10^5 \le nums[i] \le 10^5$ (treat $0$ as positive or omit based on standard convention; usually, assume no zeros or treat $0$ as positive).

---

## 2. Interview Clarification Questions

Before writing any code, a candidate should clarify:
1. **Q:** Can the array contain zero? If yes, should we treat it as positive or negative?
   * **A:** Assume no zeros, or if zero is present, treat it as positive.
2. **Q:** Does the relative order of positive and negative numbers need to be preserved (stable)?
   * **A:** Yes, stability is required for both the equal and unequal count versions.
3. **Q:** Can we use extra space?
   * **A:** For the stable version, $O(N)$ extra space is permitted and optimal for $O(N)$ time complexity. If stability is not required, we can discuss in-place $O(1)$ space partition solutions.
4. **Q:** Should the array start with a positive or a negative number?
   * **A:** The output should start with a positive number.

---

## 3. Thinking Process

### Step 1: First Instinct (Brute Force)
For the equal-count version, the simplest idea is to separate all positive numbers and negative numbers into two auxiliary arrays: `pos` and `neg`.
- Traverse `nums` and push positive elements to `pos` and negative elements to `neg`.
- Since counts are equal, both will have size $N/2$.
- Reconstruct the array by alternating: index $2 \times i$ gets `pos[i]` and index $2 \times i + 1$ gets `neg[i]`.
- This works, but requires two traversals of the input array (one to separate, one to merge) and $O(N)$ auxiliary space.

### Step 2: Optimizing the Equal Count Version (Single-Pass)
Can we do this in a single pass with a single auxiliary array?
- We know that positive elements will occupy even indices (`0, 2, 4, ...`).
- Negative elements will occupy odd indices (`1, 3, 5, ...`).
- We can maintain two pointers for the result array: `posIdx = 0` and `negIdx = 1`.
- We iterate through `nums` once. For each element:
  - If it is positive, we place it at `result[posIdx]` and increment `posIdx` by 2.
  - If it is negative, we place it at `result[negIdx]` and increment `negIdx` by 2.
- This achieves the rearrangement in a single pass of the input, maintaining stability and using only one output array.

### Step 3: Handling the General Version (Unequal Counts)
When counts are not equal, the fixed-index relation breaks. For instance, if there are only 2 positive numbers and 5 negative numbers:
- We can alternate positive and negative numbers up to index $2 \times \min(N_{pos}, N_{neg})$.
- The remaining elements of the majority sign must be appended at the end.
- To solve this:
  1. Separate elements into two vectors: `pos` and `neg`.
  2. Use two pointers `i` and `j` initialized to 0, traversing `pos` and `neg` respectively.
  3. Alternate appending to `nums` as long as both pointers are within bounds.
  4. Once one pointer reaches the end, append all remaining elements from the other vector.

---

## 4. Pattern Recognition

This problem fits the **Two-Pointer / Auxiliary Array Partitioning** pattern.
- **Why?** We are partitioning elements based on a condition (sign) and placing them at designated indices.
- **Key Hint:** The requirement to maintain relative order (stability) restricts us from using simple in-place swaps like QuickSelect partitioning, unless we accept $O(N^2)$ time complexity. Thus, utilizing helper pointers to construct the final layout is the standard approach.

---

## 5. Brute Force Solution (Separate & Merge)

### Algorithm
1. Create two vectors: `positives` and `negatives`.
2. Iterate through the input array `nums`. If `nums[i] > 0`, push it to `positives`, else push it to `negatives`.
3. Clear or overwrite the original array `nums` (or create a new array).
4. Merge them back by alternating:
   - For `nums[i]` where `i` is even, pick from `positives`.
   - For `nums[i]` where `i` is odd, pick from `negatives`.

### Dry Run
Input: `nums = [1, -2, 3, -4]`
1. Separation:
   - `positives = [1, 3]`
   - `negatives = [-2, -4]`
2. Reconstruction:
   - `i = 0` (even): `nums[0] = positives[0]` -> `nums = [1, -2, 3, -4]`
   - `i = 1` (odd): `nums[1] = negatives[0]` -> `nums = [1, -2, 3, -4]`
   - `i = 2` (even): `nums[2] = positives[1]` -> `nums = [1, -2, 3, -4]`
   - `i = 3` (odd): `nums[3] = negatives[1]` -> `nums = [1, -2, 3, -4]`
Output: `[1, -2, 3, -4]`

### Complexity
- **Time Complexity:** $O(N)$ because we iterate through the array twice (once for filtering, once for merging).
- **Space Complexity:** $O(N)$ to store the filtered positive and negative elements.

### Pros and Cons
- **Pros:** Extremely simple to implement; handles unequal counts easily with minor modifications.
- **Cons:** Performs two passes and uses extra memory for two separate arrays.

### C++17 Code
```cpp
#include <vector>

std::vector<int> rearrangeArrayBrute(const std::vector<int>& nums) {
    std::vector<int> positives;
    std::vector<int> negatives;
    
    // Step 1: Separate elements by sign
    for (int num : nums) {
        if (num > 0) {
            positives.push_back(num);
        } else {
            negatives.push_back(num);
        }
    }
    
    std::vector<int> result(nums.size());
    int posIndex = 0, negIndex = 0;
    
    // Step 2: Merge in alternating sequence
    for (size_t i = 0; i < nums.size(); ++i) {
        if (i % 2 == 0) {
            result[i] = positives[posIndex++];
        } else {
            result[i] = negatives[negIndex++];
        }
    }
    return result;
}
```

---

## 6. Better Solution

For Case 1 (Equal Counts), the brute force solution can be optimized to a single-pass implementation. This skips the step of storing intermediates in two separate arrays, placing elements directly into the result array.

Since this optimization leads directly to the optimal solution for Case 1, we will jump directly to the **Optimal Solution** section to detail it.

---

## 7. Optimal Solution

We present solutions for both cases.

### Case 1: Equal Counts (LeetCode) — Single-Pass Two-Pointer
We use a result array of size $N$ and two pointers:
- `posIdx` starting at `0` (even indices).
- `negIdx` starting at `1` (odd indices).
We iterate through `nums`. For every positive number, we assign it to `result[posIdx]` and add 2 to `posIdx`. For every negative number, we assign it to `result[negIdx]` and add 2 to `negIdx`.

### Case 2: Unequal Counts — Partition & Merge Leftovers
If positive and negative counts are unequal:
1. Extract positives and negatives into two arrays `pos` and `neg`.
2. Place them alternatingly as far as possible using two pointers `i` and `j`.
3. Append any remaining elements from the larger list.

### Dry Run (Case 1: Equal Counts)
Input: `nums = [3, 1, -2, -5, 2, -4]`
Initialize: `result = [0, 0, 0, 0, 0, 0]`, `posIdx = 0`, `negIdx = 1`

| Step | Element | Type | Action | posIdx | negIdx | result |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | 3 | Positive | `result[0] = 3` | 2 | 1 | `[3, 0, 0, 0, 0, 0]` |
| 2 | 1 | Positive | `result[2] = 1` | 4 | 1 | `[3, 0, 1, 0, 0, 0]` |
| 3 | -2 | Negative | `result[1] = -2` | 4 | 3 | `[3, -2, 1, 0, 0, 0]` |
| 4 | -5 | Negative | `result[3] = -5` | 4 | 5 | `[3, -2, 1, -5, 0, 0]` |
| 5 | 2 | Positive | `result[4] = 2` | 6 | 5 | `[3, -2, 1, -5, 2, 0]` |
| 6 | -4 | Negative | `result[5] = -4` | 6 | 7 | `[3, -2, 1, -5, 2, -4]` |

### C++17 Code

```cpp
#include <vector>
#include <stdexcept>

// Case 1: Equal Counts (Optimal Single-Pass)
std::vector<int> rearrangeArrayEqual(const std::vector<int>& nums) {
    int n = nums.size();
    std::vector<int> result(n);
    int posIndex = 0;
    int negIndex = 1;
    
    for (int num : nums) {
        if (num > 0) {
            result[posIndex] = num;
            posIndex += 2;
        } else {
            result[negIndex] = num;
            negIndex += 2;
        }
    }
    return result;
}

// Case 2: Unequal Counts (Optimal Stability-Preserving)
std::vector<int> rearrangeArrayGeneral(const std::vector<int>& nums) {
    std::vector<int> positives;
    std::vector<int> negatives;
    
    // Separate numbers
    for (int num : nums) {
        if (num >= 0) {
            positives.push_back(num);
        } else {
            negatives.push_back(num);
        }
    }
    
    std::vector<int> result;
    result.reserve(nums.size());
    
    size_t posPtr = 0, negPtr = 0;
    
    // Alternating placement
    while (posPtr < positives.size() && negPtr < negatives.size()) {
        result.push_back(positives[posPtr++]);
        result.push_back(negatives[negPtr++]);
    }
    
    // Append leftovers from positives
    while (posPtr < positives.size()) {
        result.push_back(positives[posPtr++]);
    }
    
    // Append leftovers from negatives
    while (negPtr < negatives.size()) {
        result.push_back(negatives[negPtr++]);
    }
    
    return result;
}
```

---

## 8. Code Walkthrough

### Case 1 (Equal Counts):
1. **Initialize result:** `std::vector<int> result(n);` allocates space for the output array.
2. **Two Pointer Initialization:** `posIndex` is set to `0` (for even indices) and `negIndex` is set to `1` (for odd indices).
3. **Iterate Input:** A simple range-based `for` loop inspects each element `num`.
4. **Conditional Assignment:**
   - If `num > 0`, it goes into the next available even position (`posIndex`), then `posIndex` increases by 2.
   - Otherwise, it goes into the next available odd position (`negIndex`), then `negIndex` increases by 2.
5. **Return:** The fully populated `result` array is returned.

### Case 2 (Unequal Counts):
1. **Partitioning phase:** `positives` and `negatives` arrays are filled.
2. **Double Pointers:** `posPtr` and `negPtr` index into their respective sign-specific arrays.
3. **Alternate Loop:** The `while` loop checks that both lists have elements remaining, adding one positive followed by one negative.
4. **Clean up Loops:** After one list is exhausted, two subsequent loops push any remaining elements of the larger list.

---

## 9. Dry Run

Let's dry run the **General Version** (Unequal Counts) with input `nums = [1, 2, -3, -4, -5]`.

### Partitioning Phase:
- `positives = [1, 2]`
- `negatives = [-3, -4, -5]`

### Merging Phase:

| Step | condition `posPtr < pos.size() && negPtr < neg.size()` | action | posPtr | negPtr | result |
| :--- | :--- | :--- | :--- | :--- | :--- |
| Start | - | - | 0 | 0 | `[]` |
| 1 | True (`0 < 2 && 0 < 3`) | Push `pos[0]` (1), Push `neg[0]` (-3) | 1 | 1 | `[1, -3]` |
| 2 | True (`1 < 2 && 1 < 3`) | Push `pos[1]` (2), Push `neg[1]` (-4) | 2 | 2 | `[1, -3, 2, -4]` |
| 3 | False (`2 < 2` is false) | Exit main loop | 2 | 2 | `[1, -3, 2, -4]` |

### Leftover Cleanup:
- `posPtr < positives.size()` is `2 < 2` (False) -> skip.
- `negPtr < negatives.size()` is `2 < 3` (True) -> Push `neg[2]` (-5), `negPtr` becomes 3.
- `negPtr < negatives.size()` is `3 < 3` (False) -> exit.

Final Output: `[1, -3, 2, -4, -5]`

---

## 10. Complexity Analysis

| Version / Approach | Time Complexity | Space Complexity | Notes |
| :--- | :--- | :--- | :--- |
| **Case 1: Equal (Brute)** | $O(N)$ | $O(N)$ | Two traversals, separate containers. |
| **Case 1: Equal (Optimal)** | $O(N)$ | $O(N)$ | Single traversal, direct placement. |
| **Case 2: Unequal (Optimal)** | $O(N)$ | $O(N)$ | Preserves stability; necessary extra space. |
| **Case 2: In-place Stable** | $O(N^2)$ | $O(1)$ | Shift elements rightwards to maintain order. |

*Why $O(N)$ space is optimal for stable solutions:* To maintain the relative order of positive and negative numbers without spending $O(N^2)$ time shifting elements, we must record the elements in sequential order, which mathematically requires $O(N)$ auxiliary space.

---

## 11. Common Mistakes

1. **Incorrect pointer advancement:** Incrementing pointers by `1` instead of `2` in the equal-count version, causing positives and negatives to overwrite each other.
2. **Losing stability:** Using standard two-pointer partitioning (swap from front/back like QuickSort) which violates the requirement of maintaining relative order.
3. **Out of bounds errors:** Not checking if `posIndex` or `negIndex` exceeds $N-1$ when appending.
4. **Ignoring unequal leftovers:** Forgetting to append the rest of the elements in the general version when one sign list is fully exhausted.

---

## 12. Edge Cases

| Edge Case | Input Example | Why it matters | Correct Output |
| :--- | :--- | :--- | :--- |
| **Minimum odd size (General)** | `[1]` | Array has only one positive element. | `[1]` |
| **No Negative Elements** | `[4, 2, 7]` | No negatives to alternate with. | `[4, 2, 7]` |
| **No Positive Elements** | `[-1, -3, -5]` | No positives to start with. | `[-1, -3, -5]` |
| **Already Alternating** | `[1, -2, 3, -4]` | Code must not alter the correct alignment. | `[1, -2, 3, -4]` |

---

## 13. Interview Explanation

"For the equal count version of this problem, we can solve it optimally in $O(N)$ time and $O(N)$ auxiliary space by placing positive and negative numbers directly into their target indices in a single pass. Since positives occupy even indices $(0, 2, 4...)$ and negatives occupy odd indices $(1, 3, 5...)$, we maintain two pointers starting at $0$ and $1$. As we iterate through the input, we direct positive values to the even pointer and negatives to the odd pointer, incrementing them by $2$ each time. 

If the counts are unequal, we cannot map them directly to fixed indices. Instead, we extract positives and negatives into separate containers. We then use two pointers to alternate putting them back into the result array. Once one of the containers is exhausted, we append the remaining elements of the other container to the end, ensuring we preserve the original relative ordering."

---

## 14. Follow-up Questions

1. **Q: Can we solve the equal counts version in-place with $O(1)$ auxiliary space while preserving stability?**
   * *A:* Doing it in $O(1)$ space while keeping stability requires shifting elements, which takes $O(N^2)$ time in the worst case (resembling insertion sort). An alternative is a modified merge-sort approach that takes $O(N \log N)$ time and $O(\log N)$ recursion stack space using block swaps.
2. **Q: If stability is NOT required, how do we solve the unequal version in-place in $O(N)$ time?**
   * *A:* We can use a modified partition algorithm. First, partition the array so that all positive numbers are on one side and negatives on the other (using a two-pointer swap). Once they are separated, we swap elements from the positive half with elements in the negative half at alternate positions.

---

## 15. Variations

1. **Rearrange positive and negative elements without changing relative order in $O(N^2)$ time and $O(1)$ space:**
   - Instead of auxiliary arrays, when we find an out-of-place element, we search for the next element of the opposite sign and rotate the subarray between them.
2. **Rearrange such that all negatives appear before positives (no alternating requirement):**
   - This is the standard Dutch National Flag / Partition problem, solved in $O(N)$ time and $O(1)$ space using a single pass with two pointers.

---

## 16. Revision Notes

- **Alternate Indices:** Positive indices are `2 * i`, Negative indices are `2 * i + 1`.
- **Stability Constraint:** Stable = Auxiliary space ($O(N)$) OR slow time ($O(N^2)$). Unstable = Swaps are possible.
- **Leftover Check:** When processing unequal lists, always follow up the merge loop with cleanup loops for leftovers.

---

## 17. Memorization Version

```cpp
// 1-min Review Version: Equal Counts
vector<int> rearrange(vector<int>& nums) {
    int n = nums.size(), pos = 0, neg = 1;
    vector<int> ans(n);
    for (int x : nums) {
        if (x > 0) { ans[pos] = x; pos += 2; }
        else { ans[neg] = x; neg += 2; }
    }
    return ans;
}
```
*Complexity:* $O(N)$ Time, $O(N)$ Space. Preserves stability.
