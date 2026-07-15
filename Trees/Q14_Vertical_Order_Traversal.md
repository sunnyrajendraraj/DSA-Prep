# Vertical Order Traversal of a Binary Tree

## 1. Problem Statement

Given the root of a binary tree, calculate the vertical order traversal of the binary tree.

For each node at position `(row, col)`, its left child will be at `(row + 1, col - 1)` and its right child will be at `(row + 1, col + 1)`. The root of the tree is at `(0, 0)`.

The vertical order traversal is a list of top-to-bottom orderings for each column index starting from the left-most column and ending on the right-most column. There may be multiple nodes in the same row and same column. In such a case, sort these nodes by their values.

*   **Input**:
    *   A pointer to the root of the binary tree: `TreeNode* root`.
*   **Output**:
    *   A 2D vector of integers: `vector<vector<int>>` representing the vertical order traversal.
*   **Constraints**:
    *   $1 \le N \le 1000$ (Number of nodes in the tree)
    *   $0 \le \text{Node value} \le 1000$

*   **Observations**:
    *   Nodes are categorized by their vertical column (`col`).
    *   Columns are visited from left to right (minimum `col` to maximum `col`).
    *   Within each column, nodes are processed from top to bottom (minimum `row` to maximum `row`).
    *   If two nodes share both the same column and row, they must be sorted in ascending order of their values.

---

## 2. Interview Clarification

1.  **Q**: If two nodes are at the same row and column, how are they ordered?
    *   *A*: They must be sorted by their values in ascending order.
2.  **Q**: What is the root's coordinate?
    *   *A*: The root is at `(row = 0, col = 0)`.
3.  **Q**: Can the tree be empty?
    *   *A*: Yes, though constraints say $1 \le N$, we should handle `nullptr` safely.

---

## 3. Thinking Process

*   **First Instinct**:
    *   We need to track coordinates: vertical column (`col`) and level row (`row`) for each node.
    *   Starting from `root` at `(0, 0)`:
        *   Going left decreases column: `(row + 1, col - 1)`.
        *   Going right increases column: `(row + 1, col + 1)`.
    *   We can use BFS or DFS. BFS is natural for processing level-by-level (top to bottom).
    *   To keep the output ordered:
        *   Columns must be sorted.
        *   Within a column, rows must be sorted.
        *   Within the same column and row, values must be sorted.
    *   This nested sorting requirement suggests a composite data structure:
        *   `std::map` automatically keys in sorted order.
        *   `map<int, map<int, multiset<int>>> nodes;`
        *   Outer key: `col` (sorted automatically).
        *   Inner key: `row` (sorted automatically).
        *   Value: `multiset<int>` (values sorted automatically, handles duplicates).

---

## 4. Pattern Recognition

*   **Topic**: Tree Traversal & Coordinate Tracking.
*   **Pattern**: BFS/DFS with coordinate hashing/mapping.
*   **Why**: We need to group nodes by coordinate offsets. An ordered map is perfect for maintaining sorted coordinates without manual sorting passes.

---

## 5. Brute Force Solution

*   **Algorithm**:
    *   Perform a DFS. Collect all nodes into a flat list of tuples: `(col, row, value)`.
    *   Sort the flat list of size $N$ using a custom comparator:
        *   First by `col` ascending.
        *   Then by `row` ascending.
        *   Then by `value` ascending.
    *   Group elements by `col` and populate the output list.
*   **Dry Run**:
    *   Flat list representation: `[(0, 0, 1), (-1, 1, 2), (1, 1, 3)]`.
    *   Sorted list: `[(-1, 1, 2), (0, 0, 1), (1, 1, 3)]`.
    *   Grouped: `[[2], [1], [3]]`.
*   **Complexity**:
    *   **Time Complexity**: $O(N \log N)$ due to sorting the flat list.
    *   **Space Complexity**: $O(N)$ to store all nodes in a list.
*   **Pros/Cons**:
    *   *Pros*: Easy to write standard sort logic.
    *   *Cons*: Custom comparators can be error-prone; sorting a large vector at once may have cache miss overhead.

### Brute Force C++17 Code
```cpp
#include <iostream>
#include <vector>
#include <algorithm>

struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};

struct NodeInfo {
    int col;
    int row;
    int val;
};

void dfsBrute(TreeNode* root, int row, int col, std::vector<NodeInfo>& list) {
    if (!root) return;
    list.push_back({col, row, root->val});
    dfsBrute(root->left, row + 1, col - 1, list);
    dfsBrute(root->right, row + 1, col + 1, list);
}

std::vector<std::vector<int>> verticalTraversalBrute(TreeNode* root) {
    std::vector<NodeInfo> list;
    dfsBrute(root, 0, 0, list);

    std::sort(list.begin(), list.end(), [](const NodeInfo& a, const NodeInfo& b) {
        if (a.col != b.col) return a.col < b.col;
        if (a.row != b.row) return a.row < b.row;
        return a.val < b.val;
    });

    std::vector<std::vector<int>> result;
    if (list.empty()) return result;

    int current_col = list[0].col;
    std::vector<int> col_vals;
    for (const auto& node : list) {
        if (node.col != current_col) {
            result.push_back(col_vals);
            col_vals.clear();
            current_col = node.col;
        }
        col_vals.push_back(node.val);
    }
    result.push_back(col_vals);
    return result;
}
```

---

## 6. Better Solution

*   *Note*: The transition from sorting a flat array to using structured map collections forms the optimal design. We will detail the structured mapping approach as our optimal solution.

---

## 7. Optimal Solution

*   **Approach**: BFS with nested `std::map` mapping.
*   **Data Structure**: `map<int, map<int, multiset<int>>> nodes;`
*   **Algorithm**:
    1.  Initialize a queue of pairs: `queue<pair<TreeNode*, pair<int, int>>> q;` storing `(node, (row, col))`.
    2.  Push `(root, (0, 0))` to `q`.
    3.  While `q` is not empty:
        *   Pop the front element: `curr`, `row`, `col`.
        *   Insert `curr->val` into `nodes[col][row]`.
        *   If `curr->left` exists, push `(curr->left, (row + 1, col - 1))` to `q`.
        *   If `curr->right` exists, push `(curr->right, (row + 1, col + 1))` to `q`.
    4.  Traverse the map `nodes` (since `map` is ordered, `col` keys will be iterated from lowest to highest, and nested `row` keys will also be iterated from lowest to highest):
        *   For each column, create a vector `col_vals`.
        *   Iterate through the row map of that column.
        *   Extract all values from the `multiset` and append them to `col_vals`.
        *   Push `col_vals` into the `result` vector.

### Optimal C++17 Code
```cpp
#include <iostream>
#include <vector>
#include <queue>
#include <map>
#include <set>

/**
 * Definition for a binary tree node.
 */
struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};

class Solution {
public:
    /**
     * Computes the vertical order traversal of a binary tree.
     */
    std::vector<std::vector<int>> verticalTraversal(TreeNode* root) {
        std::vector<std::vector<int>> result;
        if (root == nullptr) {
            return result;
        }

        // map<col, map<row, multiset<val>>>
        // Ordered map maintains sorted columns and sorted rows.
        // Multiset maintains sorted values for duplicate coordinates.
        std::map<int, std::map<int, std::multiset<int>>> nodes;
        
        // Queue stores pair of: (Node, (row, col))
        std::queue<std::pair<TreeNode*, std::pair<int, int>>> q;
        q.push({root, {0, 0}});

        while (!q.empty()) {
            auto [curr, coords] = q.front();
            q.pop();

            int row = coords.first;
            int col = coords.second;

            // Insert value into the structured coordinate map
            nodes[col][row].insert(curr->val);

            // Left child goes to row + 1, col - 1
            if (curr->left != nullptr) {
                q.push({curr->left, {row + 1, col - 1}});
            }
            // Right child goes to row + 1, col + 1
            if (curr->right != nullptr) {
                q.push({curr->right, {row + 1, col + 1}});
            }
        }

        // Extract values from ordered map structure
        for (auto const& [col, row_map] : nodes) {
            std::vector<int> col_vals;
            for (auto const& [row, vals] : row_map) {
                // Insert all sorted values from the multiset
                col_vals.insert(col_vals.end(), vals.begin(), vals.end());
            }
            result.push_back(col_vals);
        }

        return result;
    }
};
```

---

## 8. Code Walkthrough

1.  **Coordinate Mapping**:
    *   `map<int, map<int, multiset<int>>> nodes` acts as our coordinate grid database.
2.  **BFS Queue**:
    *   `queue<pair<TreeNode*, pair<int, int>>> q` stores nodes along with their `(row, col)` state.
3.  **Dynamic Grid Population**:
    *   `nodes[col][row].insert(curr->val)` auto-allocates maps for new coordinates and places the value into a sorted container (`multiset`).
4.  **Children Traversal**:
    *   `row + 1, col - 1` for left child.
    *   `row + 1, col + 1` for right child.
5.  **Output Reconstruction**:
    *   Iterate through the maps. The outer loop guarantees visiting columns from left-to-right (e.g., `-2, -1, 0, 1, 2`).
    *   The inner loop guarantees visiting rows from top-to-bottom (e.g., `0, 1, 2`).
    *   `col_vals.insert(col_vals.end(), vals.begin(), vals.end())` efficiently aggregates values.

---

## 9. Dry Run

Consider the tree:
```
      1
     / \
    2   3
   / \   \
  4   5   6
```

Coordinates assigned:
*   `1`: `(0, 0)`
*   `2`: `(1, -1)`, `3`: `(1, 1)`
*   `4`: `(2, -2)`, `5`: `(2, 0)`, `6`: `(2, 2)`

### Map Operations:
*   Insert `1` at `nodes[0][0]` $\rightarrow$ `nodes = { 0: {0: {1}} }`
*   Insert `2` at `nodes[-1][1]` $\rightarrow$ `nodes = { -1: {1: {2}}, 0: {0: {1}} }`
*   Insert `3` at `nodes[1][1]` $\rightarrow$ `nodes = { -1: {1: {2}}, 0: {0: {1}}, 1: {1: {3}} }`
*   Insert `4` at `nodes[-2][2]`
*   Insert `5` at `nodes[0][2]`
*   Insert `6` at `nodes[2][2]`

### Map Structure at the end of BFS:
```
nodes = {
  -2: { 2: {4} },
  -1: { 1: {2} },
   0: { 0: {1}, 2: {5} },
   1: { 1: {3} },
   2: { 2: {6} }
}
```

### Result Assembly:
*   Col `-2`: `[4]`
*   Col `-1`: `[2]`
*   Col `0`: `[1, 5]` (since row 0 is visited before row 2)
*   Col `1`: `[3]`
*   Col `2`: `[6]`
*   **Result**: `[[4], [2], [1, 5], [3], [6]]`

---

## 10. Complexity Analysis

*   **Time Complexity**: $\mathcal{O}(N \log N)$
    *   For each of the $N$ nodes, we push and pop from the queue: $\mathcal{O}(N)$.
    *   We insert each value into `std::map` (nesting level 2) and `std::multiset`.
    *   The depth of the outer map is $W$ (width of tree), and inner map is $H$ (height of tree).
    *   Inserting into map of maps with multiset takes $\mathcal{O}(\log W + \log H + \log N)$ which simplifies to $\mathcal{O}(\log N)$ per node.
    *   Thus, total time is $\mathcal{O}(N \log N)$.
*   **Space Complexity**: $\mathcal{O}(N)$
    *   To store the nodes in the map and the queue.

---

## 11. Common Mistakes

1.  **Ignoring same level, same column sorting**: Simply using standard BFS and appending to vectors. If nodes overlap at the same coordinate, they must be sorted.
2.  **Using `unordered_map`**: Using `unordered_map` requires sorting keys manually at the end, which is more complex to code correctly.
3.  **Mismatch of Row/Column Updates**: Updating left child column to `col + 1` instead of `col - 1`.

---

## 12. Edge Cases

| Edge Case | Input Tree | Expected Output | Importance |
| :--- | :--- | :--- | :--- |
| **Overlapping Coordinates** | Root has children `2` (left) and `3` (right). Left has right child `5`. Right has left child `5`. Both `5` overlap at `col=0, row=2`. | `[[2], [1, 5, 5], [3]]` | Tests correct duplicate/overlap sorting. |
| **Skewed Tree (Right)** | `1 -> 2 -> 3` | `[[1], [2], [3]]` | Coordinates are `(0,0)`, `(1,1)`, `(2,2)`. |
| **Single Node** | `[1]` | `[[1]]` | Base validation. |

---

## 13. Interview Explanation

"To perform a vertical order traversal, we represent the nodes as coordinates on a 2D plane. The root starts at `(row = 0, col = 0)`. Moving to a left child shifts the coordinates to `(row + 1, col - 1)`, and a right child shifts them to `(row + 1, col + 1)`.

We use a Breadth-First Search (BFS) combined with an ordered map data structure to group nodes by column and row. Specifically, we use a `std::map<int, std::map<int, std::multiset<int>>>`. The outer map automatically sorts our columns from left to right. The inner map sorts the rows from top to bottom. The `multiset` handles duplicate coordinates, keeping values sorted in ascending order. After traversing all nodes, we iterate through the map to build our final 2D results list. The overall time complexity is $O(N \log N)$ and the space complexity is $O(N)$."

---

## 14. Follow-up Questions

1.  **Q**: Can we optimize this if we know there are no overlaps at the exact same coordinate?
    *   *A*: Yes, we can omit the `multiset` and use a simpler `std::vector` inside the inner map, saving some sorting overhead.
2.  **Q**: How can we optimize this to $O(N)$ time?
    *   *A*: If we precalculate the minimum and maximum column offsets using a fast DFS pass, we can use an array of lists indexed by `col - min_col` instead of an ordered map. However, sorting overlapping nodes at the exact same coordinate still requires a local sort.

---

## 15. Variations

1.  **Top View of Binary Tree**: Only return the top-most node for each vertical column.
2.  **Bottom View of Binary Tree**: Only return the bottom-most node for each vertical column.

---

## 16. Revision Notes

*   **Coordinate mapping formula**:
    *   Left child: `(row + 1, col - 1)`
    *   Right child: `(row + 1, col + 1)`
*   **Key Data Structure**: `map<int, map<int, multiset<int>>>`
*   **Complexity**: $O(N \log N)$ time, $O(N)$ space.

---

## 17. Memorization Version

```cpp
vector<vector<int>> verticalTraversal(TreeNode* root) {
    map<int, map<int, multiset<int>>> nodes;
    queue<pair<TreeNode*, pair<int, int>>> q;
    q.push({root, {0, 0}});
    while (!q.empty()) {
        auto [n, coord] = q.front(); q.pop();
        int r = coord.first, c = coord.second;
        nodes[c][r].insert(n->val);
        if (n->left) q.push({n->left, {r + 1, c - 1}});
        if (n->right) q.push({n->right, {r + 1, c + 1}});
    }
    vector<vector<int>> res;
    for (auto& [c, r_map] : nodes) {
        vector<int> col;
        for (auto& [r, vals] : r_map) 
            col.insert(col.end(), vals.begin(), vals.end());
        res.push_back(col);
    }
    return res;
}
```
*   **Mnemonic**: BFS with coordinates, `map<col, map<row, multiset<val>>>` for auto-sorting, reconstruct.
*   **Complexity**: $O(N \log N)$ time, $O(N)$ space.
