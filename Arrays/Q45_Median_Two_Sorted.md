# Find the Median of Two Sorted Arrays - Comprehensive Interview Preparation Module

## 1. Problem Statement
Given two sorted arrays `nums1` and `nums2` of size $m$ and $n$ respectively, return the median of the two sorted arrays.

The overall run time complexity should be $O(\log(m+n))$ or more specifically $O(\log(\min(m, n)))$.

### Input
- `nums1`: A sorted integer array of size $m$ ($0 \le m \le 1000$).
- `nums2`: A sorted integer array of size $n$ ($0 \le n \le 1000$).
- Both arrays are sorted in non-decreasing order.
- Total size: $1 \le m + n \le 2000$.

### Output
- A `double` value representing the median of the combined sorted arrays.

### Constraints
- $O(\log(\min(m, n)))$ Time Complexity.
- $O(1)$ Auxiliary Space.

### Observations
1. The median is the middle element if the total size is odd, or the average of the two middle elements if the total size is even.
2. A brute force merge takes $O(m + n)$ time and space.
3. To achieve logarithmic time, we must avoid merging. Instead, we can use binary search to find a partition.

---

## 2. Interview Clarification
Before coding, a candidate should clarify:
1. **Can one of the arrays be empty?**
   - *Answer:* Yes, either $m$ or $n$ can be 0, but not both ($m + n \ge 1$).
2. **What should we return if the total number of elements is even?**
   - *Answer:* Return the average of the two middle numbers as a `double`.
3. **Are duplicate elements allowed?**
   - *Answer:* Yes, both arrays can contain duplicates and identical values.

---

## 3. Thinking Process
Let's build the optimal partition intuition step-by-step:

### Goal
We want to split the combined elements of `nums1` and `nums2` into two halves: a Left Half and a Right Half, such that:
1. $\text{Size of Left Half} = \lfloor \frac{m + n + 1}{2} \rfloor$.
2. Every element in the Left Half is less than or equal to every element in the Right Half.

### The Partition Approach
Suppose we partition `nums1` at index $i$, splitting it into:
- Left: `nums1[0 ... i-1]` (size $i$)
- Right: `nums1[i ... m-1]` (size $m-i$)

To maintain the correct total left-half size, the partition index $j$ in `nums2` is uniquely determined:
$$i + j = \lfloor \frac{m + n + 1}{2} \rfloor \implies j = \lfloor \frac{m + n + 1}{2} \rfloor - i$$

Let's define the boundary elements:
- `leftA` = `nums1[i-1]` (largest element on left of `nums1`)
- `rightA` = `nums1[i]` (smallest element on right of `nums1`)
- `leftB` = `nums2[j-1]` (largest element on left of `nums2`)
- `rightB` = `nums2[j]` (smallest element on right of `nums2`)

```
nums1:  [ ... leftA ] | [ rightA ... ]
nums2:  [ ... leftB ] | [ rightB ... ]
```

### The Conditions for a Valid Partition
For the partition to be valid, all left elements must be $\le$ all right elements. Since `nums1` and `nums2` are individually sorted:
1. `leftA <= rightA` is already true.
2. `leftB <= rightB` is already true.
3. We only need to ensure cross-conditions:
   - **`leftA <= rightB`**
   - **`leftB <= rightA`**

### Binary Search Strategy
If these cross-conditions are not met, we adjust the binary search interval for $i$ in `nums1`:
- If `leftA > rightB`: $i$ is too large. We must decrease $i$ (move left: `high = i - 1`).
- If `leftB > rightA`: $i$ is too small. We must increase $i$ (move right: `low = i + 1`).

To ensure we don't index out of bounds, we define boundary values as $\infty$ or $-\infty$:
- If $i == 0$: `leftA` = $-\infty$
- If $i == m$: `rightA` = $+\infty$
- If $j == 0$: `leftB` = $-\infty$
- If $j == n$: `rightB` = $+\infty$

### Ensuring $O(\log(\min(m, n)))$
If we search on the larger array, the calculated index $j$ for the smaller array could become negative or out of bounds. To avoid this and ensure minimum time complexity, we **always perform the binary search on the smaller array**. If $m > n$, we swap the arrays.

---

## 4. Pattern Recognition
- **Topic:** Advanced Binary Search
- **Pattern:** Binary Search on Solution Space / Partitioning
- **Why this pattern?** Instead of searching for a specific value, we are searching for a "split point" (partition index) that satisfies a mathematical balance condition. This is a classic pattern for optimization problems on sorted data.

---

## 5. Brute Force Solution (Merge and Find)

### Algorithm
1. Merge the two sorted arrays into a single sorted array of size $m+n$ using the two-pointer merge step of Merge Sort.
2. If $(m+n)$ is odd, return the element at index $(m+n)/2$.
3. If $(m+n)$ is even, return the average of the elements at $(m+n)/2 - 1$ and $(m+n)/2$.

### Complexity Analysis
- **Time Complexity:** $O(m + n)$ to merge the arrays.
- **Space Complexity:** $O(m + n)$ to store the merged array.

### Pros and Cons
- **Pros:** Simple, highly readable, works without complex edge-case logic.
- **Cons:** Violates the $O(\log(m+n))$ constraint and uses linear auxiliary space.

### C++17 Brute Force Code
```cpp
#include <vector>

class MedianBrute {
public:
    double findMedianSortedArrays(std::vector<int>& nums1, std::vector<int>& nums2) {
        int m = nums1.size();
        int n = nums2.size();
        std::vector<int> merged;
        merged.reserve(m + n);
        
        int i = 0, j = 0;
        while (i < m && j < n) {
            if (nums1[i] <= nums2[j]) {
                merged.push_back(nums1[i++]);
            } else {
                merged.push_back(nums2[j++]);
            }
        }
        
        while (i < m) merged.push_back(nums1[i++]);
        while (j < n) merged.push_back(nums2[j++]);
        
        int total = m + n;
        if (total % 2 != 0) {
            return merged[total / 2];
        } else {
            return (merged[total / 2 - 1] + merged[total / 2]) / 2.0;
        }
    }
};
```

---

## 6. Better Solution (Two-Pointer Space Optimization)

### Algorithm
Instead of creating a new merged array, we can use two pointers to count up to the median indices. We track the current element and the previous element during our traversal.

### Complexity Analysis
- **Time Complexity:** $O(m + n)$
- **Space Complexity:** $O(1)$ auxiliary space since we only keep a few tracking variables.

### C++17 Better Code
```cpp
#include <vector>
#include <algorithm>

class MedianBetter {
public:
    double findMedianSortedArrays(std::vector<int>& nums1, std::vector<int>& nums2) {
        int m = nums1.size();
        int n = nums2.size();
        int total = m + n;
        
        int i = 0, j = 0;
        int m1 = 0, m2 = 0; // To store middle elements
        
        for (int count = 0; count <= total / 2; ++count) {
            m2 = m1; // Store previous value
            if (i < m && (j >= n || nums1[i] <= nums2[j])) {
                m1 = nums1[i++];
            } else {
                m1 = nums2[j++];
            }
        }
        
        if (total % 2 != 0) {
            return m1;
        } else {
            return (m1 + m2) / 2.0;
        }
    }
};
```

---

## 7. Optimal Solution (Binary Search on Partitions)

### Algorithm
1. Check if `nums1` is smaller than `nums2`. If not, recursively call the function swapping the arguments. This ensures we perform binary search on the smaller array.
2. Initialize binary search bounds on the smaller array: `low = 0`, `high = m`.
3. While `low <= high`:
   - Calculate partition indices:
     - $i = \text{low} + (\text{high} - \text{low}) / 2$
     - $j = (m + n + 1) / 2 - i$
   - Get boundary values (handling $-\infty$ and $+\infty$ conditions):
     - `leftA` = $(i == 0) ? \text{INT\_MIN} : nums1[i-1]$
     - `rightA` = $(i == m) ? \text{INT\_MAX} : nums1[i]$
     - `leftB` = $(j == 0) ? \text{INT\_MIN} : nums2[j-1]$
     - `rightB` = $(j == n) ? \text{INT\_MAX} : nums2[j]$
   - If `leftA <= rightB` and `leftB <= rightA` (Correct Partition):
     - If total elements $(m + n)$ is odd, return $\max(leftA, leftB)$.
     - If total elements $(m + n)$ is even, return $\frac{\max(leftA, leftB) + \min(rightA, rightB)}{2.0}$.
   - Else if `leftA > rightB`:
     - $i$ is too big, move left: `high = i - 1`.
   - Else:
     - $i$ is too small, move right: `low = i + 1`.
4. Return `0.0` as a fallback (should not be reached if inputs are valid).

### Dry Run
`nums1 = [1, 3]`, `nums2 = [2]`. Sizes: $m = 2, n = 1$. Total size $= 3$ (odd).
- Search array is `nums2` because $n < m$. Swap them:
  - `A = [2]` (size $m = 1$)
  - `B = [1, 3]` (size $n = 2$)
- Target size for left half: $(1 + 2 + 1) / 2 = 2$.
- Binary search range for $i$: `low = 0, high = 1`.

#### Iteration 1:
- $i = (0 + 1) / 2 = 0$.
- $j = 2 - 0 = 2$.
- Boundary values:
  - `leftA` = $(i==0) ? -\infty = -\infty$
  - `rightA` = `A[0] = 2`
  - `leftB` = `B[1] = 3`
  - `rightB` = $(j==2) ? +\infty = +\infty$
- Check conditions:
  - `leftA <= rightB` $\to -\infty \le \infty$ (True)
  - `leftB <= rightA` $\to 3 \le 2$ (False!)
- Since `leftB > rightA` ($3 > 2$), $i$ is too small. We need to move right.
- `low = i + 1 = 1`.

#### Iteration 2:
- `low = 1, high = 1`.
- $i = 1$.
- $j = 2 - 1 = 1$.
- Boundary values:
  - `leftA` = `A[0] = 2`
  - `rightA` = $(i==1) ? +\infty = +\infty$
  - `leftB` = `B[0] = 1`
  - `rightB` = `B[1] = 3`
- Check conditions:
  - `leftA <= rightB` $\to 2 \le 3$ (True)
  - `leftB <= rightA` $\to 1 \le \infty$ (True)
- Partition is correct!
- Since total size $3$ is odd, Median = $\max(leftA, leftB) = \max(2, 1) = 2.0$.

### Why Optimal
- We halve the search space of the smaller array at each step, resulting in a time complexity of $O(\log(\min(m, n)))$.
- No extra space is allocated.

### C++17 Optimal Code
```cpp
#include <vector>
#include <algorithm>
#include <climits>

class MedianOptimal {
public:
    double findMedianSortedArrays(std::vector<int>& nums1, std::vector<int>& nums2) {
        // Ensure nums1 is the smaller array
        if (nums1.size() > nums2.size()) {
            return findMedianSortedArrays(nums2, nums1);
        }
        
        int m = nums1.size();
        int n = nums2.size();
        
        int low = 0;
        int high = m;
        
        while (low <= high) {
            int partitionX = low + (high - low) / 2;
            int partitionY = (m + n + 1) / 2 - partitionX;
            
            // Boundary checks
            int maxLeftX = (partitionX == 0) ? INT_MIN : nums1[partitionX - 1];
            int minRightX = (partitionX == m) ? INT_MAX : nums1[partitionX];
            
            int maxLeftY = (partitionY == 0) ? INT_MIN : nums2[partitionY - 1];
            int minRightY = (partitionY == n) ? INT_MAX : nums2[partitionY];
            
            // Check if partition is correct
            if (maxLeftX <= minRightY && maxLeftY <= minRightX) {
                // If odd number of elements
                if ((m + n) % 2 != 0) {
                    return std::max(maxLeftX, maxLeftY);
                } 
                // If even number of elements
                else {
                    return (std::max(maxLeftX, maxLeftY) + std::min(minRightX, minRightY)) / 2.0;
                }
            } 
            else if (maxLeftX > minRightY) {
                // We are too far right in nums1, move left
                high = partitionX - 1;
            } 
            else {
                // We are too far left in nums1, move right
                low = partitionX + 1;
            }
        }
        
        return 0.0; // Fallback
    }
};
```

---

## 8. Code Walkthrough
- **Line 9-11:** If `nums1` is larger than `nums2`, swap them to make sure we binary search on the smaller array.
- **Line 16-17:** Standard binary search pointers initialized on `nums1` boundaries `[0, m]`.
- **Line 20-21:** Compute partitions. `partitionX` is $i$ and `partitionY` is $j$.
- **Line 24-28:** Safely retrieve values at `partition-1` and `partition`. If partitioning at boundary, use `INT_MIN` or `INT_MAX`.
- **Line 31:** Condition check `maxLeftX <= minRightY && maxLeftY <= minRightX`.
- **Line 33-39:** Depending on odd/even parity, extract the max of left boundaries and min of right boundaries to compute the median.
- **Line 41-43:** If `maxLeftX > minRightY`, the elements in left of `nums1` are too large. Shift search left (`high = partitionX - 1`).
- **Line 45-47:** Else, left of `nums2` has too large elements, shift search right (`low = partitionX + 1`).

---

## 9. Dry Run
`nums1 = [7, 12, 14, 15]`, `nums2 = [1, 2, 3, 8, 9, 10]`.
$m = 4, n = 6$. Total $= 10$ (Even). Partition size target $= 5$.

| `low` | `high` | `partX` | `partY` | `maxLeftX` | `minRightX` | `maxLeftY` | `minRightY` | Decision |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 0 | 4 | 2 | 3 | `nums1[1]=12` | `nums1[2]=14` | `nums2[2]=3` | `nums2[3]=8` | `maxLeftX (12) > minRightY (8)` $\to$ `high = 1` |
| 0 | 1 | 0 | 5 | `INT_MIN` | `nums1[0]=7` | `nums2[4]=9` | `nums2[5]=10` | `maxLeftY (9) > minRightX (7)` $\to$ `low = 1` |
| 1 | 1 | 1 | 4 | `nums1[0]=7` | `nums1[1]=12` | `nums2[3]=8` | `nums2[4]=9` | `maxLeftX(7)<=minRightY(9)` AND `maxLeftY(8)<=minRightX(12)` $\to$ Match! |

Result calculations:
- `maxLeft = max(7, 8) = 8`
- `minRight = min(12, 9) = 9`
- Median $= (8 + 9) / 2.0 = 8.5$.

---

## 10. Complexity Analysis
| Approach | Time Complexity | Auxiliary Space Complexity | Reasoning |
| :--- | :--- | :--- | :--- |
| **Brute Force (Merge)** | $O(m + n)$ | $O(m + n)$ | Merging both arrays requires copying all elements. |
| **Two-Pointer (Scan)** | $O(m + n)$ | $O(1)$ | Scan up to half the elements without extra storage. |
| **Optimal (Binary Search)** | $O(\log(\min(m, n)))$ | $O(1)$ | Binary search performed on the smaller array. |

---

## 11. Common Mistakes
1. **Not swapping inputs:** Performing binary search on `nums1` without ensuring it's the smaller array. This causes `partitionY` to go out of bounds.
2. **Off-by-one in partition formula:** Forgetting the `+ 1` in `(m + n + 1) / 2`. The `+1` handles both odd and even lengths correctly by ensuring the left half gets the extra element in the odd case.
3. **Improper boundary values:** Not using `INT_MIN` or `INT_MAX` for partition edges, causing segmentation faults.

---

## 12. Edge Cases
| Edge Case | Input Example | Expected Output | Why it Matters |
| :--- | :--- | :--- | :--- |
| **One empty array** | `[]`, `[2, 3]` | `2.5` | Checks boundary logic when `partitionX == 0` or `partitionY == 0`. |
| **No overlapping ranges**| `[1, 2]`, `[3, 4]` | `2.5` | Validates binary search boundaries moving completely to one side. |
| **Single element each**| `[1]`, `[2]` | `1.5` | Smallest even case. |
| **Identical elements** | `[1, 1]`, `[1, 1]` | `1.0` | Tests stability under duplicate comparisons. |

---

## 13. Interview Explanation
> "To find the median of two sorted arrays in logarithmic time, we use a binary search partition method on the smaller of the two arrays. 
> Instead of merging, we partition both arrays into left and right halves such that the left halves of both arrays combine to contain exactly half of the total elements.
> We then check the boundary conditions: the largest element on the left of array A must be less than or equal to the smallest element on the right of array B, and vice-versa.
> If this holds, we have partitioned the combined arrays correctly, and we can compute the median in $O(1)$ time from the boundary elements. If not, we adjust our partition using binary search on the smaller array. This ensures a time complexity of $O(\log(\min(m, n)))$ and $O(1)$ auxiliary space."

---

## 14. Follow-up Questions
1. **Can this approach be extended to find the K-th smallest element of two sorted arrays?**
   - *Answer:* Yes, the exact same partition logic can be used. Instead of target size $(m+n+1)/2$, we search for a partition of combined size $K$.
2. **What if the arrays are not sorted?**
   - *Answer:* We would need to sort them first ($O(N \log N)$), or use the QuickSelect algorithm which takes $O(N)$ average time but is not guaranteed $O(\log N)$.

---

## 15. Variations
1. **K-th Element of Two Sorted Arrays:**
   - Same binary partition algorithm, but partition target is $K$ instead of $(m+n+1)/2$.
2. **Median of $K$ Sorted Arrays:**
   - Usually solved using a min-heap in $O(N \log K)$ time.

---

## 16. Revision Notes
- **Binary Search on Smaller:** Always search on `nums1` where $m \le n$.
- **Partition Relation:** $j = (m + n + 1) / 2 - i$.
- **Even/Odd Medians:** Max of lefts for odd; average of max-left/min-right for even.

---

## 17. Memorization Version
- Ensure `nums1` is smaller.
- Binary search `low = 0, high = m`.
- `partX = mid`, `partY = (m+n+1)/2 - partX`.
- Check if `maxLeftX <= minRightY` and `maxLeftY <= minRightX`.
- If yes: return `max(maxLeftX, maxLeftY)` (if odd) or `(max+min)/2.0` (if even).
- If `maxLeftX > minRightY` move left: `high = partX - 1`. Else move right.
