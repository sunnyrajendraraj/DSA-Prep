# Majority Element (Boyer-Moore Voting)

## 1. Problem Statement

Given an array `arr` of size `N`, find the majority element. The majority element is the element that appears more than $\lfloor N/2 \rfloor$ times. 

You may assume that the majority element may or may not exist in the array. If no such element exists, return `-1`.

### Input
- An integer array `arr` of size `N`.

### Output
- An integer representing the majority element, or `-1` if it does not exist.

### Constraints
- $1 \le N \le 10^5$
- $-10^9 \le arr[i] \le 10^9$

### Observations
1. A majority element appears strictly more than $N/2$ times. This means there can be **at most one** majority element in any array.
2. The sum of frequencies of all other elements is strictly less than the frequency of the majority element.
3. If we sort the array and a majority element exists, it must lie at the index $\lfloor N/2 \rfloor$.

---

## 2. Interview Clarification

- **Q:** Is the majority element guaranteed to exist?
  - **A:** In some problems it is, but here we assume it is **not guaranteed**. Therefore, we must perform a verification step to confirm if our candidate actually appears more than $N/2$ times.
- **Q:** What should we return if no majority element exists?
  - **A:** Return `-1`.
- **Q:** Can the elements be negative or zero?
  - **A:** Yes, the elements can be any 32-bit signed integer.
- **Q:** Can we modify the array?
  - **A:** Modifying the array is allowed, but an optimal algorithm that runs in $O(1)$ auxiliary space without modification is highly preferred.

---

## 3. Thinking Process

### First Instinct (Brute Force)
For each element, count how many times it appears in the array.
- Run a loop with index `i` from `0` to `N-1`.
- For each `arr[i]`, start a secondary loop to count its occurrences.
- If the count is greater than $N/2$, return `arr[i]`.
- This takes $O(N^2)$ time and $O(1)$ space.

### Sorting (Better Solution 1)
If we sort the array, identical elements will become contiguous.
- Sort the array in $O(N \log N)$ time.
- The majority element (if it exists) must occupy the middle index $\lfloor N/2 \rfloor$ because its frequency is $> N/2$.
- We verify if the element at index $\lfloor N/2 \rfloor$ actually appears $> N/2$ times.
- Time Complexity: $O(N \log N)$ for sorting. Space Complexity: $O(1)$ auxiliary space.

### Hash Map (Better Solution 2)
We can use a hash map to keep track of frequencies.
- Traverse the array once and count frequencies: `unordered_map<int, int> freq`.
- During or after the traversal, check if any element has `freq[num] > N/2`.
- Time Complexity: $O(N)$ average time. Space Complexity: $O(N)$ to store the map.

### The Boyer-Moore Voting Algorithm (Optimal Insight)
Can we find the majority element in $O(N)$ time and $O(1)$ space without sorting?
Yes, using the **Boyer-Moore Voting Algorithm**.

#### The Core Intuition: Candidate Cancellation
Imagine a democratic voting process. If we pair up different numbers and let them "cancel" each other out, the majority element will survive because it has more votes than all other elements combined.
- We maintain two variables: a `candidate` and a `count` (initially 0).
- For each element `x` in the array:
  - If `count == 0`: We choose `x` as our new `candidate` and set `count = 1`.
  - Else if `x == candidate`: We increment `count`.
  - Else (i.e., `x != candidate`): We decrement `count` (representing a cancellation).
- At the end of the array, the `candidate` variable will hold the only possible candidate for the majority element.

#### Why Cancellation Works (Mathematical Proof)
Let the majority element be $M$ with frequency $F > N/2$. The remaining $N - F$ elements are not equal to $M$.
Since $F > N/2$, we have:
$$F > N - F$$
This means the count of $M$ is strictly greater than the count of all other elements combined. 
- In the worst-case scenario, every non-majority element cancels out one occurrence of $M$.
- After all $N - F$ elements have cancelled out $N - F$ occurrences of $M$, there will still be at least $F - (N - F) = 2F - N > 0$ occurrences of $M$ left.
- Therefore, $M$ is guaranteed to be the final candidate.

#### Verification Step
Since the majority element is not guaranteed to exist in the input, the survivor candidate might just be a residual element from an array that has no majority element (e.g. `[1, 2, 3]`). We must perform a second pass to count the actual occurrences of the candidate and verify if `count > N/2`.

---

## 4. Pattern Recognition

- **Pattern:** Boyer-Moore Voting / State Reduction.
- **Why this topic?** The majority requirement ($> N/2$) allows us to frame the problem as a balance of power where one element dominates all others. The Boyer-Moore algorithm is the classic pattern for majority problems.

---

## 5. Brute Force Solution

### Algorithm
1. Loop `i` from `0` to `N - 1`:
   - Set `count = 0`.
   - Loop `j` from `0` to `N - 1`:
     - If `arr[j] == arr[i]`, increment `count`.
   - If `count > N/2`, return `arr[i]`.
2. Return `-1`.

### Complexity Analysis
- **Time Complexity:** $O(N^2)$ due to nested scanning loops.
- **Space Complexity:** $O(1)$ auxiliary space.

### Brute Force C++17 Code
```cpp
#include <vector>

class Solution {
public:
    int majorityElementBrute(const std::vector<int>& arr) {
        int n = arr.size();
        for (int i = 0; i < n; ++i) {
            int count = 0;
            for (int j = 0; j < n; ++j) {
                if (arr[j] == arr[i]) {
                    count++;
                }
            }
            if (count > n / 2) {
                return arr[i];
            }
        }
        return -1;
    }
};
```

---

## 6. Better Solution (Frequency Hash Map)

### Algorithm
1. Initialize an `std::unordered_map<int, int> freq`.
2. Loop through `arr` and update counts: `freq[num]++`.
3. If at any point `freq[num] > N/2`, return `num`.
4. If the loop completes without finding such an element, return `-1`.

### Complexity Analysis
- **Time Complexity:** $O(N)$ average time.
- **Space Complexity:** $O(N)$ auxiliary space for the map.

### Better C++17 Code
```cpp
#include <vector>
#include <unordered_map>

class Solution {
public:
    int majorityElementBetter(const std::vector<int>& arr) {
        int n = arr.size();
        std::unordered_map<int, int> freq;
        
        for (int num : arr) {
            freq[num]++;
            if (freq[num] > n / 2) {
                return num;
            }
        }
        return -1;
    }
};
```

---

## 7. Optimal Solution (Boyer-Moore Voting)

### Algorithm
1. **Pass 1: Find Candidate**
   - Initialize `candidate = 0`, `count = 0`.
   - Loop through `arr`:
     - If `count == 0`, set `candidate = num` and `count = 1`.
     - Else if `num == candidate`, increment `count`.
     - Else, decrement `count`.
2. **Pass 2: Verify Candidate**
   - Reset `actual_count = 0`.
   - Loop through `arr`:
     - If `num == candidate`, increment `actual_count`.
   - If `actual_count > N / 2`, return `candidate`.
   - Else, return `-1`.

### Why this is Optimal
It runs in $O(N)$ time (exactly two linear passes) and uses $O(1)$ auxiliary space. This is the absolute minimum resource requirement for this problem.

### C++17 Optimal Code
```cpp
#include <vector>

class Solution {
public:
    int majorityElement(const std::vector<int>& arr) {
        int n = arr.size();
        if (n == 0) return -1;
        
        int candidate = 0;
        int count = 0;
        
        // Pass 1: Find the majority candidate
        for (int num : arr) {
            if (count == 0) {
                candidate = num;
                count = 1;
            } else if (num == candidate) {
                count++;
            } else {
                count--;
            }
        }
        
        // Pass 2: Verify if the candidate is indeed the majority element
        int actual_count = 0;
        for (int num : arr) {
            if (num == candidate) {
                actual_count++;
            }
        }
        
        if (actual_count > n / 2) {
            return candidate;
        }
        
        return -1; // No majority element exists
    }
};
```

---

## 8. Code Walkthrough

1. **Safety check**: `if (n == 0) return -1;` handles empty arrays safely.
2. **Pass 1 (Candidate Identification)**:
   - We iterate through `arr`.
   - When `count` drops to `0`, the current element `num` claims the candidacy (`candidate = num`, `count = 1`).
   - If the next element matches the candidate, `count` increases.
   - If it differs, they cancel out, and `count` decreases.
3. **Pass 2 (Verification)**:
   - Since Boyer-Moore only outputs a candidate, we must verify it. For example, in `[1, 2, 3]`, Boyer-Moore would output `3` as the candidate. But `3` only occurs once, which is not $> 3/2$.
   - We loop through the array again and count how many times `candidate` occurs.
   - If `actual_count > n / 2`, we return `candidate`. Otherwise, we return `-1`.

---

## 9. Dry Run

Let's dry run the optimal solution on `arr = [2, 2, 1, 1, 1, 2, 2]`.
- $N = 7$. Threshold is $N/2 = 3$.

### Pass 1: Finding Candidate

| Step | `num` | `count` before | `candidate` before | Action | `candidate` after | `count` after |
|:---:|:---:|:---:|:---:|:---|:---:|:---:|
| 1 | 2 | 0 | - | `count == 0` $\rightarrow$ set candidate | 2 | 1 |
| 2 | 2 | 1 | 2 | `num == candidate` $\rightarrow$ increment | 2 | 2 |
| 3 | 1 | 2 | 2 | `num != candidate` $\rightarrow$ decrement | 2 | 1 |
| 4 | 1 | 1 | 2 | `num != candidate` $\rightarrow$ decrement | 2 | 0 |
| 5 | 1 | 0 | 2 | `count == 0` $\rightarrow$ set candidate | 1 | 1 |
| 6 | 2 | 1 | 1 | `num != candidate` $\rightarrow$ decrement | 1 | 0 |
| 7 | 2 | 0 | 1 | `count == 0` $\rightarrow$ set candidate | 2 | 1 |

Candidate at end of Pass 1: **2**

### Pass 2: Verification
- Count occurrences of `2` in `[2, 2, 1, 1, 1, 2, 2]`:
  - `num = 2` $\rightarrow$ count = 1
  - `num = 2` $\rightarrow$ count = 2
  - `num = 1` $\rightarrow$ count = 2
  - `num = 1` $\rightarrow$ count = 2
  - `num = 1` $\rightarrow$ count = 2
  - `num = 2` $\rightarrow$ count = 3
  - `num = 2` $\rightarrow$ count = 4
- Actual count of `2` is `4`.
- Is `4 > 7/2` (3)? **Yes**.
- Return **`2`**.

---

## 10. Complexity Analysis

- **Time Complexity:** 
  - **Best/Average/Worst Case:** $O(N)$ since we perform at most two sequential passes over the array.
- **Space Complexity:**
  - **Auxiliary Space:** $O(1)$ because we only use a few tracking variables.

---

## 11. Common Mistakes

1. **Skipping the verification step**: If the majority element is not guaranteed to exist, failing to verify the candidate will lead to wrong answers (e.g. returning `3` for `[1, 2, 3]`).
2. **Incorrectly updating candidate**: Resetting `count = 0` but forgetting to set `candidate = num` on the same step. The state transition must be atomic: when `count` is 0, the current element becomes the candidate and count becomes 1.
3. **Using wrong check for majority threshold**: Using `count >= N / 2` instead of `count > N / 2`. The majority element must appear **strictly more** than $N/2$ times.

---

## 12. Edge Cases

| Edge Case | Input Example | Expected Output | Why it matters |
|:---|:---|:---|:---|
| **Single Element** | `[5]` | `5` | Loop runs once, candidate is verified correctly. |
| **No Majority Element** | `[1, 2, 3]` | `-1` | Candidate 3 fails verification. |
| **Alternating Elements** | `[1, 2, 1, 2, 1]` | `1` | Count drops to 0 repeatedly, but correct candidate wins. |
| **Two Elements (No Majority)**| `[1, 2]` | `-1` | Verify threshold $2/2 = 1$; actual count must be $> 1$. |

---

## 13. Interview Explanation

"To find the majority element (the element appearing strictly more than $N/2$ times), we can use the **Boyer-Moore Voting Algorithm**. 

The key insight is that since the majority element occupies more than half of the array, its count is greater than all other elements combined. If we pair up different elements and cancel them out, the majority element is guaranteed to be the survivor. 

We scan the array maintaining a candidate and a counter. If the counter is zero, we choose the current element as the candidate. If the current element matches the candidate, we increment the counter; otherwise, we decrement it. 

After completing the scan, we get a candidate. Since a majority element is not guaranteed to exist, we perform a second pass to count the actual occurrences of this candidate. If its frequency is strictly greater than $N/2$, we return it; otherwise, we return -1. This approach runs in $O(N)$ time and $O(1)$ space."

---

## 14. Follow-up Questions

- **Q:** How would you modify the algorithm if we need to find all elements that appear more than $\lfloor N/3 \rfloor$ times?
  - **A:** This is the **Majority Element II** problem. An array can contain at most two elements that appear more than $N/3$ times. We maintain **two candidates** and **two counters** and apply the same cancellation logic.
- **Q:** Can we solve the majority element problem in a distributed system where the array is partitioned across multiple machines?
  - **A:** Yes, each machine can find its local candidate and frequency using Boyer-Moore. We then send all candidates to a central node, which queries the distributed partitions to find the global counts of these candidates.

---

## 15. Variations

- **Majority Element II:** Find elements appearing $> N/3$ times.
- **Check if Array is Monotonic Majority:** Check if there is an element that appears more than $N/2$ times in any contiguous subarray of length $\ge 3$.

---

## 16. Revision Notes

- **Core Boyer-Moore Rule:** Different elements cancel each other out.
- **Verification Rule:** Always run Pass 2 unless the problem guarantees majority element existence.
- **Complexity:** $O(N)$ Time, $O(1)$ Space.
- **Threshold:** Strictly $> N/2$ occurrences.

---

## 17. Memorization Version

- **Pass 1:** 
  - For each `x` in `arr`:
    - `if (count == 0) { candidate = x; count = 1; }`
    - `else if (x == candidate) count++;`
    - `else count--;`
- **Pass 2:** 
  - Count actual occurrences of `candidate`. 
  - Return `candidate` if `count > N / 2` else `-1`.
- **Complexity:** $O(N)$ time, $O(1)$ space.
