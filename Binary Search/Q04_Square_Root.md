# Square Root using Binary Search

---

## 1. Problem Statement

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

---

## 2. Interview Clarification

Before coding, clarify these details:
1. **How should we handle $n = 0$?** (Square root of $0$ is $0$. Our code should handle this correctly).
2. **What should be returned if the square root is not a perfect square?** (Return the floor value, i.e., truncate decimal parts. For example, $\sqrt{8} \approx 2.828$, so we return $2$).
3. **Can we use `long long` to prevent overflow when calculating $x^2$?** (Yes, standard 32-bit integers will overflow when squaring values $\ge 46341$, since $46341^2 > 2^{31} - 1$. Using `long long` or division prevents this).

---

## 3. Thinking Process

### First Instinct (Brute Force)
We can start from $i = 0$ and check $i \times i$ for consecutive integers. As soon as we find an $i$ such that $i \times i > n$, the previous integer $i-1$ is our answer.
This linear search will take $O(\sqrt{n})$ time.

### Optimal Insight (Binary Search on Answer)
Since the range of potential answers $[0, n]$ is sorted, and the condition $x^2 \le n$ is monotonic (i.e., it is true up to some value and then false for all larger values), we can binary search on the answer:
1. Set `low = 0` and `high = n`.
2. While `low <= high`:
   - Compute `mid = low + (high - low) / 2`.
   - Calculate `square = mid * mid`.
   - If `square == n`, then `mid` is the exact square root. Return `mid`.
   - If `square < n`, then `mid` is a valid candidate for $\lfloor \sqrt{n} \rfloor$. We save `mid` as our best answer so far and search to the right to see if a larger valid integer exists: `low = mid + 1`.
   - If `square > n`, then `mid` is too large. We search to the left: `high = mid - 1`.
3. Return the saved candidate.

---

## 4. Pattern Recognition

This is a **Binary Search on Answer** problem.
- **Why this topic?** The target we are searching for lies within a contiguous, ordered range of integers $[0, n]$, and the validation condition ($x^2 \le n$) has a monotonic truth pattern: `[True, True, True, False, False]`. We want to find the last `True`.

---

## 5. Brute Force Solution

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

---

## 6. Better Solution

Instead of using multiplication which can overflow, we can rewrite the condition as `mid <= n / mid`. This avoids `long long` entirely, but requires special handling for $mid = 0$. However, it doesn't improve the time complexity class over binary search, so we jump directly to the optimal implementation.

---

## 7. Optimal Solution

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

---

## 8. Code Walkthrough

- **`long long square = static_cast<long long>(mid) * mid`**: Without the cast, `mid * mid` is calculated as a 32-bit signed integer first, which overflows before being assigned to a `long long`. Casting `mid` to `long long` first ensures that the multiplication is done using 64-bit precision.
- **`ans = mid`**: We store the last value of `mid` that satisfied `mid * mid < n`. This handles the floor condition correctly.
- **Base cases `n == 0 || n == 1`**: Directly handled to avoid edge conditions with division or range bounds.

---

## 9. Dry Run

Let's dry run for `n = 16`:

| Step | `low` | `high` | `mid` | `square` | Comparison | Action | `ans` |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| Init | 1 | 16 | - | - | - | - | 0 |
| 1 | 1 | 16 | 8 | 64 | `64 > 16` | `high = 7` | 0 |
| 2 | 1 | 7 | 4 | 16 | `16 == 16`| Return `4` | 4 |

---

## 10. Complexity Analysis

| Metric | Complexity | Reasoning |
| :--- | :--- | :--- |
| **Best Case Time** | $O(1)$ | Target is found at the first midpoint (e.g., $n = 16$, first mid is $8$, second is $4$, exact match). |
| **Average Case Time** | $O(\log n)$ | Search space of size $n$ is halved at each step. |
| **Worst Case Time** | $O(\log n)$ | Occurs when there is no perfect square root (e.g., $n = 8$), requiring full search space exhaustion. |
| **Space Complexity** | $O(1)$ | No extra space used besides a few variables. |

---

## 11. Common Mistakes

1. **Integer Overflow:**
   ```cpp
   int square = mid * mid; // If mid = 46341, mid * mid will overflow 32-bit signed int.
   ```
2. **Missing `static_cast<long long>`:**
   ```cpp
   long long square = mid * mid; // Bug! The multiplication is evaluated in 32-bit integer arithmetic first, overflows, and then gets assigned to long long.
   ```
   *Correction:* `long long square = (long long)mid * mid;`
3. **No Overflow Division Alternative:**
   If you cannot use `long long`, use:
   ```cpp
   if (mid <= n / mid) { ... }
   ```
   Note: this requires checking `mid != 0`.

---

## 12. Edge Cases

| Edge Case | Input | Expected Output | Rationale |
| :--- | :--- | :--- | :--- |
| **Zero** | `n = 0` | `0` | $\sqrt{0} = 0$. Base case checks this. |
| **One** | `n = 1` | `1` | $\sqrt{1} = 1$. Base case checks this. |
| **Max Int** | `n = 2147483647` | `46340` | Checks overflow protection. $46340^2 = 2147395600 \le n$, but $46341^2 = 2147488281 > n$. |
| **Non-perfect square** | `n = 2` | `1` | $\sqrt{2} \approx 1.414$, truncated to `1`. |

---

## 13. Interview Explanation

> "To calculate the integer square root of a non-negative integer $n$ without using built-in functions, we can perform a binary search. The search space for the square root is bounded between $1$ and $n$ for $n \ge 1$. Since this range is sorted, we check the midpoint `mid`. We calculate its square using `long long` to prevent integer overflow. If `mid * mid` is exactly $n$, we return `mid`. If it's less than $n$, then `mid` is a potential floor answer. We store it and search the right half for a larger potential square root. If `mid * mid` is greater than $n$, `mid` is too large, and we search the left half. This allows us to find the answer in $O(\log n)$ time."

---

## 14. Follow-up Questions

1. **How would you find the square root with a precision of up to 5 decimal places?**
   - *Answer:* We can extend binary search to floating-point numbers. Instead of using `low <= high` and `low = mid + 1`, we use `high - low > 1e-6` as the loop condition.
   - Example template:
     ```cpp
     double low = 0, high = n, mid;
     while (high - low > 1e-6) {
         mid = low + (high - low) / 2;
         if (mid * mid < n) low = mid;
         else high = mid;
     }
     return low;
     ```
2. **How does Newton's method compare to binary search for this problem?**
   - *Answer:* Newton's method (using the update $x_{k+1} = \frac{1}{2}(x_k + \frac{n}{x_k})$) converges quadratically, which means it doubles the number of correct digits at each step. It is generally faster than binary search but harder to implement correctly regarding integer divisions and convergence bounds.

---

## 15. Variations

1. **Valid Perfect Square (LeetCode 367):** Check if a number is a perfect square. The code is identical, but instead of returning `ans`, we check if `ans * ans == n`.
2. **Cube Root:** Find the integer cube root of $n$. The search space is $[0, n]$, and the condition becomes `mid * mid * mid <= n`.

---

## 16. Revision Notes

- **Condition:** `(long long)mid * mid <= n`
- **Result:** Store the best match in `ans` when search condition is met, then go right (`low = mid + 1`).
- **Standard Overflow Threshold:** $\sqrt{2^{31}-1} \approx 46340.95$. Any `mid` value above `46340` will overflow 32-bit signed integers when squared.

---

## 17. Memorization Version

```cpp
int mySqrt(int n) {
    if (n < 2) return n;
    int low = 1, high = n, ans = 0;
    while (low <= high) {
        int mid = low + (high - low) / 2;
        if ((long long)mid * mid <= n) {
            ans = mid;
            low = mid + 1;
        } else {
            high = mid - 1;
        }
    }
    return ans;
}
```
*Time:* $O(\log n)$, *Space:* $O(1)$. Use `long long` for squaring. Store `ans` when `mid * mid <= n` and search right.
