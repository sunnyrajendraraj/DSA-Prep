# Kth Smallest Element in Sorted Matrix

## 1. Problem Statement
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

---

## 2. Interview Clarification
Before coding, a candidate should clarify:
1. **Q:** Can the matrix contain duplicate elements?
   - **A:** Yes, duplicates are allowed, and we need the $K$-th smallest element considering all duplicates (i.e. standard 1-based index in the sorted list of all $N^2$ elements).
2. **Q:** Can $K$ be larger than $N^2$?
   - **A:** No, $1 \le K \le N^2$.
3. **Q:** What is the range of values in the matrix?
   - **A:** Elements can range from $-10^9$ to $10^9$. This means we must compute `mid` carefully to avoid overflow, and use `long long` if needed.

---

## 3. Thinking Process
### First Instinct (Brute Force)
We can flatten the 2D matrix into a 1D array of size $N^2$, sort it, and return the element at index $K - 1$.
- This takes $O(N^2 \log(N^2)) = O(N^2 \log N)$ time and $O(N^2)$ space.

### Can we do better? (Min-Heap / Priority Queue)
Since both rows and columns are sorted, we can treat this as a "Merge $N$ Sorted Lists" problem.
- We can insert the first element of each row into a Min-Heap: elements stored will be tuples of `(value, row, col)`.
- We extract the minimum element from the heap $K$ times.
- When we extract `(value, r, c)`, if there is a next element in the same row (i.e., `c + 1 < N`), we insert `(matrix[r][c+1], r, c+1)` into the heap.
- The $K$-th extracted element is our answer.
- Time complexity: $O(K \log N)$. Space complexity: $O(N)$ to store the heap elements.
- This is a solid improvement, especially if $K$ is small. However, if $K \approx N^2$, the time complexity becomes $O(N^2 \log N)$, which is the same as the brute force.

### Can we do even better? (Optimal Binary Search)
We can perform Binary Search on the value range `[low, high]`, where `low = matrix[0][0]` and `high = matrix[N-1][N-1]`.
- For a candidate value `mid`, we count the number of elements in the matrix that are less than or equal to `mid`.
- Since both rows and columns are sorted, we can count the elements $\le$ `mid` in $O(N)$ time instead of $O(N \log N)$ or $O(N^2)$ by using a **Staircase Search** (starting from the bottom-left or top-right of the matrix).
- Let's start from the bottom-left corner `(N-1, 0)`:
  - If `matrix[row][col] <= mid`, it means all elements in the current column from index `0` to `row` are also less than or equal to `mid`. So we add `row + 1` to our count and move to the right: `col++`.
  - If `matrix[row][col] > mid`, we move up: `row--`.
- If the total count of elements $\le$ `mid` is less than $K$, it means `mid` is too small. We search the right half: `low = mid + 1`.
- If the total count is $\ge K$, `mid` could be the $K$-th smallest element or larger. We search the left half: `high = mid`.
- When `low == high`, we have found the $K$-th smallest element.

---

## 4. Pattern Recognition
This problem belongs to the **Binary Search on Value Range (with Staircase Search)** pattern.
- Why? We are searching for an element of a specific rank ($K$-th smallest) in a sorted 2D grid.
- We utilize the sorted rows and columns to count elements $\le \text{mid}$ in linear time $O(N)$.

---

## 5. Brute Force Solution
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

---

## 6. Better Solution (Min-Heap / Merge K Sorted Lists)
### Algorithm
1. Initialize a Min-Heap storing `tuple<value, row, col>`.
2. Push the first element of each row: `(matrix[i][0], i, 0)` for all $0 \le i < N$.
3. Do the following $K$ times:
   - Extract the top element `(val, r, c)`.
   - If this is the $K$-th extraction, return `val`.
   - If `c + 1 < N`, push `(matrix[r][c + 1], r, c + 1)` into the heap.

### Complexity Analysis
- **Time Complexity:** $O(K \log N)$. If $K$ is small, this is extremely fast. In the worst case (e.g., $K = N^2$), it takes $O(N^2 \log N)$.
- **Space Complexity:** $O(N)$ to store the heap elements (max size of heap is $N$).

### C++17 Code
```cpp
#include <vector>
#include <queue>
#include <tuple>

class Solution {
public:
    int kthSmallestHeap(std::vector<std::vector<int>>& matrix, int k) {
        int n = matrix.size();
        // Min-heap storing: {value, row, col}
        using Element = std::tuple<int, int, int>;
        std::priority_queue<Element, std::vector<Element>, std::greater<Element>> minHeap;

        // Push the first element of each row
        for (int i = 0; i < n; ++i) {
            minHeap.push({matrix[i][0], i, 0});
        }

        int count = 0;
        int result = 0;

        while (!minHeap.empty()) {
            auto [val, r, c] = minHeap.top();
            minHeap.pop();
            count++;

            if (count == k) {
                result = val;
                break;
            }

            // Push the next element from the same row
            if (c + 1 < n) {
                minHeap.push({matrix[r][c + 1], r, c + 1});
            }
        }

        return result;
    }
};
```

---

## 7. Optimal Solution
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

---

## 8. Code Walkthrough
- `int low = matrix[0][0]; int high = matrix[n - 1][n - 1];`: Set boundaries of values. Since the matrix is sorted, the minimum element is at top-left, and the maximum is at bottom-right.
- `while (low <= high)`: Standard binary search range loop.
- `countLessEqual(matrix, n, mid)`: Evaluates rank of `mid`.
  - Start at bottom-left `row = n - 1`, `col = 0`.
  - If current cell value $\le$ `mid`, add `row + 1` to `count` (since everything above in this column is smaller) and move right (`col++`).
  - Otherwise, move up (`row--`).
- `if (count >= k)`: If there are at least $K$ elements less than or equal to `mid`, this means the $K$-th element must be $\le$ `mid`. We record `ans = mid` and search the left half (`high = mid - 1`).
- `low = mid + 1`: If there are fewer than $K$ elements $\le$ `mid`, then `mid` is too small, so we search the right half.

---

## 9. Dry Run
Using same Matrix, $K=8$.

| Step | low | high | mid | Staircase Count | Count >= K ($8$)? | Action | ans |
|---|---|---|---|---|---|---|---|
| 1 | 1 | 15 | 8 | 2 | No | `low = 9` | 1 |
| 2 | 9 | 15 | 12 | 6 | No | `low = 13` | 1 |
| 3 | 13 | 15 | 14 | 8 | Yes | `high = 13` | 14 |
| 4 | 13 | 13 | 13 | 8 | Yes | `high = 12` | 13 |

Loop ends since `low` (13) > `high` (12). Return `13`.

---

## 10. Complexity Analysis
- **Time Complexity:**
  - The range of values is $R = \text{matrix}[N-1][N-1] - \text{matrix}[0][0]$. The binary search takes $O(\log R)$ iterations. For integer constraints, this is at most $\approx 32$ iterations.
  - In each iteration, we run the staircase search. In the worst case, the path from bottom-left to top-right takes at most $2N$ steps. So, `countLessEqual` runs in $O(N)$ time.
  - **Total Time Complexity:** $O(N \log(\text{max\_val} - \text{min\_val}))$. For $N = 300$, this is extremely fast, requiring only about $300 \times 32 \approx 9600$ operations.
- **Space Complexity:**
  - $O(1)$ auxiliary space.

---

## 11. Common Mistakes
1. **Wrong staircase initialization:** Initializing at `row = 0, col = 0` or `row = N-1, col = N-1`. The staircase search only works if we start at one of the two corners where one direction increases values and the other direction decreases values (i.e., bottom-left or top-right).
2. **Standard binary search on index:** Attempting to treat the 2D grid index as a 1D index and doing binary search on indexes `[0, N^2 - 1]`. This only works if the whole matrix is sorted in a row-major 1D array order (like LeetCode 74: Search a 2D Matrix), which is not the case here because the start of a row can be smaller than the end of the previous row.
3. **Integer Overflow:** If matrix values are near `INT_MAX` or `INT_MIN`, subtracting `high - low` can overflow. Use `long long` for calculations.

---

## 12. Edge Cases
| Edge Case | Example | Why it matters | Solution |
|---|---|---|---|
| $K = 1$ | Target is minimum element | Answer must be the top-left element. | Correctly resolved since count for any value $< \text{matrix}[0][0]$ is 0. |
| $K = N^2$ | Target is maximum element | Answer must be bottom-right element. | Correctly resolved since count for any value $\ge \text{matrix}[N-1][N-1]$ is $N^2$. |
| $N = 1$ | Grid size is $1 \times 1$ | Edge dimension. | Loop terminates immediately and returns the single element. |
| Negative values | Values between $-10^9$ and $-1$ | Binary search must handle negative coordinates properly. | Standard range-based logic supports negative bounds. |

---

## 13. Interview Explanation
"To find the $K$-th smallest element in a sorted $N \times N$ matrix, we can optimize beyond the heap-based approach by using Binary Search on the value range. 
The values in the matrix range from the minimum element at `matrix[0][0]` to the maximum element at `matrix[N-1][N-1]`. We perform binary search on this range. 
For each candidate value `mid`, we need to count how many elements in the matrix are less than or equal to `mid`. 
Since both rows and columns are sorted, we can count this in $O(N)$ time using a Staircase Search: we start at the bottom-left corner of the matrix. If the element is $\le$ `mid`, it means all elements above it in the same column are also $\le$ `mid`. We add `row + 1` to our count and move to the right. If the element is $>$ `mid`, we move up. 
If the total count is $\ge K$, it means the $K$-th smallest element is $\le$ `mid`, so we record `mid` as a candidate and search the left half. Otherwise, we search the right half. 
This optimal approach runs in $O(N \log(\text{max\_val} - \text{min\_val}))$ time and uses $O(1)$ space."

---

## 14. Follow-up Questions
1. **Q: Why does the heap-based approach perform better for very small $K$?**
   - **A:** If $K$ is extremely small (e.g. $K \le O(N)$), the heap approach runs in $O(K \log N)$ time, which is faster than $O(N \log R)$ since $K$ is small. Thus, we should use heap for $K \ll N$ and binary search for large $K$.
2. **Q: Can we use this method to find the median of the matrix?**
   - **A:** Yes, finding the median is a special case of finding the $K$-th smallest element where $K = (N^2 + 1) / 2$.

---

## 15. Variations
1. **Search a 2D Matrix II (LeetCode 240):**
   - Find if a target value exists in a sorted grid. Uses the same Staircase Search algorithm.
2. **K-th Smallest Element in a Multiplication Table:**
   - Same value-range binary search, but the count of elements is calculated in $O(M)$ by division: $\sum \min(N, \lfloor \text{mid} / i \rfloor)$.

---

## 16. Revision Notes
- **Corner Choice:** Start at bottom-left `(N-1, 0)` or top-right `(0, N-1)`.
- **Count Calculation:** If cell $\le \text{mid}$, `count += row + 1`, `col++`. Else `row--`.
- **Binary Search Space:** Value space, not index space.
- **Time Complexity:** $O(N \log(\text{Max} - \text{Min}))$. Space: $O(1)$.

---

## 17. Memorization Version
- Range: `low = matrix[0][0]`, `high = matrix[N-1][N-1]`.
- Loop: `mid = low + (high - low) / 2`.
- Staircase count of elements $\le$ `mid`:
  - Start `r = N-1`, `c = 0`.
  - If `val <= mid`: `count += r + 1`, `c++`.
  - Else: `r--`.
- Update: If `count >= K`: `ans = mid`, `high = mid - 1`. Else: `low = mid + 1`.
- Return `ans`.
