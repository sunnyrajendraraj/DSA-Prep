# Find Pair with Given Sum (Two Sum)

---

## 1. Problem Statement

Given an array of integers `nums` and an integer `target`, return indices of the two numbers such that they add up to `target`.

You may assume that each input would have ***exactly* one solution**, and you may not use the *same* element twice. You can return the answer in any order.

### Input
- An integer array `nums` of size `n` ($2 \le n \le 10^4$).
- An integer `target` ($-10^9 \le target \le 10^9$).

### Output
- A vector of two integers containing the indices of the two elements that sum to `target`.

### Constraints
- $2 \le n \le 10^4$
- $-10^9 \le nums[i] \le 10^9$
- $-10^9 \le target \le 10^9$
- Only one valid answer exists.

### Observations
1. **Uniqueness:** Only one pair satisfies the condition.
2. **Index vs. Value:** If we need to return indices, sorting the array directly will destroy the original index information. If we sort, we must track indices.
3. **Complement:** For any number $x$, we need to check if the complement $target - x$ exists in the array.

---

## 2. Interview Clarification

Before writing any code, clarify these points with the interviewer:
1. **Can the array contain duplicates?**
   - *Answer:* Yes, but there is only one unique pair of indices that sums to the target.
2. **Are the indices 0-based or 1-based?**
   - *Answer:* Standard 0-based indexing.
3. **What should we return if no such pair exists?**
   - *Answer:* The problem guarantees exactly one solution, so you can assume a valid pair always exists.
4. **Can we use the same element twice?**
   - *Answer:* No, the two indices in the result must be distinct.

---

## 3. Thinking Process

Let's think aloud:
* **Initial thoughts:** We are looking for $nums[i] + nums[j] = target$ where $i \neq j$.
* **Brute Force:** Let's check every possible pair. For each element at index $i$, loop through all subsequent elements at index $j$ and check if $nums[i] + nums[j] == target$. If they match, return `{i, j}`. This uses $O(n^2)$ time and $O(1)$ space.
* **Better (Sorting + Two Pointers):** What if the array was sorted? We could use a two-pointer approach (one at the beginning, one at the end).
  - Let $L = 0$ and $R = n - 1$.
  - If $nums[L] + nums[R] == target$, we found it.
  - If $nums[L] + nums[R] < target$, we need a larger sum, so increment $L$.
  - If $nums[L] + nums[R] > target$, we need a smaller sum, so decrement $R$.
  - *Catch:* Sorting the array destroys the original indices. To return indices, we must store the values along with their original indices before sorting. This requires $O(n \log n)$ time and $O(n)$ space.
* **Optimal (Hash Map / One-pass Lookups):** Can we do it in $O(n)$ time?
  - When we are at element $nums[i]$, we need to find if $complement = target - nums[i]$ has been seen before.
  - A Hash Map (like `std::unordered_map` in C++) allows $O(1)$ average time complexity for insertions and lookups.
  - We can traverse the array, and for each element, check if its complement exists in the map. If it does, we return the stored index and the current index. If not, we store the current element and its index in the map.
  - This solves the problem in $O(n)$ time and $O(n)$ space in a single pass.

---

## 4. Pattern Recognition

This problem belongs to the **Hashing** and **Two Pointers** patterns.
* **Why this topic?** Lookups and matching complements are prime use cases for Hash Maps.
* **Which pattern?**
  - **Complement lookup using Hash Map:** Storing visited values to search in $O(1)$ time.
  - **Two Pointers (Sorted Arrays):** Shrinking search space from both ends.

---

## 5. Brute Force Solution

Check every pair of elements using nested loops.

### Algorithm
1. Loop $i$ from $0$ to $n-1$.
2. Loop $j$ from $i+1$ to $n-1$.
3. If $nums[i] + nums[j] == target$, return `{i, j}`.
4. If no pair is found, return `{}`.

### Dry Run
Input: `nums = [3, 2, 4]`, `target = 6`.
- $i = 0, j = 1$: $nums[0] + nums[1] = 3 + 2 = 5 \neq 6$.
- $i = 0, j = 2$: $nums[0] + nums[2] = 3 + 4 = 7 \neq 6$.
- $i = 1, j = 2$: $nums[1] + nums[2] = 2 + 4 = 6 == 6$. Match found! Return `{1, 2}`.

### Complexity Analysis
- **Time Complexity:** $O(n^2)$ — Two nested loops.
- **Space Complexity:** $O(1)$ — No extra space is allocated.

### Pros/Cons
- **Pros:** Extremely simple, no extra space.
- **Cons:** Inefficient for large arrays ($n = 10^5$ leads to $10^{10}$ operations).

### C++17 Code
```cpp
#include <vector>

class Solution {
public:
    std::vector<int> twoSum(std::vector<int>& nums, int target) {
        int n = nums.size();
        for (int i = 0; i < n; ++i) {
            for (int j = i + 1; j < n; ++j) {
                // Check if pair sums to target
                if (nums[i] + nums[j] == target) {
                    return {i, j};
                }
            }
        }
        return {}; // Fallback
    }
};
```

---

## 6. Better Solution

Sort the array and use the Two-Pointer technique. Since we need to return indices, we store the original index with each value.

### Algorithm
1. Store elements as pairs: `{nums[i], i}`.
2. Sort the pairs based on their values in ascending order.
3. Initialize two pointers: `left = 0`, `right = n - 1`.
4. While `left < right`:
   - Calculate current sum: `current_sum = pairs[left].first + pairs[right].first`.
   - If `current_sum == target`, return `{pairs[left].second, pairs[right].second}`.
   - If `current_sum < target`, increment `left`.
   - If `current_sum > target`, decrement `right`.

### Dry Run
Input: `nums = [3, 2, 4]`, `target = 6`.
1. Pair representation: `pairs = [ {3, 0}, {2, 1}, {4, 2} ]`.
2. Sorted: `pairs = [ {2, 1}, {3, 0}, {4, 2} ]`.
3. Pointers: `left = 0`, `right = 2`.
   - Step 1: `pairs[0].first + pairs[2].first` $\rightarrow 2 + 4 = 6$.
   - Since $6 == target$, return indices `{pairs[0].second, pairs[2].second}` $\rightarrow$ `{1, 2}`.

### Complexity Analysis
- **Time Complexity:** $O(n \log n)$ — Sorting takes $O(n \log n)$, and the two-pointer traversal takes $O(n)$.
- **Space Complexity:** $O(n)$ — Required to store the pairs of values and indices.

### Pros/Cons
- **Pros:** Can easily be adapted to return actual values without storing indices (reducing space complexity to $O(1)$ if the array is already sorted or if indices aren't needed).
- **Cons:** Worse time complexity than the hash map solution, and requires auxiliary storage to preserve original indices.

### C++17 Code
```cpp
#include <vector>
#include <algorithm>

class Solution {
public:
    std::vector<int> twoSum(std::vector<int>& nums, int target) {
        int n = nums.size();
        std::vector<std::pair<int, int>> pairs(n);
        
        // Save original indices
        for (int i = 0; i < n; ++i) {
            pairs[i] = {nums[i], i};
        }
        
        // Sort based on value
        std::sort(pairs.begin(), pairs.end());
        
        int left = 0;
        int right = n - 1;
        while (left < right) {
            int current_sum = pairs[left].first + pairs[right].first;
            if (current_sum == target) {
                return {pairs[left].second, pairs[right].second};
            } else if (current_sum < target) {
                left++;
            } else {
                right--;
            }
        }
        return {};
    }
};
```

---

## 7. Optimal Solution

Using an HashTable (unordered_map) to achieve linear time complexity.

### Algorithm
1. Initialize an empty hash map `seen` mapping values to their indices.
2. Traverse the array `nums` using index `i`:
   a. Compute the required value `complement = target - nums[i]`.
   b. Check if `complement` exists in `seen`.
   c. If it exists, return `{seen[complement], i}`.
   d. Otherwise, insert `nums[i]` and its index `i` into `seen`.

### Dry Run
Input: `nums = [2, 7, 11, 15]`, `target = 9`.

| Step | Current Value (`nums[i]`) | Complement (`target - nums[i]`) | Found in Map? | Action | Map State |
| :--- | :--- | :--- | :--- | :--- | :--- |
| 1 ($i=0$) | 2 | $9 - 2 = 7$ | No | Insert `{2, 0}` | `{2: 0}` |
| 2 ($i=1$) | 7 | $9 - 7 = 2$ | Yes (at index 0) | Return `{0, 1}` | — |

Result: `{0, 1}`.

### C++17 Code
```cpp
#include <vector>
#include <unordered_map>

class Solution {
public:
    std::vector<int> twoSum(std::vector<int>& nums, int target) {
        // Map to store: key = element value, value = element index
        std::unordered_map<int, int> seen;
        
        for (int i = 0; i < nums.size(); ++i) {
            int complement = target - nums[i];
            
            // Check if complement has been seen
            if (seen.find(complement) != seen.end()) {
                return {seen[complement], i};
            }
            
            // Store current number and index
            seen[nums[i]] = i;
        }
        
        return {}; // Guarantee of exactly one solution means this is unreachable
    }
};
```

---

## 8. Code Walkthrough

* `std::unordered_map<int, int> seen;`: Instantiates a hash map using a bucket-based hash table where lookups and insertions are $O(1)$ average time.
* `int complement = target - nums[i];`: For each number, we calculate what other number is needed to reach the target sum.
* `seen.find(complement)`: We search the hash map. If the complement is found, it means we have already visited the matching number.
* `return {seen[complement], i};`: Returns the index of the previously visited element and the current element's index.
* `seen[nums[i]] = i;`: If the complement is not found, we record the current number's index so that future elements can match with it.

---

## 9. Dry Run

Input: `nums = [3, 2, 4]`, `target = 6`.

| Loop Index `i` | Value `nums[i]` | Complement `6 - nums[i]` | Map Lookup | Map Insertion | Return Val |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **0** | 3 | 3 | Look for `3` $\rightarrow$ Not Found | Insert `{3: 0}` | — |
| **1** | 2 | 4 | Look for `4` $\rightarrow$ Not Found | Insert `{2: 1}` | — |
| **2** | 4 | 2 | Look for `2` $\rightarrow$ Found at `1` | — | `{1, 2}` |

---

## 10. Complexity Analysis

| Approach | Time Complexity | Space Complexity | Notes |
| :--- | :--- | :--- | :--- |
| **Brute Force** | $O(n^2)$ | $O(1)$ | No memory overhead, but slow. |
| **Better (Sorting)**| $O(n \log n)$ | $O(n)$ | Helpful if indices are not required (reduces space to $O(1)$). |
| **Optimal (HashMap)**| $O(n)$ | $O(n)$ | Fast lookups, requires dynamic memory. |

* **Time Complexity (Optimal):** $O(n)$ since we iterate through the array once and perform $O(1)$ average time hash map lookups.
* **Space Complexity (Optimal):** $O(n)$ because we store up to $n$ elements in the hash map in the worst case (when the match is at the very end).

---

## 11. Common Mistakes

1. **Re-using the same element:** For example, if `nums = [3, 2, 4]`, `target = 6`, returning `{0, 0}` because $3 + 3 = 6$. The problem statement forbids using the same element twice.
2. **Sorting without tracking indices:** Sorting the array directly and returning the new index values.
3. **Handling duplicate values in the map:** Writing `seen[nums[i]] = i` before doing the lookup can cause a number to match with itself if the complement equals the number (e.g. target is 6, current number is 3). The check must always happen *before* the insertion.

---

## 12. Edge Cases

| Edge Case | Input Example | Expected Output | Why it matters |
| :--- | :--- | :--- | :--- |
| **Target includes negative values** | `nums=[-3,4,3,90]`, `target=0` | `{0, 2}` | Ensures addition handles negative signs. |
| **Duplicates sum to target** | `nums=[3,3]`, `target=6` | `{0, 1}` | Map should handle duplicate values properly. |
| **First and last elements match** | `nums=[1,5,8,9]`, `target=10` | `{0, 3}` | Pointers/traversals must traverse fully. |

---

## 13. Interview Explanation

"To solve the Two Sum problem, the goal is to find two numbers that sum to a given target.
A brute-force solution uses two nested loops to check every pair, which takes $O(n^2)$ time.
We can improve this by using a Hash Map. As we iterate through the array, we check if the complement, which is `target - nums[i]`, has already been visited and stored in our map. If it has, we immediately return the index of the complement and the current index. If it hasn't, we insert the current element and its index into the map.
This allows us to solve the problem in a single pass with $O(n)$ time complexity and $O(n)$ space complexity."

---

## 14. Follow-up Questions

1. **How does the complexity change if the array is already sorted?**
   * *Answer:* If the array is sorted, the Two-Pointer approach becomes optimal, reducing space complexity to $O(1)$ while keeping time complexity at $O(n)$.
2. **What if the hash map has collisions?**
   * *Answer:* In C++, `std::unordered_map` handles collisions using chaining (linked lists or balanced trees). While collision resolution keeps average operations at $O(1)$, the worst-case time complexity could degrade to $O(n)$ if all elements hash to the same bucket.

---

## 15. Variations

1. **Two Sum II (Input array is sorted):** Uses $O(1)$ space with Two Pointers.
2. **Two Sum III (Data structure design):** Design a class that supports `add` and `find` operations.
3. **Two Sum IV (Input is a BST):** Find pairs in a Binary Search Tree (uses DFS + Hash Set or Inorder traversal + Two Pointers).

---

## 16. Revision Notes

* **Core Formula:** $A + B = Target \implies B = Target - A$.
* **Data Structure:** Use `std::unordered_map` for average $O(1)$ lookups.
* **Important Order:** Lookup *before* insertion to avoid matching an element with itself.

---

## 17. Memorization Version

```cpp
// Target sum complement lookup
unordered_map<int, int> seen;
for (int i = 0; i < nums.size(); ++i) {
    int comp = target - nums[i];
    if (seen.count(comp)) return {seen[comp], i};
    seen[nums[i]] = i;
}
return {};
```
* **Time:** $O(n)$, **Space:** $O(n)$.
