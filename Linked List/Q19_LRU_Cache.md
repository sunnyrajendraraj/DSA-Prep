# LRU Cache Implementation

## 1. Problem Statement

Design a data structure that follows the constraints of a **Least Recently Used (LRU) Cache**.

Implement the `LRUCache` class:
- `LRUCache(int capacity)`: Initialize the LRU cache with positive size `capacity`.
- `int get(int key)`: Return the value of the `key` if the `key` exists, otherwise return `-1`.
- `void put(int key, int value)`: Update the value of the `key` if the `key` exists. Otherwise, add the `key-value` pair to the cache. If the number of keys exceeds the `capacity` from this operation, **evict** the least recently used key.

The functions `get` and `put` must each run in $O(1)$ average time complexity.

### Input
- Series of `get` and `put` operations.

### Output
- Values returned by the `get` operations.

### Constraints
- $1 \le \text{capacity} \le 3000$
- $0 \le key \le 10^4$
- $0 \le value \le 10^5$
- At most $2 \cdot 10^5$ calls will be made to `get` and `put`.

---

## 2. Interview Clarification

Before coding, clarify:
- **Q:** When a key is updated via `put`, does it count as being "recently used"?
  **A:** Yes, any successful read (`get`) or write/update (`put`) makes that key the most recently used (MRU).
- **Q:** What should we do when the capacity is exceeded?
  **A:** Evict the key that was accessed (either read or updated) furthest in the past (Least Recently Used).
- **Q:** Should the cache handle concurrent access?
  **A:** Usually, a single-threaded cache is expected. If multi-threading is mentioned, we can discuss mutexes/locks later.

---

## 3. Thinking Process

Let's build the intuition step-by-step:
1. **Requirements Analysis:**
   - We need $O(1)$ lookup for `get(key)`. The only data structure that provides $O(1)$ average time lookup is a **Hash Map** (like `std::unordered_map` in C++).
   - We need to maintain the order of keys based on their recency.
   - When a key is accessed, we need to move it to the "most recently used" position in $O(1)$ time.
   - When the cache is full, we need to remove the "least recently used" element in $O(1)$ time.
2. **Choosing the Ordering Data Structure:**
   - An array or vector allows $O(1)$ inserts at the end, but moving an element from the middle to the end takes $O(N)$ time because we have to shift elements.
   - A singly linked list allows $O(1)$ insertion at the head. But if we want to remove a node from the middle, we need its predecessor node. Finding the predecessor takes $O(N)$ time unless we store a pointer to the predecessor, which is extremely complex to maintain.
   - A **Doubly Linked List (DLL)** allows us to remove any node in $O(1)$ time because every node has direct pointers to both its successor and predecessor.
3. **Combining Hash Map and DLL:**
   - The **Hash Map** will map a `key` to its corresponding node pointer in the **DLL**: `std::unordered_map<int, Node*>`.
   - The **DLL** will store the actual `(key, value)` pairs.
     - The **Head** of the DLL (right after dummy head) will represent the **Most Recently Used (MRU)** elements.
     - The **Tail** of the DLL (right before dummy tail) will represent the **Least Recently Used (LRU)** elements.
4. **Operations:**
   - **`get(key)`:**
     - Look up `key` in the hash map.
     - If it doesn't exist, return `-1`.
     - If it exists, retrieve the node pointer. Remove the node from its current position in the DLL and insert it at the MRU position (right after dummy head). Return `node->value`.
   - **`put(key, value)`:**
     - Look up `key` in the hash map.
     - If it exists, update the node's value, remove it from its current position in the DLL, and insert it at the MRU position.
     - If it does not exist:
       - If the cache is at capacity, get the LRU node (located at `tail->prev`). Remove this node from the DLL and erase its key from the hash map. Delete the node to free memory.
       - Create a new node with `(key, value)`.
       - Insert the new node at the MRU position in the DLL.
       - Insert the mapping `{key -> new_node}` into the hash map.

---

## 4. Pattern Recognition

This problem uses the **Composite Data Structure Pattern (Hash Map + Doubly Linked List)**.
- **Why this pattern?** We combine the fast retrieval properties of a Hash Map with the fast in-order insertion and deletion properties of a Doubly Linked List.

---

## 5. Brute Force Solution

### Algorithm
1. Store key-value pairs in a dynamic array of pairs: `std::vector<std::pair<int, int>>`.
2. Keep a separate timestamp or counter for each element to track when it was accessed.
3. **`get(key)`**: Linear scan through the array to find `key`. Update its timestamp and return its value. ($O(N)$ time).
4. **`put(key, value)`**:
   - Linear scan to see if `key` exists. If so, update the value and timestamp.
   - If not:
     - If at capacity, find the index of the pair with the smallest timestamp (linear scan). Remove it.
     - Append the new key-value pair with the current timestamp. ($O(N)$ time).

### Complexity Analysis
- **Time Complexity:** $O(N)$ per operation, where $N$ is the capacity.
- **Space Complexity:** $O(\text{capacity})$ to store the elements.

---

## 6. Better Solution

There is no intermediate "better" solution; we directly implement the optimal $O(1)$ Hash Map + DLL design.

---

## 7. Optimal Solution (Hash Map + DLL)

### Algorithm Detail
- We use two dummy nodes: `head` and `tail`.
  - Initially, `head->next = tail` and `tail->prev = head`.
  - Using dummy nodes avoids null-pointer checks when adding/removing nodes at the boundaries.
- Helper functions:
  - `addNode(Node* new_node)`: Inserts `new_node` right after `head` (MRU position).
  - `removeNode(Node* node)`: Unlinks `node` from its current position in the DLL.

### C++17 Code
```cpp
#include <unordered_map>

class LRUCache {
private:
    // Doubly Linked List Node Definition
    struct Node {
        int key;
        int val;
        Node* prev;
        Node* next;
        Node(int k, int v) : key(k), val(v), prev(nullptr), next(nullptr) {}
    };
    
    int capacity;
    std::unordered_map<int, Node*> cache_map;
    Node* head;
    Node* tail;
    
    // Inserts a node right after the dummy head (Most Recently Used)
    void addNode(Node* new_node) {
        Node* temp = head->next;
        
        new_node->next = temp;
        new_node->prev = head;
        
        head->next = new_node;
        temp->prev = new_node;
    }
    
    // Removes a node from its current position in the DLL
    void removeNode(Node* node) {
        Node* prev_node = node->prev;
        Node* next_node = node->next;
        
        prev_node->next = next_node;
        next_node->prev = prev_node;
    }
    
    // Moves an existing node to the MRU position
    void moveToHead(Node* node) {
        removeNode(node);
        addNode(node);
    }

public:
    LRUCache(int cap) : capacity(cap) {
        head = new Node(-1, -1);
        tail = new Node(-1, -1);
        head->next = tail;
        tail->prev = head;
    }
    
    ~LRUCache() {
        // Clean up allocated memory
        Node* curr = head;
        while (curr != nullptr) {
            Node* temp = curr;
            curr = curr->next;
            delete temp;
        }
    }
    
    int get(int key) {
        if (cache_map.find(key) == cache_map.end()) {
            return -1;
        }
        
        Node* node = cache_map[key];
        moveToHead(node); // Accessing the node makes it MRU
        return node->val;
    }
    
    void put(int key, int value) {
        if (cache_map.find(key) != cache_map.end()) {
            // Key already exists: update value and make it MRU
            Node* node = cache_map[key];
            node->val = value;
            moveToHead(node);
        } else {
            // Key does not exist
            if (cache_map.size() == capacity) {
                // Evict the Least Recently Used (LRU) node from DLL and Map
                Node* lru_node = tail->prev;
                cache_map.erase(lru_node->key);
                removeNode(lru_node);
                delete lru_node; // Free memory
            }
            
            // Create and insert new node
            Node* new_node = new Node(key, value);
            cache_map[key] = new_node;
            addNode(new_node);
        }
    }
};
```

---

## 8. Code Walkthrough

1. **`addNode(Node* new_node)`**:
   - `Node* temp = head->next;` saves the current first element.
   - Adjusts four pointers:
     - `new_node->next = temp;`
     - `new_node->prev = head;`
     - `head->next = new_node;`
     - `temp->prev = new_node;`
2. **`removeNode(Node* node)`**:
   - Adjusts two pointers of the neighbors of `node`:
     - `node->prev->next = node->next;`
     - `node->next->prev = node->prev;`
3. **`get(key)`**:
   - If not in `cache_map`, returns `-1`.
   - Otherwise, `moveToHead(node)` shifts it to the front, and we return `node->val`.
4. **`put(key, value)`**:
   - If present, update `node->val = value`, then `moveToHead(node)`.
   - If not present and cache size equals capacity:
     - `lru_node = tail->prev` identifies the oldest node.
     - `cache_map.erase(lru_node->key)` removes it from index lookup.
     - `removeNode(lru_node)` removes it from the list.
     - `delete lru_node` prevents a memory leak.
     - Create new node, map it, and call `addNode(new_node)`.

---

## 9. Dry Run

Suppose capacity is 2.
1. `put(1, 1)`:
   - Cache map: `{1 -> Node(1)}`
   - DLL: `head <-> Node(1, 1) <-> tail`
2. `put(2, 2)`:
   - Cache map: `{1 -> Node(1), 2 -> Node(2)}`
   - DLL: `head <-> Node(2, 2) <-> Node(1, 1) <-> tail`
3. `get(1)`:
   - Node(1) exists. Move to head.
   - DLL: `head <-> Node(1, 1) <-> Node(2, 2) <-> tail`
   - Returns `1`.
4. `put(3, 3)` (exceeds capacity!):
   - LRU node is `tail->prev` which is `Node(2, 2)`.
   - Erase `2` from map and delete `Node(2)`.
   - Add new node `Node(3, 3)` to head.
   - Cache map: `{1 -> Node(1), 3 -> Node(3)}`
   - DLL: `head <-> Node(3, 3) <-> Node(1, 1) <-> tail`
5. `get(2)`:
   - Not in map. Returns `-1`.
6. `put(4, 4)` (exceeds capacity!):
   - LRU node is `tail->prev` which is `Node(1, 1)`.
   - Erase `1` from map and delete `Node(1)`.
   - Add new node `Node(4, 4)` to head.
   - Cache map: `{3 -> Node(3), 4 -> Node(4)}`
   - DLL: `head <-> Node(4, 4) <-> Node(3, 3) <-> tail`

---

## 10. Complexity Analysis

| Operation | Time Complexity (Average) | Space Complexity | Reasoning |
| :--- | :--- | :--- | :--- |
| **`get(key)`** | $O(1)$ | $O(1)$ | Hash Map lookup is $O(1)$, moving node in DLL is $O(1)$. |
| **`put(key, value)`**| $O(1)$ | $O(1)$ | Hash Map insertion/deletion is $O(1)$, adding/deleting DLL node is $O(1)$. |
| **Overall Cache** | - | $O(C)$ | $C$ is the capacity. At most $C$ nodes and map entries are maintained. |

---

## 11. Common Mistakes

1. **Memory Leak during Eviction:**
   - Erasing the key from the map and unlinking the node from the DLL, but forgetting to write `delete lru_node`.
2. **Missing Key Erase on Eviction:**
   - Deleting the node but forgetting `cache_map.erase(lru_node->key)`. The map will contain a dangling pointer, leading to a crash on the next lookup.
3. **Invalidating Node Key Storage:**
   - Not storing the `key` inside the `Node` struct. If the node only contains `value`, when you evict `tail->prev`, you won't know which key to erase from the hash map.
4. **Pointer Order Errors:**
   - Modifying pointers in `addNode` or `removeNode` in the incorrect sequence, causing lost links or circular loops.

---

## 12. Edge Cases

| Edge Case | Scenario | Expected Behavior | Why It Matters |
| :--- | :--- | :--- | :--- |
| **Capacity = 1** | `put(1,1)`, `put(2,2)`, `get(1)` | Evicts immediately on second put. Returns `-1` for `get(1)`. | Boundary limit testing. |
| **Updating same key** | `put(1, 10)`, `put(1, 20)` | Value updates to `20`. No eviction occurs. | Checks if update resets recency correctly. |
| **Get of non-existent key** | `get(99)` | Returns `-1`. | Handles missing items gracefully. |

---

## 13. Interview Explanation

"To implement an LRU Cache with $O(1)$ operations, we combine a Hash Map and a Doubly Linked List. 

The Doubly Linked List maintains the order of elements based on their access frequency: the head represents the most recently used elements, and the tail represents the least recently used elements. The Hash Map stores the key pointing to its corresponding node in the Doubly Linked List. 

For a `get` operation, we find the node in the map, move it to the head of the DLL, and return its value. For a `put` operation, if the key already exists, we update the value and move the node to the head. If it does not exist, we check if we are at capacity. If so, we evict the node right before the dummy tail, delete it from both the list and the hash map, and then insert the new node at the head of the DLL. This design ensures all operations take $O(1)$ average time."

---

## 14. Follow-up Questions

1. **Q:** How would you make this LRU Cache thread-safe?
   **A:** We can wrap `get` and `put` operations with a mutex lock (like `std::mutex` and `std::lock_guard<std::mutex>`). Alternatively, to improve performance, we can use read-write locks (`std::shared_mutex`), allowing multiple concurrent reads but exclusive writes.
2. **Q:** What is LFU Cache? How does it differ?
   **A:** Least Frequently Used (LFU) evicts the key with the lowest access frequency count. If there is a tie, it evicts the least recently used key. It is more complex, requiring frequency tracking maps and multiple linked lists.

---

## 15. Variations

1. **LFU Cache (LeetCode 460):**
   - Evict least frequently used, tie-break with LRU. Uses double maps and list collections.
2. **Design a FIFO Cache:**
   - Evict the oldest key added, regardless of updates or reads. Can be solved using a simple queue and a hash map.

---

## 16. Revision Notes

- **Node Struct:** Must contain BOTH `key` and `val` (so we can find the key to delete from the map during eviction).
- **Dummy head/tail:** Keep them initialized at construction to eliminate null checks.
- **Eviction Order:** LRU node is at `tail->prev`. MRU is placed at `head->next`.

---

## 17. Memorization Version

```cpp
// 1. Map: unordered_map<int, Node*> cacheMap. DLL: head <-> tail.
// 2. addNode(node): link right after head.
// 3. removeNode(node): unlink node from prev and next.
// 4. get(key): if not found return -1. Else, moveToHead(node), return node->val.
// 5. put(key, val): if found, update node->val, moveToHead(node).
// 6. If not found: if (full) { lru = tail->prev; erase(lru->key); removeNode(lru); delete lru; }
// 7. Create new node, addNode(node), cacheMap[key] = node.
```
