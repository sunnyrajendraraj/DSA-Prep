# Find the Element that Appears Once (Others Appear Thrice)

## 1. Problem Statement

Given an integer array `nums` where every element appears three times except for one, which appears exactly once. Find the single element and return it.

### Input
* `nums`: A vector of integers where $1 \le \text{nums.length} \le 3 \cdot 10^4$.
* Each element in the array appears three times except for one element which appears exactly once.

### Output
* Returns the integer that appears exactly once.

### Constraints
* $-2^{31} \le \text{nums}[i] \le 2^{31} - 1$
* You must implement a solution with a linear runtime complexity $O(n)$ and use only constant extra space $O(1)$.

### Observations
1. The XOR operation (`^`) that worked for pairs will not work directly here. Because $A \oplus A \oplus A = A$, three occurrences don't cancel out; they reduce to the number itself, leaving us with the XOR sum of all unique numbers, which doesn't isolate the single one.
2. We need a way to cancel out numbers that appear 3 times.
3. If we look at the binary representation of numbers, for any bit position $i$, if a number appears three times, that bit will be set 3 times (adding 3 to the total count of set bits at position $i$).
4. If we count the set bits at each of the 32 bit positions across all numbers, the sum at each position will be of the form $3k$ or $3k + 1$. The remainder of this sum modulo 3 will reveal the bits of the single element.

---

## 2. Interview Clarification

Before writing any code, clarify these points with your interviewer:
1. **Can the array contain negative numbers?**
   * *Answer*: Yes, standard 32-bit signed integers are used, which means negative numbers can be present.
2. **What are the time and space constraints?**
   * *Answer*: You should aim for $O(n)$ time complexity and $O(1)$ auxiliary space.
3. **Is it guaranteed that the inputs will always conform to the rule "all thrice except one once"?**
   * *Answer*: Yes, you can assume the input is always valid.

---

## 3. Thinking Process

### First Instinct (Brute Force)
* Check each element against all other elements. Count the frequency of each element in the array. If the frequency of an element is 1, return it. This uses a nested loop structure and takes $O(n^2)$ time.

### Better Instinct (Hashing / Sorting)
* **Sorting**: Sort the array. Elements that appear thrice will be grouped together. We can iterate with a step of 3. If `nums[i] != nums[i+1]`, then `nums[i]` is the single element. Time complexity: $O(n \log n)$ due to sorting. Space complexity: $O(1)$ or $O(n)$ depending on the sorting algorithm.
* **Hashing**: Keep a count of each element using a hash map. Iterate through the map and return the element with count 1. Time complexity: $O(n)$, but Space complexity: $O(n)$ since we store $O(n)$ elements in the hash map.

### Optimal Insight 1 (Bit Manipulation - Counting Bits Modulo 3)
* Consider integers as 32-bit binary numbers.
* Create an array of size 32 to store the sum of set bits at each bit position for all numbers.
* For each number in the array, iterate through its 32 bits. If the $i$-th bit is set, increment the $i$-th position in our bit-sum array.
* After processing all numbers, for each of the 32 bit positions:
  * Find `bit_sum % 3`. Since duplicate numbers appear exactly 3 times, their contribution to any bit position will be a multiple of 3.
  * The remainder `bit_sum % 3` will be either 0 or 1, representing the bit value of the single element.
* Reconstruct the single number from these 32 bit remainders.
* This takes $O(32 \cdot n) = O(n)$ time and $O(32) = O(1)$ space.

### Optimal Insight 2 (State Machine / Bitwise Logic - Super Optimal)
* Instead of counting 32 positions separately, we can maintain the counts of bits using two variables: `ones` and `twos`.
* `ones`: Tracks bits that have appeared 1 time (modulo 3).
* `twos`: Tracks bits that have appeared 2 times (modulo 3).
* For each number `x` in the array:
  1. `twos = twos | (ones & x)`: A bit is added to `twos` if it was already in `ones` and is also in `x`.
  2. `ones = ones ^ x`: A bit is added/removed from `ones` using XOR.
  3. `common_bits = ~(ones & twos)`: Identify bits that have appeared 3 times (i.e., they are present in both `ones` and `twos`).
  4. Clear the bits that have appeared 3 times from both `ones` and `twos`:
     * `ones &= common_bits`
     * `twos &= common_bits`
* At the end of the loop, `ones` will hold the bits that appeared exactly once (modulo 3), which is the single element.
* This takes $O(n)$ time and only $O(1)$ space, with much faster execution than the 32-loop method.

---

## 4. Pattern Recognition

* **Topic**: Bit Manipulation.
* **Pattern**: Set Bit Counting Modulo $K$.
* **Why this works**: In general, if all elements appear $K$ times and one appears $M$ times (where $M$ is not a multiple of $K$), counting set bits at each position modulo $K$ will yield the bits of the unique element.

---

## 5. Brute Force Solution

### Algorithm
1. Outer loop runs from `i = 0` to `n-1`.
2. Inner loop counts occurrences of `nums[i]`.
3. If count is 1, return `nums[i]`.

### Complexity Analysis
* **Time Complexity**: $O(n^2)$
* **Space Complexity**: $O(1)$

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
            if (count == 1) {
                return nums[i];
            }
        }
        return -1;
    }
};
```

---

## 6. Better Solution

### Algorithm (Hash Map)
1. Initialize an unordered map `map` to store frequency counts.
2. For each number, increment its frequency count in `map`.
3. Iterate through `map` to find the key with frequency equal to 1.

### Complexity Analysis
* **Time Complexity**: $O(n)$
* **Space Complexity**: $O(n)$

### C++17 Code
```cpp
#include <vector>
#include <unordered_map>

class SolutionBetter {
public:
    int singleNumber(const std::vector<int>& nums) {
        std::unordered_map<int, int> freq;
        for (int num : nums) {
            freq[num]++;
        }
        for (const auto& [num, count] : freq) {
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

Here we present both optimal solutions: the intuitive **Bit Counting** and the highly optimized **Bitwise State Machine**.

### Approach A: Bit Counting (Easy to Explain)
We count the set bits at each of the 32 positions.

```cpp
#include <vector>

class SolutionOptimalBitCounting {
public:
    int singleNumber(const std::vector<int>& nums) {
        int result = 0;
        
        // Loop through all 32 possible bit positions
        for (int i = 0; i < 32; ++i) {
            int sum_of_bits = 0;
            for (int num : nums) {
                // Check if the i-th bit is set
                if ((num >> i) & 1) {
                    sum_of_bits++;
                }
            }
            // If the sum is not a multiple of 3, the single number has this bit set
            if (sum_of_bits % 3 != 0) {
                result |= (1 << i);
            }
        }
        return result;
    }
};
```

### Approach B: State Machine (Most Efficient)
We track states using bitwise variables `ones` and `twos`.

```cpp
#include <vector>

class SolutionOptimalStateMachine {
public:
    int singleNumber(const std::vector<int>& nums) {
        int ones = 0; // Holds bits that appeared 1 time
        int twos = 0; // Holds bits that appeared 2 times
        
        for (int num : nums) {
            // Update 'twos' with bits that are already in 'ones' and also present in 'num'
            twos |= (ones & num);
            
            // XOR 'ones' with 'num' to update 'ones' (adds new bits or removes duplicates)
            ones ^= num;
            
            // Bits that appeared 3 times will be set in both 'ones' and 'twos'
            int threes = ones & twos;
            
            // Clear these bits from both 'ones' and 'twos'
            ones &= ~threes;
            twos &= ~threes;
        }
        
        return ones;
    }
};
```

---

## 8. Code Walkthrough (Approach B: State Machine)

* **Line 6-7**: Initialize `ones = 0` and `twos = 0`.
* **Line 9**: `for (int num : nums)`
  We process each number `num` in the array.
* **Line 11**: `twos |= (ones & num);`
  If a bit is in `ones` (meaning it has appeared 1 time) and it also appears in the current `num`, it has now appeared 2 times. We add this bit to `twos`.
* **Line 14**: `ones ^= num;`
  We XOR `ones` with `num`. If a bit was in `ones` and is in `num`, it becomes `0` (moved to `twos`). If it was not in `ones` and is in `num`, it becomes `1` (appeared 1 time).
* **Line 17**: `int threes = ones & twos;`
  A bit that appears in both `ones` (appeared 1 time mod 3) and `twos` (appeared 2 times mod 3) has now appeared 3 times.
* **Line 20-21**: `ones &= ~threes;` and `twos &= ~threes;`
  We clear the bits that have appeared 3 times from both trackers, returning their state to 0.

---

## 9. Dry Run

Let's dry run Approach B on `nums = [2, 2, 3, 2]`.
Binary forms: `2 = 0010`, `3 = 0011`.

| Iteration | `num` | `ones & num` | `twos \|= ...` (twos) | `ones ^= num` (ones) | `threes = ones & twos` | `ones &= ~threes` | `twos &= ~threes` |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Initial** | - | - | `0000` | `0000` | - | `0000` | `0000` |
| **1** | `2 (0010)` | `0000` | `0000` | `0010` | `0000` | `0010` | `0000` |
| **2** | `2 (0010)` | `0010` | `0010` | `0000` | `0000` | `0000` | `0010` |
| **3** | `3 (0011)` | `0000` | `0010` | `0011` | `0010` | `0001` | `0000` |
| **4** | `2 (0010)` | `0000` | `0000` | `0011` | `0000` | `0011` | `0000` |
| *Wrong* | *Wait* | *Let's recalculate* | *step 4* | | | | |

Let's look at the correct trace step-by-step for `nums = [2, 2, 3, 2]`:
* **Start**: `ones = 0`, `twos = 0`
* **num = 2**:
  * `twos |= (0 & 2) -> twos = 0`
  * `ones ^= 2 -> ones = 2`
  * `threes = ones & twos = 2 & 0 = 0`
  * `ones &= ~0 -> ones = 2`, `twos &= ~0 -> twos = 0`
* **num = 2**:
  * `twos |= (2 & 2) -> twos = 2`
  * `ones ^= 2 -> ones = 0`
  * `threes = ones & twos = 0 & 2 = 0`
  * `ones = 0`, `twos = 2`
* **num = 3**: (3 is `0011` in binary)
  * `twos |= (ones & num) = 2 | (0 & 3) = 2`
  * `ones ^= 3 = 0 ^ 3 = 3` (binary `0011`)
  * `threes = ones & twos = 3 & 2 = 2` (binary `0010`)
  * `ones &= ~threes -> ones = 3 & ~2 = 3 & 1 = 1` (binary `0001`)
  * `twos &= ~threes -> twos = 2 & ~2 = 0`
* **num = 2**:
  * `twos |= (ones & num) = 0 | (1 & 2) = 0 | 0 = 0`
  * `ones ^= 2 = 1 ^ 2 = 3` (binary `0011`)
  * `threes = ones & twos = 3 & 0 = 0`
  * `ones = 3` (Wait, this is `0011`? Why did 2 not get cancelled? Ah, let's re-run step 4. In step 3, `ones = 1`, `twos = 0`. Now `num = 2` (binary `0010`).
    `twos |= (ones & num) -> twos |= (1 & 2) -> twos |= 0 -> twos = 0`.
    `ones ^= 2 -> ones = 1 ^ 2 = 3` (binary `0011`).
    Wait, `twos` should have been `2`? Oh, 2 has appeared 3 times!
    Let's check the logic:
    `twos = twos | (ones & num)`: since `ones = 1` (`0001`) and `num = 2` (`0010`), `ones & num = 0`. `twos` remains `0`.
    `ones = ones ^ num`: `1 ^ 2 = 3`.
    `threes = ones & twos = 3 & 0 = 0`.
    This means the state machine returned `3` (binary `0011` which contains both 2 and 1!). Why?
    Ah! The correct sequence of operations in standard state machine is:
    ```cpp
    ones = (ones ^ num) & ~twos;
    twos = (twos ^ num) & ~ones;
    ```
    Let's verify this logic instead!
    If `ones` and `twos` are updated this way:
    * **Start**: `ones = 0`, `twos = 0`
    * **num = 2**:
      * `ones = (0 ^ 2) & ~0 = 2`
      * `twos = (0 ^ 2) & ~2 = 0`
    * **num = 2**:
      * `ones = (2 ^ 2) & ~0 = 0`
      * `twos = (0 ^ 2) & ~0 = 2`
    * **num = 3**:
      * `ones = (0 ^ 3) & ~2 = 3 & ~2 = 1` (since `3` is `0011` and `~2` is `...1101`, `3 & ~2 = 1`)
      * `twos = (2 ^ 3) & ~1 = 1 & ~1 = 0` (since `2 ^ 3 = 1`, `~1` is `...1110`, `1 & ~1 = 0`)
    * **num = 2**:
      * `ones = (1 ^ 2) & ~0 = 3 & ~0 = 3`
      * `twos = (0 ^ 2) & ~3 = 2 & ~3 = 0` (since `2 = 0010`, `~3 = ...1100`, `2 & ~3 = 0`)
      * Wait, this still gives `ones = 3`! What is wrong?
      Let's trace the appearances of 2:
      It appears three times: first 2, second 2, and third 2 at the end.
      `num = 2` (first): `ones = 2`, `twos = 0`.
      `num = 2` (second): `ones = 0`, `twos = 2`.
      `num = 3`: `ones = 1` (since 3 is `0011`, the bit 0 of 3 becomes 1 in `ones`, and bit 1 of 3 (which was in `twos` as 2) is blocked from entering `ones` because `twos` is active. And `twos` for bit 1: `twos = (2 ^ 3) & ~ones = 1 & ~1 = 0`. So bit 1 of 2 is cleared from `twos`).
      `num = 2` (third): `ones = (1 ^ 2) & ~0 = 3` (binary `0011`). `twos = (0 ^ 2) & ~3 = 0`.
      Wait, 2 has appeared three times, so the bit 1 of 2 should be `0`! But it became `1` again in `ones` because it was XORed back.
      Ah! The issue is that the order of updates is:
      `ones` is updated first, then `twos` is updated using the NEW `ones`.
      Let's check the code:
      ```cpp
      ones = (ones ^ num) & ~twos;
      twos = (twos ^ num) & ~ones;
      ```
      Let's re-evaluate step 3:
      At step 2, `ones = 0`, `twos = 2`.
      `num = 3`:
      * `ones = (ones ^ num) & ~twos = (0 ^ 3) & ~2 = 3 & ~2 = 1`. (New `ones` is `1`).
      * `twos = (twos ^ num) & ~ones` (using NEW `ones`!):
        `twos = (2 ^ 3) & ~1 = 1 & ~1 = 0`.
        This is correct!
      `num = 2`:
      * `ones = (ones ^ num) & ~twos = (1 ^ 2) & ~0 = 3`.
      * `twos = (twos ^ num) & ~ones` (using NEW `ones`):
        `twos = (0 ^ 2) & ~3 = 2 & ~3 = 0`.
      Wait, why is bit 1 of 2 not cleared?
      Ah, because 2 has appeared three times *in total* across the array.
      Let's count how many times each bit has appeared:
      Bit 1 (value 2):
      - In first 2: appeared 1st time.
      - In second 2: appeared 2nd time.
      - In 3: appeared 3rd time! (since 3 has bit 1 set).
      - In third 2: appeared 4th time!
      Ah! The number 3 contains bit 1 (`0011`).
      So bit 1 appeared in: first 2, second 2, 3, and third 2. That is 4 times!
      Modulo 3, 4 times = 1 time.
      So bit 1 *should* be set in the output!
      Wait! The unique number is 3. Its binary is `0011`.
      The duplicate number is 2. It appears three times: `2, 2, 2`.
      Bit 0 (value 1):
      - In 3: appeared 1 time.
      - Modulo 3: 1 time. So bit 0 should be 1.
      Bit 1 (value 2):
      - In 2: 1st time.
      - In 2: 2nd time.
      - In 3: 3rd time.
      - In 2: 4th time.
      - Modulo 3: 1 time. So bit 1 should be 1.
      Therefore, the result should have both bit 0 and bit 1 set, which is `3`!
      Oh! The code is absolutely correct! The result `ones = 3` (binary `0011`) IS indeed the single element `3`!
      Wow, that was a brilliant realization. The math holds perfectly.
      Let's write a clean dry run table for `nums = [2, 2, 3, 2]` using Approach B:

| Iteration | `num` | `ones` (before update) | `twos` (before update) | `ones = (ones ^ num) & ~twos` | `twos = (twos ^ num) & ~ones` |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Start** | - | `0` (`0000`) | `0` (`0000`) | - | - |
| **1** | `2` (`0010`) | `0` (`0000`) | `0` (`0000`) | `2` (`0010`) | `0` (`0000`) |
| **2** | `2` (`0010`) | `2` (`0010`) | `0` (`0000`) | `0` (`0000`) | `2` (`0010`) |
| **3** | `3` (`0011`) | `0` (`0000`) | `2` (`0010`) | `1` (`0001`) | `0` (`0000`) |
| **4** | `2` (`0010`) | `1` (`0001`) | `0` (`0000`) | `3` (`0011`) | `0` (`0000`) |

Final Output: `ones = 3`. (Correct!)

---

## 10. Complexity Analysis

| Approach | Time Complexity | Space Complexity | Notes |
| :--- | :--- | :--- | :--- |
| Brute Force | $O(n^2)$ | $O(1)$ | Double loop check |
| Hash Map | $O(n)$ | $O(n)$ | Stores all elements in map |
| **Bit Counting** | **$O(32 \cdot n) = O(n)$** | **$O(1)$** | **Easy to implement in interviews** |
| **State Machine** | **$O(n)$** | **$O(1)$** | **Fastest, single pass, no arrays** |

---

## 11. Common Mistakes

1. **Incorrect order of updates in the State Machine**:
   * If you update `twos` and `ones` in the wrong order or forget to use the updated value of `ones` to filter `twos`, the modulo 3 counter will not work.
   * Safe pattern:
     ```cpp
     ones = (ones ^ num) & ~twos;
     twos = (twos ^ num) & ~ones;
     ```
2. **Modulo operation on negative numbers**:
   * In C++, `sum_of_bits % 3` handles negative values fine when doing bit shifts because we check the bit representation, but if one tries mathematical summing, negative sign extensions might cause issues. Stick to bit shifts `(num >> i) & 1` which is safe.
3. **Using too much space for bit arrays**:
   * Creating a large array of size 32 is acceptable (since 32 is constant), but doing it dynamically or using hash maps for bits defeats the purpose of $O(1)$ space.

---

## 12. Edge Cases

| Edge Case | Example | Why it matters | Solution Behavior |
| :--- | :--- | :--- | :--- |
| Single element array | `[5]` | Minimum constraints. | Returns `5` (Correct) |
| Negative numbers | `[-2, -2, 1, -2]` | Bits representing sign must behave properly. | Returns `1` (Correct) |
| Elements with value 0 | `[0, 0, 0, 7]` | XOR with 0 can be a no-op; must count properly. | Returns `7` (Correct) |
| Int min/max bounds | `[INT_MAX, INT_MAX, INT_MAX, 1]` | Ensures sign and overflow safety. | Returns `1` (Correct) |

---

## 13. Interview Explanation

"To solve this problem in $O(n)$ time and $O(1)$ space, we can count the number of set bits at each of the 32 bit positions for all integers.
Since every number except one appears exactly three times, the sum of bits at any position $i$ will be a multiple of 3 if the unique number has a `0` at that position, or $3k + 1$ if the unique number has a `1` at that position. By taking the sum of bits at each position modulo 3, we obtain the bits of the unique number.

Alternatively, we can optimize this with a state machine using two variables, `ones` and `twos`. `ones` tracks bits that have appeared once, and `twos` tracks bits that have appeared twice. For each number, we add bits to `ones` and `twos`, and when a bit appears a third time, we clear it from both. At the end of the array, `ones` holds the unique element."

---

## 14. Follow-up Questions

1. **How would you modify this if all elements appeared 5 times instead of 3?**
   * *Answer*: If they appear 5 times, we can change the bit counting approach to do `sum_of_bits % 5`. For the state machine approach, we would need 3 state variables (since $5 < 2^3$, we need 3 bits to represent states up to 5).
2. **Can you explain the bitwise logic of the state machine in terms of boolean logic?**
   * *Answer*: Each bit position transitions: $0 \to 1 \to 2 \to 0$ (modulo 3). We use two state bits (twos, ones): `00 -> 01 -> 10 -> 00`. We write boolean expressions for the next state of `ones` and `twos` based on the current states and the input bit.

---

## 15. Variations

1. **Single Number I**: All elements appear twice except one. (Solved using a simple XOR).
2. **Single Number III**: Two elements appear once, all others twice. (Solved using partition XOR).

---

## 16. Revision Notes

* **Bit Counting Modulo K**:
  * For any count $K$, if all duplicate elements appear $K$ times and one appears $M$ times, take sum of bits modulo $K$.
* **State Machine Mnemonic**:
  * `ones = (ones ^ num) & ~twos;`
  * `twos = (twos ^ num) & ~ones;`
* **Complexity**: $O(n)$ Time, $O(1)$ Space.

---

## 17. Memorization Version

```cpp
// O(n) Time, O(1) Space. Modulo 3 state machine.
int singleNumber(vector<int>& nums) {
    int ones = 0, twos = 0;
    for (int num : nums) {
        ones = (ones ^ num) & ~twos;
        twos = (twos ^ num) & ~ones;
    }
    return ones;
}
```
