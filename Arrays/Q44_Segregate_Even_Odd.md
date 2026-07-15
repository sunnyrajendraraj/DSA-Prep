# Segregate Even and Odd Numbers - Comprehensive Interview Preparation Module

## 1. Problem Statement
Given an integer array `arr`, rearrange its elements in-place such that all even numbers are placed on the left side of the array, and all odd numbers are placed on the right side.

We need to cover two versions:
1. **Unstable Segregation:** Order of elements does not matter, in-place $O(N)$ time, $O(1)$ auxiliary space.
2. **Stable Segregation:** The relative order of even elements among themselves and odd elements among themselves must be maintained.

### Input
- An integer array `arr` of size $N$ ($1 \le N \le 10^6$).
- Elements can be positive, negative, or zero.

### Output
- The array modified in-place to contain even numbers first, followed by odd numbers.

### Constraints
- Unstable: Time Complexity $O(N)$, Space Complexity $O(1)$
- Stable: Analyze space-time trade-offs.

---

## 2. Interview Clarification
Before coding, a candidate should clarify:
1. **Does the relative order of numbers matter?**
   - *Answer:* For the primary problem, relative order does not matter. However, be prepared to discuss the stable version.
2. **Does the array contain negative numbers?**
   - *Answer:* Yes. Note that in C++, negative even numbers (like `-2`, `-4`) satisfy `x % 2 == 0`, and negative odd numbers (like `-3`) satisfy `x % 2 != 0` (in C++, `-3 % 2` is `-1`).
3. **What is the definition of even/odd for negative numbers?**
   - *Answer:* Any integer divisible by 2 is even. Negative integers are even if `x % 2 == 0` and odd if `x % 2 != 0`.

---

## 3. Thinking Process
Let's build the intuition for both versions:

### Unstable Version (Two-Pointer Approach)
- We have two classes of elements: even and odd. This is a binary partitioning problem, similar to QuickSelect partition or a simplified Dutch National Flag (which partitions into 3 classes).
- We can maintain two pointers:
  - `left` starting at `0` (tracks the boundary of even numbers).
  - `right` starting at `N - 1` (tracks the boundary of odd numbers).
- While `left < right`:
  - If `arr[left]` is even, increment `left` (it's in the correct partition).
  - If `arr[right]` is odd, decrement `right` (it's in the correct partition).
  - If `arr[left]` is odd and `arr[right]` is even, swap them, then increment `left` and decrement `right`.

### Stable Version (Maintaining Relative Order)
- If we must preserve the relative order, a simple swap of elements from the ends will destroy order.
- **Approach 1 (Extra Space):** Create an auxiliary array. Copy all even numbers, then copy all odd numbers. This is $O(N)$ time and $O(N)$ space.
- **Approach 2 (In-Place but slow - Insertion Sort Style):** For every odd number, find the next even number and shift elements. This takes $O(N^2)$ time and $O(1)$ space.
- **Approach 3 (Divide & Conquer - Merge Sort Style):**
  - Divide the array into two halves.
  - Recursively segregate left and right halves.
  - Merge the two halves. To merge stably in $O(1)$ extra space, we need to rotate a subarray.
    - Suppose left half segregated is `[E1, E2, O1, O2]` and right half is `[E3, E4, O3, O4]`.
    - We need to transform this to `[E1, E2, E3, E4, O1, O2, O3, O4]`.
    - This is done by reversing the middle subarray `[O1, O2]` and `[E3, E4]` to rotate the segments.
    - Time complexity: $O(N \log N)$, Space complexity: $O(\log N)$ recursion stack or $O(1)$ if done iteratively.

---

## 4. Pattern Recognition
- **Topic:** Two-Pointer Partitioning
- **Pattern:** Binary Partitioning / Classification
- **Why this pattern?** We are dividing elements into two disjoint sets (Even/Odd) based on a boolean property. The two-pointer approach avoids extra memory by swapping mismatched elements from both ends.

---

## 5. Brute Force Solution (Stable Version - Extra Space)

### Algorithm
1. Create an auxiliary vector `result` of size $N$.
2. Traverse the input array and copy all even elements to `result`.
3. Traverse the input array again and copy all odd elements to `result`.
4. Copy `result` back to the original array.

### Dry Run
Input: `arr = [1, 2, 4, 3, 5, 8]`
- Pass 1 (Even): `result = [2, 4, 8]`
- Pass 2 (Odd): `result = [2, 4, 8, 1, 3, 5]`
- Copy back: `arr = [2, 4, 8, 1, 3, 5]`
- Relative order of evens `(2, 4, 8)` and odds `(1, 3, 5)` is perfectly maintained.

### Complexity Analysis
- **Time Complexity:** $O(N)$ as we make two linear scans.
- **Space Complexity:** $O(N)$ auxiliary space for the `result` vector.

### Pros and Cons
- **Pros:** Preserves relative order (Stable); easy to write.
- **Cons:** Uses $O(N)$ extra memory, which is sub-optimal for constraints requiring $O(1)$ space.

### C++17 Stable Brute Force Code
```cpp
#include <vector>

class SegregateStableBrute {
public:
    void segregateEvenOdd(std::vector<int>& arr) {
        std::vector<int> result;
        result.reserve(arr.size());
        
        // Step 1: Append even numbers
        for (int num : arr) {
            if (num % 2 == 0) {
                result.push_back(num);
            }
        }
        
        // Step 2: Append odd numbers
        for (int num : arr) {
            if (num % 2 != 0) {
                result.push_back(num);
            }
        }
        
        // Step 3: Copy back to original array
        arr = std::move(result);
    }
};
```

---

## 6. Better Solution (Stable Version - In-Place Divide & Conquer)

### Algorithm
To keep it stable and in-place, we use the Divide and Conquer strategy:
1. Divide the array into two halves: `[left, mid]` and `[mid+1, right]`.
2. Segregate both halves recursively.
3. Merge the two segregated halves:
   - Let left half be `[E_L... O_L...]` and right half be `[E_R... O_R...]`.
   - We need to swap the odd block of the left half with the even block of the right half.
   - This can be achieved by rotating the subarray containing `[O_L...]` and `[E_R...]`.
   - To rotate a block of size $K$ in-place: reverse the first part, reverse the second part, then reverse the whole block.

### Complexity Analysis
- **Time Complexity:** $O(N \log N)$ since at each level of recursion we do a rotation of size up to $O(N)$, and recursion depth is $O(\log N)$.
- **Space Complexity:** $O(\log N)$ due to recursive call stack.

### C++17 Stable Better Code
```cpp
#include <vector>
#include <algorithm>

class SegregateStableInPlace {
private:
    // Helper function to reverse a subarray
    void reverseSubarray(std::vector<int>& arr, int start, int end) {
        while (start < end) {
            std::swap(arr[start], arr[end]);
            start++;
            end--;
        }
    }

    // Helper to rotate arr[start...end] such that arr[start...mid] and arr[mid+1...end] are swapped
    void rotate(std::vector<int>& arr, int start, int mid, int end) {
        reverseSubarray(arr, start, mid);
        reverseSubarray(arr, mid + 1, end);
        reverseSubarray(arr, start, end);
    }

    void merge(std::vector<int>& arr, int start, int mid, int end) {
        // Find the first odd number in left half
        int first_odd_idx = start;
        while (first_odd_idx <= mid && arr[first_odd_idx] % 2 == 0) {
            first_odd_idx++;
        }

        // Find the first odd number in right half
        int first_odd_right_idx = mid + 1;
        while (first_odd_right_idx <= end && arr[first_odd_right_idx] % 2 == 0) {
            first_odd_right_idx++;
        }

        // Rotate the subarray containing odd numbers of left half and even numbers of right half
        // Subarray to rotate: from first_odd_idx to first_odd_right_idx - 1
        // mid is the partition point
        if (first_odd_idx <= mid && first_odd_right_idx - 1 > mid) {
            rotate(arr, first_odd_idx, mid, first_odd_right_idx - 1);
        }
    }

    void divideAndConquer(std::vector<int>& arr, int start, int end) {
        if (start >= end) return;
        
        int mid = start + (end - start) / 2;
        divideAndConquer(arr, start, mid);
        divideAndConquer(arr, mid + 1, end);
        merge(arr, start, mid, end);
    }

public:
    void segregateEvenOdd(std::vector<int>& arr) {
        divideAndConquer(arr, 0, arr.size() - 1);
    }
};
```

---

## 7. Optimal Solution (Unstable Version - $O(N)$ Time, $O(1)$ Space)

### Algorithm
1. Initialize two pointers: `left = 0` and `right = N - 1`.
2. While `left < right`:
   - Increment `left` while `arr[left]` is even and `left < right`.
   - Decrement `right` while `arr[right]` is odd and `left < right`.
   - If `left < right`, it means `arr[left]` is odd and `arr[right]` is even. Swap them.
   - Increment `left` and decrement `right` to continue partitioning.

### Dry Run
Input: `arr = [12, 34, 45, 9, 8, 90, 3]`

| Step | `left` | `right` | Array State | Description |
| :--- | :--- | :--- | :--- | :--- |
| Start | 0 | 6 | `[12, 34, 45, 9, 8, 90, 3]` | Initial state |
| 1 | 0 | 6 | `[12, 34, 45, 9, 8, 90, 3]` | `arr[left]=12` is even $\to$ `left++` to 1. |
| 2 | 1 | 6 | `[12, 34, 45, 9, 8, 90, 3]` | `arr[left]=34` is even $\to$ `left++` to 2. |
| 3 | 2 | 6 | `[12, 34, 45, 9, 8, 90, 3]` | `arr[left]=45` is odd. Stop left. |
| 4 | 2 | 6 | `[12, 34, 45, 9, 8, 90, 3]` | `arr[right]=3` is odd $\to$ `right--` to 5. |
| 5 | 2 | 5 | `[12, 34, 45, 9, 8, 90, 3]` | `arr[right]=90` is even. Stop right. |
| 6 | 2 | 5 | `[12, 34, 45, 9, 8, 90, 3]` | `left < right`. Swap `arr[2]` (45) & `arr[5]` (90). |
| | 3 | 4 | `[12, 34, 90, 9, 8, 45, 3]` | Array updated. `left` and `right` updated. |
| 7 | 3 | 4 | `[12, 34, 90, 9, 8, 45, 3]` | `arr[left]=9` is odd. Stop left. |
| 8 | 3 | 4 | `[12, 34, 90, 9, 8, 45, 3]` | `arr[right]=8` is even. Stop right. |
| 9 | 3 | 4 | `[12, 34, 90, 9, 8, 45, 3]` | Swap `arr[3]` (9) & `arr[4]` (8). |
| | 4 | 3 | `[12, 34, 90, 8, 9, 45, 3]` | `left > right`. Loop terminates. |

Final Output: `[12, 34, 90, 8, 9, 45, 3]` (Evens on left, Odds on right).

### Why Optimal
- We scan each element at most once.
- We do not use any extra array or auxiliary memory; partitioning is done fully in-place using standard pointers.

### C++17 Optimal Code
```cpp
#include <vector>
#include <utility>

class SegregateOptimal {
public:
    void segregateEvenOdd(std::vector<int>& arr) {
        int left = 0;
        int right = arr.size() - 1;
        
        while (left < right) {
            // Find the first odd number from left
            while (left < right && arr[left] % 2 == 0) {
                left++;
            }
            
            // Find the first even number from right
            while (left < right && arr[right] % 2 != 0) {
                right--;
            }
            
            // If pointers haven't crossed, swap mismatched elements
            if (left < right) {
                std::swap(arr[left], arr[right]);
                left++;
                right--;
            }
        }
    }
};
```

---

## 8. Code Walkthrough
- **Line 6-7:** Initialize `left` to start and `right` to end.
- **Line 9:** Run loop as long as `left < right`.
- **Line 11-13:** Increment `left` if `arr[left]` is even. The condition `left < right` prevents pointer out-of-bounds.
- **Line 16-18:** Decrement `right` if `arr[right]` is odd.
- **Line 21-25:** If they haven't crossed, `arr[left]` is odd and `arr[right]` is even. Swapping swaps them into their correct positions. Increment `left` and decrement `right` to prevent infinite loops.

---

## 9. Dry Run
Input: `[3, 5, 2, 4, 8]`

| Step | `left` | `right` | Action | Array State |
| :--- | :--- | :--- | :--- | :--- |
| Start | 0 | 4 | Initial | `[3, 5, 2, 4, 8]` |
| 1 | 0 | 4 | `arr[left]` is odd, `arr[right]` is even $\to$ Swap | `[8, 5, 2, 4, 3]` |
| 2 | 1 | 3 | `arr[left]` is odd, `arr[right]` is even $\to$ Swap | `[8, 4, 2, 5, 3]` |
| 3 | 2 | 2 | `left == right` $\to$ Exit | `[8, 4, 2, 5, 3]` |

Result: `[8, 4, 2, 5, 3]` (Evens on left: `8, 4, 2`, Odds on right: `5, 3`).

---

## 10. Complexity Analysis
| Approach | Time Complexity | Auxiliary Space Complexity | Reasoning |
| :--- | :--- | :--- | :--- |
| **Brute Force (Stable)** | $O(N)$ | $O(N)$ | Uses an auxiliary array to copy evens and odds. |
| **Divide & Conquer (Stable)** | $O(N \log N)$ | $O(\log N)$ | Split in halves, merge using in-place rotations. |
| **Two-Pointer (Optimal Unstable)** | $O(N)$ | $O(1)$ | Single pass with two pointers, elements swapped in-place. |

---

## 11. Common Mistakes
1. **Handling Negative Numbers:** Using `arr[left] % 2 == 1` to check for odd. In C++, negative odd numbers return `-1` for `% 2`. Always use `num % 2 != 0` or bitwise `(num & 1) != 0` to check for odd numbers.
2. **Missing Boundary checks:** Forgetting `left < right` inside the nested while loops, leading to `IndexOutOfBoundsException`.
3. **Infinite Loops:** Forgetting to increment `left` or decrement `right` after a swap.

---

## 12. Edge Cases
| Edge Case | Input Example | Expected Output | Why it Matters |
| :--- | :--- | :--- | :--- |
| **All Evens** | `[2, 4, 6]` | `[2, 4, 6]` | `left` pointer scans to the end, no swaps. |
| **All Odds** | `[1, 3, 5]` | `[1, 3, 5]` | `right` pointer scans to the start, no swaps. |
| **Already Segregated** | `[2, 4, 1, 3]` | `[2, 4, 1, 3]` | Verifies algorithm does not perform unnecessary swaps. |
| **Negative Numbers** | `[-3, -2, 4]` | `[4, -2, -3]` or `[-2, 4, -3]` | Tests correctness of modulo operation for negative signs. |
| **Single Element** | `[7]` | `[7]` | Immediate termination without errors. |

---

## 13. Interview Explanation
> "To segregate even and odd numbers in-place with optimal space complexity, we can use a two-pointer approach. We place one pointer, `left`, at the beginning of the array, and another, `right`, at the end.
> The `left` pointer moves forward as long as it points to an even number, and the `right` pointer moves backward as long as it points to an odd number. When `left` is pointing to an odd number and `right` is pointing to an even number, we swap them. We repeat this until the two pointers cross.
> This partitions the array in a single pass taking $O(N)$ time and $O(1)$ space. Note that this is unstable. If a stable partition is required, we can either use $O(N)$ extra space, or use a Divide and Conquer approach with in-place rotation which runs in $O(N \log N)$ time."

---

## 14. Follow-up Questions
1. **How can you implement this with bitwise operations?**
   - *Answer:* Use `(arr[i] & 1) == 0` for even numbers, which is faster than division/modulo.
2. **Can we make the stable version $O(N)$ time and $O(1)$ space?**
   - *Answer:* Yes, but it is mathematically complex (e.g., using block-swap algorithms or stable merge algorithms), which are not typically expected in standard 45-minute interviews.
3. **What standard C++ library functions can do this?**
   - *Answer:* `std::partition` (unstable) and `std::stable_partition` (stable).

---

## 15. Variations
1. **Segregate 0s and 1s:**
   - Same binary partition, replace `x % 2 == 0` with `x == 0`.
2. **Dutch National Flag (0s, 1s, and 2s):**
   - Requires 3 pointers (`low`, `mid`, `high`) to create 3 partitions.
3. **Move Zeroes to End:**
   - Keep relative order of non-zeroes: Stable partition where zeroes are placed at the end.

---

## 16. Revision Notes
- **Odd Check:** `(num & 1) != 0` is robust for negatives and positives.
- **Two-Pointer partition:** Unstable, $O(N)$ time, $O(1)$ space.
- **Stable partition:** $O(N)$ time and $O(N)$ space or $O(N \log N)$ in-place.

---

## 17. Memorization Version
- Set `l = 0`, `r = n - 1`.
- While `l < r`: increment `l` if even, decrement `r` if odd.
- If `l < r` still, swap `arr[l]` and `arr[r]`, then `l++`, `r--`.
- For odd checks, use `(num & 1) != 0`. Time: $O(N)$, Space: $O(1)$.
