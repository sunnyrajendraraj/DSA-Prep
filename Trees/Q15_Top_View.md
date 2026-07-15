# Top View of a Binary Tree

## 1. Problem Statement

Given a Binary Tree, return the values of nodes as they would be seen from the top of the tree, ordered from left to right.

A node is visible from the top if it is the first node encountered in its vertical column when looking from the top down.

*   **Input**:
    *   A pointer to the root of the binary tree: `TreeNode* root`.
*   **Output**:
    *   A vector of integers: `vector<int>` representing the top view values ordered from the leftmost column to the rightmost column.
*   **Constraints**:
    *   $1 \le N \le 10^5$ (Number of nodes in the tree)
    *   $1 \le \text{Node value} \le 10^5$
*   **Observations**:
    *   We use horizontal distance (column index) to determine top view visibility.
    *   The root has a horizontal distance of `0`.
    *   Its left child is at `-1`, right child is at `+1`.
    *   The first node we visit at a specific horizontal distance (using BFS) will be the one visible from the top.
    *   Subsequent nodes at the same horizontal distance are hidden under the top-most node.

---

## 2. Interview Clarification

1.  **Q**: What if the tree is empty?
    *   *A*: Return an empty vector `std::vector<int>{}`.
2.  **Q**: If two nodes are at the same horizontal distance and same level, which one is visible?
    *   *A*: In a standard BFS level-order traversal, the node visited first (from left to right) is selected.
3.  **Q**: Do we need to return the nodes ordered by their horizontal distance?
    *   *A*: Yes, the output should start from the leftmost column (minimum horizontal distance) to the rightmost column (maximum horizontal distance).

---

## 3. Thinking Process

*   **First Instinct**:
    *   We want to assign a column index to each node.
    *   If we use DFS, we might visit a lower node before an upper node in the same column (e.g., in a deep left subtree vs. a shallow right subtree).
    *   Therefore, **BFS (Level Order Traversal)** is ideal here. Since BFS visits nodes level by level from top to bottom, the first node we encounter at any horizontal distance `HD` is guaranteed to be the highest node at that column.
    *   We can use a map: `std::map<int, int> topNodeMap` where the key is the horizontal distance `HD` and the value is the node's value.
    *   As we do BFS, if `HD` is not already present in `topNodeMap`, we add it: `topNodeMap[HD] = node->val`. If it is already present, we ignore it because a node higher up has already claimed it.
    *   Using an ordered `std::map` ensures that the keys are sorted. At the end, we can simply copy the values from the map to our output vector.

---

## 4. Pattern Recognition

*   **Topic**: Tree Traversal & Coordinate Mapping.
*   **Pattern**: BFS (Level Order) with horizontal distance tracking.
*   **Why**: BFS ensures we process nodes top-to-bottom. A map helps filter and store only the first encountered node for each vertical column.

---

## 5. Brute Force Solution

*   **Algorithm**:
    *   Perform a complete vertical order traversal of the tree (e.g., using DFS or BFS to collect all coordinates).
    *   Store all nodes grouped by their column index.
    *   For each column, sort the nodes by their level (row) and value, then pick the first element.
    *   Sort the columns from left to right.
*   **Complexity**:
    *   **Time Complexity**: $O(N \log N)$ to store and sort all node coordinates.
    *   **Space Complexity**: $O(N)$ to store all nodes in memory.
*   **Pros/Cons**:
    *   *Pros*: Heavy reuse of the "Vertical Order Traversal" solution.
    *   *Cons*: Inefficient, because we store all nodes of the tree, even though we only care about the top node of each column.

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

std::vector<int> topViewBrute(TreeNode* root) {
    if (!root) return {};
    std::map<int, std::vector<DFSNode>> colMap;
    collectAllNodes(root, 0, 0, colMap);
    
    std::vector<int> result;
    for (auto& [col, nodeList] : colMap) {
        // Sort by row (level) ascending
        std::sort(nodeList.begin(), nodeList.end(), [](const DFSNode& a, const DFSNode& b) {
            return a.row < b.row;
        });
        result.push_back(nodeList[0].val); // Grab the top-most node
    }
    return result;
}
```

---

## 6. Better Solution

*   *Note*: The transition from sorting a full DFS list to performing a simple BFS with a queue and map forms the optimal solution. There is no intermediate "better" solution that is not already optimal. We will jump directly to the optimal solution.

---

## 7. Optimal Solution

*   **Approach**: BFS with standard Queue and `std::map`.
*   **Algorithm**:
    1.  Initialize a `std::map<int, int> topNodeMap` where the key is horizontal distance `hd` and value is the node's value.
    2.  Initialize a queue of pairs: `queue<pair<TreeNode*, int>> q;` storing `(node, hd)`.
    3.  Push `(root, 0)` into `q`.
    4.  While `q` is not empty:
        *   Pop the front element: `curr` and `hd`.
        *   If `hd` is not in `topNodeMap`, insert it: `topNodeMap[hd] = curr->val`.
        *   If `curr->left` exists, push `(curr->left, hd - 1)` into `q`.
        *   If `curr->right` exists, push `(curr->right, hd + 1)` into `q`.
    5.  Iterate through `topNodeMap` and extract the values into the result vector.

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
     * Function to return the top view of a binary tree.
     */
    std::vector<int> topView(TreeNode* root) {
        std::vector<int> result;
        if (root == nullptr) {
            return result;
        }

        // Ordered map to store first node value at each horizontal distance (hd)
        std::map<int, int> topNodeMap;

        // Queue to store pair of (Node, Horizontal Distance)
        std::queue<std::pair<TreeNode*, int>> q;
        q.push({root, 0});

        while (!q.empty()) {
            auto [curr, hd] = q.front();
            q.pop();

            // If the horizontal distance is visited for the first time, 
            // it is the top-most node of this vertical column.
            if (topNodeMap.find(hd) == topNodeMap.end()) {
                topNodeMap[hd] = curr->val;
            }

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
        for (auto const& [hd, val] : topNodeMap) {
            result.push_back(val);
        }

        return result;
    }
};
```

---

## 8. Code Walkthrough

1.  **Queue of Pairs**: `queue<pair<TreeNode*, int>> q`
    *   Tracks the exploration frontier along with the Horizontal Distance (`hd`).
2.  **Ordered Map**: `map<int, int> topNodeMap`
    *   Maps `hd` to the node value. Insertion only happens once per key.
3.  **Top Node Selection Check**:
    *   `if (topNodeMap.find(hd) == topNodeMap.end())`
    *   This is the key check: it ensures that once a column is claimed by a higher-level node, it is never overwritten by a lower-level node.
4.  **Sorted Aggregation**:
    *   Iterating through `topNodeMap` visits horizontal distances from left to right (e.g. `-2, -1, 0, 1, 2`), which matches the expected output sequence.

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
2.  Pop `(1, 0)`. `0` not in map. `topNodeMap = {0: 1}`. Push children: `q = [(2, -1), (3, 1)]`.
3.  Pop `(2, -1)`. `-1` not in map. `topNodeMap = {-1: 2, 0: 1}`. Push child: `q = [(3, 1), (4, 0)]`.
4.  Pop `(3, 1)`. `1` not in map. `topNodeMap = {-1: 2, 0: 1, 1: 3}`. Push child: `q = [(4, 0), (5, 0)]`.
5.  Pop `(4, 0)`. `0` is ALREADY in map. Ignore. `q = [(5, 0)]`.
6.  Pop `(5, 0)`. `0` is ALREADY in map. Ignore. `q = []`.

### Result Extraction:
*   Key `-1` $\rightarrow$ `2`
*   Key `0` $\rightarrow$ `1`
*   Key `1` $\rightarrow$ `3`
*   **Result**: `[2, 1, 3]`

---

## 10. Complexity Analysis

*   **Time Complexity**: $\mathcal{O}(N \log N)$
    *   We visit each of the $N$ nodes exactly once: $\mathcal{O}(N)$.
    *   For each node, we check and insert into a `std::map`. The size of the map is bounded by the width of the tree $W$ (which is at most $N$).
    *   Map lookup and insertion take $\mathcal{O}(\log W)$ time.
    *   Total time complexity: $\mathcal{O}(N \log W) \approx \mathcal{O}(N \log N)$.
*   **Space Complexity**: $\mathcal{O}(N)$
    *   The map contains at most $W$ elements.
    *   The queue contains at most the maximum width of the tree $W$ at any level.
    *   In the worst case (a broad complete tree), $W \approx N/2$. Hence, $\mathcal{O}(N)$ space is required.

---

## 11. Common Mistakes

1.  **Using DFS without tracking level heights**: If you use DFS, you might visit a lower node in column `0` before visiting the root's right child which might also occupy column `0` (or similar). DFS does not guarantee top-down order without explicitly checking and comparing node depths. Use BFS to easily handle this.
2.  **Overwriting Map Values**: Using `topNodeMap[hd] = curr->val` directly without checking `find() == end()`. This turns the top view into a bottom view!

---

## 12. Edge Cases

| Edge Case | Input Tree | Expected Output | Importance |
| :--- | :--- | :--- | :--- |
| **Empty Tree** | `nullptr` | `[]` | Check handling of null pointers. |
| **Single Node** | `[1]` | `[1]` | Simple base case. |
| **Left Skewed Tree** | `1 -> 2 -> 3` | `[3, 2, 1]` | Correctly maps columns `-2, -1, 0`. |
| **Right Skewed Tree**| `1 -> 2 -> 3` | `[1, 2, 3]` | Correctly maps columns `0, 1, 2`. |

---

## 13. Interview Explanation

"To find the top view of a binary tree, we want to capture the first node visible in each vertical column. We can represent columns as horizontal distances from the root, where the root is at distance `0`, its left child is at `-1`, and its right child is at `+1`.

We perform a Level Order Traversal (BFS) using a queue to ensure we process nodes from top to bottom. We also maintain an ordered map that maps each horizontal distance to its corresponding node value. During our traversal, when we process a node, we check if its horizontal distance is already in our map. If it is not, we store it. Since BFS processes nodes level-by-level, the first node to occupy a horizontal distance is guaranteed to be the highest. Finally, we copy the map values into our output array. This gives us a time complexity of $O(N \log N)$ and a space complexity of $O(N)$."

---

## 14. Follow-up Questions

1.  **Q**: Can we optimize the time complexity to $O(N)$?
    *   *A*: Yes. Instead of an ordered `std::map`, we can track the minimum and maximum horizontal distances seen so far. We can store the nodes in an array/vector of size `max_hd - min_hd + 1`, using an offset of `abs(min_hd)` to map negative columns to positive array indices. This avoids the $O(\log W)$ map insertion overhead.
2.  **Q**: What if we are forced to use DFS? How do we implement it?
    *   *A*: In DFS, we must store both the node value and its level (depth). We update our map if the horizontal distance is not visited, OR if we find a node at the same horizontal distance with a smaller level (depth).

---

## 15. Variations

1.  **Bottom View**: Get the last node visible in each vertical column.
2.  **Vertical Order Traversal**: Get all nodes in each vertical column.

---

## 16. Revision Notes

*   **Horizontal Distance**: left goes to `hd - 1`, right goes to `hd + 1`.
*   **BFS Rule**: First node to claim `hd` wins.
*   **Complexity**: $O(N \log N)$ time (using map), $O(N)$ space.

---

## 17. Memorization Version

```cpp
vector<int> topView(TreeNode* root) {
    if (!root) return {};
    map<int, int> topMap;
    queue<pair<TreeNode*, int>> q;
    q.push({root, 0});
    while (!q.empty()) {
        auto [curr, hd] = q.front(); q.pop();
        if (topMap.find(hd) == topMap.end()) {
            topMap[hd] = curr->val;
        }
        if (curr->left) q.push({curr->left, hd - 1});
        if (curr->right) q.push({curr->right, hd + 1});
    }
    vector<int> res;
    for (auto& [hd, val] : topMap) res.push_back(val);
    return res;
}
```
*   **Mnemonic**: BFS, `map[hd]` first-write-only, left `hd - 1`, right `hd + 1`.
*   **Complexity**: $O(N \log N)$ time, $O(N)$ space.
