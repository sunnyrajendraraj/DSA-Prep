# Construct Binary Tree from Inorder and Postorder Traversal

## 1. Problem Statement

Given two integer arrays `inorder` and `postorder` where `inorder` is the inorder traversal of a binary tree and `postorder` is the postorder traversal of the same tree, construct and return the binary tree.

### Input
- `inorder`: A vector of integers.
- `postorder`: A vector of integers.

### Output
- The root of the constructed binary tree.

### Constraints
- $1 \le \text{inorder.length} \le 3000$.
- `postorder.length == inorder.length`.
- $-3000 \le \text{inorder[i]}, \text{postorder[i]} \le 3000$.
- `inorder` and `postorder` consist of **unique** values.
- Each value of `postorder` also appears in `inorder`.
- `inorder` is guaranteed to be the inorder traversal of the tree.
- `postorder` is guaranteed to be the postorder traversal of the tree.

---

## 2. Interview Clarification

1. **How does postorder differ from preorder for tree construction?**
   - *Answer:* In preorder, the root is at the beginning (`preorder[preStart]`). In postorder, the root is at the end (`postorder[postEnd]`).
2. **Does the recursion order matter?**
   - *Answer:* Yes, if we keep track of the postorder root index using a global decreasing pointer, we must process the **right** subtree before the **left** subtree. If we use explicit boundaries for postorder ranges, we can recurse in any order, but right-then-left is standard and clean.
3. **Are duplicate values possible?**
   - *Answer:* No, values are unique.

---

## 3. Thinking Process

Let's dissect the traversals:
- **Inorder:** `[Left Subtree, Root, Right Subtree]`
- **Postorder:** `[Left Subtree, Right Subtree, Root]`

### Divide and Conquer Strategy
1. The last element in the current range of `postorder` is the root node of the current subtree. Let's call it $R = \text{postorder[postEnd]}$.
2. Locate the index of $R$ in the `inorder` array. Let this index be `inRoot`.
3. The elements to the left of `inRoot` in `inorder` represent the **left subtree**.
4. The elements to the right of `inRoot` in `inorder` represent the **right subtree**.
5. Calculate the size of the left subtree: `numsLeft = inRoot - inStart`.
6. Recursively build:
   - **Left subtree** using:
     - Inorder range: `[inStart, inRoot - 1]`
     - Postorder range: `[postStart, postStart + numsLeft - 1]`
   - **Right subtree** using:
     - Inorder range: `[inRoot + 1, inEnd]`
     - Postorder range: `[postStart + numsLeft, postEnd - 1]`

### Optimization:
- As with the preorder-inorder problem, searching linearly for `inRoot` takes $O(N)$ time per step.
- We map each value of `inorder` to its index in a hash map (`std::unordered_map`) to perform this lookup in $O(1)$ time, yielding an optimal overall time complexity of $O(N)$.

---

## 4. Pattern Recognition

- **Divide and Conquer:** Recursively partitioning ranges to reconstruct subtrees.
- **Hash Table Index Lookup:** Replacing search operations with constant-time map accesses.

---

## 5. Brute Force Solution

Construct by splitting vectors via copy operations and scanning linearly for the root index.

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
    TreeNode* buildTree(std::vector<int>& inorder, std::vector<int>& postorder) {
        if (inorder.empty() || postorder.empty()) {
            return nullptr;
        }

        int rootVal = postorder.back();
        TreeNode* root = new TreeNode(rootVal);

        auto it = std::find(inorder.begin(), inorder.end(), rootVal);
        int index = std::distance(inorder.begin(), it);

        std::vector<int> leftInorder(inorder.begin(), inorder.begin() + index);
        std::vector<int> rightInorder(inorder.begin() + index + 1, inorder.end());

        std::vector<int> leftPostorder(postorder.begin(), postorder.begin() + index);
        std::vector<int> rightPostorder(postorder.begin() + index, postorder.end() - 1);

        root->left = buildTree(leftInorder, leftPostorder);
        root->right = buildTree(rightInorder, rightPostorder);

        return root;
    }
};
```

### Complexity Analysis
- **Time Complexity:** $O(N^2)$ due to linear searches and subarray copy operations.
- **Space Complexity:** $O(N^2)$ due to vector allocations in recursion.

---

## 6. Better Solution

Use index-based boundaries instead of creating copies of subarrays, though search remains linear.

### C++17 Code
```cpp
class SolutionBetter {
private:
    TreeNode* build(std::vector<int>& inorder, int inStart, int inEnd,
                    std::vector<int>& postorder, int postStart, int postEnd) {
        if (inStart > inEnd || postStart > postEnd) return nullptr;

        int rootVal = postorder[postEnd];
        TreeNode* root = new TreeNode(rootVal);

        int inRoot = inStart;
        while (inorder[inRoot] != rootVal) {
            inRoot++;
        }

        int numsLeft = inRoot - inStart;

        root->left = build(inorder, inStart, inRoot - 1,
                           postorder, postStart, postStart + numsLeft - 1);
        root->right = build(inorder, inRoot + 1, inEnd,
                            postorder, postStart + numsLeft, postEnd - 1);
        return root;
    }
public:
    TreeNode* buildTree(std::vector<int>& inorder, std::vector<int>& postorder) {
        return build(inorder, 0, inorder.size() - 1, postorder, 0, postorder.size() - 1);
    }
};
```

### Complexity Analysis
- **Time Complexity:** $O(N^2)$ in the worst case (skewed tree).
- **Space Complexity:** $O(H)$ recursion stack space.

---

## 7. Optimal Solution

Using index-based boundary arguments and a hash map for $O(1)$ index lookup of inorder elements.

### C++17 Code

```cpp
#include <vector>
#include <unordered_map>

class Solution {
private:
    TreeNode* build(const std::vector<int>& inorder, int inStart, int inEnd,
                    const std::vector<int>& postorder, int postStart, int postEnd,
                    const std::unordered_map<int, int>& inMap) {
        // Base case: empty boundaries
        if (inStart > inEnd || postStart > postEnd) {
            return nullptr;
        }

        // 1. The last element in current postorder range is the root
        int rootVal = postorder[postEnd];
        TreeNode* root = new TreeNode(rootVal);

        // 2. Locate root index in inorder array
        int inRoot = inMap.at(rootVal);

        // 3. Count elements in the left subtree
        int numsLeft = inRoot - inStart;

        // 4. Recurse for Left and Right subtrees
        root->left = build(inorder, inStart, inRoot - 1,
                           postorder, postStart, postStart + numsLeft - 1,
                           inMap);
                           
        root->right = build(inorder, inRoot + 1, inEnd,
                            postorder, postStart + numsLeft, postEnd - 1,
                            inMap);

        return root;
    }

public:
    TreeNode* buildTree(std::vector<int>& inorder, std::vector<int>& postorder) {
        // Build map for O(1) index queries
        std::unordered_map<int, int> inMap;
        for (int i = 0; i < static_cast<int>(inorder.size()); ++i) {
            inMap[inorder[i]] = i;
        }

        return build(inorder, 0, static_cast<int>(inorder.size()) - 1,
                     postorder, 0, static_cast<int>(postorder.size()) - 1,
                     inMap);
    }
};
```

---

## 8. Code Walkthrough

- **`buildTree(...)`**:
  - Initializes `inMap` with the indices of elements in the `inorder` array.
  - Calls `build` passing boundaries `0` and `N - 1` for both `inorder` and `postorder`.
- **`build(...)`**:
  - Checks if `inStart > inEnd || postStart > postEnd`. If true, returns `nullptr`.
  - Determines the root value from `postorder[postEnd]`.
  - Creates the root node `root`.
  - Finds `inRoot`, the position of `rootVal` in the inorder array.
  - Calculates `numsLeft = inRoot - inStart` (how many nodes are in the left subtree of the current root).
  - Recursively builds `root->left` by setting:
    - Inorder boundaries to `[inStart, inRoot - 1]`.
    - Postorder boundaries to `[postStart, postStart + numsLeft - 1]`.
  - Recursively builds `root->right` by setting:
    - Inorder boundaries to `[inRoot + 1, inEnd]`.
    - Postorder boundaries to `[postStart + numsLeft, postEnd - 1]`.
  - Returns `root`.

---

## 9. Dry Run

Let's dry run with:
`inorder = [9, 3, 15, 20, 7]`
`postorder = [9, 15, 7, 20, 3]`

### Map Construction
`inMap = {9:0, 3:1, 15:2, 20:3, 7:4}`

### Call 1: `build(inStart=0, inEnd=4, postStart=0, postEnd=4)`
- `rootVal = postorder[4] = 3`. Create `root = Node(3)`.
- `inRoot = inMap[3] = 1`.
- `numsLeft = 1 - 0 = 1`.
- Left Subtree call: `build(inStart=0, inEnd=0, postStart=0, postEnd=0)`
- Right Subtree call: `build(inStart=2, inEnd=4, postStart=1, postEnd=3)`

---

### Call 2 (Left): `build(0, 0, 0, 0)`
- `rootVal = postorder[0] = 9`. Create `root = Node(9)`.
- `inRoot = inMap[9] = 0`.
- `numsLeft = 0`.
- Recursion calls on left and right will hit base case and return `nullptr`.
- Return `Node(9)`.

---

### Call 3 (Right): `build(inStart=2, inEnd=4, postStart=1, postEnd=3)`
- `rootVal = postorder[3] = 20`. Create `root = Node(20)`.
- `inRoot = inMap[20] = 3`.
- `numsLeft = 3 - 2 = 1`.
- Left Subtree call: `build(inStart=2, inEnd=2, postStart=1, postEnd=1)`
- Right Subtree call: `build(inStart=4, inEnd=4, postStart=2, postEnd=2)`

---

### Call 4 (Left of 20): `build(2, 2, 1, 1)`
- `rootVal = postorder[1] = 15`. Returns `Node(15)`.

---

### Call 5 (Right of 20): `build(4, 4, 2, 2)`
- `rootVal = postorder[2] = 7`. Returns `Node(7)`.

---

### Final Constructed Tree
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
| **Time Complexity** | $O(N)$ | Building `inMap` takes $O(N)$ time. The recursive function runs $N$ times, each taking $O(1)$ work because index lookup in the map is $O(1)$. |
| **Space Complexity** | $O(N)$ | The index map uses $O(N)$ memory. The recursion stack uses $O(H)$ space, which is $O(N)$ in the worst case (skewed tree). |

---

## 11. Common Mistakes

1. **Incorrect Postorder Right Subtree boundaries:** A common mistake is using `[postStart + numsLeft + 1, postEnd - 1]`. The correct start is `postStart + numsLeft` because postorder has no root element at the beginning of the right partition (unlike preorder).
2. **Incorrect Postorder Left Subtree end boundary:** Make sure it is `postStart + numsLeft - 1`.

---

## 12. Edge Cases

| Edge Case | Input | Handling |
| :--- | :--- | :--- |
| **Single Node** | `in = [5]`, `post = [5]` | Base cases stop recursion at depth 1. Returns `Node(5)`. |
| **Right Skewed Tree** | `in = [1,2,3]`, `post = [3,2,1]` | Handled correctly. Left subtrees evaluate to empty ranges. |

---

## 13. Interview Explanation

"To build a binary tree from its inorder and postorder traversals, we use the fact that the last element of the postorder array is always the root. Once we identify this root, we find its index in the inorder array, which allows us to partition the tree into left and right subtrees. 

By calculating the number of elements in the left subtree, we can accurately slice the postorder array into left and right subtree sections. We build a hash map upfront to store the indices of the inorder array, giving us $O(1)$ lookup for the root's position. We then recursively reconstruct the left and right subtrees, linking them to the root. This operates in $O(N)$ time and uses $O(N)$ auxiliary space."

---

## 14. Follow-up Questions

1. **Why does postorder need different boundary offsets compared to preorder?**
   - *Answer:* Preorder is structured as `[Root][Left][Right]`, so the left subtree starts right after root: `preStart + 1`. Postorder is structured as `[Left][Right][Root]`, so the left subtree starts at `postStart` and the right subtree starts at `postStart + numsLeft`.

---

## 15. Variations

1. **Construct Tree from Preorder and Postorder Traversal:** Without inorder, we cannot construct a unique tree unless we assume it is a full binary tree (every node has 0 or 2 children).

---

## 16. Revision Notes

- **Postorder rule:** Root is at `postEnd`.
- **Interval mappings:**
  - Left subtree: `in: [inStart, inRoot - 1]`, `post: [postStart, postStart + numsLeft - 1]`
  - Right subtree: `in: [inRoot + 1, inEnd]`, `post: [postStart + numsLeft, postEnd - 1]`
- **Height space complexity:** $O(H)$ recursion stack space.

---

## 17. Memorization Version

```text
Construct Tree from Inorder & Postorder:
1. Build index map of inorder.
2. Helper(inStart, inEnd, postStart, postEnd):
   - If inStart > inEnd, return null.
   - rootVal = postorder[postEnd].
   - inRoot = map[rootVal], numsLeft = inRoot - inStart.
   - root->left = Helper(inStart, inRoot - 1, postStart, postStart + numsLeft - 1).
   - root->right = Helper(inRoot + 1, inEnd, postStart + numsLeft, postEnd - 1).
   - Return root.
Complexity: Time O(N), Space O(N).
```
