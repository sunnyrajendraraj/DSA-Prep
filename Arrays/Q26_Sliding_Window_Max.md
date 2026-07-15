# Sliding Window Maximum

## 1. Problem Statement

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

---

## 2. Interview Clarification

- **Q:** Can `K` be greater than `N`?
  - **A:** No, standard constraints state $1 \le K \le N$. If $K > N$, we can return the maximum of the entire array.
- **Q:** What should we return if `K = 1`?
  - **A:** The output array should be identical to the input array.
- **Q:** Are there duplicate values in the array?
  - **A:** Yes, the array can contain duplicates.
- **Q:** Can we use standard library templates (STL)?
  - **A:** Yes, using standard containers like `std::deque` or `std::multiset` is expected.

---

## 3. Thinking Process

### First Instinct (Brute Force)
For each window of size `K`, iterate through all its elements to find the maximum.
- Start index `i` ranges from `0` to `N - K`.
- In a nested loop, find the maximum of `arr[i]` through `arr[i + K - 1]`.
- Store this maximum in the result.
- This takes $O(N \cdot K)$ time, which is too slow when $N, K \approx 10^5$.

### Improving with BST / Heaps (Better Solution)
Can we keep track of the elements in the window using a sorted structure?
- If we use a Max-Heap, we can get the maximum in $O(1)$ time. However, removing an element that leaves the window from the middle of a standard binary heap is not supported in $O(\log K)$ unless we use lazy deletion.
- Alternatively, we can use a Self-balancing Binary Search Tree (BST) like `std::multiset` in C++.
  - Inserting a new element takes $O(\log K)$.
  - Deleting the element that falls out of the window takes $O(\log K)$.
  - Finding the maximum element takes $O(1)$ (the last element in the multiset).
- This achieves $O(N \log K)$ time complexity.

### Achieving Linear Time (Optimal Insight)
Can we do this in $O(N)$?
Let's think: if we have an element `arr[i]` and a new element `arr[j]` enters the window where `j > i` and `arr[j] >= arr[i]`, then `arr[i]` can **never** be the maximum for the current or any future window. Why? Because `arr[j]` is larger than or equal to `arr[i]` and will stay in the sliding window longer (since it entered later).
- This means we can discard `arr[i]` immediately.
- This observation suggests maintaining a **Monotonic Decreasing Queue** (using a `std::deque`).
- The deque will store indices (not values) so we can easily check if the element at the front has fallen out of the current window.
- When a new element `arr[i]` enters:
  1. Remove indices from the front of the deque if they are out of the current window boundary (`deque.front() <= i - K`).
  2. Remove indices from the back of the deque as long as the elements they point to are less than or equal to `arr[i]`.
  3. Push the current index `i` to the back of the deque.
  4. The element at the index stored at the front of the deque (`arr[deque.front()]`) is the maximum for the current window.

---

## 4. Pattern Recognition

- **Pattern:** Sliding Window + Monotonic Queue (Deque).
- **Why this topic?** The window slides sequentially, and we need to query a range-maximum query dynamically. A monotonic deque allows us to insert and delete elements while maintaining the maximum element at the front in amortized $O(1)$ time.

---

## 5. Brute Force Solution

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

---

## 6. Better Solution (Balanced BST / Multiset)

### Algorithm
1. Use a `std::multiset<int>` to store the elements of the current window.
2. Insert the first `K` elements into the multiset.
3. The largest element is the last element in the multiset, accessed via `*multiset.rbegin()`. Add it to the result.
4. For each subsequent element from index `K` to `N - 1`:
   - Find and erase one instance of `arr[i - K]` (the element leaving the window). Note: use `ms.erase(ms.find(arr[i-k]))` instead of `ms.erase(arr[i-k])` to delete only one instance if there are duplicates.
   - Insert `arr[i]` (the element entering the window).
   - Get the maximum using `*ms.rbegin()` and add it to the result.

### Complexity Analysis
- **Time Complexity:** $O(N \log K)$ because each insertion and deletion takes $O(\log K)$ time.
- **Space Complexity:** $O(K)$ to store the elements of the window in the multiset.

### Better C++17 Code
```cpp
#include <vector>
#include <set>
#include <algorithm>

class Solution {
public:
    std::vector<int> maxSlidingWindowBetter(const std::vector<int>& arr, int k) {
        int n = arr.size();
        if (n == 0 || k == 0) return {};
        
        std::vector<int> result;
        std::multiset<int> window;
        
        // Initialize the first window
        for (int i = 0; i < k; ++i) {
            window.insert(arr[i]);
        }
        result.push_back(*window.rbegin());
        
        // Slide the window
        for (int i = k; i < n; ++i) {
            // Remove the outgoing element
            auto it = window.find(arr[i - k]);
            window.erase(it);
            
            // Insert the incoming element
            window.insert(arr[i]);
            
            // Get the maximum
            result.push_back(*window.rbegin());
        }
        return result;
    }
};
```

---

## 7. Optimal Solution (Monotonic Deque)

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

---

## 8. Code Walkthrough

1. **Window Expiry Check**: `dq.front() == i - k`. Since we increment `i` one by one, only one element can fall out of the window at each step. If the front of our deque equals `i - k`, it is no longer in the window `[i - k + 1, i]`, so we remove it.
2. **Monotonic Maintenance**: `while (!dq.empty() && arr[dq.back()] <= arr[i])`. We compare the incoming element `arr[i]` with elements from the back of the deque. Any element in the deque that is smaller than or equal to `arr[i]` is popped because it has no chance of being the maximum now that a larger element has entered the window.
3. **Insertion**: `dq.push_back(i)`. We insert the index `i` of the current element.
4. **Recording Maximum**: `if (i >= k - 1)`. Once we have visited the first `k` elements, every step represents a completed window, and the maximum is always at `arr[dq.front()]`.

---

## 9. Dry Run

Let's dry run the optimal solution on `arr = [1, 3, -1, -3, 5, 3]`, `K = 3`.

| `i` | `arr[i]` | Deque Contents (Indices) | Deque Values | Window | Result Added | Action Notes |
|:---:|:---:|:---|:---|:---|:---:|:---|
| 0 | 1 | `[0]` | `[1]` | - | - | Push index 0. |
| 1 | 3 | `[1]` | `[3]` | - | - | `arr[1]=3 >= arr[0]=1` $\rightarrow$ pop 0, push index 1. |
| 2 | -1 | `[1, 2]` | `[3, -1]` | `[1, 3, -1]` | **3** | Push 2. Window complete. Max is `arr[1] = 3`. |
| 3 | -3 | `[1, 2, 3]` | `[3, -1, -3]` | `[3, -1, -3]` | **3** | Push 3. Max is `arr[1] = 3`. |
| 4 | 5 | `[4]` | `[5]` | `[-1, -3, 5]` | **5** | Index 1 falls out (`i - k = 1`). Pop 1. Then pop 2, 3 (smaller than 5). Push 4. Max is `arr[4] = 5`. |
| 5 | 3 | `[4, 5]` | `[5, 3]` | `[-3, 5, 3]` | **5** | Push 5. Max is `arr[4] = 5`. |

**Final Result:** `[3, 3, 5, 5]`

---

## 10. Complexity Analysis

- **Time Complexity:** 
  - **Best Case:** $O(N)$ (e.g., strictly decreasing array, where we do not do any back popping).
  - **Average Case:** $O(N)$.
  - **Worst Case:** $O(N)$ (e.g., strictly increasing array, where we pop all previous elements, but each element is still pushed and popped at most once).
- **Space Complexity:**
  - **Auxiliary Space:** $O(K)$ to store the indices in the deque.

---

## 11. Common Mistakes

1. **Storing values instead of indices in the deque**: Storing values makes it difficult to check whether the element at the front of the deque has fallen out of the sliding window. Storing indices is much cleaner.
2. **Strictly less vs less-than-or-equal**: When checking `arr[dq.back()] <= arr[i]`, using `<` instead of `<=` is functionally correct, but keeping duplicate equal elements in the deque is redundant and uses unnecessary space.
3. **Off-by-one errors in window boundary check**: Removing elements from the front when `dq.front() < i - k + 1` (or `dq.front() == i - k`). Make sure the indices are checked correctly.

---

## 12. Edge Cases

| Edge Case | Input Example | Expected Output | Why it matters |
|:---|:---|:---|:---|
| **K = 1** | `[2, 5, -1]`, `K = 1` | `[2, 5, -1]` | Output size equals input size. |
| **K = N** | `[1, 3, 2]`, `K = 3` | `[3]` | Output has a single element. |
| **Decreasing Array** | `[5, 4, 3, 2]`, `K = 2` | `[5, 4, 3]` | Deque front is never popped from the back. |
| **Increasing Array** | `[1, 2, 3, 4]`, `K = 2` | `[2, 3, 4]` | Deque is popped from the back at every step. |

---

## 13. Interview Explanation

"To find the sliding window maximum efficiently, the brute-force approach of checking every element in each window takes $O(N \cdot K)$ time. We can improve this to $O(N \log K)$ by maintaining the window elements in a balanced BST like a multiset.

However, the optimal approach is to use a **Monotonic Deque** that runs in $O(N)$ time. The key observation is that if an element inside the window is smaller than a newly arrived element to its right, that smaller element can never be the maximum of the current or any future window because the new, larger element will outlast it. 

Therefore, as we iterate through the array:
1. We pop indices from the front of the deque if they fall outside the sliding window.
2. We pop indices from the back of the deque if their corresponding values are smaller than or equal to the current element.
3. We push the current index onto the back.
4. The maximum of the window is always at the front of the deque. 

Since each element is pushed and popped at most once, the time complexity is $O(N)$ and the space complexity is $O(K)$."

---

## 14. Follow-up Questions

- **Q:** How would you modify the solution to find the sliding window **minimum**?
  - **A:** We change the comparison from `arr[dq.back()] <= arr[i]` to `arr[dq.back()] >= arr[i]`. This maintains a monotonic increasing queue where the front element is always the minimum.
- **Q:** Can we solve this problem using two stacks?
  - **A:** Yes, we can implement a Queue using two stacks (similar to the Queue using Stacks problem) and store the running maximums in each stack. This is called the **sliding window aggregation** technique, which also achieves $O(N)$ time.

---

## 15. Variations

- **Sliding Window Median:** Requires maintaining two heaps (min-heap and max-heap) or a balanced BST with a middle iterator.
- **Shortest Subarray with Sum at Least K:** Uses a monotonic deque to find prefix sum differences.

---

## 16. Revision Notes

- **Core Data Structure:** `std::deque<int>` storing indices.
- **Queue Monotonicity:** Kept in decreasing order. Front contains the index of the max element.
- **Removal Conditions:** 
  - Front element is out of window: `dq.front() == i - k`.
  - Back element is smaller than new element: `arr[dq.back()] <= arr[i]`.
- **Complexity:** $O(N)$ Time, $O(K)$ Space.

---

## 17. Memorization Version

- **Loop** `i` from `0` to `N - 1`:
  - `if (!dq.empty() && dq.front() == i - k) dq.pop_front();`
  - `while (!dq.empty() && arr[dq.back()] <= arr[i]) dq.pop_back();`
  - `dq.push_back(i);`
  - `if (i >= k - 1) result.push_back(arr[dq.front()]);`
- **Complexity:** $O(N)$ time, $O(K)$ space.
