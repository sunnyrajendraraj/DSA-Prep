# Subarray with Given Sum

---

## 1. Problem Statement

Given an unsorted array `arr` of non-negative integers and an integer `S`, find a contiguous subarray that adds up to `S`.

### Input
- An array of non-negative integers `arr` of size $N$.
- A target sum `S`.

### Output
- A vector containing the 1-based starting and ending indices of the first such subarray. If no such subarray exists, return `[-1]`.

### Constraints
- $1 \le N \le 10^5$
- $0 \le \text{arr}[i] \le 10^9$
- $0 \le S \le 10^9$

### Observations
- Since all elements are **non-negative**, the sum of a subarray increases monotonically as we add elements to its right, and decreases monotonically as we remove elements from its left.
- This monotonicity property is the key to achieving a linear time complexity using a sliding window.
- If the array contained **negative numbers**, this monotonicity would be lost, and we would have to use a Prefix Sum + Hash Map approach instead.

---

## 2. Interview Clarification

1. **Are elements strictly non-negative?**
   - *Answer:* Yes, elements are $\ge 0$.
2. **What if the target sum $S$ is 0?**
   - *Answer:* A subarray of size 0 isn't considered, but since elements are non-negative, a subarray containing a single element `0` can sum to `0`. Your solution should handle $S = 0$ correctly.
3. **If there are multiple subarrays, which one should be returned?**
   - *Answer:* Return the one that occurs first (i.e., has the smallest starting index).
4. **Are the returned indices 0-based or 1-based?**
   - *Answer:* Return 1-based indices. If no subarray is found, return `[-1]`.
5. **What if we have negative numbers in the array?**
   - *Answer:* Clarify this scenario. If negatives are present, we cannot use two pointers because decreasing the window size might increase the sum, and increasing the window size might decrease the sum. We will address both cases.

---

## 3. Thinking Process

1. **First Instinct (Brute Force):**
   - Check every possible subarray. Use two nested loops: the outer loop chooses the starting index `i`, and the inner loop calculates the sum by expanding the ending index `j`.
   - If the sum equals `S`, return `[i+1, j+1]`.
   - Time Complexity: $O(N^2)$, Space Complexity: $O(1)$. Fails for $N = 10^5$.

2. **Exploiting Non-Negativity (Sliding Window):**
   - Since all elements are non-negative, we can maintain a sliding window `[left, right]`.
   - If `currentSum < S`, we expand the window by moving `right` to the right and adding `arr[right]` to `currentSum`.
   - If `currentSum > S`, we shrink the window by moving `left` to the right and subtracting `arr[left]` from `currentSum`.
   - If `currentSum == S`, we return the 1-based indices `[left+1, right+1]`.
   - This approach only moves `left` and `right` from $0$ to $N-1$, which takes $O(N)$ time and $O(1)$ space.

3. **Handling Negatives (Prefix Sum + Hash Map):**
   - If the array has negative numbers, we need a different approach. Let `prefixSum[i]` be the sum of elements from $0$ to $i$.
   - The sum of subarray `arr[j...i]` is `prefixSum[i] - prefixSum[j-1]`.
   - We want `prefixSum[i] - prefixSum[j-1] = S`, which means `prefixSum[j-1] = prefixSum[i] - S`.
   - As we traverse the array, we store `prefixSum[i]` and its index `i` in a Hash Map. At each step `i`, we check if `prefixSum[i] - S` exists in our map. If it does, we found the subarray.
   - This takes $O(N)$ time and $O(N)$ space.

---

## 4. Pattern Recognition

- **Pattern:** Sliding Window / Two Pointers (when array contains only non-negative integers).
- **Pattern:** Prefix Sum + Hash Map (when negative numbers are allowed).
- **Why this topic?** Finding a contiguous subarray with a sum constraint is a classic windowing or prefix-difference search problem.

---

## 5. Brute Force Solution

### Algorithm
1. Loop `i` from `0` to $N-1$.
2. Initialize `currentSum = 0`.
3. Loop `j` from `i` to $N-1$:
   - `currentSum += arr[j]`
   - If `currentSum == S`, return `[i + 1, j + 1]`.
   - If `currentSum > S`, break the inner loop (since elements are non-negative, sum will only increase further).
4. If no subarray is found, return `[-1]`.

### Dry Run
Input: `arr = [1, 2, 3, 7, 5]`, `S = 12`
- `i = 0`:
  - `j = 0`: `sum = 1`
  - `j = 1`: `sum = 3`
  - `j = 2`: `sum = 6`
  - `j = 3`: `sum = 13` (> 12, break)
- `i = 1`:
  - `j = 1`: `sum = 2`
  - `j = 2`: `sum = 5`
  - `j = 3`: `sum = 12` (Matches! Return `[2, 4]`).

### Complexity
- **Time Complexity:** $O(N^2)$ in the worst case (e.g., when all elements are small or zero).
- **Space Complexity:** $O(1)$ auxiliary space.

### C++17 Code
```cpp
#include <vector>

class Solution {
public:
    std::vector<int> subarraySumBrute(const std::vector<int>& arr, int S) {
        int n = arr.size();
        for (int i = 0; i < n; ++i) {
            long long currentSum = 0;
            for (int j = i; j < n; ++j) {
                currentSum += arr[j];
                if (currentSum == S) {
                    return {i + 1, j + 1}; // 1-based indexing
                }
                if (currentSum > S) {
                    break; // Optimization: sum can only increase
                }
            }
        }
        return {-1};
    }
};
```

---

## 6. Better Solution

### Prefix Sum + Hash Map (Handles Negatives)
This approach works whether elements are positive, negative, or zero. It utilizes $O(N)$ auxiliary space to store the prefix sums.

### Algorithm
1. Maintain a Hash Map `map<PrefixSum, Index>`.
2. Initialize `currentSum = 0`.
3. For each index `i` from `0` to $N-1$:
   - `currentSum += arr[i]`
   - If `currentSum == S`, return `[1, i + 1]`.
   - If `currentSum - S` exists in the map, return `[map[currentSum - S] + 2, i + 1]`.
   - Store the first occurrence of `currentSum` in the map: `map[currentSum] = i`.
4. Return `[-1]`.

### Complexity
- **Time Complexity:** $O(N)$ on average (using `std::unordered_map`).
- **Space Complexity:** $O(N)$ to store prefix sums in the map.

### C++17 Code
```cpp
#include <vector>
#include <unordered_map>

class Solution {
public:
    std::vector<int> subarraySumHashMap(const std::vector<int>& arr, int S) {
        int n = arr.size();
        std::unordered_map<long long, int> prefixMap;
        long long currentSum = 0;
        
        for (int i = 0; i < n; ++i) {
            currentSum += arr[i];
            
            // Case 1: Subarray starting from index 0
            if (currentSum == S) {
                return {1, i + 1};
            }
            
            // Case 2: Subarray starting from index prefixMap[currentSum - S] + 1
            if (prefixMap.find(currentSum - S) != prefixMap.end()) {
                return {prefixMap[currentSum - S] + 2, i + 1}; // +2 because of 1-based indexing
            }
            
            // Store only the first occurrence of prefixSum to find the earliest subarray
            if (prefixMap.find(currentSum) == prefixMap.end()) {
                prefixMap[currentSum] = i;
            }
        }
        return {-1};
    }
};
```

---

## 7. Optimal Solution

### Sliding Window (Two Pointers)
For non-negative numbers, we can achieve $O(1)$ space using a sliding window.

### Algorithm
1. Initialize `left = 0`, `currentSum = 0`.
2. Loop `right` from `0` to $N-1$:
   - Add `arr[right]` to `currentSum`.
   - While `currentSum > S` and `left < right`:
     - Subtract `arr[left]` from `currentSum`.
     - Increment `left`.
   - If `currentSum == S`, return `[left + 1, right + 1]`.
3. Return `[-1]`.

### Why Optimal?
- The sliding window pointers `left` and `right` only traverse the array from left to right. Each element is added at most once and removed at most once. Time complexity is $O(N)$.
- We only store pointers and sums in variables. Auxiliary space is $O(1)$.
- Monotonicity guarantees we don't miss any valid window.

### Beautiful C++17 Code
```cpp
#include <vector>

class Solution {
public:
    std::vector<int> subarraySumOptimal(const std::vector<int>& arr, int S) {
        int n = arr.size();
        int left = 0;
        long long currentSum = 0;
        
        for (int right = 0; right < n; ++right) {
            currentSum += arr[right];
            
            // Shrink the window from the left while currentSum exceeds S
            // Ensure left does not cross right (unless S = 0, which requires extra care)
            while (currentSum > S && left < right) {
                currentSum -= arr[left];
                left++;
            }
            
            // Check if we found the sum
            if (currentSum == S) {
                return {left + 1, right + 1}; // Return 1-based indices
            }
        }
        
        return {-1};
    }
};
```

---

## 8. Code Walkthrough

1. **Initialize sliding window pointers:**
   - `left` is the start of the window.
   - `right` is the end of the window.
   - `currentSum` keeps track of the sum of elements in the current window `[left, right]`.

2. **Expand the window:**
   ```cpp
   currentSum += arr[right];
   ```
   We add the element at `right` to our running sum.

3. **Shrink the window when sum exceeds target:**
   ```cpp
   while (currentSum > S && left < right) {
       currentSum -= arr[left];
       left++;
   }
   ```
   If the sum is too large, we remove elements from the left of our window until `currentSum <= S` or the window is reduced to a single element.

4. **Verify match:**
   ```cpp
   if (currentSum == S) {
       return {left + 1, right + 1};
   }
   ```
   If our window sum matches `S`, we return the 1-based indices immediately.

---

## 9. Dry Run

Input: `arr = [1, 2, 3, 7, 5]`, `S = 12`

| Step | `right` | `arr[right]` | `currentSum` (before shrink) | Shrink Condition (`currentSum > S && left < right`) | `currentSum` (after shrink) | `left` | Match? |
|:---:|:---:|:---:|:---:|:---|:---:|:---:|:---:|
| 1 | 0 | 1 | 1 | No | 1 | 0 | No |
| 2 | 1 | 2 | 3 | No | 3 | 0 | No |
| 3 | 2 | 3 | 6 | No | 6 | 0 | No |
| 4 | 3 | 7 | 13 | Yes ($13 > 12$, subtract `arr[0]=1`) | 12 | 1 | **Yes! Return [2, 4]** |

---

## 10. Complexity Analysis

### Comparison Table

| Approach | Time Complexity | Space Complexity | Handles Negatives? |
|:---|:---:|:---:|:---:|
| **Brute Force** | $O(N^2)$ | $O(1)$ | Yes |
| **Prefix Sum + Map** | $O(N)$ | $O(N)$ | Yes |
| **Sliding Window** | $O(N)$ | $O(1)$ | No (Non-negative only) |

### Optimal Space-Time Complexity
- **Time Complexity:**
  - **Best Case:** $O(1)$ (if `arr[0] == S`).
  - **Average Case:** $O(N)$
  - **Worst Case:** $O(N)$ (each element is processed at most twice: once added to the sum, once subtracted).
- **Space Complexity:**
  - $O(1)$ auxiliary space since we only store pointer variables.

---

## 11. Common Mistakes

1. **Incorrectly applying Sliding Window to arrays with negative numbers:**
   - If the interviewer mentions negative elements, sliding window is **incorrect**. You must use Prefix Sum + Hash Map.
2. **Missing `left < right` check:**
   - If $S$ is smaller than individual elements, the while loop might shrink past the current element.
3. **Off-by-One Indexing:**
   - Returning 0-based indices when the problem asks for 1-based indices.

---

## 12. Edge Cases

| Edge Case | Input Example | Expected Output | Why it Matters |
|:---|:---|:---:|:---|
| **$S$ at the start** | `arr = [4, 1, 2]`, `S = 4` | `[1, 1]` | Valid single element subarray at the beginning. |
| **$S$ at the end** | `arr = [1, 2, 4]`, `S = 4` | `[3, 3]` | Valid single element subarray at the end. |
| **No matching subarray** | `arr = [1, 2, 3]`, `S = 100` | `[-1]` | Gracefully terminates and returns `[-1]`. |
| **$S = 0$ and element is 0** | `arr = [0, 2, 3]`, `S = 0` | `[1, 1]` | Correctly identifies zero-element subarray. |
| **Large Sums (Overflow)** | `arr = [2^30, 2^30]`, `S = 2^31` | `[1, 2]` | Sum variable must be `long long` to prevent overflow. |

---

## 13. Interview Explanation

**Speaker Script:**
> "To solve the Subarray with Given Sum problem, I will first ask the interviewer if the array contains negative numbers. 
> If the array contains only non-negative integers, we can solve it in $O(N)$ time and $O(1)$ space using a sliding window approach. We maintain a running sum of a window defined by two pointers. We expand the window by moving the right pointer and adding elements. If the sum exceeds our target, we shrink the window from the left until the sum is less than or equal to the target. If the sum equals the target, we return the indices.
> If the array can contain negative numbers, the monotonicity is lost. In that case, I would use the Prefix Sum + Hash Map approach. I will maintain a running prefix sum and store the first occurrence of each prefix sum in a hash map. At each element, I check if `prefixSum - S` has been seen before. If yes, it implies the subarray between that seen index and the current index sums to `S`. This runs in $O(N)$ time and $O(N)$ space."

---

## 14. Follow-up Questions

1. **How would you find the longest/shortest subarray that sums to S?**
   - *Answer:* For sliding window, instead of returning immediately when `currentSum == S`, we update a `maxLength` or `minLength` variable and continue searching. For the hash map, we also search the map but do not return immediately, updating the optimal length.
2. **What if the target sum is negative?**
   - *Answer:* The Prefix Sum + Hash Map solution works without any modification for negative target sums.

---

## 15. Variations

1. **Subarray Sum Equals K (LeetCode 560):** Find the total count of subarrays with sum $K$ (requires Hash Map).
2. **Minimum Size Subarray Sum (LeetCode 209):** Find the minimal length of a contiguous subarray for which the sum $\ge$ target.

---

## 16. Revision Notes

- **Non-negative elements:** Sliding Window ($O(1)$ space).
- **Negative elements:** Prefix Sum + Hash Map ($O(N)$ space).
- **Formula:** `prefixSum[i] - prefixSum[j-1] == S` $\implies$ `prefixSum[j-1] == prefixSum[i] - S`.
- **Indexing:** Watch out for 1-based indexing in return statements.

---

## 17. Memorization Version

```cpp
// Non-negative only: Time O(N), Space O(1)
vector<int> subarraySum(vector<int>& arr, int S) {
    int left = 0; long long sum = 0;
    for (int right = 0; right < arr.size(); ++right) {
        sum += arr[right];
        while (sum > S && left < right) {
            sum -= arr[left];
            left++;
        }
        if (sum == S) return {left + 1, right + 1};
    }
    return {-1};
}
```
