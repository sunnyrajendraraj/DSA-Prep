# Arrays — Complete Topic Reference

## Q01: Find Second Largest Element

### 1. Problem Explanation

**Input:** An integer array `arr` of size `n`.  
**Output:** The second largest distinct element in the array.  
**Constraints:**
- `n >= 2`
- Array may contain duplicates
- Elements can be negative, zero, or positive

**Important observations:**
- "Second largest" means the second distinct value when elements are sorted in descending order.
- If all elements are the same, there is no second largest — handle this as an edge case.


### 2. Brute Solution

### Why this comes first
The most natural idea: sort the array, then find the second distinct element from the end.

### Algorithm
1. Sort the array in ascending order.
2. Start from the second-to-last element and move left until you find an element different from the last (largest) element.
3. That element is the second largest.

### Dry Run
```
arr = [12, 35, 1, 10, 34, 1]
After sorting: [1, 1, 10, 12, 34, 35]
Largest = 35 (last element)
Scan leftward: 34 ≠ 35 → Second Largest = 34
```

### Time Complexity
- **O(n log n)** — dominated by sorting

### Space Complexity
- **O(1)** if in-place sort (like `std::sort`), **O(n)** if extra array is made

### Pros
- Simple to write and understand
- No tricky logic

### Cons
- Sorting is unnecessary — we don't need the full sorted order
- Modifies the array

### C++ Code (Brute Force)

```cpp
#include <bits/stdc++.h>
using namespace std;

int findSecondLargestBrute(vector<int>& arr) {
    int n = arr.size();
    
    // Sort array in ascending order
    sort(arr.begin(), arr.end());
    
    // The largest element is arr[n-1]
    // Scan leftward to find the first element different from the largest
    for (int i = n - 2; i >= 0; i--) {
        if (arr[i] != arr[n - 1]) {
            return arr[i];  // This is the second largest distinct element
        }
    }
    
    // If no second distinct element exists (all elements are equal)
    return -1;
}
```


### 3. Optimal Solution & Complexities

### Why this is optimal
We only need to make **one pass** through the array. We track two variables:
- `largest`: the maximum element seen so far
- `secondLargest`: the maximum element seen so far that is **strictly less than `largest`**

Every element is visited exactly once → **O(n) time, O(1) space** — this is the best possible for an unsorted array.

### Algorithm
1. Initialize `largest = INT_MIN`, `secondLargest = INT_MIN`.
2. For each element `x` in the array:
   - If `x > largest`: update `secondLargest = largest`, then `largest = x`
   - Else if `x > secondLargest` AND `x != largest`: update `secondLargest = x`
3. If `secondLargest == INT_MIN`, no second largest exists → return -1.
4. Return `secondLargest`.

### Complete Dry Run
```
arr = [12, 35, 1, 10, 34, 1]
Initialize: largest = INT_MIN, secondLargest = INT_MIN

Step 1: x = 12
  12 > INT_MIN (largest) → secondLargest = INT_MIN, largest = 12
  State: largest=12, secondLargest=INT_MIN

Step 2: x = 35
  35 > 12 (largest) → secondLargest = 12, largest = 35
  State: largest=35, secondLargest=12

Step 3: x = 1
  1 < 35, 1 < 12 → no update
  State: largest=35, secondLargest=12

Step 4: x = 10
  10 < 35, 10 < 12 → no update
  State: largest=35, secondLargest=12

Step 5: x = 34
  34 < 35, 34 > 12 → secondLargest = 34
  State: largest=35, secondLargest=34

Step 6: x = 1 (duplicate)
  1 < 35, 1 < 34 → no update

Final: secondLargest = 34
```

### Why this is the best approach
- **O(n) time** — single pass, cannot be faster for unsorted arrays
- **O(1) space** — only two variables
- Handles duplicates correctly
- Handles negative numbers correctly

### C++ Code (Optimal)

```cpp
#include <bits/stdc++.h>
using namespace std;

int findSecondLargest(vector<int>& arr) {
    int n = arr.size();
    
    // Initialize both tracking variables to the smallest possible integer
    int largest = INT_MIN;
    int secondLargest = INT_MIN;
    
    for (int i = 0; i < n; i++) {
        int currentElement = arr[i];
        
        if (currentElement > largest) {
            // Current element is the new maximum
            // The previous maximum becomes a candidate for second largest
            secondLargest = largest;
            largest = currentElement;
        } else if (currentElement > secondLargest && currentElement != largest) {
            // Current element is not the largest, but it's better than second largest
            // The condition currentElement != largest handles duplicates of the maximum
            secondLargest = currentElement;
        }
    }
    
    // If secondLargest was never updated, no second distinct element exists
    if (secondLargest == INT_MIN) {
        return -1;  // Signal: no second largest exists
    }
    
    return secondLargest;
}
```


#### Complexity Summary

- **Time Complexity:** **O(n)**. We visit each element of the array exactly once.
- **Space Complexity:** **O(1)**. We only use two integer variables.


---

## Q02 Missing Number

### 1. Problem Explanation

Given an array containing $N-1$ distinct integers in the range $[1, N]$, find the single missing number.

*   **Input:** An integer array `nums` of size $N-1$, containing distinct integers in the range $[1, N]$.
*   **Output:** The single integer from the range $[1, N]$ that does not appear in the array.
*   **Constraints:**
    *   $N \ge 1$
    *   `nums` contains unique integers.
    *   $1 \le nums[i] \le N$
*   **Observations:**
    *   Since the array is of size $N-1$ and values are from $1$ to $N$, exactly one number is missing.
    *   The array is not necessarily sorted.
    *   Since all elements are distinct and positive, we can leverage mathematical properties or bitwise operations.


### 2. Brute Solution

### Algorithm
1.  Sort the input array in ascending order.
2.  Iterate through the array from index `0` to `N-2`.
3.  If `nums[i]` is not equal to `i + 1`, return `i + 1`.
4.  If the loop completes without returning, it means all elements from $1$ to $N-1$ are present. Return $N$.

### Dry Run
Input: `nums = [3, 1, 4]`, $N=4$
1.  Sort: `nums = [1, 3, 4]`
2.  `i = 0`: `nums[0] = 1`, matches `i + 1 = 1`.
3.  `i = 1`: `nums[1] = 3`, does not match `i + 1 = 2`.
4.  Return `2`.

### Complexity
*   **Time Complexity:** $O(N \log N)$ due to sorting.
*   **Space Complexity:** $O(1)$ (or $O(N)$ depending on the sorting algorithm implementation, but generally in-place).

### Pros/Cons
*   **Pros:** Very simple to understand.
*   **Cons:** Unnecessarily slow due to sorting. Modifies the input array.

### Code (C++17)
```cpp
#include <vector>
#include <algorithm>

class SolutionBrute {
public:
    int findMissingNumber(std::vector<int>& nums) {
        int n = nums.size() + 1; // Array size is N-1, so N is nums.size() + 1
        std::sort(nums.begin(), nums.end());
        
        for (int i = 0; i < n - 1; ++i) {
            if (nums[i] != i + 1) {
                return i + 1;
            }
        }
        return n;
    }
};
```


### 3. Optimal Solution & Complexities

We present two optimal approaches: the **Sum Formula** and the **XOR Method**.

### Approach A: Sum Formula
*   Sum of first $N$ natural numbers: $S_N = \frac{N(N+1)}{2}$.
*   Sum of array elements: $S_{arr} = \sum nums[i]$.
*   Missing number = $S_N - S_{arr}$.

### Approach B: XOR Method
*   XOR all numbers from $1$ to $N$: $X_1 = 1 \oplus 2 \oplus \dots \oplus N$.
*   XOR all elements in the array: $X_2 = nums[0] \oplus nums[1] \dots \oplus nums[N-2]$.
*   Missing number = $X_1 \oplus X_2$.

### Dry Run (XOR Method)
Input: `nums = [3, 1, 4]`, $N = 4$
1.  Initialize `xorSum = 0`.
2.  XOR all numbers from $1$ to $N$:
    `xorSum = 0 ^ 1 ^ 2 ^ 3 ^ 4`
3.  XOR all numbers in the array:
    `xorSum = (0 ^ 1 ^ 2 ^ 3 ^ 4) ^ (3 ^ 1 ^ 4)`
4.  Rearrange:
    `xorSum = (1 ^ 1) ^ (3 ^ 3) ^ (4 ^ 4) ^ 2`
    `xorSum = 0 ^ 0 ^ 0 ^ 2 = 2`
5.  Return `2`.

### Why Optimal
Both methods achieve $O(N)$ time with $O(1)$ auxiliary space. The XOR method is preferred in production because it prevents any risk of integer overflow.

### Beautiful C++17 Code

```cpp
#include <vector>
#include <numeric>

class SolutionOptimal {
public:
    // Optimal 1: XOR Approach (Safe from overflow)
    int findMissingNumberXOR(const std::vector<int>& nums) {
        int n = nums.size() + 1;
        int xorSum = 0;
        
        // XOR all numbers from 1 to N
        for (int i = 1; i <= n; ++i) {
            xorSum ^= i;
        }
        
        // XOR all numbers in the array
        for (int num : nums) {
            xorSum ^= num;
        }
        
        return xorSum;
    }

    // Optimal 2: Sum Approach (Uses long long to prevent overflow)
    int findMissingNumberSum(const std::vector<int>& nums) {
        long long n = nums.size() + 1;
        long long expectedSum = (n * (n + 1)) / 2;
        
        long long actualSum = 0;
        for (int num : nums) {
            actualSum += num;
        }
        
        return static_cast<int>(expectedSum - actualSum);
    }
};
```


#### Complexity Summary

| Approach | Time Complexity (Best/Avg/Worst) | Space Complexity | Reasoning |
| :--- | :--- | :--- | :--- |
| **Brute Force (Sorting)** | $O(N \log N)$ | $O(1)$ | Sorting takes $O(N \log N)$ time. Iteration takes $O(N)$. |
| **Better (Hashing)** | $O(N)$ | $O(N)$ | Extra array of size $N+1$ requires linear auxiliary space. |
| **Optimal (Sum)** | $O(N)$ | $O(1)$ | Single pass to sum up array elements. |
| **Optimal (XOR)** | $O(N)$ | $O(1)$ | Two simple loops of sizes $N$ and $N-1$ with constant space. |


---

## Q03 Leaders Array

### 1. Problem Explanation

Given an integer array `arr` of size $N$, find all the "leaders" in the array. An element is a leader if it is strictly greater than all the elements to its right side. The rightmost element is always a leader.

*   **Input:** An integer array `arr` of size $N$.
*   **Output:** A list containing all the leaders in the array.
*   **Constraints:**
    *   $1 \le N \le 10^6$
    *   $-10^9 \le arr[i] \le 10^9$
*   **Observations:**
    *   The last element has no elements to its right, so it is always a leader.
    *   If an element is a leader, it must be larger than the maximum element to its right.
    *   The leaders do not have to be in any specific sorted order, but they should generally be returned in either their original order of appearance (left-to-right) or reversed order (right-to-left), as specified by the interviewer.


### 2. Brute Solution

### Algorithm
1.  Initialize an empty list/vector `leaders`.
2.  Iterate `i` from `0` to `N - 1`:
    *   Set a boolean flag `isLeader = true`.
    *   Iterate `j` from `i + 1` to `N - 1`:
        *   If `arr[i] <= arr[j]`, set `isLeader = false` and break the inner loop.
    *   If `isLeader` is true, add `arr[i]` to `leaders`.
3.  Return `leaders`.

### Dry Run
Input: `arr = [16, 17, 4, 3, 5, 2]`, $N = 6$
*   `i = 0` (`16`): Compare with `17, 4, 3, 5, 2`. Since `16 <= 17`, `isLeader = false`.
*   `i = 1` (`17`): Compare with `4, 3, 5, 2`. All are smaller. `isLeader = true`. Add `17`.
*   `i = 2` (`4`): Compare with `3, 5, 2`. Since `4 <= 5`, `isLeader = false`.
*   `i = 3` (`3`): Compare with `5, 2`. Since `3 <= 5`, `isLeader = false`.
*   `i = 4` (`5`): Compare with `2`. `5 > 2`. `isLeader = true`. Add `5`.
*   `i = 5` (`2`): No elements to right. `isLeader = true`. Add `2`.
*   Output: `[17, 5, 2]`

### Complexity
*   **Time Complexity:** $O(N^2)$ in the worst case (e.g., sorted in ascending order or flat array).
*   **Space Complexity:** $O(1)$ auxiliary space (excluding the output list).

### Pros/Cons
*   **Pros:** Straightforward to implement, requires no extra thinking.
*   **Cons:** Inefficient. TLE (Time Limit Exceeded) for $N \ge 10^5$.

### Code (C++17)
```cpp
#include <vector>

class SolutionBrute {
public:
    std::vector<int> getLeaders(const std::vector<int>& arr) {
        std::vector<int> leaders;
        int n = arr.size();
        
        for (int i = 0; i < n; ++i) {
            bool isLeader = true;
            for (int j = i + 1; j < n; ++j) {
                if (arr[i] <= arr[j]) {
                    isLeader = false;
                    break;
                }
            }
            if (isLeader) {
                leaders.push_back(arr[i]);
            }
        }
        return leaders;
    }
};
```


### 3. Optimal Solution & Complexities

### Algorithm
1.  Initialize an empty list `leaders`.
2.  Set `max_from_right` to a very small value, or initialize it with the last element `arr[N - 1]`.
3.  Add `arr[N - 1]` to `leaders` (the rightmost element is always a leader).
4.  Iterate `i` from `N - 2` down to `0`:
    *   If `arr[i] > max_from_right`, then `arr[i]` is a leader:
        *   Add `arr[i]` to `leaders`.
        *   Update `max_from_right = arr[i]`.
5.  Since we processed the array from right to left, reverse the `leaders` list to match the original left-to-right order.
6.  Return the reversed list.

### Dry Run
Input: `arr = [16, 17, 4, 3, 5, 2]`, $N = 6$
*   Initialize: `max_from_right = arr[5] = 2`. Add `2` to `leaders`. `leaders = [2]`.
*   `i = 4` (`arr[4] = 5`): Is `5 > 2`? Yes. Add `5` to `leaders`. Update `max_from_right = 5`. `leaders = [2, 5]`.
*   `i = 3` (`arr[3] = 3`): Is `3 > 5`? No.
*   `i = 2` (`arr[2] = 4`): Is `4 > 5`? No.
*   `i = 1` (`arr[1] = 17`): Is `17 > 5`? Yes. Add `17` to `leaders`. Update `max_from_right = 17`. `leaders = [2, 5, 17]`.
*   `i = 0` (`arr[0] = 16`): Is `16 > 17`? No.
*   Reversed: `[17, 5, 2]`. Return this array.

### Why Optimal
*   We only visit each element exactly once.
*   We only need $O(1)$ space to track the maximum element seen so far.
*   It is impossible to find leaders in less than $O(N)$ time because we must examine every element at least once.

### Beautiful C++17 Code

```cpp
#include <vector>
#include <algorithm>

class SolutionOptimal {
public:
    std::vector<int> getLeaders(const std::vector<int>& arr) {
        std::vector<int> leaders;
        int n = arr.size();
        if (n == 0) return leaders;
        
        // Rightmost element is always a leader
        int maxFromRight = arr[n - 1];
        leaders.push_back(maxFromRight);
        
        // Scan the array from right to left
        for (int i = n - 2; i >= 0; --i) {
            if (arr[i] > maxFromRight) {
                leaders.push_back(arr[i]);
                maxFromRight = arr[i]; // Update the maximum seen so far
            }
        }
        
        // Reverse the leaders vector to restore original left-to-right order
        std::reverse(leaders.begin(), leaders.end());
        return leaders;
    }
};
```


#### Complexity Summary

| Approach | Time Complexity (Best/Avg/Worst) | Space Complexity (Auxiliary) | Reasoning |
| :--- | :--- | :--- | :--- |
| **Brute Force** | $O(N^2)$ | $O(1)$ | Double nested loops scan the suffix for every element. |
| **Optimal** | $O(N)$ | $O(1)$ | Single pass from right-to-left. Reversing the output takes $O(N)$ time. |


---

## Q04 Kadanes Algorithm

### 1. Problem Explanation

Given an integer array `nums`, find the contiguous subarray (containing at least one number) which has the largest sum and return its sum. Additionally, determine the start and end indices of the subarray to print the actual elements.

*   **Input:** An integer array `nums` of size $N$.
*   **Output:** The maximum subarray sum (integer) and the actual subarray.
*   **Constraints:**
    *   $1 \le N \le 10^5$
    *   $-10^4 \le nums[i] \le 10^4$
*   **Observations:**
    *   If all numbers are positive, the maximum subarray is the entire array.
    *   If all numbers are negative, the maximum subarray is the single largest (least negative) element.
    *   The subarray must be contiguous (elements must be adjacent).


### 2. Brute Solution

### Algorithm
1.  Initialize `maxSum` to the lowest possible integer (`INT_MIN`).
2.  Run a loop `i` from `0` to `N - 1` (start of subarray).
3.  Run a nested loop `j` from `i` to `N - 1` (end of subarray).
4.  Run a third loop `k` from `i` to `j` to compute the sum of `nums[i...j]`.
5.  If `currentSum > maxSum`, update `maxSum = currentSum`.
6.  Return `maxSum`.

### Dry Run
Input: `nums = [-2, 1, -3]`
*   Subarrays:
    *   `[-2]` $\to$ sum = `-2`
    *   `[-2, 1]` $\to$ sum = `-1`
    *   `[-2, 1, -3]` $\to$ sum = `-4`
    *   `[1]` $\to$ sum = `1`
    *   `[1, -3]` $\to$ sum = `-2`
    *   `[-3]` $\to$ sum = `-3`
*   Max sum is `1`.

### Complexity
*   **Time Complexity:** $O(N^3)$ due to three nested loops.
*   **Space Complexity:** $O(1)$ auxiliary space.

### Code (C++17)
```cpp
#include <vector>
#include <climits>
#include <algorithm>

class SolutionBrute {
public:
    int maxSubArray(const std::vector<int>& nums) {
        int n = nums.size();
        int maxSum = INT_MIN;
        
        for (int i = 0; i < n; ++i) {
            for (int j = i; j < n; ++j) {
                int currentSum = 0;
                for (int k = i; k <= j; ++k) {
                    currentSum += nums[k];
                }
                maxSum = std::max(maxSum, currentSum);
            }
        }
        return maxSum;
    }
};
```


### 3. Optimal Solution & Complexities

### Algorithm
1.  Initialize `maxSum` to `INT_MIN` and `currentSum` to `0`.
2.  To track the subarray indices:
    *   Initialize `start = 0`, `tempStart = 0`, and `end = 0`.
3.  Iterate `i` from `0` to `N - 1`:
    *   Add `nums[i]` to `currentSum`.
    *   If `currentSum > maxSum`:
        *   Update `maxSum = currentSum`.
        *   Update `start = tempStart`.
        *   Update `end = i`.
    *   If `currentSum < 0`:
        *   Reset `currentSum = 0`.
        *   Set `tempStart = i + 1` (propose starting the next subarray from index `i + 1`).
4.  Print/Return the maximum sum and the elements from index `start` to `end`.

### Dry Run
Input: `nums = [-2, 1, -3, 4, -1, 2, 1, -5, 4]`
*   `i = 0` (`-2`): `currentSum = -2`. `maxSum` becomes `-2`. `start = 0, end = 0`. Since `currentSum < 0`, reset `currentSum = 0`, `tempStart = 1`.
*   `i = 1` (`1`): `currentSum = 1`. `maxSum` becomes `1`. `start = 1, end = 1`.
*   `i = 2` (`-3`): `currentSum = -2`. `currentSum < 0`, reset `currentSum = 0`, `tempStart = 3`.
*   `i = 3` (`4`): `currentSum = 4`. `maxSum` becomes `4`. `start = 3, end = 3`.
*   `i = 4` (`-1`): `currentSum = 3`.
*   `i = 5` (`2`): `currentSum = 5`. `maxSum` becomes `5`. `start = 3, end = 5`.
*   `i = 6` (`1`): `currentSum = 6`. `maxSum` becomes `6`. `start = 3, end = 6`. Subarray is `[4, -1, 2, 1]`.
*   `i = 7` (`-5`): `currentSum = 1`.
*   `i = 8` (`4`): `currentSum = 5`.
*   Final maximum sum is `6`, subarray is `[4, -1, 2, 1]`.

### Beautiful C++17 Code

```cpp
#include <vector>
#include <climits>
#include <iostream>
#include <algorithm>

struct SubarrayResult {
    int maxSum;
    std::vector<int> subarray;
};

class SolutionOptimal {
public:
    SubarrayResult maxSubArray(const std::vector<int>& nums) {
        int n = nums.size();
        int maxSum = INT_MIN;
        int currentSum = 0;
        
        int start = 0;
        int end = 0;
        int tempStart = 0;
        
        for (int i = 0; i < n; ++i) {
            currentSum += nums[i];
            
            if (currentSum > maxSum) {
                maxSum = currentSum;
                start = tempStart;
                end = i;
            }
            
            // If sum becomes negative, discard the prefix
            if (currentSum < 0) {
                currentSum = 0;
                tempStart = i + 1;
            }
        }
        
        // Extract the subarray elements
        std::vector<int> sub(nums.begin() + start, nums.begin() + end + 1);
        
        return {maxSum, sub};
    }
};
```


#### Complexity Summary

| Approach | Time Complexity | Space Complexity | Reasoning |
| :--- | :--- | :--- | :--- |
| **Brute Force** | $O(N^3)$ | $O(1)$ | Three nested loops to calculate all subarray sums. |
| **Better** | $O(N^2)$ | $O(1)$ | Two nested loops, utilizing cumulative sums. |
| **Optimal (Kadane's)** | $O(N)$ | $O(1)$ | Single pass over the array. Vector slicing at the end takes $O(\text{length of subarray})$ time. |


---

## Q05 Move Zeroes

### 1. Problem Explanation

Given an integer array `nums`, move all `0`'s to the end of it while maintaining the relative order of the non-zero elements. This operation must be performed in-place without making a copy of the array.

*   **Input:** An integer array `nums` of size $N$.
*   **Output:** The same array `nums` modified in-place.
*   **Constraints:**
    *   $1 \le N \le 10^4$
    *   $-2^{31} \le nums[i] \le 2^{31} - 1$
*   **Observations:**
    *   Minimize the total number of operations (writes).
    *   Maintain the original relative ordering of all non-zero elements.
    *   Perform this in $O(1)$ auxiliary space.


### 2. Brute Solution

### Algorithm
1.  Initialize a temporary array `temp` of size $N$ with all zeroes.
2.  Initialize a pointer `tempIdx = 0`.
3.  Iterate `i` from `0` to `N - 1`:
    *   If `nums[i] != 0`, assign `temp[tempIdx] = nums[i]` and increment `tempIdx`.
4.  Copy the elements of `temp` back into `nums`.

### Dry Run
Input: `nums = [0, 1, 0, 3, 12]`
1.  `temp = [0, 0, 0, 0, 0]`, `tempIdx = 0`
2.  Scan `nums`:
    *   `nums[0] = 0` $\to$ skip.
    *   `nums[1] = 1` $\to$ `temp[0] = 1`, `tempIdx = 1`.
    *   `nums[2] = 0` $\to$ skip.
    *   `nums[3] = 3` $\to$ `temp[1] = 3`, `tempIdx = 2`.
    *   `nums[4] = 12` $\to$ `temp[2] = 12`, `tempIdx = 3`.
3.  `temp` is now `[1, 3, 12, 0, 0]`.
4.  Copy `temp` back to `nums`.

### Complexity
*   **Time Complexity:** $O(N)$ — We traverse the array twice.
*   **Space Complexity:** $O(N)$ — We create an auxiliary array of size $N$.

### Code (C++17)
```cpp
#include <vector>

class SolutionBrute {
public:
    void moveZeroes(std::vector<int>& nums) {
        std::vector<int> temp(nums.size(), 0);
        int tempIdx = 0;
        
        for (int num : nums) {
            if (num != 0) {
                temp[tempIdx++] = num;
            }
        }
        
        nums = temp;
    }
};
```


### 3. Optimal Solution & Complexities

### Algorithm
1.  Initialize `insertPos = 0` (this pointer tracks where the next non-zero element should go).
2.  Iterate `i` from `0` to `N - 1`:
    *   If `nums[i]` is not `0`:
        *   Swap `nums[i]` and `nums[insertPos]`.
        *   Increment `insertPos`.

### Why Optimal
*   It performs the entire shifting in a single pass.
*   It minimizes write operations. For example, if there are no zeroes, no unnecessary assignments are made (or we can add a check `if (i != insertPos)` to prevent self-swaps).
*   Space complexity is strictly $O(1)$.

### Beautiful C++17 Code

```cpp
#include <vector>
#include <algorithm>

class SolutionOptimal {
public:
    void moveZeroes(std::vector<int>& nums) {
        int n = nums.size();
        int insertPos = 0;
        
        for (int i = 0; i < n; ++i) {
            if (nums[i] != 0) {
                // Avoid redundant swaps with itself
                if (i != insertPos) {
                    std::swap(nums[i], nums[insertPos]);
                }
                insertPos++;
            }
        }
    }
};
```


#### Complexity Summary

| Approach | Time Complexity (Best/Avg/Worst) | Space Complexity | Reasoning |
| :--- | :--- | :--- | :--- |
| **Brute Force** | $O(N)$ | $O(N)$ | Uses an auxiliary array of size $N$. |
| **Better (Two-Pass)** | $O(N)$ | $O(1)$ | Two sequential scans. Max writes is $N$. |
| **Optimal (One-Pass)** | $O(N)$ | $O(1)$ | Single pass. Number of swap operations is equal to the count of non-zero elements. |


---

## Q06 Find Duplicates

### 1. Problem Explanation

Given an integer array `nums` of size $N$ where all the integers in the array are in the range $[1, N]$, and each integer appears at most twice, find and return an array of all the integers that appear twice.

*   **Input:** An integer array `nums` of size $N$.
*   **Output:** A list containing all numbers that appear more than once (exactly twice).
*   **Constraints:**
    *   $N \ge 1$
    *   $1 \le nums[i] \le N$
    *   Each integer appears once or twice.
*   **Observations:**
    *   Since elements are bounded by the size of the array ($1 \le nums[i] \le N$), we can map values directly to array indices.
    *   We want to achieve $O(N)$ time complexity and $O(1)$ auxiliary space.


### 2. Brute Solution

### Algorithm
1.  Initialize an empty list `duplicates`.
2.  Iterate `i` from `0` to `N - 1`:
    *   Iterate `j` from `i + 1` to `N - 1`:
        *   If `nums[i] == nums[j]`, check if it's already in the duplicates list. If not, add `nums[i]` to `duplicates`.
3.  Return `duplicates`.

### Complexity
*   **Time Complexity:** $O(N^2)$ due to nested loops.
*   **Space Complexity:** $O(1)$ auxiliary space.

### Code (C++17)
```cpp
#include <vector>
#include <algorithm>

class SolutionBrute {
public:
    std::vector<int> findDuplicates(const std::vector<int>& nums) {
        std::vector<int> duplicates;
        int n = nums.size();
        
        for (int i = 0; i < n; ++i) {
            for (int j = i + 1; j < n; ++j) {
                if (nums[i] == nums[j]) {
                    // Check if duplicate already exists in results
                    if (std::find(duplicates.begin(), duplicates.end(), nums[i]) == duplicates.end()) {
                        duplicates.push_back(nums[i]);
                    }
                }
            }
        }
        return duplicates;
    }
};
```


### 3. Optimal Solution & Complexities

### Algorithm
1.  Initialize an empty list `duplicates`.
2.  Iterate through the array using index `i` from `0` to `N - 1`:
    *   Find the target index: `val = abs(nums[i])` and `idx = val - 1`.
    *   Check the sign of `nums[idx]`:
        *   If `nums[idx] < 0`, it means we have seen `val` before. Add `val` to `duplicates`.
        *   If `nums[idx] > 0`, negate it: `nums[idx] = -nums[idx]`.
3.  Optionally, restore the original array by taking the absolute value of all elements.
4.  Return `duplicates`.

### Dry Run
Input: `nums = [4, 3, 2, 7, 8, 2, 3, 1]`
*   `i = 0`: `val = 4`, `idx = 3`. `nums[3] = 7` (positive). Negate it: `nums[3] = -7`.
    Array: `[4, 3, 2, -7, 8, 2, 3, 1]`
*   `i = 1`: `val = 3`, `idx = 2`. `nums[2] = 2` (positive). Negate it: `nums[2] = -2`.
    Array: `[4, 3, -2, -7, 8, 2, 3, 1]`
*   `i = 2`: `val = 2`, `idx = 1`. `nums[1] = 3` (positive). Negate it: `nums[1] = -3`.
    Array: `[4, -3, -2, -7, 8, 2, 3, 1]`
*   `i = 3`: `val = 7`, `idx = 6`. `nums[6] = 3` (positive). Negate it: `nums[6] = -3`.
    Array: `[4, -3, -2, -7, 8, 2, -3, 1]`
*   `i = 4`: `val = 8`, `idx = 7`. `nums[7] = 1` (positive). Negate it: `nums[7] = -1`.
    Array: `[4, -3, -2, -7, -8, 2, -3, -1]` (Wait, idx for 8 is 7. `nums[4]` is 8, so `nums[7]` negated to `-1`).
*   `i = 5`: `val = 2`, `idx = 1`. `nums[1] = -3` (negative!). Duplicate found: `2`.
    Array: `[4, -3, -2, -7, -8, 2, -3, -1]`
*   `i = 6`: `val = 3` (obtained from `abs(-3)`), `idx = 2`. `nums[2] = -2` (negative!). Duplicate found: `3`.
    Array: `[4, -3, -2, -7, -8, 2, -3, -1]`
*   `i = 7`: `val = 1`, `idx = 0`. `nums[0] = 4` (positive). Negate it: `nums[0] = -4`.
*   Duplicates returned: `[2, 3]`.

### Why Optimal
*   We use the input array to store visited flags, eliminating auxiliary space requirements.
*   Time complexity is linear $O(N)$ with a single pass.
*   We can easily restore the input array back to its original values if needed.

### Beautiful C++17 Code

```cpp
#include <vector>
#include <cmath>

class SolutionOptimal {
public:
    std::vector<int> findDuplicates(std::vector<int>& nums) {
        std::vector<int> duplicates;
        
        for (int i = 0; i < nums.size(); ++i) {
            // Use absolute value to get original element value
            int val = std::abs(nums[i]);
            int idx = val - 1;
            
            // If the element at index is already negative, we've seen 'val' before
            if (nums[idx] < 0) {
                duplicates.push_back(val);
            } else {
                nums[idx] = -nums[idx]; // Mark as visited
            }
        }
        
        // Optional: Restore original values in the array
        for (int& num : nums) {
            num = std::abs(num);
        }
        
        return duplicates;
    }
};
```


#### Complexity Summary

| Approach | Time Complexity | Space Complexity (Auxiliary) | Reasoning |
| :--- | :--- | :--- | :--- |
| **Brute Force** | $O(N^2)$ | $O(1)$ | Double loops scan for matches. |
| **Better (Hashing)** | $O(N)$ | $O(N)$ | Uses a visited array/hash set. |
| **Optimal (Index Negation)**| $O(N)$ | $O(1)$ | In-place marking utilizing the sign bit. Restore phase runs in linear time. |


---

## Q07 Rotate Array

### 1. Problem Explanation

Given an integer array `nums` and a non-negative integer `k`, rotate the array to the right by `k` steps, where `k` is non-negative. Rotating an array to the right means shifting each element to the right, and the elements at the end wrap around to the front.

### Input
- An integer array `nums` of size `n` ($1 \le n \le 10^5$).
- An integer `k` ($0 \le k \le 10^5$).

### Output
- The array `nums` modified in-place (or returned depending on the platform) representing the rotated array.

### Constraints
- $1 \le n \le 10^5$
- $-2^{31} \le nums[i] \le 2^{31} - 1$
- $0 \le k \le 10^5$

### Observations
1. **Modulo Arithmetic:** If $k \ge n$, rotating the array $n$ times returns it to its original state. Therefore, rotating by $k$ is equivalent to rotating by $k \pmod n$.
2. **Direction:** Rotating to the right by $k$ is equivalent to rotating to the left by $n - k$.
3. **In-place:** The optimal solution should perform the rotation without allocating a new array (i.e., $O(1)$ auxiliary space).


### 2. Brute Solution

The brute force approach uses an extra array to temporarily store the elements in their new rotated positions.

### Algorithm
1. Retrieve the size of the array $n$.
2. Calculate the effective rotation index $k = k \pmod n$. If $k == 0$ or $n \le 1$, return immediately.
3. Instantiate a temporary array `temp` of size $n$.
4. Iterate through the array `nums` from index $0$ to $n-1$.
5. Place each element `nums[i]` at its new position in the temporary array: `temp[(i + k) % n] = nums[i]`.
6. Copy all elements from `temp` back into `nums`.

### Dry Run
Input: `nums = [1, 2, 3, 4, 5]`, $k = 2$, $n = 5$.
1. Effective $k = 2 \pmod 5 = 2$.
2. Temp array of size 5 created: `temp = [0, 0, 0, 0, 0]`.
3. Iteration:
   - $i = 0$: `temp[(0+2)%5] = nums[0]` $\rightarrow$ `temp[2] = 1`.
   - $i = 1$: `temp[(1+2)%5] = nums[1]` $\rightarrow$ `temp[3] = 2`.
   - $i = 2$: `temp[(2+2)%5] = nums[2]` $\rightarrow$ `temp[4] = 3`.
   - $i = 3$: `temp[(3+2)%5] = nums[3]` $\rightarrow$ `temp[0] = 4`.
   - $i = 4$: `temp[(4+2)%5] = nums[4]` $\rightarrow$ `temp[1] = 5`.
4. Copy `temp` (`[4, 5, 1, 2, 3]`) back to `nums`.

### Complexity Analysis
- **Time Complexity:** $O(n)$ — We iterate through the array twice (once to copy to `temp`, once to copy back).
- **Space Complexity:** $O(n)$ — Required to store the temporary array of size $n$.

### Pros/Cons
- **Pros:** Extremely easy to understand; requires no complex pointer arithmetic.
- **Cons:** High memory footprint. Memory allocation can be slow for very large arrays.

### C++17 Code
```cpp
#include <vector>

class Solution {
public:
    void rotate(std::vector<int>& nums, int k) {
        int n = nums.size();
        if (n <= 1) return;
        
        // Handle k greater than array size
        k = k % n;
        if (k == 0) return;
        
        // Allocate temporary array
        std::vector<int> temp(n);
        
        // Copy to rotated positions
        for (int i = 0; i < n; ++i) {
            temp[(i + k) % n] = nums[i];
        }
        
        // Copy back to original array
        nums = temp;
    }
};
```


### 3. Optimal Solution & Complexities

The optimal approach uses the **Reversal Algorithm**. It rotates the array in-place using three reverse operations, yielding $O(n)$ time and $O(1)$ space.

### Algorithm
1. Calculate effective $k = k \pmod n$. If $k == 0$ or $n \le 1$, return.
2. Reverse the first $n - k$ elements (from index $0$ to $n - k - 1$).
3. Reverse the last $k$ elements (from index $n - k$ to $n - 1$).
4. Reverse the entire array (from index $0$ to $n - 1$).

*(Alternatively: Reverse the entire array first, then reverse the first $k$ elements, and then the remaining $n - k$ elements. Both approaches yield the same result.)*

### Dry Run
Input: `nums = [1, 2, 3, 4, 5, 6, 7]`, $k = 3$, $n = 7$.
1. Effective $k = 3 \pmod 7 = 3$.
2. Reverse first $n - k = 4$ elements (`nums[0...3]`):
   - Before: `[1, 2, 3, 4, 5, 6, 7]`
   - After: `[4, 3, 2, 1, 5, 6, 7]`
3. Reverse last $k = 3$ elements (`nums[4...6]`):
   - Before: `[4, 3, 2, 1, 5, 6, 7]`
   - After: `[4, 3, 2, 1, 7, 6, 5]`
4. Reverse the whole array (`nums[0...6]`):
   - Before: `[4, 3, 2, 1, 7, 6, 5]`
   - After: `[5, 6, 7, 1, 2, 3, 4]`

The array is successfully rotated.

### C++17 Code
```cpp
#include <vector>
#include <algorithm> // For std::reverse

class Solution {
private:
    // Helper function to reverse subarray in place
    void reverseSubarray(std::vector<int>& nums, int start, int end) {
        while (start < end) {
            std::swap(nums[start], nums[end]);
            start++;
            end--;
        }
    }

public:
    void rotate(std::vector<int>& nums, int k) {
        int n = nums.size();
        if (n <= 1) return;
        
        // Handle k > n
        k = k % n;
        if (k == 0) return;
        
        // Step 1: Reverse first n - k elements
        reverseSubarray(nums, 0, n - k - 1);
        
        // Step 2: Reverse last k elements
        reverseSubarray(nums, n - k, n - 1);
        
        // Step 3: Reverse the entire array
        reverseSubarray(nums, 0, n - 1);
    }
};
```


#### Complexity Summary

| Approach | Time Complexity | Space Complexity | Notes |
| :--- | :--- | :--- | :--- |
| **Brute Force** | $O(n)$ | $O(n)$ | Allocates a full copy of the array. |
| **Better** | $O(n \times k)$ | $O(1)$ | Rotates elements one-by-one; prone to TLE. |
| **Optimal** | $O(n)$ | $O(1)$ | Three passes of array reversal; very efficient. |

* **Time Complexity Reason (Optimal):** We reverse three segments. The total elements swapped across the steps is $2 \times n$, which simplifies to $O(n)$.
* **Space Complexity Reason (Optimal):** We swap elements in-place using temporary variables, so no dynamic memory is allocated.


---

## Q08 Two Sum

### 1. Problem Explanation

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


### 2. Brute Solution

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


### 3. Optimal Solution & Complexities

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


#### Complexity Summary

| Approach | Time Complexity | Space Complexity | Notes |
| :--- | :--- | :--- | :--- |
| **Brute Force** | $O(n^2)$ | $O(1)$ | No memory overhead, but slow. |
| **Better (Sorting)**| $O(n \log n)$ | $O(n)$ | Helpful if indices are not required (reduces space to $O(1)$). |
| **Optimal (HashMap)**| $O(n)$ | $O(n)$ | Fast lookups, requires dynamic memory. |

* **Time Complexity (Optimal):** $O(n)$ since we iterate through the array once and perform $O(1)$ average time hash map lookups.
* **Space Complexity (Optimal):** $O(n)$ because we store up to $n$ elements in the hash map in the worst case (when the match is at the very end).


---

## Q09 Three Sum

### 1. Problem Explanation

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


### 2. Brute Solution

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


### 3. Optimal Solution & Complexities

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


#### Complexity Summary

| Approach | Time Complexity | Space Complexity | Notes |
| :--- | :--- | :--- | :--- |
| **Brute Force** | $O(n^3 \log T)$ | $O(T)$ | Extremely slow due to triple nested loops and set sorting. |
| **Better** | $O(n^2 \log T)$ | $O(n + T)$ | Faster, but uses hash set and global set overhead. |
| **Optimal** | $O(n^2)$ | $O(1)$ or $O(\log n)$ | In-place pointers. Space is just for sorting recursion stack. |

* **Time Complexity (Optimal):** Sorting takes $O(n \log n)$. The outer loop runs $n-2$ times, and the inner two-pointer search runs in $O(n)$ time. Overall time complexity is $O(n^2)$.
* **Space Complexity (Optimal):** $O(1)$ auxiliary space if we do not count the output list and stack space for sorting. In C++, `std::sort` typically uses $O(\log n)$ space for call stack.


---

## Q10 Merge Sorted Arrays

### 1. Problem Explanation

Given two sorted integer arrays `nums1` and `nums2` of size `m` and `n` respectively, merge them into a single sorted array.

There are two primary variants of this problem:
1. **With Extra Space:** Merge `nums1` and `nums2` into a new sorted array of size `m + n`.
2. **In-place (LeetCode style):** `nums1` has a size of `m + n`, where the first `m` elements denote the elements that should be merged, and the last `n` elements are set to `0` and should be ignored. `nums2` has a size of `n`. Merge `nums2` into `nums1` in-place.
3. **In-place (Without Extra Space - Shell/Gap Method):** `nums1` of size `m` and `nums2` of size `n` are individual arrays. Merge them such that the first `m` smallest elements are in `nums1` (sorted) and the remaining `n` elements are in `nums2` (sorted), without using any extra space.

We will focus on the **LeetCode style In-place Merge** as the main optimal version and the **Gap Method** as the advanced in-place merge without extra space.

### Input
- `nums1` of size `m + n` (first `m` elements are sorted).
- `nums2` of size `n` (sorted).

### Output
- `nums1` is modified in-place to contain all `m + n` elements in sorted order.

### Constraints
- $nums1.length == m + n$
- $nums2.length == n$
- $0 \le m, n \le 200$
- $1 \le m + n \le 200$
- $-10^9 \le nums1[i], nums2[j] \le 10^9$


### 2. Brute Solution

Concatenate the elements of the second array into the end of the first array, then sort.

### Algorithm
1. Loop $i$ from $0$ to $n-1$.
2. Copy `nums2[i]` to `nums1[m + i]`.
3. Sort `nums1` from index $0$ to $m + n - 1$.

### Dry Run
Input: `nums1 = [1, 2, 3, 0, 0, 0]`, $m = 3$, `nums2 = [2, 5, 6]`, $n = 3$.
1. Copying:
   - `nums1[3] = nums2[0]` $\rightarrow$ `nums1[3] = 2`
   - `nums1[4] = nums2[1]` $\rightarrow$ `nums1[4] = 5`
   - `nums1[5] = nums2[2]` $\rightarrow$ `nums1[5] = 6`
   - Array becomes: `nums1 = [1, 2, 3, 2, 5, 6]`
2. Sorting:
   - Sort `[1, 2, 3, 2, 5, 6]` $\rightarrow$ `[1, 2, 2, 3, 5, 6]`.

### Complexity Analysis
- **Time Complexity:** $O((m+n) \log(m+n))$ — Due to sorting the merged array of size $m+n$.
- **Space Complexity:** $O(1)$ — If using in-place sort (like heap sort or introsort).

### Pros/Cons
- **Pros:** Simple, few lines of code.
- **Cons:** Does not take advantage of the fact that the individual arrays are already sorted. Slow for large values.

### C++17 Code
```cpp
#include <vector>
#include <algorithm>

class Solution {
public:
    void merge(std::vector<int>& nums1, int m, std::vector<int>& nums2, int n) {
        // Concatenate nums2 into nums1's empty space
        for (int i = 0; i < n; ++i) {
            nums1[m + i] = nums2[i];
        }
        
        // Sort the entire array
        std::sort(nums1.begin(), nums1.end());
    }
};
```


### 3. Optimal Solution & Complexities

Since the target array `nums1` has pre-allocated space at the end, we can merge elements from back to front in linear time and constant space.

### Algorithm
1. Initialize pointers:
   - `i = m - 1` (last active element in `nums1`).
   - `j = n - 1` (last element in `nums2`).
   - `write_idx = m + n - 1` (last position of `nums1`).
2. While `j >= 0` (as long as there are elements left in `nums2` to merge):
   - If `i >= 0` and `nums1[i] > nums2[j]`:
     - Set `nums1[write_idx] = nums1[i]`.
     - Decrement `i`.
   - Else:
     - Set `nums1[write_idx] = nums2[j]`.
     - Decrement `j`.
   - Decrement `write_idx`.
3. If `i < 0` and `j >= 0`, the loop will copy the remaining elements of `nums2` to the front of `nums1`. If `j < 0` first, the remaining elements of `nums1` are already in their correct places.

### Dry Run
Input: `nums1 = [1, 3, 5, 0, 0, 0]`, $m = 3$, `nums2 = [2, 4, 6]`, $n = 3$.
- `i = 2` (val: 5), `j = 2` (val: 6), `write_idx = 5`.
- Compare `5` and `6`. $6 > 5 \rightarrow$ `nums1[5] = 6`. `j` becomes 1, `write_idx` becomes 4.
- `i = 2` (val: 5), `j = 1` (val: 4). Compare `5` and `4`. $5 > 4 \rightarrow$ `nums1[4] = 5`. `i` becomes 1, `write_idx` becomes 3.
- `i = 1` (val: 3), `j = 1` (val: 4). Compare `3` and `4`. $4 > 3 \rightarrow$ `nums1[3] = 4`. `j` becomes 0, `write_idx` becomes 2.
- `i = 1` (val: 3), `j = 0` (val: 2). Compare `3` and `2`. $3 > 2 \rightarrow$ `nums1[2] = 3`. `i` becomes 0, `write_idx` becomes 1.
- `i = 0` (val: 1), `j = 0` (val: 2). Compare `1` and `2`. $2 > 1 \rightarrow$ `nums1[1] = 2`. `j` becomes -1, `write_idx` becomes 0.
- Loop terminates because `j < 0`. Array is `[1, 2, 3, 4, 5, 6]`.

### C++17 Code
```cpp
#include <vector>

class Solution {
public:
    void merge(std::vector<int>& nums1, int m, std::vector<int>& nums2, int n) {
        // Pointers for nums1, nums2, and write target
        int i = m - 1;
        int j = n - 1;
        int write_idx = m + n - 1;
        
        // Merge starting from the back
        while (j >= 0) {
            // If nums1 has elements left and its current element is larger
            if (i >= 0 && nums1[i] > nums2[j]) {
                nums1[write_idx] = nums1[i];
                i--;
            } else {
                nums1[write_idx] = nums2[j];
                j--;
            }
            write_idx--;
        }
    }
};
```


#### Complexity Summary

| Approach | Time Complexity | Space Complexity | Notes |
| :--- | :--- | :--- | :--- |
| **Brute Force** | $O((m+n) \log(m+n))$ | $O(1)$ | Simple copy + sort. |
| **Gap Method** | $O((m+n) \log(m+n))$ | $O(1)$ | Used when arrays are separate and cannot be resized. |
| **Optimal (Backwards)**| $O(m + n)$ | $O(1)$ | Linear time, in-place, zero auxiliary space. |

* **Time Complexity (Optimal):** $O(m + n)$ since we make at most $m + n$ comparisons and writes in a single pass.
* **Space Complexity (Optimal):** $O(1)$ auxiliary space because we reuse the allocated padding in `nums1`.


---

## Q11 Intersection

### 1. Problem Explanation

Given two integer arrays `nums1` and `nums2`, return an array of their intersection.

There are two common variations of this problem:
* **Variant A (Unique Intersection):** Each element in the result must be **unique**, and you may return the result in any order.
* **Variant B (Frequency-Preserving Intersection):** Each element in the result must appear as many times as it shows in **both** arrays, and you may return the result in any order.

We will explain how to handle **both variants**, focusing on Variant A as the baseline and showing how Variant B differs.

### Input
- `nums1`: An integer array of size `m` ($1 \le m \le 1000$).
- `nums2`: An integer array of size `n` ($1 \le n \le 1000$).

### Output
- An integer array representing the intersection of `nums1` and `nums2`.

### Constraints
- $1 \le nums1.length, nums2.length \le 1000$
- $0 \le nums1[i], nums2[i] \le 1000$


### 2. Brute Solution

Use nested loops to check if each element of `nums1` is present in `nums2`.

### Algorithm (Variant A)
1. Initialize an empty result vector `result`.
2. Iterate $i$ from $0$ to $m-1$:
   - Check if `nums1[i]` is already in `result` (to avoid duplicates). If so, skip.
   - Iterate $j$ from $0$ to $n-1$:
     - If `nums1[i] == nums2[j]`:
       - Append `nums1[i]` to `result`.
       - Break the inner loop.
3. Return `result`.

### Complexity Analysis
- **Time Complexity:** $O(m \times n)$ — For each element in `nums1`, we scan `nums2` linearly.
- **Space Complexity:** $O(\min(m, n))$ — To store the output array.

### C++17 Code
```cpp
#include <vector>
#include <algorithm>

class Solution {
public:
    std::vector<int> intersection(std::vector<int>& nums1, std::vector<int>& nums2) {
        std::vector<int> result;
        
        for (int i = 0; i < nums1.size(); ++i) {
            // Check if nums1[i] is already added to result
            if (std::find(result.begin(), result.end(), nums1[i]) != result.end()) {
                continue;
            }
            
            // Search in nums2
            for (int j = 0; j < nums2.size(); ++j) {
                if (nums1[i] == nums2[j]) {
                    result.push_back(nums1[i]);
                    break; // Found match, stop searching for this element
                }
            }
        }
        
        return result;
    }
};
```


### 3. Optimal Solution & Complexities

If memory is constrained (we want $O(1)$ auxiliary space), sorting both arrays first allows using the two-pointer technique.

### Algorithm (Variant A - Unique Intersection)
1. Sort both `nums1` and `nums2`.
2. Initialize pointers `i = 0`, `j = 0`.
3. While `i < nums1.size()` and `j < nums2.size()`:
   - If `nums1[i] == nums2[j]`:
     - If the result vector is empty or `nums1[i]` is not equal to `result.back()` (to avoid duplicates), add it to `result`.
     - Increment both `i` and `j`.
   - If `nums1[i] < nums2[j]`: increment `i`.
   - If `nums1[i] > nums2[j]`: increment `j`.
4. Return `result`.

### Algorithm (Variant B - Duplicate-Preserving)
* Same as above, but delete the duplicate check (`result.back() != nums1[i]`). Simply push `nums1[i]` every time they match, and increment both pointers by 1.

### C++17 Code (Variant A: Unique)
```cpp
#include <vector>
#include <algorithm>

class Solution {
public:
    std::vector<int> intersection(std::vector<int>& nums1, std::vector<int>& nums2) {
        // Step 1: Sort both arrays
        std::sort(nums1.begin(), nums1.end());
        std::sort(nums2.begin(), nums2.end());
        
        std::vector<int> result;
        int i = 0;
        int j = 0;
        
        // Step 2: Two-pointer scan
        while (i < nums1.size() && j < nums2.size()) {
            if (nums1[i] == nums2[j]) {
                // To maintain unique values in the result
                if (result.empty() || result.back() != nums1[i]) {
                    result.push_back(nums1[i]);
                }
                i++;
                j++;
            } 
            else if (nums1[i] < nums2[j]) {
                i++;
            } 
            else {
                j++;
            }
        }
        
        return result;
    }
};
```


#### Complexity Summary

| Variant / Approach | Time Complexity | Space Complexity | Notes |
| :--- | :--- | :--- | :--- |
| **Brute Force** | $O(m \times n)$ | $O(\min(m, n))$ | Slow nested comparisons. |
| **Hash Set (Variant A)**| $O(m + n)$ | $O(m + \min(m, n))$ | Fast, but uses auxiliary hash tables. |
| **Hash Map (Variant B)**| $O(m + n)$ | $O(m + \min(m, n))$ | Preserves exact duplicate occurrence counts. |
| **Two Pointer (Sorted)**| $O(m \log m + n \log n)$| $O(1)$ | No extra space needed (in-place sorting). |

* **Time Complexity (Two Pointer):** Sorting takes $O(m \log m + n \log n)$ time. The linear traversal takes $O(m + n)$. The sorting step dominates, so total time is $O(m \log m + n \log n)$.
* **Space Complexity (Two Pointer):** $O(1)$ auxiliary space if sorted in-place (ignoring recursive stack frame allocations during sorting, which take $O(\log m + \log n)$ space).


---

## Q12 Union

### 1. Problem Explanation

Given two integer arrays `nums1` and `nums2`, find their union. The union of two arrays is defined as a collection of all **distinct** elements present in both arrays combined. The output can be returned in any order (unless specified as sorted).

### Input
- `nums1`: An integer array of size `m` ($1 \le m \le 10^5$).
- `nums2`: An integer array of size `n` ($1 \le n \le 10^5$).

### Output
- An integer array containing the distinct elements representing the union of `nums1` and `nums2`.

### Constraints
- $1 \le m, n \le 10^5$
- $-10^9 \le nums1[i], nums2[j] \le 10^9$

### Observations
1. **Uniqueness:** The key constraint is that the output must not contain duplicate values, even if a value appears multiple times in `nums1` or `nums2`.
2. **Order:** Union is mathematically unordered, but sorting the inputs can help optimize the identification of duplicates.


### 2. Brute Solution

Concatenate the arrays, sort the combination, and extract unique elements.

### Algorithm
1. Create a vector `combined` of size $m+n$.
2. Copy all elements of `nums1` and `nums2` into `combined`.
3. Sort `combined` in ascending order.
4. Create an empty vector `result`.
5. Iterate through `combined`:
   - If `result` is empty or the current element is different from the last element in `result`, append the current element to `result`.
6. Return `result`.

### Dry Run
Input: `nums1 = [3, 1, 3]`, `nums2 = [2, 1]`.
1. `combined = [3, 1, 3, 2, 1]`.
2. Sorted: `combined = [1, 1, 2, 3, 3]`.
3. Unique extraction:
   - Item 1: `1` $\rightarrow$ Push (result is empty) $\rightarrow$ `result = [1]`.
   - Item 2: `1` $\rightarrow$ Skip (matches `result.back()`).
   - Item 3: `2` $\rightarrow$ Push (doesn't match `1`) $\rightarrow$ `result = [1, 2]`.
   - Item 4: `3` $\rightarrow$ Push (doesn't match `2`) $\rightarrow$ `result = [1, 2, 3]`.
   - Item 5: `3` $\rightarrow$ Skip.
4. Return `[1, 2, 3]`.

### Complexity Analysis
- **Time Complexity:** $O((m+n) \log(m+n))$ — Due to sorting the combined array.
- **Space Complexity:** $O(m + n)$ — To store the concatenated array before sorting.

### C++17 Code
```cpp
#include <vector>
#include <algorithm>

class Solution {
public:
    std::vector<int> makeUnion(std::vector<int>& nums1, std::vector<int>& nums2) {
        std::vector<int> combined;
        combined.reserve(nums1.size() + nums2.size());
        
        // Step 1: Concatenate both arrays
        combined.insert(combined.end(), nums1.begin(), nums1.end());
        combined.insert(combined.end(), nums2.begin(), nums2.end());
        
        // Step 2: Sort
        std::sort(combined.begin(), combined.end());
        
        // Step 3: Extract unique elements
        std::vector<int> result;
        for (int val : combined) {
            if (result.empty() || result.back() != val) {
                result.push_back(val);
            }
        }
        
        return result;
    }
};
```


### 3. Optimal Solution & Complexities

If the inputs are already sorted, or if we want to run in-place without $O(m+n)$ auxiliary set storage, we sort both arrays and use the Two-Pointer approach.

### Algorithm
1. Sort both `nums1` and `nums2`.
2. Initialize pointers `i = 0` (for `nums1`) and `j = 0` (for `nums2`).
3. Create an empty result vector `result`.
4. While `i < m` and `j < n`:
   - If `nums1[i] < nums2[j]`:
     - If `result` is empty or `result.back() != nums1[i]`, push `nums1[i]`.
     - Increment `i`.
   - Else if `nums1[i] > nums2[j]`:
     - If `result` is empty or `result.back() != nums2[j]`, push `nums2[j]`.
     - Increment `j`.
   - Else (i.e. `nums1[i] == nums2[j]`):
     - If `result` is empty or `result.back() != nums1[i]`, push `nums1[i]`.
     - Increment both `i` and `j`.
5. While `i < m` (flush remaining elements in `nums1`):
   - If `result.empty() || result.back() != nums1[i]`, push `nums1[i]`.
   - Increment `i`.
6. While `j < n` (flush remaining elements in `nums2`):
   - If `result.empty() || result.back() != nums2[j]`, push `nums2[j]`.
   - Increment `j`.
7. Return `result`.

### C++17 Code
```cpp
#include <vector>
#include <algorithm>

class Solution {
private:
    // Helper function to push unique elements into the result vector
    void pushUnique(std::vector<int>& result, int val) {
        if (result.empty() || result.back() != val) {
            result.push_back(val);
        }
    }

public:
    std::vector<int> makeUnion(std::vector<int>& nums1, std::vector<int>& nums2) {
        int m = nums1.size();
        int n = nums2.size();
        
        // Step 1: Sort both arrays
        std::sort(nums1.begin(), nums1.end());
        std::sort(nums2.begin(), nums2.end());
        
        std::vector<int> result;
        int i = 0;
        int j = 0;
        
        // Step 2: Parallel scan
        while (i < m && j < n) {
            if (nums1[i] < nums2[j]) {
                pushUnique(result, nums1[i]);
                i++;
            } 
            else if (nums2[j] < nums1[i]) {
                pushUnique(result, nums2[j]);
                j++;
            } 
            else {
                pushUnique(result, nums1[i]);
                i++;
                j++;
            }
        }
        
        // Step 3: Flush remaining elements of nums1
        while (i < m) {
            pushUnique(result, nums1[i]);
            i++;
        }
        
        // Step 4: Flush remaining elements of nums2
        while (j < n) {
            pushUnique(result, nums2[j]);
            j++;
        }
        
        return result;
    }
};
```


#### Complexity Summary

| Approach | Time Complexity | Space Complexity | Notes |
| :--- | :--- | :--- | :--- |
| **Brute Force** | $O((m+n) \log(m+n))$ | $O(m + n)$ | Concat + sort. Easy but memory heavy. |
| **Hash Set** | $O(m + n)$ | $O(m + n)$ | Fastest when inputs are unsorted. |
| **Two Pointer** | $O(m \log m + n \log n)$| $O(1)$ auxiliary | Best if arrays are already sorted. |

* **Time Complexity (Two Pointer):** Sorting takes $O(m \log m + n \log n)$. Scanning both arrays takes $O(m + n)$. The sorting step dominates.
* **Space Complexity (Two Pointer):** $O(1)$ auxiliary space if sorted in-place. If the output array is included, space is $O(m + n)$.


---

## Q13 Equilibrium Index

### 1. Problem Explanation

An **Equilibrium Index** of an array is an index such that the sum of elements at lower indices is equal to the sum of elements at higher indices. More formally, an index $i$ (0-based) is an equilibrium index if:

$$\sum_{j=0}^{i-1} \text{arr}[j] = \sum_{k=i+1}^{N-1} \text{arr}[k]$$

If $i = 0$, the left sum is assumed to be `0`. 
If $i = N-1$, the right sum is assumed to be `0`.

### Input
- An array of integers `arr` of size $N$.

### Output
- The first equilibrium index (0-based index) if it exists, otherwise return `-1`.

### Constraints
- $1 \le N \le 10^6$
- $-10^5 \le \text{arr}[i] \le 10^5$

### Observations
- Elements can be positive, negative, or zero.
- The equilibrium index itself is not included in either the left sum or the right sum.
- Since elements can be negative, the cumulative sum does not strictly increase or decrease, meaning multiple equilibrium indices are possible.


### 2. Brute Solution

### Algorithm
1. Loop through each index $i$ from $0$ to $N-1$.
2. Initialize `leftSum = 0` and `rightSum = 0`.
3. Compute `leftSum` by summing all elements from index $0$ to $i-1$.
4. Compute `rightSum` by summing all elements from index $i+1$ to $N-1$.
5. If `leftSum == rightSum`, return $i$.
6. If the loop completes without finding any such index, return `-1`.

### Dry Run
Input: `arr = [1, 7, 3, 6, 5, 6]`
- $i=0$: `leftSum` = 0, `rightSum` = 7+3+6+5+6 = 27. (Not equal)
- $i=1$: `leftSum` = 1, `rightSum` = 3+6+5+6 = 20. (Not equal)
- $i=2$: `leftSum` = 1+7 = 8, `rightSum` = 6+5+6 = 17. (Not equal)
- $i=3$: `leftSum` = 1+7+3 = 11, `rightSum` = 5+6 = 11. (Equal! Return index 3)

### Complexity
- **Time Complexity:** $O(N^2)$ because for each element, we traverse the rest of the array to calculate the sums.
- **Space Complexity:** $O(1)$ auxiliary space.

### Pros and Cons
- **Pros:** Extremely easy to understand and implement.
- **Cons:** Too slow for large inputs ($N \ge 10^5$ will time out).

### C++17 Code
```cpp
#include <vector>
#include <numeric>

class Solution {
public:
    int findEquilibriumIndexBrute(const std::vector<int>& arr) {
        int n = arr.size();
        for (int i = 0; i < n; ++i) {
            long long leftSum = 0;
            long long rightSum = 0;
            
            // Calculate left sum
            for (int j = 0; j < i; ++j) {
                leftSum += arr[j];
            }
            
            // Calculate right sum
            for (int k = i + 1; k < n; ++k) {
                rightSum += arr[k];
            }
            
            if (leftSum == rightSum) {
                return i; // First equilibrium index found
            }
        }
        return -1; // No equilibrium index found
    }
};
```


### 3. Optimal Solution & Complexities

### Best Approach
We can optimize the space of the "Better Solution" by using a single variable for `totalSum` and updating a running `leftSum` on the fly. 

### Algorithm
1. Compute the `totalSum` of all elements in the array.
2. Initialize `leftSum = 0`.
3. Loop through the array from $i = 0$ to $N-1$:
   - Calculate `rightSum = totalSum - leftSum - arr[i]`.
   - If `leftSum == rightSum`, then $i$ is the equilibrium index. Return $i$.
   - Add `arr[i]` to `leftSum`.
4. If the loop ends, return `-1`.

### Full Dry Run
Input: `arr = [1, 7, 3, 6, 5, 6]`
- `totalSum` = $1+7+3+6+5+6 = 28$.
- `leftSum` = $0$.
- Step-by-step iteration details are shown in the **Dry Run** section below.

### Why Optimal?
- We only visit each element twice (once for total sum, once to find the index). Time is $O(N)$.
- We only store a few scalar variables (`totalSum`, `leftSum`, `rightSum`). Space is $O(1)$.
- We cannot do better than $O(N)$ time because we must examine every element at least once to compute the sum.

### Beautiful C++17 Code
```cpp
#include <vector>
#include <numeric>

class Solution {
public:
    int pivotIndex(const std::vector<int>& nums) {
        // totalSum stores the sum of all elements in the array
        long long totalSum = 0;
        for (int num : nums) {
            totalSum += num;
        }
        
        // leftSum keeps track of the running sum of elements to the left of index i
        long long leftSum = 0;
        
        for (size_t i = 0; i < nums.size(); ++i) {
            // rightSum is the total sum minus the left sum and the current element
            long long rightSum = totalSum - leftSum - nums[i];
            
            if (leftSum == rightSum) {
                return i; // First equilibrium index
            }
            
            // Add current element to leftSum for the next iteration
            leftSum += nums[i];
        }
        
        return -1; // No equilibrium index exists
    }
};
```


#### Complexity Summary

### Comparison Table

| Approach | Time Complexity | Space Complexity | Overflow Safety |
|:---|:---:|:---:|:---:|
| **Brute Force** | $O(N^2)$ | $O(1)$ | No |
| **Better (Prefix Array)** | $O(N)$ | $O(N)$ | Yes (using `long long`) |
| **Optimal (Two Variables)** | $O(N)$ | $O(1)$ | Yes (using `long long`) |

### Optimal Space-Time Complexity
- **Time Complexity:**
  - **Best Case:** $O(N)$ (e.g., equilibrium is at index 0, but we still need one pass to calculate the total sum).
  - **Average Case:** $O(N)$
  - **Worst Case:** $O(N)$
- **Space Complexity:**
  - $O(1)$ auxiliary space as we only use variables to store the sums.


---

## Q14 Max Product Subarray

### 1. Problem Explanation

Given an integer array `nums`, find a contiguous non-empty subarray that has the largest product, and return the product.

### Input
- An array of integers `nums` of size $N$.

### Output
- An integer representing the maximum product of a contiguous subarray.

### Constraints
- $1 \le N \le 2 \times 10^4$
- $-10 \le \text{nums}[i] \le 10$
- The product of any prefix or suffix of `nums` is guaranteed to fit in a 64-bit integer (and usually a 32-bit integer for standard test suites).

### Observations
- If the array contains only positive numbers, the maximum product is the product of the entire array.
- If there are negative numbers:
  - An even number of negative numbers results in a positive product.
  - An odd number of negative numbers requires us to exclude at least one negative number (and its adjacent elements on one side) to get the maximum positive product.
  - A negative number multiplied by a negative number becomes positive. This means a very small negative product (`min_so_far`) could suddenly become a very large positive product (`max_so_far`) when multiplied by another negative number.
- Zeros reset the product to 0. A subarray cannot "extend" through a zero without its product becoming 0. Hence, zeros effectively divide the array into smaller independent subproblems.


### 2. Brute Solution

### Algorithm
1. Initialize `maxProduct` to `nums[0]` (or negative infinity).
2. For each start index `i` from `0` to $N-1$:
   - Initialize `currentProduct = 1`.
   - For each end index `j` from `i` to $N-1$:
     - Multiply `currentProduct` by `nums[j]`.
     - Update `maxProduct = max(maxProduct, currentProduct)`.
3. Return `maxProduct`.

### Dry Run
Input: `nums = [2, 3, -2, 4]`
- `i = 0`:
  - `j = 0`: `currentProduct = 2`. `maxProduct = 2`.
  - `j = 1`: `currentProduct = 6`. `maxProduct = 6`.
  - `j = 2`: `currentProduct = -12`. `maxProduct = 6`.
  - `j = 3`: `currentProduct = -48`. `maxProduct = 6`.
- `i = 1`:
  - `j = 1`: `currentProduct = 3`. `maxProduct = 6`.
  - `j = 2`: `currentProduct = -6`. `maxProduct = 6`.
  ... and so on.
Return `6`.

### Complexity
- **Time Complexity:** $O(N^2)$ due to nested loops.
- **Space Complexity:** $O(1)$ auxiliary space.

### Pros and Cons
- **Pros:** Simple, handles zeros and negatives naturally.
- **Cons:** Inefficient. TLE for $N \ge 10^4$.

### C++17 Code
```cpp
#include <vector>
#include <algorithm>

class Solution {
public:
    int maxProductBrute(const std::vector<int>& nums) {
        int n = nums.size();
        int maxProduct = nums[0];
        
        for (int i = 0; i < n; ++i) {
            int currentProduct = 1;
            for (int j = i; j < n; ++j) {
                currentProduct *= nums[j];
                maxProduct = std::max(maxProduct, currentProduct);
            }
        }
        
        return maxProduct;
    }
};
```


### 3. Optimal Solution & Complexities

### Space-Optimized Dynamic Programming (Kadane's Variation)
Since `dpMax[i]` and `dpMin[i]` only depend on `dpMax[i-1]` and `dpMin[i-1]`, we can replace the arrays with two scalar variables: `max_so_far` and `min_so_far`.

### Algorithm
1. Initialize `max_so_far = nums[0]`, `min_so_far = nums[0]`, and `ans = nums[0]`.
2. Iterate `i` from `1` to $N-1$:
   - If `nums[i]` is negative, swap `max_so_far` and `min_so_far`. (Multiplying by a negative number flips the inequality, so the previous minimum product becomes the candidate for the new maximum, and vice versa).
   - Update `max_so_far = max(nums[i], max_so_far * nums[i])`.
   - Update `min_so_far = min(nums[i], min_so_far * nums[i])`.
   - Update `ans = max(ans, max_so_far)`.
3. Return `ans`.

### Why Optimal?
- It processes each element in a single pass: $O(N)$ time.
- It uses only three variables for tracking the state: $O(1)$ auxiliary space.
- We cannot achieve better than $O(N)$ time because any element could potentially be part of the optimal subarray.

### Beautiful C++17 Code
```cpp
#include <vector>
#include <algorithm>

class Solution {
public:
    int maxProduct(const std::vector<int>& nums) {
        if (nums.empty()) return 0;
        
        // Track the max and min products ending at the current position
        long long maxSoFar = nums[0];
        long long minSoFar = nums[0];
        long long globalMax = nums[0];
        
        for (size_t i = 1; i < nums.size(); ++i) {
            long long current = nums[i];
            
            // If the current element is negative, multiplying by it will
            // swap the roles of maximum and minimum products.
            if (current < 0) {
                std::swap(maxSoFar, minSoFar);
            }
            
            // The candidate for max product is either the current element itself
            // or the current element multiplied by the previous max
            maxSoFar = std::max(current, maxSoFar * current);
            
            // The candidate for min product is either the current element itself
            // or the current element multiplied by the previous min
            minSoFar = std::min(current, minSoFar * current);
            
            // Update the overall maximum product found so far
            globalMax = std::max(globalMax, maxSoFar);
        }
        
        return static_cast<int>(globalMax);
    }
};
```


#### Complexity Summary

### Comparison Table

| Approach | Time Complexity | Space Complexity | Extra Space Usage |
|:---|:---:|:---:|:---|
| **Brute Force** | $O(N^2)$ | $O(1)$ | No extra arrays |
| **DP with Arrays** | $O(N)$ | $O(N)$ | Two DP arrays of size $N$ |
| **Optimal (Space-Optimized DP)** | $O(N)$ | $O(1)$ | Only 3 variables |

### Optimal Space-Time Complexity
- **Time Complexity:**
  - **Best Case:** $O(N)$
  - **Average Case:** $O(N)$
  - **Worst Case:** $O(N)$
- **Space Complexity:**
  - $O(1)$ auxiliary space as we only maintain state variables.


---

## Q15 Subarray Given Sum

### 1. Problem Explanation

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


### 2. Brute Solution

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


### 3. Optimal Solution & Complexities

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


#### Complexity Summary

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

## Q16 Count Inversions

### 1. Problem Explanation

Given an array of integers `arr`, find the **Inversion Count** of the array. 

Two elements `arr[i]` and `arr[j]` form an inversion if:
1. `i < j` (the element appears earlier in the array)
2. `arr[i] > arr[j]` (the earlier element is strictly greater than the later element)

### Input
- An array of integers `arr` of size $N$.

### Output
- A 64-bit integer (`long long` in C++) representing the total number of inversions.

### Constraints
- $1 \le N \le 5 \times 10^5$
- $1 \le \text{arr}[i] \le 10^6$

### Observations
- If the array is already sorted in ascending order, the inversion count is `0`.
- If the array is sorted in descending order, the inversion count is at its maximum, which is:
  $$\frac{N(N - 1)}{2}$$
- The inversion count indicates how far the array is from being sorted.


### 2. Brute Solution

### Algorithm
1. Initialize `inversionCount = 0`.
2. Loop `i` from `0` to $N-2$.
3. Loop `j` from `i + 1` to $N-1$.
4. If `arr[i] > arr[j]`, increment `inversionCount`.
5. Return `inversionCount`.

### Dry Run
Input: `arr = [8, 4, 2, 1]`
- `i = 0` (`arr[i] = 8`):
  - `j = 1` (`4`): $8 > 4$ (count = 1)
  - `j = 2` (`2`): $8 > 2$ (count = 2)
  - `j = 3` (`1`): $8 > 1$ (count = 3)
- `i = 1` (`arr[i] = 4`):
  - `j = 2` (`2`): $4 > 2$ (count = 4)
  - `j = 3` (`1`): $4 > 1$ (count = 5)
- `i = 2` (`arr[i] = 2`):
  - `j = 3` (`1`): $2 > 1$ (count = 6)
Total Inversions = 6.

### Complexity
- **Time Complexity:** $O(N^2)$ due to nested loops.
- **Space Complexity:** $O(1)$ auxiliary space.

### C++17 Code
```cpp
#include <vector>

class Solution {
public:
    long long countInversionsBrute(const std::vector<int>& arr) {
        long long inversionCount = 0;
        int n = arr.size();
        for (int i = 0; i < n; ++i) {
            for (int j = i + 1; j < n; ++j) {
                if (arr[i] > arr[j]) {
                    inversionCount++;
                }
            }
        }
        return inversionCount;
    }
};
```


### 3. Optimal Solution & Complexities

### Modified Merge Sort
We merge two sorted subarrays and count how many elements from the left subarray are larger than the element being merged from the right subarray.

### Algorithm
1. Define a recursive function `mergeAndCount(arr, temp, left, mid, right)`:
   - Pointer `i` starts at `left`, pointer `j` starts at `mid + 1`, and `k` starts at `left`.
   - Initialize `invCount = 0`.
   - While `i <= mid` and `j <= right`:
     - If `arr[i] <= arr[j]`, copy `arr[i]` to `temp[k]` and increment `i` and `k`.
     - If `arr[i] > arr[j]`, copy `arr[j]` to `temp[k]`. All elements from `arr[i]` to `arr[mid]` are greater than `arr[j]`. So, add `mid - i + 1` to `invCount`. Increment `j` and `k`.
   - Copy remaining elements of Left and Right subarrays to `temp`.
   - Copy sorted elements from `temp` back to the original array `arr`.
   - Return `invCount`.
2. Define a recursive function `mergeSortAndCount(arr, temp, left, right)`:
   - If `left >= right`, return 0.
   - `mid = left + (right - left) / 2`.
   - `invCount = mergeSortAndCount(arr, temp, left, mid)` + `mergeSortAndCount(arr, temp, mid + 1, right)` + `mergeAndCount(arr, temp, left, mid, right)`.
   - Return `invCount`.

### Why Optimal?
- It reduces the time complexity to $O(N \log N)$, which is the theoretical lower limit for comparison-based sorting.
- It processes elements systematically, counting all inversions without missing any.

### Beautiful C++17 Code
```cpp
#include <vector>

class Solution {
private:
    long long mergeAndCount(std::vector<int>& arr, std::vector<int>& temp, int left, int mid, int right) {
        int i = left;      // Starting index for left subarray
        int j = mid + 1;   // Starting index for right subarray
        int k = left;      // Starting index to be filled in temp array
        long long invCount = 0;

        while (i <= mid && j <= right) {
            if (arr[i] <= arr[j]) {
                temp[k++] = arr[i++];
            } else {
                // There is an inversion because arr[i] > arr[j]
                // Since left subarray is sorted, all elements from arr[i] to arr[mid]
                // are greater than arr[j].
                temp[k++] = arr[j++];
                invCount += (mid - i + 1);
            }
        }

        // Copy the remaining elements of left subarray, if any
        while (i <= mid) {
            temp[k++] = arr[i++];
        }

        // Copy the remaining elements of right subarray, if any
        while (j <= right) {
            temp[k++] = arr[j++];
        }

        // Copy back the merged elements to original array
        for (i = left; i <= right; i++) {
            arr[i] = temp[i];
        }

        return invCount;
    }

    long long mergeSortAndCount(std::vector<int>& arr, std::vector<int>& temp, int left, int right) {
        long long invCount = 0;
        if (left < right) {
            int mid = left + (right - left) / 2;

            // Inversions in the left half
            invCount += mergeSortAndCount(arr, temp, left, mid);

            // Inversions in the right half
            invCount += mergeSortSortAndCount(arr, temp, mid + 1, right);

            // Inversions across the left and right halves
            invCount += mergeAndCount(arr, temp, left, mid, right);
        }
        return invCount;
    }

public:
    long long countInversions(std::vector<int>& arr) {
        int n = arr.size();
        std::vector<int> temp(n);
        return mergeSortAndCount(arr, temp, 0, n - 1);
    }
};
```


#### Complexity Summary

### Comparison Table

| Approach | Time Complexity | Space Complexity | Stack Space |
|:---|:---:|:---:|:---:|
| **Brute Force** | $O(N^2)$ | $O(1)$ | $O(1)$ |
| **Optimal (Modified Merge Sort)** | $O(N \log N)$ | $O(N)$ | $O(\log N)$ |

### Optimal Space-Time Complexity
- **Time Complexity:**
  - **Best Case:** $O(N \log N)$ (The division tree has depth $\log N$, and we do linear work at each level).
  - **Average Case:** $O(N \log N)$
  - **Worst Case:** $O(N \log N)$
- **Space Complexity:**
  - $O(N)$ auxiliary space for the temporary merge array.


---

## Q17 Buy Sell Stock

### 1. Problem Explanation

You are given an array `prices` where `prices[i]` is the price of a given stock on the $i^{\text{th}}$ day.

You want to maximize your profit by choosing a **single day** to buy one stock and choosing a **different day in the future** to sell that stock.

Return the maximum profit you can achieve from this transaction. If you cannot achieve any profit, return `0`.

### Input
- An array of integers `prices` of size $N$.

### Output
- An integer representing the maximum profit.

### Constraints
- $1 \le N \le 10^5$
- $0 \le \text{prices}[i] \le 10^4$

### Observations
- We must buy the stock before we can sell it. This means the buy day index must be strictly less than the sell day index.
- If the prices are sorted in descending order, the maximum profit is `0` because any transaction would result in a loss.
- We want to find the maximum difference between two elements `prices[j] - prices[i]` such that $j > i$.


### 2. Brute Solution

### Algorithm
1. Initialize `maxProfit = 0`.
2. Loop `i` from `0` to $N-2$:
   - Loop `j` from `i + 1` to $N-1$:
     - Calculate `profit = prices[j] - prices[i]`.
     - If `profit > maxProfit`, update `maxProfit = profit`.
3. Return `maxProfit`.

### Dry Run
Input: `prices = [7, 1, 5, 3, 6, 4]`
- `i = 0` (buy = 7):
  - `j = 1` (sell = 1): profit = -6
  - ...
- `i = 1` (buy = 1):
  - `j = 2` (sell = 5): profit = 4
  - `j = 3` (sell = 3): profit = 2
  - `j = 4` (sell = 6): profit = 5 (maxProfit = 5)
  - `j = 5` (sell = 4): profit = 3
- Output: `5`.

### Complexity
- **Time Complexity:** $O(N^2)$ due to nested loops.
- **Space Complexity:** $O(1)$ auxiliary space.

### C++17 Code
```cpp
#include <vector>
#include <algorithm>

class Solution {
public:
    int maxProfitBrute(const std::vector<int>& prices) {
        int n = prices.size();
        int maxProfit = 0;
        
        for (int i = 0; i < n; ++i) {
            for (int j = i + 1; j < n; ++j) {
                int profit = prices[j] - prices[i];
                maxProfit = std::max(maxProfit, profit);
            }
        }
        
        return maxProfit;
    }
};
```


### 3. Optimal Solution & Complexities

### Single Pass (Track Minimum Price)
We maintain the lowest price seen so far and compare each day's price with this minimum price to calculate the maximum profit.

### Algorithm
1. Initialize `minPrice = prices[0]` (or infinity) and `maxProfit = 0`.
2. Iterate through `prices` from index `1` to $N-1$:
   - Calculate the current profit: `prices[i] - minPrice`.
   - Update `maxProfit = max(maxProfit, currentProfit)`.
   - Update `minPrice = min(minPrice, prices[i])`.
3. Return `maxProfit`.

### Why Optimal?
- It processes the array in a single pass: $O(N)$ time.
- It uses only two variables: $O(1)$ auxiliary space.
- We must look at each price at least once to know if it yields the best buy or sell day, so we cannot do better than $O(N)$ time.

### Beautiful C++17 Code
```cpp
#include <vector>
#include <algorithm>

class Solution {
public:
    int maxProfit(const std::vector<int>& prices) {
        if (prices.empty()) return 0;
        
        // Track the minimum price encountered so far
        int minPrice = prices[0];
        
        // Track the maximum profit we can get
        int maxProfit = 0;
        
        for (size_t i = 1; i < prices.size(); ++i) {
            // If selling today yields a higher profit, update maxProfit
            int currentProfit = prices[i] - minPrice;
            maxProfit = std::max(maxProfit, currentProfit);
            
            // Update minPrice if the current day's price is lower
            minPrice = std::min(minPrice, prices[i]);
        }
        
        return maxProfit;
    }
};
```


#### Complexity Summary

### Comparison Table

| Approach | Time Complexity | Space Complexity | Number of Passes |
|:---|:---:|:---:|:---:|
| **Brute Force** | $O(N^2)$ | $O(1)$ | $N$ |
| **Suffix Max Array** | $O(N)$ | $O(N)$ | 2 |
| **Optimal (Single Pass)** | $O(N)$ | $O(1)$ | 1 |

### Optimal Space-Time Complexity
- **Time Complexity:**
  - **Best Case:** $O(N)$
  - **Average Case:** $O(N)$
  - **Worst Case:** $O(N)$
- **Space Complexity:**
  - $O(1)$ auxiliary space as we only use scalar variables.


---

## Q18 Buy Sell Stock II

### 1. Problem Explanation

You are given an integer array `prices` where `prices[i]` is the price of a given stock on the $i^{\text{th}}$ day.

On each day, you may decide to buy and/or sell the stock. You can only hold **at most one** share of the stock at any time. However, you can buy it and then sell it on the **same day**.

Find and return the **maximum profit** you can achieve.

### Input
- An array of integers `prices` of size $N$.

### Output
- An integer representing the maximum profit.

### Constraints
- $1 \le N \le 3 \times 10^4$
- $0 \le \text{prices}[i] \le 10^4$

### Observations
- We can make as many transactions as we want.
- Since we can buy and sell on the same day, any upward price movement between consecutive days can be captured as profit.
- For example, if prices are `[1, 5, 8]`:
  - Buying on day 0 (price 1) and selling on day 2 (price 8) yields a profit of $8 - 1 = 7$.
  - This is exactly equivalent to: (buying on day 0, selling on day 1 for $5 - 1 = 4$) + (buying on day 1, selling on day 2 for $8 - 5 = 3$). Total profit = $4 + 3 = 7$.
  - Therefore, we can maximize profit by simply summing up all positive differences between consecutive days.


### 2. Brute Solution

### Algorithm
We use a recursive helper function `calculateProfit(index, buy)`:
1. `index`: Current day we are evaluating.
2. `buy`: Boolean representing if we can buy (true) or sell (false).
3. Base case: If `index >= N`, return 0.
4. If `buy` is true:
   - Option 1 (Buy): `-prices[index] + calculateProfit(index + 1, false)`
   - Option 2 (Skip): `calculateProfit(index + 1, true)`
   - Return `max(Option 1, Option 2)`.
5. If `buy` is false:
   - Option 1 (Sell): `prices[index] + calculateProfit(index + 1, true)`
   - Option 2 (Skip): `calculateProfit(index + 1, false)`
   - Return `max(Option 1, Option 2)`.

### Complexity
- **Time Complexity:** $O(2^N)$ because at each step we have 2 choices.
- **Space Complexity:** $O(N)$ auxiliary stack space.

### C++17 Code
```cpp
#include <vector>
#include <algorithm>

class Solution {
private:
    int calculateProfit(const std::vector<int>& prices, int index, bool buy) {
        if (index >= prices.size()) {
            return 0;
        }
        
        int profit = 0;
        if (buy) {
            // Option 1: Buy today
            int buyToday = -prices[index] + calculateProfit(prices, index + 1, false);
            // Option 2: Skip buying today
            int skipToday = calculateProfit(prices, index + 1, true);
            profit = std::max(buyToday, skipToday);
        } else {
            // Option 1: Sell today
            int sellToday = prices[index] + calculateProfit(prices, index + 1, true);
            // Option 2: Skip selling today
            int skipToday = calculateProfit(prices, index + 1, false);
            profit = std::max(sellToday, skipToday);
        }
        return profit;
    }

public:
    int maxProfitBrute(const std::vector<int>& prices) {
        return calculateProfit(prices, 0, true);
    }
};
```


### 3. Optimal Solution & Complexities

### Simple Greedy Approach
Instead of finding peaks and valleys, we simply add the difference between consecutive days if the price increases.

### Algorithm
1. Initialize `maxProfit = 0`.
2. Iterate `i` from `1` to $N-1$:
   - If `prices[i] > prices[i-1]`:
     - Add `prices[i] - prices[i-1]` to `maxProfit`.
3. Return `maxProfit`.

### Why Optimal?
- It makes only one pass over the array: $O(N)$ time.
- It uses no extra variables besides the profit accumulator: $O(1)$ space.
- The logic is extremely simple and avoids nested loops or complex index updates.

### Beautiful C++17 Code
```cpp
#include <vector>

class Solution {
public:
    int maxProfit(const std::vector<int>& prices) {
        int maxProfit = 0;
        
        // Loop starts from day 1 to compare with the previous day
        for (size_t i = 1; i < prices.size(); ++i) {
            // If the price increases, buy on day i-1 and sell on day i
            if (prices[i] > prices[i - 1]) {
                maxProfit += prices[i] - prices[i - 1];
            }
        }
        
        return maxProfit;
    }
};
```


#### Complexity Summary

### Comparison Table

| Approach | Time Complexity | Space Complexity | Stack Space |
|:---|:---:|:---:|:---:|
| **Brute Force (Recursion)** | $O(2^N)$ | $O(1)$ | $O(N)$ |
| **Peak-Valley** | $O(N)$ | $O(1)$ | $O(1)$ |
| **Optimal (Greedy)** | $O(N)$ | $O(1)$ | $O(1)$ |

### Optimal Space-Time Complexity
- **Time Complexity:**
  - **Best Case:** $O(N)$
  - **Average Case:** $O(N)$
  - **Worst Case:** $O(N)$
- **Space Complexity:**
  - $O(1)$ auxiliary space.


---

## Q19 Rearrange Alt Sign

### 1. Problem Explanation

Given an array of integers `nums`, rearrange the elements such that they follow an alternating pattern of positive and negative numbers. 

### Case 1: LeetCode Version (Equal Counts)
- **Input:** An array `nums` of even length $N$, containing exactly $N/2$ positive integers and $N/2$ negative integers.
- **Output:** Rearranged array starting with a positive integer, followed by a negative integer, and alternating thereafter. The relative order of same-sign integers must be preserved (stable rearrangement).
- **Constraints:**
  - $2 \le N \le 2 \times 10^5$
  - `nums` length is even.
  - Exactly $N/2$ positive and $N/2$ negative integers.
  - $-10^5 \le nums[i] \le 10^5$ (no zeros).

### Case 2: General Version (Unequal Counts)
- **Input:** An array `nums` of length $N$ containing any number of positive and negative integers.
- **Output:** Rearranged array starting with a positive integer, followed by a negative integer, and alternating thereafter. If one sign runs out of elements, the remaining elements of the other sign should be appended to the end of the array while maintaining their relative order.
- **Constraints:**
  - $1 \le N \le 2 \times 10^5$
  - $-10^5 \le nums[i] \le 10^5$ (treat $0$ as positive or omit based on standard convention; usually, assume no zeros or treat $0$ as positive).


### 2. Brute Solution

### Algorithm
1. Create two vectors: `positives` and `negatives`.
2. Iterate through the input array `nums`. If `nums[i] > 0`, push it to `positives`, else push it to `negatives`.
3. Clear or overwrite the original array `nums` (or create a new array).
4. Merge them back by alternating:
   - For `nums[i]` where `i` is even, pick from `positives`.
   - For `nums[i]` where `i` is odd, pick from `negatives`.

### Dry Run
Input: `nums = [1, -2, 3, -4]`
1. Separation:
   - `positives = [1, 3]`
   - `negatives = [-2, -4]`
2. Reconstruction:
   - `i = 0` (even): `nums[0] = positives[0]` -> `nums = [1, -2, 3, -4]`
   - `i = 1` (odd): `nums[1] = negatives[0]` -> `nums = [1, -2, 3, -4]`
   - `i = 2` (even): `nums[2] = positives[1]` -> `nums = [1, -2, 3, -4]`
   - `i = 3` (odd): `nums[3] = negatives[1]` -> `nums = [1, -2, 3, -4]`
Output: `[1, -2, 3, -4]`

### Complexity
- **Time Complexity:** $O(N)$ because we iterate through the array twice (once for filtering, once for merging).
- **Space Complexity:** $O(N)$ to store the filtered positive and negative elements.

### Pros and Cons
- **Pros:** Extremely simple to implement; handles unequal counts easily with minor modifications.
- **Cons:** Performs two passes and uses extra memory for two separate arrays.

### C++17 Code
```cpp
#include <vector>

std::vector<int> rearrangeArrayBrute(const std::vector<int>& nums) {
    std::vector<int> positives;
    std::vector<int> negatives;
    
    // Step 1: Separate elements by sign
    for (int num : nums) {
        if (num > 0) {
            positives.push_back(num);
        } else {
            negatives.push_back(num);
        }
    }
    
    std::vector<int> result(nums.size());
    int posIndex = 0, negIndex = 0;
    
    // Step 2: Merge in alternating sequence
    for (size_t i = 0; i < nums.size(); ++i) {
        if (i % 2 == 0) {
            result[i] = positives[posIndex++];
        } else {
            result[i] = negatives[negIndex++];
        }
    }
    return result;
}
```


### 3. Optimal Solution & Complexities

We present solutions for both cases.

### Case 1: Equal Counts (LeetCode) — Single-Pass Two-Pointer
We use a result array of size $N$ and two pointers:
- `posIdx` starting at `0` (even indices).
- `negIdx` starting at `1` (odd indices).
We iterate through `nums`. For every positive number, we assign it to `result[posIdx]` and add 2 to `posIdx`. For every negative number, we assign it to `result[negIdx]` and add 2 to `negIdx`.

### Case 2: Unequal Counts — Partition & Merge Leftovers
If positive and negative counts are unequal:
1. Extract positives and negatives into two arrays `pos` and `neg`.
2. Place them alternatingly as far as possible using two pointers `i` and `j`.
3. Append any remaining elements from the larger list.

### Dry Run (Case 1: Equal Counts)
Input: `nums = [3, 1, -2, -5, 2, -4]`
Initialize: `result = [0, 0, 0, 0, 0, 0]`, `posIdx = 0`, `negIdx = 1`

| Step | Element | Type | Action | posIdx | negIdx | result |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | 3 | Positive | `result[0] = 3` | 2 | 1 | `[3, 0, 0, 0, 0, 0]` |
| 2 | 1 | Positive | `result[2] = 1` | 4 | 1 | `[3, 0, 1, 0, 0, 0]` |
| 3 | -2 | Negative | `result[1] = -2` | 4 | 3 | `[3, -2, 1, 0, 0, 0]` |
| 4 | -5 | Negative | `result[3] = -5` | 4 | 5 | `[3, -2, 1, -5, 0, 0]` |
| 5 | 2 | Positive | `result[4] = 2` | 6 | 5 | `[3, -2, 1, -5, 2, 0]` |
| 6 | -4 | Negative | `result[5] = -4` | 6 | 7 | `[3, -2, 1, -5, 2, -4]` |

### C++17 Code

```cpp
#include <vector>
#include <stdexcept>

// Case 1: Equal Counts (Optimal Single-Pass)
std::vector<int> rearrangeArrayEqual(const std::vector<int>& nums) {
    int n = nums.size();
    std::vector<int> result(n);
    int posIndex = 0;
    int negIndex = 1;
    
    for (int num : nums) {
        if (num > 0) {
            result[posIndex] = num;
            posIndex += 2;
        } else {
            result[negIndex] = num;
            negIndex += 2;
        }
    }
    return result;
}

// Case 2: Unequal Counts (Optimal Stability-Preserving)
std::vector<int> rearrangeArrayGeneral(const std::vector<int>& nums) {
    std::vector<int> positives;
    std::vector<int> negatives;
    
    // Separate numbers
    for (int num : nums) {
        if (num >= 0) {
            positives.push_back(num);
        } else {
            negatives.push_back(num);
        }
    }
    
    std::vector<int> result;
    result.reserve(nums.size());
    
    size_t posPtr = 0, negPtr = 0;
    
    // Alternating placement
    while (posPtr < positives.size() && negPtr < negatives.size()) {
        result.push_back(positives[posPtr++]);
        result.push_back(negatives[negPtr++]);
    }
    
    // Append leftovers from positives
    while (posPtr < positives.size()) {
        result.push_back(positives[posPtr++]);
    }
    
    // Append leftovers from negatives
    while (negPtr < negatives.size()) {
        result.push_back(negatives[negPtr++]);
    }
    
    return result;
}
```


#### Complexity Summary

| Version / Approach | Time Complexity | Space Complexity | Notes |
| :--- | :--- | :--- | :--- |
| **Case 1: Equal (Brute)** | $O(N)$ | $O(N)$ | Two traversals, separate containers. |
| **Case 1: Equal (Optimal)** | $O(N)$ | $O(N)$ | Single traversal, direct placement. |
| **Case 2: Unequal (Optimal)** | $O(N)$ | $O(N)$ | Preserves stability; necessary extra space. |
| **Case 2: In-place Stable** | $O(N^2)$ | $O(1)$ | Shift elements rightwards to maintain order. |

*Why $O(N)$ space is optimal for stable solutions:* To maintain the relative order of positive and negative numbers without spending $O(N^2)$ time shifting elements, we must record the elements in sequential order, which mathematically requires $O(N)$ auxiliary space.


---

## Q20 Zero Sum Subarrays

### 1. Problem Explanation

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


### 2. Brute Solution

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


### 3. Optimal Solution & Complexities

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


#### Complexity Summary

- **Time Complexity:** 
  - **Average Case:** $O(N + K)$, where $N$ is the size of the array and $K$ is the number of zero-sum subarrays. The hash map lookups take $O(1)$ on average.
  - **Worst Case:** $O(N^2)$ if all elements are `0`. In this case, every prefix sum is `0`, and for each element, we loop through all previous indices. However, since the output size itself is $O(N^2)$, this is mathematically optimal because writing the output takes $O(K)$ time.
- **Space Complexity:** $O(N)$ for storing prefix sums in the hash map.


---

## Q21 Longest 01 Subarray

### 1. Problem Explanation

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


### 2. Brute Solution

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


### 3. Optimal Solution & Complexities

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


#### Complexity Summary

- **Time Complexity:** $O(N)$ on average. We perform a single loop through the array, and hash map operations (find and insert) take $O(1)$ time on average.
- **Space Complexity:** $O(N)$ since the hash map can grow to store at most $N$ unique prefix sums in the worst case (e.g., all 0s or all 1s).


---

## Q22 Min Rotated Sorted

### 1. Problem Explanation

Suppose an array of length $N$ sorted in ascending order is rotated between $1$ and $N$ times. For example, the array `nums = [0, 1, 2, 4, 5, 6, 7]` might become:
- `[4, 5, 6, 7, 0, 1, 2]` if it was rotated 4 times.
- `[0, 1, 2, 4, 5, 6, 7]` if it was rotated 7 times.

Given the sorted rotated array `nums` of **unique** elements, return the *minimum element* of this array.

### Examples
- **Input:** `nums = [3, 4, 5, 1, 2]`
- **Output:** 1
  *(Explanation: The original array was `[1, 2, 3, 4, 5]` rotated 3 times.)*

- **Input:** `nums = [4, 5, 6, 7, 0, 1, 2]`
- **Output:** 0
  *(Explanation: The minimum element is 0.)*

- **Input:** `nums = [11, 13, 15, 17]`
- **Output:** 11
  *(Explanation: The array was rotated 4 times (which is equivalent to 0 rotations).)*

### Constraints
- $1 \le N \le 5000$
- $-5000 \le nums[i] \le 5000$
- All integers in `nums` are **unique**.


### 2. Brute Solution

### Algorithm
1. Initialize `minVal = nums[0]`.
2. Iterate through the array starting from index 1.
3. Update `minVal = min(minVal, nums[i])`.
4. Return `minVal`.

### Complexity
- **Time Complexity:** $O(N)$ because we inspect all $N$ elements.
- **Space Complexity:** $O(1)$ auxiliary space.

### Pros and Cons
- **Pros:** Extremely simple to implement; works even if there are duplicates.
- **Cons:** Inefficient; fails to exploit the sorted structure of the array.

### C++17 Code
```cpp
#include <vector>
#include <algorithm>

int findMinBrute(const std::vector<int>& nums) {
    int minVal = nums[0];
    for (int num : nums) {
        minVal = std::min(minVal, num);
    }
    return minVal;
}
```


### 3. Optimal Solution & Complexities

### Algorithm
1. Initialize two pointers: `low = 0` and `high = nums.size() - 1`.
2. While `low < high`:
   - Calculate the midpoint: `mid = low + (high - low) / 2`.
   - Compare `nums[mid]` with `nums[high]`:
     - If `nums[mid] > nums[high]`, set `low = mid + 1` (the minimum is in the right half).
     - If `nums[mid] < nums[high]`, set `high = mid` (the minimum is at `mid` or in the left half).
3. Return `nums[low]`.

### C++17 Code

```cpp
#include <vector>

int findMin(const std::vector<int>& nums) {
    int low = 0;
    int high = nums.size() - 1;
    
    // Binary search loop
    while (low < high) {
        int mid = low + (high - low) / 2;
        
        // If mid element is greater than the high element,
        // the minimum element must be in the right unsorted subarray.
        if (nums[mid] > nums[high]) {
            low = mid + 1;
        } 
        // Otherwise, the right subarray is sorted, meaning the minimum
        // is in the left subarray (including mid itself).
        else {
            high = mid;
        }
    }
    
    // low and high converge to the index of the minimum element
    return nums[low];
}
```


#### Complexity Summary

- **Time Complexity:** $O(\log N)$. At each iteration, we compare the middle element and reduce the search space by half.
- **Space Complexity:** $O(1)$ as we only use a few pointer variables.


---

## Q23 Search Rotated Sorted

### 1. Problem Explanation

There is an integer array `nums` sorted in ascending order (with **distinct** values). Prior to being passed to your function, `nums` is possibly rotated at an unknown pivot index `k` ($1 \le k < nums.length$) such that the resulting array is `[nums[k], nums[k+1], ..., nums[n-1], nums[0], nums[1], ..., nums[k-1]]` (0-indexed).

For example, `[0, 1, 2, 4, 5, 6, 7]` might be rotated at pivot index `3` and become `[4, 5, 6, 7, 0, 1, 2]`.

Given the array `nums` **after** the possible rotation and an integer `target`, return the *index* of `target` if it is in `nums`, or `-1` if it is not in `nums`.

You must write an algorithm with $O(\log N)$ runtime complexity.

### Examples
- **Input:** `nums = [4, 5, 6, 7, 0, 1, 2]`, `target = 0`
- **Output:** 4

- **Input:** `nums = [4, 5, 6, 7, 0, 1, 2]`, `target = 3`
- **Output:** -1

- **Input:** `nums = [1]`, `target = 0`
- **Output:** -1

### Constraints
- $1 \le N \le 5000$
- $-10^4 \le nums[i] \le 10^4$
- All values of `nums` are **unique**.
- `nums` is an ascending array that is rotated.
- $-10^4 \le target \le 10^4$


### 2. Brute Solution

### Algorithm
1. Loop through `i` from `0` to `n - 1`.
2. If `nums[i] == target`, return `i`.
3. Return `-1` after the loop.

### Complexity
- **Time Complexity:** $O(N)$ since we may scan the entire array.
- **Space Complexity:** $O(1)$ auxiliary space.

### Pros and Cons
- **Pros:** Simple, handles duplicates automatically.
- **Cons:** Too slow, does not satisfy the $O(\log N)$ constraint.

### C++17 Code
```cpp
#include <vector>

int searchBrute(const std::vector<int>& nums, int target) {
    for (size_t i = 0; i < nums.size(); ++i) {
        if (nums[i] == target) {
            return static_cast<int>(i);
        }
    }
    return -1;
}
```


### 3. Optimal Solution & Complexities

### Algorithm
1. Initialize two pointers: `low = 0` and `high = nums.size() - 1`.
2. While `low <= high`:
   - Calculate midpoint: `mid = low + (high - low) / 2`.
   - If `nums[mid] == target`, return `mid`.
   - **Identify the sorted half:**
     - **If Left Half is Sorted (`nums[low] <= nums[mid]`):**
       - Check if target is within the left boundaries: `nums[low] <= target && target < nums[mid]`.
       - If yes, narrow search to left: `high = mid - 1`.
       - If no, search right: `low = mid + 1`.
     - **If Right Half is Sorted:**
       - Check if target is within the right boundaries: `nums[mid] < target && target <= nums[high]`.
       - If yes, narrow search to right: `low = mid + 1`.
       - If no, search left: `high = mid - 1`.
3. If the loop ends, return `-1`.

### C++17 Code

```cpp
#include <vector>

int search(const std::vector<int>& nums, int target) {
    int low = 0;
    int high = nums.size() - 1;
    
    while (low <= high) {
        int mid = low + (high - low) / 2;
        
        if (nums[mid] == target) {
            return mid;
        }
        
        // Check if the left half is sorted
        if (nums[low] <= nums[mid]) {
            // Check if target lies in the sorted left half
            if (nums[low] <= target && target < nums[mid]) {
                high = mid - 1; // Search left
            } else {
                low = mid + 1;  // Search right
            }
        } 
        // Otherwise, the right half must be sorted
        else {
            // Check if target lies in the sorted right half
            if (nums[mid] < target && target <= nums[high]) {
                low = mid + 1;  // Search right
            } else {
                high = mid - 1; // Search left
            }
        }
    }
    
    return -1; // Target not found
}
```


#### Complexity Summary

- **Time Complexity:** $O(\log N)$. At each iteration, we eliminate half of the search space, performing a constant number of comparisons.
- **Space Complexity:** $O(1)$ because we only use a few pointer variables.


---

## Q24 Merge Intervals

### 1. Problem Explanation

Given an array of `intervals` where `intervals[i] = [start_i, end_i]`, merge all overlapping intervals, and return an array of the non-overlapping intervals that cover all the input intervals.

### Examples
- **Input:** `intervals = [[1, 3], [2, 6], [8, 10], [15, 18]]`
- **Output:** `[[1, 6], [8, 10], [15, 18]]`
  *(Explanation: Since intervals `[1, 3]` and `[2, 6]` overlap, they are merged into `[1, 6]`.)*

- **Input:** `intervals = [[1, 4], [4, 5]]`
- **Output:** `[[1, 5]]`
  *(Explanation: Intervals `[1, 4]` and `[4, 5]` are considered overlapping because they touch at `4`.)*

### Constraints
- $1 \le intervals.length \le 10^4$
- `intervals[i].length == 2`
- $0 \le start_i \le end_i \le 10^4$


### 2. Brute Solution

### Algorithm
1. Keep track of which intervals have been merged using a boolean array `visited`.
2. Compare each interval `i` with every other interval `j`:
   - If they overlap and neither is visited:
     - Merge them by setting `start_i = min(start_i, start_j)` and `end_i = max(end_i, end_j)`.
     - Mark `j` as visited.
     - Reset the search to check the newly expanded interval against all others.
3. Push all unvisited (or newly formed) intervals into a result list.

### Complexity
- **Time Complexity:** $O(N^2)$ due to recursive merges and searching.
- **Space Complexity:** $O(N)$ for the `visited` array.

### C++17 Code
```cpp
#include <vector>
#include <algorithm>

std::vector<std::vector<int>> mergeBrute(std::vector<std::vector<int>>& intervals) {
    int n = intervals.size();
    if (n <= 1) return intervals;
    
    std::vector<bool> merged(n, false);
    
    for (int i = 0; i < n; ++i) {
        if (merged[i]) continue;
        bool hasMerge = true;
        while (hasMerge) {
            hasMerge = false;
            for (int j = 0; j < n; ++j) {
                if (i != j && !merged[j]) {
                    // Check overlap
                    if (std::max(intervals[i][0], intervals[j][0]) <= std::min(intervals[i][1], intervals[j][1])) {
                        intervals[i][0] = std::min(intervals[i][0], intervals[j][0]);
                        intervals[i][1] = std::max(intervals[i][1], intervals[j][1]);
                        merged[j] = true;
                        hasMerge = true; // Restart check with updated interval
                    }
                }
            }
        }
    }
    
    std::vector<std::vector<int>> result;
    for (int i = 0; i < n; ++i) {
        if (!merged[i]) {
            result.push_back(intervals[i]);
        }
    }
    return result;
}
```


### 3. Optimal Solution & Complexities

We present two implementations:
1. **Standard Optimal:** Uses a new list, highly readable.
2. **In-place Optimal:** Modifies the input vector directly to achieve $O(1)$ auxiliary space.

### Algorithm (In-Place Sweep)
1. Sort the `intervals` array in ascending order of start times: `intervals[i][0]`.
2. Initialize a write index `writeIdx = 0`.
3. Iterate `i` from `1` to `N - 1`:
   - If the current interval `intervals[i]` overlaps with the active merged interval at `intervals[writeIdx]`:
     - Update the end of the active interval: `intervals[writeIdx][1] = max(intervals[writeIdx][1], intervals[i][1])`.
   - If they do not overlap:
     - Increment `writeIdx`.
     - Copy `intervals[i]` to `intervals[writeIdx]`.
4. Resize the `intervals` vector to `writeIdx + 1` elements.
5. Return the modified `intervals` vector.

### C++17 Code

```cpp
#include <vector>
#include <algorithm>

// Version 1: Standard Optimal (using extra result vector)
std::vector<std::vector<int>> merge(std::vector<std::vector<int>>& intervals) {
    if (intervals.empty()) return {};
    
    // Sort by start times
    std::sort(intervals.begin(), intervals.end(), [](const auto& a, const auto& b) {
        return a[0] < b[0];
    });
    
    std::vector<std::vector<int>> merged;
    merged.push_back(intervals[0]);
    
    for (size_t i = 1; i < intervals.size(); ++i) {
        // If current interval overlaps with the last merged interval
        if (intervals[i][0] <= merged.back()[1]) {
            merged.back()[1] = std::max(merged.back()[1], intervals[i][1]);
        } else {
            // No overlap, append current interval
            merged.push_back(intervals[i]);
        }
    }
    return merged;
}

// Version 2: In-place Space Optimized (O(1) auxiliary space)
std::vector<std::vector<int>> mergeInPlace(std::vector<std::vector<int>>& intervals) {
    if (intervals.size() <= 1) return intervals;
    
    // Sort by start times
    std::sort(intervals.begin(), intervals.end(), [](const auto& a, const auto& b) {
        return a[0] < b[0];
    });
    
    int writeIdx = 0;
    
    for (size_t i = 1; i < intervals.size(); ++i) {
        // If current interval overlaps with the interval at writeIdx
        if (intervals[i][0] <= intervals[writeIdx][1]) {
            intervals[writeIdx][1] = std::max(intervals[writeIdx][1], intervals[i][1]);
        } else {
            // Move write pointer and copy the non-overlapping interval
            writeIdx++;
            intervals[writeIdx] = intervals[i];
        }
    }
    
    // Shrink vector to keep only the merged elements
    intervals.resize(writeIdx + 1);
    return intervals;
}
```


#### Complexity Summary

- **Time Complexity:** $O(N \log N)$ where $N$ is the number of intervals. Sorting takes $O(N \log N)$ time, and the linear sweep takes $O(N)$ time.
- **Space Complexity:**
  - **Standard Version:** $O(N)$ to store the merged intervals.
  - **In-Place Version:** $O(1)$ auxiliary space (ignoring sorting recursion stack space which is $O(\log N)$).


---

## Q25 Circular Subarray Sum

### 1. Problem Explanation

Given a circular integer array `arr` of size `N`, find the maximum possible sum of a non-empty subarray in the circular array.

A circular array means the end of the array connects to the beginning of the array. Formally, the next element of `arr[N-1]` is `arr[0]`. A subarray here can be a normal subarray (contiguous block without wrapping) or a circular subarray (wrapping from the end of the array back to the beginning). Note that a subarray cannot contain any element of the original array more than once (i.e., its maximum length is `N`).

### Input
- An integer array `arr` of size `N`.

### Output
- An integer representing the maximum subarray sum.

### Constraints
- $1 \le N \le 10^5$
- $-10^4 \le arr[i] \le 10^4$

### Observations
1. If the array is NOT circular, this is the classic **Maximum Subarray Sum** problem solvable in $O(N)$ time using **Kadane's Algorithm**.
2. If the maximum subarray wraps around, it contains a prefix of the array and a suffix of the array.
3. The elements that are *not* included in this wrapping subarray must form a contiguous subarray in the middle.
4. To maximize the sum of the wrapping prefix and suffix, we must minimize the sum of the excluded contiguous middle subarray.
5. $\text{Circular Subarray Sum} = \text{Total Sum} - \text{Minimum Subarray Sum}$.
6. If all elements in the array are negative, the minimum subarray sum will equal the total sum, making the circular sum 0 (representing an empty subarray). However, since the problem specifies a *non-empty* subarray, we must handle this edge case.


### 2. Brute Solution

### Algorithm
1. Initialize `max_sum = INT_MIN`.
2. Loop through every starting index `i` from `0` to `N - 1`.
3. For each start index, initialize `current_sum = 0`.
4. Loop through lengths `L` from `1` to `N`.
5. Add the element at index `(i + L - 1) % N` to `current_sum`.
6. Update `max_sum = max(max_sum, current_sum)`.
7. Return `max_sum`.

### Dry Run
Input: `arr = [1, -2, 3]`
- `i = 0`:
  - `L = 1`: sum = `arr[0] = 1`. `max_sum` = 1.
  - `L = 2`: sum = `1 + arr[1] = 1 - 2 = -1`.
  - `L = 3`: sum = `-1 + arr[2] = 2`. `max_sum` = 2.
- `i = 1`:
  - `L = 1`: sum = `arr[1] = -2`.
  - `L = 2`: sum = `-2 + arr[2] = 1`.
  - `L = 3`: sum = `1 + arr[0] = 2`.
- `i = 2`:
  - `L = 1`: sum = `arr[2] = 3`. `max_sum` = 3.
  - `L = 2`: sum = `3 + arr[0] = 4`. `max_sum` = 4.
  - `L = 3`: sum = `4 + arr[1] = 2`.
Output: `4`

### Complexity Analysis
- **Time Complexity:** $O(N^2)$ because we have nested loops of size $N$.
- **Space Complexity:** $O(1)$ auxiliary space.

### Pros and Cons
- **Pros:** Extremely simple to understand and implement.
- **Cons:** TLE (Time Limit Exceeded) for $N \ge 10^5$.

### Brute Force C++17 Code
```cpp
#include <vector>
#include <algorithm>
#include <climits>

class Solution {
public:
    int maxSubarraySumCircularBrute(const std::vector<int>& arr) {
        int n = arr.size();
        int max_sum = INT_MIN;

        // Try every starting position
        for (int i = 0; i < n; ++i) {
            int current_sum = 0;
            // Try every subarray length from 1 to n
            for (int len = 1; len <= n; ++len) {
                int idx = (i + len - 1) % n;
                current_sum += arr[idx];
                max_sum = std::max(max_sum, current_sum);
            }
        }
        return max_sum;
    }
};
```


### 3. Optimal Solution & Complexities

### Algorithm
1. Compute the maximum subarray sum of the array using standard Kadane's algorithm. Let this be `max_normal`.
2. Compute the minimum subarray sum of the array using a modified Kadane's algorithm. Let this be `min_normal`.
3. Compute the sum of all elements in the array. Let this be `total_sum`.
4. If `total_sum == min_normal`, all elements are negative. Return `max_normal`.
5. Otherwise, return `max(max_normal, total_sum - min_normal)`.

### Why this is Optimal
It runs in a single pass (or two simple linear passes) over the array. Since we must read every element at least once to determine the maximum sum, $O(N)$ is the absolute lower bound for time complexity. We achieve this with $O(1)$ extra space.

### C++17 Optimal Code
```cpp
#include <vector>
#include <algorithm>
#include <numeric>

class Solution {
public:
    int maxSubarraySumCircular(const std::vector<int>& arr) {
        if (arr.empty()) return 0;

        int total_sum = 0;
        
        // Variables for finding maximum subarray sum (Standard Kadane)
        int max_so_far = arr[0];
        int curr_max = arr[0];
        
        // Variables for finding minimum subarray sum (Modified Kadane)
        int min_so_far = arr[0];
        int curr_min = arr[0];
        
        total_sum += arr[0];

        for (size_t i = 1; i < arr.size(); ++i) {
            total_sum += arr[i];
            
            // Standard Kadane to find maximum subarray sum
            curr_max = std::max(arr[i], curr_max + arr[i]);
            max_so_far = std::max(max_so_far, curr_max);
            
            // Modified Kadane to find minimum subarray sum
            curr_min = std::min(arr[i], curr_min + arr[i]);
            min_so_far = std::min(min_so_far, curr_min);
        }
        
        // If all numbers are negative, total_sum will equal min_so_far.
        // In this case, total_sum - min_so_far = 0, which corresponds to an empty subarray.
        // We must return the maximum single element (max_so_far).
        if (total_sum == min_so_far) {
            return max_so_far;
        }
        
        // Return the maximum of the non-wrapped and wrapped subarray sum
        return std::max(max_so_far, total_sum - min_so_far);
    }
};
```


#### Complexity Summary

- **Time Complexity:** 
  - **Best Case:** $O(N)$ when the array is processed in a single pass.
  - **Average Case:** $O(N)$.
  - **Worst Case:** $O(N)$ as we only perform constant-time operations for each element.
- **Space Complexity:**
  - **Auxiliary Space:** $O(1)$ because we only use a few tracking variables.


---

## Q26 Sliding Window Max

### 1. Problem Explanation

Given an array of integers `arr` of size `N` and a sliding window of size `K`, which moves from the very left of the array to the very right. You can only see the `K` numbers in the window at any one time. Each time the sliding window moves right by one position, return the maximum of the elements inside the window.

### Input
- An integer array `arr` of size `N`.
- An integer `K` ($1 \le K \le N$).

### Output
- An array of size `N - K + 1` containing the maximum element for each sliding window of size `K`.

### Constraints
- $1 \le N \le 10^5$
- $-10^4 \le arr[i] \le 10^4$
- $1 \le K \le N$

### Observations
1. For any window starting at index `i`, it ends at index `i + K - 1`. The total number of windows is `N - K + 1`.
2. As the window slides, one element leaves from the left and one element enters from the right.
3. If we can dynamically maintain the maximum of the current window in $O(1)$ or $O(\log K)$ time, we can beat the brute force $O(N \cdot K)$ approach.


### 2. Brute Solution

### Algorithm
1. Initialize a vector `result` of size `N - K + 1`.
2. Loop `i` from `0` to `N - K`:
   - Initialize `max_val = arr[i]`.
   - Loop `j` from `i` to `i + K - 1`:
     - Update `max_val = max(max_val, arr[j])`.
   - Store `max_val` in `result[i]`.
3. Return `result`.

### Complexity Analysis
- **Time Complexity:** $O(N \cdot K)$ in the worst case (e.g., when $K \approx N/2$).
- **Space Complexity:** $O(1)$ auxiliary space (excluding the output array).

### Brute Force C++17 Code
```cpp
#include <vector>
#include <algorithm>
#include <climits>

class Solution {
public:
    std::vector<int> maxSlidingWindowBrute(const std::vector<int>& arr, int k) {
        int n = arr.size();
        if (n == 0 || k == 0) return {};
        
        std::vector<int> result;
        for (int i = 0; i <= n - k; ++i) {
            int max_val = INT_MIN;
            for (int j = i; j < i + k; ++j) {
                max_val = std::max(max_val, arr[j]);
            }
            result.push_back(max_val);
        }
        return result;
    }
};
```


### 3. Optimal Solution & Complexities

### Algorithm
1. Initialize a `std::deque<int> dq` which will store indices of elements.
2. Initialize `result` vector.
3. Loop through the array from `i = 0` to `N - 1`:
   - **Remove Out-of-Window Elements:** If the index at the front of the deque is out of the window boundary (`dq.front() <= i - k`), pop it.
   - **Maintain Monotonic Property:** While the deque is not empty and the element at the back is less than or equal to `arr[i]`, pop from the back.
   - **Push Current Element:** Push the current index `i` onto the back of the deque.
   - **Add to Result:** If we have processed at least `K` elements (i.e., `i >= K - 1`), the element at the front of the deque is the maximum of the current window. Add `arr[dq.front()]` to the result.
4. Return `result`.

### Why this is Optimal
Each index is pushed into the deque exactly once and popped from the deque at most once. Thus, the total number of operations on the deque is at most $2N$. This results in a time complexity of $O(N)$, which is optimal.

### C++17 Optimal Code
```cpp
#include <vector>
#include <deque>

class Solution {
public:
    std::vector<int> maxSlidingWindow(const std::vector<int>& arr, int k) {
        int n = arr.size();
        if (n == 0 || k == 0) return {};
        
        std::vector<int> result;
        // dq will store indices of elements in the array
        std::deque<int> dq;
        
        for (int i = 0; i < n; ++i) {
            // 1. Remove elements that are out of the current sliding window
            if (!dq.empty() && dq.front() == i - k) {
                dq.pop_front();
            }
            
            // 2. Remove elements from the back that are smaller than the current element
            // because they cannot be the maximum of the current or future windows
            while (!dq.empty() && arr[dq.back()] <= arr[i]) {
                dq.pop_back();
            }
            
            // 3. Add the current element's index to the deque
            dq.push_back(i);
            
            // 4. The first window ends at index k - 1
            if (i >= k - 1) {
                result.push_back(arr[dq.front()]);
            }
        }
        
        return result;
    }
};
```


#### Complexity Summary

- **Time Complexity:** 
  - **Best Case:** $O(N)$ (e.g., strictly decreasing array, where we do not do any back popping).
  - **Average Case:** $O(N)$.
  - **Worst Case:** $O(N)$ (e.g., strictly increasing array, where we pop all previous elements, but each element is still pushed and popped at most once).
- **Space Complexity:**
  - **Auxiliary Space:** $O(K)$ to store the indices in the deque.


---

## Q27 Trapping Rain Water

### 1. Problem Explanation

Given `N` non-negative integers representing an elevation map where the width of each bar is `1`, compute how much water it can trap after raining.

### Input
- An integer array `height` of size `N`.

### Output
- An integer representing the total volume of trapped rainwater.

### Constraints
- $1 \le N \le 10^5$
- $0 \le height[i] \le 10^4$

### Observations
1. Water cannot be trapped at the first bar (index `0`) and the last bar (index `N-1`) because there is no boundary on one side to contain the water.
2. The water trapped on top of any bar `i` depends on the height of the tallest bar to its left and the height of the tallest bar to its right.
3. Specifically, for any bar `i`:
   - Let `L_max` be the maximum height in `height[0...i]`.
   - Let `R_max` be the maximum height in `height[i...N-1]`.
   - The water level above bar `i` is determined by `min(L_max, R_max)`.
   - The water trapped at bar `i` is `min(L_max, R_max) - height[i]`.

```
       Water Level = min(L_max, R_max)
       ~~~~~~~|~~~~~~~
       |      |      |
       |L_max |      | R_max
       |      |bar i |
_______|______|_height|_______
```


### 2. Brute Solution

### Algorithm
1. Initialize `total_water = 0`.
2. Loop `i` from `0` to `N-1`:
   - Initialize `left_max = 0`, `right_max = 0`.
   - Loop `j` from `i` down to `0`: `left_max = max(left_max, height[j])`.
   - Loop `j` from `i` to `N-1`: `right_max = max(right_max, height[j])`.
   - Add `min(left_max, right_max) - height[i]` to `total_water`.
3. Return `total_water`.

### Complexity Analysis
- **Time Complexity:** $O(N^2)$ because of nested scans for each index.
- **Space Complexity:** $O(1)$ auxiliary space.

### Brute Force C++17 Code
```cpp
#include <vector>
#include <algorithm>

class Solution {
public:
    int trapBrute(const std::vector<int>& height) {
        int n = height.size();
        int total_water = 0;
        
        for (int i = 0; i < n; ++i) {
            int left_max = 0;
            int right_max = 0;
            
            // Find the maximum element to the left of i
            for (int j = i; j >= 0; --j) {
                left_max = std::max(left_max, height[j]);
            }
            
            // Find the maximum element to the right of i
            for (int j = i; j < n; ++j) {
                right_max = std::max(right_max, height[j]);
            }
            
            // Water trapped at bar i
            total_water += std::min(left_max, right_max) - height[i];
        }
        return total_water;
    }
};
```


### 3. Optimal Solution & Complexities

### Algorithm
1. Initialize pointers `left = 0`, `right = N - 1`.
2. Initialize variables `left_max = 0`, `right_max = 0`, and `total_water = 0`.
3. While `left < right`:
   - If `height[left] <= height[right]`:
     - If `height[left] >= left_max`, update `left_max = height[left]`.
     - Else, add `left_max - height[left]` to `total_water`.
     - Increment `left`.
   - Else:
     - If `height[right] >= right_max`, update `right_max = height[right]`.
     - Else, add `right_max - height[right]` to `total_water`.
     - Decrement `right`.
4. Return `total_water`.

### Why this is Optimal
It processes the array in a single pass ($O(N)$ time) and requires no extra arrays ($O(1)$ space). It operates on the principle that the water height at any bar is restricted by the shorter of its two bounding walls. By comparing `height[left]` and `height[right]`, we always proceed from the side that is guaranteed to be bounded by the other.

### C++17 Optimal Code
```cpp
#include <vector>
#include <algorithm>

class Solution {
public:
    int trap(const std::vector<int>& height) {
        int n = height.size();
        if (n < 3) return 0;
        
        int left = 0;
        int right = n - 1;
        
        int left_max = 0;
        int right_max = 0;
        
        int total_water = 0;
        
        while (left < right) {
            // The water level is limited by the shorter height
            if (height[left] <= height[right]) {
                if (height[left] >= left_max) {
                    // Update the left boundary
                    left_max = height[left];
                } else {
                    // Trap water above the left bar
                    total_water += left_max - height[left];
                }
                ++left;
            } else {
                if (height[right] >= right_max) {
                    // Update the right boundary
                    right_max = height[right];
                } else {
                    // Trap water above the right bar
                    total_water += right_max - height[right];
                }
                --right;
            }
        }
        
        return total_water;
    }
};
```


#### Complexity Summary

- **Time Complexity:** 
  - **Best Case:** $O(N)$ (no matter what, the two pointers meet at the center, visiting each element once).
  - **Average Case:** $O(N)$.
  - **Worst Case:** $O(N)$.
- **Space Complexity:**
  - **Auxiliary Space:** $O(1)$ since only a few integer variables are kept in memory.


---

## Q28 Pairs Difference K

### 1. Problem Explanation

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


### 2. Brute Solution

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


### 3. Optimal Solution & Complexities

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


#### Complexity Summary

- **Time Complexity:** 
  - **Best Case:** $O(N)$ (populating map takes $O(N)$, iteration takes $O(U)$ where $U \le N$ is the number of unique elements).
  - **Average Case:** $O(N)$.
  - **Worst Case:** $O(N)$ with standard hash maps. (If there are collision issues, it could theoretically degrade to $O(N^2)$, but this can be avoided by using custom hashers or `std::map` which guarantees $O(N \log N)$ worst-case).
- **Space Complexity:**
  - **Auxiliary Space:** $O(N)$ to store the frequency map of size up to $N$.


---

## Q29 Juggling Rotation

### 1. Problem Explanation

Given an array of integers `arr` of size `N`, rotate the array to the left by `d` positions in-place.

### Input
- An integer array `arr` of size `N`.
- An integer `d` ($0 \le d \le 10^5$).

### Output
- The array `arr` rotated left by `d` positions in-place.

### Constraints
- $1 \le N \le 10^5$
- $0 \le d \le 10^5$
- $-10^9 \le arr[i] \le 10^9$

### Observations
1. Rotating left by `d` positions means moving the first `d` elements to the back.
2. Rotating by `d` is equivalent to rotating by `d % N`. If `d` is a multiple of `N`, the array remains unchanged.
3. The rotation must be done in-place with $O(1)$ auxiliary space.


### 2. Brute Solution

### Algorithm
1. Set `d = d % N`.
2. Loop `d` times:
   - Save `temp = arr[0]`.
   - Shift elements left: loop `j` from `0` to `N-2`: `arr[j] = arr[j+1]`.
   - Place `arr[N-1] = temp`.

### Complexity Analysis
- **Time Complexity:** $O(N \cdot d)$ (if $d \approx N$, it becomes $O(N^2)$).
- **Space Complexity:** $O(1)$ auxiliary space.

### Brute Force C++17 Code
```cpp
#include <vector>

class Solution {
public:
    void rotateBrute(std::vector<int>& arr, int d) {
        int n = arr.size();
        if (n == 0) return;
        d = d % n;
        if (d == 0) return;
        
        for (int i = 0; i < d; ++i) {
            int temp = arr[0];
            for (int j = 0; j < n - 1; ++j) {
                arr[j] = arr[j + 1];
            }
            arr[n - 1] = temp;
        }
    }
};
```


### 3. Optimal Solution & Complexities

### Algorithm
1. Set `d = d % N`.
2. Compute `g = gcd(N, d)`.
3. Loop `i` from `0` to `g - 1` (for each cycle):
   - Store `temp = arr[i]`.
   - Set `j = i`.
   - Inside an infinite loop:
     - Compute the next index: `k = j + d`.
     - If `k >= N`, wrap it around: `k = k - N`.
     - If `k == i`, we have completed the cycle. Break the loop.
     - Shift the element: `arr[j] = arr[k]`.
     - Move the pointer: `j = k`.
   - Place the stored element back into the final vacancy: `arr[j] = temp`.

### Why this is Optimal
It achieves $O(N)$ time complexity and $O(1)$ space complexity. More importantly, it writes to each array slot exactly once. In hardware architectures where memory write operations are slower or consume more energy (such as flash memory), this algorithm is superior to the Reversal Algorithm because it performs half the write operations.

### C++17 Optimal Code
```cpp
#include <vector>
#include <numeric>

class Solution {
private:
    // Helper function to find greatest common divisor
    int getGCD(int a, int b) {
        if (b == 0) return a;
        return getGCD(b, a % b);
    }
public:
    void rotateJuggling(std::vector<int>& arr, int d) {
        int n = arr.size();
        if (n == 0) return;
        
        d = d % n;
        if (d == 0) return;
        
        // Find GCD of N and d to determine the number of cycles
        int gcd_val = getGCD(n, d);
        
        for (int i = 0; i < gcd_val; ++i) {
            // Save the starting element of the current cycle
            int temp = arr[i];
            int j = i;
            
            while (true) {
                int k = j + d;
                if (k >= n) {
                    k = k - n; // Wrap around index
                }
                
                // If we are back at the start, the cycle is complete
                if (k == i) {
                    break;
                }
                
                // Move elements
                arr[j] = arr[k];
                j = k;
            }
            // Put the saved element in its final position
            arr[j] = temp;
        }
    }
};
```


#### Complexity Summary

- **Time Complexity:** 
  - **Best/Average/Worst Case:** $O(N)$ because each element is visited and shifted exactly once.
- **Space Complexity:**
  - **Auxiliary Space:** $O(1)$ since only a few temporary integers are used.


---

## Q30 Majority Element

### 1. Problem Explanation

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


### 2. Brute Solution

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


### 3. Optimal Solution & Complexities

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


#### Complexity Summary

- **Time Complexity:** 
  - **Best/Average/Worst Case:** $O(N)$ since we perform at most two sequential passes over the array.
- **Space Complexity:**
  - **Auxiliary Space:** $O(1)$ because we only use a few tracking variables.


---

## Q31 Appears Once Twice

### 1. Problem Explanation

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


### 2. Brute Solution

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


### 3. Optimal Solution & Complexities

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


#### Complexity Summary

| Approach | Time Complexity | Space Complexity | Notes |
| :--- | :--- | :--- | :--- |
| Brute Force | $O(n^2)$ | $O(1)$ | Double loop check |
| Sorting | $O(n \log n)$ | $O(1)$ or $O(n)$ | Modifies input or needs copy |
| Hash Map | $O(n)$ | $O(n)$ | Extra space proportional to unique elements |
| **XOR Optimal** | **$O(n)$** | **$O(1)$** | **Optimal, linear time, constant space** |


---

## Q32 Appears Once Thrice

### 1. Problem Explanation

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


### 2. Brute Solution

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


### 3. Optimal Solution & Complexities

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


#### Complexity Summary

| Approach | Time Complexity | Space Complexity | Notes |
| :--- | :--- | :--- | :--- |
| Brute Force | $O(n^2)$ | $O(1)$ | Double loop check |
| Hash Map | $O(n)$ | $O(n)$ | Stores all elements in map |
| **Bit Counting** | **$O(32 \cdot n) = O(n)$** | **$O(1)$** | **Easy to implement in interviews** |
| **State Machine** | **$O(n)$** | **$O(1)$** | **Fastest, single pass, no arrays** |


---

## Q33 Product Except Self

### 1. Problem Explanation

Given an integer array `nums`, return an array `answer` such that `answer[i]` is equal to the product of all the elements of `nums` except `nums[i]`.

The product of any prefix or suffix of `nums` is guaranteed to fit in a 32-bit integer.

* You must write an algorithm that runs in $O(n)$ time and without using the division operator.
* Could you solve it with $O(1)$ extra space complexity? (The output array does not count as extra space for space complexity analysis.)

### Input
* `nums`: A vector of integers where $2 \le \text{nums.length} \le 10^5$.

### Output
* Returns a vector of integers of length `nums.length`.

### Constraints
* $-30 \le \text{nums}[i] \le 30$
* The product of any prefix or suffix of `nums` is guaranteed to fit in a 32-bit integer.


### 2. Brute Solution

### Algorithm
1. Initialize `answer` vector of size `n` with all 1s.
2. For each index `i` from `0` to `n-1`:
   * Set `prod = 1`.
   * For each index `j` from `0` to `n-1`:
     * If `i != j`, set `prod = prod * nums[j]`.
   * Store `prod` in `answer[i]`.
3. Return `answer`.

### Complexity Analysis
* **Time Complexity**: $O(n^2)$
* **Space Complexity**: $O(1)$ auxiliary space (excluding the output array).

### C++17 Code
```cpp
#include <vector>

class SolutionBrute {
public:
    std::vector<int> productExceptSelf(const std::vector<int>& nums) {
        int n = nums.size();
        std::vector<int> answer(n, 1);
        
        for (int i = 0; i < n; ++i) {
            int product = 1;
            for (int j = 0; j < n; ++j) {
                if (i != j) {
                    product *= nums[j];
                }
            }
            answer[i] = product;
        }
        
        return answer;
    }
};
```


### 3. Optimal Solution & Complexities

### Algorithm
1. Initialize an `answer` vector of size `n` with `answer[0] = 1`.
2. First pass (Left to Right): For each index `i` from `1` to `n-1`, set `answer[i] = answer[i-1] * nums[i-1]`.
   * Now `answer[i]` holds the product of all elements to the left of `i`.
3. Second pass (Right to Left):
   * Initialize a variable `suffix_product = 1`.
   * For each index `i` from `n-1` down to `0`:
     * Multiply `answer[i]` by `suffix_product`.
     * Update `suffix_product` by multiplying it with `nums[i]`.
4. Return `answer`.

### Why this is Optimal
* It uses only one additional scalar variable (`suffix_product`).
* It modifies the output array directly, which is allowed under the $O(1)$ extra space constraint.
* The time complexity remains $O(n)$ with only two passes.

### C++17 Code
```cpp
#include <vector>

class SolutionOptimal {
public:
    std::vector<int> productExceptSelf(const std::vector<int>& nums) {
        int n = nums.size();
        std::vector<int> answer(n);
        
        // Step 1: Accumulate prefix products in the answer array
        answer[0] = 1;
        for (int i = 1; i < n; ++i) {
            answer[i] = answer[i - 1] * nums[i - 1];
        }
        
        // Step 2: Accumulate suffix products on the fly using a single variable
        int suffix_product = 1;
        for (int i = n - 1; i >= 0; --i) {
            answer[i] *= suffix_product;
            suffix_product *= nums[i];
        }
        
        return answer;
    }
};
```


#### Complexity Summary

| Approach | Time Complexity | Space Complexity | Notes |
| :--- | :--- | :--- | :--- |
| Brute Force | $O(n^2)$ | $O(1)$ auxiliary | Nested loops |
| Prefix & Suffix Arrays | $O(n)$ | $O(n)$ auxiliary | Uses two extra arrays |
| **Optimal space** | **$O(n)$** | **$O(1)$ auxiliary** | **Uses output array to store prefix** |


---

## Q34 Dutch National Flag

### 1. Problem Explanation

Given an array `nums` with `n` objects colored red, white, or blue, sort them **in-place** so that objects of the same color are adjacent, with the colors in the order red, white, and blue.

We will use the integers `0`, `1`, and `2` to represent the color red, white, and blue, respectively.

You must solve this problem without using the library's sort function.

### Input
* `nums`: A vector of integers containing only `0`, `1`, and `2` where $1 \le \text{nums.length} \le 300$.

### Output
* The array is sorted in-place. No value is returned.

### Constraints
* `nums[i]` is either `0`, `1`, or `2`.
* Could you come up with a one-pass algorithm using only constant extra space?


### 2. Brute Solution

### Algorithm (Standard Sort)
* Call `std::sort` on the array.

### Complexity Analysis
* **Time Complexity**: $O(n \log n)$
* **Space Complexity**: $O(1)$ or $O(\log n)$ depending on the implementation of `std::sort`.

### C++17 Code
```cpp
#include <vector>
#include <algorithm>

class SolutionBrute {
public:
    void sortColors(std::vector<int>& nums) {
        std::sort(nums.begin(), nums.end());
    }
};
```


### 3. Optimal Solution & Complexities

### Algorithm (Dutch National Flag)
1. Initialize `low = 0`, `mid = 0`, `high = nums.size() - 1`.
2. While `mid <= high`:
   * If `nums[mid] == 0`:
     * `std::swap(nums[low], nums[mid])`
     * `low++`, `mid++`
   * Else if `nums[mid] == 1`:
     * `mid++`
   * Else (if `nums[mid] == 2`):
     * `std::swap(nums[mid], nums[high])`
     * `high--`

### Why this is Optimal
* It makes a single pass through the array.
* Only swaps elements when necessary.
* Does not require extra memory space beyond three index pointers.

### C++17 Code
```cpp
#include <vector>
#include <utility>

class SolutionOptimal {
public:
    void sortColors(std::vector<int>& nums) {
        int low = 0;
        int mid = 0;
        int high = static_cast<int>(nums.size()) - 1;
        
        while (mid <= high) {
            if (nums[mid] == 0) {
                std::swap(nums[low], nums[mid]);
                low++;
                mid++;
            } 
            else if (nums[mid] == 1) {
                mid++;
            } 
            else { // nums[mid] == 2
                std::swap(nums[mid], nums[high]);
                high--;
            }
        }
    }
};
```


#### Complexity Summary

| Approach | Time Complexity | Space Complexity | Notes |
| :--- | :--- | :--- | :--- |
| Brute Force (std::sort) | $O(n \log n)$ | $O(1)$ or $O(\log n)$ | General sorting |
| Counting Sort | $O(n)$ | $O(1)$ | Two passes over the array |
| **Dutch National Flag** | **$O(n)$** | **$O(1)$** | **Single pass, in-place swaps** |


---

## Q35 Kth Smallest Largest

### 1. Problem Explanation

Given an integer array `nums` and an integer `k`, return the $k$-th largest element in the array.

Note that it is the $k$-th largest element in the sorted order, not the $k$-th distinct element.

*(Note: Finding the $k$-th smallest element is the symmetric problem: the $k$-th smallest element is equivalent to the $(n - k + 1)$-th largest element.)*

### Input
* `nums`: A vector of integers where $1 \le k \le \text{nums.length} \le 10^5$.
* `k`: An integer representing the target order.

### Output
* Returns the $k$-th largest integer.

### Constraints
* $-10^4 \le \text{nums}[i] \le 10^4$


### 2. Brute Solution

### Algorithm
1. Sort the vector `nums` using `std::sort`.
2. Return `nums[nums.size() - k]`.

### Complexity Analysis
* **Time Complexity**: $O(n \log n)$
* **Space Complexity**: $O(1)$ auxiliary space (if sorting in-place).

### C++17 Code
```cpp
#include <vector>
#include <algorithm>

class SolutionBrute {
public:
    int findKthLargest(std::vector<int>& nums, int k) {
        std::sort(nums.begin(), nums.end());
        return nums[nums.size() - k];
    }
};
```


### 3. Optimal Solution & Complexities

### Algorithm (QuickSelect with Random Pivot)
1. We want to find the element at index `target = n - k` in the sorted array (which is the $k$-th largest).
2. Define a function `quickSelect(left, right, target)`:
   * Choose a random pivot index between `left` and `right`. Swap it with `nums[right]`.
   * Partition the array from `left` to `right` using Lomuto's partition scheme:
     * Iterate `i` from `left` to `right - 1`. If `nums[i] <= pivot`, swap `nums[i]` with `nums[store_index]` and increment `store_index`.
     * Swap `nums[right]` with `nums[store_index]`.
   * Now the pivot is at index `store_index`.
   * If `store_index == target`, return `nums[store_index]`.
   * If `store_index > target`, recursively call `quickSelect(left, store_index - 1, target)`.
   * If `store_index < target`, recursively call `quickSelect(store_index + 1, right, target)`.

### Why this is Optimal
* On average, we do not partition both sides of the array (unlike QuickSort). This reduces the recurrence relation to $T(n) = T(n/2) + O(n)$, which solves to $O(n)$ by Master's Theorem.
* Space complexity is $O(1)$ auxiliary (ignoring recursion stack space which is $O(\log n)$ on average).

### C++17 Code
```cpp
#include <vector>
#include <algorithm>
#include <cstdlib>
#include <ctime>

class SolutionOptimal {
private:
    int partition(std::vector<int>& nums, int left, int right) {
        // Choose a random pivot to avoid O(n^2) worst case on sorted inputs
        int pivot_index = left + std::rand() % (right - left + 1);
        std::swap(nums[pivot_index], nums[right]);
        
        int pivot = nums[right];
        int store_index = left;
        
        for (int i = left; i < right; ++i) {
            if (nums[i] < pivot) {
                std::swap(nums[i], nums[store_index]);
                store_index++;
            }
        }
        std::swap(nums[store_index], nums[right]);
        return store_index;
    }

    int quickSelect(std::vector<int>& nums, int left, int right, int target) {
        if (left == right) {
            return nums[left];
        }
        
        int pivot_index = partition(nums, left, right);
        
        if (pivot_index == target) {
            return nums[pivot_index];
        } 
        else if (pivot_index < target) {
            return quickSelect(nums, pivot_index + 1, right, target);
        } 
        else {
            return quickSelect(nums, left, pivot_index - 1, target);
        }
    }

public:
    int findKthLargest(std::vector<int>& nums, int k) {
        std::srand(std::time(nullptr)); // Seed random number generator
        int n = nums.size();
        return quickSelect(nums, 0, n - 1, n - k);
    }
};
```


#### Complexity Summary

| Approach | Time Complexity (Best/Avg) | Time (Worst Case) | Space Complexity | Notes |
| :--- | :--- | :--- | :--- | :--- |
| Brute Force (Sort) | $O(n \log n)$ | $O(n \log n)$ | $O(1)$ | Simple, modifies array |
| Heap | $O(n \log k)$ | $O(n \log k)$ | $O(k)$ | Good for stream inputs |
| **QuickSelect** | **$O(n)$** | **$O(n^2)$** | **$O(1)$ auxiliary** | **Linear time on average** |

*Note: Worst-case $O(n^2)$ time in QuickSelect happens when the pivot chosen is always the minimum or maximum element. Randomizing the pivot makes this case highly improbable ($O(2^{-n})$ probability).*


---

## Q36 Minimum Platforms

### 1. Problem Explanation

Given arrival and departure times of all trains that reach a railway station, find the minimum number of platforms required for the railway station so that no train waits.

We are given two arrays `arr[]` (arrival times) and `dep[]` (departure times) of size `n`.

* Time values are represented in 24-hour format as integers (e.g., `900` for 9:00 AM, `1230` for 12:30 PM).
* If a train arrives and another departs at the same time, the platform cannot be shared immediately; the departing train must leave before the arriving train can use the platform.

### Input
* `arr[]`: An array of arrival times of size `n`.
* `dep[]`: An array of departure times of size `n`.
* `n`: The number of trains ($1 \le n \le 50000$).

### Output
* Returns the minimum number of platforms required.

### Constraints
* `0 <= arr[i] <= dep[i] <= 2359`


### 2. Brute Solution

### Algorithm
1. Initialize `max_platforms = 1`.
2. For each train `i` from `0` to `n-1`:
   * Set `overlap = 1`.
   * For each train `j` from `0` to `n-1`:
     * If `i != j`:
       * If `arr[i] >= arr[j]` and `dep[j] >= arr[i]` (train `j` is at the station when train `i` arrives), increment `overlap`.
   * Update `max_platforms = max(max_platforms, overlap)`.
3. Return `max_platforms`.

### Complexity Analysis
* **Time Complexity**: $O(n^2)$ due to nested comparisons.
* **Space Complexity**: $O(1)$ auxiliary space.

### C++17 Code
```cpp
#include <vector>
#include <algorithm>

class SolutionBrute {
public:
    int findPlatform(const std::vector<int>& arr, const std::vector<int>& dep) {
        int n = arr.size();
        int max_platforms = 0;
        
        for (int i = 0; i < n; ++i) {
            int overlap = 0;
            for (int j = 0; j < n; ++j) {
                // Check if train j overlaps with arrival of train i
                if (arr[j] <= arr[i] && dep[j] >= arr[i]) {
                    overlap++;
                }
            }
            max_platforms = std::max(max_platforms, overlap);
        }
        
        return max_platforms;
    }
};
```


### 3. Optimal Solution & Complexities

### Algorithm
1. Make a copy of `arr` and `dep` (or sort the input arrays directly if modification is permitted).
2. Sort both arrays:
   * `std::sort(arr.begin(), arr.end())`
   * `std::sort(dep.begin(), dep.end())`
3. Initialize pointers: `i = 0` (for arrivals), `j = 0` (for departures).
4. Initialize counters: `platforms_needed = 0`, `max_platforms = 0`.
5. Loop while `i < n` and `j < n`:
   * If `arr[i] <= dep[j]`:
     * A train is arriving before the earliest departure. We need an extra platform.
     * `platforms_needed++`
     * `i++`
   * Else:
     * A train departs, freeing a platform.
     * `platforms_needed--`
     * `j++`
   * Update `max_platforms = max(max_platforms, platforms_needed)`.
6. Return `max_platforms`.

### Why this is Optimal
* Sorting takes $O(n \log n)$ time.
* The two-pointer sweep takes $O(n)$ time.
* No extra data structures are used, so space complexity is $O(1)$ auxiliary.

### C++17 Code
```cpp
#include <vector>
#include <algorithm>

class SolutionOptimal {
public:
    int findPlatform(std::vector<int>& arr, std::vector<int>& dep) {
        int n = arr.size();
        
        // Sort arrival and departure times independently
        std::sort(arr.begin(), arr.end());
        std::sort(dep.begin(), dep.end());
        
        int platforms_needed = 0;
        int max_platforms = 0;
        
        int i = 0; // Pointer for arrivals
        int j = 0; // Pointer for departures
        
        while (i < n && j < n) {
            // If next event is an arrival
            if (arr[i] <= dep[j]) {
                platforms_needed++;
                i++;
            }
            // If next event is a departure
            else {
                platforms_needed--;
                j++;
            }
            // Update the peak number of platforms
            max_platforms = std::max(max_platforms, platforms_needed);
        }
        
        return max_platforms;
    }
};
```


#### Complexity Summary

| Approach | Time Complexity | Space Complexity | Notes |
| :--- | :--- | :--- | :--- |
| Brute Force | $O(n^2)$ | $O(1)$ | Checks all pairs |
| Event Sorting | $O(n \log n)$ | $O(n)$ | Allocates space for structures |
| **Two Pointers (Optimal)** | **$O(n \log n)$** | **$O(1)$ auxiliary** | **Sorts in place, linear sweep** |


---

## Q37 Max Non Adjacent

### 1. Problem Explanation

Given an array of integers (which can contain positive, negative, or zero elements, though the classic version focuses on non-negative integers), find the maximum sum of elements such that no two elements in the sum are adjacent to each other in the original array.

### Input
- An array of integers `nums` of size `n`.

### Output
- A single integer representing the maximum possible sum of non-adjacent elements.

### Constraints
- $1 \le n \le 10^5$
- $0 \le nums[i] \le 10^4$ (For the classic House Robber problem, values are non-negative. If negatives are allowed, we can choose not to pick any negative numbers if the sum becomes smaller, but usually elements are non-negative).

### Observations
1. If we select an element at index `i`, we cannot select elements at index `i-1` and `i+1`.
2. For each element, we make a binary choice: **Include** it in our sum (and skip the adjacent element) or **Exclude** it (and consider the adjacent element).
3. This is a optimization problem with overlapping subproblems, making it a classic candidate for Dynamic Programming.


### 2. Brute Solution

### Algorithm
1. Define a recursive helper function `solve(index, nums)` that returns the maximum sum possible from index `0` up to `index`.
2. If `index < 0`, return 0.
3. If `index == 0`, return `nums[0]`.
4. Calculate two possibilities:
   - `pick = nums[index] + solve(index - 2, nums)`
   - `notPick = 0 + solve(index - 1, nums)`
5. Return the maximum of `pick` and `notPick`.

### Dry Run
Let `nums = [2, 1, 4, 9]`.
- Call `solve(3)`:
  - `pick = 9 + solve(1)`
    - `solve(1) = max(nums[1] + solve(-1), solve(0)) = max(1 + 0, 2) = 2`
    - So `pick = 9 + 2 = 11`
  - `notPick = solve(2)`
    - `solve(2) = max(nums[2] + solve(0), solve(1))`
      - `solve(0) = 2`
      - `solve(1) = 2` (computed above)
      - `solve(2) = max(4 + 2, 2) = 6`
    - So `notPick = 6`
  - `solve(3) = max(11, 6) = 11`

### Complexity Analysis
- **Time Complexity:** $O(2^n)$ because each recursive step branches into two subproblems.
- **Space Complexity:** $O(n)$ due to the recursion stack depth.

### Pros and Cons
- **Pros:** Extremely simple to understand and implement.
- **Cons:** Extremely slow; triggers Time Limit Exceeded (TLE) for $n > 30$.

### C++17 Code
```cpp
#include <vector>
#include <algorithm>
#include <iostream>

class SolutionBrute {
private:
    int solveRecursively(int index, const std::vector<int>& nums) {
        // Base cases
        if (index < 0) {
            return 0;
        }
        if (index == 0) {
            return nums[0];
        }
        
        // Pick the current element, move to index - 2
        int pick = nums[index] + solveRecursively(index - 2, nums);
        
        // Skip the current element, move to index - 1
        int notPick = solveRecursively(index - 1, nums);
        
        return std::max(pick, notPick);
    }

public:
    int getMaxSum(const std::vector<int>& nums) {
        if (nums.empty()) return 0;
        return solveRecursively(nums.size() - 1, nums);
    }
};
```


### 3. Optimal Solution & Complexities

### Algorithm
We optimize the Space Complexity of the Tabulation DP to $O(1)$ by noting that we only ever reference the previous two states: `dp[i-1]` and `dp[i-2]`.
1. Let `prev2` represent the max sum up to `i-2` (initially 0).
2. Let `prev1` represent the max sum up to `i-1` (initially `nums[0]`).
3. Iterate from `i = 1` to `n-1`:
   - `take = nums[i] + (i > 1 ? prev2 : 0)` (if $i=1$, we cannot add `prev2` as there is no element before `nums[0]`).
   - `skip = prev1`
   - `current_max = max(take, skip)`
   - Update state: `prev2 = prev1`, `prev1 = current_max`.
4. Return `prev1`.

### C++17 Code
```cpp
#include <vector>
#include <algorithm>

class SolutionOptimal {
public:
    int getMaxSum(const std::vector<int>& nums) {
        int n = nums.size();
        if (n == 0) return 0;
        
        // prev1 represents dp[i-1], initialized to the first element
        int prev1 = nums[0];
        // prev2 represents dp[i-2], initialized to 0
        int prev2 = 0;
        
        for (int i = 1; i < n; ++i) {
            // Decide whether to pick the current element or skip it
            int pick = nums[i];
            if (i > 1) {
                pick += prev2; // Add max sum from 2 steps back
            }
            int notPick = prev1; // Carry forward max sum from 1 step back
            
            int current = std::max(pick, notPick);
            
            // Move our pointers forward
            prev2 = prev1;
            prev1 = current;
        }
        
        return prev1;
    }
};
```


#### Complexity Summary

| Approach | Time Complexity | Space Complexity | Notes |
| :--- | :--- | :--- | :--- |
| **Brute Force** | $O(2^n)$ | $O(n)$ | Call stack depth is $O(n)$ |
| **Better (Tabulation DP)** | $O(n)$ | $O(n)$ | Linear scan, uses $O(n)$ array |
| **Optimal (Variables DP)** | $O(n)$ | $O(1)$ | No extra array or recursion |


---

## Q38 Chocolate Distribution

### 1. Problem Explanation

Given an array `arr` of $n$ integers, where each value represents the number of chocolates in a packet. There are $m$ students. The task is to distribute chocolate packets such that:
1. Each student gets exactly one packet.
2. The difference between the maximum number of chocolates given to a student and the minimum number of chocolates given to a student is minimized.

### Input
- An array of integers `arr` of size `n` representing packet sizes.
- An integer `m` representing the number of students.

### Output
- An integer representing the minimum possible difference between the maximum and minimum chocolate packets distributed.

### Constraints
- $1 \le n \le 10^5$
- $1 \le arr[i] \le 10^9$
- $1 \le m \le n$

### Observations
1. If we choose a subset of size $m$ from the array, the difference between the max and min of this subset needs to be minimized.
2. If the packets are sorted in ascending order, any contiguous subarray of size $m$ represents a selection of packets where the minimum is at the start of the subarray, and the maximum is at the end of the subarray.


### 2. Brute Solution

### Algorithm
1. Generate all possible subsets of size $m$ using recursion (combinations).
2. For each subset, find the maximum element and the minimum element.
3. Compute the difference `max - min`.
4. Keep track of the minimum difference found across all subsets.
5. Return this minimum difference.

### Dry Run
Let `arr = [7, 3, 2, 4]`, `m = 3`.
- Subsets of size 3:
  - `{7, 3, 2}`: min = 2, max = 7. Difference = 5.
  - `{7, 3, 4}`: min = 3, max = 7. Difference = 4.
  - `{7, 2, 4}`: min = 2, max = 7. Difference = 5.
  - `{3, 2, 4}`: min = 2, max = 4. Difference = 2.
- Minimum difference is `2`.

### Complexity Analysis
- **Time Complexity:** $O(\binom{n}{m} \cdot m)$ because we generate all subsets of size $m$, and for each, we find the min and max in $O(m)$ time.
- **Space Complexity:** $O(m)$ for recursion stack.

### Pros and Cons
- **Pros:** Simple recursive logic; correct results for very small datasets.
- **Cons:** Factorial/Exponential time complexity; will crash or time out for $n > 20$.

### C++17 Code
```cpp
#include <vector>
#include <algorithm>
#include <climits>
#include <iostream>

class SolutionBrute {
private:
    void getMinDiffSubsets(const std::vector<int>& arr, int m, int index, 
                           std::vector<int>& currentSubset, int& minDiff) {
        if (currentSubset.size() == static_cast<size_t>(m)) {
            int currentMin = *std::min_element(currentSubset.begin(), currentSubset.end());
            int currentMax = *std::max_element(currentSubset.begin(), currentSubset.end());
            minDiff = std::min(minDiff, currentMax - currentMin);
            return;
        }
        
        if (index == static_cast<int>(arr.size())) {
            return;
        }
        
        // Option 1: Include current element
        currentSubset.push_back(arr[index]);
        getMinDiffSubsets(arr, m, index + 1, currentSubset, minDiff);
        currentSubset.pop_back(); // Backtrack
        
        // Option 2: Exclude current element
        getMinDiffSubsets(arr, m, index + 1, currentSubset, minDiff);
    }

public:
    int findMinDiff(const std::vector<int>& arr, int m) {
        int n = arr.size();
        if (m == 0 || n == 0 || m > n) return -1;
        
        int minDiff = INT_MAX;
        std::vector<int> currentSubset;
        getMinDiffSubsets(arr, m, 0, currentSubset, minDiff);
        return minDiff;
    }
};
```


### 3. Optimal Solution & Complexities

### Algorithm
1. If $m > n$ or $n = 0$, return $-1$.
2. Sort the chocolate packets array `arr` in ascending order.
3. Initialize `minDiff` to a very large value (`INT_MAX`).
4. Slide a window of size `m` from index `0` to `n - m`:
   - Let `i` be the start of the window.
   - The end of the window is at index `i + m - 1`.
   - Calculate the difference: `diff = arr[i + m - 1] - arr[i]`.
   - Update `minDiff = min(minDiff, diff)`.
5. Return `minDiff`.

### C++17 Code
```cpp
#include <vector>
#include <algorithm>
#include <climits>

class SolutionOptimal {
public:
    int findMinDiff(std::vector<int>& arr, int m) {
        int n = arr.size();
        
        // Edge Case: If there are fewer packets than students, distribution is impossible
        if (n < m || n == 0 || m == 0) {
            return -1; 
        }
        
        // Step 1: Sort the packets in ascending order
        std::sort(arr.begin(), arr.end());
        
        int minDifference = INT_MAX;
        
        // Step 2: Slide a window of size m
        for (int i = 0; i <= n - m; ++i) {
            // The maximum elements in this window is at arr[i + m - 1]
            // The minimum elements in this window is at arr[i]
            int currentDifference = arr[i + m - 1] - arr[i];
            
            // Track the minimum difference
            if (currentDifference < minDifference) {
                minDifference = currentDifference;
            }
        }
        
        return minDifference;
    }
};
```


#### Complexity Summary

| Metric | Complexity | Reasoning |
| :--- | :--- | :--- |
| **Time Complexity** | $O(n \log n)$ | Sorting the array takes $O(n \log n)$ time. The subsequent sliding window loop runs $(n - m + 1)$ times, each taking $O(1)$ time, yielding $O(n)$ time. Thus, the sorting step dominates. |
| **Space Complexity** | $O(1)$ or $O(n)$ | Depending on the sorting algorithm implementation. C++ `std::sort` generally uses $O(\log n)$ auxiliary stack space. If we sort in-place, the auxiliary space is $O(1)$. |


---

## Q39 Min Jumps

### 1. Problem Explanation

Given an array of non-negative integers `arr` of size `n`, where each element represents the maximum number of steps that can be made forward from that element. Write a program to return the minimum number of jumps to reach the last index of the array (starting from the first index). If an element is 0, then you cannot move through that element. If it is impossible to reach the end, return -1.

### Input
- An array of integers `arr` of size `n`.

### Output
- An integer representing the minimum number of jumps to reach index `n - 1`. If the end is unreachable, return `-1`.

### Constraints
- $1 \le n \le 10^6$
- $0 \le arr[i] \le 10^5$

### Observations
1. At any index `i`, we can jump to any index `j` in the range `i + 1` to `i + arr[i]`.
2. This is a optimization problem of finding the shortest path in a directed graph where edges exist from `i` to all indices reachable from `i`.
3. Since we want to find the *minimum* number of jumps, it fits standard shortest-path BFS or greedy structures.


### 2. Brute Solution

### Algorithm
1. Define a recursive function `minJumpsRecursion(index, arr)` that returns the min jumps to reach the end from the current `index`.
2. If `index >= n - 1`, return 0 (we are already at or past the end).
3. If `arr[index] == 0`, return `INT_MAX` (we cannot move from here).
4. Loop through all possible jump sizes `j` from 1 to `arr[index]`:
   - Recursively calculate jumps needed from `index + j`.
   - If that path is valid, update the minimum jumps.
5. Return $1 + \min\_jumps\_found$.

### Complexity Analysis
- **Time Complexity:** $O(2^n)$ or $O(n^n)$ in the worst case because we explore every branch.
- **Space Complexity:** $O(n)$ for the recursion stack.

### Pros and Cons
- **Pros:** Simple to think of recursively.
- **Cons:** Triggers TLE even for $n > 20$.

### C++17 Code
```cpp
#include <vector>
#include <algorithm>
#include <climits>

class SolutionBrute {
private:
    int getMinJumps(int index, const std::vector<int>& arr) {
        int n = arr.size();
        if (index >= n - 1) return 0;
        if (arr[index] == 0) return INT_MAX;
        
        int minJumps = INT_MAX;
        for (int j = 1; j <= arr[index]; ++j) {
            int result = getMinJumps(index + j, arr);
            if (result != INT_MAX) {
                minJumps = std::min(minJumps, 1 + result);
            }
        }
        
        return minJumps;
    }

public:
    int minJumps(const std::vector<int>& arr) {
        int result = getMinJumps(0, arr);
        return (result == INT_MAX) ? -1 : result;
    }
};
```


### 3. Optimal Solution & Complexities

### Algorithm
We use a **Greedy Single-pass BFS Simulation**.
1. If $n \le 1$, return `0`.
2. If `arr[0] == 0`, return `-1` (we cannot make even the first step).
3. Initialize:
   - `maxReach = arr[0]`: Farthest index we can reach from all positions visited in the current jump level.
   - `currentEnd = arr[0]`: The end of our current jump level.
   - `jumps = 1`: We must make at least 1 jump from the start.
4. Loop through the array from `i = 1` to `n - 2`:
   - Update `maxReach = max(maxReach, i + arr[i])`.
   - If we reach the end of the current jump level (`i == currentEnd`):
     - Increment `jumps`.
     - Update the jump level boundary: `currentEnd = maxReach`.
     - If `currentEnd <= i`, it means we cannot advance further, so return `-1`.
5. After the loop, return `jumps` if `maxReach >= n - 1` else `-1`.

### C++17 Code
```cpp
#include <vector>
#include <algorithm>

class SolutionOptimal {
public:
    int minJumps(const std::vector<int>& arr) {
        int n = arr.size();
        
        // Base case: If array size is 1, we are already at the destination
        if (n <= 1) {
            return 0;
        }
        
        // If the first element is 0, we can never move anywhere
        if (arr[0] == 0) {
            return -1;
        }
        
        // maxReach stores the maximum index we can reach from any index visited so far
        int maxReach = arr[0];
        
        // currentEnd stores the end of the window of the current jump level
        int currentEnd = arr[0];
        
        // jumps stores the number of jumps taken
        int jumps = 1;
        
        // We only loop up to n - 2 because once we reach or pass n - 2,
        // we can always reach the last element on our next step if maxReach >= n - 1.
        for (int i = 1; i < n - 1; ++i) {
            // Update the furthest we can reach
            maxReach = std::max(maxReach, i + arr[i]);
            
            // If we have reached the end of the current jump level
            if (i == currentEnd) {
                jumps++; // Must jump again
                currentEnd = maxReach; // Update the boundary to the new max reach
                
                // If the new boundary cannot advance beyond the current index, we are stuck
                if (currentEnd <= i) {
                    return -1;
                }
            }
        }
        
        // If we can reach the last index, return the jumps count, else -1
        if (maxReach >= n - 1) {
            return jumps;
        }
        return -1;
    }
};
```


#### Complexity Summary

| Approach | Time Complexity | Space Complexity | Notes |
| :--- | :--- | :--- | :--- |
| **Brute Force** | $O(2^n)$ | $O(n)$ | Call stack depth |
| **Better DP** | $O(n^2)$ | $O(n)$ | $N$ states, each checks up to $N$ jumps |
| **Optimal Greedy** | $O(n)$ | $O(1)$ | Single pass scan, constant extra space |


---

## Q40 Next Permutation

### 1. Problem Explanation

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


### 2. Brute Solution

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


### 3. Optimal Solution & Complexities

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


#### Complexity Summary

| Metric | Complexity | Reasoning |
| :--- | :--- | :--- |
| **Time Complexity** | $O(n)$ | In the worst case, we scan the array at most three times: once to find the pivot index, once to find the swap candidate, and once to reverse the suffix. Each pass is linear. |
| **Space Complexity** | $O(1)$ | No extra memory is allocated. All operations are done in-place. |


---

## Q41 First Last Position

### 1. Problem Explanation

Given an array of integers `nums` sorted in non-decreasing order, find the starting and ending position of a given `target` value.
If `target` is not found in the array, return `[-1, -1]`.

### Input
- A sorted array of integers `nums`.
- An integer `target`.

### Output
- A vector/array of size 2: `[first_position, last_position]`.

### Constraints
- $0 \le nums.size() \le 10^5$
- $-10^9 \le nums[i], target \le 10^9$
- `nums` is sorted in non-decreasing order.

### Observations
1. The array is already sorted, which strongly hints at using Binary Search to find elements in $O(\log n)$ time.
2. The target can appear multiple times. We need to find the boundary occurrences (leftmost and rightmost).


### 2. Brute Solution

### Algorithm
1. Initialize `first = -1` and `last = -1`.
2. Iterate through the array from `i = 0` to `n - 1`:
   - If `nums[i] == target`:
     - If `first == -1`, set `first = i`.
     - Update `last = i`.
3. Return `[first, last]`.

### Complexity Analysis
- **Time Complexity:** $O(n)$ since we scan the entire array of size $n$.
- **Space Complexity:** $O(1)$ auxiliary space.

### Pros and Cons
- **Pros:** Extremely simple to implement.
- **Cons:** Very inefficient for large arrays; doesn't utilize sorting.

### C++17 Code
```cpp
#include <vector>

class SolutionBrute {
public:
    std::vector<int> searchRange(const std::vector<int>& nums, int target) {
        int first = -1;
        int last = -1;
        int n = nums.size();
        
        for (int i = 0; i < n; ++i) {
            if (nums[i] == target) {
                if (first == -1) {
                    first = i;
                }
                last = i;
            }
        }
        
        return {first, last};
    }
};
```


### 3. Optimal Solution & Complexities

### Algorithm
We write a helper binary search function `findBound(nums, target, isFirst)` which returns the index of the boundary.
- If `isFirst` is `true`, search for the first (leftmost) occurrence.
- If `isFirst` is `false`, search for the last (rightmost) occurrence.

Inside `findBound`:
1. Initialize `low = 0`, `high = n - 1`, and `ans = -1`.
2. While `low <= high`:
   - Compute `mid = low + (high - low) / 2`.
   - If `nums[mid] == target`:
     - Set `ans = mid`.
     - If `isFirst` is `true`, search left: set `high = mid - 1`.
     - Else, search right: set `low = mid + 1`.
   - Else if `nums[mid] < target`, set `low = mid + 1`.
   - Else, set `high = mid - 1`.
3. Return `ans`.

Our main function simply returns `{findBound(nums, target, true), findBound(nums, target, false)}`.

### C++17 Code
```cpp
#include <vector>

class SolutionOptimal {
private:
    // Helper function to find the first or last index of the target
    int findBound(const std::vector<int>& nums, int target, bool isFirst) {
        int n = nums.size();
        int low = 0;
        int high = n - 1;
        int resultIndex = -1;
        
        while (low <= high) {
            int mid = low + (high - low) / 2;
            
            if (nums[mid] == target) {
                resultIndex = mid; // Record potential answer
                
                if (isFirst) {
                    // Search left to find an earlier occurrence
                    high = mid - 1;
                } else {
                    // Search right to find a later occurrence
                    low = mid + 1;
                }
            } else if (nums[mid] < target) {
                low = mid + 1; // Target is in the right half
            } else {
                high = mid - 1; // Target is in the left half
            }
        }
        
        return resultIndex;
    }

public:
    std::vector<int> searchRange(const std::vector<int>& nums, int target) {
        int first = findBound(nums, target, true);
        int last = findBound(nums, target, false);
        return {first, last};
    }
};
```


#### Complexity Summary

| Metric | Complexity | Reasoning |
| :--- | :--- | :--- |
| **Time Complexity** | $O(\log n)$ | We run two independent binary searches. Each binary search splits the search space in half at every step, taking $O(\log n)$ time. Total time is $2 \cdot O(\log n) = O(\log n)$. |
| **Space Complexity** | $O(1)$ | No dynamic memory allocation; only a few variables are stored in the stack frame. |


---

## Q42 Count Occurrences

### 1. Problem Explanation

Given a sorted array of integers `arr` of size `n` and a target value `x`, count the number of occurrences of `x` in `arr`.
If `x` is not present in the array, return `0`.

### Input
- A sorted array of integers `arr` of size `n`.
- An integer `target`.

### Output
- An integer representing the count of occurrences of `target` in `arr`.

### Constraints
- $1 \le n \le 10^5$
- $-10^9 \le arr[i], target \le 10^9$
- `arr` is sorted in non-decreasing order.

### Observations
1. The array is already sorted.
2. If `target` is present, all occurrences of `target` will be contiguous in the array.
3. The count of occurrences is given by the difference between the index of the last occurrence and the index of the first occurrence, plus 1: `(lastIndex - firstIndex + 1)`.


### 2. Brute Solution

### Algorithm
1. Initialize `count = 0`.
2. For each element in the array:
   - If the element is equal to the target, increment `count`.
3. Return `count`.

### Complexity Analysis
- **Time Complexity:** $O(n)$ as we visit all elements in the array.
- **Space Complexity:** $O(1)$ auxiliary space.

### Pros and Cons
- **Pros:** Simple, requires no sorted guarantees to work.
- **Cons:** Inefficient; ignores the sorted property of the array.

### C++17 Code
```cpp
#include <vector>

class SolutionBrute {
public:
    int countOccurrences(const std::vector<int>& arr, int target) {
        int count = 0;
        for (int num : arr) {
            if (num == target) {
                count++;
            }
        }
        return count;
    }
};
```


### 3. Optimal Solution & Complexities

### Algorithm
We reuse the two binary search concept to find the exact boundaries:
1. Define a helper function `findBound(arr, target, isFirst)`:
   - Perform binary search. If target matches, save the index, and redirect the search space left (`high = mid - 1`) if `isFirst` is true, or right (`low = mid + 1`) if `isFirst` is false.
2. Find the first occurrence index: `first = findBound(arr, target, true)`.
3. If `first == -1`, the element doesn't exist, so return `0`.
4. Find the last occurrence index: `last = findBound(arr, target, false)`.
5. Return `last - first + 1`.

### C++17 Code
```cpp
#include <vector>

class SolutionOptimal {
private:
    // Helper function to find the first or last occurrence index of the target
    int findBound(const std::vector<int>& arr, int target, bool isFirst) {
        int n = arr.size();
        int low = 0;
        int high = n - 1;
        int resultIndex = -1;
        
        while (low <= high) {
            int mid = low + (high - low) / 2;
            
            if (arr[mid] == target) {
                resultIndex = mid; // Record the index
                
                if (isFirst) {
                    high = mid - 1; // Keep searching in the left half
                } else {
                    low = mid + 1;  // Keep searching in the right half
                }
            } else if (arr[mid] < target) {
                low = mid + 1;
            } else {
                high = mid - 1;
            }
        }
        
        return resultIndex;
    }

public:
    int countOccurrences(const std::vector<int>& arr, int target) {
        int firstIndex = findBound(arr, target, true);
        
        // If the target is not present, count is 0
        if (firstIndex == -1) {
            return 0;
        }
        
        int lastIndex = findBound(arr, target, false);
        
        // Return the count based on indices
        return lastIndex - firstIndex + 1;
    }
};
```


#### Complexity Summary

| Approach | Time Complexity | Space Complexity | Notes |
| :--- | :--- | :--- | :--- |
| **Brute Force** | $O(n)$ | $O(1)$ | Full scan |
| **Better (Expand)** | $O(\log n)$ average / $O(n)$ worst | $O(1)$ | Degrades when all elements are duplicates |
| **Optimal** | $O(\log n)$ | $O(1)$ | Two independent binary searches |


---

## Q43 Wave Array

### 1. Problem Explanation

Given an unsorted array of integers `arr`, rearrange the elements in-place into a **wave-like array** such that:
$$arr[0] \ge arr[1] \le arr[2] \ge arr[3] \le arr[4] \dots$$

In other words, the elements at even indices ($0, 2, 4, \dots$) must be greater than or equal to their immediate neighboring elements, while elements at odd indices ($1, 3, 5, \dots$) must be less than or equal to their neighbors.

### Input
- An integer array `arr` of size $N$ ($1 \le N \le 10^6$).
- Element values: $-10^9 \le arr[i] \le 10^9$.

### Output
- The array rearranged in-place to satisfy the wave property. (If there are multiple valid wave arrays, any is acceptable).

### Constraints
- Time Complexity: Optimal $O(N)$
- Auxiliary Space: $O(1)$ in-place

### Observations
1. In a wave array, elements alternate between "local maxima" (peaks) at even indices and "local minima" (valleys) at odd indices.
2. The problem does not require a sorted wave (like lexicographically smallest). Any wave arrangement is acceptable.
3. Swapping adjacent elements is a key operation to fix local violations of the wave property.


### 2. Brute Solution

### Algorithm
1. Sort the input array in ascending order.
2. Traverse the array with a step of 2 (i.e., $i = 0, 2, 4, \dots$).
3. Swap each element at index $i$ with its adjacent element at $i+1$ (if $i+1 < N$).

### Dry Run
Input: `arr = [3, 6, 5, 10, 7, 20]`
1. Sort the array: `arr = [3, 5, 6, 7, 10, 20]`
2. Step 2 iteration:
   - $i = 0$: Swap `arr[0]` (3) and `arr[1]` (5) $\to$ `[5, 3, 6, 7, 10, 20]`
   - $i = 2$: Swap `arr[2]` (6) and `arr[3]` (7) $\to$ `[5, 3, 7, 6, 10, 20]`
   - $i = 4$: Swap `arr[4]` (10) and `arr[5]` (20) $\to$ `[5, 3, 7, 6, 20, 10]`
3. Output: `[5, 3, 7, 6, 20, 10]`
   - Checking: $5 \ge 3 \le 7 \ge 6 \le 20 \ge 10$. Valid wave!

### Complexity Analysis
- **Time Complexity:** $O(N \log N)$ due to sorting.
- **Space Complexity:** $O(1)$ (or $O(\log N)$ for sorting stack space) if sorted in-place.

### Pros and Cons
- **Pros:** Extremely simple to implement; works correctly even if the interviewer demands the lexicographically smallest wave array (assuming sorted input swap style).
- **Cons:** Sub-optimal time complexity of $O(N \log N)$.

### C++17 Brute Force Code
```cpp
#include <vector>
#include <algorithm>

class WaveArrayBrute {
public:
    void convertToWave(std::vector<int>& arr) {
        int n = arr.size();
        // Step 1: Sort the array
        std::sort(arr.begin(), arr.end());
        
        // Step 2: Swap adjacent elements at even positions
        for (int i = 0; i < n - 1; i += 2) {
            std::swap(arr[i], arr[i + 1]);
        }
    }
};
```


### 3. Optimal Solution & Complexities

### Algorithm
1. Traverse the array at even positions: $i = 0, 2, 4, \dots, N-1$.
2. For each even index $i$:
   - If there is a left neighbor ($i > 0$) and $arr[i] < arr[i-1]$, swap $arr[i]$ and $arr[i-1]$.
   - If there is a right neighbor ($i < N-1$) and $arr[i] < arr[i+1]$, swap $arr[i]$ and $arr[i+1]$.
3. Since we only visit even indices, the step size is 2, ensuring $O(N)$ time.

### Dry Run
Input: `arr = [10, 90, 49, 2, 1, 5, 23]`

| Step | Index `i` | Current Array | Action / Logic |
| :--- | :--- | :--- | :--- |
| Start | - | `[10, 90, 49, 2, 1, 5, 23]` | Initial state |
| 1 | `i = 0` | `[10, 90, 49, 2, 1, 5, 23]` | Check right: `arr[0] (10) < arr[1] (90)`. Swap! |
| | | `[90, 10, 49, 2, 1, 5, 23]` | Array updated. |
| 2 | `i = 2` | `[90, 10, 49, 2, 1, 5, 23]` | Check left: `arr[2] (49) >= arr[1] (10)`. No swap. <br> Check right: `arr[2] (49) >= arr[3] (2)`. No swap. |
| 3 | `i = 4` | `[90, 10, 49, 2, 1, 5, 23]` | Check left: `arr[4] (1) < arr[3] (2)`. Swap! |
| | | `[90, 10, 49, 1, 2, 5, 23]` | Array updated. <br> Check right: `arr[4] (2) < arr[5] (5)`. Swap! |
| | | `[90, 10, 49, 1, 5, 2, 23]` | Array updated. |
| 4 | `i = 6` | `[90, 10, 49, 1, 5, 2, 23]` | Check left: `arr[6] (23) >= arr[5] (2)`. No swap. |

Final Output: `[90, 10, 49, 1, 5, 2, 23]`
Checking relations: $90 \ge 10 \le 49 \ge 1 \le 5 \ge 2 \le 23$. Valid wave!

### Why Optimal
- We visit each element a constant number of times (at most twice: once as a center, and potentially once as a neighbor).
- No additional space is allocated.
- Lower bound for checking an array of size $N$ is $\Omega(N)$ because every element must be inspected at least once. Hence, $O(N)$ is optimal.

### C++17 Optimal Code
```cpp
#include <vector>
#include <utility>

class WaveArrayOptimal {
public:
    void convertToWave(std::vector<int>& arr) {
        int n = arr.size();
        
        // Traverse all even indices
        for (int i = 0; i < n; i += 2) {
            // If current even element is smaller than its left neighbor, swap them
            if (i > 0 && arr[i] < arr[i - 1]) {
                std::swap(arr[i], arr[i - 1]);
            }
            
            // If current even element is smaller than its right neighbor, swap them
            if (i < n - 1 && arr[i] < arr[i + 1]) {
                std::swap(arr[i], arr[i + 1]);
            }
        }
    }
};
```


#### Complexity Summary

| Approach | Time Complexity | Auxiliary Space Complexity | Reasoning |
| :--- | :--- | :--- | :--- |
| **Brute Force (Sort)** | $O(N \log N)$ | $O(1)$ | Sorting the array takes $O(N \log N)$ time, swapping takes $O(N)$. |
| **Optimal (Greedy)** | $O(N)$ | $O(1)$ | Single pass over even indices. Constant number of comparisons/swaps per index. |


---

## Q44 Segregate Even Odd

### 1. Problem Explanation

Given an integer array `arr`, rearrange its elements in-place such that all even numbers are placed on the left side of the array, and all odd numbers are placed on the right side.

We need to cover two versions:
1. **Unstable Segregation:** Order of elements does not matter, in-place $O(N)$ time, $O(1)$ auxiliary space.
2. **Stable Segregation:** The relative order of even elements among themselves and odd elements among themselves must be maintained.

### Input
- An integer array `arr` of size $N$ ($1 \le N \le 10^6$).
- Elements can be positive, negative, or zero.

### Output
- The array modified in-place to contain even numbers first, followed by odd numbers.

### Constraints
- Unstable: Time Complexity $O(N)$, Space Complexity $O(1)$
- Stable: Analyze space-time trade-offs.


### 2. Brute Solution

### Algorithm
1. Create an auxiliary vector `result` of size $N$.
2. Traverse the input array and copy all even elements to `result`.
3. Traverse the input array again and copy all odd elements to `result`.
4. Copy `result` back to the original array.

### Dry Run
Input: `arr = [1, 2, 4, 3, 5, 8]`
- Pass 1 (Even): `result = [2, 4, 8]`
- Pass 2 (Odd): `result = [2, 4, 8, 1, 3, 5]`
- Copy back: `arr = [2, 4, 8, 1, 3, 5]`
- Relative order of evens `(2, 4, 8)` and odds `(1, 3, 5)` is perfectly maintained.

### Complexity Analysis
- **Time Complexity:** $O(N)$ as we make two linear scans.
- **Space Complexity:** $O(N)$ auxiliary space for the `result` vector.

### Pros and Cons
- **Pros:** Preserves relative order (Stable); easy to write.
- **Cons:** Uses $O(N)$ extra memory, which is sub-optimal for constraints requiring $O(1)$ space.

### C++17 Stable Brute Force Code
```cpp
#include <vector>

class SegregateStableBrute {
public:
    void segregateEvenOdd(std::vector<int>& arr) {
        std::vector<int> result;
        result.reserve(arr.size());
        
        // Step 1: Append even numbers
        for (int num : arr) {
            if (num % 2 == 0) {
                result.push_back(num);
            }
        }
        
        // Step 2: Append odd numbers
        for (int num : arr) {
            if (num % 2 != 0) {
                result.push_back(num);
            }
        }
        
        // Step 3: Copy back to original array
        arr = std::move(result);
    }
};
```


### 3. Optimal Solution & Complexities

### Algorithm
1. Initialize two pointers: `left = 0` and `right = N - 1`.
2. While `left < right`:
   - Increment `left` while `arr[left]` is even and `left < right`.
   - Decrement `right` while `arr[right]` is odd and `left < right`.
   - If `left < right`, it means `arr[left]` is odd and `arr[right]` is even. Swap them.
   - Increment `left` and decrement `right` to continue partitioning.

### Dry Run
Input: `arr = [12, 34, 45, 9, 8, 90, 3]`

| Step | `left` | `right` | Array State | Description |
| :--- | :--- | :--- | :--- | :--- |
| Start | 0 | 6 | `[12, 34, 45, 9, 8, 90, 3]` | Initial state |
| 1 | 0 | 6 | `[12, 34, 45, 9, 8, 90, 3]` | `arr[left]=12` is even $\to$ `left++` to 1. |
| 2 | 1 | 6 | `[12, 34, 45, 9, 8, 90, 3]` | `arr[left]=34` is even $\to$ `left++` to 2. |
| 3 | 2 | 6 | `[12, 34, 45, 9, 8, 90, 3]` | `arr[left]=45` is odd. Stop left. |
| 4 | 2 | 6 | `[12, 34, 45, 9, 8, 90, 3]` | `arr[right]=3` is odd $\to$ `right--` to 5. |
| 5 | 2 | 5 | `[12, 34, 45, 9, 8, 90, 3]` | `arr[right]=90` is even. Stop right. |
| 6 | 2 | 5 | `[12, 34, 45, 9, 8, 90, 3]` | `left < right`. Swap `arr[2]` (45) & `arr[5]` (90). |
| | 3 | 4 | `[12, 34, 90, 9, 8, 45, 3]` | Array updated. `left` and `right` updated. |
| 7 | 3 | 4 | `[12, 34, 90, 9, 8, 45, 3]` | `arr[left]=9` is odd. Stop left. |
| 8 | 3 | 4 | `[12, 34, 90, 9, 8, 45, 3]` | `arr[right]=8` is even. Stop right. |
| 9 | 3 | 4 | `[12, 34, 90, 9, 8, 45, 3]` | Swap `arr[3]` (9) & `arr[4]` (8). |
| | 4 | 3 | `[12, 34, 90, 8, 9, 45, 3]` | `left > right`. Loop terminates. |

Final Output: `[12, 34, 90, 8, 9, 45, 3]` (Evens on left, Odds on right).

### Why Optimal
- We scan each element at most once.
- We do not use any extra array or auxiliary memory; partitioning is done fully in-place using standard pointers.

### C++17 Optimal Code
```cpp
#include <vector>
#include <utility>

class SegregateOptimal {
public:
    void segregateEvenOdd(std::vector<int>& arr) {
        int left = 0;
        int right = arr.size() - 1;
        
        while (left < right) {
            // Find the first odd number from left
            while (left < right && arr[left] % 2 == 0) {
                left++;
            }
            
            // Find the first even number from right
            while (left < right && arr[right] % 2 != 0) {
                right--;
            }
            
            // If pointers haven't crossed, swap mismatched elements
            if (left < right) {
                std::swap(arr[left], arr[right]);
                left++;
                right--;
            }
        }
    }
};
```


#### Complexity Summary

| Approach | Time Complexity | Auxiliary Space Complexity | Reasoning |
| :--- | :--- | :--- | :--- |
| **Brute Force (Stable)** | $O(N)$ | $O(N)$ | Uses an auxiliary array to copy evens and odds. |
| **Divide & Conquer (Stable)** | $O(N \log N)$ | $O(\log N)$ | Split in halves, merge using in-place rotations. |
| **Two-Pointer (Optimal Unstable)** | $O(N)$ | $O(1)$ | Single pass with two pointers, elements swapped in-place. |


---

## Q45 Median Two Sorted

### 1. Problem Explanation

Given two sorted arrays `nums1` and `nums2` of size $m$ and $n$ respectively, return the median of the two sorted arrays.

The overall run time complexity should be $O(\log(m+n))$ or more specifically $O(\log(\min(m, n)))$.

### Input
- `nums1`: A sorted integer array of size $m$ ($0 \le m \le 1000$).
- `nums2`: A sorted integer array of size $n$ ($0 \le n \le 1000$).
- Both arrays are sorted in non-decreasing order.
- Total size: $1 \le m + n \le 2000$.

### Output
- A `double` value representing the median of the combined sorted arrays.

### Constraints
- $O(\log(\min(m, n)))$ Time Complexity.
- $O(1)$ Auxiliary Space.

### Observations
1. The median is the middle element if the total size is odd, or the average of the two middle elements if the total size is even.
2. A brute force merge takes $O(m + n)$ time and space.
3. To achieve logarithmic time, we must avoid merging. Instead, we can use binary search to find a partition.


### 2. Brute Solution

### Algorithm
1. Merge the two sorted arrays into a single sorted array of size $m+n$ using the two-pointer merge step of Merge Sort.
2. If $(m+n)$ is odd, return the element at index $(m+n)/2$.
3. If $(m+n)$ is even, return the average of the elements at $(m+n)/2 - 1$ and $(m+n)/2$.

### Complexity Analysis
- **Time Complexity:** $O(m + n)$ to merge the arrays.
- **Space Complexity:** $O(m + n)$ to store the merged array.

### Pros and Cons
- **Pros:** Simple, highly readable, works without complex edge-case logic.
- **Cons:** Violates the $O(\log(m+n))$ constraint and uses linear auxiliary space.

### C++17 Brute Force Code
```cpp
#include <vector>

class MedianBrute {
public:
    double findMedianSortedArrays(std::vector<int>& nums1, std::vector<int>& nums2) {
        int m = nums1.size();
        int n = nums2.size();
        std::vector<int> merged;
        merged.reserve(m + n);
        
        int i = 0, j = 0;
        while (i < m && j < n) {
            if (nums1[i] <= nums2[j]) {
                merged.push_back(nums1[i++]);
            } else {
                merged.push_back(nums2[j++]);
            }
        }
        
        while (i < m) merged.push_back(nums1[i++]);
        while (j < n) merged.push_back(nums2[j++]);
        
        int total = m + n;
        if (total % 2 != 0) {
            return merged[total / 2];
        } else {
            return (merged[total / 2 - 1] + merged[total / 2]) / 2.0;
        }
    }
};
```


### 3. Optimal Solution & Complexities

### Algorithm
1. Check if `nums1` is smaller than `nums2`. If not, recursively call the function swapping the arguments. This ensures we perform binary search on the smaller array.
2. Initialize binary search bounds on the smaller array: `low = 0`, `high = m`.
3. While `low <= high`:
   - Calculate partition indices:
     - $i = \text{low} + (\text{high} - \text{low}) / 2$
     - $j = (m + n + 1) / 2 - i$
   - Get boundary values (handling $-\infty$ and $+\infty$ conditions):
     - `leftA` = $(i == 0) ? \text{INT\_MIN} : nums1[i-1]$
     - `rightA` = $(i == m) ? \text{INT\_MAX} : nums1[i]$
     - `leftB` = $(j == 0) ? \text{INT\_MIN} : nums2[j-1]$
     - `rightB` = $(j == n) ? \text{INT\_MAX} : nums2[j]$
   - If `leftA <= rightB` and `leftB <= rightA` (Correct Partition):
     - If total elements $(m + n)$ is odd, return $\max(leftA, leftB)$.
     - If total elements $(m + n)$ is even, return $\frac{\max(leftA, leftB) + \min(rightA, rightB)}{2.0}$.
   - Else if `leftA > rightB`:
     - $i$ is too big, move left: `high = i - 1`.
   - Else:
     - $i$ is too small, move right: `low = i + 1`.
4. Return `0.0` as a fallback (should not be reached if inputs are valid).

### Dry Run
`nums1 = [1, 3]`, `nums2 = [2]`. Sizes: $m = 2, n = 1$. Total size $= 3$ (odd).
- Search array is `nums2` because $n < m$. Swap them:
  - `A = [2]` (size $m = 1$)
  - `B = [1, 3]` (size $n = 2$)
- Target size for left half: $(1 + 2 + 1) / 2 = 2$.
- Binary search range for $i$: `low = 0, high = 1`.

#### Iteration 1:
- $i = (0 + 1) / 2 = 0$.
- $j = 2 - 0 = 2$.
- Boundary values:
  - `leftA` = $(i==0) ? -\infty = -\infty$
  - `rightA` = `A[0] = 2`
  - `leftB` = `B[1] = 3`
  - `rightB` = $(j==2) ? +\infty = +\infty$
- Check conditions:
  - `leftA <= rightB` $\to -\infty \le \infty$ (True)
  - `leftB <= rightA` $\to 3 \le 2$ (False!)
- Since `leftB > rightA` ($3 > 2$), $i$ is too small. We need to move right.
- `low = i + 1 = 1`.

#### Iteration 2:
- `low = 1, high = 1`.
- $i = 1$.
- $j = 2 - 1 = 1$.
- Boundary values:
  - `leftA` = `A[0] = 2`
  - `rightA` = $(i==1) ? +\infty = +\infty$
  - `leftB` = `B[0] = 1`
  - `rightB` = `B[1] = 3`
- Check conditions:
  - `leftA <= rightB` $\to 2 \le 3$ (True)
  - `leftB <= rightA` $\to 1 \le \infty$ (True)
- Partition is correct!
- Since total size $3$ is odd, Median = $\max(leftA, leftB) = \max(2, 1) = 2.0$.

### Why Optimal
- We halve the search space of the smaller array at each step, resulting in a time complexity of $O(\log(\min(m, n)))$.
- No extra space is allocated.

### C++17 Optimal Code
```cpp
#include <vector>
#include <algorithm>
#include <climits>

class MedianOptimal {
public:
    double findMedianSortedArrays(std::vector<int>& nums1, std::vector<int>& nums2) {
        // Ensure nums1 is the smaller array
        if (nums1.size() > nums2.size()) {
            return findMedianSortedArrays(nums2, nums1);
        }
        
        int m = nums1.size();
        int n = nums2.size();
        
        int low = 0;
        int high = m;
        
        while (low <= high) {
            int partitionX = low + (high - low) / 2;
            int partitionY = (m + n + 1) / 2 - partitionX;
            
            // Boundary checks
            int maxLeftX = (partitionX == 0) ? INT_MIN : nums1[partitionX - 1];
            int minRightX = (partitionX == m) ? INT_MAX : nums1[partitionX];
            
            int maxLeftY = (partitionY == 0) ? INT_MIN : nums2[partitionY - 1];
            int minRightY = (partitionY == n) ? INT_MAX : nums2[partitionY];
            
            // Check if partition is correct
            if (maxLeftX <= minRightY && maxLeftY <= minRightX) {
                // If odd number of elements
                if ((m + n) % 2 != 0) {
                    return std::max(maxLeftX, maxLeftY);
                } 
                // If even number of elements
                else {
                    return (std::max(maxLeftX, maxLeftY) + std::min(minRightX, minRightY)) / 2.0;
                }
            } 
            else if (maxLeftX > minRightY) {
                // We are too far right in nums1, move left
                high = partitionX - 1;
            } 
            else {
                // We are too far left in nums1, move right
                low = partitionX + 1;
            }
        }
        
        return 0.0; // Fallback
    }
};
```


#### Complexity Summary

| Approach | Time Complexity | Auxiliary Space Complexity | Reasoning |
| :--- | :--- | :--- | :--- |
| **Brute Force (Merge)** | $O(m + n)$ | $O(m + n)$ | Merging both arrays requires copying all elements. |
| **Two-Pointer (Scan)** | $O(m + n)$ | $O(1)$ | Scan up to half the elements without extra storage. |
| **Optimal (Binary Search)** | $O(\log(\min(m, n)))$ | $O(1)$ | Binary search performed on the smaller array. |


---

## Q46 Bitonic Subarray

### 1. Problem Explanation

Given an array `arr` of $N$ integers, find the length of the **longest bitonic subarray**. 

A subarray is **bitonic** if it is first sorted in non-decreasing order, then sorted in non-increasing order. 
*Note:* A subarray that is only non-decreasing or only non-increasing is also considered bitonic (with the decreasing or increasing part having length 0).

### Input
- An integer array `arr` of size $N$ ($1 \le N \le 10^6$).
- Element values: $-10^9 \le arr[i] \le 10^9$.

### Output
- An integer representing the length of the longest bitonic subarray.

### Constraints
- Time Complexity: $O(N)$
- Auxiliary Space: $O(N)$ (Better: optimize to $O(1)$ space).

### Observations
1. A bitonic subarray has a single peak element at some index `i`.
2. To the left of `i` (within the subarray), elements must be non-decreasing: $arr[k-1] \le arr[k]$ for all $k \le i$.
3. To the right of `i` (within the subarray), elements must be non-increasing: $arr[k] \ge arr[k+1]$ for all $k \ge i$.
4. The peak `i` is counted in both the increasing part and the decreasing part.


### 2. Brute Solution

### Algorithm
1. Initialize `max_len = 1`.
2. Loop through each index $i$ from $0$ to $N-1$:
   - Set `len = 1`.
   - Traverse left from $i$ (using index $j = i-1$ down to 0) as long as `arr[j] <= arr[j+1]`. Increment `len`.
   - Traverse right from $i$ (using index $k = i+1$ up to $N-1$) as long as `arr[k] <= arr[k-1]`. Increment `len`.
   - Update `max_len = max(max_len, len)`.
3. Return `max_len`.

### Complexity Analysis
- **Time Complexity:** $O(N^2)$ in the worst case (e.g. `[1, 2, 3, 4, 5]`).
- **Space Complexity:** $O(1)$ auxiliary space.

### C++17 Brute Force Code
```cpp
#include <vector>
#include <algorithm>

class BitonicSubarrayBrute {
public:
    int maxBitonicLength(const std::vector<int>& arr) {
        int n = arr.size();
        if (n == 0) return 0;
        
        int max_len = 1;
        
        for (int i = 0; i < n; ++i) {
            int len = 1;
            
            // Expand left
            int j = i - 1;
            while (j >= 0 && arr[j] <= arr[j + 1]) {
                len++;
                j--;
            }
            
            // Expand right
            int k = i + 1;
            while (k < n && arr[k] <= arr[k - 1]) {
                len++;
                k++;
            }
            
            max_len = std::max(max_len, len);
        }
        
        return max_len;
    }
};
```


### 3. Optimal Solution & Complexities

### Algorithm
We can scan the array and count lengths on the fly:
1. Initialize `max_len = 1`, `inc_len = 1`, `dec_len = 0`.
2. We iterate through the array from $i = 1$ to $N-1$:
   - If `arr[i] > arr[i-1]`:
     - If we were in a decreasing phase (`dec_len > 0`), it means the previous bitonic subarray has ended. We calculate its length: `inc_len + dec_len`, update `max_len`, and reset `inc_len` to the count of the last increasing elements (which is 1 + the number of equal elements at the transition, but if it is strictly greater, it's just 1). Wait, how do we handle duplicates?
     - Let's track:
       - `inc_run`: length of current increasing run.
       - `dec_run`: length of current decreasing run.
       - `equal_run`: count of identical values at the peak or transition.
     - Let's use a simpler state machine to avoid complexity:
       - Keep track of `start` of the current bitonic subarray.
       - As we traverse, we can identify peak transitions.
       - Alternatively, we can find all local peak candidates in a single pass.
       - Let's design the one-pass algorithm using a direct pointer method:
         - A bitonic subarray is partitioned by peaks.
         - Let's traverse the array with a index pointer `i = 0`.
         - While `i < n-1`:
           - Find the end of the non-decreasing segment. This is the peak candidate.
           - Find the end of the non-increasing segment. This is the end of the bitonic subarray.
           - Calculate length. Update `max_len`.
           - The next bitonic subarray starts at the end of the decreasing segment? No, it could start earlier if the decreasing segment contains increasing subsegments.
           - Wait! The transition point from decreasing back to increasing is a valley. The next bitonic subarray can start at this valley.
           - So, if we find a valley, we start the next bitonic subarray from that valley.
           - Let's trace this:
             - An transition from decreasing to increasing occurs when `arr[i] < arr[i+1]` after we've been decreasing.
             - So the start of the next bitonic subarray is indeed that valley!
             - Let's write this transition logic clearly.

### Single-Pass Two-Pointer State Machine
1. Set `i = 0`, `max_len = 1`.
2. While `i < N-1`:
   - Keep track of the starting index: `start = i`.
   - **Step A (Non-decreasing phase):** Traverse as long as `arr[i] <= arr[i+1]`.
     - `i++`
   - **Step B (Non-increasing phase):** Traverse as long as `arr[i] >= arr[i+1]`.
     - Maintain a boolean `decreased = false` to know if we actually had a decreasing phase.
     - While `arr[i] >= arr[i+1]`, increment `i` and set `decreased = true`.
   - Update `max_len = max(max_len, i - start + 1)`.
   - If we finished a step where we did not decrease (e.g. we reached end of array or we hit a flat part), we must advance `i` to avoid infinite loop. But wait! If we didn't decrease, `i` has moved forward during Step A.
   - What is the starting point of the next bitonic subarray?
     - If we did a decreasing phase, the next subarray could start at the valley. The valley is exactly at the index `i` we reached! So the next iteration starts with `start = i` (meaning we don't change `i`).
     - But what if there are duplicates?
       - If `arr[i] == arr[i+1]`, it is allowed in both non-decreasing and non-increasing.
       - Let's test this logic on `[10, 20, 30, 40]`:
         - `start = 0`.
         - Step A: `i` increments to 3 (`arr[3] = 40`).
         - Step B: `arr[3] >= arr[4]` is false. Loop doesn't run.
         - `max_len = max(1, 3 - 0 + 1) = 4`.
         - Loop ends since `i = 3` (which is `n-1`). Correct!
       - Let's test on `[40, 30, 20, 10]`:
         - `start = 0`.
         - Step A: `arr[0] <= arr[1]` false. Loop doesn't run.
         - Step B: `i` increments to 3.
         - `max_len = max(1, 3 - 0 + 1) = 4`. Correct!
       - Let's test on `[10, 20, 30, 20, 10, 40, 30]`:
         - `start = 0`.
         - Step A: `i` goes to 2 (`arr[2] = 30`).
         - Step B: `i` goes to 4 (`arr[4] = 10`).
         - `max_len = max(1, 4 - 0 + 1) = 5`.
         - Next loop starts at `i = 4`.
         - `start = 4`.
         - Step A: `i` goes to 5 (`arr[5] = 40`).
         - Step B: `i` goes to 6 (`arr[6] = 30`).
         - `max_len = max(5, 6 - 4 + 1) = 5`. Correct!
       - Wait! What if there is a flat plateau? E.g., `[10, 20, 20, 10]`
         - `start = 0`.
         - Step A: `i` goes to 2 (`arr[2] = 20`).
         - Step B: `arr[2] >= arr[3]` (20 >= 10) $\to$ `i` goes to 3 (`arr[3] = 10`).
         - `max_len = max(1, 4) = 4`. Correct!
       - What if `[10, 20, 10, 10, 20]`?
         - `start = 0`.
         - Step A: `i` goes to 1.
         - Step B: `i` goes to 3.
         - `max_len = 4` (`[10, 20, 10, 10]`).
         - Next loop starts at `i = 3`.
         - `start = 3`.
         - Step A: `i` goes to 4.
         - `max_len = max(4, 4 - 3 + 1) = 4`. Correct!

Wait, is there any case where the next bitonic subarray should start *before* the current `i`?
Suppose `arr = [10, 22, 9, 33, 21, 50, 41, 60]`.
- Iteration 1:
  - `start = 0`.
  - Step A: `i` goes to 1 (`22`).
  - Step B: `i` goes to 2 (`9`).
  - `max_len = 3` (`[10, 22, 9]`).
  - Next search starts at `i = 2` (`9`).
- Iteration 2:
  - `start = 2`.
  - Step A: `i` goes to 3 (`33`).
  - Step B: `i` goes to 4 (`21`).
  - `max_len = max(3, 3) = 3`.
  - Next search starts at `i = 4` (`21`).
- Iteration 3:
  - `start = 4`.
  - Step A: `i` goes to 5 (`50`).
  - Step B: `i` goes to 6 (`41`).
  - `max_len = 3`.
  - Next search starts at `i = 6` (`41`).
- Iteration 4:
  - `start = 6`.
  - Step A: `i` goes to 7 (`60`).
  - Step B: no run.
  - `max_len = 3`.
All bitonic subarrays found are size 3. The logic is completely correct and runs in $O(N)$ time with $O(1)$ auxiliary space!

Let's double check if we need to do anything when `arr[i] > arr[i+1]` is not met in Step B.
Wait, if `arr[i] < arr[i+1]` at the end of Step B, we don't increment `i` manually because `i` has already advanced during Step A or Step B.
But what if `i` didn't advance at all?
If `arr[i] > arr[i+1]` at the start (meaning we cannot increase), then Step A doesn't run, but Step B will run and increment `i`.
What if `arr[i] == arr[i+1]`?
Then Step A runs and increments `i`.
Thus, `i` is guaranteed to advance by at least 1 in every outer loop iteration, preventing infinite loops.

Let's write this beautiful $O(1)$ space solution.

### C++17 Optimal Code
```cpp
#include <vector>
#include <algorithm>

class BitonicSubarrayOptimal {
public:
    int maxBitonicLength(const std::vector<int>& arr) {
        int n = arr.size();
        if (n <= 1) return n;
        
        int max_len = 1;
        int i = 0;
        
        while (i < n - 1) {
            int start = i;
            
            // Step A: Run through non-decreasing elements
            while (i < n - 1 && arr[i] <= arr[i + 1]) {
                i++;
            }
            
            // Step B: Run through non-increasing elements
            while (i < n - 1 && arr[i] >= arr[i + 1]) {
                i++;
            }
            
            // Update the maximum length found so far
            max_len = std::max(max_len, i - start + 1);
        }
        
        return max_len;
    }
};
```


#### Complexity Summary

| Approach | Time Complexity | Auxiliary Space Complexity | Reasoning |
| :--- | :--- | :--- | :--- |
| **Brute Force** | $O(N^2)$ | $O(1)$ | Expands left and right for every element in the worst case. |
| **Precomputation (Better)** | $O(N)$ | $O(N)$ | Uses two extra arrays of size $N$ for precomputing counts. |
| **Two-Pointer (Optimal)** | $O(N)$ | $O(1)$ | Uses a single pass with index tracking, no extra arrays. |


---

## Q47 Count Set Bits

### 1. Problem Explanation

Given a positive integer $N$, count the total number of set bits (1-bits) in the binary representation of all numbers from $1$ to $N$.

### Input
- A single integer $N$ ($1 \le N \le 10^9$).

### Output
- An integer representing the total count of set bits from $1$ to $N$.

### Constraints
- Time Complexity: Optimal $O(\log N)$
- Auxiliary Space: $O(1)$

### Observations
- Let's write the numbers from 0 to 7 in binary and observe:
  - 0: `000`
  - 1: `001`
  - 2: `010`
  - 3: `011`
  - 4: `100`
  - 5: `101`
  - 6: `110`
  - 7: `111`
- The total set bits for $N = 7$ is $12$.
- At each bit position, the bits alternate between 0 and 1 in a periodic pattern.


### 2. Brute Solution

### Algorithm
1. Iterate through each number $i$ from $1$ to $N$.
2. For each number, count the number of set bits (using `__builtin_popcount` or division by 2).
3. Add the count to the total.

### Complexity Analysis
- **Time Complexity:** $O(N \log N)$ because for each of the $N$ numbers we do $O(\log N)$ bit checks.
- **Space Complexity:** $O(1)$ auxiliary space.

### Pros and Cons
- **Pros:** Trivial to write.
- **Cons:** For $N = 10^9$, it performs $\approx 3 \times 10^{10}$ operations, leading to Time Limit Exceeded (TLE).

### C++17 Brute Force Code
```cpp
class CountSetBitsBrute {
public:
    long long countSetBits(int n) {
        long long total_bits = 0;
        for (int i = 1; i <= n; ++i) {
            int temp = i;
            while (temp > 0) {
                total_bits += (temp & 1);
                temp >>= 1;
            }
        }
        return total_bits;
    }
};
```


### 3. Optimal Solution & Complexities

### Algorithm
1. Initialize `total_bits = 0`.
2. Let $N' = N + 1$ (since we count from $0$ to $N$).
3. Loop through bit positions $p = 0, 1, 2, \dots$ as long as $2^p \le N$:
   - Calculate period size: `period = 1LL << (p + 1)`.
   - Calculate complete cycles: `complete_cycles = N_prime / period`.
   - Add bits from complete cycles: `total_bits += complete_cycles * (1LL << p)`.
   - Calculate remainder: `remainder = N_prime % period`.
   - Add bits from remainder: `total_bits += std::max(0LL, remainder - (1LL << p))`.
4. Return `total_bits`.

### Dry Run
Let's dry run for $N = 6$ ($N' = 7$).

| Position `p` | `period = 2^(p+1)` | `complete_cycles = 7 / period` | Bits from cycles | `remainder = 7 % period` | Bits from remainder | Total added |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **0** (LSB) | 2 | $7 / 2 = 3$ | $3 \times 1 = 3$ | $7 \% 2 = 1$ | $\max(0, 1 - 1) = 0$ | $3 + 0 = 3$ |
| **1** | 4 | $7 / 4 = 1$ | $1 \times 2 = 2$ | $7 \% 4 = 3$ | $\max(0, 3 - 2) = 1$ | $2 + 1 = 3$ |
| **2** | 8 | $7 / 8 = 0$ | $0 \times 4 = 0$ | $7 \% 8 = 7$ | $\max(0, 7 - 4) = 3$ | $0 + 3 = 3$ |

Grand Total $= 3 + 3 + 3 = 9$. Correct!

### Why Optimal
- The loop runs at most 31 times (since $N \le 10^9 < 2^{30}$).
- At each step, we perform basic $O(1)$ arithmetic operations.
- The space complexity is strictly $O(1)$.

### C++17 Optimal Code
```cpp
class CountSetBitsOptimal {
public:
    long long countSetBits(int n) {
        long long total_bits = 0;
        long long n_prime = n + 1; // Count includes 0
        
        // Loop through each bit position
        for (int p = 0; (1LL << p) <= n; ++p) {
            long long period = 1LL << (p + 1);
            long long complete_cycles = n_prime / period;
            
            // Contribution of fully completed cycles at bit position p
            total_bits += complete_cycles * (1LL << p);
            
            // Contribution of the remainder cycle
            long long remainder = n_prime % period;
            total_bits += std::max(0LL, remainder - (1LL << p));
        }
        
        return total_bits;
    }
};
```


#### Complexity Summary

| Approach | Time Complexity | Auxiliary Space Complexity | Reasoning |
| :--- | :--- | :--- | :--- |
| **Brute Force** | $O(N \log N)$ | $O(1)$ | Counts bits of all $N$ numbers individually. |
| **Recursive (Better)** | $O(\log N)$ | $O(\log N)$ | Logarithmic depth, each step takes $O(1)$ if finding MSB is optimized. |
| **Iterative (Optimal)** | $O(\log N)$ | $O(1)$ | Iterates up to the number of bits in $N$ ($\le 31$ iterations). |


---

## Q48 Longest Mountain

### 1. Problem Explanation

Given an integer array `arr`, return the length of the **longest subarray** that is a **mountain**. If no mountain subarray exists, return `0`.

A subarray `arr[i...j]` (of length $\ge 3$) is a **mountain** if:
1. There exists some index `k` with $i < k < j$ such that:
   - $arr[i] < arr[i+1] < \dots < arr[k-1] < arr[k]$ (strictly increasing)
   - $arr[k] > arr[k+1] > \dots > arr[j-1] > arr[j]$ (strictly decreasing)

### Input
- An integer array `arr` of size $N$ ($1 \le N \le 10^4$).
- Element values: $0 \le arr[i] \le 10^4$.

### Output
- An integer representing the length of the longest mountain subarray.

### Constraints
- Time Complexity: $O(N)$
- Auxiliary Space: $O(1)$ optimal ($O(N)$ acceptable as intermediate).

### Observations
1. A mountain must have at least 3 elements.
2. The peak of the mountain cannot be the first or last element of the subarray (i.e. we need both a non-empty increasing phase and a non-empty decreasing phase).
3. The transitions must be **strictly** increasing and **strictly** decreasing (no duplicates allowed inside the slopes).


### 2. Brute Solution

### Algorithm
1. Initialize `max_len = 0`.
2. For each element $i$ from $1$ to $N-2$:
   - Check if $i$ is a peak: `arr[i] > arr[i-1] && arr[i] > arr[i+1]`.
   - If it is, expand leftwards as long as `arr[left] > arr[left-1]` and rightwards as long as `arr[right] > arr[right+1]`.
   - Length of mountain $= right - left + 1$.
   - Update `max_len = max(max_len, length)`.
3. Return `max_len`.

### Complexity Analysis
- **Time Complexity:** $O(N^2)$ in the worst case (e.g. alternating zig-zag array).
- **Space Complexity:** $O(1)$ auxiliary space.

### C++17 Brute Force Code
```cpp
#include <vector>
#include <algorithm>

class MountainBrute {
public:
    int longestMountain(std::vector<int>& arr) {
        int n = arr.size();
        int max_len = 0;
        
        for (int i = 1; i < n - 1; ++i) {
            // Check if arr[i] is a peak
            if (arr[i] > arr[i - 1] && arr[i] > arr[i + 1]) {
                int left = i;
                int right = i;
                
                // Expand left
                while (left > 0 && arr[left] > arr[left - 1]) {
                    left--;
                }
                
                // Expand right
                while (right < n - 1 && arr[right] > arr[right + 1]) {
                    right++;
                }
                
                max_len = std::max(max_len, right - left + 1);
            }
        }
        
        return max_len;
    }
};
```


### 3. Optimal Solution & Complexities

### Algorithm
1. Initialize `max_len = 0` and `base = 0`.
2. While `base < N - 1`:
   - Set `end = base`.
   - **Step A:** If `end + 1 < N` and `arr[end] < arr[end + 1]`:
     - Move `end` forward while `end + 1 < N` and `arr[end] < arr[end + 1]`.
     - (We have now climbed to a peak).
     - **Step B:** If `end + 1 < N` and `arr[end] > arr[end + 1]`:
       - Move `end` forward while `end + 1 < N` and `arr[end] > arr[end + 1]`.
       - (We have now descended to the valley).
       - Update `max_len = max(max_len, end - base + 1)`.
       - Set `base = end` (the next mountain can start at the current valley).
     - Else:
       - If we didn't descend, `end` is not a valid peak. Set `base = end` (or `base = end + 1` if they are equal). To be safe, we can just set `base = end` and if it is equal, the next check will fail to increase and increment `base` by 1.
   - If we couldn't ascend at all (i.e. `arr[base] >= arr[base+1]`), increment `base` by 1.
3. Return `max_len`.

### Dry Run
Input: `arr = [2, 1, 4, 7, 3, 2, 5]`

| Iteration | `base` | Step A (Ascend) | Peak Index | Step B (Descend) | Valley Index | `max_len` Update | New `base` |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **1** | 0 | `2 < 1` (False) | - | - | - | - | `base++` $\to$ 1 |
| **2** | 1 | `1 < 4 < 7` | 3 | `7 > 3 > 2` | 5 | $\max(0, 5 - 1 + 1) = 5$ | `base = 5` |
| **3** | 5 | `2 < 5` | 6 | `5 > (end of array)` (False) | - | - | `base = 6` |
| **4** | 6 | `base = n - 1` $\to$ Loop Terminated | - | - | - | - | - |

Output: `5` (Subarray: `[1, 4, 7, 3, 2]`).

### Why Optimal
- We visit each element at most twice (once climbing up, once climbing down).
- No auxiliary memory is used.
- Lower bound is $\Omega(N)$ since we must examine all heights. Hence, $O(N)$ time and $O(1)$ space is optimal.

### C++17 Optimal Code
```cpp
#include <vector>
#include <algorithm>

class MountainOptimal {
public:
    int longestMountain(std::vector<int>& arr) {
        int n = arr.size();
        int max_len = 0;
        int base = 0;
        
        while (base < n - 1) {
            int end = base;
            
            // Step A: Climb the mountain (Strictly Increasing)
            if (end + 1 < n && arr[end] < arr[end + 1]) {
                while (end + 1 < n && arr[end] < arr[end + 1]) {
                    end++;
                }
                
                // Step B: Descend the mountain (Strictly Decreasing)
                if (end + 1 < n && arr[end] > arr[end + 1]) {
                    while (end + 1 < n && arr[end] > arr[end + 1]) {
                        end++;
                    }
                    // Valid mountain found! Update max_len
                    max_len = std::max(max_len, end - base + 1);
                    
                    // Next mountain can start from the valley
                    base = end;
                    continue;
                }
            }
            
            // If not a valid mountain, shift base forward
            base = std::max(base + 1, end);
        }
        
        return max_len;
    }
};
```


#### Complexity Summary

| Approach | Time Complexity | Auxiliary Space Complexity | Reasoning |
| :--- | :--- | :--- | :--- |
| **Brute Force** | $O(N^2)$ | $O(1)$ | Expands left and right from every potential peak. |
| **Two-Pass DP** | $O(N)$ | $O(N)$ | Uses two extra arrays of size $N$. |
| **Two-Pointer (Optimal)** | $O(N)$ | $O(1)$ | Single pass over array using sliding base pointer. |


---
