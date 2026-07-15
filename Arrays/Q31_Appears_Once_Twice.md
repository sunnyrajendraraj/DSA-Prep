# Find the Element that Appears Once (Others Appear Twice)

## 1. Problem Statement

Given a non-empty array of integers `nums`, every element appears twice except for one. Find that single one.

### Input
* `nums`: A vector of integers where $1 \le \text{nums.length} \le 3 \cdot 10^4$.
* Each element in the array appears twice except for one element which appears exactly once.

### Output
* Returns the integer that appears exactly once.

### Constraints
* $-3 \cdot 10^4 \le \text{nums}[i] \le 3 \cdot 10^4$
* You must implement a solution with a linear runtime complexity $O(n)$ and use only constant extra space $O(1)$.

### Observations
1. If we sum up all unique elements and multiply by 2, then subtract the sum of the array, the difference is the single element. However, this could lead to overflow and requires extra space to find unique elements.
2. Sorting the array would bring duplicate elements adjacent to each other, but sorting takes $O(n \log n)$ time.
3. A hash map can track frequencies, but it takes $O(n)$ space.
4. We need a way to "cancel out" numbers that appear in pairs. Bitwise XOR ($\oplus$) has this exact properties.

---

## 2. Interview Clarification

Before writing any code, clarify these points with your interviewer:
1. **Can the array be empty?**
   * *Answer*: No, the constraints specify it is non-empty (at least 1 element).
2. **Is the array sorted?**
   * *Answer*: No, it can be in any order. If it were sorted, we could use binary search in $O(\log n)$ time.
3. **Are there negative numbers?**
   * *Answer*: Yes, integers can be negative, positive, or zero.
4. **Is it guaranteed that exactly one element appears once and all others appear exactly twice?**
   * *Answer*: Yes, this is guaranteed.

---

## 3. Thinking Process

### First Instinct (Brute Force)
* Compare each element with every other element in the array. If an element does not have a duplicate, return it. This uses nested loops and takes $O(n^2)$ time.

### Better Instinct (Hashing / Sorting)
* **Sorting**: Sort the array. Then, traverse the array with a step of 2. If `nums[i] != nums[i+1]`, then `nums[i]` is the single element. Time complexity: $O(n \log n)$ due to sorting. Space complexity: $O(1)$ or $O(n)$ depending on the sorting algorithm.
* **Hashing**: Store the frequency of each element in a hash map. Iterate through the map to find the element with frequency 1. Time complexity: $O(n)$, but Space complexity: $O(n)$ for storing elements in the map.

### Optimal Insight (Bit Manipulation)
* We want $O(n)$ time and $O(1)$ space. This points directly to bitwise operations.
* The XOR operator ($\oplus$) has three properties that are extremely useful here:
  1. **Commutative**: $A \oplus B = B \oplus A$ (Order of operations does not matter).
  2. **Associative**: $A \oplus (B \oplus C) = (A \oplus B) \oplus C$.
  3. **Identity**: $A \oplus 0 = A$.
  4. **Self-inverse**: $A \oplus A = 0$.
* If we XOR all elements in the array:
  $$\text{Result} = \text{nums}[0] \oplus \text{nums}[1] \oplus \dots \oplus \text{nums}[n-1]$$
  Since order doesn't matter, we can group all duplicate pairs together:
  $$\text{Result} = (x_1 \oplus x_1) \oplus (x_2 \oplus x_2) \oplus \dots \oplus (y)$$
  where $y$ is the single element and $x_i$ are the duplicates.
  Since $x_i \oplus x_i = 0$:
  $$\text{Result} = 0 \oplus 0 \oplus \dots \oplus y = y$$
* This gives us the single element using $O(n)$ time and $O(1)$ auxiliary space.

---

## 4. Pattern Recognition

* **Topic**: Bit Manipulation / Array traversal.
* **Pattern**: Duplicate cancellation using Bitwise XOR.
* **Why this works**: XOR is a "difference" operator at the bit level. When two identical numbers are XORed, every matching set bit cancels out to `0`. A single unmatched number will retain its bits intact when XORed with `0`.

---

## 5. Brute Force Solution

### Algorithm
1. Loop through each element `nums[i]` of the array.
2. For each element, initialize a counter `count = 0`.
3. Loop through the array again with index `j` to count occurrences of `nums[i]`.
4. If `count == 1` after the inner loop finishes, return `nums[i]`.

### Dry Run
Input: `nums = [4, 1, 2, 1, 2]`
* `i = 0` (`nums[i] = 4`): Inner loop checks all elements. Counts = 1. Returns 4.

### Complexity Analysis
* **Time Complexity**: $O(n^2)$ due to nested loops.
* **Space Complexity**: $O(1)$ auxiliary space.

### Pros and Cons
* **Pros**: Simple to understand, no extra space.
* **Cons**: Extremely slow for large inputs ($N = 3 \cdot 10^4$ leads to $9 \cdot 10^8$ operations).

### C++17 Code
```cpp
#include <vector>

class SolutionBrute {
public:
    int singleNumber(const std::vector<int>& nums) {
        int n = nums.size();
        for (int i = 0; i < n; ++i) {
            int count = 0;
            for (int j = 0; j < n; ++j) {
                if (nums[i] == nums[j]) {
                    count++;
                }
            }
            // If the element appears only once
            if (count == 1) {
                return nums[i];
            }
        }
        return -1; // Fallback
    }
};
```

---

## 6. Better Solution

### Algorithm (Hash Map)
1. Initialize an unordered map `frequencies` to store `element -> count`.
2. Iterate through `nums` and increment `frequencies[num]`.
3. Iterate through `nums` again (or the map keys) and return the key which has a value of `1`.

### Dry Run
Input: `nums = [4, 1, 2, 1, 2]`
* Step 1: Map becomes `{4: 1, 1: 2, 2: 2}`.
* Step 2: Iterate map. Key `4` has value `1`. Return `4`.

### Complexity Analysis
* **Time Complexity**: $O(n)$ on average because hash map insertion and lookup are $O(1)$.
* **Space Complexity**: $O(n)$ to store up to $n/2 + 1$ unique keys.

### C++17 Code
```cpp
#include <vector>
#include <unordered_map>

class SolutionBetter {
public:
    int singleNumber(const std::vector<int>& nums) {
        std::unordered_map<int, int> frequencies;
        
        // Count frequencies of each number
        for (int num : nums) {
            frequencies[num]++;
        }
        
        // Find the number with frequency 1
        for (const auto& [num, count] : frequencies) {
            if (count == 1) {
                return num;
            }
        }
        
        return -1;
    }
};
```

---

## 7. Optimal Solution

### Algorithm
1. Initialize a variable `xor_sum = 0`.
2. Loop through every element `num` in `nums`.
3. Update `xor_sum = xor_sum ^ num`.
4. Return `xor_sum`.

### Why this is Optimal
* It visits each element exactly once: $O(n)$ time.
* It uses only one integer variable to maintain the running XOR sum: $O(1)$ space.
* This meets the strict constraints of the problem description perfectly.

### C++17 Code
```cpp
#include <vector>

class SolutionOptimal {
public:
    int singleNumber(const std::vector<int>& nums) {
        int xor_sum = 0;
        
        // XOR all elements of the array
        for (int num : nums) {
            xor_sum ^= num;
        }
        
        return xor_sum;
    }
};
```

---

## 8. Code Walkthrough

* **Line 5**: `int xor_sum = 0;`
  We initialize our accumulator to 0. Since $0 \oplus X = X$, starting with 0 is neutral and correct.
* **Line 8**: `for (int num : nums) {`
  A range-based loop traverses each element in the input vector from index `0` to `n-1`.
* **Line 9**: `xor_sum ^= num;`
  This is equivalent to `xor_sum = xor_sum ^ num`. The bitwise XOR operation is executed.
* **Line 12**: `return xor_sum;`
  After processing all elements, all pairs have canceled out to `0`, leaving only the unique element.

---

## 9. Dry Run

Input: `nums = [4, 1, 2, 1, 2]`

| Step | Current Element (`num`) | Bitwise Representation of `num` | Operation | Running `xor_sum` (Decimal) | Running `xor_sum` (Binary) |
| :--- | :--- | :--- | :--- | :--- | :--- |
| Initial | - | - | - | 0 | `0000` |
| 1 | 4 | `0100` | `0000 ^ 0100` | 4 | `0100` |
| 2 | 1 | `0001` | `0100 ^ 0001` | 5 | `0101` |
| 3 | 2 | `0010` | `0101 ^ 0010` | 7 | `0111` |
| 4 | 1 | `0001` | `0111 ^ 0001` | 6 | `0110` |
| 5 | 2 | `0010` | `0110 ^ 0010` | 4 | `0100` |

Output: `4`.

---

## 10. Complexity Analysis

| Approach | Time Complexity | Space Complexity | Notes |
| :--- | :--- | :--- | :--- |
| Brute Force | $O(n^2)$ | $O(1)$ | Double loop check |
| Sorting | $O(n \log n)$ | $O(1)$ or $O(n)$ | Modifies input or needs copy |
| Hash Map | $O(n)$ | $O(n)$ | Extra space proportional to unique elements |
| **XOR Optimal** | **$O(n)$** | **$O(1)$** | **Optimal, linear time, constant space** |

---

## 11. Common Mistakes

1. **Initializing XOR sum with the first element and loop starting from 0**:
   * If you start `xor_sum = nums[0]` and then iterate from `i = 0`, you will XOR `nums[0]` with itself, canceling it. If you initialize with `nums[0]`, you must start the loop from `i = 1`.
   * Standard practice is initializing `xor_sum = 0` and iterating the entire array.
2. **Confusing XOR (`^`) with Exponentiation**:
   * In C++, `^` is the bitwise XOR operator, not exponentiation.
3. **Not recognizing overflow issues in alternate mathematical solutions**:
   * Summing up elements ($2 \times \text{Sum of Unique} - \text{Sum of All}$) can overflow 32-bit integers if the array is large or contains big values.

---

## 12. Edge Cases

| Edge Case | Example | Why it matters | Solution Behavior |
| :--- | :--- | :--- | :--- |
| Single element array | `[1]` | Minimum size boundary. Loop runs once. | Returns `1` (Correct) |
| Negative numbers | `[-1, -2, -1]` | Verifies sign bit behavior in XOR. | Returns `-2` (Correct) |
| Zero present | `[0, 3, 3]` | Zero should cancel with nothing or not affect XOR. | Returns `0` (Correct) |
| Large values | `[2^30, 5, 2^30]` | Standard integers, fits within `int` limits. | Returns `5` (Correct) |

---

## 13. Interview Explanation

"To solve this problem efficiently in $O(n)$ time and $O(1)$ auxiliary space, we can leverage the properties of the bitwise XOR operation. 
XOR has two very important properties: first, any number XORed with itself is zero ($A \oplus A = 0$), and second, any number XORed with zero remains unchanged ($A \oplus 0 = A$). 

If we initialize a variable `xor_sum` to `0` and XOR it with every element in the array, the duplicate elements will pair up and cancel each other out to `0`. The single element, having no duplicate, will be XORed with `0` and will remain as the final value. This approach runs in $O(n)$ time since we iterate through the array exactly once, and uses $O(1)$ space because we only need a single integer variable."

---

## 14. Follow-up Questions

1. **What if the array is sorted? Can we do better than $O(n)$?**
   * *Answer*: Yes, if the array is sorted, we can use Binary Search to find the single element in $O(\log n)$ time. We look at pairs: before the single element, pairs start at even indices `(even, odd)`. After the single element, they shift to `(odd, even)`.
2. **What if there are two single numbers (i.e. two numbers appear once, others twice)?**
   * *Answer*: This is a classic variation (Single Number III). We XOR all numbers to get `X ^ Y`. We find a set bit in `X ^ Y` to partition the numbers into two groups and resolve each group separately using the single number algorithm.

---

## 15. Variations

1. **Single Number II**: Every element appears three times except one which appears once. (Requires bit counting modulo 3).
2. **Single Number III**: Two elements appear once, all others appear twice. (Requires bitwise grouping).
3. **Find the Duplicate Number**: Every number appears once except one which appears twice. (Can be solved using Floyd's Cycle Detection/Tortoise and Hare).

---

## 16. Revision Notes

* **XOR Rules**:
  * $A \oplus A = 0$
  * $A \oplus 0 = A$
  * $A \oplus B \oplus A = B$
* **Complexity**: $O(n)$ Time, $O(1)$ Space.
* **Mnemonic**: "Pairs cancel out, single stays."

---

## 17. Memorization Version

```cpp
// O(n) time, O(1) space. XOR cancels duplicates.
int singleNumber(vector<int>& nums) {
    int ans = 0;
    for (int x : nums) ans ^= x;
    return ans;
}
```
