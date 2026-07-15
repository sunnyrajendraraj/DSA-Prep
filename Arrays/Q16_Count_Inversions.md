# Count Inversions in an Array

---

## 1. Problem Statement

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

---

## 2. Interview Clarification

1. **Can the array contain duplicate elements?**
   - *Answer:* Yes. However, if `arr[i] == arr[j]`, they do **not** form an inversion since the requirement is strictly `arr[i] > arr[j]`.
2. **What if the array is empty or has only one element?**
   - *Answer:* The inversion count is `0`.
3. **Can the total number of inversions overflow a 32-bit signed integer?**
   - *Answer:* Yes. With $N = 5 \times 10^5$, the maximum number of inversions is $\approx \frac{5 \times 10^5 \times 5 \times 10^5}{2} = 1.25 \times 10^{11}$, which far exceeds $2 \times 10^9$. We must use a 64-bit integer (`long long` in C++) to store and return the count.

---

## 3. Thinking Process

1. **First Instinct (Brute Force):**
   - We can check all possible pairs `(i, j)` where $i < j$.
   - Using two nested loops, the outer loop selects $i$ and the inner loop selects $j$ from $i+1$ to $N-1$. If `arr[i] > arr[j]`, we increment our inversion count.
   - Time Complexity: $O(N^2)$, Space Complexity: $O(1)$. This will time out for $N = 5 \times 10^5$.

2. **Optimizing via Divide and Conquer (Merge Sort):**
   - The bottleneck in brute force is comparing every element with every other element. Can we count inversions during a sorting process?
   - In **Merge Sort**, we recursively split the array into two halves: Left and Right.
   - Suppose we count inversions in the Left half, count inversions in the Right half, and then count **cross-inversions** (where `i` is in the Left half and `j` is in the Right half).
     $$\text{Total Inversions} = \text{Inversions(Left)} + \text{Inversions(Right)} + \text{Inversions(Cross)}$$
   - Sorting the subarrays doesn't change the count of cross-inversions, but it makes counting them much easier!
   - Let's assume the Left subarray and Right subarray are already sorted:
     - Left: `[3, 5, 8]`, Right: `[2, 4, 6]`
     - We merge them using two pointers: `i` for Left and `j` for Right.
     - Compare `Left[i]` and `Right[j]`:
       - If `Left[i] <= Right[j]`, there is no inversion. We copy `Left[i]` to our temp array and increment `i`.
       - If `Left[i] > Right[j]`, then since Left is sorted, all remaining elements in the Left subarray (`Left[i]`, `Left[i+1]`, ..., `Left[mid]`) must also be strictly greater than `Right[j]`.
       - All these elements form inversions with `Right[j]`.
       - The number of elements from `i` to `mid` is `mid - i + 1`. We add this value directly to our inversion count, copy `Right[j]` to temp, and increment `j`.
   - This allows us to count all cross-inversions in $O(N)$ time during the merge step, leading to a total time complexity of $O(N \log N)$.

---

## 4. Pattern Recognition

- **Pattern:** Divide and Conquer / Modified Merge Sort.
- **Why this topic?** The problem involves counting pairs that violate a sorted order. Merge sort naturally identifies when a element from the right partition jumps ahead of elements in the left partition, which corresponds exactly to an inversion.

---

## 5. Brute Force Solution

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

---

## 6. Better Solution

> [!NOTE]
> Since this problem transitions directly from $O(N^2)$ brute force to $O(N \log N)$ optimal using modified merge sort, there is no intermediate "Better" solution. 
> Alternatively, one could use a **Fenwick Tree (Binary Indexed Tree)** or a **Segment Tree** which also runs in $O(N \log N)$ time and $O(N)$ space. However, modified Merge Sort is the standard optimal approach expected in interviews.

---

## 7. Optimal Solution

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

---

## 8. Code Walkthrough

1. **Helper Array Initialization:**
   ```cpp
   std::vector<int> temp(n);
   return mergeSortAndCount(arr, temp, 0, n - 1);
   ```
   We allocate a temporary array once to avoid repeated allocations during recursive steps.

2. **Divide Step:**
   ```cpp
   int mid = left + (right - left) / 2;
   invCount += mergeSortAndCount(arr, temp, left, mid);
   invCount += mergeSortAndCount(arr, temp, mid + 1, right);
   ```
   We divide the array into left and right halves and recursively count inversions within those halves.

3. **Merge and Count Step:**
   ```cpp
   if (arr[i] <= arr[j]) {
       temp[k++] = arr[i++];
   } else {
       temp[k++] = arr[j++];
       invCount += (mid - i + 1);
   }
   ```
   If the element in the left half `arr[i]` is greater than the element in the right half `arr[j]`, then all elements from `i` to `mid` form an inversion with `arr[j]`. We add `mid - i + 1` to `invCount`.

---

## 9. Dry Run

Consider the merge step for `Left = [3, 5]` and `Right = [2, 4]`, where `left = 0, mid = 1, right = 3`. 
Initially `i = 0`, `j = 2`, `invCount = 0`.

| Step | `i` | `j` | `arr[i]` | `arr[j]` | Comparison | Action | Added Inversions | Cumulative `invCount` |
|:---:|:---:|:---:|:---:|:---:|:---|:---|:---:|:---:|
| 1 | 0 | 2 | 3 | 2 | $3 > 2$ (Inversion) | Copy `arr[j]` (2) to temp; `j++` | `mid - i + 1` = $1 - 0 + 1 = 2$ | 2 |
| 2 | 0 | 3 | 3 | 4 | $3 \le 4$ (No Inversion) | Copy `arr[i]` (3) to temp; `i++` | 0 | 2 |
| 3 | 1 | 3 | 5 | 4 | $5 > 4$ (Inversion) | Copy `arr[j]` (4) to temp; `j++` (Right empty) | `mid - i + 1` = $1 - 1 + 1 = 1$ | 3 |
| 4 | 1 | - | 5 | - | Left remaining | Copy `arr[i]` (5) to temp; `i++` | 0 | 3 |

Total inversions counted in this merge step = 3.

---

## 10. Complexity Analysis

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

## 11. Common Mistakes

1. **Incorrect Inversion Count Calculation:**
   - Writing `invCount += (mid - i)` instead of `invCount += (mid - i + 1)`. Since indices are 0-based, the number of elements from index `i` to `mid` (inclusive) is `mid - i + 1`.
2. **Standard Integer Overflow:**
   - Accumulating the count in an `int` variable instead of `long long`. The count can reach $\approx 10^{11}$ for $N = 5 \times 10^5$.
3. **Handling Equality Incorrectly:**
   - Counting `arr[i] == arr[j]` as an inversion. The code must have `arr[i] <= arr[j]` in the primary check to avoid this.

---

## 12. Edge Cases

| Edge Case | Input Example | Expected Output | Why it Matters |
|:---|:---|:---:|:---|
| **Already Sorted** | `[1, 2, 3, 4]` | `0` | No elements should trigger the inversion count block. |
| **Reverse Sorted** | `[4, 3, 2, 1]` | `6` | Max possible inversions: $\frac{4 \times 3}{2} = 6$. |
| **All Duplicates** | `[2, 2, 2, 2]` | `0` | Equal elements do not form inversions. |
| **Single Element** | `[5]` | `0` | Base case must return 0 immediately. |

---

## 13. Interview Explanation

**Speaker Script:**
> "To count inversions in an array efficiently, we can modify the Merge Sort algorithm. 
> An inversion is defined as a pair of indices $i < j$ such that $arr[i] > arr[j]$. 
> A brute-force double loop compares all pairs, which takes $O(N^2)$ time. 
> By using Merge Sort, we divide the array recursively. During the merge step, we have two sorted subarrays. Since they are sorted, if we find an element in the left subarray $arr[i]$ that is strictly greater than an element in the right subarray $arr[j]$, then all subsequent elements in the left subarray from index $i$ to $mid$ must also be strictly greater than $arr[j]$. 
> Thus, we can count all these inversions in $O(1)$ time by adding $mid - i + 1$ to our count. This approach allows us to find the total count of inversions in $O(N \log N)$ time and $O(N)$ auxiliary space."

---

## 14. Follow-up Questions

1. **Can we solve this problem with $O(1)$ auxiliary space?**
   - *Answer:* No, comparison-based sorting and inversion counting generally require $O(N)$ space. Even in-place merge sort is complex and has high constant factor overhead.
2. **How would you count inversions using a Fenwick Tree?**
   - *Answer:* We can coordinate-compress the array elements. Then, traversing the array from right to left, we query the Fenwick Tree for the count of elements smaller than the current element (which gives the inversion count for the current element) and then insert the current element into the tree. This also takes $O(N \log N)$ time.

---

## 15. Variations

1. **Reverse Pairs (LeetCode 493):** Count pairs $i < j$ where $arr[i] > 2 \times arr[j]$.
2. **Count of Smaller Numbers After Self (LeetCode 315):** Return an array representing the number of smaller elements to the right of each element (uses merge sort tracking original indices).

---

## 16. Revision Notes

- **Inversion formula:** `invCount += mid - i + 1` (occurs when `arr[i] > arr[j]`).
- **Data Type:** Always use `long long` for counting variable.
- **Merge Step:** Make sure to copy elements back to the original array after merging.

---

## 17. Memorization Version

```cpp
// Time: O(N log N), Space: O(N)
long long merge(vector<int>& arr, vector<int>& temp, int left, int mid, int right) {
    int i = left, j = mid + 1, k = left; long long count = 0;
    while (i <= mid && j <= right) {
        if (arr[i] <= arr[j]) temp[k++] = arr[i++];
        else { temp[k++] = arr[j++]; count += (mid - i + 1); }
    }
    while (i <= mid) temp[k++] = arr[i++];
    while (j <= right) temp[k++] = arr[j++];
    for (i = left; i <= right; ++i) arr[i] = temp[i];
    return count;
}
long long solve(vector<int>& arr, vector<int>& temp, int left, int right) {
    if (left >= right) return 0;
    int mid = left + (right - left) / 2;
    return solve(arr, temp, left, mid) + solve(arr, temp, mid + 1, right) + merge(arr, temp, left, mid, right);
}
```
