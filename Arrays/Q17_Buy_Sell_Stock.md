# Best Time to Buy and Sell Stock (Single Transaction)

---

## 1. Problem Statement

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

---

## 2. Interview Clarification

1. **What if the array has only 1 day?**
   - *Answer:* If there is only 1 day, we cannot perform both a buy and a sell on different days. The profit is `0`.
2. **Can we buy and sell on the same day?**
   - *Answer:* No, the sell must happen in the future (on a different day).
3. **What should we return if prices only decrease?**
   - *Answer:* Return `0`.
4. **Do we need to return the days on which to buy and sell?**
   - *Answer:* The problem only asks for the maximum profit, but it is good to design the code so it could track the days if needed.

---

## 3. Thinking Process

1. **First Instinct (Brute Force):**
   - Compare every possible buy day with every possible sell day that occurs after it.
   - Using two nested loops, the outer loop represents the buy day $i$, and the inner loop represents the sell day $j$ (from $i+1$ to $N-1$).
   - The profit for each pair is `prices[j] - prices[i]`. We keep track of the maximum profit.
   - Time Complexity: $O(N^2)$, Space Complexity: $O(1)$. This is too slow for $N = 10^5$.

2. **Suffix Maximum Array (Better):**
   - If we buy on day $i$, the best day to sell in the future is the day with the maximum price among all days from $i+1$ to $N-1$.
   - We can precompute this. Let `maxFuturePrice[i]` be the maximum price from index $i$ to $N-1$.
   - We can construct `maxFuturePrice` from right to left:
     - `maxFuturePrice[N-1] = prices[N-1]`
     - `maxFuturePrice[i] = max(prices[i], maxFuturePrice[i+1])`
   - Then, for each day $i$, the maximum profit if we buy on day $i$ is `maxFuturePrice[i] - prices[i]`.
   - We find the maximum of these values.
   - Time Complexity: $O(N)$, Space Complexity: $O(N)$ for the suffix array.

3. **Tracking Minimum So Far (Optimal):**
   - Instead of looking forward (what is the maximum future price?), we can look backward: if we sell on day $j$, what was the lowest price we could have bought it at?
   - The lowest buy price is the minimum price seen from day $0$ to day $j-1$.
   - As we traverse the array from left to right, we can dynamically maintain the minimum price seen so far (`minPrice`).
   - For each price `prices[j]` at index $j$:
     - The potential profit is `prices[j] - minPrice`.
     - Update `maxProfit` if this potential profit is larger.
     - Update `minPrice` if `prices[j]` is smaller than the current `minPrice`.
   - This approach runs in $O(N)$ time and requires only $O(1)$ space.

---

## 4. Pattern Recognition

- **Pattern:** Greedy / Single-pass Optimization.
- **Why this topic?** The optimal selling decision on day $j$ only depends on the minimum buying price seen before day $j$. Since we traverse left-to-right, we can maintain this minimum incrementally in $O(1)$ space.

---

## 5. Brute Force Solution

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

---

## 6. Better Solution

### Suffix Maximum Array
Precompute the maximum price to the right of each element.

### Algorithm
1. Create a `maxFuturePrice` array of size $N$.
2. Initialize `maxFuturePrice[N-1] = prices[N-1]`.
3. Fill `maxFuturePrice` from right to left:
   - `maxFuturePrice[i] = max(prices[i], maxFuturePrice[i+1])`
4. Initialize `maxProfit = 0`.
5. For each index `i` from `0` to $N-1$:
   - Calculate potential profit: `maxFuturePrice[i] - prices[i]`.
   - Update `maxProfit = max(maxProfit, potentialProfit)`.
6. Return `maxProfit`.

### Complexity
- **Time Complexity:** $O(N)$ - One pass to build the suffix array, one pass to compute profit.
- **Space Complexity:** $O(N)$ to store the suffix array.

### C++17 Code
```cpp
#include <vector>
#include <algorithm>

class Solution {
public:
    int maxProfitBetter(const std::vector<int>& prices) {
        int n = prices.size();
        if (n <= 1) return 0;
        
        std::vector<int> maxFuturePrice(n);
        maxFuturePrice[n - 1] = prices[n - 1];
        
        for (int i = n - 2; i >= 0; --i) {
            maxFuturePrice[i] = std::max(prices[i], maxFuturePrice[i + 1]);
        }
        
        int maxProfit = 0;
        for (int i = 0; i < n; ++i) {
            maxProfit = std::max(maxProfit, maxFuturePrice[i] - prices[i]);
        }
        
        return maxProfit;
    }
};
```

---

## 7. Optimal Solution

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

---

## 8. Code Walkthrough

1. **State Initialization:**
   ```cpp
   int minPrice = prices[0];
   int maxProfit = 0;
   ```
   We assume the first day is our starting minimum price and set our initial profit to `0`.

2. **Single Pass Evaluation:**
   ```cpp
   int currentProfit = prices[i] - minPrice;
   maxProfit = std::max(maxProfit, currentProfit);
   ```
   For each day `i`, we check the profit if we were to buy at `minPrice` (which occurred before day `i`) and sell today. We update our global `maxProfit` if this profit is higher.

3. **Update Minimum Price:**
   ```cpp
   minPrice = std::min(minPrice, prices[i]);
   ```
   We check if the current day's price is cheaper than our previous `minPrice`. If so, this becomes the new buying candidate for all future selling days.

---

## 9. Dry Run

Input: `prices = [7, 1, 5, 3, 6, 4]`

- **Initial State:** `minPrice = 7`, `maxProfit = 0`.

| Iteration | Index `i` | Price `prices[i]` | `currentProfit = prices[i] - minPrice` | `maxProfit` (Updated) | `minPrice` (Updated) |
|:---:|:---:|:---:|:---:|:---:|:---:|
| 1 | 1 | 1 | $1 - 7 = -6$ | $\max(0, -6) = 0$ | $\min(7, 1) = 1$ |
| 2 | 2 | 5 | $5 - 1 = 4$ | $\max(0, 4) = 4$ | $\min(1, 5) = 1$ |
| 3 | 3 | 3 | $3 - 1 = 2$ | $\max(4, 2) = 4$ | $\min(1, 3) = 1$ |
| 4 | 4 | 6 | $6 - 1 = 5$ | $\max(4, 5) = 5$ | $\min(1, 6) = 1$ |
| 5 | 5 | 4 | $4 - 1 = 3$ | $\max(5, 3) = 5$ | $\min(1, 4) = 1$ |

Output: `5` (Buy on Day 1 at price 1, Sell on Day 4 at price 6).

---

## 10. Complexity Analysis

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

## 11. Common Mistakes

1. **Selling Before Buying:**
   - Attempting to track the largest element and the smallest element in the array and subtracting them directly without ensuring the smallest element's index is smaller than the largest element's index.
2. **Incorrect Initialization of `minPrice`:**
   - Initializing `minPrice` to `0`. If prices are all positive, `0` will remain the minimum price, which is incorrect. Initialize it to `prices[0]` or `INT_MAX`.
3. **Resetting `minPrice` and `maxProfit` in the wrong order:**
   - If you update `minPrice` *before* calculating `currentProfit`, you would calculate the profit of buying and selling on the exact same day, which is invalid (though it results in 0 profit, it's a logical bug if prices only decrease).

---

## 12. Edge Cases

| Edge Case | Input Example | Expected Output | Why it Matters |
|:---|:---|:---:|:---|
| **Strictly Decreasing Prices** | `[7, 6, 4, 3, 1]` | `0` | No profit can be made. Output should be `0`. |
| **Strictly Increasing Prices** | `[1, 2, 4, 6, 7]` | `6` | Maximum profit is obtained by buying on the first day and selling on the last day. |
| **Single Element Array** | `[5]` | `0` | Standard loop shouldn't run; output should be `0`. |
| **Identical Prices** | `[3, 3, 3, 3]` | `0` | Profit remains 0. |

---

## 13. Interview Explanation

**Speaker Script:**
> "To find the maximum profit from a single buy and sell transaction, my objective is to find two days $i$ and $j$ such that we buy on day $i$ and sell on day $j$ (where $j > i$) to maximize `prices[j] - prices[i]`.
> A brute-force solution would check all pairs of days, which takes $O(N^2)$ time.
> We can optimize this to $O(N)$ time and $O(1)$ space in a single pass. As we iterate through the prices from left to right, we maintain the lowest price we have seen so far. At each day, we calculate the potential profit if we sold our stock today (which is the current price minus the minimum price seen so far). 
> If this profit is greater than our maximum profit recorded, we update our maximum profit. Then, we update our minimum price if today's price is even lower. This ensures we evaluate the best buy-sell combination in linear time."

---

## 14. Follow-up Questions

1. **How would you find the actual buy and sell days?**
   - *Answer:* We can maintain two indices: `buyDay` and `sellDay`. We update a temporary `minPriceDay` whenever we find a new minimum price. When `maxProfit` is updated, we set `buyDay = minPriceDay` and `sellDay = currentDay`.
2. **What if you can perform at most two transactions?**
   - *Answer:* This is a classic DP problem (Best Time to Buy and Sell Stock III). We can solve it using state transitions in $O(N)$ time and $O(1)$ space.

---

## 15. Variations

1. **Best Time to Buy and Sell Stock II:** Unlimited transactions.
2. **Best Time to Buy and Sell Stock III:** At most two transactions.
3. **Best Time to Buy and Sell Stock IV:** At most $K$ transactions.

---

## 16. Revision Notes

- **Concept:** Buy low, sell high (in that chronological order).
- **State Tracked:** `minPriceSoFar` and `maxProfit`.
- **Formula:** `profit = prices[i] - minPrice`
- **Complexity:** $O(N)$ Time, $O(1)$ Space.

---

## 17. Memorization Version

```cpp
// Time: O(N), Space: O(1)
int maxProfit(vector<int>& prices) {
    int minPrice = prices[0], maxProfit = 0;
    for (size_t i = 1; i < prices.size(); ++i) {
        maxProfit = max(maxProfit, prices[i] - minPrice);
        minPrice = min(minPrice, prices[i]);
    }
    return maxProfit;
}
```
