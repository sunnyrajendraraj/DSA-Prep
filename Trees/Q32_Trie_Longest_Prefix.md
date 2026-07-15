# Trie – Longest Prefix Matching

## 1. Problem Statement

Given a dictionary of words and a query string, find the **longest prefix** of the query string that exists in the dictionary. 

If no prefix of the query string is present in the dictionary, return an empty string.

### Input
- `dictionary`: A list of strings.
- `query`: A string.

### Output
- A string representing the longest prefix match.

### Constraints
- $1 \le \text{dictionary.length} \le 10^4$.
- $1 \le \text{word.length} \le 100$.
- $1 \le \text{query.length} \le 1000$.
- All strings consist of lowercase English letters.

---

## 2. Interview Clarification

1. **Does the matching prefix have to be a complete word in the dictionary?**
   - *Answer:* Yes, the matching prefix must be a complete word present in the dictionary (not just a partial prefix of some word).
2. **What if multiple words match the prefix?**
   - *Answer:* We want the *longest* match.
3. **What should we return if no prefix matches?**
   - *Answer:* Return an empty string `""`.

---

## 3. Thinking Process

Let's trace:
Suppose `dictionary = ["are", "area", "base", "cat"]` and `query = "arealand"`.
- Prefixes of `"arealand"` are: `"a"`, `"ar"`, `"are"`, `"area"`, `"areal"`, etc.
- Matching complete words in the dictionary: `"are"`, `"area"`.
- The longest is `"area"`.

### Approach 1: Brute Force
- Check all prefixes of `query`: `query.substr(0, 1)`, `query.substr(0, 2)`, ..., `query.substr(0, L)`.
- For each prefix, search it in the dictionary.
- Keep the longest matching prefix.
- Searching each prefix in a dictionary of size $N$ takes $O(N \cdot L)$ time. For $L$ prefixes, total time is $O(N \cdot L^2)$.

### Approach 2: Hash Set Lookup
- Store the dictionary in a `std::unordered_set<std::string>`.
- For $i$ from $L$ down to $1$:
  - Extract prefix `query.substr(0, i)`.
  - If the prefix exists in the set, return it immediately (since we check from longest to shortest).
- Time Complexity: $O(L^2)$ since hashing a string of length $L$ takes $O(L)$ time, and we do it $L$ times.
- This is efficient, but space complexity is high, and it doesn't scale well for streaming prefixes.

### Approach 3: Optimal (Trie-based)
- Insert all dictionary words into a Trie.
- Traverse the Trie using characters of the `query`.
- As we traverse, whenever we reach a node with `isEndOfWord == true`, we update the `longestPrefixIndex` to the current index.
- If we hit a null child, we stop early.
- Return the substring of `query` up to `longestPrefixIndex`.
- This takes $O(D \cdot W)$ to build the Trie (where $D$ is dictionary size, $W$ is word length) and only $O(L)$ to query.

---

## 4. Pattern Recognition

- **Prefix Structures (Trie):** A Trie matches prefixes character-by-character along a single path.
- **State tracking during traversal:** Storing the index of the last valid termination state (`isEndOfWord`).

---

## 5. Brute Force Solution

Iterating through all prefixes of the query string and scanning the dictionary.

### C++17 Code
```cpp
#include <string>
#include <vector>
#include <algorithm>

class SolutionBrute {
public:
    std::string longestPrefix(const std::vector<std::string>& dictionary, const std::string& query) {
        std::string result = "";
        for (int i = 1; i <= static_cast<int>(query.length()); ++i) {
            std::string prefix = query.substr(0, i);
            if (std::find(dictionary.begin(), dictionary.end(), prefix) != dictionary.end()) {
                result = prefix;
            }
        }
        return result;
    }
};
```

### Complexity Analysis
- **Time Complexity:** $O(N \cdot L^2)$ where $N = \text{dictionary.size()}$ and $L = \text{query.length()}$.
- **Space Complexity:** $O(L)$ for the substring copies.

---

## 6. Better Solution (Hash Set Lookup)

Use a hash set to lookup prefixes from longest to shortest.

### C++17 Code
```cpp
#include <string>
#include <vector>
#include <unordered_set>

class SolutionBetter {
public:
    std::string longestPrefix(const std::vector<std::string>& dictionary, const std::string& query) {
        std::unordered_set<std::string> dictSet(dictionary.begin(), dictionary.end());
        
        for (int i = static_cast<int>(query.length()); i > 0; --i) {
            std::string prefix = query.substr(0, i);
            if (dictSet.count(prefix)) {
                return prefix;
            }
        }
        return "";
    }
};
```

### Complexity Analysis
- **Time Complexity:** $O(N \cdot W + L^2)$ where $W$ is average dictionary word length and $L$ is query length.
- **Space Complexity:** $O(N \cdot W)$ to store words in the set.

---

## 7. Optimal Solution (Trie-based)

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

public:
    Trie() {
        root = new TrieNode();
    }

    ~Trie() {
        delete root;
    }

    // Insert a word into the Trie
    void insert(const std::string& word) {
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

    // Finds the longest prefix of the query that matches a complete word in the Trie
    std::string findLongestPrefix(const std::string& query) const {
        TrieNode* curr = root;
        int longestMatchIndex = -1; // -1 denotes no match found yet
        
        for (int i = 0; i < static_cast<int>(query.length()); ++i) {
            int idx = query[i] - 'a';
            if (curr->children[idx] == nullptr) {
                break; // No further match possible
            }
            curr = curr->children[idx];
            if (curr->isEndOfWord) {
                longestMatchIndex = i; // Update with the index of the last valid word character
            }
        }

        if (longestMatchIndex == -1) {
            return "";
        }
        return query.substr(0, longestMatchIndex + 1);
    }
};

class Solution {
public:
    std::string longestPrefixMatching(const std::vector<std::string>& dictionary, const std::string& query) {
        Trie trie;
        for (const std::string& word : dictionary) {
            trie.insert(word);
        }
        return trie.findLongestPrefix(query);
    }
};
```

---

## 8. Code Walkthrough

- **`TrieNode` & `Trie`**: Standard Trie construction setup.
- **`findLongestPrefix(query)`**:
  - Initializes `longestMatchIndex = -1`.
  - Loops over characters of `query` at index `i`.
  - Moves down the child pointer `curr->children[idx]`. If it is `nullptr`, we break out of the loop immediately because no longer prefix exists.
  - If `curr->isEndOfWord` is true, we update `longestMatchIndex = i`. This marks the end of a valid dictionary word.
  - If we exit the loop, we return `query.substr(0, longestMatchIndex + 1)` which grabs the query up to the last successful word boundary. If no match was found (`-1`), returns `""`.

---

## 9. Dry Run

### Setup
`dictionary = ["are", "area", "base", "cat"]`
`query = "arealand"`

### Trie representation:
- `a -> r -> e (isEndOfWord = true) -> a (isEndOfWord = true)`
- `b -> a -> s -> e (isEndOfWord = true)`
- `c -> a -> t (isEndOfWord = true)`

### Search Query `"arealand"`
- `i = 0`, `c = 'a'`: child exists. `curr` points to `Node 'a'`.
- `i = 1`, `c = 'r'`: child exists. `curr` points to `Node 'r'`.
- `i = 2`, `c = 'e'`: child exists. `curr` points to `Node 'e'`. It is `isEndOfWord = true`. `longestMatchIndex = 2`.
- `i = 3`, `c = 'a'`: child exists. `curr` points to `Node 'a'`. It is `isEndOfWord = true`. `longestMatchIndex = 3`.
- `i = 4`, `c = 'l'`: child `children['l' - 'a']` is null.
- Break loop.
- Return `query.substr(0, 3 + 1)` $\Rightarrow$ `"area"`.

---

## 10. Complexity Analysis

| Operation / Phase | Time Complexity | Space Complexity | Reasoning |
| :--- | :--- | :--- | :--- |
| **Trie Build** | $O(N \cdot W)$ | $O(N \cdot W \cdot \Sigma)$ | Where $N$ is dictionary size, $W$ is average word length, and $\Sigma = 26$. |
| **Prefix Query** | $O(L)$ | $O(1)$ | Where $L$ is the length of the query string. We only traverse the query string once. |

---

## 11. Common Mistakes

1. **Incorrect substring length:** Writing `query.substr(0, longestMatchIndex)` instead of `query.substr(0, longestMatchIndex + 1)`. Since indices are 0-based, a match ending at index 3 has length 4.
2. **Checking only leaf nodes:** Forgetting that an internal node can also have `isEndOfWord = true` (e.g., `"are"` is inside `"area"`).

---

## 12. Edge Cases

| Edge Case | Input | Handling |
| :--- | :--- | :--- |
| **No matching prefix** | `dict = ["cat"]`, `query = "dog"` | Returns `""` (index remains `-1`). |
| **Query matches word exactly**| `dict = ["cat"]`, `query = "cat"` | Returns `"cat"`. |
| **Query shorter than dict words**| `dict = ["cater"]`, `query = "cat"`| Returns `""` (no word completes at `"cat"`). |

---

## 13. Interview Explanation

"To solve the Longest Prefix Matching problem, we can use a Trie. 

First, we insert all words from the dictionary into our Trie. 

Then, we traverse the Trie using the characters of the query string. During the traversal, we keep track of the last index where a valid dictionary word ends (denoted by the `isEndOfWord` flag on that Trie node). 

If we encounter a character that has no transition in the Trie, we terminate the search early since no longer match is possible. 

Finally, we return the portion of the query string up to the last matched index we recorded. This achieves an optimal lookup time of $O(L)$ where $L$ is the length of the query string, which is highly efficient for multiple queries."

---

## 14. Follow-up Questions

1. **What if the dictionary is dynamic (words added/deleted)?**
   - *Answer:* A Trie handles dynamic insertions in $O(W)$ time. Deletions can also be handled by walking the path and decremented references or deleting nodes if they have no other children.
2. **How does this compare to a Hash Set for prefix matching?**
   - *Answer:* A Hash Set requires hashing the query prefixes of sizes $1$ to $L$, which takes $O(L^2)$ time. A Trie avoids redundant character checks, taking only $O(L)$ time.

---

## 15. Variations

1. **Shortest Prefix Matching:** Return the shortest matching complete word from the dictionary (return immediately on the first `isEndOfWord` encountered).
2. **Replace Words (e.g., LeetCode 648):** Replace every word in a sentence with its shortest prefix match in a dictionary.

---

## 16. Revision Notes

- **Tracking match:** Only update the match index when `isEndOfWord` is true.
- **Substring range:** `substr(0, index + 1)`.
- **Query Complexity:** $O(L)$ time where $L$ is the query length.

---

## 17. Memorization Version

```text
Trie Longest Prefix Matching:
1. Build Trie with all dictionary words.
2. Traverse query string character by character:
   - If child pointer is null, break.
   - Go to child.
   - If node.isEndOfWord, update longestIndex = current_char_index.
3. Return query.substr(0, longestIndex + 1) if longestIndex != -1, else "".
Complexity: Query is O(L) time.
```
