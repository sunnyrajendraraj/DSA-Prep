# Product of Array Except Self

## 1. Problem Statement

Given an integer array `nums`, return an array `answer` such that `answer[i]` is equal to the product of all the elements of `nums` except `nums[i]`.

The product of any prefix or suffix of `nums` is guaranteed to fit in a 32-bit integer.

* You must write an algorithm that runs in $O(n)$ time and without using the division operator.
* Could you solve it with $O(1)$ extra space complexity? (The output array does not count as extra space for space complexity analysis.)

### Input
* `nums`: A vector of integers where $2 \le \text{nums.length} \le 10^5$.

### Output
* Returns a vector of integers of length `nums.length`.

### Constraints
* $-30 \le \text{nums}[i] \le 30$
* The product of any prefix or suffix of `nums` is guaranteed to fit in a 32-bit integer.

---

## 2. Interview Clarification

Before writing any code, clarify these points with your interviewer:
1. **Can the array contain zeros?**
   * *Answer*: Yes, and this is a key reason why we cannot simply compute the total product of the array and divide by `nums[i]` (division by zero is undefined).
2. **What if there are multiple zeros in the array?**
   * *Answer*: If there are two or more zeros, the product except self at any position will be `0`.
3. **Can we use division?**
   * *Answer*: No, the problem explicitly forbids using division.

---

## 3. Thinking Process

### First Instinct (Brute Force)
* For each index `i`, loop through all other indices `j` (where `j != i`) and multiply all elements together. Put this value in `answer[i]`.
* This requires two nested loops, taking $O(n^2)$ time.

### Better Instinct (Prefix and Suffix Arrays)
* The product except self at index `i` is the product of all elements to its left (prefix) multiplied by the product of all elements to its right (suffix).
  $$\text{answer}[i] = (\text{Prefix product up to } i-1) \times (\text{Suffix product from } i+1)$$
* We can precompute two arrays:
  * `prefix[i]`: Product of all elements from index `0` to `i-1`.
  * `suffix[i]`: Product of all elements from index `i+1` to `n-1`.
* Then, `answer[i] = prefix[i] * suffix[i]`.
* This takes $O(n)$ time and $O(n)$ auxiliary space to store the prefix and suffix arrays.

### Optimal Insight (O(1) Auxiliary Space)
* We can avoid using extra arrays for prefix and suffix by using the output `answer` array to store the prefix products first.
* **Step 1**: Loop from left to right. Set `answer[i]` to be the running prefix product of elements before `i`.
  * `answer[0] = 1`
  * `answer[i] = answer[i-1] * nums[i-1]`
* **Step 2**: Maintain a single variable `suffix_product = 1`. Loop from right to left.
  * For each index `i`, update `answer[i] = answer[i] * suffix_product`.
  * Then, update `suffix_product = suffix_product * nums[i]`.
* This gives the correct answer array using $O(n)$ time and $O(1)$ extra space (since the output array does not count towards the space limit).

---

## 4. Pattern Recognition

* **Topic**: Array Prefix/Suffix preprocessing.
* **Pattern**: Cumulative product tracking.
* **Why this works**: Split the calculation into independent left and right halves. By accumulating the left-side product in a forward pass and the right-side product in a backward pass, we calculate the product of all elements excluding the current index.

---

## 5. Brute Force Solution

### Algorithm
1. Initialize `answer` vector of size `n` with all 1s.
2. For each index `i` from `0` to `n-1`:
   * Set `prod = 1`.
   * For each index `j` from `0` to `n-1`:
     * If `i != j`, set `prod = prod * nums[j]`.
   * Store `prod` in `answer[i]`.
3. Return `answer`.

### Complexity Analysis
* **Time Complexity**: $O(n^2)$
* **Space Complexity**: $O(1)$ auxiliary space (excluding the output array).

### C++17 Code
```cpp
#include <vector>

class SolutionBrute {
public:
    std::vector<int> productExceptSelf(const std::vector<int>& nums) {
        int n = nums.size();
        std::vector<int> answer(n, 1);
        
        for (int i = 0; i < n; ++i) {
            int product = 1;
            for (int j = 0; j < n; ++j) {
                if (i != j) {
                    product *= nums[j];
                }
            }
            answer[i] = product;
        }
        
        return answer;
    }
};
```

---

## 6. Better Solution

### Algorithm (Prefix & Suffix Arrays)
1. Initialize `prefix` array of size `n` where `prefix[0] = 1`.
2. Populate `prefix` array: `prefix[i] = prefix[i-1] * nums[i-1]`.
3. Initialize `suffix` array of size `n` where `suffix[n-1] = 1`.
4. Populate `suffix` array: `suffix[i] = suffix[i+1] * nums[i+1]`.
5. Construct `answer`: `answer[i] = prefix[i] * suffix[i]`.

### Dry Run
Input: `nums = [1, 2, 3, 4]`
* `prefix = [1, 1, 2, 6]` (e.g., prefix[3] = prefix[2]*nums[2] = 2 * 3 = 6)
* `suffix = [24, 12, 4, 1]` (e.g., suffix[0] = suffix[1]*nums[1] = 12 * 2 = 24)
* `answer = [1*24, 1*12, 2*4, 6*1] = [24, 12, 8, 6]`

### Complexity Analysis
* **Time Complexity**: $O(n)$ (three linear passes: prefix pass, suffix pass, merge pass).
* **Space Complexity**: $O(n)$ auxiliary space to store prefix and suffix arrays.

### C++17 Code
```cpp
#include <vector>

class SolutionBetter {
public:
    std::vector<int> productExceptSelf(const std::vector<int>& nums) {
        int n = nums.size();
        std::vector<int> prefix(n, 1);
        std::vector<int> suffix(n, 1);
        std::vector<int> answer(n);
        
        // Build prefix product array
        for (int i = 1; i < n; ++i) {
            prefix[i] = prefix[i - 1] * nums[i - 1];
        }
        
        // Build suffix product array
        for (int i = n - 2; i >= 0; --i) {
            suffix[i] = suffix[i + 1] * nums[i + 1];
        }
        
        // Combine prefix and suffix products
        for (int i = 0; i < n; ++i) {
            answer[i] = prefix[i] * suffix[i];
        }
        
        return answer;
    }
};
```

---

## 7. Optimal Solution

### Algorithm
1. Initialize an `answer` vector of size `n` with `answer[0] = 1`.
2. First pass (Left to Right): For each index `i` from `1` to `n-1`, set `answer[i] = answer[i-1] * nums[i-1]`.
   * Now `answer[i]` holds the product of all elements to the left of `i`.
3. Second pass (Right to Left):
   * Initialize a variable `suffix_product = 1`.
   * For each index `i` from `n-1` down to `0`:
     * Multiply `answer[i]` by `suffix_product`.
     * Update `suffix_product` by multiplying it with `nums[i]`.
4. Return `answer`.

### Why this is Optimal
* It uses only one additional scalar variable (`suffix_product`).
* It modifies the output array directly, which is allowed under the $O(1)$ extra space constraint.
* The time complexity remains $O(n)$ with only two passes.

### C++17 Code
```cpp
#include <vector>

class SolutionOptimal {
public:
    std::vector<int> productExceptSelf(const std::vector<int>& nums) {
        int n = nums.size();
        std::vector<int> answer(n);
        
        // Step 1: Accumulate prefix products in the answer array
        answer[0] = 1;
        for (int i = 1; i < n; ++i) {
            answer[i] = answer[i - 1] * nums[i - 1];
        }
        
        // Step 2: Accumulate suffix products on the fly using a single variable
        int suffix_product = 1;
        for (int i = n - 1; i >= 0; --i) {
            answer[i] *= suffix_product;
            suffix_product *= nums[i];
        }
        
        return answer;
    }
};
```

---

## 8. Code Walkthrough

* **Line 7**: Initialize `answer` vector of size `n`.
* **Line 10-13**: Populate `answer` with prefix products.
  * For index `0`, there are no left elements, so prefix is `1`.
  * For any other `i`, `answer[i]` is computed by taking the product of the previous prefix `answer[i-1]` and the element `nums[i-1]`.
* **Line 16**: Initialize `suffix_product = 1`.
* **Line 17**: Iterate backwards from `n-1` down to `0`.
* **Line 18**: `answer[i] *= suffix_product;`
  We multiply the existing prefix product of `i` (which is stored in `answer[i]`) by the suffix product of all elements to the right of `i`.
* **Line 19**: `suffix_product *= nums[i];`
  We update our running suffix product to include the current element `nums[i]` before moving to the next element on the left.

---

## 9. Dry Run

Input: `nums = [1, 2, 3, 4]`

### Forward Pass:
* `i = 0`: `answer[0] = 1`
* `i = 1`: `answer[1] = answer[0] * nums[0] = 1 * 1 = 1`
* `i = 2`: `answer[2] = answer[1] * nums[1] = 1 * 2 = 2`
* `i = 3`: `answer[3] = answer[2] * nums[2] = 2 * 3 = 6`

State of `answer` after forward pass: `[1, 1, 2, 6]`

### Backward Pass:
Initialize `suffix_product = 1`.

| Index `i` | `nums[i]` | `answer[i]` (before) | `answer[i] *= suffix_product` | Updated `answer[i]` | `suffix_product *= nums[i]` | Updated `suffix_product` |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 3 | 4 | 6 | `6 * 1` | 6 | `1 * 4` | 4 |
| 2 | 3 | 2 | `2 * 4` | 8 | `4 * 3` | 12 |
| 1 | 2 | 1 | `1 * 12` | 12 | `12 * 2` | 24 |
| 0 | 1 | 1 | `1 * 24` | 24 | `24 * 1` | 24 |

Final `answer`: `[24, 12, 8, 6]`

---

## 10. Complexity Analysis

| Approach | Time Complexity | Space Complexity | Notes |
| :--- | :--- | :--- | :--- |
| Brute Force | $O(n^2)$ | $O(1)$ auxiliary | Nested loops |
| Prefix & Suffix Arrays | $O(n)$ | $O(n)$ auxiliary | Uses two extra arrays |
| **Optimal space** | **$O(n)$** | **$O(1)$ auxiliary** | **Uses output array to store prefix** |

---

## 11. Common Mistakes

1. **Using division**:
   * Some candidates will divide the total product by the current element. This is explicitly banned and fails completely when the array contains `0`.
2. **Incorrect bounds on loops**:
   * Be very careful with index off-by-one errors. For prefix, `answer[i]` depends on `nums[i-1]`. For suffix, iteration must go down to `0` (inclusive).
3. **Assuming array size is less than 2**:
   * Make sure to check if the input vector size is smaller than 2, though constraints specify $2 \le n$.

---

## 12. Edge Cases

| Edge Case | Example | Why it matters | Solution Behavior |
| :--- | :--- | :--- | :--- |
| Array contains one zero | `[1, 0, 3, 4]` | Division would fail. Suffix/prefix logic handles it. | Returns `[0, 12, 0, 0]` (Correct) |
| Array contains multiple zeros | `[1, 0, 3, 0]` | Every position's output must be `0`. | Returns `[0, 0, 0, 0]` (Correct) |
| Negative numbers | `[-1, 2, -3, 4]` | Sign changes must be handled correctly. | Returns `[-24, 12, -8, 6]` (Correct) |
| Two elements only | `[5, 10]` | Minimum constraints. | Returns `[10, 5]` (Correct) |

---

## 13. Interview Explanation

"To solve this problem without using division in $O(n)$ time and $O(1)$ auxiliary space, we split the calculation of the product except self into two components: the product of all elements to the left (prefix) and the product of all elements to the right (suffix).

We first perform a forward pass where we build the prefix product directly inside the output array. After this pass, each position `i` in the output array contains the product of all elements before `i`. 
Then, we iterate backwards from the end of the array, maintaining a running `suffix_product` variable. At each step, we multiply the prefix product at index `i` by this running `suffix_product`, and then update the `suffix_product` by multiplying it with the element at index `i`. This gives us the correct answer without any division or extra space."

---

## 14. Follow-up Questions

1. **Can this run in a single pass?**
   * *Answer*: We cannot compute the products in a single sequential pass because the suffix product requires information from the right which hasn't been visited yet. However, we can use two pointers moving from both ends towards the middle to fill a prefix and suffix array simultaneously, but to achieve $O(1)$ auxiliary space, two passes on the output array is the most standard way.
2. **How does this scale if the product overflows standard integer types?**
   * *Answer*: In real-world systems, we would use a 64-bit integer (`long long` in C++) or a big integer library, but the problem constraints guarantee products fit in 32-bit signed integers.

---

## 15. Variations

1. **Product of Array Except Self with division allowed**: If division is allowed, we can count the number of zeros. If zero count is $> 1$, all outputs are 0. If zero count is 1, only the index of the zero gets the product of non-zero elements, others get 0. If zero count is 0, each element is `Total Product / nums[i]`.
2. **Subarray Product Less Than K**: Sliding window problem to find subarrays.

---

## 16. Revision Notes

* **Concept**: Prefix Product $\times$ Suffix Product.
* **Algorithm**:
  * Forward: `ans[i] = ans[i-1] * nums[i-1]`
  * Backward: `ans[i] *= suffix; suffix *= nums[i]`
* **Key Constraint**: No Division, $O(n)$ Time, $O(1)$ auxiliary Space.

---

## 17. Memorization Version

```cpp
// O(n) Time, O(1) Space (Auxiliary). Prefix-suffix computation.
vector<int> productExceptSelf(vector<int>& nums) {
    int n = nums.size();
    vector<int> ans(n, 1);
    for (int i = 1; i < n; ++i) {
        ans[i] = ans[i - 1] * nums[i - 1];
    }
    int suffix = 1;
    for (int i = n - 1; i >= 0; --i) {
        ans[i] *= suffix;
        suffix *= nums[i];
    }
    return ans;
}
```
