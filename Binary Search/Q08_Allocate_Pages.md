# Allocate Minimum Number of Pages

## 1. Problem Statement
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

---

## 2. Interview Clarification
Before coding, a candidate should clarify:
1. **Q:** Can books be allocated non-contiguously?
   - **A:** No, books must be allocated contiguously (e.g., student 1 gets books $0$ to $i$, student 2 gets books $i+1$ to $j$, etc.).
2. **Q:** What if there are more students than books?
   - **A:** Since each student must get at least one book, it is impossible to distribute them. In this case, return `-1`.
3. **Q:** Can page counts be negative?
   - **A:** No, page counts are always positive integers.
4. **Q:** Are the book page numbers sorted?
   - **A:** No, the page counts can be in any arbitrary order.

---

## 3. Thinking Process
### First Instinct
We need to divide the array of size $N$ into $M$ contiguous subarrays such that the maximum sum among these subarrays is minimized. 
We could try to find all possible partitions using recursion/backtracking. However, the number of ways to partition $N$ elements into $M$ groups is extremely large ($O(\binom{N-1}{M-1})$), which leads to an exponential time complexity of $O(2^N)$ and is impractical for $N = 10^5$.

### Can we do better?
Let's consider the possible range of the answer:
- The minimum possible answer is $\max(\text{pages})$ because the book with the most pages must be allocated to some student, and that student will read at least that many pages.
- The maximum possible answer is $\sum(\text{pages})$ when all books are assigned to one student.

If we pick a candidate maximum limit $X$, we can check if it is possible to allocate the books such that no student reads more than $X$ pages:
- We can iterate through the books and greedily assign them to the current student.
- If adding the current book exceeds $X$, we assign it to the next student and reset the current student's pages to the current book's pages.
- If we can complete this allocation using at most $M$ students, then $X$ is a **valid limit**.
- If it requires more than $M$ students, then $X$ is **too small**.

Since the validity of a limit $X$ is monotonic:
- If $X$ is valid, any $Y > X$ will also be valid.
- If $X$ is invalid, any $Y < X$ will also be invalid.

This monotonicity is a clear signal for **Binary Search on the Answer**.

---

## 4. Pattern Recognition
This problem belongs to the **Binary Search on Answer** pattern.
- Why? We are searching for a minimum target value in a defined, contiguous range $[\max(\text{pages}), \sum(\text{pages})]$. 
- A helper function `isValid(X)` determines if a candidate value $X$ is feasible. The search space is monotonic: `[Invalid, Invalid, ..., Valid, Valid, Valid]`. We want to find the first "Valid" index.
- Similar problems: Painter's Partition, Split Array Largest Sum, Capacity To Ship Packages Within D Days.

---

## 5. Brute Force Solution
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

---

## 6. Better Solution
*Note: For this specific problem, there is no intermediate "Better" solution that sits between the brute force (linear search on answer) and the optimal Binary Search on the answer. We jump directly from Brute Force to the Optimal Binary Search.*

---

## 7. Optimal Solution
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

---

## 8. Code Walkthrough
- `if (m > n) return -1;`: Immediate exit check. We cannot allocate books if students exceed books.
- `int low = *std::max_element(...)`: Initializes the bottom of our search range. Since we cannot split a book, the minimum possible limit must be at least the largest single book.
- `long long high = std::accumulate(...)`: Initializes the top of our search range. The absolute maximum pages a student could read is the sum of all pages. We use `long long` to prevent integer overflow during summation.
- `while (low <= high)`: Standard binary search loop that covers the range.
- `long long mid = low + (high - low) / 2;`: Midpoint calculation.
- `isValidLimit(pages, n, m, mid)`: Greedily tests whether we can allocate the books using at most `m` students under the maximum page limit `mid`.
- `ans = mid; high = mid - 1;`: If the configuration is valid, we try to optimize further by minimizing the maximum limit, so we check the left subarray.
- `low = mid + 1`: If invalid, the limit `mid` is too small, so we discard it and search the right subarray.

---

## 9. Dry Run
Using `pages` = `[12, 34, 67, 90]`, $M = 2$, $N = 4$, `low` = 90, `high` = 203.

| Step | low | high | mid | Student Count (Limit = mid) | Valid? | Action | ans |
|---|---|---|---|---|---|---|---|
| 1 | 90 | 203 | 146 | 2 (`[12,34,67]`, `[90]`) | Yes | `high = 145` | 146 |
| 2 | 90 | 145 | 117 | 2 (`[12,34,67]`, `[90]`) | Yes | `high = 116` | 117 |
| 3 | 90 | 116 | 103 | 3 (`[12,34]`, `[67]`, `[90]`) | No | `low = 104` | 117 |
| 4 | 104 | 116 | 110 | 3 (`[12,34]`, `[67]`, `[90]`) | No | `low = 111` | 117 |
| 5 | 111 | 116 | 113 | 2 (`[12,34,67]`, `[90]`) | Yes | `high = 112` | 113 |
| 6 | 111 | 112 | 111 | 3 (`[12,34]`, `[67]`, `[90]`) | No | `low = 112` | 113 |
| 7 | 112 | 112 | 112 | 3 (`[12,34]`, `[67]`, `[90]`) | No | `low = 113` | 113 |

Loop terminates. Returns `113`.

---

## 10. Complexity Analysis
- **Time Complexity:**
  - Finding maximum and sum takes $O(N)$ time.
  - The binary search range length is $S = \sum(\text{pages}) - \max(\text{pages})$. The binary search takes $\log(S)$ iterations.
  - In each iteration, we run `isValidLimit` which scans the array of size $N$ once, taking $O(N)$ time.
  - **Total Time Complexity:** $O(N \log(\sum(\text{pages})))$. For $N = 10^5$ and page values up to $10^5$, sum is $10^{10}$, $\log_2(10^{10}) \approx 34$. Total operations $\approx 3.4 \times 10^6$, which runs well within 0.05 seconds.
- **Space Complexity:**
  - $O(1)$ auxiliary space. We do not use any extra space during search.

---

## 11. Common Mistakes
1. **Incorrect Search Range:** Starting `low` at 0 or 1 instead of `max_element(pages)`. If `low` is smaller than the largest book, then `isValidLimit` will correctly fail, but initializing it to `max_element` saves range and prevents logic bugs.
2. **Integer Overflow:** Calculating the sum of pages using standard `int` can cause integer overflow if sum exceeds $2 \times 10^9$. Always use `long long` for the sum and `high`/`mid` variables.
3. **Invalid Check Logic:** Forgetting to handle the case where a single book has page count larger than `maxLimit` inside `isValidLimit`.
4. **Incorrect condition on allocation impossibility:** Failing to return `-1` when $M > N$.

---

## 12. Edge Cases
| Edge Case | Example | Why it matters | Solution |
|---|---|---|---|
| More students than books | `pages=[10, 20]`, $M=3$ | Impossible to allocate at least one book to each. | Immediate check `if (m > n) return -1;` |
| Students equals books | `pages=[10, 20, 30]`, $M=3$ | Each student gets exactly one book. | Minimum max pages is the max element: 30. |
| Single book | `pages=[10]`, $M=1$ | Edge case for array size. | Handled correctly: `low=10`, `high=10`. Returns 10. |
| Large page values | `pages=[10^9, 10^9]`, $M=1$ | Sum exceeds $2^{31}-1$. | Use `long long` for binary search boundaries. |

---

## 13. Interview Explanation
"The problem asks us to distribute $N$ books among $M$ students contiguously to minimize the maximum workload of any student. 
Since the books must be assigned contiguously, this is equivalent to partitioning the array into $M$ subarrays such that the maximum subarray sum is minimized.
The possible range of our answer lies between the maximum single book's page count (which is the absolute minimum a student must read if they get that book) and the sum of all pages (which is what one student reads if they read all books). 
Since the viability of a maximum page limit $X$ is monotonic (if a limit of $100$ pages is possible, then $105$ is also possible), we can perform binary search on this range. 
For each candidate limit `mid`, we write a greedy checker that iterates through the books and counts how many students are needed. If the students needed are $\le M$, then `mid` is a valid limit, and we try to find a smaller one by searching the left half. Otherwise, `mid` is too small, and we search the right half.
This approach runs in $O(N \log(\sum(\text{pages})))$ time and uses $O(1)$ space, which is optimal."

---

## 14. Follow-up Questions
1. **Q: How does this problem relate to the Painter's Partition Problem?**
   - **A:** It is identical. In Painter's Partition, we have boards of various lengths and painters who paint contiguously. The goal is to minimize the maximum time a painter spends. If each unit takes $1$ unit of time, it is exactly the same problem.
2. **Q: How does this relate to Split Array Largest Sum?**
   - **A:** It is the exact same problem under a different name. We split an array into $M$ subarrays to minimize the maximum subarray sum.
3. **Q: What if the students can read non-contiguously?**
   - **A:** If contiguity is not required, this is the NP-hard multiprocessor scheduling problem. We would have to use backtracking or dynamic programming / approximation algorithms instead of simple binary search.

---

## 15. Variations
1. **Painter's Partition Problem:**
   - Same logic, but typically introduces a multiplier $K$ (time taken per unit length). Calculate the optimal length distribution first, then multiply the final answer by $K$.
2. **Capacity To Ship Packages Within D Days (LeetCode 1011):**
   - Ship packages with weights $W_i$ within $D$ days. The daily capacity is the answer we binary search on.

---

## 16. Revision Notes
- **Key Pattern:** Binary Search on Answer.
- **Monotonicity Property:** If we can distribute books with max pages $\le X$, we can also do it for any limit $> X$.
- **Lower bound:** `max_element`
- **Upper bound:** `sum_elements`
- **Checker complexity:** $O(N)$ greedy pass.

---

## 17. Memorization Version
- Check `if (M > N) return -1;`
- Range: `low = max(pages)`, `high = sum(pages)`.
- Binary Search: `mid = low + (high - low) / 2`.
- Checker: Greedily sum elements. If sum > `mid`, increment student count and start new sum. If students count > `M`, return false.
- Update range: If valid, `ans = mid`, `high = mid - 1`. If invalid, `low = mid + 1`.
- Return `ans`.
