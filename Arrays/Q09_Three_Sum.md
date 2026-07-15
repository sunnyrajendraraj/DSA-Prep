# Find Triplet with Given Sum (3-Sum)

---

## 1. Problem Statement

Given an integer array `nums`, return all the unique triplets `[nums[i], nums[j], nums[k]]` such that `i != j`, `i != k`, and `j != k`, and `nums[i] + nums[j] + nums[k] == 0` (or a given target sum).

Notice that the solution set must not contain duplicate triplets.

### Input
- An integer array `nums` of size `n` ($3 \le n \le 3000$).

### Output
- A 2D vector of integers containing all unique triplets that sum to zero.

### Constraints
- $3 \le n \le 3000$
- $-10^5 \le nums[i] \le 10^5$

### Observations
1. **Uniqueness:** The order of numbers in a triplet does not matter (e.g., `[-1, 0, 1]` is the same as `[0, -1, 1]`). We must avoid reporting duplicate triplets.
2. **Target:** In the standard version, the target is `0`.
3. **Sorting:** Sorting the array helps in easily skipping duplicate elements and using two-pointer techniques.

---

## 2. Interview Clarification

Before coding, clarify the following with the interviewer:
1. **Should the returned triplets be sorted?**
   - *Answer:* The triplets themselves and the order of triplets can be returned in any order, but each individual triplet should not contain duplicate entries from different index combinations that have the same values.
2. **Can we reuse the same index multiple times in a single triplet?**
   - *Answer:* No, each element in the triplet must come from a distinct index.
3. **What if the array size is less than 3?**
   - *Answer:* The constraints specify $n \ge 3$. If it can be less than 3, return an empty list.

---

## 3. Thinking Process

Let's think aloud:
* **Initial thoughts:** We want to find three indices $i < j < k$ such that $nums[i] + nums[j] + nums[k] = 0$.
* **Brute Force:** Write three nested loops.
  - For $i$ from $0$ to $n-1$:
    - For $j$ from $i+1$ to $n-1$:
      - For $k$ from $j+1$ to $n-1$:
        - If $nums[i] + nums[j] + nums[k] == 0$, sort the triplet and insert it into a hash set to ensure uniqueness.
  - Set lookup/insertion handles duplicates. Time complexity: $O(n^3 \log(\text{number of triplets}))$. Space complexity: $O(\text{number of unique triplets})$ for the set.
* **Better (Hash Set lookup):** Can we optimize the third loop?
  - For a fixed $i$ and $j$, we need to find if there is a $k$ such that $nums[k] = -(nums[i] + nums[j])$.
  - We can use a hash set for the elements between $i$ and $j$ (or after $j$).
  - For each fixed $i$, we iterate $j$ from $i+1$ to $n-1$. The required third value is $-(nums[i] + nums[j])$. If it is in the hash set, we found a triplet.
  - To prevent duplicates, we sort the triplet and insert it into a global `std::set`.
  - Time complexity: $O(n^2 \log(\text{triplets}))$. Space complexity: $O(n)$ for the inner set + $O(\text{triplets})$ for the global set.
* **Optimal (Sorting + Two Pointers):** Can we do it in $O(n^2)$ time with $O(1)$ auxiliary space (excluding the output)?
  - First, sort the array `nums`. Sorting takes $O(n \log n)$.
  - Once sorted, we fix the first element $nums[i]$.
  - The remaining problem is to find two elements in the subarray `nums[i+1 ... n-1]` that sum to $-nums[i]$. This is exactly the **Two Sum II (sorted array)** problem!
  - We can place pointers `left = i + 1` and `right = n - 1`.
  - If $nums[left] + nums[right] < -nums[i]$, we increment `left`.
  - If $nums[left] + nums[right] > -nums[i]$, we decrement `right`.
  - If they equal, we record the triplet `[nums[i], nums[left], nums[right]]`.
  - **Handling Duplicates:**
    - To avoid duplicates for the first element, if $nums[i] == nums[i-1]$, we skip it.
    - To avoid duplicates for the second/third elements, when a match is found, we increment `left` and decrement `right` while skipping elements that are identical to the ones we just matched.

---

## 4. Pattern Recognition

This problem belongs to the **Two Pointers** and **Sorting** patterns.
* **Why this topic?** Summing multiple elements to a target is best optimized by sorting the collection and narrowing search boundaries from both sides.
* **Which pattern?**
  - **Fix One and Two-Pointer:** Reducing 3-Sum to 2-Sum by fixing one variable.
  - **Duplicate Skipping:** Moving pointers past identical elements in a sorted array to maintain uniqueness without using set data structures.

---

## 5. Brute Force Solution

Use three nested loops to find all combinations, sorting each triplet and inserting it into a set to eliminate duplicate entries.

### Algorithm
1. Create a `std::set<std::vector<int>>` to hold unique triplets.
2. Iterate $i$ from $0$ to $n-1$.
3. Iterate $j$ from $i+1$ to $n-1$.
4. Iterate $k$ from $j+1$ to $n-1$.
5. If $nums[i] + nums[j] + nums[k] == 0$:
   - Create a triplet vector `[nums[i], nums[j], nums[k]]`.
   - Sort the vector.
   - Insert it into the set.
6. Convert the set to a 2D vector and return.

### Dry Run
Input: `nums = [-1, 0, 1, 2, -1, -4]`.
- All possible triplets are generated.
- Triplet `[-1, 0, 1]` is found: sorted `[-1, 0, 1]`, inserted to set.
- Triplet `[-1, 2, -1]` is found: sorted `[-1, -1, 2]`, inserted to set.
- Duplicate triplet `[-1, 0, 1]` from another index set is found: sorted `[-1, 0, 1]`, rejected by set because it already exists.

### Complexity Analysis
- **Time Complexity:** $O(n^3 \log(\text{triplets}))$ — Three nested loops, and inserting into a set of size $T$ takes $O(\log T)$.
- **Space Complexity:** $O(T)$ — Where $T$ is the number of unique triplets stored in the set.

### Pros/Cons
- **Pros:** Simple logic, guarantees unique outputs.
- **Cons:** Extremely slow; runs out of time for $n > 200$.

### C++17 Code
```cpp
#include <vector>
#include <set>
#include <algorithm>

class Solution {
public:
    std::vector<std::vector<int>> threeSum(std::vector<int>& nums) {
        int n = nums.size();
        std::set<std::vector<int>> unique_triplets;
        
        for (int i = 0; i < n; ++i) {
            for (int j = i + 1; j < n; ++j) {
                for (int k = j + 1; k < n; ++k) {
                    if (nums[i] + nums[j] + nums[k] == 0) {
                        std::vector<int> triplet = {nums[i], nums[j], nums[k]};
                        std::sort(triplet.begin(), triplet.end());
                        unique_triplets.insert(triplet);
                    }
                }
            }
        }
        
        return std::vector<std::vector<int>>(unique_triplets.begin(), unique_triplets.end());
    }
};
```

---

## 6. Better Solution

We can use a hash set to optimize the innermost loop.

### Algorithm
1. Create a global `std::set<std::vector<int>>` to collect unique triplets.
2. Iterate $i$ from $0$ to $n-1$.
3. For each $i$, initialize a hash set `seen`.
4. Iterate $j$ from $i+1$ to $n-1$:
   - Calculate the third element needed: `required = -(nums[i] + nums[j])`.
   - If `required` is in `seen`:
     - Construct triplet `[nums[i], nums[j], required]`, sort it, and insert it into the global set.
   - Insert `nums[j]` into `seen`.
5. Return the elements of the global set as a 2D vector.

### Dry Run
Input: `nums = [-1, 0, 1, 2]`.
- $i = 0$ (`nums[i] = -1`):
  - `seen = {}`
  - $j = 1$ (`nums[j] = 0`): `required = -(-1 + 0) = 1`. Not in `seen`. Insert `0` $\rightarrow$ `seen = {0}`.
  - $j = 2$ (`nums[j] = 1`): `required = -(-1 + 1) = 0`. Found in `seen`! Insert sorted `[-1, 0, 1]` to global set. Insert `1` $\rightarrow$ `seen = {0, 1}`.
  - $j = 3$ (`nums[j] = 2`): `required = -(-1 + 2) = -1`. Not in `seen`. Insert `2`.

### Complexity Analysis
- **Time Complexity:** $O(n^2 \log(\text{triplets}))$ — Two nested loops with $O(1)$ hash set lookups, and set insertion taking logarithmic time.
- **Space Complexity:** $O(n + T)$ — Hash set `seen` uses $O(n)$ space; global set uses $O(T)$ space.

### Pros/Cons
- **Pros:** Faster than brute force, does not require sorting the main array first.
- **Cons:** Still requires a set to filter duplicates, causing memory overhead and logarithmic insertion overhead.

### C++17 Code
```cpp
#include <vector>
#include <set>
#include <unordered_set>
#include <algorithm>

class Solution {
public:
    std::vector<std::vector<int>> threeSum(std::vector<int>& nums) {
        int n = nums.size();
        std::set<std::vector<int>> global_set;
        
        for (int i = 0; i < n; ++i) {
            std::unordered_set<int> seen;
            for (int j = i + 1; j < n; ++j) {
                int required = -(nums[i] + nums[j]);
                if (seen.find(required) != seen.end()) {
                    std::vector<int> triplet = {nums[i], nums[j], required};
                    std::sort(triplet.begin(), triplet.end());
                    global_set.insert(triplet);
                }
                seen.insert(nums[j]);
            }
        }
        
        return std::vector<std::vector<int>>(global_set.begin(), global_set.end());
    }
};
```

---

## 7. Optimal Solution

Sort the array and apply a two-pointer approach, skipping duplicate numbers inline to avoid using a separate set.

### Algorithm
1. Sort the input array `nums`.
2. Iterate $i$ from $0$ to $n-3$:
   - If $nums[i] > 0$, break the loop (since the array is sorted, three positive numbers can never sum to zero).
   - If $i > 0$ and $nums[i] == nums[i-1]$, skip this element to avoid duplicate triplets.
   - Set two pointers: `left = i + 1` and `right = n - 1`.
   - While `left < right`:
     - Calculate `sum = nums[i] + nums[left] + nums[right]`.
     - If `sum == 0`:
       - Record triplet `{nums[i], nums[left], nums[right]}`.
       - Move pointers inward: `left++`, `right--`.
       - Skip identical elements to avoid duplicates:
         - While `left < right` and $nums[left] == nums[left-1]$, increment `left`.
         - While `left < right` and $nums[right] == nums[right+1]$, decrement `right`.
     - Else if `sum < 0`, we need a larger value $\rightarrow$ `left++`.
     - Else (i.e., `sum > 0`), we need a smaller value $\rightarrow$ `right--`.
3. Return the result vector.

### C++17 Code
```cpp
#include <vector>
#include <algorithm>

class Solution {
public:
    std::vector<std::vector<int>> threeSum(std::vector<int>& nums) {
        std::vector<std::vector<int>> triplets;
        int n = nums.size();
        
        // Step 1: Sort the array
        std::sort(nums.begin(), nums.end());
        
        for (int i = 0; i < n - 2; ++i) {
            // Optimization: If the smallest number of the triplet is positive, 
            // the sum can never be 0.
            if (nums[i] > 0) break;
            
            // Skip duplicate values for the first element
            if (i > 0 && nums[i] == nums[i - 1]) continue;
            
            int left = i + 1;
            int right = n - 1;
            
            while (left < right) {
                int current_sum = nums[i] + nums[left] + nums[right];
                
                if (current_sum == 0) {
                    triplets.push_back({nums[i], nums[left], nums[right]});
                    
                    left++;
                    right--;
                    
                    // Skip duplicate values for the second element
                    while (left < right && nums[left] == nums[left - 1]) {
                        left++;
                    }
                    // Skip duplicate values for the third element
                    while (left < right && nums[right] == nums[right + 1]) {
                        right--;
                    }
                } 
                else if (current_sum < 0) {
                    left++;
                } 
                else {
                    right--;
                }
            }
        }
        
        return triplets;
    }
};
```

---

## 8. Code Walkthrough

* `std::sort(nums.begin(), nums.end());`: Sorting groups duplicates together and allows us to use two-pointer logic.
* `if (nums[i] > 0) break;`: Since the array is sorted, if the first element `nums[i]` is positive, all subsequent elements are also positive, making a zero sum impossible.
* `if (i > 0 && nums[i] == nums[i - 1]) continue;`: Skips duplicates for the first element. This is crucial for avoiding duplicate triplets.
* `int left = i + 1, right = n - 1;`: Sets up the search window.
* `while (left < right && nums[left] == nums[left - 1]) left++;`: Skips duplicate values for the second element after finding a match.
* `while (left < right && nums[right] == nums[right + 1]) right--;`: Skips duplicate values for the third element after finding a match.

---

## 9. Dry Run

Input: `nums = [-1, 0, 1, 2, -1, -4]`.
1. Sort `nums` $\rightarrow$ `[-4, -1, -1, 0, 1, 2]`. $n = 6$.
2. **$i = 0$ (`nums[0] = -4`):**
   - `left = 1` (`nums[1] = -1`), `right = 5` (`nums[5] = 2`).
   - `sum = -4 + (-1) + 2 = -3 < 0` $\rightarrow$ `left++` (now 2).
   - `left = 2` (`nums[2] = -1`), `right = 5` (`nums[5] = 2`).
   - `sum = -4 + (-1) + 2 = -3 < 0` $\rightarrow$ `left++` (now 3).
   - `left = 3` (`nums[3] = 0`), `right = 5` (`nums[5] = 2`).
   - `sum = -4 + 0 + 2 = -2 < 0` $\rightarrow$ `left++` (now 4).
   - `left = 4` (`nums[4] = 1`), `right = 5` (`nums[5] = 2`).
   - `sum = -4 + 1 + 2 = -1 < 0` $\rightarrow$ `left++` (now 5). Loop ends.
3. **$i = 1$ (`nums[1] = -1`):**
   - `left = 2` (`nums[2] = -1`), `right = 5` (`nums[5] = 2`).
   - `sum = -1 + (-1) + 2 = 0`. Found! Add `[-1, -1, 2]`.
   - `left++` (3), `right--` (4).
   - Duplicate checks:
     - `nums[left] == nums[3] = 0` $\neq$ `nums[2]` (-1) $\rightarrow$ stop.
     - `nums[right] == nums[4] = 1` $\neq$ `nums[5]` (2) $\rightarrow$ stop.
   - Pointers are now: `left = 3` (`0`), `right = 4` (`1`).
   - `sum = -1 + 0 + 1 = 0`. Found! Add `[-1, 0, 1]`.
   - `left++` (4), `right--` (3). Loop ends since `left >= right`.
4. **$i = 2$ (`nums[2] = -1`):**
   - Since `nums[2] == nums[1]`, skip this iteration.
5. **$i = 3$ (`nums[3] = 0`):**
   - `left = 4` (`1`), `right = 5` (`2`).
   - `sum = 0 + 1 + 2 = 3 > 0` $\rightarrow$ `right--`. Loop ends.
6. **$i = 4$ (`nums[4] = 1`):**
   - Outer loop terminates because $i < n - 2$ is no longer true.

Result: `[[-1, -1, 2], [-1, 0, 1]]`.

---

## 10. Complexity Analysis

| Approach | Time Complexity | Space Complexity | Notes |
| :--- | :--- | :--- | :--- |
| **Brute Force** | $O(n^3 \log T)$ | $O(T)$ | Extremely slow due to triple nested loops and set sorting. |
| **Better** | $O(n^2 \log T)$ | $O(n + T)$ | Faster, but uses hash set and global set overhead. |
| **Optimal** | $O(n^2)$ | $O(1)$ or $O(\log n)$ | In-place pointers. Space is just for sorting recursion stack. |

* **Time Complexity (Optimal):** Sorting takes $O(n \log n)$. The outer loop runs $n-2$ times, and the inner two-pointer search runs in $O(n)$ time. Overall time complexity is $O(n^2)$.
* **Space Complexity (Optimal):** $O(1)$ auxiliary space if we do not count the output list and stack space for sorting. In C++, `std::sort` typically uses $O(\log n)$ space for call stack.

---

## 11. Common Mistakes

1. **Not skipping duplicates properly:** Forgetting to skip duplicate elements for both the outer variable `nums[i]` and the inner pointers `nums[left]`/`nums[right]`.
2. **Missing `left < right` bounds during duplicate skipping:** Loops like `while (nums[left] == nums[left-1]) left++;` can run out of bounds without the `left < right` safety condition.
3. **Not sorting first:** The two-pointer technique relies entirely on the elements being sorted.

---

## 12. Edge Cases

| Edge Case | Input Example | Expected Output | Why it matters |
| :--- | :--- | :--- | :--- |
| **All zeros** | `[0, 0, 0, 0]` | `[[0, 0, 0]]` | Must output only one triplet, skipping duplicates. |
| **No solution** | `[1, 2, -2, -1]` | `[]` | Must exit gracefully and return empty list. |
| **Minimum size** | `[0, 1, -1]` | `[[-1, 0, 1]]` | Smallest valid input, boundary test. |

---

## 13. Interview Explanation

"To find all unique triplets in an array that sum to zero, a brute force approach of checking every combination of three elements takes $O(n^3)$ time.
We can optimize this to $O(n^2)$ by first sorting the array. Once sorted, we iterate through the array and fix the first element, `nums[i]`. We can then find the remaining two elements using a two-pointer approach starting from `i+1` and `n-1`.
To ensure all triplets are unique without using extra memory (like sets), we skip duplicate values for the first element `nums[i]`, and when we find a matching triplet, we also skip duplicate values for the second and third elements before continuing our search. This gives us an optimal $O(n^2)$ time complexity and $O(1)$ auxiliary space complexity."

---

## 14. Follow-up Questions

1. **How would you solve 4-Sum?**
   * *Answer:* Sort the array, use two nested loops to fix the first two elements, and then use two pointers for the remaining two. This runs in $O(n^3)$ time.
2. **What if the target is not 0 but a variable `target`?**
   * *Answer:* The algorithm remains identical, but the two-pointer target becomes `target - nums[i]`.

---

## 15. Variations

1. **3-Sum Closest:** Find a triplet whose sum is closest to a target.
   - *Difference:* Maintain a minimum difference variable and update it at each step. Do not return early.
2. **3-Sum Smaller:** Count the number of triplets with a sum less than target.
   - *Difference:* If `sum < target`, all pairs between `left` and `right` also sum to less than target, so we add `right - left` to our count and increment `left`.

---

## 16. Revision Notes

* **Core Equation:** $A + B + C = 0 \implies B + C = -A$.
* **Avoid Sets:** Sorting + skipping duplicates using `while` loops is the standard way to achieve $O(1)$ space.
* **Skip Conditions:**
  - Outer: `if (i > 0 && nums[i] == nums[i-1]) continue;`
  - Inner Left: `while (left < right && nums[left] == nums[left-1]) left++;`
  - Inner Right: `while (left < right && nums[right] == nums[right+1]) right--;`

---

## 17. Memorization Version

```cpp
// 3-Sum Optimal
sort(nums.begin(), nums.end());
for (int i = 0; i < n - 2; ++i) {
    if (nums[i] > 0) break;
    if (i > 0 && nums[i] == nums[i-1]) continue;
    int L = i + 1, R = n - 1;
    while (L < R) {
        int sum = nums[i] + nums[L] + nums[R];
        if (sum == 0) {
            res.push_back({nums[i], nums[L], nums[R]});
            L++; R--;
            while (L < R && nums[L] == nums[L-1]) L++;
            while (L < R && nums[R] == nums[R+1]) R--;
        } else if (sum < 0) L++;
        else R--;
    }
}
```
* **Time:** $O(n^2)$, **Space:** $O(1)$ auxiliary.
