# Longest Subarray with Equal 0s and 1s

## 1. Problem Statement

Given a binary array `nums` consisting of only `0`s and `1`s, find the maximum length of a contiguous subarray that contains an equal number of `0`s and `1`s.

### Examples
- **Input:** `nums = [0, 1]`
- **Output:** 2
  *(Explanation: `[0, 1]` is the longest subarray with equal 0s and 1s, which has a length of 2.)*

- **Input:** `nums = [0, 1, 0]`
- **Output:** 2
  *(Explanation: Both `[0, 1]` (length 2) and `[1, 0]` (length 2) have equal 0s and 1s. The maximum length is 2.)*

- **Input:** `nums = [0, 0, 1, 1, 0, 1, 0]`
- **Output:** 6
  *(Explanation: The subarray `nums[0..5]` which is `[0, 0, 1, 1, 0, 1]` contains three 0s and three 1s. Its length is 6.)*

### Constraints
- $1 \le N \le 10^5$
- `nums[i]` is either `0` or `1`.

---

## 2. Interview Clarification Questions

Before writing any code, a candidate should clarify:
1. **Q:** What should we return if the array has length less than 2, or if there is no subarray with equal 0s and 1s?
   * **A:** Return `0`.
2. **Q:** Can we modify the input array during the process?
   * **A:** Yes, or we can just treat the zeros as -1s on-the-fly without modifying the array.
3. **Q:** Is there any constraint on the space complexity?
   * **A:** An optimal time complexity of $O(N)$ is expected. Auxiliary space of $O(N)$ is acceptable.

---

## 3. Thinking Process

### Step 1: First Instinct (Brute Force)
The simplest way is to check every possible subarray:
- Loop `i` from $0$ to $N-1$ representing the start.
- Loop `j` from `i` to $N-1$ representing the end.
- Count the number of `0`s and `1`s in `nums[i..j]`.
- If `zeros == ones`, update `maxLength = max(maxLength, j - i + 1)`.
- This approach takes $O(N^2)$ time by keeping track of the counts using variables instead of recounting.

### Step 2: The Mathematical Transformation (Key Insight)
How do we find a relationship where "equal counts of 0s and 1s" becomes a solvable subarray problem?
- Let's replace every `0` with `-1` (either conceptually or by modifying the array).
- In this transformed array:
  - Every `1` adds $+1$ to the sum.
  - Every `0` (now `-1`) adds $-1$ to the sum.
- If a subarray has equal counts of `0`s and `1`s, the sum of its elements in the transformed array will be:
  $$\text{Sum} = (\text{number of 1s}) \times 1 + (\text{number of 0s}) \times (-1) = \text{ones} - \text{zeros}$$
- Since `ones == zeros`, the sum is:
  $$\text{Sum} = 0$$
- **Therefore, the problem of finding the longest subarray with equal 0s and 1s is identical to finding the longest subarray with sum = 0 in the transformed array.**

### Step 3: Solving the Longest Subarray with Sum 0
To find the longest subarray with sum 0:
- Let the prefix sum up to index `i` be `runningSum`.
- If `runningSum` at index `j` is equal to `runningSum` at index `i` (where `i < j`), then:
  $$\text{runningSum}[j] - \text{runningSum}[i] = 0 \implies \text{subarray } nums[i+1..j] \text{ has sum } 0$$
- To maximize the length of this subarray (`j - i`), we want `i` (the first occurrence index of this prefix sum) to be as small as possible.
- Thus, we use a hash map `firstOccurrence` to store the *first index* where each prefix sum is encountered.
- If we see a prefix sum again, we calculate the length: `length = current_index - firstOccurrence[prefixSum]`. We update `maxLength` with this value.
- We do **not** update the index in the map if the prefix sum is already present, because we want to keep the oldest (smallest) index to maximize the length.

---

## 4. Pattern Recognition

This problem fits the **Transformation + Prefix Sum with Hash Map** pattern.
- **Why?**
  - **Transformation:** Translating categorical features (`0` and `1`) into numeric values (`-1` and `1`) simplifies counting conditions into algebraic sums.
  - **Prefix Sum + Hash Map:** Finding the *longest* subarray with a target sum (0) requires tracking the *earliest index* where each prefix sum was seen.

---

## 5. Brute Force Solution

### Algorithm
1. Initialize `maxLength = 0`.
2. Iterate `i` from $0$ to $N-1$:
   - Maintain `zeros = 0` and `ones = 0`.
   - Iterate `j` from `i` to $N-1$:
     - If `nums[j] == 0`, increment `zeros`, else increment `ones`.
     - If `zeros == ones`, update `maxLength = max(maxLength, j - i + 1)`.
3. Return `maxLength`.

### Dry Run
Input: `nums = [0, 1, 0]`
- `i = 0`:
  - `j = 0`: `nums[0] = 0` -> `zeros = 1`, `ones = 0`
  - `j = 1`: `nums[1] = 1` -> `zeros = 1`, `ones = 1`. Equal! `maxLength = max(0, 1 - 0 + 1) = 2`.
  - `j = 2`: `nums[2] = 0` -> `zeros = 2`, `ones = 1`
- `i = 1`:
  - `j = 1`: `nums[1] = 1` -> `zeros = 0`, `ones = 1`
  - `j = 2`: `nums[2] = 0` -> `zeros = 1`, `ones = 1`. Equal! `maxLength = max(2, 2 - 1 + 1) = 2`.
- `i = 2`:
  - `j = 2`: `nums[2] = 0` -> `zeros = 1`, `ones = 0`
Output: `2`

### Complexity
- **Time Complexity:** $O(N^2)$ due to nested loops.
- **Space Complexity:** $O(1)$ auxiliary space.

### Pros and Cons
- **Pros:** Easy to write, no extra space.
- **Cons:** TLE (Time Limit Exceeded) for $N = 10^5$.

### C++17 Code
```cpp
#include <vector>
#include <algorithm>

int findMaxLengthBrute(const std::vector<int>& nums) {
    int n = nums.size();
    int maxLength = 0;
    
    for (int i = 0; i < n; ++i) {
        int zeros = 0, ones = 0;
        for (int j = i; j < n; ++j) {
            if (nums[j] == 0) {
                zeros++;
            } else {
                ones++;
            }
            if (zeros == ones) {
                maxLength = std::max(maxLength, j - i + 1);
            }
        }
    }
    return maxLength;
}
```

---

## 6. Better Solution

This problem does not have a distinct intermediate "better" solution. The brute force solution is $O(N^2)$ and the hash map solution is $O(N)$. We jump directly to the optimal solution.

---

## 7. Optimal Solution (Prefix Sum + Hash Map)

### Algorithm
1. Initialize a hash map `firstOccurrence` to store the earliest index of each prefix sum.
2. Seed the map with prefix sum `0` at index `-1`: `firstOccurrence[0] = -1`. This ensures that if the prefix sum becomes `0` at index `i`, the calculated length is `i - (-1) = i + 1`, which is correct.
3. Initialize `runningSum = 0` and `maxLength = 0`.
4. Traverse the array from `i = 0` to `N - 1`:
   - If `nums[i] == 0`, add `-1` to `runningSum`.
   - Else (if `nums[i] == 1`), add `+1` to `runningSum`.
   - Check if `runningSum` is in `firstOccurrence`:
     - If yes, a subarray with sum 0 exists. Calculate length as `i - firstOccurrence[runningSum]`. Update `maxLength`.
     - If no, add `runningSum` and its index `i` to the map: `firstOccurrence[runningSum] = i`.
5. Return `maxLength`.

### C++17 Code

```cpp
#include <vector>
#include <unordered_map>
#include <algorithm>

int findMaxLength(const std::vector<int>& nums) {
    int n = nums.size();
    
    // Key: prefix sum, Value: earliest index where it occurred
    std::unordered_map<int, int> firstOccurrence;
    
    // Base case: prefix sum 0 occurs at index -1
    firstOccurrence[0] = -1;
    
    int runningSum = 0;
    int maxLength = 0;
    
    for (int i = 0; i < n; ++i) {
        // Conceptually convert 0 to -1 on-the-fly
        runningSum += (nums[i] == 0) ? -1 : 1;
        
        if (firstOccurrence.find(runningSum) != firstOccurrence.end()) {
            // Calculate length of the zero-sum subarray
            int currentLength = i - firstOccurrence[runningSum];
            maxLength = std::max(maxLength, currentLength);
        } else {
            // Store the first occurrence index only
            firstOccurrence[runningSum] = i;
        }
    }
    
    return maxLength;
}
```

---

## 8. Code Walkthrough

1. **Map Setup:** `std::unordered_map<int, int> firstOccurrence;` maps each prefix sum to the first index where it was encountered.
2. **Initial Seeding:** `firstOccurrence[0] = -1;` allows us to measure subarrays starting from index 0 correctly.
3. **On-the-fly Conversion:** `runningSum += (nums[i] == 0) ? -1 : 1;` processes `0` as `-1` without altering the input array `nums`.
4. **Map Check:**
   - If `firstOccurrence.find(runningSum) != firstOccurrence.end()` is true, it means we have seen this prefix sum before. The elements between that first occurrence and the current index sum to 0. We compute `currentLength = i - firstOccurrence[runningSum]` and update `maxLength`.
   - If not found, we insert `firstOccurrence[runningSum] = i`. We deliberately avoid updating the map if it was already found, to ensure we keep the leftmost possible index (maximizing `i - index`).

---

## 9. Dry Run

Input: `nums = [0, 0, 1, 1, 0]`

### Initialization:
- `firstOccurrence = {0: -1}`
- `runningSum = 0`, `maxLength = 0`

### Iterations:

| Index `i` | Element | Value Shift | `runningSum` | Map Lookup / Action | Updated Map | `maxLength` |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| `0` | `0` | `-1` | `-1` | Not found. Insert `{ -1 : 0 }` | `{0:-1, -1:0}` | `0` |
| `1` | `0` | `-1` | `-2` | Not found. Insert `{ -2 : 1 }` | `{0:-1, -1:0, -2:1}` | `0` |
| `2` | `1` | `+1` | `-1` | **Found `-1` at index `0`**.<br>`len = 2 - 0 = 2`. | `{0:-1, -1:0, -2:1}` | `max(0, 2) = 2` |
| `3` | `1` | `+1` | `0` | **Found `0` at index `-1`**.<br>`len = 3 - (-1) = 4`. | `{0:-1, -1:0, -2:1}` | `max(2, 4) = 4` |
| `4` | `0` | `-1` | `-1` | **Found `-1` at index `0`**.<br>`len = 4 - 0 = 4`. | `{0:-1, -1:0, -2:1}` | `max(4, 4) = 4` |

Output: `4` (Subarray `nums[0..3]` which is `[0, 0, 1, 1]`)

---

## 10. Complexity Analysis

- **Time Complexity:** $O(N)$ on average. We perform a single loop through the array, and hash map operations (find and insert) take $O(1)$ time on average.
- **Space Complexity:** $O(N)$ since the hash map can grow to store at most $N$ unique prefix sums in the worst case (e.g., all 0s or all 1s).

---

## 11. Common Mistakes

1. **Incorrectly updating the hash map:** Updating the index of the prefix sum when it is seen again. This shrinks the window instead of maximizing it.
   * *Correct:* Only write to the hash map if the prefix sum does not already exist.
2. **Missing the base case `firstOccurrence[0] = -1`:** If omitted, subarrays starting from index `0` that sum to 0 will not be calculated correctly.
3. **Adding the indices instead of subtracting:** Calculating `i + firstOccurrence[runningSum]` instead of `i - firstOccurrence[runningSum]`.

---

## 12. Edge Cases

| Edge Case | Input Example | Why it matters | Correct Output |
| :--- | :--- | :--- | :--- |
| **All Zeros** | `[0, 0, 0]` | No `1`s, so no balanced subarray exists. | `0` |
| **All Ones** | `[1, 1, 1]` | No `0`s, so no balanced subarray exists. | `0` |
| **No elements** | `[]` | Empty array. | `0` |
| **Balanced Single Pair** | `[0, 1]` | Smallest possible valid array. | `2` |
| **Entire Array is Balanced** | `[1, 0, 0, 1]` | Correctly maps back to prefix sum 0 at the end. | `4` |

---

## 13. Interview Explanation

"To find the longest subarray with equal numbers of `0`s and `1`s, we can transform the problem. If we replace all `0`s with `-1`s, then a subarray with an equal number of `0`s and `1`s will sum up to exactly $0$. 

Thus, the problem reduces to finding the longest subarray with a sum of $0$. We can solve this in $O(N)$ time using a prefix sum and a hash map. We traverse the array while maintaining a running prefix sum (adding $+1$ for a `1` and $-1$ for a `0`). We store each prefix sum and its earliest occurrence index in a hash map. If we encounter a prefix sum that is already in the map, it means the subarray between its first occurrence and our current index sums to $0$. We compute the length of this subarray and update our maximum length. To handle subarrays starting at index `0`, we seed the map with prefix sum `0` at index `-1`."

---

## 14. Follow-up Questions

1. **Q: How can we optimize space if the array contains only a few alternating elements?**
   * *A:* The space complexity is bounded by the number of unique prefix sums. In the best case (perfectly alternating), the prefix sum oscillates between `0` and `-1`, so the map size remains $O(1)$. In the worst case, it is $O(N)$.
2. **Q: Can we solve this problem using a sliding window?**
   * *A:* No. A sliding window works when the target condition is monotonic (e.g., all positive numbers where expanding the window always increases the sum). Here, because we introduce negative values (`-1`), the sum fluctuates up and down, so we cannot make greedy decisions about when to shrink the window.

---

## 15. Variations

1. **Longest Subarray with Equal number of Category A, B, and C elements:**
   - *Change:* With three categories, we need two running differences (e.g., $Count(A) - Count(B)$ and $Count(B) - Count(C)$). We store the pair of differences in a hash map as the key.
2. **Count of Subarrays with Equal 0s and 1s:**
   - *Change:* Instead of storing the *earliest* index in the map, we store the *frequency* of each prefix sum. When a prefix sum is seen, we add its current frequency to our total count.

---

## 16. Revision Notes

- **Transformation trick:** `0` $\to$ `-1`.
- **Target condition:** Subarray sum = 0.
- **Hash Map Strategy:** Store the *first* occurrence of each prefix sum to maximize length: `i - first_index`.
- **Base state:** `map[0] = -1`.

---

## 17. Memorization Version

```cpp
// 1-min Review Version
int findMaxLength(vector<int>& nums) {
    unordered_map<int, int> mp;
    mp[0] = -1;
    int sum = 0, max_len = 0;
    for (int i = 0; i < nums.size(); ++i) {
        sum += (nums[i] == 0) ? -1 : 1;
        if (mp.count(sum)) {
            max_len = max(max_len, i - mp[sum]);
        } else {
            mp[sum] = i;
        }
    }
    return max_len;
}
```
*Complexity:* $O(N)$ Time, $O(N)$ Space.
