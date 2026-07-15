# Median in Row-wise Sorted Matrix

## 1. Problem Statement
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

---

## 2. Interview Clarification
Before coding, a candidate should clarify:
1. **Q:** Can the matrix have duplicate elements?
   - **A:** Yes, duplicates are allowed.
2. **Q:** Is $R \times C$ guaranteed to be odd?
   - **A:** Yes, in standard versions of this problem, it is guaranteed. If it is even, we might need to take the average of the two middle elements, which is a standard follow-up.
3. **Q:** What are the range of values in the matrix?
   - **A:** Knowing the bounds (e.g., $1$ to $10^9$) helps determine the search space for Binary Search.
4. **Q:** Are the columns sorted?
   - **A:** No, only the individual rows are sorted. The column elements do not have any sorted relationship.

---

## 3. Thinking Process
### First Instinct
If we copy all elements of the matrix into a 1D array, sort that array, and return the middle element at index $(R \times C) / 2$, we would get the correct median. 
- This approach requires $O(R \times C)$ extra space and $O((R \times C) \log(R \times C))$ time.
- However, this completely ignores the fact that each row is already sorted!

### Can we do better?
Since the rows are sorted, we can look at this as a problem of finding the element $X$ such that the count of elements less than or equal to $X$ is at least $(R \times C) / 2 + 1$. 
Let's define $K = (R \times C) / 2 + 1$. We want to find the smallest number $X$ in the range $[\text{min\_val}, \text{max\_val}]$ which has at least $K$ elements less than or equal to it.

Since we are searching for a value $X$ in a range of values $[\text{min\_val}, \text{max\_val}]$, and the count of elements $\le X$ is a monotonically increasing function with respect to $X$, we can use **Binary Search on the Answer**.

### Optimal Insight
1. Identify the range of possible answers:
   - The minimum possible value `low` is the minimum element in the first column: $\min(\text{matrix}[i][0])$ for all $0 \le i < R$.
   - The maximum possible value `high` is the maximum element in the last column: $\max(\text{matrix}[i][C - 1])$ for all $0 \le i < R$.
2. Perform Binary Search in the range `[low, high]`:
   - Calculate `mid = low + (high - low) / 2`.
   - Count how many elements in the matrix are less than or equal to `mid`.
   - Since each row is sorted, we can find the count of elements $\le$ `mid` in each row in $O(\log C)$ time using `std::upper_bound`.
   - Total count of elements $\le$ `mid` across all rows is $\sum_{i=0}^{R-1} \text{count\_less\_equal}(\text{row}[i], \text{mid})$.
   - If the total count is less than $K = (R \times C) / 2 + 1$, then our candidate `mid` is too small. We must search in the right half: `low = mid + 1`.
   - If the total count is greater than or equal to $K$, then `mid` could be the median or a value larger than the median. We search in the left half: `high = mid`.
3. When `low == high`, we have found the median.

---

## 4. Pattern Recognition
This problem belongs to the **Binary Search on Answer / Value Range** pattern.
- Why? We are not binary searching on indices of an array. Instead, we are binary searching on the range of possible values of the elements.
- The property that allows Binary Search is **Monotonicity**: as the value $X$ increases, the count of elements $\le X$ in the matrix can only increase or remain the same (monotonically non-decreasing).

---

## 5. Brute Force Solution
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

---

## 6. Better Solution
*Note: For this specific problem, there is no intermediate "Better" solution that sits between the brute force (flattening and sorting) and the optimal Binary Search on value range. We jump directly from Brute Force to the Optimal Binary Search.*

---

## 7. Optimal Solution
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

---

## 8. Code Walkthrough
- `int low = matrix[0][0]; int high = matrix[0][c-1];`: Initialize binary search range. Since each row is sorted, the minimum is in the first column and the maximum is in the last column. We loop through all rows to find the exact minimum and maximum of the entire matrix.
- `int k = (r * c) / 2 + 1;`: Sets target number of elements. The median is the element at $K$-th position when elements are sorted.
- `while (low < high)`: Standard binary search loop that terminates when the search space reduces to a single element.
- `int mid = low + (high - low) / 2;`: Avoids integer overflow during calculation.
- `countLessEqual(matrix, mid)`: Calls a helper function that loops through each row and uses `std::upper_bound` to count elements $\le$ `mid` in $O(R \log C)$ time.
- `low = mid + 1`: Eliminates values $\le$ `mid` from consideration since they don't cover enough elements.
- `high = mid`: Keeps `mid` as a candidate since it covers at least `k` elements, but searches left for any smaller candidate.

---

## 9. Dry Run
Let's trace `findMedian` for the matrix:
```
[1, 3, 6]
[2, 6, 9]
[3, 6, 10]
```
$R = 3, C = 3$, Total Elements = 9, $K = 5$.
`low` = $\min(1, 2, 3) = 1$, `high` = $\max(6, 9, 10) = 10$.

| Step | low | high | mid | Row Counts (Row 0, 1, 2) | Total Count | Count < K ($5$)? | Action |
|---|---|---|---|---|---|---|---|
| 1 | 1 | 10 | 5 | $2$ (1,3), $1$ (2), $1$ (3) | 4 | Yes | `low = mid + 1` $\rightarrow$ 6 |
| 2 | 6 | 10 | 8 | $3$ (all), $2$ (2,6), $2$ (3,6) | 7 | No | `high = mid` $\rightarrow$ 8 |
| 3 | 6 | 8 | 7 | $3$ (all), $2$ (2,6), $2$ (3,6) | 7 | No | `high = mid` $\rightarrow$ 7 |
| 4 | 6 | 7 | 6 | $3$ (all), $2$ (2,6), $2$ (3,6) | 7 | No | `high = mid` $\rightarrow$ 6 |
| 5 | 6 | 6 | - | Loop terminates (`low == high`) | - | - | Return `6` |

---

## 10. Complexity Analysis
- **Time Complexity:**
  - Finding `low` and `high` takes $O(R)$ time.
  - The binary search runs $\log(\text{max\_val} - \text{min\_val})$ times.
  - In each step, we query `std::upper_bound` for each row, taking $O(R \log C)$ time.
  - **Total Time Complexity:** $O(R \log C \cdot \log(\text{max\_val} - \text{min\_val}))$. For $1 \le \text{matrix}[i][j] \le 10^9$, this is roughly $30 \cdot R \log C$.
- **Space Complexity:**
  - $O(1)$ auxiliary space since we only store a few variables.

---

## 11. Common Mistakes
1. **Wrong low/high bounds:** Candidates often set `low = matrix[0][0]` and `high = matrix[R-1][C-1]` without checking other rows. The matrix as a whole is not sorted in a 1D spiral or row-major sorted order. Only individual rows are sorted. Thus, the minimum element can be in any row's first column.
2. **Off-by-one in count target:** Forgetting that $K$ must be $(R \times C) / 2 + 1$ (1-based index) and using $(R \times C) / 2$ instead.
3. **Using lower_bound instead of upper_bound:** `lower_bound` counts elements strictly less than `mid`. We need elements less than or equal to `mid`, which is computed using `upper_bound`.
4. **Infinite Loop:** Not handling search range reduction properly. When `count < k`, `low = mid + 1` pushes the boundary forward, ensuring termination.

---

## 12. Edge Cases
| Edge Case | Example | Why it matters | Solution |
|---|---|---|---|
| Single element matrix | `[[5]]` | Range is `[5, 5]`. Loop shouldn't execute. | `low = 5`, `high = 5`. Loop condition `low < high` prevents execution, returns 5. |
| Duplicate values | `[[2, 2, 2], [2, 2, 2], [2, 2, 2]]` | Upper bound will return count of all elements. | Handled correctly by `std::upper_bound` and standard binary search logic. |
| Negative numbers | `[[-10, -5, -2], [-8, -2, 0], [-9, -1, 1]]` | Binary search range handles negative values. | `mid` calculated using `low + (high - low) / 2` avoids overflow/underflow. |
| Large value range | Elements up to $10^9$ | Risk of integer overflow if adding `low + high`. | Use `low + (high - low) / 2` to prevent overflow. |

---

## 13. Interview Explanation
"To find the median of a row-wise sorted matrix in an optimal way, we can avoid sorting or flattening the matrix by using Binary Search on the value range. 
Since each row is sorted, the minimum element of the matrix must be in the first column, and the maximum element must be in the last column. We can find the exact minimum `low` and maximum `high` in $O(R)$ time. 
Then, we perform binary search on the range `[low, high]`. For each candidate value `mid`, we count how many elements in the matrix are less than or equal to `mid`. We do this efficiently by calling `std::upper_bound` on each of the $R$ rows, which takes $O(R \log C)$ time.
If the total count is less than $(R \times C) / 2 + 1$, it means the median must be strictly greater than `mid`, so we adjust our range to `[mid + 1, high]`. Otherwise, `mid` is a potential median, and we look for smaller candidates in `[low, mid]`. This reduces the search space by half each time, leading to a highly efficient time complexity of $O(R \log C \cdot \log(\text{max\_val} - \text{min\_val}))$ and using $O(1)$ extra space."

---

## 14. Follow-up Questions
1. **Q: What if $R \times C$ is even?**
   - **A:** If it is even, there is no single middle element. We would search for the $K$-th and $(K+1)$-th elements where $K = (R \times C) / 2$, and return their average.
2. **Q: Can we optimize if columns are also sorted?**
   - **A:** If the columns are also sorted (like a Young Tableau), we can count elements less than or equal to `mid` in $O(R + C)$ time using a staircase search from top-right to bottom-left, instead of $O(R \log C)$.

---

## 15. Variations
1. **K-th Smallest Element in a Sorted Matrix:**
   - Instead of the median (which is the $((R \times C) / 2 + 1)$-th element), find the $K$-th smallest element. The exact same logic applies, just replace $K$ with the given parameter.
2. **K-th Smallest Element in a Multiplication Table:**
   - A grid of size $M \times N$ where `grid[i][j] = i * j`. Find the $K$-th smallest value. Here, rows are sorted, and we can count elements $\le$ `mid` in $O(M)$ time using division.

---

## 16. Revision Notes
- **Key formula:** Median target position $K = (R \times C) / 2 + 1$.
- **Why upper_bound?** It returns the first element strictly greater than `mid`, which helps count elements $\le$ `mid`.
- **Search Space:** Bound the value range using first and last columns.
- **Complexity mnemonic:** $\text{Rows} \times \log(\text{Cols}) \times \log(\text{Max} - \text{Min})$.

---

## 17. Memorization Version
- **Find bounds:** `low` = min of first col, `high` = max of last col.
- **Binary Search:** `mid = low + (high - low) / 2`.
- **Count elements $\le$ mid:** Sum of `upper_bound(row) - row.begin()` for all rows.
- **Check count:** If `count < (R * C) / 2 + 1`, then `low = mid + 1`, else `high = mid`.
- **Result:** Return `low` when `low == high`. Space $O(1)$, Time $O(R \log C \log(\text{ValRange}))$.
