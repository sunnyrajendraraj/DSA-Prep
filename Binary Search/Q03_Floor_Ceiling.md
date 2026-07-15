# Floor and Ceiling in a Sorted Array

---

## 1. Problem Statement

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

---

## 2. Interview Clarification

Before coding, clarify these details:
1. **Should we return the index or the value?** (This problem asks for the values. Clarify if target value can be `-1` itself, in which case returning `-1` for 'not found' could be ambiguous. In this version, assume elements are positive or `-1` represents non-existence clearly).
2. **Is the array sorted in ascending order?** (Yes).
3. **Can the array contain duplicates?** (Yes, duplicates are allowed. The definitions of floor and ceiling still hold).
4. **What if the target is smaller than the smallest element?** (Ceiling is the first element, floor does not exist $\rightarrow$ return `-1`).
5. **What if the target is larger than the largest element?** (Floor is the last element, ceiling does not exist $\rightarrow$ return `-1`).

---

## 3. Thinking Process

### First Instinct (Brute Force)
We can scan the array from left to right:
- For **floor**: Keep track of the maximum element we have seen so far that is $\le x$. Since the array is sorted, this will simply be the last element we encounter that is $\le x$.
- For **ceiling**: The first element we encounter that is $\ge x$ will be the ceiling.
This linear search will take $O(n)$ time.

### Optimal Insight (Binary Search)
Since the array is sorted, we can use binary search to find the floor and ceiling independently or simultaneously:

#### Finding Floor:
- We initialize `floor_val = -1`.
- Binary search loop with `low` and `high` pointers:
  - If `nums[mid] == x`, then `nums[mid]` is the perfect floor. We can set `floor_val = nums[mid]` and terminate (since no element can be closer to `x` than `x` itself).
  - If `nums[mid] < x`, then `nums[mid]` is a valid candidate for floor (since it is $\le x$). Since we want the *largest* possible floor, we record `nums[mid]` and try to find a larger one by searching to the right: `low = mid + 1`.
  - If `nums[mid] > x`, then `nums[mid]` is too large to be the floor. We search to the left: `high = mid - 1`.

#### Finding Ceiling:
- We initialize `ceiling_val = -1`.
- Binary search loop with `low` and `high` pointers:
  - If `nums[mid] == x`, then `nums[mid]` is the perfect ceiling. We set `ceiling_val = nums[mid]` and terminate.
  - If `nums[mid] > x`, then `nums[mid]` is a valid candidate for ceiling (since it is $\ge x$). Since we want the *smallest* possible ceiling, we record `nums[mid]` and try to find a smaller one by searching to the left: `high = mid - 1`.
  - If `nums[mid] < x`, then `nums[mid]` is too small to be the ceiling. We search to the right: `low = mid + 1`.

---

## 4. Pattern Recognition

This problem belongs to the **Binary Search Boundary** pattern.
- **Why this topic?** We are looking for bounds (upper/lower limits) on a sorted domain.
- **Similar concept:** Finding `std::lower_bound` in C++ corresponds to finding the index of the ceiling.

---

## 5. Brute Force Solution

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

---

## 6. Better Solution

For this problem, there is no meaningful intermediate solution between the $O(n)$ brute force scan and the optimal $O(\log n)$ binary search.

---

## 7. Optimal Solution

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

---

## 8. Code Walkthrough

### `findFloor` walkthrough:
- **`floor_val` initialization:** Set to `-1`. If no element in the array is $\le x$, the function returns `-1`.
- **`nums[mid] < x` branch:** The element `nums[mid]` is a valid floor. We save it in `floor_val`. Because we want to maximize the floor, we shift our pointer to `mid + 1` to search for a larger valid element.
- **`nums[mid] > x` branch:** The element is too large. We search left (`high = mid - 1`).

### `findCeiling` walkthrough:
- **`ceiling_val` initialization:** Set to `-1`. If no element is $\ge x$, we return `-1`.
- **`nums[mid] > x` branch:** The element is a valid ceiling. We save it in `ceiling_val`. Because we want to minimize the ceiling, we shift our pointer to `mid - 1` to search for a smaller valid element.
- **`nums[mid] < x` branch:** The element is too small. We search right (`low = mid + 1`).

---

## 9. Dry Run

Let's dry run `nums = [2, 3, 5, 9]`, `x = 1`:

**Floor Search:**
- `low = 0`, `high = 3`, `floor_val = -1`
- `mid = 1`, `nums[mid] = 3 > 1` $\rightarrow$ `high = 0`
- `low = 0`, `high = 0`, `mid = 0`, `nums[mid] = 2 > 1` $\rightarrow$ `high = -1`
- Loop terminates. Returns `-1`.

**Ceiling Search:**
- `low = 0`, `high = 3`, `ceiling_val = -1`
- `mid = 1`, `nums[mid] = 3 > 1` $\rightarrow$ `ceiling_val = 3`, `high = 0`
- `low = 0`, `high = 0`, `mid = 0`, `nums[mid] = 2 > 1` $\rightarrow$ `ceiling_val = 2`, `high = -1`
- Loop terminates. Returns `2`.

Result: `{-1, 2}`.

---

## 10. Complexity Analysis

| Scenario | Time Complexity | Space Complexity | Reason |
| :--- | :--- | :--- | :--- |
| **Best Case** | $O(1)$ | $O(1)$ | Target $x$ exists at the exact midpoint for both functions. |
| **Average Case** | $O(\log n)$ | $O(1)$ | Standard binary search space halving. |
| **Worst Case** | $O(\log n)$ | $O(1)$ | Target $x$ is not present, requiring full tree traversal. |

---

## 11. Common Mistakes

1. **Incorrect Update of Candidates:**
   - Forgetting to save `nums[mid]` before moving the pointers. For example, doing `high = mid - 1` without assigning `ceiling_val = nums[mid]`.
2. **Confusing Floor and Ceiling Conditions:**
   - Updating `floor_val` when `nums[mid] > x` or updating `ceiling_val` when `nums[mid] < x`.
   - *Mnemonic:* **F**loor is below (smaller $\le$), **C**eiling is above (greater $\ge$).

---

## 12. Edge Cases

| Edge Case | Input Example | Expected Output | Rationale |
| :--- | :--- | :--- | :--- |
| **Target smaller than smallest element** | `nums = [3, 5, 7]`, `x = 1` | `{-1, 3}` | Floor doesn't exist, ceiling is the first element. |
| **Target larger than largest element** | `nums = [3, 5, 7]`, `x = 10` | `{7, -1}` | Floor is the last element, ceiling doesn't exist. |
| **Target matches element** | `nums = [3, 5, 7]`, `x = 5` | `{5, 5}` | Floor and ceiling are both equal to the target. |
| **Array has duplicates** | `nums = [2, 2, 5, 5]`, `x = 2` | `{2, 2}` | Correctly handles duplicates. |

---

## 13. Interview Explanation

> "To find the floor and ceiling of a target `x` in a sorted array, we can perform two separate binary searches. 
> To find the floor (the largest element $\le x$), we search the array by checking the midpoint. If the midpoint value is less than or equal to $x$, it's a potential floor. We save it and look to the right to see if there is an even larger element that still satisfies the condition. 
> To find the ceiling (the smallest element $\ge x$), if the midpoint value is greater than or equal to $x$, we save it and look to the left to find a smaller element that is still $\ge x$. 
> This approach splits the search range in half at each iteration, giving us an optimal time complexity of $O(\log n)$ and constant space complexity of $O(1)$."

---

## 14. Follow-up Questions

1. **Can we implement this using C++ Standard Library functions?**
   - *Answer:* Yes. `std::lower_bound` returns an iterator to the first element $\ge x$. This element is the ceiling. The element immediately before it (if it exists) is the floor.
   - Example code using STL:
     ```cpp
     auto it = std::lower_bound(nums.begin(), nums.end(), x);
     int ceiling = (it != nums.end()) ? *it : -1;
     int floor = (it != nums.begin()) ? *(it - 1) : -1;
     // If x is present in the array, the floor is also *it (since floor is <= x)
     if (it != nums.end() && *it == x) floor = x;
     ```
2. **What if the array was sorted in descending order?**
   - *Answer:* The logic reverses. When `nums[mid] < x`, it is still a potential floor, but since the array is descending, larger elements lie to the *left*, so we would update `high = mid - 1`.

---

## 15. Variations

1. **Find Smallest Letter Greater Than Target (LeetCode 744):** Find the smallest character in a sorted array that is strictly larger than a target character. This is equivalent to finding the ceiling but with strict inequality ($>$) and circular wrap-around.
2. **Search Insertion Position (LeetCode 35):** Find the index where target should be inserted. This is identical to finding the index of the ceiling.

---

## 16. Revision Notes

- **Floor:** Candidate if `nums[mid] <= x`, search right (`low = mid + 1`).
- **Ceiling:** Candidate if `nums[mid] >= x`, search left (`high = mid - 1`).
- **STL equivalence:** `lower_bound` corresponds to ceiling.

---

## 17. Memorization Version

```cpp
pair<int, int> getFloorAndCeiling(vector<int>& nums, int x) {
    int f = -1, c = -1;
    // Floor
    int l = 0, h = nums.size() - 1;
    while (l <= h) {
        int m = l + (h - l) / 2;
        if (nums[m] <= x) { f = nums[m]; l = m + 1; }
        else h = m - 1;
    }
    // Ceiling
    l = 0; h = nums.size() - 1;
    while (l <= h) {
        int m = l + (h - l) / 2;
        if (nums[m] >= x) { c = nums[m]; h = m - 1; }
        else l = m + 1;
    }
    return {f, c};
}
```
*Time:* $O(\log n)$, *Space:* $O(1)$. Floor saves on $\le$ and goes right. Ceiling saves on $\ge$ and goes left.
