# Binary Search — Complete Topic Reference

## Q01 Binary Search Standard

### 1. Problem Explanation

Given a sorted array of integers `nums` and an integer `target`, write a function to search for `target` in `nums`. If `target` exists, then return its index. Otherwise, return `-1`.

You must write both **iterative** and **recursive** implementations.

### Input
- `nums`: A sorted array of integers (non-decreasing order).
- `target`: An integer to search for.

### Output
- The 0-based index of `target` in the array if it exists, otherwise `-1`.

### Constraints
- $1 \le \text{nums.length} \le 10^4$
- $-10^4 < \text{nums}[i], \text{target} < 10^4$
- All integers in `nums` are unique and sorted in ascending order.

### Key Observations
1. The input array is **already sorted**. This is the single most critical precondition for Binary Search.
2. Elements are unique, meaning there is at most one correct index.
3. Linear search takes $O(n)$ time. The sorted property allows us to discard half of the search space at each step, yielding $O(\log n)$ complexity.


### 2. Brute Solution

### Algorithm
Perform a linear scan from index `0` to `n-1`. Compare each element with the target.

### Dry Run
Input: `nums = [-1, 0, 3, 5, 9, 12]`, `target = 9`
- Index `0`: `nums[0] = -1` (Not target)
- Index `1`: `nums[1] = 0` (Not target)
- Index `2`: `nums[2] = 3` (Not target)
- Index `3`: `nums[3] = 5` (Not target)
- Index `4`: `nums[4] = 9` (Target found! Return index `4`)

### Complexity Analysis
- **Time Complexity:** $O(n)$ in the worst case (target is at the end or not present).
- **Space Complexity:** $O(1)$ auxiliary space.

### Pros/Cons
- **Pros:** Extremely simple; works on unsorted arrays.
- **Cons:** Inefficient for large sorted arrays.

### Commented C++17 Code
```cpp
#include <vector>

class BruteForceSearch {
public:
    int search(const std::vector<int>& nums, int target) {
        int n = nums.size();
        for (int i = 0; i < n; ++i) {
            // Compare each element with target
            if (nums[i] == target) {
                return i; // Return index immediately if found
            }
        }
        return -1; // Target not found
    }
};
```


### 3. Optimal Solution & Complexities

The optimal solution is **Binary Search** implemented both **iteratively** and **recursively**.

### Algorithm (Iterative)
1. Initialize two pointers: `low = 0` and `high = nums.size() - 1`.
2. While `low <= high`:
   - Compute `mid = low + (high - low) / 2`.
   - If `nums[mid] == target`, return `mid`.
   - If `nums[mid] < target`, discard the left half by setting `low = mid + 1`.
   - If `nums[mid] > target`, discard the right half by setting `high = mid - 1`.
3. If the loop terminates without finding the target, return `-1`.

### Algorithm (Recursive)
1. Define a helper function `binarySearchHelper(nums, low, high, target)`.
2. Base case: If `low > high`, target is not present. Return `-1`.
3. Calculate `mid = low + (high - low) / 2`.
4. If `nums[mid] == target`, return `mid`.
5. If `nums[mid] < target`, recursively search the right half: `binarySearchHelper(nums, mid + 1, high, target)`.
6. If `nums[mid] > target`, recursively search the left half: `binarySearchHelper(nums, low, mid - 1, target)`.

### Full Dry Run
Input: `nums = [-1, 0, 3, 5, 9, 12]`, `target = 9`

#### Iterative Dry Run:
- **Initialization:** `low = 0`, `high = 5`
- **Iteration 1:**
  - `low = 0`, `high = 5`
  - `mid = 0 + (5 - 0) / 2 = 2`
  - `nums[mid] = nums[2] = 3`
  - Since `3 < 9`, target must be in the right half. Update `low = mid + 1 = 3`.
- **Iteration 2:**
  - `low = 3`, `high = 5`
  - `mid = 3 + (5 - 3) / 2 = 4`
  - `nums[mid] = nums[4] = 9`
  - Target found! Return `4`.

### Why Optimal?
Each comparison halves the remaining search space. This log-scale division means the maximum number of iterations is $\approx \log_2(n)$. This is mathematically optimal for comparison-based searching in a sorted array.

### Beautiful C++17 Code

```cpp
#include <vector>

class BinarySearch {
public:
    // 1. Iterative Implementation (Standard & Preferred for Space Complexity)
    int searchIterative(const std::vector<int>& nums, int target) {
        int low = 0;
        int high = static_cast<int>(nums.size()) - 1;

        while (low <= high) {
            // Avoid (low + high)/2 to prevent integer overflow
            int mid = low + (high - low) / 2;

            if (nums[mid] == target) {
                return mid; // Target found
            } else if (nums[mid] < target) {
                low = mid + 1; // Discard left half
            } else {
                high = mid - 1; // Discard right half
            }
        }

        return -1; // Target not found
    }

    // 2. Recursive Implementation
    int searchRecursive(const std::vector<int>& nums, int target) {
        return recursiveHelper(nums, 0, static_cast<int>(nums.size()) - 1, target);
    }

private:
    int recursiveHelper(const std::vector<int>& nums, int low, int high, int target) {
        // Base case: Search space is exhausted
        if (low > high) {
            return -1;
        }

        // Avoid integer overflow
        int mid = low + (high - low) / 2;

        if (nums[mid] == target) {
            return mid;
        } else if (nums[mid] < target) {
            // Tail recursion: Search right half
            return recursiveHelper(nums, mid + 1, high, target);
        } else {
            // Tail recursion: Search left half
            return recursiveHelper(nums, low, mid - 1, target);
        }
    }
};
```


#### Complexity Summary

| Metric | Iterative | Recursive | Reasoning |
| :--- | :--- | :--- | :--- |
| **Best Case Time** | $O(1)$ | $O(1)$ | Target is exactly at the initial `mid` position. |
| **Average Case Time** | $O(\log n)$ | $O(\log n)$ | On average, the search space is halved $\log_2 n$ times. |
| **Worst Case Time** | $O(\log n)$ | $O(\log n)$ | When target is at the boundary or absent. |
| **Space Complexity** | $O(1)$ | $O(\log n)$ | Iterative uses constant memory. Recursive uses $O(\log n)$ call stack space. |


---

## Q02 Binary Search On Answer

### 1. Problem Explanation

Given an array of integers `nums` and an integer `k`, you need to partition the array into `k` contiguous subarrays such that the **maximum sum of any subarray is minimized**. Return this minimized maximum sum.

This problem is also classically known as the **Book Allocation Problem** or the **Painter's Partition Problem**.

### Input
- `nums`: A vector of positive integers.
- `k`: An integer representing the number of contiguous subarrays to partition the array into.

### Output
- An integer representing the minimized maximum subarray sum.

### Constraints
- $1 \le \text{nums.length} \le 10^5$
- $1 \le \text{nums}[i] \le 10^5$
- $1 \le k \le \text{nums.length}$

### Key Observations
1. **Contiguous Subarrays:** The division must be sequential (no sorting or reordering of elements allowed).
2. **Monotonic Decision Space:**
   - If it is possible to partition the array such that no subarray exceeds a sum $S$, then it is definitely possible for any sum greater than $S$.
   - If it is impossible for sum $S$, then it is also impossible for any sum less than $S$.
   - This monotonic property enables **Binary Search on the Answer**.
3. **Subarray Limit ($k$):** If the array can be partitioned into $M$ subarrays (where $M \le k$) each having a sum $\le S$, then it can also be partitioned into exactly $k$ subarrays (since we can always split any existing subarray further without increasing the maximum sum).


### 2. Brute Solution

### Algorithm
Linear scan through every possible answer from $\max(\text{nums})$ to $\sum(\text{nums})$. For each value, use the greedy check to see if it can form $\le k$ subarrays. The first value that succeeds is our minimum maximum subarray sum.

### Dry Run
`nums = [7, 2, 5, 10, 8]`, `k = 2`
- $\max(\text{nums}) = 10$, $\sum(\text{nums}) = 32$.
- Search range: $[10, 32]$
- Try $x = 10$:
  - Subarrays: `[7, 2]`, `[5]`, `[10]`, `[8]`. Total subarrays = 4. Since $4 > 2$, $x = 10$ is invalid.
- Try $x = 11, 12, \dots$
- Try $x = 18$:
  - Subarrays: `[7, 2, 5]` (sum 14), `[10, 8]` (sum 18). Total subarrays = 2. Valid! Return 18.

### Complexity Analysis
- **Time Complexity:** $O((\sum(\text{nums}) - \max(\text{nums})) \times n)$. In the worst case, this can be extremely slow if the sum is large.
- **Space Complexity:** $O(1)$ auxiliary space.

### Pros/Cons
- **Pros:** Conceptually simple.
- **Cons:** Too slow; times out for large array sums.

### Commented C++17 Code
```cpp
#include <vector>
#include <numeric>
#include <algorithm>

class BruteForce {
private:
    // Helper to check if a limit is feasible with <= k subarrays
    bool isValid(const std::vector<int>& nums, long long limit, int k) {
        long long current_sum = 0;
        int subarrays = 1;
        for (int num : nums) {
            if (current_sum + num > limit) {
                // Start a new subarray
                subarrays++;
                current_sum = num;
                if (subarrays > k) return false;
            } else {
                current_sum += num;
            }
        }
        return true;
    }

public:
    int splitArray(const std::vector<int>& nums, int k) {
        long long low = *std::max_element(nums.begin(), nums.end());
        long long high = std::accumulate(nums.begin(), nums.end(), 0LL);

        // Linear scan of the entire range
        for (long long limit = low; limit <= high; ++limit) {
            if (isValid(nums, limit, k)) {
                return static_cast<int>(limit);
            }
        }
        return -1;
    }
};
```


### 3. Optimal Solution & Complexities

The optimal solution is to apply binary search on the range of answers.

### Algorithm
1. Set `low = max(nums)` and `high = sum(nums)`.
2. Initialize `ans = high`.
3. While `low <= high`:
   - Compute `mid = low + (high - low) / 2`.
   - Run `isValid(nums, mid, k)`.
   - If `isValid` returns `true`:
     - Update `ans = mid` (candidate answer).
     - Try to find a smaller maximum sum: `high = mid - 1`.
   - If `isValid` returns `false`:
     - Limit is too tight. Increase allowed sum: `low = mid + 1`.
4. Return `ans`.

### Helper Function `isValid(nums, limit, k)`
- Track `current_sum = 0` and `subarrays_count = 1`.
- For each `num` in `nums`:
  - If `current_sum + num > limit`:
    - Increment `subarrays_count`.
    - Reset `current_sum = num`.
  - Else:
    - Add `num` to `current_sum`.
- Return `subarrays_count <= k`.

### Full Dry Run
`nums = [7, 2, 5, 10, 8]`, `k = 2`
- `low = 10` (max element), `high = 32` (sum of elements).

#### Iteration 1:
- `low = 10`, `high = 32`, `mid = 21`
- Check `isValid(21)`:
  - Accumulate: `7 + 2 + 5 = 14 <= 21`
  - Next element `10`: `14 + 10 = 24 > 21` $\rightarrow$ Start new subarray. count = 2. current = 10.
  - Next element `8`: `10 + 8 = 18 <= 21`.
  - End of array. Subarrays count = 2 $\le$ 2. Valid!
- Update `ans = 21`, `high = 20`.

#### Iteration 2:
- `low = 10`, `high = 20`, `mid = 15`
- Check `isValid(15)`:
  - Accumulate: `7 + 2 + 5 = 14 <= 15`
  - Next element `10`: `14 + 10 = 24 > 15` $\rightarrow$ Start new subarray. count = 2. current = 10.
  - Next element `8`: `10 + 8 = 18 > 15` $\rightarrow$ Start new subarray. count = 3. current = 8.
  - Subarrays count = 3 > 2. Invalid!
- Update `low = 16`.

#### Iteration 3:
- `low = 16`, `high = 20`, `mid = 18`
- Check `isValid(18)`:
  - Accumulate: `7 + 2 + 5 = 14 <= 18`
  - Next element `10`: `14 + 10 = 24 > 18` $\rightarrow$ Start new subarray. count = 2. current = 10.
  - Next element `8`: `10 + 8 = 18 <= 18`.
  - Subarrays count = 2 $\le$ 2. Valid!
- Update `ans = 18`, `high = 17`.

#### Iteration 4:
- `low = 16`, `high = 17`, `mid = 16`
- Check `isValid(16)`:
  - Accumulate: `7 + 2 + 5 = 14 <= 16`
  - Next element `10`: `14 + 10 = 24 > 16` $\rightarrow$ Start new subarray. count = 2. current = 10.
  - Next element `8`: `10 + 8 = 18 > 16` $\rightarrow$ Start new subarray. count = 3. current = 8.
  - Subarrays count = 3 > 2. Invalid!
- Update `low = 17`.

#### Iteration 5:
- `low = 17`, `high = 17`, `mid = 17`
- Check `isValid(17)`:
  - Accumulate: `7 + 2 + 5 = 14 <= 17`
  - Next element `10`: `14 + 10 = 24 > 17` $\rightarrow$ Start new subarray. count = 2. current = 10.
  - Next element `8`: `10 + 8 = 18 > 17` $\rightarrow$ Start new subarray. count = 3. current = 8.
  - Subarrays count = 3 > 2. Invalid!
- Update `low = 18`.
- Loop terminates since `low (18) > high (17)`.
- Final answer: `18`.

### Why Optimal?
Binary search divides the search space of size $S$ (where $S = \sum(\text{nums}) - \max(\text{nums})$) by half each time. The validation helper runs in $O(n)$ time. The resulting total time complexity is $O(n \log(\sum(\text{nums})))$, which is extremely efficient and easily runs within limits for $N = 10^5$ and sum $\approx 10^{10}$.

### Beautiful C++17 Code

```cpp
#include <vector>
#include <numeric>
#include <algorithm>

class Solution {
private:
    // Greedily checks if it's possible to split nums into <= k subarrays
    // such that the maximum sum of any subarray does not exceed 'limit'.
    bool isValid(const std::vector<int>& nums, long long limit, int k) {
        long long current_sum = 0;
        int subarrays_count = 1;

        for (int num : nums) {
            // If a single element exceeds limit, it's impossible.
            // (Though low is initialized to max_element, so this is safe)
            if (num > limit) {
                return false;
            }

            if (current_sum + num > limit) {
                // Assign current element to a new subarray
                subarrays_count++;
                current_sum = num;

                // If number of subarrays exceeds allowed k, limit is invalid
                if (subarrays_count > k) {
                    return false;
                }
            } else {
                current_sum += num;
            }
        }
        return true;
    }

public:
    int splitArray(const std::vector<int>& nums, int k) {
        if (nums.empty()) return 0;

        // The answer range is between the maximum single element (min possible max sum)
        // and the sum of all elements (max possible max sum).
        long long low = *std::max_element(nums.begin(), nums.end());
        long long high = std::accumulate(nums.begin(), nums.end(), 0LL);
        long long ans = high;

        while (low <= high) {
            long long mid = low + (high - low) / 2;

            if (isValid(nums, mid, k)) {
                ans = mid;         // Record potential valid minimized sum
                high = mid - 1;    // Try to find a smaller maximum sum
            } else {
                low = mid + 1;     // Limit too small, search right side
            }
        }

        return static_cast<int>(ans);
    }
};
```


#### Complexity Summary

- **Time Complexity:**
  - **Initialization:** Finding the maximum element takes $O(n)$ time. Summing elements takes $O(n)$ time.
  - **Binary Search Loop:** The search range is $R = \sum(\text{nums}) - \max(\text{nums})$. The loop runs $\log_2(R)$ times.
  - **Validation Check:** Each call to `isValid` takes $O(n)$ time.
  - **Total Time Complexity:** $O(n \log(\sum(\text{nums})))$. For $N = 10^5$ and sum $= 10^{10}$, $\log_2(10^{10}) \approx 34$ iterations. $34 \times 10^5 \approx 3.4 \times 10^6$ operations, which easily finishes in < 10ms.
- **Space Complexity:**
  - $O(1)$ auxiliary space as we only use a few tracking variables.


---

## Q03 Floor Ceiling

### 1. Problem Explanation

Given a sorted array `nums` of size `n` and a target value `x`, find the **floor** and **ceiling** of `x` in the array. 

- **Floor** of `x` is the largest element in the array that is smaller than or equal to `x` (i.e., $\le x$).
- **Ceiling** of `x` is the smallest element in the array that is greater than or equal to `x` (i.e., $\ge x$).

If the floor or ceiling does not exist, return `-1` for that value. Return the result as a pair of integers: `{floor, ceiling}`.

### Input
- `nums`: A sorted array of integers.
- `x`: The target integer.

### Output
- A pair of integers `{floor_value, ceiling_value}`. If not found, represent it with `-1`.

### Constraints
- $1 \le \text{nums.length} \le 10^5$
- $-10^9 \le \text{nums}[i], x \le 10^9$
- The array is sorted in ascending order.

### Key Observations
1. **Sorted Structure:** The array is already sorted, suggesting we can find both boundaries in $O(\log n)$ using binary search.
2. **Floor condition:** Any element $y \le x$ is a candidate. To find the *largest* such element, we must search as far right as possible.
3. **Ceiling condition:** Any element $y \ge x$ is a candidate. To find the *smallest* such element, we must search as far left as possible.


### 2. Brute Solution

### Algorithm
1. Initialize `floor_val = -1` and `ceiling_val = -1`.
2. Loop through the array:
   - If `nums[i] <= x`, update `floor_val = nums[i]` (since array is sorted, this gets updated to the largest such value).
   - If `nums[i] >= x` and `ceiling_val == -1`, set `ceiling_val = nums[i]` (the first element $\ge x$ is the smallest).
3. Return `{floor_val, ceiling_val}`.

### Dry Run
`nums = [1, 2, 8, 10, 10, 12, 19]`, `x = 5`
- `i = 0`: `nums[0] = 1 <= 5` $\rightarrow$ `floor_val = 1`. `nums[0] < 5` $\rightarrow$ ceiling unaffected.
- `i = 1`: `nums[1] = 2 <= 5` $\rightarrow$ `floor_val = 2`.
- `i = 2`: `nums[2] = 8 > 5` $\rightarrow$ Floor unaffected. `8 >= 5`, since `ceiling_val == -1`, set `ceiling_val = 8`.
- For subsequent elements, `floor_val` remains `2` (since they are all $> 5$). `ceiling_val` remains `8` because it is no longer `-1`.
- Return `{2, 8}`.

### Complexity Analysis
- **Time Complexity:** $O(n)$ because we iterate through the array once.
- **Space Complexity:** $O(1)$ auxiliary space.

### Pros/Cons
- **Pros:** Trivial to write.
- **Cons:** Inefficient. We scan elements that we already know cannot be the answer.

### Commented C++17 Code
```cpp
#include <vector>
#include <utility>

class BruteForce {
public:
    std::pair<int, int> getFloorAndCeiling(const std::vector<int>& nums, int x) {
        int floor_val = -1;
        int ceiling_val = -1;

        for (int num : nums) {
            // Find floor: largest element <= x
            if (num <= x) {
                floor_val = num; 
            }
            // Find ceiling: first element >= x
            if (num >= x && ceiling_val == -1) {
                ceiling_val = num;
            }
        }
        return {floor_val, ceiling_val};
    }
};
```


### 3. Optimal Solution & Complexities

The optimal approach performs binary search to find the floor and ceiling in $O(\log n)$ time. We can either write two separate binary search functions or combine the logic. Writing them separately is cleaner and easier to debug during an interview.

### Algorithm (Floor)
1. Initialize `low = 0`, `high = n - 1`, `ans = -1`.
2. While `low <= high`:
   - `mid = low + (high - low) / 2`.
   - If `nums[mid] == x`, return `nums[mid]` (exact match is the floor).
   - If `nums[mid] < x`, update `ans = nums[mid]` (candidate floor found) and search right: `low = mid + 1`.
   - If `nums[mid] > x`, search left: `high = mid - 1`.
3. Return `ans`.

### Algorithm (Ceiling)
1. Initialize `low = 0`, `high = n - 1`, `ans = -1`.
2. While `low <= high`:
   - `mid = low + (high - low) / 2`.
   - If `nums[mid] == x`, return `nums[mid]` (exact match is the ceiling).
   - If `nums[mid] > x`, update `ans = nums[mid]` (candidate ceiling found) and search left: `high = mid - 1`.
   - If `nums[mid] < x`, search right: `low = mid + 1`.
3. Return `ans`.

### Full Dry Run
`nums = [1, 2, 8, 10, 10, 12, 19]`, `x = 5`

#### Floor Search:
- `low = 0`, `high = 6`, `ans = -1`
- **Step 1:** `mid = 3`, `nums[mid] = 10`. Since `10 > 5`, search left: `high = 2`.
- **Step 2:** `low = 0`, `high = 2`, `mid = 1`, `nums[mid] = 2`. Since `2 < 5`, record `ans = 2` and search right: `low = 2`.
- **Step 3:** `low = 2`, `high = 2`, `mid = 2`, `nums[mid] = 8`. Since `8 > 5`, search left: `high = 1`.
- Loop ends (`low > high`). Return `ans = 2`.

#### Ceiling Search:
- `low = 0`, `high = 6`, `ans = -1`
- **Step 1:** `mid = 3`, `nums[mid] = 10`. Since `10 > 5`, record `ans = 10` and search left: `high = 2`.
- **Step 2:** `low = 0`, `high = 2`, `mid = 1`, `nums[mid] = 2`. Since `2 < 5`, search right: `low = 2`.
- **Step 3:** `low = 2`, `high = 2`, `mid = 2`, `nums[mid] = 8`. Since `8 > 5`, record `ans = 8` and search left: `high = 1`.
- Loop ends. Return `ans = 8`.

Result: `{2, 8}`.

### Why Optimal?
By dividing the search space in half at each step, we find each boundary in $\log_2(n)$ steps. Running two binary searches takes $2 \times \log_2(n)$ steps, which is still $O(\log n)$ and extremely fast.

### Beautiful C++17 Code

```cpp
#include <vector>
#include <utility>

class Solution {
private:
    // Helper to find the floor of x in nums
    int findFloor(const std::vector<int>& nums, int x) {
        int low = 0;
        int high = static_cast<int>(nums.size()) - 1;
        int floor_val = -1;

        while (low <= high) {
            int mid = low + (high - low) / 2;

            if (nums[mid] == x) {
                return nums[mid]; // Exact match is always the floor
            } else if (nums[mid] < x) {
                floor_val = nums[mid]; // Candidate floor
                low = mid + 1;         // Search right for a larger candidate
            } else {
                high = mid - 1;        // Too large, search left
            }
        }
        return floor_val;
    }

    // Helper to find the ceiling of x in nums
    int findCeiling(const std::vector<int>& nums, int x) {
        int low = 0;
        int high = static_cast<int>(nums.size()) - 1;
        int ceiling_val = -1;

        while (low <= high) {
            int mid = low + (high - low) / 2;

            if (nums[mid] == x) {
                return nums[mid]; // Exact match is always the ceiling
            } else if (nums[mid] > x) {
                ceiling_val = nums[mid]; // Candidate ceiling
                high = mid - 1;          // Search left for a smaller candidate
            } else {
                low = mid + 1;           // Too small, search right
            }
        }
        return ceiling_val;
    }

public:
    std::pair<int, int> getFloorAndCeiling(const std::vector<int>& nums, int x) {
        int floor_val = findFloor(nums, x);
        int ceiling_val = findCeiling(nums, x);
        return {floor_val, ceiling_val};
    }
};
```


#### Complexity Summary

| Scenario | Time Complexity | Space Complexity | Reason |
| :--- | :--- | :--- | :--- |
| **Best Case** | $O(1)$ | $O(1)$ | Target $x$ exists at the exact midpoint for both functions. |
| **Average Case** | $O(\log n)$ | $O(1)$ | Standard binary search space halving. |
| **Worst Case** | $O(\log n)$ | $O(1)$ | Target $x$ is not present, requiring full tree traversal. |


---

## Q04 Square Root

### 1. Problem Explanation

Given a non-negative integer `n`, compute and return the square root of `n`. Since the return type is an integer, the decimal digits are truncated, and only the integer part of the result is returned (i.e., $\lfloor \sqrt{n} \rfloor$, which is the floor of the square root of $n$).

You must solve this **without** using any built-in exponentiation function or operator (like `sqrt(x)` or `x ** 0.5`).

### Input
- `n`: A non-negative integer.

### Output
- An integer representing $\lfloor \sqrt{n} \rfloor$.

### Constraints
- $0 \le n \le 2^{31} - 1$ (fits in a standard 32-bit signed integer).

### Key Observations
1. **Monotonicity:** The square function $f(x) = x^2$ is monotonically increasing for non-negative integers.
2. **Search Range:** For any non-negative integer $n$, the integer square root must lie in the range $[0, n]$. Specifically, for $n \ge 1$, the answer lies in $[1, n]$.
3. **Truncated Floor:** We want to find the largest integer $x$ such that $x^2 \le n$.


### 2. Brute Solution

### Algorithm
Start from `0`, increment by `1`, check if `i * i <= n`. Return `i - 1` when the condition is violated.

### Dry Run
Input: `n = 8`
- `i = 0`: $0^2 = 0 \le 8$ (True)
- `i = 1`: $1^2 = 1 \le 8$ (True)
- `i = 2`: $2^2 = 4 \le 8$ (True)
- `i = 3`: $3^2 = 9 > 8$ (False $\rightarrow$ Stop. Return `i - 1 = 2`).

### Complexity Analysis
- **Time Complexity:** $O(\sqrt{n})$ because we check integers up to $\lfloor \sqrt{n} \rfloor + 1$.
- **Space Complexity:** $O(1)$ auxiliary space.

### Pros/Cons
- **Pros:** Simple.
- **Cons:** Very slow for large numbers. If $n = 2 \times 10^9$, $\sqrt{n} \approx 46340$ operations. While $46340$ is small, we can do it in just 31 operations using binary search.

### Commented C++17 Code
```cpp
class BruteForce {
public:
    int mySqrt(int n) {
        long long i = 0;
        // Increment until i * i exceeds n
        while (i * i <= n) {
            i++;
        }
        return static_cast<int>(i - 1);
    }
};
```


### 3. Optimal Solution & Complexities

The optimal solution is **Binary Search** in the range $[0, n]$.

### Algorithm
1. If `n == 0` or `n == 1`, return `n` directly.
2. Initialize `low = 1`, `high = n`, and `ans = 0`.
3. While `low <= high`:
   - Compute `mid = low + (high - low) / 2`.
   - Compute `square = (long long)mid * mid` to avoid integer overflow.
   - If `square == n`, return `mid`.
   - If `square < n`, record `ans = mid` and search the right half: `low = mid + 1`.
   - If `square > n`, search the left half: `high = mid - 1`.
4. Return `ans`.

### Full Dry Run
Input: `n = 8`
- `low = 1`, `high = 8`, `ans = 0`

#### Iteration 1:
- `mid = 1 + (8 - 1) / 2 = 4`
- `square = 4 * 4 = 16`
- Since `16 > 8`, `mid` is too large. Update `high = mid - 1 = 3`.

#### Iteration 2:
- `low = 1`, `high = 3`
- `mid = 1 + (3 - 1) / 2 = 2`
- `square = 2 * 2 = 4`
- Since `4 <= 8`, record `ans = 2`. Search right: `low = mid + 1 = 3`.

#### Iteration 3:
- `low = 3`, `high = 3`
- `mid = 3 + (3 - 3) / 2 = 3`
- `square = 3 * 3 = 9`
- Since `9 > 8`, search left: `high = mid - 1 = 2`.

- Loop terminates because `low (3) > high (2)`.
- Return `ans = 2`.

### Why Optimal?
At each step, we divide the search space by half. For $N = 2^{31} - 1$, the maximum search steps is $\approx \log_2(2^{31}) \approx 31$, which is extremely fast and execution is almost instantaneous.

### Beautiful C++17 Code

```cpp
class Solution {
public:
    int mySqrt(int n) {
        // Base cases
        if (n == 0 || n == 1) {
            return n;
        }

        int low = 1;
        int high = n;
        int ans = 0;

        while (low <= high) {
            int mid = low + (high - low) / 2;
            
            // Cast to long long to prevent 32-bit signed integer overflow
            long long square = static_cast<long long>(mid) * mid;

            if (square == n) {
                return mid; // Found exact square root
            } else if (square < n) {
                ans = mid;      // Record potential answer
                low = mid + 1;  // Try to find a larger square root
            } else {
                high = mid - 1; // Square is too large, search lower values
            }
        }

        return ans;
    }
};
```


#### Complexity Summary

| Metric | Complexity | Reasoning |
| :--- | :--- | :--- |
| **Best Case Time** | $O(1)$ | Target is found at the first midpoint (e.g., $n = 16$, first mid is $8$, second is $4$, exact match). |
| **Average Case Time** | $O(\log n)$ | Search space of size $n$ is halved at each step. |
| **Worst Case Time** | $O(\log n)$ | Occurs when there is no perfect square root (e.g., $n = 8$), requiring full search space exhaustion. |
| **Space Complexity** | $O(1)$ | No extra space used besides a few variables. |


---

## Q05 Peak Element

### 1. Problem Explanation

A peak element is an element that is strictly greater than its neighbors.

Given a 0-indexed integer array `nums`, find a peak element, and return its index. If the array contains multiple peaks, return the index to **any of the peaks**.

You may imagine that `nums[-1] = nums[n] = -∞`. In other words, an element is always considered to be strictly greater than a neighbor that is outside the array.

You must write an algorithm that runs in $O(\log n)$ time.

### Input
- `nums`: An integer array.

### Output
- The 0-based index of any peak element in `nums`.

### Constraints
- $1 \le \text{nums.length} \le 1000$
- $-2^{31} \le \text{nums}[i] \le 2^{31} - 1$
- `nums[i] != nums[i + 1]` for all valid `i`. (No two adjacent elements are equal).

### Key Observations
1. **Implicit Boundaries:** The virtual boundaries at indices `-1` and `n` are $-\infty$. This guarantees that a peak element **always exists** in any non-empty array.
2. **No Adjacent Duplicates:** Because `nums[i] != nums[i+1]`, we will never encounter flat plateaus. Every step is either strictly ascending or strictly descending.
3. **Local Property vs. Global Sorting:** Even though the array is **not sorted**, we can still apply Binary Search. This is a critical concept: binary search does not strictly require a sorted array; it only requires a way to decide which half contains the target.


### 2. Brute Solution

### Algorithm
Perform a linear scan. The first element that is greater than its right neighbor is a peak.
*Proof:* If we iterate from left to right, we know `nums[i] > nums[i-1]` is true for all visited elements (otherwise we would have stopped at `i-1` since it would be greater than `nums[i]`). Thus, we only need to check if `nums[i] > nums[i+1]`.

### Dry Run
`nums = [1, 2, 1, 3, 5, 6, 4]`
- `i = 0`: `nums[0] = 1`. `nums[1] = 2`. `1 < 2`, not a peak.
- `i = 1`: `nums[1] = 2`. `nums[2] = 1`. `2 > 1` (And we know `2 > nums[0]` since we reached here). Return index `1`.

### Complexity Analysis
- **Time Complexity:** $O(n)$ in the worst case (e.g., strictly increasing array, where peak is at the end).
- **Space Complexity:** $O(1)$ auxiliary space.

### Pros/Cons
- **Pros:** Simple, works with duplicate adjacent elements if we adapt it.
- **Cons:** Inefficient for large arrays; does not meet the $O(\log n)$ requirement.

### Commented C++17 Code
```cpp
#include <vector>

class BruteForce {
public:
    int findPeakElement(const std::vector<int>& nums) {
        int n = nums.size();
        for (int i = 0; i < n; ++i) {
            // Check boundary conditions along with neighbors
            bool left_ok = (i == 0 || nums[i] > nums[i - 1]);
            bool right_ok = (i == n - 1 || nums[i] > nums[i + 1]);
            
            if (left_ok && right_ok) {
                return i;
            }
        }
        return -1;
    }
};
```


### 3. Optimal Solution & Complexities

The optimal solution is **Binary Search** using a convergence template.

### Algorithm
1. Initialize `low = 0` and `high = nums.size() - 1`.
2. While `low < high`:
   - Compute `mid = low + (high - low) / 2`.
   - Compare `nums[mid]` with `nums[mid + 1]`:
     - If `nums[mid] < nums[mid + 1]`, we are on an ascending slope. The peak is on the right side. Set `low = mid + 1`.
     - If `nums[mid] > nums[mid + 1]`, we are on a descending slope. The peak is on the left side (including `mid`). Set `high = mid`.
3. When the loop terminates, `low == high` and both point to a peak element. Return `low`.

### Full Dry Run
`nums = [1, 2, 1, 3, 5, 6, 4]`
- `low = 0`, `high = 6`

#### Iteration 1:
- `mid = 0 + (6 - 0) / 2 = 3`
- `nums[mid] = nums[3] = 3`
- Compare with `nums[mid+1] = nums[4] = 5`.
- Since `3 < 5` (ascending slope), peak must be on the right.
- Update `low = mid + 1 = 4`.

#### Iteration 2:
- `low = 4`, `high = 6`
- `mid = 4 + (6 - 4) / 2 = 5`
- `nums[mid] = nums[5] = 6`
- Compare with `nums[mid+1] = nums[6] = 4`.
- Since `6 > 4` (descending slope), peak must be on the left (including `mid`).
- Update `high = mid = 5`.

#### Iteration 3:
- `low = 4`, `high = 5`
- `mid = 4 + (5 - 4) / 2 = 4`
- `nums[mid] = nums[4] = 5`
- Compare with `nums[mid+1] = nums[5] = 6`.
- Since `5 < 6` (ascending slope), peak must be on the right.
- Update `low = mid + 1 = 5`.

- **Termination:** `low = 5`, `high = 5`. Loop terminates since `low < high` is false.
- Return `low = 5` (which points to element `6`, a valid peak).

### Why Optimal?
Each step evaluates the slope at `mid` and shrinks the search space by half. The check is local ($O(1)$) and avoids accessing out-of-bounds indices because `mid` can never equal `high` when `low < high`, ensuring `mid + 1` is always a valid index. This guarantees finding a peak in $O(\log n)$ time.

### Beautiful C++17 Code

```cpp
#include <vector>

class Solution {
public:
    int findPeakElement(const std::vector<int>& nums) {
        int low = 0;
        int high = static_cast<int>(nums.size()) - 1;

        // Use low < high because we compare mid with mid + 1.
        // This prevents mid + 1 from going out of bounds.
        while (low < high) {
            int mid = low + (high - low) / 2;

            if (nums[mid] < nums[mid + 1]) {
                // Ascending slope: peak lies strictly to the right
                low = mid + 1;
            } else {
                // Descending slope: peak lies to the left (including mid)
                high = mid;
            }
        }

        // low and high converge to the peak index
        return low;
    }
};
```


#### Complexity Summary

- **Time Complexity:**
  - **Best Case:** $O(1)$ if the array has only 1 element.
  - **Average Case:** $O(\log n)$ as we halve the search space at each iteration.
  - **Worst Case:** $O(\log n)$ when the peak is at either extreme end of the array.
- **Space Complexity:**
  - $O(1)$ auxiliary space since we only store two pointer variables.


---

## Q06 Rotation Count

### 1. Problem Explanation

Given an ascending sorted array of distinct integers `nums` that has been rotated (shifted) to the right by some unknown number of times, find the number of times the array was rotated.

For example, if the original sorted array was `[1, 2, 3, 4, 5]`:
- Rotated 1 time: `[5, 1, 2, 3, 4]`
- Rotated 2 times: `[4, 5, 1, 2, 3]`
- Rotated 0 times: `[1, 2, 3, 4, 5]`

### Input
- `nums`: A rotated sorted array of distinct integers.

### Output
- An integer representing the rotation count.

### Constraints
- $1 \le \text{nums.length} \le 10^5$
- $-10^9 \le \text{nums}[i] \le 10^9$
- All elements in the array are **distinct**.

### Key Observations
1. **Relation to Minimum Element:** The rotation count is exactly equal to the 0-based index of the **minimum element** in the array.
   - For `[4, 5, 1, 2, 3]`, the minimum element is `1` at index `2`. The array was rotated 2 times.
   - For `[1, 2, 3]`, the minimum element is `1` at index `0`. The array was rotated 0 times.
2. **Rotated Sorted Properties:** A rotated sorted array consists of two sorted sub-segments (e.g., `[4, 5]` and `[1, 2, 3]`). The boundary between these two segments is the minimum element (inflection point).
3. **Binary Search Applicability:** We can find the index of the minimum element in $O(\log n)$ time using binary search.


### 2. Brute Solution

### Algorithm
Scan the array from left to right. Keep track of the minimum value and its index. Return the index of the minimum.

### Dry Run
`nums = [4, 5, 1, 2, 3]`
- Start: `min_val = nums[0] = 4`, `min_idx = 0`
- `i = 1`: `nums[1] = 5`. Not smaller.
- `i = 2`: `nums[2] = 1 < 4`. Update `min_val = 1`, `min_idx = 2`.
- `i = 3`: `nums[3] = 2`. Not smaller.
- `i = 4`: `nums[4] = 3`. Not smaller.
- Return `min_idx = 2`.

### Complexity Analysis
- **Time Complexity:** $O(n)$ because we inspect every element once.
- **Space Complexity:** $O(1)$ auxiliary space.

### Pros/Cons
- **Pros:** Simple, handles duplicate values without changes.
- **Cons:** Inefficient; fails to meet the $O(\log n)$ requirement.

### Commented C++17 Code
```cpp
#include <vector>

class BruteForce {
public:
    int findRotationCount(const std::vector<int>& nums) {
        int n = nums.size();
        int min_val = nums[0];
        int min_idx = 0;

        for (int i = 1; i < n; ++i) {
            // Keep track of the smallest element's index
            if (nums[i] < min_val) {
                min_val = nums[i];
                min_idx = i;
            }
        }
        return min_idx;
    }
};
```


### 3. Optimal Solution & Complexities

The optimal solution is to find the index of the minimum element using binary search.

### Algorithm
1. Initialize `low = 0` and `high = nums.size() - 1`.
2. While `low < high`:
   - Compute `mid = low + (high - low) / 2`.
   - If `nums[mid] > nums[high]`, it means the right half is unsorted and contains the minimum element. Set `low = mid + 1`.
   - Else (if `nums[mid] <= nums[high]`), it means the right half is sorted, and the minimum element is either at `mid` or to its left. Set `high = mid`.
3. When the loop terminates, `low == high`. Return `low`.

### Full Dry Run
`nums = [4, 5, 1, 2, 3]`
- `low = 0`, `high = 4`

#### Iteration 1:
- `mid = 0 + (4 - 0) / 2 = 2`
- `nums[mid] = nums[2] = 1`, `nums[high] = nums[4] = 3`.
- Compare: `1 <= 3`. The right side is sorted.
- Update `high = mid = 2`.

#### Iteration 2:
- `low = 0`, `high = 2`
- `mid = 0 + (2 - 0) / 2 = 1`
- `nums[mid] = nums[1] = 5`, `nums[high] = nums[2] = 1`.
- Compare: `5 > 1`. The minimum must be to the right of `mid`.
- Update `low = mid + 1 = 2`.

- **Termination:** `low = 2`, `high = 2`. Loop terminates because `low < high` is false.
- Return `low = 2`.

### Why Optimal?
We reduce our search space by half at each step by comparing `nums[mid]` with `nums[high]`. This yields a logarithmic time complexity of $O(\log n)$ with $O(1)$ space.

### Beautiful C++17 Code

```cpp
#include <vector>

class Solution {
public:
    int findRotationCount(const std::vector<int>& nums) {
        if (nums.empty()) return 0;

        int low = 0;
        int high = static_cast<int>(nums.size()) - 1;

        // Use low < high to avoid infinite loop when high is updated to mid
        while (low < high) {
            int mid = low + (high - low) / 2;

            if (nums[mid] > nums[high]) {
                // Inflection point (minimum) is in the right half
                low = mid + 1;
            } else {
                // Minimum is mid or lies in the left half
                high = mid;
            }
        }

        // low and high converge to the index of the minimum element
        return low;
    }
};
```


#### Complexity Summary

| Case | Time Complexity | Space Complexity | Reason |
| :--- | :--- | :--- | :--- |
| **Best Case** | $O(1)$ | $O(1)$ | Array of size 1 (loop doesn't run). |
| **Average Case** | $O(\log n)$ | $O(1)$ | Standard binary search space reduction. |
| **Worst Case** | $O(\log n)$ | $O(1)$ | Inflection point requires full search path. |


---

## Q07 Median Row Sorted Matrix

### 1. Problem Explanation

Given an $R \times C$ matrix where each row is sorted in non-decreasing order, find the median of the matrix. 
- The total number of elements in the matrix ($R \times C$) is always odd.
- A median is defined as the middle element if all the elements of the matrix were sorted in a single linear array. Since $R \times C$ is odd, there is exactly one median element at index $(R \times C) / 2$ in the sorted linear representation.

### Input
- `matrix`: A 2D grid of size $R \times C$ of integers.
- `R`: Number of rows.
- `C`: Number of columns.

### Output
- Return a single integer representing the median of the matrix.

### Constraints
- $1 \le R, C \le 2000$
- $1 \le \text{matrix}[i][j] \le 2000$
- $R \times C$ is odd.

### Observations
1. Since each row is sorted, the minimum element of the matrix must be in the first column, and the maximum element must be in the last column.
2. For an element $X$ to be the median in a sorted list of length $N = R \times C$ (where $N$ is odd), there must be exactly $(N - 1) / 2$ elements smaller than $X$, and $(N - 1) / 2$ elements larger than $X$. In other words, at least $(R \times C) / 2 + 1$ elements must be less than or equal to $X$.


### 2. Brute Solution

### Algorithm
1. Create a 1D array/vector of size $R \times C$.
2. Traverse the matrix and copy all elements to the vector.
3. Sort the vector in non-decreasing order.
4. Return the element at index $(R \times C) / 2$.

### Dry Run
Matrix:
```
[1, 3, 5]
[2, 6, 9]
[3, 6, 9]
```
1. Flattened list: `[1, 3, 5, 2, 6, 9, 3, 6, 9]`
2. Sorted list: `[1, 2, 3, 3, 5, 6, 6, 9, 9]`
3. Index of median: $(3 \times 3) / 2 = 4$.
4. Element at index 4 is `5`.

### Complexity Analysis
- **Time Complexity:** $O(R \cdot C \log(R \cdot C))$ to copy and sort.
- **Space Complexity:** $O(R \cdot C)$ to store the flattened elements.

### Pros and Cons
- **Pros:** Extremely simple to implement; works for unsorted matrices too.
- **Cons:** Inefficient; does not utilize the sorted properties of rows; consumes substantial memory.

### C++17 Code
```cpp
#include <vector>
#include <algorithm>

class Solution {
public:
    int findMedianBruteForce(const std::vector<std::vector<int>>& matrix) {
        int r = matrix.size();
        int c = matrix[0].size();
        std::vector<int> flattened;
        flattened.reserve(r * c);

        // Copy elements
        for (int i = 0; i < r; ++i) {
            for (int j = 0; j < c; ++j) {
                flattened.push_back(matrix[i][j]);
            }
        }

        // Sort
        std::sort(flattened.begin(), flattened.end());

        // Return middle element
        return flattened[(r * c) / 2];
    }
};
```


### 3. Optimal Solution & Complexities

### Algorithm
1. Find the global minimum (`low`) and global maximum (`high`) of the matrix.
   - `low` can be initialized to the minimum of the first column.
   - `high` can be initialized to the maximum of the last column.
2. The target position of the median is $K = (R \times C) / 2 + 1$.
3. Loop while `low < high`:
   - Calculate `mid = low + (high - low) / 2`.
   - Count the number of elements in the matrix that are less than or equal to `mid`.
     - To count in each row, perform a binary search (`upper_bound`) which returns an iterator to the first element strictly greater than `mid`. The distance from the beginning of the row to this iterator is the count of elements $\le$ `mid`.
   - If the total count of elements $\le$ `mid` is strictly less than $K$, we set `low = mid + 1`.
   - Otherwise, we set `high = mid`.
4. When `low == high`, `low` (or `high`) is the median.

### Dry Run
Matrix:
```
[1, 3, 5]
[2, 6, 9]
[3, 6, 9]
```
$R = 3, C = 3$, Total elements = 9. $K = 9 / 2 + 1 = 5$.
Initial `low` = $\min(1, 2, 3) = 1$.
Initial `high` = $\max(5, 9, 9) = 9$.

**Iteration 1:**
- `mid = 1 + (9 - 1) / 2 = 5`
- Count elements $\le 5$ in each row:
  - Row 0: `[1, 3, 5]` $\rightarrow$ upper_bound for 5 is index 3 (count = 3)
  - Row 1: `[2, 6, 9]` $\rightarrow$ upper_bound for 5 is index 1 (count = 1, elements: 2)
  - Row 2: `[3, 6, 9]` $\rightarrow$ upper_bound for 5 is index 1 (count = 1, elements: 3)
  - Total count = $3 + 1 + 1 = 5$.
- Is Total count ($5$) $\ge K$ ($5$)? Yes.
- We update: `high = mid = 5`.

**Iteration 2:**
- `low = 1`, `high = 5`.
- `mid = 1 + (5 - 1) / 2 = 3`
- Count elements $\le 3$ in each row:
  - Row 0: `[1, 3, 5]` $\rightarrow$ count = 2 (elements: 1, 3)
  - Row 1: `[2, 6, 9]` $\rightarrow$ count = 1 (elements: 2)
  - Row 2: `[3, 6, 9]` $\rightarrow$ count = 1 (elements: 3)
  - Total count = $2 + 1 + 1 = 4$.
- Is Total count ($4$) $\ge K$ ($5$)? No.
- We update: `low = mid + 1 = 4`.

**Iteration 3:**
- `low = 4`, `high = 5`.
- `mid = 4 + (5 - 4) / 2 = 4`
- Count elements $\le 4$ in each row:
  - Row 0: `[1, 3, 5]` $\rightarrow$ count = 2 (elements: 1, 3)
  - Row 1: `[2, 6, 9]` $\rightarrow$ count = 1 (elements: 2)
  - Row 2: `[3, 6, 9]` $\rightarrow$ count = 1 (elements: 3)
  - Total count = $2 + 1 + 1 = 4$.
- Is Total count ($4$) $\ge K$ ($5$)? No.
- We update: `low = mid + 1 = 5`.

**End of loop:**
- `low = 5`, `high = 5`. Loop terminates since `low < high` is false.
- Median is `5`.

### C++17 Code
```cpp
#include <vector>
#include <algorithm>

class Solution {
private:
    // Helper function to count elements less than or equal to 'mid' in the entire matrix
    int countLessEqual(const std::vector<std::vector<int>>& matrix, int mid) {
        int count = 0;
        for (const auto& row : matrix) {
            // std::upper_bound returns iterator to the first element > mid
            // subtracting row.begin() gives the number of elements <= mid
            count += std::upper_bound(row.begin(), row.end(), mid) - row.begin();
        }
        return count;
    }

public:
    int findMedian(const std::vector<std::vector<int>>& matrix) {
        int r = matrix.size();
        int c = matrix[0].size();
        
        // Find minimum and maximum elements in the matrix
        int low = matrix[0][0];
        int high = matrix[0][c - 1];
        
        for (int i = 1; i < r; ++i) {
            low = std::min(low, matrix[i][0]);
            high = std::max(high, matrix[i][c - 1]);
        }
        
        // The index of the median is (r * c) / 2.
        // We want to find the first value where count of elements <= value is at least K
        int k = (r * c) / 2 + 1;
        
        while (low < high) {
            int mid = low + (high - low) / 2;
            
            if (countLessEqual(matrix, mid) < k) {
                // If count of elements <= mid is less than k, mid cannot be the median.
                // The median must be strictly greater than mid.
                low = mid + 1;
            } else {
                // If count >= k, mid could be the median.
                // We try to find if there is a smaller valid number.
                high = mid;
            }
        }
        
        return low;
    }
};
```


#### Complexity Summary

- **Time Complexity:**
  - Finding `low` and `high` takes $O(R)$ time.
  - The binary search runs $\log(\text{max\_val} - \text{min\_val})$ times.
  - In each step, we query `std::upper_bound` for each row, taking $O(R \log C)$ time.
  - **Total Time Complexity:** $O(R \log C \cdot \log(\text{max\_val} - \text{min\_val}))$. For $1 \le \text{matrix}[i][j] \le 10^9$, this is roughly $30 \cdot R \log C$.
- **Space Complexity:**
  - $O(1)$ auxiliary space since we only store a few variables.


---

## Q08 Allocate Pages

### 1. Problem Explanation

Given an array `pages` of $N$ integers, where `pages[i]` represents the number of pages in the $i$-th book. There are $M$ students. The task is to allocate all the books to the $M$ students such that:
1. Each book is allocated to exactly one student.
2. Contiguous books must be allocated to a single student (i.e., a student can only read a contiguous subsegment of books).
3. All books must be allocated.
4. Each student gets at least one book.

The goal is to allocate the books in such a way that the **maximum number of pages assigned to any student is minimized**. If it is not possible to allocate books (e.g., if students $M > N$), return `-1`.

### Input
- `pages`: An array of $N$ integers representing pages in each book.
- `M`: Number of students.

### Output
- Return an integer representing the minimized maximum number of pages a student has to read. Return `-1` if allocation is impossible.

### Constraints
- $1 \le N \le 10^5$
- $1 \le \text{pages}[i] \le 10^5$
- $1 \le M \le 10^5$

### Observations
1. If $M > N$, we cannot allocate books because each student must get at least one book. Thus, return `-1`.
2. If $M = 1$, the single student has to read all the books. The answer is the sum of all pages.
3. If $M = N$, each student gets exactly one book. The student who gets the book with the maximum number of pages will read the most. Thus, the answer is the maximum element in `pages`.
4. Therefore, the answer must lie in the range $[\max(\text{pages}), \sum(\text{pages})]$.


### 2. Brute Solution

### Algorithm
1. Check if $M > N$. If so, return `-1`.
2. Find the starting search value `low` = $\max(\text{pages})$ and the ending search value `high` = $\sum(\text{pages})$.
3. Iterate through every value $X$ from `low` to `high`:
   - Check if $X$ is a valid allocation using a greedy simulator.
   - The first $X$ that is valid is our answer (since we are searching in increasing order).

### Complexity Analysis
- **Time Complexity:** $O(N \times (\sum(\text{pages}) - \max(\text{pages})))$. In the worst case, if the sum is $10^{10}$, this will TLE.
- **Space Complexity:** $O(1)$ auxiliary space.

### Pros and Cons
- **Pros:** Simple, linear scan on value range.
- **Cons:** Too slow, will result in Time Limit Exceeded (TLE) for large inputs.

### C++17 Code
```cpp
#include <vector>
#include <numeric>
#include <algorithm>

class Solution {
private:
    bool isValidLimit(const std::vector<int>& pages, int n, int m, long long maxLimit) {
        int studentCount = 1;
        long long currentPages = 0;

        for (int i = 0; i < n; ++i) {
            if (pages[i] > maxLimit) return false; // Single book exceeds limit
            
            if (currentPages + pages[i] <= maxLimit) {
                currentPages += pages[i];
            } else {
                studentCount++;
                currentPages = pages[i];
                if (studentCount > m) return false;
            }
        }
        return true;
    }

public:
    int allocatePagesBruteForce(const std::vector<int>& pages, int m) {
        int n = pages.size();
        if (m > n) return -1;

        int low = *std::max_element(pages.begin(), pages.end());
        long long high = std::accumulate(pages.begin(), pages.end(), 0LL);

        for (long long limit = low; limit <= high; ++limit) {
            if (isValidLimit(pages, n, m, limit)) {
                return limit;
            }
        }
        return -1;
    }
};
```


### 3. Optimal Solution & Complexities

### Algorithm
1. Check if $M > N$. If yes, return `-1`.
2. Find the minimum possible answer: `low` = maximum element in `pages`.
3. Find the maximum possible answer: `high` = sum of all elements in `pages`.
4. Perform Binary Search on the range `[low, high]`:
   - Calculate `mid = low + (high - low) / 2`.
   - Check if it is possible to allocate books such that no student reads more than `mid` pages using `isValidLimit(mid)`:
     - Initialize `studentCount = 1`, `currentPagesSum = 0`.
     - For each book, if `currentPagesSum + book_pages <= mid`, add to current student.
     - Else, increment `studentCount` and assign the book to the new student: `currentPagesSum = book_pages`.
     - If `studentCount > M`, return false.
   - If `isValidLimit(mid)` is true, then `mid` is a valid answer. Record `mid` as the current best answer, and try to find a smaller maximum pages by searching the left half: `high = mid - 1`.
   - If `isValidLimit(mid)` is false, it means `mid` is too small. We must search the right half: `low = mid + 1`.
5. Return the recorded best answer.

### Dry Run
`pages` = `[12, 34, 67, 90]`, $M = 2$.
$N = 4$, $M \le N$ is true.
`low` = $\max(12, 34, 67, 90) = 90$.
`high` = $12 + 34 + 67 + 90 = 203$.
Let `ans = -1`.

**Iteration 1:**
- `mid = 90 + (203 - 90) / 2 = 146`
- Check `isValidLimit(146)`:
  - Student 1: `12` $\rightarrow$ `12 + 34 = 46` $\rightarrow$ `46 + 67 = 113` $\le 146$. (Can't add 90 because $113 + 90 = 203 > 146$).
  - Student 2: starts with `90` $\le 146$.
  - Total students used = 2.
  - Since $2 \le M$, `isValidLimit(146)` is **true**.
- Update: `ans = 146`, `high = mid - 1 = 145`.

**Iteration 2:**
- `low = 90`, `high = 145`.
- `mid = 90 + (145 - 90) / 2 = 117`
- Check `isValidLimit(117)`:
  - Student 1: `12` $\rightarrow$ `12 + 34 = 46` $\rightarrow$ `46 + 67 = 113` $\le 117$.
  - Student 2: `90` $\le 117$.
  - Total students used = 2.
  - Since $2 \le M$, `isValidLimit(117)` is **true**.
- Update: `ans = 117`, `high = mid - 1 = 116`.

**Iteration 3:**
- `low = 90`, `high = 116`.
- `mid = 90 + (116 - 90) / 2 = 103`
- Check `isValidLimit(103)`:
  - Student 1: `12` $\rightarrow$ `12 + 34 = 46` $\le 103$. (Can't add 67 because $46 + 67 = 113 > 103$).
  - Student 2: `67` $\le 103$. (Can't add 90 because $67 + 90 = 157 > 103$).
  - Student 3: `90` $\le 103$.
  - Total students used = 3.
  - Since $3 > M$, `isValidLimit(103)` is **false**.
- Update: `low = mid + 1 = 104`.

**Iteration 4:**
- `low = 104`, `high = 116`.
- `mid = 104 + (116 - 104) / 2 = 110`
- Check `isValidLimit(110)`:
  - Student 1: `12` $\rightarrow$ `12 + 34 = 46` $\le 110$. (Can't add 67 because $46 + 67 = 113 > 110$).
  - Student 2: `67` $\le 110$. (Can't add 90 because $67 + 90 = 157 > 110$).
  - Student 3: `90` $\le 110$.
  - Total students used = 3.
  - Since $3 > M$, `isValidLimit(110)` is **false**.
- Update: `low = mid + 1 = 111`.

**Iteration 5:**
- `low = 111`, `high = 116`.
- `mid = 111 + (116 - 111) / 2 = 113`
- Check `isValidLimit(113)`:
  - Student 1: `12` $\rightarrow$ `12 + 34 = 46` $\rightarrow$ `46 + 67 = 113` $\le 113$.
  - Student 2: `90` $\le 113$.
  - Total students used = 2.
  - Since $2 \le M$, `isValidLimit(113)` is **true**.
- Update: `ans = 113`, `high = mid - 1 = 112`.

**Iteration 6:**
- `low = 111`, `high = 112`.
- `mid = 111 + (112 - 111) / 2 = 111`
- Check `isValidLimit(111)`:
  - Student 1: `12` $\rightarrow$ `12 + 34 = 46` $\le 111$. (Can't add 67 because $46 + 67 = 113 > 111$).
  - Student 2: `67` $\le 111$. (Can't add 90 because $67 + 90 = 157 > 111$).
  - Student 3: `90` $\le 111$.
  - Total students used = 3.
  - Since $3 > M$, `isValidLimit(111)` is **false**.
- Update: `low = mid + 1 = 112`.

**Iteration 7:**
- `low = 112`, `high = 112`.
- `mid = 112`
- Check `isValidLimit(112)`:
  - Student 1: `12` $\rightarrow$ `12 + 34 = 46` $\le 112$. (Can't add 67 because $46 + 67 = 113 > 112$).
  - Student 2: `67` $\le 112$. (Can't add 90 because $67 + 90 = 157 > 112$).
  - Student 3: `90` $\le 112$.
  - Total students used = 3.
  - Since $3 > M$, `isValidLimit(112)` is **false**.
- Update: `low = mid + 1 = 113`.

**Loop Ends:**
- `low` (113) > `high` (112). Loop terminates.
- Return `ans = 113`.

### C++17 Code
```cpp
#include <vector>
#include <numeric>
#include <algorithm>

class Solution {
private:
    // Helper function to check if it's possible to allocate books
    // such that no student reads more than 'maxLimit' pages.
    bool isValidLimit(const std::vector<int>& pages, int n, int m, long long maxLimit) {
        int studentCount = 1;
        long long currentPagesSum = 0;

        for (int i = 0; i < n; ++i) {
            // If a single book has more pages than the maximum limit allowed,
            // we cannot satisfy the condition.
            if (pages[i] > maxLimit) {
                return false;
            }

            if (currentPagesSum + pages[i] <= maxLimit) {
                currentPagesSum += pages[i];
            } else {
                // Assign to the next student
                studentCount++;
                currentPagesSum = pages[i];
                
                // If the number of students required exceeds available students, return false
                if (studentCount > m) {
                    return false;
                }
            }
        }
        return true;
    }

public:
    int findPages(const std::vector<int>& pages, int m) {
        int n = pages.size();
        
        // If there are more students than books, allocation is impossible
        if (m > n) {
            return -1;
        }

        // Search space:
        // 'low' represents the minimum possible answer (maximum of single book pages)
        // 'high' represents the maximum possible answer (sum of all pages)
        int low = *std::max_element(pages.begin(), pages.end());
        long long high = std::accumulate(pages.begin(), pages.end(), 0LL);
        
        int ans = -1;

        while (low <= high) {
            long long mid = low + (high - low) / 2;

            if (isValidLimit(pages, n, m, mid)) {
                ans = mid;          // Record candidate answer
                high = mid - 1;     // Try to find a smaller maximum limit
            } else {
                low = mid + 1;      // Increase the limit
            }
        }

        return ans;
    }
};
```


#### Complexity Summary

- **Time Complexity:**
  - Finding maximum and sum takes $O(N)$ time.
  - The binary search range length is $S = \sum(\text{pages}) - \max(\text{pages})$. The binary search takes $\log(S)$ iterations.
  - In each iteration, we run `isValidLimit` which scans the array of size $N$ once, taking $O(N)$ time.
  - **Total Time Complexity:** $O(N \log(\sum(\text{pages})))$. For $N = 10^5$ and page values up to $10^5$, sum is $10^{10}$, $\log_2(10^{10}) \approx 34$. Total operations $\approx 3.4 \times 10^6$, which runs well within 0.05 seconds.
- **Space Complexity:**
  - $O(1)$ auxiliary space. We do not use any extra space during search.


---

## Q09 Aggressive Cows

### 1. Problem Explanation

Given an array `stalls` of $N$ integers representing the coordinates of stalls on a straight line. There are $C$ cows which need to be placed in these stalls such that the **minimum distance between any two of them is as large as possible**.
Return this maximum minimum distance.

### Input
- `stalls`: An array of $N$ integers representing stall positions.
- `C`: Number of cows.

### Output
- Return an integer representing the maximum possible value of the minimum distance between any two cows.

### Constraints
- $2 \le N \le 10^5$
- $0 \le \text{stalls}[i] \le 10^9$
- $2 \le C \le N$

### Observations
1. If we sort the stall positions, we can easily calculate the distance between any two stalls.
2. The minimum possible distance between any two cows is `1` (if two stalls are adjacent).
3. The maximum possible distance between any two cows cannot exceed the distance between the first and last stall: `stalls[N-1] - stalls[0]` (after sorting).
4. As the target minimum distance $D$ increases, it becomes harder (or impossible) to place all $C$ cows in the stalls. If a distance $D$ is possible, then any distance $< D$ is also possible. This represents a monotonic search space: `[Possible, Possible, ..., Possible, Impossible, Impossible]`.


### 2. Brute Solution

### Algorithm
1. Sort the `stalls` array.
2. The search range of distance starts at $1$ and goes up to $high = \text{stalls}[N-1] - \text{stalls}[0]$.
3. Iterate $D$ from $1$ to $high$:
   - Check if we can place $C$ cows with a minimum distance of $D$ using a greedy placement function.
   - If we can, update the answer to $D$.
   - If we cannot, return the last successful $D$ (which is $D-1$).

### Complexity Analysis
- **Time Complexity:** 
  - Sorting: $O(N \log N)$
  - Search loop: $O(N \times \text{stalls}[N-1])$ in the worst case, which can be up to $10^{14}$ operations and will TLE.
- **Space Complexity:** $O(1)$ auxiliary space.

### Pros and Cons
- **Pros:** Simple, sequential search.
- **Cons:** Very slow for large coordinates (e.g. coordinates up to $10^9$).

### C++17 Code
```cpp
#include <vector>
#include <algorithm>

class Solution {
private:
    bool canPlaceCows(const std::vector<int>& stalls, int n, int cows, int minDist) {
        int count = 1; // Place the first cow at stalls[0]
        int lastPosition = stalls[0];

        for (int i = 1; i < n; ++i) {
            if (stalls[i] - lastPosition >= minDist) {
                count++;
                lastPosition = stalls[i];
                if (count == cows) return true;
            }
        }
        return false;
    }

public:
    int aggressiveCowsBruteForce(std::vector<int>& stalls, int cows) {
        int n = stalls.size();
        std::sort(stalls.begin(), stalls.end());

        int maxDist = stalls[n - 1] - stalls[0];
        int ans = 1;

        for (int d = 1; d <= maxDist; ++d) {
            if (canPlaceCows(stalls, n, cows, d)) {
                ans = d;
            } else {
                break;
            }
        }
        return ans;
    }
};
```


### 3. Optimal Solution & Complexities

### Algorithm
1. Sort the `stalls` array in non-decreasing order.
2. Initialize `low = 1` (the minimum possible distance) and `high = stalls[N-1] - stalls[0]` (the maximum possible distance).
3. Initialize `ans = 1`.
4. Perform Binary Search on the range `[low, high]`:
   - Calculate `mid = low + (high - low) / 2`.
   - Check if we can place $C$ cows with a minimum separation of `mid` using `canPlaceCows(stalls, N, C, mid)`.
   - If `canPlaceCows` is true:
     - This means `mid` is a valid minimum distance. We record `ans = mid`.
     - Since we want to maximize the minimum distance, we try to find a larger valid distance by searching the right half: `low = mid + 1`.
   - If `canPlaceCows` is false:
     - This means `mid` is too large to place $C$ cows. We must search the left half: `high = mid - 1`.
5. Return `ans`.

### Dry Run
`stalls` = `[1, 2, 8, 4, 9]`, $C = 3$.
1. Sort `stalls` $\rightarrow$ `[1, 2, 4, 8, 9]`.
2. $N = 5$.
3. `low = 1`, `high = 9 - 1 = 8`.
4. Let `ans = 1`.

**Iteration 1:**
- `mid = 1 + (8 - 1) / 2 = 4`
- Check `canPlaceCows(stalls, 5, 3, 4)`:
  - Cow 1: placed at `1`.
  - Next stall $\ge 1 + 4 = 5$. First stall matching is `8`. Cow 2 placed at `8`.
  - Next stall $\ge 8 + 4 = 12$. No stalls matching.
  - Cows placed = 2. We need 3. Return **false**.
- Update: `high = mid - 1 = 3`.

**Iteration 2:**
- `low = 1`, `high = 3`.
- `mid = 1 + (3 - 1) / 2 = 2`
- Check `canPlaceCows(stalls, 5, 3, 2)`:
  - Cow 1: placed at `1`.
  - Next stall $\ge 1 + 2 = 3$. First stall matching is `4`. Cow 2 placed at `4`.
  - Next stall $\ge 4 + 2 = 6$. First stall matching is `8`. Cow 3 placed at `8`.
  - Cows placed = 3. Return **true**.
- Update: `ans = 2`, `low = mid + 1 = 3`.

**Iteration 3:**
- `low = 3`, `high = 3`.
- `mid = 3 + (3 - 3) / 2 = 3`
- Check `canPlaceCows(stalls, 5, 3, 3)`:
  - Cow 1: placed at `1`.
  - Next stall $\ge 1 + 3 = 4$. First stall matching is `4`. Cow 2 placed at `4`.
  - Next stall $\ge 4 + 3 = 7$. First stall matching is `8`. Cow 3 placed at `8`.
  - Cows placed = 3. Return **true**.
- Update: `ans = 3`, `low = mid + 1 = 4`.

**Loop Ends:**
- `low` (4) > `high` (3). Loop terminates.
- Return `ans = 3`.

### C++17 Code
```cpp
#include <vector>
#include <algorithm>

class Solution {
private:
    // Helper function to check if we can place 'cows' cows in 'stalls'
    // such that the minimum distance between any two cows is at least 'minDist'
    bool canPlaceCows(const std::vector<int>& stalls, int n, int cows, int minDist) {
        int count = 1; // Place the first cow in the first stall
        int lastPosition = stalls[0];

        for (int i = 1; i < n; ++i) {
            if (stalls[i] - lastPosition >= minDist) {
                count++;
                lastPosition = stalls[i]; // Update the position of the last placed cow
                
                if (count == cows) {
                    return true;
                }
            }
        }
        return false;
    }

public:
    int getMaximumMinDistance(std::vector<int>& stalls, int cows) {
        int n = stalls.size();
        
        // Step 1: Sort the stall locations
        std::sort(stalls.begin(), stalls.end());

        // Step 2: Define binary search boundaries
        int low = 1;                                  // Minimum possible distance
        int high = stalls[n - 1] - stalls[0];         // Maximum possible distance
        int ans = 1;

        // Step 3: Perform Binary Search on the distance range
        while (low <= high) {
            int mid = low + (high - low) / 2;

            if (canPlaceCows(stalls, n, cows, mid)) {
                ans = mid;          // Record the candidate distance
                low = mid + 1;      // Try to find a larger minimum distance
            } else {
                high = mid - 1;     // Reduce the distance limit
            }
        }

        return ans;
    }
};
```


#### Complexity Summary

- **Time Complexity:**
  - Sorting the array: $O(N \log N)$.
  - The binary search takes $\log(\text{max\_dist})$ iterations where $\text{max\_dist} = \text{stalls}[N-1] - \text{stalls}[0] \le 10^9$. $\log(10^9) \approx 30$.
  - Inside each step of the binary search, we iterate through the stalls array of size $N$, taking $O(N)$ time.
  - **Total Time Complexity:** $O(N \log N + N \log(\text{max\_dist}))$. For $N = 10^5$ and coordinates up to $10^9$, this executes around $3 \times 10^6$ operations, which is extremely fast.
- **Space Complexity:**
  - $O(1)$ auxiliary space if sorting is done in-place.


---

## Q10 Infinite Sorted Array

### 1. Problem Explanation

Given an infinite sorted array (an array of sorted integers with no defined size or length), find the index of a target element. If the target element is not present in the array, return `-1`.
Since the array is infinite, you cannot access its size, and trying to access an element out of bounds will lead to an error or return a special sentinel value (e.g., `2^31 - 1` or `INT_MAX`). 
We are given an interface/object `ArrayReader` which has a method `int get(int index)` that returns the element at the specified index.

### Input
- `reader`: An instance of `ArrayReader` containing the sorted sequence.
- `target`: The integer value we need to find.

### Output
- Return the 0-based index of the `target` if found, otherwise return `-1`.

### Constraints
- $1 \le \text{target} \le 10^9$
- The array elements are sorted in non-decreasing order.
- The index of the target (if present) is within bounds of a standard 32-bit integer.

### Observations
1. In a typical array, we perform binary search within the bounds of `[0, size - 1]`.
2. Since this array is infinite, we do not know the upper bound.
3. However, since the array is sorted, the elements keep increasing. We can search for the target by first defining a reasonable bounding box `[low, high]` and then performing a standard binary search within that box.


### 2. Brute Solution

### Algorithm
1. Start at index `i = 0`.
2. Loop indefinitely:
   - Get value at index `i`.
   - If value equals `target`, return `i`.
   - If value exceeds `target` or equals `INT_MAX`, return `-1`.
   - Increment `i` by 1.

### Complexity Analysis
- **Time Complexity:** $O(N)$ where $N$ is the index of the target.
- **Space Complexity:** $O(1)$ auxiliary space.

### Pros and Cons
- **Pros:** Easy to write, no overflow risks with index doubling.
- **Cons:** Extremely slow if the target is at index $10^6$ or larger.

### C++17 Mock Interface and Code
```cpp
#include <climits>

// Mock ArrayReader class for compilation purposes
class ArrayReader {
public:
    int get(int index) const {
        // Returns element at index, or INT_MAX if out of bounds
        // Actual implementation provided by the system
        return index; 
    }
};

class Solution {
public:
    int searchBruteForce(const ArrayReader& reader, int target) {
        int i = 0;
        while (true) {
            int val = reader.get(i);
            if (val == target) {
                return i;
            }
            if (val > target || val == INT_MAX) {
                break;
            }
            i++;
        }
        return -1;
    }
};
```


### 3. Optimal Solution & Complexities

### Algorithm
1. Initialize the boundaries: `low = 0` and `high = 1`.
2. Expand the boundaries while `reader.get(high) < target`:
   - Store `low` in a temporary variable if needed, or simply update:
   - `low = high + 1`.
   - `high = high * 2` (exponential expansion).
3. Once the loop terminates, the target must be within `[low, high]`.
4. Perform binary search in the range `[low, high]`:
   - While `low <= high`:
     - Calculate `mid = low + (high - low) / 2`.
     - Get value `val = reader.get(mid)`.
     - If `val == target`, return `mid`.
     - If `val > target`, search in the left half: `high = mid - 1`.
     - If `val < target`, search in the right half: `low = mid + 1`.
5. If the loop ends and we haven't found the target, return `-1`.

### Dry Run
Suppose the infinite array is `[3, 5, 7, 9, 10, 90, 100, 130, 140, 160, 170, ...]`, and `target = 10`.

**Step 1: Finding bounds**
- Start: `low = 0`, `high = 1`.
  - `reader.get(high)` = `reader.get(1)` = 5.
  - Since $5 < 10$, we expand bounds.
  - `low = 2`, `high = 1 * 2 = 2`.
- Check:
  - `reader.get(high)` = `reader.get(2)` = 7.
  - Since $7 < 10$, we expand bounds.
  - `low = 3`, `high = 2 * 2 = 4`.
- Check:
  - `reader.get(high)` = `reader.get(4)` = 10.
  - Since $10 \not< 10$, loop terminates.
- Final boundaries: `low = 3`, `high = 4`.

**Step 2: Binary Search in range `[3, 4]`**
- `low = 3`, `high = 4`.
- `mid = 3 + (4 - 3) / 2 = 3`.
- `reader.get(3)` = 9.
- Since $9 < 10$, we set `low = mid + 1 = 4`.
- Next iteration: `low = 4`, `high = 4`.
- `mid = 4 + (4 - 4) / 2 = 4`.
- `reader.get(4)` = 10.
- Since $10 == 10$, we return index `4`.

### C++17 Code
```cpp
#include <climits>

// Mock ArrayReader class for compilation purposes
class ArrayReader {
public:
    int get(int index) const {
        return index; // Placeholder
    }
};

class Solution {
public:
    int search(const ArrayReader& reader, int target) {
        // Step 1: Find the bounds where target can exist
        int low = 0;
        int high = 1;

        // Exponentially increase high bound until we overshoot target
        while (reader.get(high) < target) {
            low = high + 1;
            // Prevent integer overflow when doubling high
            if (high > INT_MAX / 2) {
                high = INT_MAX;
                break;
            }
            high = high * 2;
        }

        // Step 2: Perform standard binary search in [low, high]
        while (low <= high) {
            int mid = low + (high - low) / 2;
            int midValue = reader.get(mid);

            if (midValue == target) {
                return mid;
            } else if (midValue > target) {
                // If midValue is greater than target (or is INT_MAX),
                // search in the left half
                high = mid - 1;
            } else {
                // Search in the right half
                low = mid + 1;
            }
        }

        return -1;
    }
};
```


#### Complexity Summary

- **Time Complexity:**
  - **Finding Range:** The index of the high pointer is doubled at each step: $1, 2, 4, 8, \dots, 2^k$. If the target is at index $P$, the loop runs $k$ times where $2^k \ge P$, which means $k = O(\log P)$.
  - **Binary Search:** The binary search is performed on the range `[low, high]` where the length of the range is at most $2^{k-1}$. The binary search takes $O(\log(2^{k-1})) = O(k) = O(\log P)$ steps.
  - **Total Time Complexity:** $O(\log P)$ where $P$ is the position of the target in the infinite array.
- **Space Complexity:**
  - $O(1)$ auxiliary space.


---

## Q11 Kth Smallest Matrix

### 1. Problem Explanation

Given an $N \times N$ matrix where each of the rows and columns is sorted in non-decreasing order, find the $K$-th smallest element in the matrix.
Note that it is the $K$-th smallest element in the sorted order, not the $K$-th distinct element.

### Input
- `matrix`: A 2D grid of size $N \times N$ where both rows and columns are sorted.
- `K`: An integer representing the target ordinal statistics.

### Output
- Return an integer representing the $K$-th smallest element.

### Constraints
- $N = \text{matrix.length}$
- $N = \text{matrix}[i].\text{length}$
- $1 \le N \le 300$
- $-10^9 \le \text{matrix}[i][j] \le 10^9$
- $1 \le K \le N^2$

### Observations
1. The smallest element of the matrix is at `matrix[0][0]`.
2. The largest element of the matrix is at `matrix[N-1][N-1]`.
3. For any element `matrix[i][j]`, all elements to its left and above it are smaller or equal. All elements to its right and below it are larger or equal.
4. If we count the number of elements less than or equal to a target value $X$, this count is a monotonic function of $X$. Thus, we can apply Binary Search on the range of values `[matrix[0][0], matrix[N-1][N-1]]`.


### 2. Brute Solution

### Algorithm
1. Flatten the $N \times N$ matrix into a 1D vector of size $N^2$.
2. Sort the vector.
3. Return the element at index $K - 1$.

### Complexity Analysis
- **Time Complexity:** $O(N^2 \log(N^2)) = O(N^2 \log N)$ to flatten and sort.
- **Space Complexity:** $O(N^2)$ to store the flattened grid elements.

### C++17 Code
```cpp
#include <vector>
#include <algorithm>

class Solution {
public:
    int kthSmallestBrute(std::vector<std::vector<int>>& matrix, int k) {
        int n = matrix.size();
        std::vector<int> flattened;
        flattened.reserve(n * n);
        
        for (int i = 0; i < n; ++i) {
            for (int j = 0; j < n; ++j) {
                flattened.push_back(matrix[i][j]);
            }
        }
        
        std::sort(flattened.begin(), flattened.end());
        return flattened[k - 1];
    }
};
```


### 3. Optimal Solution & Complexities

### Algorithm
1. Initialize `low = matrix[0][0]` and `high = matrix[N-1][N-1]`.
2. While `low < high`:
   - Calculate `mid = low + (high - low) / 2`.
   - Count the number of elements $\le$ `mid` using the **Staircase Search**:
     - Start at `row = N - 1` and `col = 0`.
     - Initialize `count = 0`.
     - While `row >= 0` and `col < N`:
       - If `matrix[row][col] <= mid`:
         - All elements in column `col` from `0` to `row` are $\le$ `mid`.
         - Add `row + 1` to `count`.
         - Move right: `col++`.
       - Else:
         - Move up: `row--`.
   - If `count < K`, then `mid` is too small. Set `low = mid + 1`.
   - Else, set `high = mid`.
3. Return `low`.

### Dry Run
Matrix:
```
[ 1,  5,  9]
[10, 11, 13]
[12, 13, 15]
```
$N = 3, K = 8$.
`low = 1`, `high = 15`.

**Iteration 1:**
- `mid = 1 + (15 - 1) / 2 = 8`
- Count elements $\le 8$:
  - Start at `row = 2, col = 0`.
  - `matrix[2][0] = 12 > 8` $\rightarrow$ `row--` $\rightarrow$ `row = 1`.
  - `matrix[1][0] = 10 > 8` $\rightarrow$ `row--` $\rightarrow$ `row = 0`.
  - `matrix[0][0] = 1 <= 8` $\rightarrow$ `count += (0 + 1) = 1`, `col++` $\rightarrow$ `col = 1`.
  - `matrix[0][1] = 5 <= 8` $\rightarrow$ `count += (0 + 1) = 2`, `col++` $\rightarrow$ `col = 2`.
  - `matrix[0][2] = 9 > 8` $\rightarrow$ `row--` $\rightarrow$ `row = -1` (terminates).
  - Total count = 2.
- Since `count` ($2$) < $K$ ($8$), we set `low = mid + 1 = 9`.

**Iteration 2:**
- `low = 9`, `high = 15`.
- `mid = 9 + (15 - 9) / 2 = 12`
- Count elements $\le 12$:
  - Start `row = 2, col = 0`.
  - `matrix[2][0] = 12 <= 12` $\rightarrow$ `count += 3`, `col++` $\rightarrow$ `col = 1`.
  - `matrix[2][1] = 13 > 12` $\rightarrow$ `row--` $\rightarrow$ `row = 1`.
  - `matrix[1][1] = 11 <= 12` $\rightarrow$ `count += 2` (total 5), `col++` $\rightarrow$ `col = 2`.
  - `matrix[1][2] = 13 > 12` $\rightarrow$ `row--` $\rightarrow$ `row = 0`.
  - `matrix[0][2] = 9 <= 12` $\rightarrow$ `count += 1` (total 6), `col++` $\rightarrow$ `col = 3` (terminates).
  - Total count = 6.
- Since `count` ($6$) < $K$ ($8$), we set `low = mid + 1 = 13`.

**Iteration 3:**
- `low = 13`, `high = 15`.
- `mid = 13 + (15 - 13) / 2 = 14`
- Count elements $\le 14$:
  - Start `row = 2, col = 0`.
  - `matrix[2][0] = 12 <= 14` $\rightarrow$ `count += 3`, `col++` $\rightarrow$ `col = 1`.
  - `matrix[2][1] = 13 <= 14` $\rightarrow$ `count += 3` (total 6), `col++` $\rightarrow$ `col = 2`.
  - `matrix[2][2] = 15 > 14` $\rightarrow$ `row--` $\rightarrow$ `row = 1`.
  - `matrix[1][2] = 13 <= 14` $\rightarrow$ `count += 2` (total 8), `col++` $\rightarrow$ `col = 3` (terminates).
  - Total count = 8.
- Since `count` ($8$) $\ge K$ ($8$), we set `high = mid = 14`.

**Iteration 4:**
- `low = 13`, `high = 14`.
- `mid = 13 + (14 - 13) / 2 = 13`
- Count elements $\le 13$:
  - Start `row = 2, col = 0`.
  - `matrix[2][0] = 12 <= 13` $\rightarrow$ `count += 3`, `col++` $\rightarrow$ `col = 1`.
  - `matrix[2][1] = 13 <= 13` $\rightarrow$ `count += 3` (total 6), `col++` $\rightarrow$ `col = 2`.
  - `matrix[2][2] = 15 > 13` $\rightarrow$ `row--` $\rightarrow$ `row = 1`.
  - `matrix[1][2] = 13 <= 13` $\rightarrow$ `count += 2` (total 8), `col++` $\rightarrow$ `col = 3` (terminates).
  - Total count = 8.
- Since `count` ($8$) $\ge K$ ($8$), we set `high = mid = 13`.

**Loop Ends:**
- `low == high == 13`. Returns `13`.

### C++17 Code
```cpp
#include <vector>
#include <algorithm>

class Solution {
private:
    // Helper function to count elements less than or equal to 'mid'
    // in an N x N matrix where rows and columns are sorted.
    int countLessEqual(const std::vector<std::vector<int>>& matrix, int n, int mid) {
        int count = 0;
        int row = n - 1;
        int col = 0;

        // Staircase Search starting from the bottom-left corner
        while (row >= 0 && col < n) {
            if (matrix[row][col] <= mid) {
                // If matrix[row][col] <= mid, then all elements from 
                // index 0 to row in the current column are also <= mid.
                count += (row + 1);
                col++; // Move right to find larger values
            } else {
                row--; // Move up to find smaller values
            }
        }
        return count;
    }

public:
    int kthSmallest(const std::vector<std::vector<int>>& matrix, int k) {
        int n = matrix.size();
        
        int low = matrix[0][0];
        int high = matrix[n - 1][n - 1];
        int ans = low;

        while (low <= high) {
            // Using long long for mid to prevent overflow if values are close to INT boundaries
            long long mid = low + (high - low) / 2;

            if (countLessEqual(matrix, n, mid) >= k) {
                ans = mid;          // mid is a candidate answer
                high = mid - 1;     // Try to find a smaller candidate
            } else {
                low = mid + 1;      // mid is too small, look in right half
            }
        }

        return ans;
    }
};
```


#### Complexity Summary

- **Time Complexity:**
  - The range of values is $R = \text{matrix}[N-1][N-1] - \text{matrix}[0][0]$. The binary search takes $O(\log R)$ iterations. For integer constraints, this is at most $\approx 32$ iterations.
  - In each iteration, we run the staircase search. In the worst case, the path from bottom-left to top-right takes at most $2N$ steps. So, `countLessEqual` runs in $O(N)$ time.
  - **Total Time Complexity:** $O(N \log(\text{max\_val} - \text{min\_val}))$. For $N = 300$, this is extremely fast, requiring only about $300 \times 32 \approx 9600$ operations.
- **Space Complexity:**
  - $O(1)$ auxiliary space.


---

## Q12 Nearly Sorted Array

### 1. Problem Explanation

Given a sorted array of $N$ integers where each element is at most 1 position away from its sorted position. That is, the element at index $i$ in the sorted array could be present at index $i-1$, $i$, or $i+1$ in the given nearly sorted array. 
Find the index of a target element in this nearly sorted array. If the target is not present, return `-1`.

### Input
- `arr`: An array of $N$ integers representing the nearly sorted array.
- `target`: The value to search for.

### Output
- Return the 0-based index of the target if found, otherwise return `-1`.

### Constraints
- $1 \le N \le 10^5$
- $-10^9 \le \text{arr}[i] \le 10^9$
- The array elements are distinct.

### Observations
1. In a fully sorted array, the element at index `mid` divides the array. 
2. In a nearly sorted array, the element that *should* be at `mid` could be at index `mid - 1`, `mid`, or `mid + 1`.
3. Thus, during binary search, we can inspect three indices (`mid - 1`, `mid`, `mid + 1`) to see if any of them contains the target.
4. If the target is not found at those three positions, we can determine which half of the array to search next, similar to standard binary search, but adjusting our next step to avoid re-checking elements.


### 2. Brute Solution

### Algorithm
1. Traverse the array from index $0$ to $N - 1$.
2. If `arr[i] == target`, return `i`.
3. If we loop through the entire array without finding the target, return `-1`.

### Complexity Analysis
- **Time Complexity:** $O(N)$ because we might inspect every element in the worst case.
- **Space Complexity:** $O(1)$ auxiliary space.

### Pros and Cons
- **Pros:** Extremely simple to implement; works even if the array is completely unsorted.
- **Cons:** Inefficient; ignores the nearly sorted property.

### C++17 Code
```cpp
#include <vector>

class Solution {
public:
    int searchBruteForce(const std::vector<int>& arr, int target) {
        int n = arr.size();
        for (int i = 0; i < n; ++i) {
            if (arr[i] == target) {
                return i;
            }
        }
        return -1;
    }
};
```


### 3. Optimal Solution & Complexities

### Algorithm
1. Initialize `low = 0` and `high = N - 1`.
2. While `low <= high`:
   - Calculate `mid = low + (high - low) / 2`.
   - Check if `arr[mid] == target`. If so, return `mid`.
   - Check if `mid - 1 >= low` and `arr[mid - 1] == target`. If so, return `mid - 1`.
   - Check if `mid + 1 <= high` and `arr[mid + 1] == target`. If so, return `mid + 1`.
   - If the target is not found:
     - If `target < arr[mid]`, search the left side: `high = mid - 2`.
     - Else, search the right side: `low = mid + 2`.
3. If the loop ends and the target is not found, return `-1`.

### Dry Run
`arr` = `[10, 3, 40, 20, 50, 80, 70]`, `target = 40`.
$N = 7$.
`low = 0`, `high = 6`.

**Iteration 1:**
- `mid = 0 + (6 - 0) / 2 = 3`
- Check `mid` indices:
  - `arr[mid] = arr[3] = 20` (not equal to 40)
  - `mid - 1 = 2` $\rightarrow$ `arr[2] = 40` (equal to 40!)
- Match found! Return index `2`.

Let's try another dry run where `target = 90` (not in array).
`low = 0`, `high = 6`.

**Iteration 1:**
- `mid = 3`.
- `arr[3] = 20`, `arr[2] = 40`, `arr[4] = 50`. No match.
- Since `target (90) > arr[mid] (20)`, we set `low = mid + 2 = 5`.

**Iteration 2:**
- `low = 5`, `high = 6`.
- `mid = 5 + (6 - 5) / 2 = 5`.
- Check `mid` indices:
  - `arr[5] = 80` (not equal to 90)
  - `mid - 1 = 4` $\rightarrow$ outside `[low, high]` (not checked)
  - `mid + 1 = 6` $\rightarrow$ `arr[6] = 70` (not equal to 90)
- Since `target (90) > arr[mid] (80)`, we set `low = mid + 2 = 7`.

**Loop Ends:**
- `low` (7) > `high` (6). Loop terminates.
- Return `-1`.

### C++17 Code
```cpp
#include <vector>

class Solution {
public:
    int searchNearlySorted(const std::vector<int>& arr, int target) {
        int n = arr.size();
        int low = 0;
        int high = n - 1;

        while (low <= high) {
            int mid = low + (high - low) / 2;

            // 1. Check mid
            if (arr[mid] == target) {
                return mid;
            }

            // 2. Check mid - 1 (ensure within search space bounds)
            if (mid - 1 >= low && arr[mid - 1] == target) {
                return mid - 1;
            }

            // 3. Check mid + 1 (ensure within search space bounds)
            if (mid + 1 <= high && arr[mid + 1] == target) {
                return mid + 1;
            }

            // 4. Decide search direction.
            // Since we checked mid-1, mid, and mid+1, we can skip them
            if (target < arr[mid]) {
                high = mid - 2;
            } else {
                low = mid + 2;
            }
        }

        return -1;
    }
};
```


#### Complexity Summary

- **Time Complexity:**
  - In each step of the binary search, we perform a constant number of comparisons ($O(1)$) and reduce the search space size by half (from $N$ to $N/2 - 1$).
  - **Total Time Complexity:** $O(\log N)$ in the worst case, same as standard binary search.
- **Space Complexity:**
  - $O(1)$ auxiliary space.


---
