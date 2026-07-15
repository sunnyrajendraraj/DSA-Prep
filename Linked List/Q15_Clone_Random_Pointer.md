# Clone Linked List with Random Pointer

## 1. Problem Statement

A linked list of length $N$ is given such that each node contains an additional random pointer, which could point to any node in the list, or `nullptr`.

Construct a **deep copy** of the list. The deep copy should consist of exactly $N$ brand new nodes, where each new node has its value set to the value of its corresponding original node. Both the `next` and `random` pointers of the new nodes should point to new nodes in the copied list such that the pointers in the original list and copied list represent the same list state. **None of the pointers in the new list should point to nodes in the original list.**

Return the head of the copied linked list.

### Input
- `head`: A pointer to the head of a linked list with `next` and `random` pointers.

### Output
- A pointer to the head of the cloned linked list.

### Constraints
- $0 \le N \le 1000$
- $-10^4 \le \text{Node.val} \le 10^4$

---

## 2. Interview Clarification

Before writing any code, clarify the following with the interviewer:
1. **Q:** Can the `random` pointer point to the node itself or form cycles?
   **A:** Yes, the random pointer can point to any node in the list (including itself) or can be `nullptr`.
2. **Q:** Can we modify the original list during the cloning process?
   **A:** Yes, but we must restore the original list to its exact initial state before returning.
3. **Q:** Can we use extra space like a hash map?
   **A:** Yes, but the interviewer will likely ask for an optimal solution that runs in $O(1)$ auxiliary space.

---

## 3. Thinking Process

Let's build the intuition step-by-step:
1. **The Core Difficulty:**
   If we clone the list node-by-node sequentially (along `next` pointers), we can easily create the copy nodes. But how do we set the `random` pointers?
   If the original node `A` has `A->random` pointing to `C`, then the copy node `A'` must point to the copy node `C'`.
   If we haven't created `C'` yet, or if we don't know where `C'` is in memory, we cannot link it.
2. **First Instinct (Using a Map):**
   - If we store the relationship between every original node and its copy, we can solve this.
   - Map: `original_node -> copy_node`.
   - We do a first pass to create all copy nodes and populate the map.
   - We do a second pass to set the `next` and `random` pointers of the copy nodes:
     - `copy_node->next = map[original_node->next]`
     - `copy_node->random = map[original_node->random]`
   - This works in $O(N)$ time, but uses $O(N)$ extra space for the map.
3. **Optimal Insight (Interleaving Nodes):**
   - How can we map `original_node` to `copy_node` without using an external hash map?
   - What if we store the copy node *inside* the original list itself?
   - We can insert each copy node `A'` immediately after its original node `A`:
     `A -> A' -> B -> B' -> C -> C'`
   - By doing this, for any original node `curr`, its copy is always its physical next node: `curr->next`.
   - Furthermore, the copy of `curr->random` is `curr->random->next`!
   - This provides the $O(1)$ space mapping we need. After linking the random pointers, we can split the interleaved list into the original and copied lists.

---

## 4. Pattern Recognition

This problem is a classic example of the **Interleaving Technique (In-place Mapping)**.
- **Why this pattern?** By inserting copied nodes directly after original nodes, we use the pointers of the linked list itself to hold the mapping relationship, eliminating the need for a hash map.

---

## 5. Brute Force Solution

### Algorithm
1. Create a hash map `std::unordered_map<Node*, Node*>` to map original nodes to copy nodes.
2. **First Pass:** Traverse the original list. For each node, create a new node with the same value and store the pair in the map: `map[curr] = new_node`.
3. **Second Pass:** Traverse the original list again. For each node `curr`:
   - Connect the copy's next pointer: `map[curr]->next = map[curr->next]`.
   - Connect the copy's random pointer: `map[curr]->random = map[curr->random]`.
4. Return `map[head]`.

### Complexity Analysis
- **Time Complexity:** $O(N)$ because we traverse the list twice.
- **Space Complexity:** $O(N)$ to store the mapping of $N$ nodes in the hash map.

### C++17 Code
```cpp
#include <unordered_map>

class Node {
public:
    int val;
    Node* next;
    Node* random;
    
    Node(int _val) {
        val = _val;
        next = nullptr;
        random = nullptr;
    }
};

class SolutionBrute {
public:
    Node* copyRandomList(Node* head) {
        if (head == nullptr) return nullptr;
        
        std::unordered_map<Node*, Node*> nodeMap;
        
        // Pass 1: Create copy nodes and map them
        Node* curr = head;
        while (curr != nullptr) {
            nodeMap[curr] = new Node(curr->val);
            curr = curr->next;
        }
        
        // Pass 2: Connect next and random pointers
        curr = head;
        while (curr != nullptr) {
            nodeMap[curr]->next = nodeMap[curr->next];
            nodeMap[curr]->random = nodeMap[curr->random];
            curr = curr->next;
        }
        
        return nodeMap[head];
    }
};
```

---

## 6. Better Solution

There is no intermediate "better" solution; we directly move to the optimal $O(1)$ space solution.

---

## 7. Optimal Solution (Interleaving)

### Algorithm
The optimal approach is completed in three distinct phases:

#### Phase 1: Create Cloned Nodes and Interleave
Insert copy nodes right next to their corresponding original nodes.
- For a node `curr`, create `copy = new Node(curr->val)`.
- Set `copy->next = curr->next`.
- Set `curr->next = copy`.
- Move `curr` to `copy->next`.

```
Original:    A --------> B --------> C
Interleaved: A -> A' -> B -> B' -> C -> C'
```

#### Phase 2: Connect Random Pointers of Copy Nodes
- Traverse the interleaved list. For each original node `curr` (which is at odd positions 1, 3, 5...):
  - If `curr->random` is not null, then the copy's random pointer is:
    `curr->next->random = curr->random->next`
  - Move `curr` to `curr->next->next`.

#### Phase 3: Separate the Interleaved List
Extract the copied list and restore the original list.
- Maintain a dummy node `dummy` to build the copy list, and a pointer `copy_curr = &dummy`.
- Traverse the interleaved list:
  - Extract the copy node: `copy_curr->next = curr->next`.
  - Move `copy_curr` to `copy_curr->next`.
  - Restore original pointer: `curr->next = curr->next->next`.
  - Move `curr` to `curr->next`.

### C++17 Code
```cpp
class Solution {
public:
    Node* copyRandomList(Node* head) {
        if (head == nullptr) {
            return nullptr;
        }
        
        // Phase 1: Create cloned nodes and interleave
        Node* curr = head;
        while (curr != nullptr) {
            Node* copy = new Node(curr->val);
            copy->next = curr->next;
            curr->next = copy;
            curr = copy->next;
        }
        
        // Phase 2: Link random pointers of the cloned nodes
        curr = head;
        while (curr != nullptr) {
            if (curr->random != nullptr) {
                curr->next->random = curr->random->next;
            }
            curr = curr->next->next;
        }
        
        // Phase 3: Separate original and cloned list
        Node* dummy = new Node(0);
        Node* copy_curr = dummy;
        curr = head;
        
        while (curr != nullptr) {
            Node* copy = curr->next;
            
            // Extract the copy node
            copy_curr->next = copy;
            copy_curr = copy;
            
            // Restore original list pointer
            curr->next = copy->next;
            curr = curr->next;
        }
        
        Node* clone_head = dummy->next;
        delete dummy; // Clean up memory
        return clone_head;
    }
};
```

---

## 8. Code Walkthrough

Let's trace Phase 3 (separating the lists) in detail:
1. `Node* copy = curr->next;`
   - Identifies the copy node.
2. `copy_curr->next = copy; copy_curr = copy;`
   - Appends the copy node to the copy list.
3. `curr->next = copy->next;`
   - Restores the original node's connection to the next original node.
4. `curr = curr->next;`
   - Advances `curr` to the next original node to continue separation.

---

## 9. Dry Run

List: `1 -> 2`, where `1->random = 2` and `2->random = 1`.

### Phase 1: Interleaving
- `curr = 1`: Create copy `1'`. `1'->next = 2`. `1->next = 1'`. `curr = 2`.
- `curr = 2`: Create copy `2'`. `2'->next = nullptr`. `2->next = 2'`. `curr = nullptr`.
- Interleaved list: `1 -> 1' -> 2 -> 2' -> nullptr`.

### Phase 2: Connecting Random Pointers
- `curr = 1`: `curr->random` is `2`.
  - `curr->next->random = curr->random->next` $\implies$ `1'->random = 2->next` $\implies$ `1'->random = 2'`.
  - `curr = 1'->next = 2`.
- `curr = 2`: `curr->random` is `1`.
  - `curr->next->random = curr->random->next` $\implies$ `2'->random = 1->next` $\implies$ `2'->random = 1'`.
  - `curr = 2'->next = nullptr`.

### Phase 3: Separation
- Initial: `dummy(0)`, `copy_curr = dummy`, `curr = 1`.
- **Step 1:**
  - `copy = 1'`.
  - `copy_curr->next = 1'` $\implies$ `dummy -> 1'`. `copy_curr = 1'`.
  - `curr->next = 1'->next` $\implies$ `1->next = 2`.
  - `curr = 2`.
- **Step 2:**
  - `copy = 2'`.
  - `copy_curr->next = 2'` $\implies$ `dummy -> 1' -> 2'`. `copy_curr = 2'`.
  - `curr->next = 2'->next` $\implies$ `2->next = nullptr`.
  - `curr = nullptr`.

Original list restored: `1 -> 2`. Copy list separated: `1' -> 2'`.

---

## 10. Complexity Analysis

| Solution | Time Complexity | Space Complexity | Details |
| :--- | :--- | :--- | :--- |
| **Hash Map (Brute)** | $O(N)$ | $O(N)$ | Map size scales linearly with list length. |
| **Interleaving (Optimal)** | $O(N)$ | $O(1)$ | No extra structures. Inline pointer restructuring only. |

---

## 11. Common Mistakes

1. **Forgetting Null Checks on Random Pointers:**
   - Writing `curr->next->random = curr->random->next` without verifying if `curr->random` is `nullptr`. If `curr->random` is null, trying to access `curr->random->next` causes a null-pointer dereference.
2. **Failing to Restore the Original List:**
   - Interviewers often verify if the original list is intact after your algorithm runs. Make sure Phase 3 cleanly unlinks the original nodes from the copy nodes.
3. **Improper Pointer Stepping:**
   - In Phase 2, moving by `curr = curr->next` instead of `curr = curr->next->next` will step into copy nodes, corrupting the logic.

---

## 12. Edge Cases

| Edge Case | Input | Expected Output | Why It Matters |
| :--- | :--- | :--- | :--- |
| **Empty List** | `nullptr` | `nullptr` | Must return null without throwing errors. |
| **Single Node (Self Random)** | `1`, `1->random = 1` | `1'`, `1'->random = 1'` | Verifies cycle handling. |
| **Single Node (Null Random)** | `1`, `1->random = nullptr` | `1'`, `1'->random = nullptr` | Verifies null-pointer safety. |
| **All Randoms pointing to Head** | `A->B->C`, all random $\to$ `A` | Copy list with randoms $\to$ `A'` | Verifies target copies are mapped correctly. |

---

## 13. Interview Explanation

"To clone a linked list with random pointers in $O(N)$ time and $O(1)$ auxiliary space, we can use the interleaving technique. 

We do this in three passes. In the first pass, we clone each node and insert it immediately after the original node. This creates an interleaved list where the copy of any node is always its direct next node. In the second pass, we set the random pointers: the copy node's random pointer will point to the copy of the original node's random pointer, which is `curr->random->next`. In the final pass, we separate the interleaved list into two distinct lists: restoring the original list and extracting the cloned list. This avoids using a hash map, reducing our space complexity to $O(1)$."

---

## 14. Follow-up Questions

1. **Q:** Can we solve this problem using recursion (DFS)?
   **A:** Yes. We can treat the linked list as a graph where `next` and `random` are directed edges. Using a standard DFS clone graph algorithm with a visited map, we can copy the list. However, this uses $O(N)$ stack space.
2. **Q:** Is the optimal solution thread-safe?
   **A:** No, because it temporarily mutates the original list. If another thread tries to read the list during Phase 1 or 2, it will read incorrect or interleaved data.

---

## 15. Variations

1. **Clone a Directed Graph:**
   - Generalizes the random and next pointers to an arbitrary list of neighbors. Must use a hash map to track visited nodes.

---

## 16. Revision Notes

- **Memory Layout:**
  `Original: A -> B`
  `Interleaved: A -> A' -> B -> B'`
- **Random Pointer Rule:**
  `copy->random = original->random->next` (guard against null original->random).
- **Separation Loop:** Restores original next pointers while copying next pointers to the copy list.

---

## 17. Memorization Version

```cpp
// 1. Interleave: copy = new Node(curr->val), copy->next = curr->next, curr->next = copy.
// 2. Set randoms: if (curr->random) curr->next->random = curr->random->next.
// 3. Separate: copy = curr->next, copy_curr->next = copy, curr->next = copy->next.
```
