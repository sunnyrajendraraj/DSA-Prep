# Best Time to Buy and Sell Stock II (Multiple Transactions)

---

## 1. Problem Statement

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

---

## 2. Interview Clarification

1. **Can we make multiple transactions?**
   - *Answer:* Yes, unlimited transactions.
2. **Can we hold multiple shares of stock at the same time?**
   - *Answer:* No. You must sell the stock before you can buy it again.
3. **Can we buy and sell on the same day?**
   - *Answer:* Yes, though buying and selling on the same day yields a profit of 0. It is technically allowed and helps simplify the logic.
4. **Is there any transaction fee or cooldown period?**
   - *Answer:* No, there are no fees or cooldown periods in this variation.

---

## 3. Thinking Process

1. **First Instinct (Brute Force - Recursion):**
   - At each day, we make a decision. If we are currently holding a stock, we can either:
     - Sell it today.
     - Keep holding it (skip).
   - If we are not holding a stock, we can either:
     - Buy it today.
     - Skip today.
   - This leads to a decision tree of size $2^N$. We can implement this recursively to find the maximum profit.
   - Time Complexity: $O(2^N)$, Space Complexity: $O(N)$ stack space.

2. **Peak-Valley Approach (Better):**
   - If we look at the graph of prices over time, we want to buy at every local minimum (valley) and sell at the subsequent local maximum (peak).
   - We can trace the array, locate each valley, then locate the next peak, add the difference `peak - valley` to the profit, and repeat.
   - This runs in $O(N)$ time and $O(1)$ space.

3. **Simple Greedy Approach (Optimal):**
   - The Peak-Valley approach can be simplified further.
   - Instead of explicitly searching for valleys and peaks, we can look at every consecutive pair of days.
   - If `prices[i] > prices[i-1]`, we can buy on day `i-1` and sell on day `i`. This adds `prices[i] - prices[i-1]` to our total profit.
   - If `prices[i] <= prices[i-1]`, we do nothing.
   - This simple accumulation of all positive daily differences is guaranteed to yield the same result as the Peak-Valley approach, but with much simpler and cleaner code.
   - Time Complexity: $O(N)$, Space Complexity: $O(1)$.

---

## 4. Pattern Recognition

- **Pattern:** Greedy / Subarray Difference Accumulation.
- **Why this topic?** The problem allows unlimited transactions with zero transaction fees. Thus, the global maximum profit is simply the sum of all local monotonic increases.

---

## 5. Brute Force Solution

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

---

## 6. Better Solution

### Peak-Valley Approach
Explicitly find every valley and the next peak.

### Algorithm
1. Initialize `profit = 0`, `i = 0`.
2. While `i < N - 1`:
   - Find valley: Increment `i` while `prices[i] >= prices[i+1]`. Set `valley = prices[i]`.
   - Find peak: Increment `i` while `prices[i] <= prices[i+1]`. Set `peak = prices[i]`.
   - Add `peak - valley` to `profit`.
3. Return `profit`.

### Complexity
- **Time Complexity:** $O(N)$ since we iterate through the array once.
- **Space Complexity:** $O(1)$ auxiliary space.

### C++17 Code
```cpp
#include <vector>

class Solution {
public:
    int maxProfitBetter(const std::vector<int>& prices) {
        int n = prices.size();
        int i = 0;
        int valley = prices[0];
        int peak = prices[0];
        int maxProfit = 0;
        
        while (i < n - 1) {
            // Find local valley (minimum)
            while (i < n - 1 && prices[i] >= prices[i + 1]) {
                i++;
            }
            valley = prices[i];
            
            // Find local peak (maximum)
            while (i < n - 1 && prices[i] <= prices[i + 1]) {
                i++;
            }
            peak = prices[i];
            
            maxProfit += peak - valley;
        }
        
        return maxProfit;
    }
};
```

---

## 7. Optimal Solution

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

---

## 8. Code Walkthrough

1. **Initialize Profit:**
   ```cpp
   int maxProfit = 0;
   ```
   We start with a profit of 0.

2. **Iterate Consecutive Pairs:**
   ```cpp
   for (size_t i = 1; i < prices.size(); ++i)
   ```
   We start our loop from index `1` because we need to compare the current day's price with the previous day's price (`prices[i - 1]`).

3. **Greedy Accumulation:**
   ```cpp
   if (prices[i] > prices[i - 1]) {
       maxProfit += prices[i] - prices[i - 1];
   }
   ```
   If the price today is higher than yesterday, we simulate buying yesterday and selling today. By doing this daily, we automatically capture all cumulative upward trends (valleys to peaks) while skipping all downward trends.

---

## 9. Dry Run

Input: `prices = [7, 1, 5, 3, 6, 4]`

- **Initial State:** `maxProfit = 0`.

| Iteration | Day `i` | `prices[i]` | `prices[i-1]` | `prices[i] > prices[i-1]`? | Difference | `maxProfit` (Updated) |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| 1 | 1 | 1 | 7 | $1 > 7$ (False) | - | 0 |
| 2 | 2 | 5 | 1 | $5 > 1$ (True) | $5 - 1 = 4$ | $0 + 4 = 4$ |
| 3 | 3 | 3 | 5 | $3 > 5$ (False) | - | 4 |
| 4 | 4 | 6 | 3 | $6 > 3$ (True) | $6 - 3 = 3$ | $4 + 3 = 7$ |
| 5 | 5 | 4 | 6 | $4 > 6$ (False) | - | 7 |

Output: `7` (Buy at 1, Sell at 5 [profit 4]; Buy at 3, Sell at 6 [profit 3]).

---

## 10. Complexity Analysis

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

## 11. Common Mistakes

1. **Holding multiple stocks:**
   - Forgetting that you can only hold at most one stock. However, since the math of daily increments is equivalent to buying and selling properly, the greedy code naturally avoids violating this constraint.
2. **Off-by-One Loop Starting Index:**
   - Starting the loop at index `0` and comparing `prices[0]` with `prices[-1]`, which leads to undefined behavior. Always start at index `1`.

---

## 12. Edge Cases

| Edge Case | Input Example | Expected Output | Why it Matters |
|:---|:---|:---:|:---|
| **Strictly Decreasing Prices** | `[5, 4, 3, 2, 1]` | `0` | No profit can be made. |
| **Strictly Increasing Prices** | `[1, 2, 3, 4, 5]` | `4` | Captures all intermediate increments ($5 - 1 = 4$). |
| **Single Element** | `[5]` | `0` | Loop does not run, profit is 0. |
| **Flat Prices** | `[2, 2, 2, 2]` | `0` | No increments, profit is 0. |

---

## 13. Interview Explanation

**Speaker Script:**
> "To solve the Best Time to Buy and Sell Stock II problem where we can perform unlimited transactions, the key insight is that we want to capture every upward price movement. 
> If we draw the stock prices as a line graph, our goal is to buy at every local valley and sell at the next local peak. 
> Instead of explicitly writing complex logic to identify valleys and peaks, we can simplify this greedily. We can iterate through the prices day by day, comparing the price of the current day with the price of the previous day. 
> Whenever the current day's price is higher than the previous day's price, we add the difference to our total profit. 
> This is mathematically identical to buying at the valleys and selling at the peaks. It runs in $O(N)$ time and requires $O(1)$ auxiliary space."

---

## 14. Follow-up Questions

1. **What if there is a transaction fee for every trade?**
   - *Answer:* If there is a fee, the greedy approach fails because small daily profits might be wiped out by the fee. We must use Dynamic Programming with state transition:
     - `hold[i] = max(hold[i-1], sell[i-1] - prices[i])`
     - `sell[i] = max(sell[i-1], hold[i-1] + prices[i] - fee)`
2. **What if there is a cooldown day after selling?**
   - *Answer:* We also use Dynamic Programming, keeping track of three states: `hold`, `sold` (cooldown), and `reset` (idle).

---

## 15. Variations

1. **Best Time to Buy and Sell Stock with Transaction Fee (LeetCode 714)**
2. **Best Time to Buy and Sell Stock with Cooldown (LeetCode 309)**

---

## 16. Revision Notes

- **Greedy Rule:** Sum up all positive adjacent differences: `prices[i] - prices[i-1]`.
- **Valleys & Peaks:** Equivalent to accumulating daily increases.
- **Complexity:** $O(N)$ Time, $O(1)$ Space.

---

## 17. Memorization Version

```cpp
// Time: O(N), Space: O(1)
int maxProfit(vector<int>& prices) {
    int maxProfit = 0;
    for (size_t i = 1; i < prices.size(); ++i) {
        if (prices[i] > prices[i - 1]) {
            maxProfit += prices[i] - prices[i - 1];
        }
    }
    return maxProfit;
}
```
