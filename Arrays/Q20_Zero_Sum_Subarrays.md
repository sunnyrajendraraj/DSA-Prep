# Find All Subarrays with Zero Sum

## 1. Problem Statement

Given an array of integers `nums` of size $N$, find all continuous subarrays that sum up to `0`. 
Specifically, you need to:
1. **Count** the total number of subarrays whose sum is $0$.
2. **Find and list** the start and end indices (0-indexed, inclusive) of all such subarrays.

### Examples
- **Input:** `nums = [3, 4, -7, 3, 1, 3, 1, -4, -2, -2]`
- **Output:**
  - Count: 6
  - Subarrays: `[0, 2]`, `[1, 3]`, `[2, 7]`, `[3, 9]`, `[5, 8]`, `[6, 9]`
  *(Explanation: `nums[0..2] = 3+4-7 = 0`, `nums[1..3] = 4-7+3 = 0`, etc.)*

### Constraints
- $1 \le N \le 10^5$
- $-10^5 \le nums[i] \le 10^5$
- The total number of zero-sum subarrays can be up to $O(N^2)$ in the worst case (e.g., all zeros), so the output format should be optimized.

---

## 2. Interview Clarification Questions

Before writing any code, a candidate should clarify:
1. **Q:** What should we return if no such subarray exists?
   * **A:** Return count as `0` and an empty list of index pairs.
2. **Q:** Can the array contain zeros?
   * **A:** Yes, a single zero is itself a valid subarray of sum 0. For example, `[0]` has indices `[0, 0]`.
3. **Q:** Is the time complexity limit strict? 
   * **A:** Yes, the search algorithm must be $O(N)$ in terms of finding the counts, though printing/returning indices will take $O(N + K)$ where $K$ is the number of zero-sum subarrays.
4. **Q:** Should the pairs of indices be returned in any particular order?
   * **A:** Usually sorted by start index, then end index, but any order is acceptable as long as all are listed.

---

## 3. Thinking Process

### Step 1: First Instinct (Brute Force)
The simplest way to find all subarrays is to inspect all possible pairs of start indices `i` and end indices `j` (where `i <= j`).
- Loop `i` from $0$ to $N-1$.
- Loop `j` from `i` to $N-1$.
- Compute the sum of the subarray `nums[i..j]`.
- If the sum is $0$, increment the count and store the pair `[i, j]`.
- To avoid computing the sum from scratch (which would take $O(N^3)$), we can maintain a running sum inside the inner loop, bringing it down to $O(N^2)$ time and $O(1)$ space.

### Step 2: Optimizing with Prefix Sums
Can we do better than $O(N^2)$?
Let the prefix sum up to index $i$ be $P(i) = \sum_{k=0}^{i} nums[k]$.
The sum of a subarray from index $i+1$ to $j$ is given by:
$$\text{Sum}(i+1, j) = P(j) - P(i)$$
If the subarray sum is $0$, then:
$$P(j) - P(i) = 0 \implies P(j) = P(i)$$
This is a critical observation: **If two prefix sums are equal, the sum of the elements between them is zero.**
Additionally, if any prefix sum $P(j) = 0$, it means the subarray from index $0$ to $j$ has a sum of $0$. This is equivalent to saying $P(-1) = 0$ is implicitly present.

### Step 3: Designing the Hash Map Solution
To find equal prefix sums efficiently:
- We can traverse the array and compute the prefix sum `sum` at each index `i`.
- We maintain a hash map that stores the prefix sums we have seen so far, mapping each prefix sum to a list of indices where it occurred.
  - `std::unordered_map<long long, std::vector<int>> prefixMap;`
- We also seed the map with `prefixMap[0] = {-1}`. This handles the case where the prefix sum itself becomes $0$ at index $i$, indicating a zero-sum subarray starting at index $0$ (since $i - (-1) = i + 1$ elements).
- For each index `i`:
  - Calculate `sum += nums[i]`.
  - If `sum` exists in our map, then for every index `prevIdx` stored in `prefixMap[sum]`:
    - The subarray `nums[prevIdx + 1 ... i]` has a sum of $0$.
    - We record the pair `[prevIdx + 1, i]` and increment our count.
  - Append `i` to `prefixMap[sum]`.

---

## 4. Pattern Recognition

This problem fits the **Prefix Sum + Hash Map** pattern.
- **Why?** We are looking for a subarray matching a target sum (in this case, target = 0). Whenever a problem asks for subarrays with a target sum, storing prefix sums in a hash map allows us to find matching subarrays in $O(1)$ lookup time instead of scanning.

---

## 5. Brute Force Solution

### Algorithm
1. Initialize `count = 0` and a list of index pairs `subarrays`.
2. Run a loop with variable `i` from $0$ to $N-1$ representing the start of the subarray.
3. Inside, run another loop with variable `j` from `i` to $N-1$ representing the end of the subarray.
4. Keep a running sum `currentSum` initialized to $0$ before the inner loop starts. Add `nums[j]` to `currentSum` in each iteration.
5. If `currentSum == 0`, increment `count` and add `{i, j}` to `subarrays`.

### Dry Run
Input: `nums = [1, -1, 0]`

- `i = 0`:
  - `j = 0`: `currentSum = 1`
  - `j = 1`: `currentSum = 1 + (-1) = 0`. Match! `subarrays = {{0, 1}}`, `count = 1`
  - `j = 2`: `currentSum = 0 + 0 = 0`. Match! `subarrays = {{0, 1}, {0, 2}}`, `count = 2`
- `i = 1`:
  - `j = 1`: `currentSum = -1`
  - `j = 2`: `currentSum = -1 + 0 = -1`
- `i = 2`:
  - `j = 2`: `currentSum = 0`. Match! `subarrays = {{0, 1}, {0, 2}, {2, 2}}`, `count = 3`

### Complexity
- **Time Complexity:** $O(N^2)$ because of the nested loops.
- **Space Complexity:** $O(1)$ if we do not count the output storage, or $O(K)$ where $K$ is the number of subarrays found.

### Pros and Cons
- **Pros:** Simple, requires no extra memory.
- **Cons:** Too slow for large arrays ($N \ge 10^5$).

### C++17 Code
```cpp
#include <vector>
#include <utility>

std::pair<int, std::vector<std::pair<int, int>>> findAllZeroSumSubarraysBrute(const std::vector<int>& nums) {
    int count = 0;
    std::vector<std::pair<int, int>> subarrays;
    int n = nums.size();
    
    for (int i = 0; i < n; ++i) {
        long long currentSum = 0;
        for (int j = i; j < n; ++j) {
            currentSum += nums[j];
            if (currentSum == 0) {
                count++;
                subarrays.push_back({i, j});
            }
        }
    }
    return {count, subarrays};
}
```

---

## 6. Better Solution

There is no intermediate "better" solution for this problem. The leap goes directly from the $O(N^2)$ brute force to the $O(N)$ prefix sum hash map optimal approach.

---

## 7. Optimal Solution (Prefix Sum + Hash Map)

### Algorithm
1. Create a hash map `prefixMap` that maps a prefix sum (`long long`) to a vector of indices (`std::vector<int>`).
2. Insert prefix sum `0` at index `-1`: `prefixMap[0].push_back(-1)`.
3. Initialize `sum = 0`, `count = 0`, and a result vector of pairs `subarrays`.
4. Loop through the array from `i = 0` to `n - 1`:
   - Add `nums[i]` to `sum`.
   - Check if `sum` is already present in `prefixMap`.
   - If it is, iterate through all indices stored in `prefixMap[sum]`. For each index `idx`:
     - Push `{idx + 1, i}` to `subarrays`.
     - Increment `count`.
   - Push `i` to `prefixMap[sum]`.
5. Return `{count, subarrays}`.

### C++17 Code

```cpp
#include <vector>
#include <unordered_map>
#include <utility>

std::pair<int, std::vector<std::pair<int, int>>> findAllZeroSumSubarrays(const std::vector<int>& nums) {
    int count = 0;
    std::vector<std::pair<int, int>> subarrays;
    int n = nums.size();
    
    // Maps prefix sum to list of indices where it occurred
    std::unordered_map<long long, std::vector<int>> prefixMap;
    
    // Seed the map with prefix sum 0 at index -1
    prefixMap[0].push_back(-1);
    
    long long runningSum = 0;
    for (int i = 0; i < n; ++i) {
        runningSum += nums[i];
        
        // If this prefix sum has been seen before, we found zero-sum subarrays
        if (prefixMap.find(runningSum) != prefixMap.end()) {
            const auto& indices = prefixMap[runningSum];
            for (int prevIdx : indices) {
                subarrays.push_back({prevIdx + 1, i});
                count++;
            }
        }
        // Record current index for this prefix sum
        prefixMap[runningSum].push_back(i);
    }
    
    return {count, subarrays};
}
```

---

## 8. Code Walkthrough

1. **Hash Map Declaration:** `std::unordered_map<long long, std::vector<int>> prefixMap;` maps each prefix sum to all indices where it was generated.
2. **Base Case Insertion:** `prefixMap[0].push_back(-1);` handles subarrays starting at index `0`. If the prefix sum is `0` at index `i`, the subarray starts at `-1 + 1 = 0`.
3. **Iterative Summation:** Inside the loop, `runningSum += nums[i];` computes the cumulative sum.
4. **Subarray Detection:** `prefixMap.find(runningSum)` checks if the exact same cumulative sum was recorded previously. 
5. **Inner Loop for Indices:** If a match is found, all prior occurrences are retrieved, and for each occurrence `prevIdx`, a new zero-sum subarray is recorded spanning from `prevIdx + 1` to `i`.
6. **Store current index:** Finally, the current index `i` is added to the map under `runningSum` so it can be paired with future indices.

---

## 9. Dry Run

Input: `nums = [3, 4, -7, 1]`

### Initialization:
- `prefixMap = {0: [-1]}`
- `runningSum = 0`, `count = 0`, `subarrays = []`

### Iterations:

| Index `i` | Element | `runningSum` | Map Lookup & Action | Updated Map | `subarrays` |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `0` | `3` | `3` | Not found. | `{0:[-1], 3:[0]}` | `[]` |
| `1` | `4` | `7` | Not found. | `{0:[-1], 3:[0], 7:[1]}` | `[]` |
| `2` | `-7` | `0` | **Found `0` at `-1`**.<br>Add subarray `[-1+1, 2] = [0, 2]`. | `{0:[-1, 2], 3:[0], 7:[1]}` | `[{0, 2}]` |
| `3` | `1` | `1` | Not found. | `{0:[-1, 2], 3:[0], 7:[1], 1:[3]}` | `[{0, 2}]` |

Final Count: 1. Subarrays: `[0, 2]`.

---

## 10. Complexity Analysis

- **Time Complexity:** 
  - **Average Case:** $O(N + K)$, where $N$ is the size of the array and $K$ is the number of zero-sum subarrays. The hash map lookups take $O(1)$ on average.
  - **Worst Case:** $O(N^2)$ if all elements are `0`. In this case, every prefix sum is `0`, and for each element, we loop through all previous indices. However, since the output size itself is $O(N^2)$, this is mathematically optimal because writing the output takes $O(K)$ time.
- **Space Complexity:** $O(N)$ for storing prefix sums in the hash map.

---

## 11. Common Mistakes

1. **Forgetting to seed the map with `{0, -1}`:** This is the most common bug. If you omit this, zero-sum subarrays starting at index `0` (like `[3, -3]`) will not be recognized.
2. **Using the wrong index bounds:** Storing `{prevIdx, i}` instead of `{prevIdx + 1, i}`. Remember, the subarray starts at the element *after* the matching prefix sum.
3. **Data Type Overflow:** Using a standard `int` for the running sum. If elements are large, `runningSum` can exceed 32-bit integer limits. Always use `long long`.

---

## 12. Edge Cases

| Edge Case | Input Example | Why it matters | Correct Output |
| :--- | :--- | :--- | :--- |
| **All Zeros** | `[0, 0, 0]` | Every single element and combination has sum 0. | Count: 6; Subarrays: `[0,0], [0,1], [0,2], [1,1], [1,2], [2,2]` |
| **No Zero-Sum Subarrays** | `[1, 2, 3]` | Running sum increases monotonically. | Count: 0; Subarrays: `[]` |
| **Negative and Positive Symmetry** | `[-1, 1, -1, 1]`| Alternating sums create multiple overlaps. | Count: 4; Subarrays: `[0,1], [1,2], [2,3], [0,3]` |
| **Entire Array is Zero-Sum** | `[1, 2, -3]` | Sum is zero at the very end. | Count: 1; Subarray: `[0, 2]` |

---

## 13. Interview Explanation

"To find all subarrays that sum to zero, a brute force search would check all $O(N^2)$ subarrays, taking $O(N^2)$ time. We can optimize this to $O(N)$ average time by utilizing the prefix sum technique combined with a hash map. 

If the prefix sum at index `i` is equal to the prefix sum at index `j` (where `i < j`), it implies that the sum of the elements between `i + 1` and `j` must be zero. Therefore, we traverse the array while maintaining a running prefix sum. We store every prefix sum and the index where it occurred in a hash map. Before storing, we check if the current prefix sum has been seen before. If it has, every previously recorded index for that sum marks the start of a zero-sum subarray that ends at our current index. We seed the hash map with prefix sum `0` at index `-1` to correctly capture subarrays starting at index `0`."

---

## 14. Follow-up Questions

1. **Q: How would you modify the solution if we only need to count the subarrays, and not return their indices?**
   * *A:* Instead of storing a vector of indices in the hash map, we only need to store the frequency (count) of each prefix sum: `std::unordered_map<long long, int> prefixFreq`. When we see a prefix sum again, we add its frequency to our count and then increment the frequency by 1. This uses less memory and runs in $O(N)$ time even in the worst case (e.g. all zeros).
2. **Q: Can we solve this in $O(1)$ auxiliary space?**
   * *A:* No, because finding zero-sum subarrays requires matching history. Without space, we must fall back to the $O(N^2)$ time brute force approach.

---

## 15. Variations

1. **Subarray Sum Equals K:** Find all subarrays that sum up to a target value `K`.
   - *Change:* Instead of checking if `runningSum` is in the map, check if `runningSum - K` exists in the map.
2. **Longest Subarray with Sum 0:** Find the length of the longest subarray that sums to zero.
   - *Change:* Instead of storing all indices in the map, only store the *first* occurrence of each prefix sum to maximize `i - firstOccurIdx`.

---

## 16. Revision Notes

- **Mathematical Condition:** Subarray `nums[i..j]` sums to $0$ $\iff$ PrefixSum[j] == PrefixSum[i - 1].
- **Map structure:** `unordered_map<long long, vector<int>>` mapping prefix sums to index vectors.
- **Base state:** Map `0` to index `-1`.
- **Complexity:** Time $O(N + K)$ where $K$ is output count, Space $O(N)$.

---

## 17. Memorization Version

```cpp
// 1-min Review Version: Count only
int countZeroSumSubarrays(vector<int>& nums) {
    unordered_map<long long, int> freq;
    freq[0] = 1;
    long long sum = 0;
    int count = 0;
    for (int num : nums) {
        sum += num;
        if (freq.find(sum) != freq.end()) {
            count += freq[sum];
        }
        freq[sum]++;
    }
    return count;
}
```
*Complexity:* $O(N)$ Time, $O(N)$ Space.
