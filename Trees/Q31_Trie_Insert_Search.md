# Trie (Prefix Tree) – Insert, Search, and StartsWith

## 1. Problem Statement

Implement a **Trie (Prefix Tree)** data structure. A Trie is a tree-like data structure used to efficiently store and retrieve keys in a dataset of strings. 

You need to implement the `Trie` class with the following methods:
- `void insert(string word)`: Inserts the string `word` into the trie.
- `bool search(string word)`: Returns `true` if the string `word` is in the trie (i.e., was inserted before), and `false` otherwise.
- `bool startsWith(string prefix)`: Returns `true` if there is a previously inserted string `word` that has the prefix `prefix`, and `false` otherwise.

### Constraints
- $1 \le \text{word.length, prefix.length} \le 2000$.
- `word` and `prefix` consist only of lowercase English letters.
- At most $3 \cdot 10^4$ calls in total will be made to `insert`, `search`, and `startsWith`.

---

## 2. Interview Clarification

1. **What characters are supported?**
   - *Answer:* Only lowercase English alphabets (`'a'` to `'z'`). This allows us to use a fixed-size array of size 26 for child pointers.
2. **How should we clean up memory?**
   - *Answer:* In C++, we must implement a destructor to recursively delete all allocated Trie nodes to prevent memory leaks.
3. **Is an empty string valid?**
   - *Answer:* Typically not, but we should make sure our design does not crash on empty strings.

---

## 3. Thinking Process

- **Why a Trie?**
  - If we store strings in a hash set, we get $O(L)$ average lookup where $L$ is the length of the string. But we cannot easily search for prefixes (like `startsWith`).
  - If we store strings in a sorted array, prefix searches take $O(L \log N)$ time.
  - A Trie provides $O(L)$ search time for both exact matches and prefixes, and can share common prefixes, saving space.
- **Node Structure:**
  - Each node needs:
    1. An array of 26 pointers to other Trie nodes: `TrieNode* children[26]`.
    2. A boolean flag `isEndOfWord` to denote if a node represents the final character of a stored string.
- **Operations:**
  - **`insert(word)`**:
    - Start at the root.
    - For each character `c` in `word`:
      - Find index `idx = c - 'a'`.
      - If `children[idx]` is null, create a new `TrieNode`.
      - Move to `children[idx]`.
    - Mark the last node's `isEndOfWord` as `true`.
  - **`search(word)`**:
    - Start at the root.
    - For each character `c` in `word`:
      - Move to `children[c - 'a']`. If it's null, return `false`.
    - Return `isEndOfWord` of the final node.
  - **`startsWith(prefix)`**:
    - Start at the root.
    - For each character `c` in `prefix`:
      - Move to `children[c - 'a']`. If it's null, return `false`.
    - If we successfully traverse the entire prefix, return `true`.

---

## 4. Pattern Recognition

- **Multiway Tree (Trie):** A tree where each node represents a character mapping, useful for dictionary and prefix lookup problems.
- **Character Index Offset:** Translating `'a'-'z'` characters to `0-25` indices via `c - 'a'`.

---

## 5. Brute Force Solution

Using a simple list or set of strings.
- **Insert**: Insert into `std::unordered_set<std::string>`.
- **Search**: `set.count(word)`.
- **StartsWith**: Iterate through all strings in the set and check if they start with the prefix.

### Complexity Analysis
- **Time Complexity:** 
  - `insert`: $O(L)$
  - `search`: $O(L)$
  - `startsWith`: $O(N \cdot L)$ where $N$ is the number of words.
- **Downside:** `startsWith` is extremely slow when the dictionary is large.

---

## 6. Better Solution

There is no intermediate structure between a set and a Trie for this specific prefix problem. We proceed directly to the optimal Trie implementation.

---

## 7. Optimal Solution

### C++17 Code

```cpp
#include <string>
#include <vector>
#include <iostream>

class TrieNode {
public:
    TrieNode* children[26];
    bool isEndOfWord;

    TrieNode() {
        isEndOfWord = false;
        for (int i = 0; i < 26; ++i) {
            children[i] = nullptr;
        }
    }

    ~TrieNode() {
        for (int i = 0; i < 26; ++i) {
            delete children[i];
        }
    }
};

class Trie {
private:
    TrieNode* root;

    // Helper to find node matching a string path
    TrieNode* findNode(const std::string& str) const {
        TrieNode* curr = root;
        for (char c : str) {
            int idx = c - 'a';
            if (curr->children[idx] == nullptr) {
                return nullptr;
            }
            curr = curr->children[idx];
        }
        return curr;
    }

public:
    Trie() {
        root = new TrieNode();
    }

    ~Trie() {
        delete root;
    }

    // Inserts a word into the trie
    void insert(std::string word) {
        TrieNode* curr = root;
        for (char c : word) {
            int idx = c - 'a';
            if (curr->children[idx] == nullptr) {
                curr->children[idx] = new TrieNode();
            }
            curr = curr->children[idx];
        }
        curr->isEndOfWord = true;
    }

    // Returns true if the word is in the trie
    bool search(std::string word) const {
        TrieNode* node = findNode(word);
        return (node != nullptr) && node->isEndOfWord;
    }

    // Returns true if there is any word in the trie that starts with the given prefix
    bool startsWith(std::string prefix) const {
        return findNode(prefix) != nullptr;
    }
};
```

---

## 8. Code Walkthrough

- **`TrieNode` Class**:
  - Initializes `children` pointers to `nullptr`.
  - The destructor `~TrieNode()` recursively calls `delete children[i]` on all children. Since calling `delete` on `nullptr` is safe in C++, this cleanly deallocates the entire trie tree when the root is deleted.
- **`findNode(string str)`**:
  - A private helper that starts at the root and traverses character by character.
  - If a child pointer is missing, it returns `nullptr`.
  - Otherwise, it returns the final node at the end of the path.
- **`insert(string word)`**:
  - Traverses the characters of `word`. If the corresponding child node is absent, it dynamically instantiates one.
  - After the loop, marks the final node as `isEndOfWord = true`.
- **`search(string word)`**:
  - Calls `findNode(word)`. Returns `true` if the node exists and has its `isEndOfWord` flag set.
- **`startsWith(string prefix)`**:
  - Calls `findNode(prefix)` and returns `true` if the returned node is not `nullptr`.

---

## 9. Dry Run

Let's insert `"apple"` and then search for `"app"` and `"apple"`.

### 1. `insert("apple")`
- Root starts at node `R`.
- `c = 'a'`: `R->children[0]` is null $\Rightarrow$ create Node A. Move to Node A.
- `c = 'p'`: Node A `children[15]` is null $\Rightarrow$ create Node P1. Move to Node P1.
- `c = 'p'`: Node P1 `children[15]` is null $\Rightarrow$ create Node P2. Move to Node P2.
- `c = 'l'`: Node P2 `children[11]` is null $\Rightarrow$ create Node L. Move to Node L.
- `c = 'e'`: Node L `children[4]` is null $\Rightarrow$ create Node E. Move to Node E.
- End of word. Mark Node E `isEndOfWord = true`.

### 2. `search("apple")`
- Follow: `R -> A -> P1 -> P2 -> L -> E`.
- Node E exists and `isEndOfWord = true`. Returns `true`.

### 3. `search("app")`
- Follow: `R -> A -> P1 -> P2`.
- Node P2 exists but `isEndOfWord = false`. Returns `false`.

### 4. `startsWith("app")`
- Follow: `R -> A -> P1 -> P2`.
- Path traversal successful (Node P2 is not null). Returns `true`.

---

## 10. Complexity Analysis

| Operation | Time Complexity | Space Complexity | Reasoning |
| :--- | :--- | :--- | :--- |
| **Insert** | $O(L)$ | $O(L \times \Sigma)$ | Where $L$ is the word length and $\Sigma$ is the alphabet size (26). We allocate at most $L$ nodes. |
| **Search** | $O(L)$ | $O(1)$ | We only traverse existing node paths without allocating memory. |
| **StartsWith** | $O(L)$ | $O(1)$ | Similar to search, we traverse the prefix path. |

---

## 11. Common Mistakes

1. **Memory leak in C++:** In C++, leaving out the destructor will result in memory leaks, as nodes are created dynamically using `new`.
2. **Checking only path existence in `search`:** Forgetting to check if `isEndOfWord` is true. `search("app")` should return `false` if only `"apple"` has been inserted.

---

## 12. Edge Cases

| Edge Case | Input | Handling |
| :--- | :--- | :--- |
| **Search empty string**| `""` | `findNode` returns `root`. `root->isEndOfWord` is `false`. |
| **Prefix matches exact word**| `insert("a")`, `search("a")` | Works. Node for `"a"` gets `isEndOfWord = true`. |

---

## 13. Interview Explanation

"A Trie, or Prefix Tree, is a specialized tree structure used to store and retrieve strings efficiently. 

Each node in the Trie represents a character, containing an array of pointers to child nodes (typically size 26 for lowercase letters) and a boolean flag `isEndOfWord` to denote if a stored word ends at this node.

For insertion, we walk down the tree matching the characters of the word, creating new nodes as needed, and finally marking the leaf node's `isEndOfWord` flag to true. 

For searching or prefix matching, we traverse down the path representing the word or prefix. If we hit a null pointer at any step, the string does not exist. If we successfully traverse the entire path, we return true for prefix searches; for exact searches, we also verify that the final node has the `isEndOfWord` flag set. This guarantees all operations run in $O(L)$ time, where $L$ is the length of the string."

---

## 14. Follow-up Questions

1. **How do you optimize space in a Trie if the alphabet size is large or sparse?**
   - *Answer:* Instead of a fixed array of size 26, we can use a hash map `std::unordered_map<char, TrieNode*>` or a binary search tree in each node. This saves space for sparse trees but increases lookup time from $O(1)$ to $O(\log \Sigma)$ or $O(1)$ average map overhead.
2. **What is a Compressed Trie (Radix Tree)?**
   - *Answer:* A Radix Tree compresses single-child branches into a single node with a string label. For example, the path `a -> p -> p -> l -> e` is compressed into a single node with label `"apple"`, reducing node count.

---

## 15. Variations

1. **Trie II (Word Frequency / Prefix Frequency):** Each node stores `countWordsEndingHere` and `countWordsStartingWith` instead of just a boolean.
2. **Search with Wildcard (e.g., '.'):** Requires a DFS search to explore all children when a wildcard character is matched.

---

## 16. Revision Notes

- **Data structure layout:**
  ```cpp
  struct TrieNode {
      TrieNode* children[26];
      bool isEndOfWord;
  };
  ```
- **Time Complexity:** All operations are strictly linear with respect to the length of the string query.

---

## 17. Memorization Version

```text
Trie:
- Node: TrieNode* children[26], bool isEndOfWord.
- Insert(word): Walk root to character nodes. If child is null, create it. Mark last node isEndOfWord = true.
- FindNode(str): Walk root to nodes. If any child is null, return null. Return last node.
- Search(word): node = FindNode(word); return node && node->isEndOfWord.
- StartsWith(prefix): return FindNode(prefix) != null.
Complexity: O(L) time, O(1) extra space for query, O(L) space for insertion.
```
