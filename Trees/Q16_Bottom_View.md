# Bottom View of a Binary Tree

## 1. Problem Statement

Given a Binary Tree, return the values of nodes as they would be seen from the bottom of the tree, ordered from left to right.

A node is visible from the bottom if it is the last node encountered in its vertical column when looking from the top down. 

If there are multiple bottom-most nodes at the same horizontal distance and same level, the node that is processed later in BFS (from left to right) is visible.

*   **Input**:
    *   A pointer to the root of the binary tree: `TreeNode* root`.
*   **Output**:
    *   A vector of integers: `vector<int>` representing the bottom view values ordered from the leftmost column to the rightmost column.
*   **Constraints**:
    *   $1 \le N \le 10^5$ (Number of nodes in the tree)
    *   $1 \le \text{Node value} \le 10^5$
*   **Observations**:
    *   Similar to the top view, we track the horizontal distance (column index) `hd`.
    *   Root is at `0`, left is `hd - 1`, right is `hd + 1`.
    *   Unlike the top view where the *first* node seen in a column wins, for the bottom view, the *last* node seen in a column wins.
    *   BFS naturally processes nodes from top to bottom. Hence, overwriting the value in our map for a given `hd` ensures that nodes lower down in the tree will overwrite higher ones.

---

## 2. Interview Clarification

1.  **Q**: What if the tree is empty?
    *   *A*: Return an empty vector `std::vector<int>{}`.
2.  **Q**: If two nodes are at the same horizontal distance and same level, which one is visible?
    *   *A*: The one that is processed later (the rightmost node at that level/column) is visible. A standard BFS naturally visits the left child before the right child, so the later one will overwrite the map value.
3.  **Q**: Do we need to return the nodes ordered by their horizontal distance?
    *   *A*: Yes, from leftmost column (minimum horizontal distance) to rightmost column (maximum horizontal distance).

---

## 3. Thinking Process

*   **First Instinct**:
    *   We want to track the bottom-most node for each horizontal distance.
    *   If we do BFS, we visit nodes level-by-level from top to bottom.
    *   Any node visited later in the BFS traversal that is at horizontal distance `hd` will be at a row level equal to or greater than the previously visited node at the same `hd`.
    *   Therefore, if we do a standard BFS and simply execute `bottomNodeMap[hd] = curr->val` for every node we pop, the last node processed for each `hd` will overwrite the previous ones.
    *   This naturally leaves only the bottom-most nodes in the map when the BFS finishes.
    *   We can use an ordered `std::map<int, int>` to automatically keep the horizontal distances sorted, making final vector creation simple.

---

## 4. Pattern Recognition

*   **Topic**: Tree Traversal & Coordinate Mapping.
*   **Pattern**: BFS (Level Order) with horizontal distance tracking.
*   **Why**: BFS ensures nodes are visited in top-to-bottom order. By overwriting values in a map, we naturally keep only the last (bottom-most) node seen for each column.

---

## 5. Brute Force Solution

*   **Algorithm**:
    *   Perform a complete vertical order traversal of the tree (e.g., using DFS or BFS to collect all coordinates).
    *   Store all nodes grouped by their column index.
    *   For each column, sort the nodes by their level (row) and horizontal coordinate, then pick the last element.
    *   Sort the columns from left to right.
*   **Complexity**:
    *   **Time Complexity**: $O(N \log N)$ to store and sort all node coordinates.
    *   **Space Complexity**: $O(N)$ to store all nodes in memory.
*   **Pros/Cons**:
    *   *Pros*: Standard reuse of Vertical Traversal logic.
    *   *Cons*: Too much overhead; stores all nodes instead of just updating the latest.

### Brute Force C++17 Code
```cpp
#include <iostream>
#include <vector>
#include <map>
#include <algorithm>

struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};

struct DFSNode {
    int row;
    int val;
};

// DFS helper to collect nodes
void collectAllNodes(TreeNode* root, int row, int col, std::map<int, std::vector<DFSNode>>& colMap) {
    if (!root) return;
    colMap[col].push_back({row, root->val});
    collectAllNodes(root->left, row + 1, col - 1, colMap);
    collectAllNodes(root->right, row + 1, col + 1, colMap);
}

std::vector<int> bottomViewBrute(TreeNode* root) {
    if (!root) return {};
    std::map<int, std::vector<DFSNode>> colMap;
    collectAllNodes(root, 0, 0, colMap);
    
    std::vector<int> result;
    for (auto& [col, nodeList] : colMap) {
        // Sort by row (level) ascending
        std::sort(nodeList.begin(), nodeList.end(), [](const DFSNode& a, const DFSNode& b) {
            return a.row < b.row;
        });
        // Grab the bottom-most node (last in sorted list)
        result.push_back(nodeList.back().val); 
    }
    return result;
}
```

---

## 6. Better Solution

*   *Note*: The transition from the brute-force sorted list approach straight to BFS with simple map overwriting yields the optimal solution. There is no intermediate "better" solution. We will jump directly to the optimal solution.

---

## 7. Optimal Solution

*   **Approach**: BFS with standard Queue and `std::map` (overwriting values).
*   **Algorithm**:
    1.  Initialize a `std::map<int, int> bottomNodeMap` where the key is horizontal distance `hd` and value is the node's value.
    2.  Initialize a queue of pairs: `queue<pair<TreeNode*, int>> q;` storing `(node, hd)`.
    3.  Push `(root, 0)` into `q`.
    4.  While `q` is not empty:
        *   Pop the front element: `curr` and `hd`.
        *   Overwrite the map: `bottomNodeMap[hd] = curr->val`. (Every new node seen at `hd` will replace the older one).
        *   If `curr->left` exists, push `(curr->left, hd - 1)` into `q`.
        *   If `curr->right` exists, push `(curr->right, hd + 1)` into `q`.
    5.  Iterate through `bottomNodeMap` and extract the values into the result vector.

### Optimal C++17 Code
```cpp
#include <iostream>
#include <vector>
#include <queue>
#include <map>

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
     * Function to return the bottom view of a binary tree.
     */
    std::vector<int> bottomView(TreeNode* root) {
        std::vector<int> result;
        if (root == nullptr) {
            return result;
        }

        // Ordered map to store the last node value seen at each horizontal distance (hd)
        std::map<int, int> bottomNodeMap;

        // Queue to store pair of (Node, Horizontal Distance)
        std::queue<std::pair<TreeNode*, int>> q;
        q.push({root, 0});

        while (!q.empty()) {
            auto [curr, hd] = q.front();
            q.pop();

            // Overwrite the horizontal distance with the current node value.
            // This ensures that the lowest node processed in BFS for each column remains.
            bottomNodeMap[hd] = curr->val;

            // Push left child with horizontal distance = hd - 1
            if (curr->left != nullptr) {
                q.push({curr->left, hd - 1});
            }
            // Push right child with horizontal distance = hd + 1
            if (curr->right != nullptr) {
                q.push({curr->right, hd + 1});
            }
        }

        // Extract values in sorted order of horizontal distance
        for (auto const& [hd, val] : bottomNodeMap) {
            result.push_back(val);
        }

        return result;
    }
};
```

---

## 8. Code Walkthrough

1.  **Queue of Pairs**: `queue<pair<TreeNode*, int>> q`
    *   Performs standard level-by-level traversal while carrying the horizontal coordinates.
2.  **Ordered Map**: `map<int, int> bottomNodeMap`
    *   Tracks horizontal columns.
3.  **Overwrite Action**:
    *   `bottomNodeMap[hd] = curr->val`
    *   This is the critical difference from Top View. We do not check if `hd` is already present. We unconditionally assign the value. Since BFS processes nodes top-to-bottom, the last node seen at `hd` will overwrite any previous nodes.
4.  **Extract Output**:
    *   Iterate through `bottomNodeMap` to retrieve values from leftmost to rightmost columns.

---

## 9. Dry Run

Consider the tree:
```
      1
    /   \
   2     3
    \   /
     4 5
```

Coordinates assigned:
*   `1`: `0`
*   `2`: `-1`, `3`: `+1`
*   `4`: `0` (child of 2), `5`: `0` (child of 3)

### BFS execution:
1.  Push `(1, 0)` $\rightarrow$ `q = [(1, 0)]`
2.  Pop `(1, 0)`. `bottomNodeMap = {0: 1}`. Push children: `q = [(2, -1), (3, 1)]`.
3.  Pop `(2, -1)`. `bottomNodeMap = {-1: 2, 0: 1}`. Push child: `q = [(3, 1), (4, 0)]`.
4.  Pop `(3, 1)`. `bottomNodeMap = {-1: 2, 0: 1, 1: 3}`. Push child: `q = [(4, 0), (5, 0)]`.
5.  Pop `(4, 0)`. Overwrites `0`. `bottomNodeMap = {-1: 2, 0: 4, 1: 3}`. `q = [(5, 0)]`.
6.  Pop `(5, 0)`. Overwrites `0` again (right side of BFS level). `bottomNodeMap = {-1: 2, 0: 5, 1: 3}`. `q = []`.

### Result Extraction:
*   Key `-1` $\rightarrow$ `2`
*   Key `0` $\rightarrow$ `5`
*   Key `1` $\rightarrow$ `3`
*   **Result**: `[2, 5, 3]`

---

## 10. Complexity Analysis

*   **Time Complexity**: $\mathcal{O}(N \log N)$
    *   Each of the $N$ nodes is pushed and popped from the queue once: $\mathcal{O}(N)$.
    *   For each node, we insert/overwrite in the map: $\mathcal{O}(\log W)$ where $W$ is the tree width.
    *   Total time complexity: $\mathcal{O}(N \log W) \approx \mathcal{O}(N \log N)$.
*   **Space Complexity**: $\mathcal{O}(N)$
    *   Queue and map store at most $O(N)$ elements.

---

## 11. Common Mistakes

1.  **Checking if the Key Exists**: Beginners often copy the Top View solution and keep the `if (find() == end())` check. This yields the Top View, not the Bottom View.
2.  **Using DFS**: If DFS is used, a node at a higher level might be processed *after* a node at a lower level due to the depth-first ordering, causing incorrect overwriting. DFS for Bottom View requires storing the node's depth level and only overwriting if the new node has a depth $\ge$ the existing node's depth.

---

## 12. Edge Cases

| Edge Case | Input Tree | Expected Output | Importance |
| :--- | :--- | :--- | :--- |
| **Empty Tree** | `nullptr` | `[]` | Avoid null pointer exception. |
| **Single Node** | `[1]` | `[1]` | Base case validation. |
| **Overlapping Bottom Nodes** | Root has children `2` and `3`. `2` has right child `4`. `3` has left child `5`. Both `4` and `5` are at `col=0, row=2`. | `[2, 5, 3]` | `5` is processed after `4` in BFS, so it should overwrite `4`. |

---

## 13. Interview Explanation

"To find the bottom view of a binary tree, we identify the lowest node in each vertical column. We define vertical columns using horizontal distances from the root, where the root is `0`, left children subtract `1`, and right children add `1`.

We perform a Level Order Traversal (BFS) using a queue. We maintain an ordered map that associates each horizontal distance with its corresponding node value. As we pop each node, we overwrite the map's entry for its horizontal distance. Because BFS processes nodes from top to bottom, any node processed later in the traversal will be at a level equal to or deeper than the one processed earlier. This guarantees that when the BFS ends, the map contains only the bottom-most nodes for each column. Finally, we extract the values from our map. This algorithm takes $O(N \log N)$ time and $O(N)$ space."

---

## 14. Follow-up Questions

1.  **Q**: How can we optimize this to $O(N)$ time?
    *   *A*: Just like Top View, we can track the minimum and maximum horizontal distances during BFS. We can then allocate a vector of size `max_hd - min_hd + 1` and map horizontal distances to array indices using an offset. This eliminates the map's logarithmic overhead.
2.  **Q**: How do you implement this using DFS?
    *   *A*: With DFS, we must pass the depth level of each node. We store a pair in the map: `map[hd] = {val, level}`. We overwrite the value only if the current node's `level` is greater than or equal to the stored `level`.

---

## 15. Variations

1.  **Top View**: First node seen in each vertical column.
2.  **Vertical Order Traversal**: All nodes grouped by column.

---

## 16. Revision Notes

*   **BFS Property**: Bottom view is achieved by *unconditional* map overwrites.
*   **Key Formula**: Left child gets `hd - 1`, right child gets `hd + 1`.
*   **Complexity**: $O(N \log N)$ time, $O(N)$ space.

---

## 17. Memorization Version

```cpp
vector<int> bottomView(TreeNode* root) {
    if (!root) return {};
    map<int, int> bottomMap;
    queue<pair<TreeNode*, int>> q;
    q.push({root, 0});
    while (!q.empty()) {
        auto [curr, hd] = q.front(); q.pop();
        bottomMap[hd] = curr->val; // Unconditional overwrite
        if (curr->left) q.push({curr->left, hd - 1});
        if (curr->right) q.push({curr->right, hd + 1});
    }
    vector<int> res;
    for (auto& [hd, val] : bottomMap) res.push_back(val);
    return res;
}
```
*   **Mnemonic**: BFS, `bottomMap[hd] = curr->val` (overwrite), output map values.
*   **Complexity**: $O(N \log N)$ time, $O(N)$ space.
