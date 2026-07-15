# Serialize and Deserialize a Binary Tree

## 1. Problem Statement

Serialization is the process of converting a data structure or object into a sequence of bits so that it can be stored in a file or memory buffer, or transmitted across a network connection link to be reconstructed later in the same or another computer environment.

Design an algorithm to serialize and deserialize a binary tree. There is no restriction on how your serialization/deserialization algorithm should work. You just need to ensure that a binary tree can be serialized to a string and this string can be deserialized to the original tree structure.

*   **Input / Output**:
    *   `string serialize(TreeNode* root)`: Converts a binary tree to a string.
    *   `TreeNode* deserialize(string data)`: Reconstructs the binary tree from the string.
*   **Constraints**:
    *   The number of nodes in the tree is in the range $[0, 10^4]$.
    *   $-1000 \le \text{Node value} \le 1000$.

*   **Observations**:
    *   A normal traversal (like inorder or preorder alone) cannot uniquely reconstruct a binary tree if null pointers are omitted.
    *   However, if we explicitly include markers for `nullptr` (e.g., `#` or `null`) in our traversal representation, we can uniquely reconstruct the binary tree with just a single traversal sequence.
    *   Preorder traversal (Root $\rightarrow$ Left $\rightarrow$ Right) is ideal here because the root node is serialized first. When deserializing, we can construct the tree top-down, which is highly natural.

---

## 2. Interview Clarification

1.  **Q**: What delimiter should we use to separate node values?
    *   *A*: Any character not present in the values, such as a comma `,`.
2.  **Q**: What representation should we use for null nodes?
    *   *A*: A special character, e.g., `#` or `null`. We will use `#`.
3.  **Q**: Can node values be negative?
    *   *A*: Yes, values can be negative, so we must support negative numbers (e.g., `-5`).
4.  **Q**: Are we allowed to use libraries like `stringstream` in C++?
    *   *A*: Yes, `std::stringstream` and `std::getline` are standard and highly recommended for tokenizing.

---

## 3. Thinking Process

*   **Serialization**:
    *   We want to traverse the tree in preorder (Root $\rightarrow$ Left $\rightarrow$ Right).
    *   If `root` is null, we append `#,` to our string.
    *   If `root` is not null, we append `root->val` followed by `,` and then recursively serialize the left and right subtrees.
    *   Example: For tree `1` with left child `2` and right child `3`, the preorder traversal with nulls is `1,2,#,#,3,#,#,`.

*   **Deserialization**:
    *   We receive the string `1,2,#,#,3,#,#,`.
    *   We can tokenize this string by spliting on `,` and storing the tokens in a queue: `q = ["1", "2", "#", "#", "3", "#", "#"]`.
    *   We define a recursive helper function `helper(queue<string>& q)`.
    *   We pop the front token from the queue:
        *   If the token is `#`, it represents a null node. We return `nullptr`.
        *   Otherwise, we convert the token to an integer, create a new `TreeNode` with this value, and recursively build its left and right subtrees:
            *   `root->left = helper(q);`
            *   `root->right = helper(q);`
    *   Return the `root`.

---

## 4. Pattern Recognition

*   **Topic**: Tree Traversal & Serialization.
*   **Pattern**: Preorder DFS with Explicit Null Encoding.
*   **Why**: Preorder is selected because the root node value is serialized before its subtrees. During deserialization, this allows us to create the current node first, and then recursively attach the left and right subtrees in the exact same sequence.

---

## 5. Brute Force Solution

*   *Note*: The standard BFS/Level-order traversal with queue can also be used to serialize/deserialize, but it requires tracking queue states during deserialization and is generally more verbose. DFS preorder with explicit null values is already the cleanest and most optimal solution in terms of code complexity and execution. There is no simpler brute force. We will jump directly to the optimal preorder traversal solution.

---

## 6. Better Solution

*   *Note*: Some developers use Level-order BFS (similar to how LeetCode displays trees: `[1, 2, 3, null, null, 4, 5]`). While intuitive, it is harder to code under interview pressure due to complex index math and queue manipulations. The DFS Preorder method is cleaner and less error-prone.

---

## 7. Optimal Solution

### Optimal C++17 Code
```cpp
#include <iostream>
#include <string>
#include <sstream>
#include <queue>

/**
 * Definition for a binary tree node.
 */
struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};

class Codec {
private:
    /**
     * Helper function to serialize tree recursively (Preorder).
     */
    void serializeHelper(TreeNode* root, std::stringstream& ss) {
        if (root == nullptr) {
            ss << "#,";
            return;
        }

        // Preorder: Root -> Left -> Right
        ss << root->val << ",";
        serializeHelper(root->left, ss);
        serializeHelper(root->right, ss);
    }

    /**
     * Helper function to deserialize tree recursively (Preorder).
     */
    TreeNode* deserializeHelper(std::queue<std::string>& q) {
        if (q.empty()) {
            return nullptr;
        }

        // Get the current token
        std::string val = q.front();
        q.pop();

        // If the token is '#', this indicates a null node
        if (val == "#") {
            return nullptr;
        }

        // Create the node (Root)
        TreeNode* root = new TreeNode(std::stoi(val));

        // Reconstruct subtrees (Left -> Right)
        root->left = deserializeHelper(q);
        root->right = deserializeHelper(q);

        return root;
    }

public:
    // Encodes a tree to a single string.
    std::string serialize(TreeNode* root) {
        std::stringstream ss;
        serializeHelper(root, ss);
        return ss.str();
    }

    // Decodes your encoded data to tree.
    TreeNode* deserialize(std::string data) {
        std::queue<std::string> q;
        std::stringstream ss(data);
        std::string token;

        // Split the string by comma and push tokens to queue
        while (std::getline(ss, token, ',')) {
            if (!token.empty()) {
                q.push(token);
            }
        }

        return deserializeHelper(q);
    }
};
```

---

## 8. Code Walkthrough

### Serialization Walkthrough:
1.  `serializeHelper` is called with `root` and a `stringstream ss`.
2.  If `root` is null, `ss << "#,"` is executed, representing a leaf's terminal branch.
3.  Otherwise, we append the node's integer value and a comma delimiter, then recursively process left and right subtrees.

### Deserialization Walkthrough:
1.  We initialize a `queue<string> q` to hold individual tokens.
2.  `std::getline(ss, token, ',')` splits the serialized string on commas and pushes each token onto `q`.
3.  `deserializeHelper` is called with `q`.
4.  In the helper, we pop the front token.
5.  If token is `"#"`, we return `nullptr`.
6.  Else, we convert the token to integer using `std::stoi` and instantiate a new node.
7.  We recursively resolve left and right child pointers by making subsequent calls to `deserializeHelper(q)`. The queue naturally keeps track of which token belongs to which node across the recursion.

---

## 9. Dry Run

Consider the tree:
```
      1
     / \
    2   3
```

### Serialization:
1.  `serializeHelper(1)`: `ss` gets `"1,"`.
2.  `serializeHelper(2)`: `ss` gets `"1,2,"`.
3.  `serializeHelper(2->left)` (null): `ss` gets `"1,2,#,"`.
4.  `serializeHelper(2->right)` (null): `ss` gets `"1,2,#,#,"`.
5.  `serializeHelper(3)`: `ss` gets `"1,2,#,#,3,"`.
6.  `serializeHelper(3->left)` (null): `ss` gets `"1,2,#,#,3,#,"`.
7.  `serializeHelper(3->right)` (null): `ss` gets `"1,2,#,#,3,#,#,"`.
*   **Result String**: `"1,2,#,#,3,#,#,"`

### Deserialization:
*   Tokens split into `q = ["1", "2", "#", "#", "3", "#", "#"]`

1.  `deserializeHelper()`:
    *   Pop `"1"`. Create `Node(1)`.
    *   `Node(1)->left` = call `deserializeHelper()`.
2.  `deserializeHelper()`:
    *   Pop `"2"`. Create `Node(2)`.
    *   `Node(2)->left` = call `deserializeHelper()`.
3.  `deserializeHelper()`:
    *   Pop `"#"` $\rightarrow$ return `nullptr`. `Node(2)->left = nullptr`.
    *   `Node(2)->right` = call `deserializeHelper()`.
4.  `deserializeHelper()`:
    *   Pop `"#"` $\rightarrow$ return `nullptr`. `Node(2)->right = nullptr`.
    *   `Node(2)` is fully constructed. Return `Node(2)` to `Node(1)->left`.
5.  `Node(1)->right` = call `deserializeHelper()`.
6.  `deserializeHelper()`:
    *   Pop `"3"`. Create `Node(3)`.
    *   `Node(3)->left` = call `deserializeHelper()`.
7.  `deserializeHelper()`:
    *   Pop `"#"` $\rightarrow$ return `nullptr`. `Node(3)->left = nullptr`.
    *   `Node(3)->right` = call `deserializeHelper()`.
8.  `deserializeHelper()`:
    *   Pop `"#"` $\rightarrow$ return `nullptr`. `Node(3)->right = nullptr`.
    *   `Node(3)` is constructed. Return `Node(3)` to `Node(1)->right`.
9.  `Node(1)` is fully constructed. Return `Node(1)`.

---

## 10. Complexity Analysis

*   **Time Complexity**: $\mathcal{O}(N)$
    *   During serialization, we visit each of the $N$ nodes (and their null pointers) once.
    *   During deserialization, we split the string in $\mathcal{O}(N)$ time and process each token exactly once.
*   **Space Complexity**: $\mathcal{O}(N)$
    *   For serialization: the output string and recursion stack use $\mathcal{O}(N)$ space.
    *   For deserialization: the queue of tokens and recursion stack use $\mathcal{O}(N)$ space.

---

## 11. Common Mistakes

1.  **Forgetting null pointers**: Skipping null pointers in serialization. Without them, preorder traversal cannot uniquely define a tree (e.g. tree `1 -> 2` vs `1` with right child `2` both have preorder `1, 2`).
2.  **String copying overhead**: Appending to strings using `s = s + val` inside recursion. This creates a new string on each copy, yielding $\mathcal{O}(N^2)$ time. Using `std::stringstream` keeps append operations $\mathcal{O}(1)$.
3.  **Parsing multi-digit or negative numbers**: Assuming each node is a single digit character. The code must parse arbitrary numbers like `-120`. Using `std::stoi` and tokenizing with a delimiter resolves this.

---

## 12. Edge Cases

| Edge Case | Input Tree | Serialized String | Importance |
| :--- | :--- | :--- | :--- |
| **Empty Tree** | `nullptr` | `"#,"` | Handled properly without crashes. |
| **Single Node** | `[1]` | `"1,#,#,"` | Verifies base reconstruction. |
| **Negative Values** | `[-10]` | `"-10,#,#,"` | Verifies minus character is parsed correctly. |

---

## 13. Interview Explanation

"To serialize and deserialize a binary tree, we must perform a traversal that preserves the structure of the tree. A single traversal (like preorder) is insufficient on its own, but if we explicitly encode null pointers as a special character (e.g., `#`), we can uniquely reconstruct the tree.

I choose **Preorder Traversal (DFS)** for this task. During serialization, we traverse the tree recursively. We write the current node value followed by a delimiter, and if we encounter a null pointer, we write `#`. 

During deserialization, we split the string into a queue of tokens. We recursively reconstruct the tree. We pop the first token: if it is `#`, we return null. Otherwise, we create a node with that value, and assign its left and right children by recursively calling the deserialization helper. This is highly efficient, running in $O(N)$ time and using $O(N)$ auxiliary space for both operations."

---

## 14. Follow-up Questions

1.  **Q**: Can we optimize the size of the serialized string?
    *   *A*: Yes, by serializing to a binary buffer instead of a text string (e.g. using 4 bytes for integers and 1 bit/flag for null nodes).
2.  **Q**: What if we have to use BFS (level-order) traversal instead of DFS?
    *   *A*: We would serialize using a queue. Deserialization would also use a queue. For each popped node, we look at the next two elements in the string; if they are not `#`, we create child nodes, link them, and push them to our queue.

---

## 15. Variations

1.  **Serialize and Deserialize BST**:
    *   *Difference*: Since it's a BST, we can omit null nodes in preorder because BST properties allow unique reconstruction from preorder traversal alone (by using range limits).
2.  **Construct Tree from Preorder and Inorder Traversal**:
    *   Reconstruct without null markers, requiring two traversals.

---

## 16. Revision Notes

*   **Preorder Rule**: Root comes first. So we construct the parent node before its children.
*   **Null Marker**: `#` is critical to restore structure.
*   **Delimiter**: `,` separates variable-length values and negative numbers.

---

## 17. Memorization Version

```cpp
class Codec {
    void ser(TreeNode* r, stringstream& ss) {
        if (!r) { ss << "#,"; return; }
        ss << r->val << ",";
        ser(r->left, ss); ser(r->right, ss);
    }
    TreeNode* deser(queue<string>& q) {
        if (q.empty()) return nullptr;
        string s = q.front(); q.pop();
        if (s == "#") return nullptr;
        TreeNode* r = new TreeNode(stoi(s));
        r->left = deser(q); r->right = deser(q);
        return r;
    }
public:
    string serialize(TreeNode* root) {
        stringstream ss; ser(root, ss); return ss.str();
    }
    TreeNode* deserialize(string data) {
        queue<string> q; stringstream ss(data); string t;
        while (getline(ss, t, ',')) q.push(t);
        return deser(q);
    }
};
```
*   **Mnemonic**: DFS preorder serialize: root, left, right, '#' for null. Deserializer: tokenize string into queue, pop first, if '#' return null; else create node, recurse left then right.
*   **Complexity**: $O(N)$ time, $O(N)$ space.
