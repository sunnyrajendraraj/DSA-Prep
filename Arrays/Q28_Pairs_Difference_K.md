# Count Pairs with Given Difference K

## 1. Problem Statement

Given an array of integers `arr` of size `N` and a non-negative integer `K`, count the number of pairs $(i, j)$ such that $i < j$ and the absolute difference between `arr[i]` and `arr[j]` is equal to `K`. That is:
$$|arr[i] - arr[j]| = K$$

### Input
- An integer array `arr` of size `N`.
- A non-negative integer `K`.

### Output
- An integer representing the number of pairs matching the condition.

### Constraints
- $1 \le N \le 10^5$
- $0 \le K \le 10^5$
- $-10^9 \le arr[i] \le 10^9$

### Observations
1. The condition $|arr[i] - arr[j]| = K$ implies that the two numbers must be $x$ and $x + K$ (or $x$ and $x - K$).
2. The index constraint $i < j$ means we are looking for unordered pairs of indices.
3. If $K > 0$, each pair consists of two distinct numbers.
4. If $K = 0$, the condition simplifies to $arr[i] = arr[j]$, meaning we are counting pairs of duplicate elements.

---

## 2. Interview Clarification

- **Q:** Can elements in the array be negative?
  - **A:** Yes, elements can be negative.
- **Q:** Can `K` be negative?
  - **A:** No, since difference is defined as absolute value, `K` is always non-negative.
- **Q:** How do we handle duplicate elements?
  - **A:** If `K = 0` and we have three `1`s, they form $\frac{3 \times 2}{2} = 3$ pairs. If `K = 2` and we have `[1, 1, 3]`, each `1` forms a pair with `3`, yielding 2 pairs in total.
- **Q:** Should the solution work in-place?
  - **A:** Yes, unless the interviewer specifies that the input is read-only, in which case we copy the array if we need to sort.

---

## 3. Thinking Process

### First Instinct (Brute Force)
We can check every possible pair of elements in the array.
- Run a nested loop where the outer loop variable `i` goes from `0` to `N - 1`.
- The inner loop variable `j` goes from `i + 1` to `N - 1`.
- Check if `abs(arr[i] - arr[j]) == K`. If so, increment the counter.
- This takes $O(N^2)$ time and $O(1)$ space. It will time out for $N = 10^5$.

### Space-Time Tradeoff (Optimal Insight)
Can we solve this in $O(N)$?
Let's utilize a frequency hash map (`unordered_map` in C++) to store the counts of each number.
Let's split the logic based on whether $K = 0$ or $K > 0$:

#### Case 1: $K > 0$
- For each unique element $x$ in our map:
  - We look for $x + K$.
  - If $x + K$ exists, the number of pairs formed by these two values is:
    $$\text{Pairs} = \text{frequency}(x) \times \text{frequency}(x + K)$$
  - We sum this over all unique elements $x$. By looking only for $x + K$ (and not $x - K$), we avoid double counting.

#### Case 2: $K = 0$
- If $K = 0$, we want pairs where $arr[i] = arr[j]$.
- For each unique element $x$ with frequency $F$:
  - The number of ways to choose 2 indices out of $F$ is given by the combination formula:
    $$\text{Pairs} = \frac{F \times (F - 1)}{2}$$
  - We sum this over all unique elements.

This hash map approach runs in $O(N)$ time and $O(N)$ space.

### Space Optimization (Sorting + Two Pointers)
If we want to avoid $O(N)$ extra space, we can sort the array first. Sorting takes $O(N \log N)$ time and $O(1)$ auxiliary space.
- Once sorted, we can use two pointers: `left = 0` and `right = 1`.
- While `right < N` and `left < N`:
  - If `arr[right] - arr[left] < K`: we need a larger difference, so move `right++`.
  - If `arr[right] - arr[left] > K`: we need a smaller difference, so move `left++`.
  - If `arr[right] - arr[left] == K`:
    - We must handle duplicates carefully.
    - If `K == 0`: the elements at `left` and `right` are equal. We count the run of equal elements and add their combination count.
    - If `K > 0`: count how many duplicates of `arr[left]` exist (say, `c1`) and how many duplicates of `arr[right]` exist (say, `c2`). The pairs contributed are `c1 * c2`. Then update both pointers past these duplicates.

---

## 4. Pattern Recognition

- **Pattern:** Hash Map Frequency Count / Sorting + Two Pointers.
- **Why this topic?** Finding pairs with a target difference is a variant of the Two-Sum problem. Using a hash map allows $O(1)$ lookups, reducing the problem to linear time. Sorting provides an alternative $O(N \log N)$ time, $O(1)$ space solution.

---

## 5. Brute Force Solution

### Algorithm
1. Initialize `count = 0`.
2. Loop `i` from `0` to `N - 1`.
3. Loop `j` from `i + 1` to `N - 1`.
4. If `abs(arr[i] - arr[j]) == K`, increment `count`.
5. Return `count`.

### Complexity Analysis
- **Time Complexity:** $O(N^2)$ due to the nested loops.
- **Space Complexity:** $O(1)$ auxiliary space.

### Brute Force C++17 Code
```cpp
#include <vector>
#include <cmath>

class Solution {
public:
    long long countPairsBrute(const std::vector<int>& arr, int k) {
        int n = arr.size();
        long long count = 0;
        for (int i = 0; i < n; ++i) {
            for (int j = i + 1; j < n; ++j) {
                if (std::abs(arr[i] - arr[j]) == k) {
                    count++;
                }
            }
        }
        return count;
    }
};
```

---

## 6. Better Solution (Sorting + Two Pointers)

### Algorithm
1. Sort the array `arr`.
2. Initialize `count = 0`, `left = 0`, `right = 1`.
3. While `right < N`:
   - If `left == right`, increment `right` to maintain distinct indices.
   - If `arr[right] - arr[left] < K`, increment `right`.
   - Else if `arr[right] - arr[left] > K`, increment `left`.
   - Else (difference is exactly `K`):
     - If `K == 0`:
       - Count how many identical elements there are starting from `left`. Let this count be `len`.
       - Add `len * (len - 1) / 2` to `count`.
       - Advance both pointers past this group.
     - If `K > 0`:
       - Count duplicates of `arr[left]` (let it be `c1`).
       - Count duplicates of `arr[right]` (let it be `c2`).
       - Add `c1 * c2` to `count`.
       - Advance `left` by `c1` and `right` by `c2`.

### Complexity Analysis
- **Time Complexity:** $O(N \log N)$ for sorting; the two-pointer pass takes $O(N)$ time.
- **Space Complexity:** $O(1)$ auxiliary space (assuming in-place sorting).

### Better C++17 Code
```cpp
#include <vector>
#include <algorithm>

class Solution {
public:
    long long countPairsTwoPointers(std::vector<int>& arr, int k) {
        int n = arr.size();
        if (n < 2) return 0;
        
        std::sort(arr.begin(), arr.end());
        long long count = 0;
        
        int left = 0;
        int right = 1;
        
        while (right < n) {
            if (left == right) {
                right++;
                continue;
            }
            
            long long diff = (long long)arr[right] - arr[left];
            
            if (diff < k) {
                right++;
            } else if (diff > k) {
                left++;
            } else {
                if (k == 0) {
                    // Count identical elements
                    long long len = 1;
                    while (right < n && arr[right] == arr[left]) {
                        len++;
                        right++;
                    }
                    count += (len * (len - 1)) / 2;
                    left = right;
                    right = left + 1;
                } else {
                    // Count duplicates at left
                    long long c1 = 1;
                    while (left + 1 < n && arr[left + 1] == arr[left]) {
                        c1++;
                        left++;
                    }
                    // Count duplicates at right
                    long long c2 = 1;
                    while (right + 1 < n && arr[right + 1] == arr[right]) {
                        c2++;
                        right++;
                    }
                    count += c1 * c2;
                    left++;
                    right++;
                }
            }
        }
        return count;
    }
};
```

---

## 7. Optimal Solution (Frequency Hash Map)

### Algorithm
1. Create a hash map `freq` to store the frequency of each element in `arr`.
2. Populate the hash map with a single pass over the array.
3. Initialize `pair_count = 0`.
4. Iterate through each key-value pair `(x, f)` in the hash map:
   - **If K > 0**:
     - Check if `x + K` exists in the hash map.
     - If yes, add `f * freq[x + K]` to `pair_count`.
   - **If K == 0**:
     - Add `f * (f - 1) / 2` to `pair_count`.
5. Return `pair_count`.

### Why this is Optimal
It counts the matching pairs in $O(N)$ time using $O(N)$ space. Since we must inspect every element to find pairs, we cannot do better than $O(N)$ time.

### C++17 Optimal Code
```cpp
#include <vector>
#include <unordered_map>

class Solution {
public:
    long long countPairs(const std::vector<int>& arr, int k) {
        // Frequency map to store occurrences of each number
        std::unordered_map<long long, long long> freq;
        for (int num : arr) {
            freq[num]++;
        }
        
        long long pair_count = 0;
        
        // Process each unique number in the map
        for (const auto& [val, count] : freq) {
            if (k > 0) {
                // If k > 0, check if val + k exists in the map.
                // We only check val + k (not val - k) to prevent double counting.
                long long target = val + k;
                if (freq.count(target)) {
                    pair_count += count * freq[target];
                }
            } else {
                // If k == 0, count combinations of choosing 2 items from 'count' items.
                pair_count += (count * (count - 1)) / 2;
            }
        }
        
        return pair_count;
    }
};
```

---

## 8. Code Walkthrough

1. **Populate Hash Map**:
   - We loop through `arr` and increment the count in `freq`.
2. **Loop over Map**:
   - We iterate over the key-value pairs `[val, count]` in the `freq` map.
3. **Difference Check (K > 0)**:
   - For each unique element `val`, we check if `val + k` is present in the map.
   - If it exists, there are `count` occurrences of `val` and `freq[val + k]` occurrences of `val + k`. 
   - Multiplying these gives all pairs of indices `(i, j)` where `arr[j] - arr[i] = k`.
   - Because we do this for all unique elements, checking `val + k` exactly once per unique element covers all pairs without double counting.
4. **Equality Check (K == 0)**:
   - We calculate the combination of picking any two indices containing the same value: $\frac{\text{count} \times (\text{count} - 1)}{2}$.

---

## 9. Dry Run

Let's dry run the optimal solution on:
1. `arr = [1, 5, 3, 4, 2]`, `K = 2`
2. `arr = [1, 1, 1, 2]`, `K = 0`

### Trace 1: `arr = [1, 5, 3, 4, 2]`, `K = 2`
- Map: `{1: 1, 5: 1, 3: 1, 4: 1, 2: 1}`

| Element `val` | `count` | `target = val + 2` | `freq.count(target)` | Pairs Added (`count * freq[target]`) | Total Pairs |
|:---:|:---:|:---:|:---:|:---:|:---:|
| 1 | 1 | 3 | True | `1 * 1 = 1` | 1 |
| 5 | 1 | 7 | False | 0 | 1 |
| 3 | 1 | 5 | True | `1 * 1 = 1` | 2 |
| 4 | 1 | 6 | False | 0 | 2 |
| 2 | 1 | 4 | True | `1 * 1 = 1` | **3** |

Output: `3` (The pairs are `(1, 3)`, `(3, 5)`, and `(2, 4)`).

---

### Trace 2: `arr = [1, 1, 1, 2]`, `K = 0`
- Map: `{1: 3, 2: 1}`

| Element `val` | `count` | `k == 0` Action | Pairs Added | Total Pairs |
|:---:|:---:|:---|:---:|:---:|
| 1 | 3 | `(3 * 2) / 2` | 3 | 3 |
| 2 | 1 | `(1 * 0) / 2` | 0 | **3** |

Output: `3` (The pairs of indices are `(0, 1)`, `(0, 2)`, and `(1, 2)`).

---

## 10. Complexity Analysis

- **Time Complexity:** 
  - **Best Case:** $O(N)$ (populating map takes $O(N)$, iteration takes $O(U)$ where $U \le N$ is the number of unique elements).
  - **Average Case:** $O(N)$.
  - **Worst Case:** $O(N)$ with standard hash maps. (If there are collision issues, it could theoretically degrade to $O(N^2)$, but this can be avoided by using custom hashers or `std::map` which guarantees $O(N \log N)$ worst-case).
- **Space Complexity:**
  - **Auxiliary Space:** $O(N)$ to store the frequency map of size up to $N$.

---

## 11. Common Mistakes

1. **Double Counting**: If `K > 0` and you check both `val + K` and `val - K` for every element, you will count every pair twice. 
2. **Handling K = 0 incorrectly**: If you apply the `val + K` check when `K = 0`, it will check if `val` is in the map (which it always is), leading to `count * count` additions instead of the correct combinations formula `count * (count - 1) / 2`.
3. **Integer Overflow**: If the count of duplicates is large (e.g. $10^5$ copies of `1`), the number of pairs will be $\approx \frac{10^{10}}{2} = 5 \times 10^9$, which overflows a standard 32-bit signed integer. Always use `long long` for pair counts.

---

## 12. Edge Cases

| Edge Case | Input Example | Expected Output | Why it matters |
|:---|:---|:---|:---|
| **K = 0** | `[1, 1, 1]` | `3` | Tests duplicate selection logic. |
| **No Pairs Exist** | `[1, 2, 3]`, `K = 5` | `0` | Returns 0 correctly. |
| **All Negative Elements** | `[-3, -1, -5]`, `K = 2` | `2` | Handles negative integers correctly. |
| **Very Large Values** | `[2000000000, -2000000000]`, `K = 4000000000` | `0` (or `1` if fits) | Tests boundary differences and integer promotions. |

---

## 13. Interview Explanation

"To count the pairs with an absolute difference of `K`, we can first implement a brute force solution using nested loops that takes $O(N^2)$ time. 

However, we can optimize this to $O(N)$ time by using a frequency hash map. We count the frequencies of all elements. 
- If `K > 0`, we iterate through the unique values in the map. For each value `x`, we check if `x + K` exists in the map. The number of pairs formed by these two values is `freq[x] * freq[x + K]`. We add this to our total. By only checking `x + K` (and not `x - K`), we prevent counting any pair twice.
- If `K = 0`, we are looking for pairs of identical elements. For each unique element with frequency `F`, we calculate the combinations of choosing two identical elements, which is `F * (F - 1) / 2`.

This map-based solution runs in $O(N)$ time and $O(N)$ space. If auxiliary space is restricted to $O(1)$, we can sort the array and use a two-pointer approach, which runs in $O(N \log N)$ time."

---

## 14. Follow-up Questions

- **Q:** How would you solve this if you are not allowed to use any extra space, and the array is read-only?
  - **A:** We cannot modify (sort) the array in-place, and we cannot use a hash map. We are forced to use the $O(N^2)$ brute force approach, or $O(N \log N)$ if we copy the array to sort it.
- **Q:** What if we need to return the index pairs instead of just the count?
  - **A:** We can store the indices of each element in a hash map: `unordered_map<int, vector<int>>`. For each `val`, we pair every index in `indices[val]` with every index in `indices[val + K]`.

---

## 15. Variations

- **Two Sum (Count Pairs with Sum K):** Similar concept, but we check if `K - val` exists in the map.
- **Pairs with Difference Less than K:** Requires sorting and two pointers where we find the count of elements in the range `[arr[i], arr[i] + K)`.

---

## 16. Revision Notes

- **Map Formula (K > 0):** `total += freq[x] * freq[x + K]`.
- **Map Formula (K = 0):** `total += freq[x] * (freq[x] - 1) / 2`.
- **Space-Time Tradeoff:** Hash Map is $O(N)/O(N)$. Sorting + Two Pointers is $O(N \log N)/O(1)$.
- **Overflow Alert:** Use `long long` for counts to avoid integer overflows on large inputs.

---

## 17. Memorization Version

- **Populate** `freq` map.
- **Loop** through `freq` map:
  - If `K > 0` and `freq.count(x + K)` $\rightarrow$ `ans += freq[x] * freq[x + K]`.
  - If `K == 0` $\rightarrow$ `ans += freq[x] * (freq[x] - 1) / 2`.
- **Complexity:** $O(N)$ time, $O(N)$ space.
