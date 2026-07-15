# Next Permutation

## 1. Problem Statement

Implement **next permutation**, which rearranges numbers into the lexicographically next greater permutation of numbers.
If such an arrangement is not possible, it must rearrange it as the lowest possible order (i.e., sorted in ascending order).
The replacement must be **in-place** and use only constant extra memory.

### Input
- A vector of integers `nums`.

### Output
- Modify `nums` in-place to represent its next permutation.

### Constraints
- $1 \le nums.size() \le 100$
- $0 \le nums[i] \le 100$

### Observations
1. Let's look at lexicographical order. For numbers `[1, 2, 3]`, the permutations in order are:
   - `[1, 2, 3]`
   - `[1, 3, 2]`
   - `[2, 1, 3]`
   - `[2, 3, 1]`
   - `[3, 1, 2]`
   - `[3, 2, 1]`
2. If the array is sorted in descending order (e.g., `[3, 2, 1]`), no greater permutation is possible. The next permutation is the smallest one: `[1, 2, 3]`.
3. To find a larger permutation, we need to modify the array starting from the rightmost place where a larger digit can replace a smaller digit.

---

## 2. Interview Clarification

Before writing code, clarify these points with the interviewer:
1. **Should the solution be in-place?**
   - *Yes, the problem requires modifying the input array directly and utilizing $O(1)$ auxiliary space.*
2. **How to handle duplicates?**
   - *Duplicate values should be treated correctly. For example, the next permutation of `[1, 1, 5]` is `[1, 5, 1]`.*
3. **What if the array is already the largest permutation?**
   - *As stated, reverse the entire array to make it the lowest permutation (sorted in ascending order).*

---

## 3. Thinking Process

Let's build the intuition step-by-step:
- **First Instinct (Brute Force):**
  We could find all possible permutations of the array, sort them lexicographically, locate the current permutation, and return the next one in the sorted list.
  - To generate all permutations of an array of size $n$, it takes $O(n!)$ time.
  - For $n = 100$, $100!$ is astronomically large, which makes this approach completely infeasible.

- **Identifying the Pattern (Single-pass greedy analysis):**
  How do we manually find the next permutation of a sequence? Let's take `[1, 5, 8, 4, 7, 6, 5, 3, 1]`.
  Look from the right end of the array. We notice that the suffix `[7, 6, 5, 3, 1]` is sorted in descending order. In a descending suffix, no greater permutation can be formed because it is already at its maximum possible lexicographical value.
  
  Therefore, we must find the first element from the right that breaks this descending trend (i.e., decreases).
  Scanning from right to left:
  - `1 -> 3` (increasing)
  - `3 -> 5` (increasing)
  - `5 -> 6` (increasing)
  - `6 -> 7` (increasing)
  - `7 -> 4` (**decreasing!**).
  
  This element `4` at index `3` is our **pivot (or break-point)**. To make the sequence lexicographically larger, we must replace this `4` with the smallest element in the suffix `[7, 6, 5, 3, 1]` that is strictly greater than `4`.
  Scanning the suffix from right to left to find the first element greater than `4`:
  - `1` is not > 4.
  - `3` is not > 4.
  - `5` **is** > 4.
  
  So we **swap** `4` and `5`. The array becomes:
  `[1, 5, 8, 5, 7, 6, 4, 3, 1]`.
  
  Now, the prefix `[1, 5, 8, 5]` is set. To make the entire permutation as small as possible (lexicographically next), we want the suffix `[7, 6, 4, 3, 1]` to be sorted in ascending order.
  Since the suffix was already in descending order, swapping `4` (which was the pivot) with `5` (the next greater element) preserves the descending order of the suffix.
  To turn a descending suffix into an ascending suffix, we simply **reverse** it!
  Reversing `[7, 6, 4, 3, 1]` gives `[1, 3, 4, 6, 7]`.
  
  The final array is:
  `[1, 5, 8, 5, 1, 3, 4, 6, 7]`.
  This is the exact next permutation!

---

## 4. Pattern Recognition

This problem fits the **Suffix Traversal and In-Place Permutation** pattern.
- **Why this topic?** It requires manipulating sequence order lexicographically.
- **Pattern:**
  1. Identify the rightmost dip (ascent from right).
  2. Locate the next greater element in the suffix.
  3. Swap them.
  4. Reverse the suffix to minimize its contribution.

---

## 5. Brute Force Solution

### Algorithm
1. Generate all permutations of the given array recursively.
2. Store them in a set or list to sort them lexicographically.
3. Find the index of the input array in the list.
4. Return the next permutation in the list. If it's the last one, return the first one.

### Complexity Analysis
- **Time Complexity:** $O(n! \cdot n)$ to generate and sort all permutations.
- **Space Complexity:** $O(n! \cdot n)$ to store all permutations.

### Pros and Cons
- **Pros:** Conceptually simple.
- **Cons:** Astronomical time/space requirements, unusable for $n > 10$.

### C++17 Code
```cpp
#include <vector>
#include <algorithm>
#include <set>

class SolutionBrute {
private:
    void generatePermutations(std::vector<int>& nums, int index, 
                              std::set<std::vector<int>>& allPerms) {
        if (index == static_cast<int>(nums.size())) {
            allPerms.insert(nums);
            return;
        }
        for (size_t i = index; i < nums.size(); ++i) {
            std::swap(nums[index], nums[i]);
            generatePermutations(nums, index + 1, allPerms);
            std::swap(nums[index], nums[i]); // Backtrack
        }
    }

public:
    void nextPermutation(std::vector<int>& nums) {
        std::set<std::vector<int>> allPerms;
        std::vector<int> temp = nums;
        generatePermutations(temp, 0, allPerms);
        
        auto it = allPerms.find(nums);
        if (it != allPerms.end() && std::next(it) != allPerms.end()) {
            nums = *std::next(it);
        } else {
            nums = *allPerms.begin(); // Wrap around
        }
    }
};
```

---

## 6. Better Solution

> [!NOTE]
> In an interview, since the brute force is $O(n!)$ and the optimal algorithm is $O(n)$ time and $O(1)$ space, there is no intermediate "better" solution. We jump directly to the optimal three-step algorithm.

---

## 7. Optimal Solution

### Algorithm
The optimal solution is the classic **three-step algorithm**:
1. **Find the break-point (rightmost dip)**: Traverse from right to left. Find the first index `i` (from $n-2$ down to 0) such that `nums[i] < nums[i+1]`.
2. **Find the swap target**: If such index `i` exists:
   - Search from the right end of the array to find the first index `j` such that `nums[j] > nums[i]`.
   - Swap `nums[i]` and `nums[j]`.
3. **Reverse the suffix**: Reverse the elements from index `i + 1` to the end of the array. (If no break-point is found, i.e., $i = -1$, reverse the entire array).

### C++17 Code
```cpp
#include <vector>
#include <algorithm>

class SolutionOptimal {
public:
    void nextPermutation(std::vector<int>& nums) {
        int n = nums.size();
        
        // Step 1: Find the rightmost dip (first decreasing element from the right)
        int pivotIndex = -1;
        for (int i = n - 2; i >= 0; --i) {
            if (nums[i] < nums[i + 1]) {
                pivotIndex = i;
                break;
            }
        }
        
        // Step 2: If pivot is found, find the swap candidate and swap
        if (pivotIndex != -1) {
            // Find the smallest element in suffix strictly greater than nums[pivotIndex]
            int swapIndex = -1;
            for (int i = n - 1; i > pivotIndex; --i) {
                if (nums[i] > nums[pivotIndex]) {
                    swapIndex = i;
                    break;
                }
            }
            std::swap(nums[pivotIndex], nums[swapIndex]);
        }
        
        // Step 3: Reverse the suffix to get the smallest lexicographical arrangement
        std::reverse(nums.begin() + pivotIndex + 1, nums.end());
    }
};
```

---

## 8. Code Walkthrough

1. `int pivotIndex = -1;`
   - Initialized to `-1`. If the array is sorted in descending order, it will remain `-1`.
2. `for (int i = n - 2; i >= 0; --i)`
   - Starts scanning from the second-to-last element down to the beginning of the array.
3. `if (nums[i] < nums[i + 1]) { pivotIndex = i; break; }`
   - Finds the first pair where the left element is smaller than the right. That left index is our `pivotIndex`.
4. `if (pivotIndex != -1)`
   - If a valid pivot index is found (meaning the array is not entirely in descending order):
5. `for (int i = n - 1; i > pivotIndex; --i) { if (nums[i] > nums[pivotIndex]) { swapIndex = i; break; } }`
   - Scans from the end back to `pivotIndex`. Finds the first element strictly larger than `nums[pivotIndex]`.
6. `std::swap(nums[pivotIndex], nums[swapIndex]);`
   - Swaps the pivot and the swap target.
7. `std::reverse(nums.begin() + pivotIndex + 1, nums.end());`
   - Reverses the subarray to the right of `pivotIndex`. If `pivotIndex == -1`, it reverses the entire array (`nums.begin()` to `nums.end()`), which resets the descending array back to ascending order.

---

## 9. Dry Run

Let's trace `nums = [2, 3, 6, 5, 4, 1]`.
$n = 6$.
- **Step 1: Find the pivot index `pivotIndex`**
  - Compare `nums[4]` (4) and `nums[5]` (1): $4 > 1$
  - Compare `nums[3]` (5) and `nums[4]` (4): $5 > 4$
  - Compare `nums[2]` (6) and `nums[3]` (5): $6 > 5$
  - Compare `nums[1]` (3) and `nums[2]` (6): $3 < 6$ (Ascending! Pivot index = 1, pivot value = 3).
- **Step 2: Find swap candidate `swapIndex`**
  Scan from right to left ($i$ from 5 down to 2) for an element $> 3$:
  - `nums[5] = 1` ($\le 3$)
  - `nums[4] = 4` ($> 3$). Swap index = 4.
  - Swap `nums[1]` and `nums[4]`. Array becomes: `[2, 4, 6, 5, 3, 1]`.
- **Step 3: Reverse suffix `nums[pivotIndex + 1 ... n-1]`**
  - Pivot index + 1 = 2. Suffix is `[6, 5, 3, 1]`.
  - Reverse suffix: `[1, 3, 5, 6]`.
  - Array becomes: `[2, 4, 1, 3, 5, 6]`.

| Step | Array State | Variables |
| :--- | :--- | :--- |
| **Initial** | `[2, 3, 6, 5, 4, 1]` | `n=6`, `pivotIndex=-1` |
| **Find Pivot** | `[2, 3, 6, 5, 4, 1]` | `pivotIndex=1` (value 3) |
| **Find Swap** | `[2, 3, 6, 5, 4, 1]` | `swapIndex=4` (value 4) |
| **Swap** | `[2, 4, 6, 5, 3, 1]` | Swapped indices 1 and 4 |
| **Reverse** | `[2, 4, 1, 3, 5, 6]` | Suffix `nums[2...5]` reversed |

**Final Output:** `[2, 4, 1, 3, 5, 6]`.

---

## 10. Complexity Analysis

| Metric | Complexity | Reasoning |
| :--- | :--- | :--- |
| **Time Complexity** | $O(n)$ | In the worst case, we scan the array at most three times: once to find the pivot index, once to find the swap candidate, and once to reverse the suffix. Each pass is linear. |
| **Space Complexity** | $O(1)$ | No extra memory is allocated. All operations are done in-place. |

---

## 11. Common Mistakes

1. **Incorrect Search Direction**:
   - Searching for the pivot from left to right instead of right to left. Left-to-right scans do not isolate the rightmost descending suffix.
2. **Improper Comparison for Swap Candidate**:
   - Using `>=` instead of `>` when looking for the swap target. We need a *strictly* greater element to ensure the permutation is strictly larger.
3. **Failing to Handle Completely Descending Array**:
   - If the array is `[3, 2, 1]`, `pivotIndex` remains `-1`. If not handled correctly, the swap check might try to access index `-1`. Always check `pivotIndex != -1` before swapping.

---

## 12. Edge Cases

| Edge Case | Input | Expected Output | Why It Matters |
| :--- | :--- | :--- | :--- |
| Already largest permutation | `[3, 2, 1]` | `[1, 2, 3]` | Tests fallback to reverse entire array |
| Already smallest permutation | `[1, 2, 3]` | `[1, 3, 2]` | Simple single swap and reverse |
| Duplicate elements | `[1, 5, 1]` | `[5, 1, 1]` | Tests strict inequalities |
| Two elements | `[1, 2]` | `[2, 1]` | Minimal size check |

---

## 13. Interview Explanation

*"To find the next lexicographically greater permutation in-place, we can analyze the array from right to left.
First, we look for the first element from the right that decreases compared to its next element. Let's call this the break-point or pivot. Suffix to the right of this pivot is in descending order, meaning it is at its maximum permutation.
If no pivot is found, the entire array is in descending order, so we reverse it to return the smallest permutation.
If a pivot is found, we scan the suffix from the right to find the first element that is strictly greater than our pivot. We swap these two elements.
Finally, we reverse the suffix to make it sorted in ascending order, which minimizes its value and gives us the next lexicographically greater permutation. This runs in $O(n)$ time and $O(1)$ space."*

---

## 14. Follow-up Questions

1. **How does this relate to C++ STL `std::next_permutation`?**
   - *`std::next_permutation` uses this exact algorithm under the hood to modify the container and returns `true` if a next permutation was found, or `false` if the container reset to the sorted ascending state.*
2. **Can we implement a `prev_permutation` algorithm?**
   - *Yes, the steps are symmetric: find the first ascending element from the right (`nums[i] > nums[i+1]`), swap it with the largest element in the suffix smaller than it, and then reverse the suffix.*

---

## 15. Variations

1. **Previous Permutation With One Swap (LeetCode 1053)**: Find the lexicographically largest permutation smaller than the current one by swapping exactly one pair.
2. **Permutation Sequence (LeetCode 60)**: Find the $k$-th permutation of a sequence from $1$ to $n$ (uses mathematical factorial division).

---

## 16. Revision Notes

- **Step 1**: Find rightmost dip: `nums[i] < nums[i+1]`.
- **Step 2**: Swap with rightmost element $> nums[i]$.
- **Step 3**: Reverse suffix from `i + 1` to end.
- **Mnemonic**: "Find dip, swap with next larger from right, reverse tail."

---

## 17. Memorization Version

```cpp
// Optimal Next Permutation
void nextPermutation(vector<int>& nums) {
    int n = nums.size(), i = n - 2;
    while (i >= 0 && nums[i] >= nums[i + 1]) i--;
    if (i >= 0) {
        int j = n - 1;
        while (nums[j] <= nums[i]) j--;
        swap(nums[i], nums[j]);
    }
    reverse(nums.begin() + i + 1, nums.end());
}
```
*Time: $O(n)$, Space: $O(1)$*
