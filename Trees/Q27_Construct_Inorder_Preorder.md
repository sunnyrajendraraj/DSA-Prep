# Construct Binary Tree from Preorder and Inorder Traversal

## 1. Problem Statement

Given two integer arrays `preorder` and `inorder` where `preorder` is the preorder traversal of a binary tree and `inorder` is the inorder traversal of the same tree, construct and return the binary tree.

### Input
- `preorder`: A vector of integers.
- `inorder`: A vector of integers.

### Output
- The root of the constructed binary tree.

### Constraints
- $1 \le \text{preorder.length} \le 3000$.
- `inorder.length == preorder.length`.
- $-3000 \le \text{preorder[i]}, \text{inorder[i]} \le 3000$.
- `preorder` and `inorder` consist of **unique** values.
- Each value of `inorder` also appears in `preorder`.
- `preorder` is guaranteed to be the preorder traversal of the tree.
- `inorder` is guaranteed to be the inorder traversal of the tree.

---

## 2. Interview Clarification

1. **Are the tree values unique?**
   - *Answer:* Yes, the constraints guarantee that all values are unique. If values were duplicate, reconstruction wouldn't be unique without additional structural information.
2. **Is it possible to have an empty input?**
   - *Answer:* Constraints say length $\ge 1$, so the input is non-empty. But we should still write general code that handles empty arrays safely.
3. **Can we modify the input arrays?**
   - *Answer:* It's better not to copy or modify them, as we want to optimize space. We can pass them by reference and use indices.

---

## 3. Thinking Process

Let's understand the traversals:
- **Preorder:** `[Root, Left, Right]` $\Rightarrow$ the first element is always the root of the current tree/subtree.
- **Inorder:** `[Left, Root, Right]` $\Rightarrow$ once we know which element is the root, its position splits the array into a left subtree and a right subtree.

### Divide and Conquer Strategy
1. The first element of `preorder` is the root node. Let's call it $R$.
2. Find the index of $R$ in the `inorder` array. Let this index be `inRoot`.
3. The elements to the left of `inRoot` in `inorder` belong to the **left subtree**.
4. The elements to the right of `inRoot` in `inorder` belong to the **right subtree**.
5. Calculate the size of the left subtree: `numsLeft = inRoot - inStart`.
6. Recursively build:
   - Left subtree using:
     - Preorder range: `[preStart + 1, preStart + numsLeft]`
     - Inorder range: `[inStart, inRoot - 1]`
   - Right subtree using:
     - Preorder range: `[preStart + numsLeft + 1, preEnd]`
     - Inorder range: `[inRoot + 1, inEnd]`

### Optimization:
- If we search for `inRoot` linearly in the `inorder` array every time, it takes $O(N)$ time per recursion.
- In a skewed tree, recursion depth is $O(N)$, resulting in an overall time complexity of $O(N^2)$.
- We can optimize this by building a hash map (`std::unordered_map`) mapping each value in `inorder` to its index. This allows us to find `inRoot` in $O(1)$ time, reducing overall complexity to $O(N)$.

---

## 4. Pattern Recognition

- **Divide and Conquer:** Breaking down the tree reconstruction into independent left and right subtree reconstructions.
- **Hash Map Index Mapping:** Using a lookup table to optimize search operations from $O(N)$ to $O(1)$.

---

## 5. Brute Force Solution

The brute force solution is to search for the root index linearly inside `inorder` at each recursive step, and make copies of the sub-arrays.

### C++17 Code
```cpp
#include <vector>
#include <algorithm>

struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};

class SolutionBrute {
public:
    TreeNode* buildTree(std::vector<int>& preorder, std::vector<int>& inorder) {
        if (preorder.empty() || inorder.empty()) {
            return nullptr;
        }
        
        int rootVal = preorder[0];
        TreeNode* root = new TreeNode(rootVal);
        
        // Find index linearly
        auto it = std::find(inorder.begin(), inorder.end(), rootVal);
        int index = std::distance(inorder.begin(), it);
        
        // Split arrays
        std::vector<int> leftInorder(inorder.begin(), inorder.begin() + index);
        std::vector<int> rightInorder(inorder.begin() + index + 1, inorder.end());
        
        std::vector<int> leftPreorder(preorder.begin() + 1, preorder.begin() + 1 + index);
        std::vector<int> rightPreorder(preorder.begin() + 1 + index, preorder.end());
        
        root->left = buildTree(leftPreorder, leftInorder);
        root->right = buildTree(rightPreorder, rightInorder);
        
        return root;
    }
};
```

### Complexity Analysis
- **Time Complexity:** $O(N^2)$ due to linear search and vector copies.
- **Space Complexity:** $O(N^2)$ due to creating new sub-vectors at every recursive step.

---

## 6. Better Solution

Instead of making copies of vectors, we can pass references and use indices. However, we still search linearly for the root in the inorder subarray.

### C++17 Code
```cpp
class SolutionBetter {
private:
    TreeNode* build(std::vector<int>& preorder, int preStart, int preEnd,
                    std::vector<int>& inorder, int inStart, int inEnd) {
        if (preStart > preEnd || inStart > inEnd) return nullptr;
        
        int rootVal = preorder[preStart];
        TreeNode* root = new TreeNode(rootVal);
        
        // Linear search
        int inRoot = inStart;
        while (inorder[inRoot] != rootVal) {
            inRoot++;
        }
        
        int numsLeft = inRoot - inStart;
        
        root->left = build(preorder, preStart + 1, preStart + numsLeft,
                           inorder, inStart, inRoot - 1);
        root->right = build(preorder, preStart + numsLeft + 1, preEnd,
                            inorder, inRoot + 1, inEnd);
        return root;
    }
public:
    TreeNode* buildTree(std::vector<int>& preorder, std::vector<int>& inorder) {
        return build(preorder, 0, preorder.size() - 1, inorder, 0, inorder.size() - 1);
    }
};
```

### Complexity Analysis
- **Time Complexity:** $O(N^2)$ in the worst case (skewed tree).
- **Space Complexity:** $O(H)$ recursion stack space, where $H$ is the tree height.

---

## 7. Optimal Solution

Using indices and a hash map to lookup values in the `inorder` array in $O(1)$ time.

### C++17 Code

```cpp
#include <vector>
#include <unordered_map>

class Solution {
private:
    TreeNode* build(const std::vector<int>& preorder, int preStart, int preEnd,
                    const std::vector<int>& inorder, int inStart, int inEnd,
                    const std::unordered_map<int, int>& inMap) {
        // Base case: empty ranges
        if (preStart > preEnd || inStart > inEnd) {
            return nullptr;
        }

        // 1. The first element in current preorder range is the root
        int rootVal = preorder[preStart];
        TreeNode* root = new TreeNode(rootVal);

        // 2. Locate the root value in inorder to partition subtrees
        int inRoot = inMap.at(rootVal);
        
        // 3. Count elements in left subtree
        int numsLeft = inRoot - inStart;

        // 4. Recurse for Left and Right subtrees
        root->left = build(preorder, preStart + 1, preStart + numsLeft,
                           inorder, inStart, inRoot - 1, inMap);
                           
        root->right = build(preorder, preStart + numsLeft + 1, preEnd,
                            inorder, inRoot + 1, inEnd, inMap);

        return root;
    }

public:
    TreeNode* buildTree(std::vector<int>& preorder, std::vector<int>& inorder) {
        // Map inorder values to their indices for O(1) lookup
        std::unordered_map<int, int> inMap;
        for (int i = 0; i < static_cast<int>(inorder.size()); ++i) {
            inMap[inorder[i]] = i;
        }

        return build(preorder, 0, static_cast<int>(preorder.size()) - 1,
                     inorder, 0, static_cast<int>(inorder.size()) - 1,
                     inMap);
    }
};
```

---

## 8. Code Walkthrough

- **`buildTree(preorder, inorder)`**:
  - Initializes `inMap` which maps values in `inorder` to their respective indices.
  - Calls `build` helper passing the start and end boundary indices of both arrays.
- **`build(...)`**:
  - Checks base case `preStart > preEnd || inStart > inEnd`. If true, returns `nullptr`.
  - Creates a new `TreeNode` with value `preorder[preStart]`.
  - Queries `inMap` to find where this root value sits in the inorder traversal (`inRoot`).
  - Computes `numsLeft = inRoot - inStart` (number of nodes in the left subtree).
  - Recursively links `root->left` by slicing the preorder left range (`[preStart + 1, preStart + numsLeft]`) and inorder left range (`[inStart, inRoot - 1]`).
  - Recursively links `root->right` by slicing preorder right range (`[preStart + numsLeft + 1, preEnd]`) and inorder right range (`[inRoot + 1, inEnd]`).
  - Returns `root`.

---

## 9. Dry Run

Let's dry run with:
`preorder = [3, 9, 20, 15, 7]`
`inorder = [9, 3, 15, 20, 7]`

### Map Construction
`inMap = {9:0, 3:1, 15:2, 20:3, 7:4}`

### Call 1: `build(preStart=0, preEnd=4, inStart=0, inEnd=4)`
- `rootVal = preorder[0] = 3`. Create `root = Node(3)`.
- `inRoot = inMap[3] = 1`.
- `numsLeft = 1 - 0 = 1`.
- Left Subtree call: `build(preStart=1, preEnd=1, inStart=0, inEnd=0)`
- Right Subtree call: `build(preStart=2, preEnd=4, inStart=2, inEnd=4)`

---

### Call 2 (Left): `build(preStart=1, preEnd=1, inStart=0, inEnd=0)`
- `rootVal = preorder[1] = 9`. Create `root = Node(9)`.
- `inRoot = inMap[9] = 0`.
- `numsLeft = 0 - 0 = 0`.
- Left: `build(preStart=2, preEnd=1, inStart=0, inEnd=-1)` $\Rightarrow$ returns `nullptr`.
- Right: `build(preStart=2, preEnd=1, inStart=1, inEnd=0)` $\Rightarrow$ returns `nullptr`.
- Return `Node(9)`.

---

### Call 3 (Right): `build(preStart=2, preEnd=4, inStart=2, inEnd=4)`
- `rootVal = preorder[2] = 20`. Create `root = Node(20)`.
- `inRoot = inMap[20] = 3`.
- `numsLeft = 3 - 2 = 1`.
- Left: `build(preStart=3, preEnd=3, inStart=2, inEnd=2)`
- Right: `build(preStart=4, preEnd=4, inStart=4, inEnd=4)`

---

### Call 4 (Left of 20): `build(preStart=3, preEnd=3, inStart=2, inEnd=2)`
- `rootVal = 15`. Returns `Node(15)`.

---

### Call 5 (Right of 20): `build(preStart=4, preEnd=4, inStart=4, inEnd=4)`
- `rootVal = 7`. Returns `Node(7)`.

---

### Final Tree
```
    3
   / \
  9   20
     /  \
    15   7
```

---

## 10. Complexity Analysis

| Metric | Complexity | Reasoning |
| :--- | :--- | :--- |
| **Time Complexity** | $O(N)$ | Building the hash map takes $O(N)$ time. We create exactly $N$ nodes, each taking $O(1)$ work because the root lookup in inorder is $O(1)$. |
| **Space Complexity** | $O(N)$ | The hash map occupies $O(N)$ auxiliary space. The recursive call stack takes $O(H)$ space, which is $O(N)$ in the worst case (skewed tree). |

---

## 11. Common Mistakes

1. **Incorrect range indices:** Off-by-one errors when partitioning ranges are very common. Pay special attention to `preStart + numsLeft` and `inRoot - 1`.
2. **Re-creating map recursively:** Do not build the index map inside the recursive helper function; build it once in the main wrapper.

---

## 12. Edge Cases

| Edge Case | Input | Handling |
| :--- | :--- | :--- |
| **Single Node** | `preorder = [1]`, `inorder = [1]` | Base case checks pass; root created, sub-calls return `nullptr`. |
| **Skewed Left Tree** | `pre = [1,2,3]`, `in = [3,2,1]` | Root splits everything to the left; right range becomes invalid (`preStart > preEnd`). |

---

## 13. Interview Explanation

"To construct a binary tree from its preorder and inorder traversals, we utilize the structural patterns of the traversals. Preorder always lists the root first, while inorder groups the left subtree nodes to the left of the root and right subtree nodes to the right of the root. 

In our algorithm, we first construct a hash map of the inorder values to their indices. This lets us find the position of any root in $O(1)$ time. We then write a recursive helper function that takes the current range of preorder and inorder lists. We designate the first element of preorder as the root, locate its index in inorder, calculate the number of elements in the left subtree, and partition the arrays. 

We recursively build the left subtree and right subtree with their corresponding ranges, eventually stitching them back to the root node. This results in an optimal time complexity of $O(N)$."

---

## 14. Follow-up Questions

1. **Can you do this without unique values?**
   - *Answer:* No. If values are duplicated, a unique tree cannot be determined solely from inorder and preorder. For instance, `preorder = [1, 1]`, `inorder = [1, 1]` could represent either $1 \to \text{left } 1$ or $1 \to \text{right } 1$.

---

## 15. Variations

1. **Construct BST from Preorder Traversal:** In a BST, inorder is just the sorted preorder array. Thus, we can sort the preorder to get inorder, or we can use a range-bound recursion ($O(N)$ time, no inorder needed).

---

## 16. Revision Notes

- **Preorder rule:** root is at the start (`preStart`).
- **Indices mapping:**
  - Left subtree: `pre: [preStart + 1, preStart + numsLeft]`, `in: [inStart, inRoot - 1]`
  - Right subtree: `pre: [preStart + numsLeft + 1, preEnd]`, `in: [inRoot + 1, inEnd]`
- **Uniqueness requirement:** The algorithm relies on unique node values.

---

## 17. Memorization Version

```text
Construct Tree from Preorder & Inorder:
1. Build index map of inorder.
2. Helper(preStart, preEnd, inStart, inEnd):
   - If preStart > preEnd, return null.
   - rootVal = preorder[preStart].
   - inRoot = map[rootVal], numsLeft = inRoot - inStart.
   - root->left = Helper(preStart + 1, preStart + numsLeft, inStart, inRoot - 1).
   - root->right = Helper(preStart + numsLeft + 1, preEnd, inRoot + 1, inEnd).
   - Return root.
Complexity: Time O(N), Space O(N).
```
