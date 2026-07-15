# Linked List — Complete Topic Reference

## Q01 Reverse LL

### 1. Problem Explanation

Given the `head` of a singly linked list, reverse the list in-place and return the new head of the reversed list.

### Input
- `head`: A pointer to the first node of a singly linked list.

### Output
- A pointer to the new head of the reversed singly linked list.

### Constraints
- The number of nodes in the list is in the range `[0, 5000]`.
- `-5000 <= Node.val <= 5000`

### Observations
- A singly linked list can only be traversed in the forward direction because each node only has a pointer to its `next` node.
- Reversing the list requires modifying the `next` pointer of each node to point to its predecessor instead of its successor.
- We must be careful not to lose the reference to the rest of the list when we reassign a node's `next` pointer.


### 2. Brute Solution

The brute force solution involves copying the values of the linked list into an auxiliary container (like a `std::vector` or `std::stack`), reversing the container, and writing the values back to the list.

### Algorithm
1. Traverse the linked list and push all node values onto a stack.
2. Traverse the linked list a second time, pop values from the stack, and update the nodes' values.
3. Return the original head.

### Dry Run
Input: `1 -> 2 -> 3 -> nullptr`
1. First pass: Stack = `[1, 2, 3]` (top is 3).
2. Second pass:
   - Node 1: set val to pop() -> `3`. Stack = `[1, 2]`.
   - Node 2: set val to pop() -> `2`. Stack = `[1]`.
   - Node 3: set val to pop() -> `1`. Stack = `[]`.
3. Output list: `3 -> 2 -> 1 -> nullptr`.

### Complexity Analysis
- **Time Complexity:** $O(N)$ because we traverse the list twice.
- **Space Complexity:** $O(N)$ to store the node values in the stack.

### Pros/Cons
- **Pros:** Extremely simple to implement; does not involve complex pointer redirection.
- **Cons:** High space complexity ($O(N)$); does not actually reverse the physical node connections (violates the constraint of actual list structure reversal).

### C++17 Brute Force Code
```cpp
#include <stack>

struct ListNode {
    int val;
    ListNode* next;
    ListNode(int x) : val(x), next(nullptr) {}
};

class SolutionBrute {
public:
    ListNode* reverseList(ListNode* head) {
        if (!head || !head->next) return head;
        
        std::stack<int> values;
        ListNode* curr = head;
        
        // Step 1: Store all values in a stack
        while (curr) {
            values.push(curr->val);
            curr = curr->next;
        }
        
        // Step 2: Write values back in reverse order
        curr = head;
        while (curr) {
            curr->val = values.top();
            values.pop();
            curr = curr->next;
        }
        
        return head;
    }
};
```


### 3. Optimal Solution & Complexities

### Approach 1: Iterative (In-place, Three-pointer)
We maintain three pointers: `prev` (initially `nullptr`), `curr` (initially `head`), and `nextNode` (temporary). We iterate through the list, redirecting pointers one by one.

#### Algorithm
1. Initialize `prev = nullptr` and `curr = head`.
2. While `curr` is not `nullptr`:
   - Store the next node: `nextNode = curr->next`.
   - Reverse the pointer: `curr->next = prev`.
   - Move `prev` forward: `prev = curr`.
   - Move `curr` forward: `curr = nextNode`.
3. Return `prev` (which now points to the new head of the reversed list).

#### Iterative C++17 Code
```cpp
class SolutionIterative {
public:
    ListNode* reverseList(ListNode* head) {
        ListNode* prev = nullptr;
        ListNode* curr = head;
        
        while (curr != nullptr) {
            ListNode* nextNode = curr->next; // 1. Save the next node
            curr->next = prev;              // 2. Reverse the link
            prev = curr;                    // 3. Move prev one step forward
            curr = nextNode;                // 4. Move curr one step forward
        }
        
        return prev; // prev will be pointing to the new head
    }
};
```

### Approach 2: Recursive
Recursively reverse the rest of the list and then fix the pointers at the current level.

#### Algorithm
1. Base cases: If `head` is null or there's only one node, return `head` (it's already reversed).
2. Recursively reverse the list starting from `head->next`. Let the head of this reversed sub-list be `newHead`.
3. Make the next node (`head->next`) point back to the current node: `head->next->next = head`.
4. Disconnect the current node's next pointer: `head->next = nullptr` (preventing cycles).
5. Return `newHead`.

#### Recursive C++17 Code
```cpp
class SolutionRecursive {
public:
    ListNode* reverseList(ListNode* head) {
        // Base case: empty list or single node
        if (head == nullptr || head->next == nullptr) {
            return head;
        }
        
        // Reverse the rest of the list
        ListNode* newHead = reverseList(head->next);
        
        // Link the next node's next back to current head
        head->next->next = head;
        
        // Disconnect the current head's outgoing link to avoid cycles
        head->next = nullptr;
        
        return newHead;
    }
};
```


#### Complexity Summary

| Approach | Time Complexity | Space Complexity | Reasoning |
| :--- | :--- | :--- | :--- |
| **Iterative** | $O(N)$ | $O(1)$ | Single pass over $N$ nodes. Only three pointers are used. |
| **Recursive** | $O(N)$ | $O(N)$ | Visits each node once. The recursion stack uses $O(N)$ frames. |


---

## Q02 Detect Loop

### 1. Problem Explanation

Given the `head` of a singly linked list, determine if the linked list contains a cycle (loop). A cycle exists if there is some node in the list that can be reached again by continuously following the `next` pointer.

### Input
- `head`: A pointer to the first node of a singly linked list.

### Output
- `true` if there is a cycle in the linked list, otherwise `false`.

### Constraints
- The number of nodes in the list is in the range `[0, 10^4]`.
- `-10^5 <= Node.val <= 10^5`

### Observations
- If there is no cycle, the traversal will eventually hit `nullptr`.
- If there is a cycle, the traversal will run infinitely in a loop, and we will never encounter `nullptr`.


### 2. Brute Solution

We track all visited node pointers in a hash set.

### Algorithm
1. Create a hash set of `ListNode*` to store visited node addresses.
2. Traverse the list node by node.
3. For each node, check if its address is already in the hash set.
   - If yes, a cycle exists. Return `true`.
   - If no, insert the address into the hash set and move to `curr->next`.
4. If the traversal reaches `nullptr`, return `false`.

### Dry Run
Input: `3 -> 2 -> 0 -> -4` (where `-4` points back to `2`)
1. `curr = 3`: Visited set = `{3}`.
2. `curr = 2`: Visited set = `{3, 2}`.
3. `curr = 0`: Visited set = `{3, 2, 0}`.
4. `curr = -4`: Visited set = `{3, 2, 0, -4}`.
5. `curr = 2`: Address `2` is already in the set. Return `true`.

### Complexity Analysis
- **Time Complexity:** $O(N)$ as we visit each node at most twice.
- **Space Complexity:** $O(N)$ to store the node addresses in the hash set.

### Pros/Cons
- **Pros:** Conceptually simple; works for finding the start of the loop immediately.
- **Cons:** High space complexity ($O(N)$), which is unacceptable for large lists.

### C++17 Brute Force Code
```cpp
#include <unordered_set>

struct ListNode {
    int val;
    ListNode* next;
    ListNode(int x) : val(x), next(nullptr) {}
};

class SolutionBrute {
public:
    bool hasCycle(ListNode* head) {
        std::unordered_set<ListNode*> visited;
        ListNode* curr = head;
        
        while (curr != nullptr) {
            if (visited.count(curr)) {
                return true; // Cycle detected
            }
            visited.insert(curr);
            curr = curr->next;
        }
        
        return false; // Reached nullptr, no cycle
    }
};
```


### 3. Optimal Solution & Complexities

### Algorithm
1. Initialize `slow = head` and `fast = head`.
2. Traverse the list using a loop. The loop condition is `while (fast != nullptr && fast->next != nullptr)`. This ensures `fast` can safely jump two steps ahead.
3. Move `slow` by 1 step: `slow = slow->next`.
4. Move `fast` by 2 steps: `fast = fast->next->next`.
5. If `slow == fast`, a cycle is detected. Return `true`.
6. If the loop terminates because `fast` reached the end, return `false`.

### Proof: Why they MUST meet
Let's mathematically prove why `slow` and `fast` pointers will always meet if a cycle exists:
1. Let the non-cyclic part of the list have length $X$, and the cyclic loop have length $C$.
2. When the `slow` pointer enters the loop (at the start of the cycle), it has traveled $X$ steps.
3. At this moment, the `fast` pointer (which moves at twice the speed) has traveled $2X$ steps.
4. This means `fast` is already inside the cycle. Let the distance (number of steps) from `slow` to `fast` inside the cycle (moving clockwise/forward) be $D$. The distance from `fast` to `slow` is $C - D$.
5. In each subsequent step:
   - `slow` moves 1 node forward.
   - `fast` moves 2 nodes forward.
   - The gap between `fast` and `slow` (with `fast` chasing `slow` from behind) decreases by exactly $2 - 1 = 1$ node per step.
6. Since the relative distance decreases by exactly 1 node per step, the gap will reduce to 0 in exactly $C - D$ steps.
7. Since $C - D < C$, they are guaranteed to meet before `slow` completes a single full lap of the cycle.

### Optimal C++17 Code
```cpp
class SolutionOptimal {
public:
    bool hasCycle(ListNode* head) {
        // Edge case: empty list or single node cannot have a loop
        if (!head || !head->next) {
            return false;
        }
        
        ListNode* slow = head;
        ListNode* fast = head;
        
        while (fast != nullptr && fast->next != nullptr) {
            slow = slow->next;          // Move slow by 1 step
            fast = fast->next->next;    // Move fast by 2 steps
            
            if (slow == fast) {
                return true;            // They met! Cycle detected.
            }
        }
        
        return false; // fast reached the end, no cycle
    }
};
```


#### Complexity Summary

| Approach | Time Complexity | Space Complexity | Reasoning |
| :--- | :--- | :--- | :--- |
| **Hash Set** | $O(N)$ | $O(N)$ | Visits each node once. Stores up to $N$ addresses in the set. |
| **Floyd's Algorithm**| $O(N)$ | $O(1)$ | Slower pointer enters loop after $X$ steps and meets fast within $C$ steps. Total steps $\leq X + C = N$. Uses only two pointer variables. |


---

## Q03 Find Loop Start

### 1. Problem Explanation

Given the `head` of a singly linked list, return the node where the cycle begins. If there is no cycle, return `nullptr`.

### Input
- `head`: A pointer to the first node of a singly linked list.

### Output
- A pointer to the node where the cycle starts, or `nullptr` if no cycle exists.

### Constraints
- The number of nodes in the list is in the range `[0, 10^4]`.
- `-10^5 <= Node.val <= 10^5`

### Observations
- A cycle means that traversing forward will eventually lead back to a node that has already been visited.
- The entry node to the cycle is the first node that is pointed to by two different nodes (the predecessor in the linear portion and the last node in the cycle).


### 2. Brute Solution

Store visited nodes in a hash set. The first duplicate pointer we visit is the start of the loop.

### Algorithm
1. Initialize a hash set of `ListNode*`.
2. Traverse the list starting from `head`.
3. For each node, check if it exists in the hash set.
   - If yes, this is the cycle entry node. Return it.
   - If no, add it to the set and move to `curr->next`.
4. If `curr` becomes `nullptr`, return `nullptr`.

### Dry Run
Input: `3 -> 2 -> 0 -> -4` (where `-4` links to `2`)
1. Visit `3`: visited = `{3}`.
2. Visit `2`: visited = `{3, 2}`.
3. Visit `0`: visited = `{3, 2, 0}`.
4. Visit `-4`: visited = `{3, 2, 0, -4}`.
5. Visit `2` (next of `-4`): `2` is already in the set.
6. Return Node `2`.

### Complexity Analysis
- **Time Complexity:** $O(N)$ since we visit each node at most twice.
- **Space Complexity:** $O(N)$ to store node pointers in the set.

### Pros/Cons
- **Pros:** Extremely simple to implement; handles the entry node directly.
- **Cons:** Space complexity scales linearly with list size.

### C++17 Brute Force Code
```cpp
#include <unordered_set>

struct ListNode {
    int val;
    ListNode* next;
    ListNode(int x) : val(x), next(nullptr) {}
};

class SolutionBrute {
public:
    ListNode* detectCycle(ListNode* head) {
        std::unordered_set<ListNode*> visited;
        ListNode* curr = head;
        
        while (curr != nullptr) {
            if (visited.count(curr)) {
                return curr; // First repeated node is the loop start
            }
            visited.insert(curr);
            curr = curr->next;
        }
        
        return nullptr; // No cycle
    }
};
```


### 3. Optimal Solution & Complexities

### Mathematical Proof
Let:
- $L_1$ = distance from `head` to the start of the cycle.
- $L_2$ = distance from the start of the cycle to the meeting point of `slow` and `fast`.
- $C$ = total length of the cycle.

```
          L1 (Linear)          L2 (Meeting)
head --------------------> Start --------> Meeting (slow & fast meet here)
                            ^                |
                            |________________|
                                C - L2 (Remaining Loop)
```

1. **Distance traveled by `slow` at meeting point:**
   $$d_{slow} = L_1 + L_2$$
   *(We assume `slow` hasn't completed a full lap, which is guaranteed as proven in loop detection).*

2. **Distance traveled by `fast` at meeting point:**
   $$d_{fast} = L_1 + L_2 + k \cdot C$$
   *(where $k \ge 1$ is the number of full laps `fast` has completed).*

3. **Since `fast` is twice as fast as `slow`:**
   $$d_{fast} = 2 \cdot d_{slow}$$
   $$L_1 + L_2 + k \cdot C = 2 \cdot (L_1 + L_2)$$
   $$L_1 + L_2 + k \cdot C = 2L_1 + 2L_2$$
   $$k \cdot C = L_1 + L_2$$
   $$L_1 = k \cdot C - L_2$$
   $$L_1 = (k - 1) \cdot C + (C - L_2)$$

4. **Interpretation:**
   - $L_1$ is the distance from `head` to the `Start` of the loop.
   - $(C - L_2)$ is the distance from the `Meeting` point to the `Start` of the loop.
   - This equation tells us that traveling a distance of $L_1$ from the `head` is exactly equivalent to traveling from the `Meeting` point for $(C - L_2)$ steps plus $(k-1)$ full cycles.
   - Therefore, if we place one pointer at `head` and another pointer at `Meeting`, and move both at a speed of 1 step per unit time, they will meet exactly at the `Start` of the cycle!

### Algorithm
1. Initialize `slow = head` and `fast = head`.
2. Move `slow` by 1 step, and `fast` by 2 steps.
3. If `fast` or `fast->next` becomes `nullptr`, return `nullptr` (no cycle).
4. If `slow == fast`, they have met. Break the loop.
5. Reset `slow = head`. Keep `fast` at the meeting point.
6. Loop while `slow != fast`:
   - Move `slow` by 1 step: `slow = slow->next`.
   - Move `fast` by 1 step: `fast = fast->next`.
7. Return `slow` (or `fast`), which now points to the start node of the cycle.

### Optimal C++17 Code
```cpp
class SolutionOptimal {
public:
    ListNode* detectCycle(ListNode* head) {
        if (!head || !head->next) {
            return nullptr;
        }
        
        ListNode* slow = head;
        ListNode* fast = head;
        bool hasCycle = false;
        
        // Step 1: Detect if a cycle exists
        while (fast != nullptr && fast->next != nullptr) {
            slow = slow->next;
            fast = fast->next->next;
            
            if (slow == fast) {
                hasCycle = true;
                break;
            }
        }
        
        // If no cycle, return nullptr
        if (!hasCycle) {
            return nullptr;
        }
        
        // Step 2: Find the start of the cycle
        slow = head; // Reset slow to head
        
        while (slow != fast) {
            slow = slow->next; // Move slow by 1 step
            fast = fast->next; // Move fast by 1 step (now at same speed)
        }
        
        return slow; // Both meet at the start of the loop
    }
};
```


#### Complexity Summary

| Approach | Time Complexity | Space Complexity | Reasoning |
| :--- | :--- | :--- | :--- |
| **Hash Set** | $O(N)$ | $O(N)$ | Single pass traversal. Hash set stores up to $N$ pointers. |
| **Floyd's Algorithm**| $O(N)$ | $O(1)$ | Phase 1 takes $\le N$ steps. Phase 2 takes $L_1 \le N$ steps. Total operations are linear. No auxiliary memory allocated. |


---

## Q04 Middle LL

### 1. Problem Explanation

Given the `head` of a singly linked list, find and return the middle node of the list.

- If there are two middle nodes (even length list), return the **second middle node** (Standard LeetCode definition).
- We will also cover the variant where we return the **first middle node**.

### Input
- `head`: A pointer to the first node of a singly linked list.

### Output
- A pointer to the middle node of the linked list.

### Constraints
- The number of nodes in the list is in the range `[1, 100]`.
- `1 <= Node.val <= 100`

### Observations
- In a list of length $N$:
  - If $N$ is odd (e.g., $5$ nodes: `1 -> 2 -> 3 -> 4 -> 5`), there is a single middle node at index $2$ (0-indexed): Node `3`.
  - If $N$ is even (e.g., $4$ nodes: `1 -> 2 -> 3 -> 4`), there are two middle nodes: `2` (index 1) and `3` (index 2).
    - Second Middle: Node `3`.
    - First Middle: Node `2`.


### 2. Brute Solution

Count the length, then traverse to the middle index.

### Algorithm
1. Initialize `count = 0` and a pointer `curr = head`.
2. Traverse the list to count the total nodes: `count++`.
3. Calculate the target index: `target = count / 2`.
4. Reset `curr = head`.
5. Loop `target` times: `curr = curr->next`.
6. Return `curr`.

### Dry Run
Input: `1 -> 2 -> 3 -> 4 -> nullptr` (Even length $N = 4$)
1. First pass: count = 4.
2. Target index: `4 / 2 = 2`.
3. Second pass starting at `1`:
   - Step 1: `curr` moves to `2`.
   - Step 2: `curr` moves to `3`.
4. Return Node `3` (Second middle).

### Complexity Analysis
- **Time Complexity:** $O(N)$ because we traverse the list 1.5 times ($N$ steps for count, $N/2$ steps to reach the middle).
- **Space Complexity:** $O(1)$ auxiliary space.

### Pros/Cons
- **Pros:** Conceptually simple and easy to debug.
- **Cons:** Requires two passes. We visit elements multiple times.

### C++17 Brute Force Code
```cpp
struct ListNode {
    int val;
    ListNode* next;
    ListNode(int x) : val(x), next(nullptr) {}
};

class SolutionBrute {
public:
    ListNode* middleNode(ListNode* head) {
        if (!head || !head->next) return head;
        
        int length = 0;
        ListNode* curr = head;
        while (curr) {
            length++;
            curr = curr->next;
        }
        
        int mid = length / 2;
        curr = head;
        for (int i = 0; i < mid; ++i) {
            curr = curr->next;
        }
        
        return curr;
    }
};
```


### 3. Optimal Solution & Complexities

### Case 1: Returning the Second Middle Node (Standard)
- Condition: `while (fast != nullptr && fast->next != nullptr)`
- For odd length `[1, 2, 3, 4, 5]`:
  - `slow` and `fast` start at `1`.
  - `slow` = `2`, `fast` = `3`.
  - `slow` = `3`, `fast` = `5`.
  - Loop terminates because `fast->next` is `nullptr`. `slow` is at `3` (middle).
- For even length `[1, 2, 3, 4]`:
  - `slow` and `fast` start at `1`.
  - `slow` = `2`, `fast` = `3`.
  - `slow` = `3`, `fast` = `nullptr`.
  - Loop terminates because `fast` is `nullptr`. `slow` is at `3` (second middle).

### Case 2: Returning the First Middle Node (Variant)
- Condition: `while (fast->next != nullptr && fast->next->next != nullptr)`
- For odd length `[1, 2, 3, 4, 5]`:
  - `slow` = `3` (same).
- For even length `[1, 2, 3, 4]`:
  - `slow` and `fast` start at `1`.
  - `slow` = `2`, `fast` = `3`.
  - Loop terminates because `fast->next->next` (which is `nullptr`) is checked. `slow` stays at `2` (first middle).

### Optimal C++17 Code
```cpp
class SolutionOptimal {
public:
    // Standard: Returns Second Middle Node for Even Length
    ListNode* middleNode(ListNode* head) {
        if (!head) return nullptr;
        
        ListNode* slow = head;
        ListNode* fast = head;
        
        while (fast != nullptr && fast->next != nullptr) {
            slow = slow->next;       // Moves 1 step
            fast = fast->next->next; // Moves 2 steps
        }
        
        return slow;
    }

    // Variant: Returns First Middle Node for Even Length
    ListNode* middleNodeFirst(ListNode* head) {
        if (!head || !head->next) return head;
        
        ListNode* slow = head;
        ListNode* fast = head;
        
        while (fast->next != nullptr && fast->next->next != nullptr) {
            slow = slow->next;
            fast = fast->next->next;
        }
        
        return slow;
    }
};
```


#### Complexity Summary

| Approach | Time Complexity | Space Complexity | Reasoning |
| :--- | :--- | :--- | :--- |
| **Length Counting (Brute)** | $O(N)$ | $O(1)$ | 1.5 passes over the list. No extra memory. |
| **Fast & Slow (Optimal)** | $O(N)$ | $O(1)$ | Exactly 1 pass. `fast` pointer moves $N$ steps, loop runs $N/2$ times. |


---

## Q05 Remove Nth From End

### 1. Problem Explanation

Given the `head` of a singly linked list and an integer `n`, remove the `n`-th node from the end of the list and return its head.

### Input
- `head`: A pointer to the first node of a singly linked list.
- `n`: An integer representing the position from the end of the list (1-indexed).

### Output
- A pointer to the head of the modified linked list.

### Constraints
- The number of nodes in the list is $L$ where $1 <= L <= 30$.
- `0 <= Node.val <= 100`
- `1 <= n <= L`

### Observations
- The $n$-th node from the end is the $(L - n + 1)$-th node from the beginning (where $L$ is the length of the list).
- To delete a node, we must position our pointer at the node *immediately before* the target node.


### 2. Brute Solution

Calculate length, then traverse to the target.

### Algorithm
1. Traverse the list to calculate the total length $L$.
2. If $n == L$, the target is the head. Return `head->next`.
3. Traverse the list to the $(L - n - 1)$-th node. Let this be `curr`.
4. Store the node to delete: `toDelete = curr->next`.
5. Update: `curr->next = curr->next->next`.
6. Delete `toDelete` and return `head`.

### Dry Run
Input: `1 -> 2 -> 3 -> 4 -> 5`, `n = 2`
1. First pass: $L = 5$.
2. Target index to stop before: $L - n - 1 = 5 - 2 - 1 = 2$ (Node `3` at 0-indexed position 2).
3. Second pass: Stop at Node `3`.
4. `toDelete` = Node `4`.
5. Update `3->next = 5`.
6. Delete Node `4`.
7. Output: `1 -> 2 -> 3 -> 5`.

### Complexity Analysis
- **Time Complexity:** $O(N)$ because we traverse the list twice (once for length, once to locate node).
- **Space Complexity:** $O(1)$ auxiliary space.

### C++17 Brute Force Code
```cpp
struct ListNode {
    int val;
    ListNode* next;
    ListNode(int x) : val(x), next(nullptr) {}
};

class SolutionBrute {
public:
    ListNode* removeNthFromEnd(ListNode* head, int n) {
        int length = 0;
        ListNode* curr = head;
        while (curr) {
            length++;
            curr = curr->next;
        }
        
        // Edge Case: Remove head node
        if (length == n) {
            ListNode* newHead = head->next;
            delete head;
            return newHead;
        }
        
        curr = head;
        // Move to the (length - n - 1)-th node
        for (int i = 0; i < length - n - 1; ++i) {
            curr = curr->next;
        }
        
        ListNode* toDelete = curr->next;
        curr->next = curr->next->next;
        delete toDelete;
        
        return head;
    }
};
```


### 3. Optimal Solution & Complexities

Using a dummy node simplifies the edge cases, particularly when removing the head node.

### Algorithm
1. Create a `dummy` node and set `dummy->next = head`.
2. Initialize two pointers: `slow = dummy` and `fast = dummy`.
3. Move `fast` pointer $n + 1$ steps forward. This creates a gap of exactly $n$ nodes between `slow` and `fast`.
4. Move both `slow` and `fast` pointers 1 step at a time until `fast` becomes `nullptr`.
5. At this point, `slow` is pointing to the node *before* the target node.
6. Store the target node to delete: `toDelete = slow->next`.
7. Bypass the target node: `slow->next = slow->next->next`.
8. Delete `toDelete` and return `dummy->next` (new head).

### Optimal C++17 Code
```cpp
class SolutionOptimal {
public:
    ListNode* removeNthFromEnd(ListNode* head, int n) {
        // Create dummy node to handle head removal seamlessly
        ListNode dummy(0);
        dummy.next = head;
        
        ListNode* slow = &dummy;
        ListNode* fast = &dummy;
        
        // Step 1: Move fast pointer n + 1 steps ahead
        for (int i = 0; i <= n; ++i) {
            fast = fast->next;
        }
        
        // Step 2: Move both pointers until fast reaches the end
        while (fast != nullptr) {
            slow = slow->next;
            fast = fast->next;
        }
        
        // Step 3: slow now points to the node BEFORE the target node
        ListNode* toDelete = slow->next;
        slow->next = slow->next->next;
        delete toDelete; // Clean up memory
        
        return dummy.next;
    }
};
```


#### Complexity Summary

| Approach | Time Complexity | Space Complexity | Reasoning |
| :--- | :--- | :--- | :--- |
| **Two-Pass (Brute)** | $O(N)$ | $O(1)$ | Traverses the list twice. |
| **One-Pass (Optimal)**| $O(N)$ | $O(1)$ | Single pass. `fast` traverses exactly $N+1$ steps. `slow` traverses $N-n$ steps. |


---

## Q06 Merge Two Sorted

### 1. Problem Explanation

Given the heads of two sorted linked lists, `list1` and `list2`, merge them into a single sorted linked list. The merged list should be made by splicing together the nodes of the first two lists. Return the head of the merged linked list.

### Input
- `list1`: A pointer to the head of the first sorted singly linked list.
- `list2`: A pointer to the head of the second sorted singly linked list.

### Output
- A pointer to the head of the merged sorted singly linked list.

### Constraints
- The number of nodes in both lists is in the range `[0, 50]`.
- `-100 <= Node.val <= 100`
- Both `list1` and `list2` are sorted in non-decreasing order.

### Observations
- Since the input lists are already sorted, we can build the merged list in a single pass by comparing the head nodes of both lists.
- We should splice the existing nodes in-place rather than allocating new nodes to save space.


### 2. Brute Solution

Collect all node values, sort them, and rebuild the list.

### Algorithm
1. Traverse `list1` and push all values into a vector.
2. Traverse `list2` and push all values into the same vector.
3. Sort the vector in ascending order.
4. Create a new dummy node. Traverse the sorted vector, creating a new node for each value and appending it to the list.
5. Return the dummy's next node.

### Complexity Analysis
- **Time Complexity:** $O((M+N) \log(M+N))$ where $M$ and $N$ are the lengths of `list1` and `list2` respectively, due to the sorting step.
- **Space Complexity:** $O(M+N)$ to store values in the vector and to allocate the new list nodes.

### Pros/Cons
- **Pros:** Conceptually simple; doesn't require modifying existing pointer links.
- **Cons:** High space complexity; does not modify the lists in-place; slow due to sorting.

### C++17 Brute Force Code
```cpp
#include <vector>
#include <algorithm>

struct ListNode {
    int val;
    ListNode* next;
    ListNode(int x) : val(x), next(nullptr) {}
};

class SolutionBrute {
public:
    ListNode* mergeTwoLists(ListNode* list1, ListNode* list2) {
        std::vector<int> values;
        
        // Step 1: Collect values
        ListNode* curr = list1;
        while (curr) {
            values.push_back(curr->val);
            curr = curr->next;
        }
        curr = list2;
        while (curr) {
            values.push_back(curr->val);
            curr = curr->next;
        }
        
        // Step 2: Sort
        std::sort(values.begin(), values.end());
        
        // Step 3: Rebuild
        ListNode dummy(0);
        curr = &dummy;
        for (int val : values) {
            curr->next = new ListNode(val);
            curr = curr->next;
        }
        
        return dummy.next;
    }
};
```


### 3. Optimal Solution & Complexities

### Approach 1: Iterative (In-place with Dummy Node)
We maintain a dummy node to build the list, and comparison pointers for `list1` and `list2`.

#### Algorithm
1. Create a `dummy` node. Initialize `tail = &dummy`.
2. While `list1 != nullptr` and `list2 != nullptr`:
   - Compare `list1->val` and `list2->val`.
   - If `list1->val <= list2->val`:
     - Link `tail->next = list1`.
     - Advance `list1 = list1->next`.
   - Else:
     - Link `tail->next = list2`.
     - Advance `list2 = list2->next`.
   - Move `tail` forward: `tail = tail->next`.
3. If one list is exhausted, append the other list: `tail->next = (list1 != nullptr) ? list1 : list2`.
4. Return `dummy.next`.

#### Iterative C++17 Code
```cpp
class SolutionIterative {
public:
    ListNode* mergeTwoLists(ListNode* list1, ListNode* list2) {
        ListNode dummy(0); // Stack-allocated dummy node
        ListNode* tail = &dummy;
        
        while (list1 != nullptr && list2 != nullptr) {
            if (list1->val <= list2->val) {
                tail->next = list1;
                list1 = list1->next;
            } else {
                tail->next = list2;
                list2 = list2->next;
            }
            tail = tail->next;
        }
        
        // Link the remaining nodes of the non-empty list
        tail->next = (list1 != nullptr) ? list1 : list2;
        
        return dummy.next;
    }
};
```

### Approach 2: Recursive (In-place)
Recursively merge the lists.

#### Algorithm
1. Base cases: If `list1` is null, return `list2`. If `list2` is null, return `list1`.
2. Compare head values:
   - If `list1->val <= list2->val`:
     - `list1->next = mergeTwoLists(list1->next, list2)`.
     - Return `list1`.
   - Else:
     - `list2->next = mergeTwoLists(list1, list2->next)`.
     - Return `list2`.

#### Recursive C++17 Code
```cpp
class SolutionRecursive {
public:
    ListNode* mergeTwoLists(ListNode* list1, ListNode* list2) {
        // Base Cases
        if (list1 == nullptr) return list2;
        if (list2 == nullptr) return list1;
        
        // Compare heads and merge recursively
        if (list1->val <= list2->val) {
            list1->next = mergeTwoLists(list1->next, list2);
            return list1;
        } else {
            list2->next = mergeTwoLists(list1, list2->next);
            return list2;
        }
    }
};
```


#### Complexity Summary

| Approach | Time Complexity | Space Complexity | Reasoning |
| :--- | :--- | :--- | :--- |
| **Iterative** | $O(M+N)$ | $O(1)$ | Single pass over both lists. Spliced in-place; uses only dummy and tail pointers. |
| **Recursive** | $O(M+N)$ | $O(M+N)$ | Visits each node once. The recursion call stack size is $O(M+N)$ frames. |


---

## Q07 Merge K Sorted

### 1. Problem Explanation

Given an array of `K` linked lists, where each linked list is sorted in ascending order, merge all the linked lists into one sorted linked list and return it.

### Input
- `lists`: A vector of head pointers to `K` sorted singly linked lists.

### Output
- A single head pointer to the merged sorted linked list containing all the nodes from the input lists.

### Constraints
- $K = \text{lists.length}$
- $0 \le K \le 10^4$
- $0 \le \text{Length of each list} \le 500$
- $-10^4 \le \text{Node.val} \le 10^4$
- The cumulative number of nodes in all lists does not exceed $5 \times 10^5$.
- Each individual list is sorted in strictly non-decreasing order.

### Observations
1. If the input array of lists is empty, we must return `nullptr`.
2. If there's only one list in the array, we can return it directly.
3. The nodes themselves should be rearranged; we should not allocate new nodes to replicate the values unless specified otherwise. We should rearrange the pointers in-place.
4. Since the individual lists are already sorted, we can leverage this order. We do not need to sort the nodes from scratch.


### 2. Brute Solution

### Algorithm
1. If `lists` is empty, return `nullptr`.
2. Initialize `mergedHead` as `lists[0]`.
3. Iterate from `i = 1` to `K - 1`:
   * Merge `mergedHead` with `lists[i]` using the standard two-pointer list merging algorithm.
   * Update `mergedHead` with the result.
4. Return `mergedHead`.

### Dry Run
Let `lists = [[1, 4], [2, 5], [3, 6]]`.
* Initialize `mergedHead = [1, 4]`.
* `i = 1`: Merge `[1, 4]` and `[2, 5]`.
  * Comparing heads: `1 < 2` $\rightarrow$ list starts with `1`.
  * Next compare `4` and `2` $\rightarrow$ next is `2`.
  * Next compare `4` and `5` $\rightarrow$ next is `4`.
  * Remainder is `5`.
  * Merged: `[1, 2, 4, 5]`. Update `mergedHead = [1, 2, 4, 5]`.
* `i = 2`: Merge `[1, 2, 4, 5]` and `[3, 6]`.
  * Comparing elements: `1`, then `2`, then `3`, then `4`, then `5`, then `6`.
  * Merged: `[1, 2, 3, 4, 5, 6]`. Update `mergedHead = [1, 2, 3, 4, 5, 6]`.
* End loop. Return `[1, 2, 3, 4, 5, 6]`.

### Complexity Analysis
* **Time Complexity**: $O(N \cdot K^2)$ where $N$ is the average length of a list and $K$ is the number of lists.
  * Specifically, the total number of operations is $\sum_{i=1}^{K-1} O(i \cdot N) = O(N \cdot \frac{K(K-1)}{2}) = O(N \cdot K^2)$.
* **Space Complexity**: $O(1)$ auxiliary space if the merge is done in-place.

### Pros and Cons
* **Pros**: Simple to implement; uses $O(1)$ extra space.
* **Cons**: Extremely slow when $K$ is large ($10^4$). TLE (Time Limit Exceeded) on typical online judges.

### Commented C++17 Code
```cpp
#include <vector>

// Definition for singly-linked list.
struct ListNode {
    int val;
    ListNode *next;
    ListNode() : val(0), next(nullptr) {}
    ListNode(int x) : val(x), next(nullptr) {}
    ListNode(int x, ListNode *next) : val(x), next(next) {}
};

class BruteMergeKSorted {
private:
    // Helper function to merge two sorted lists in-place
    ListNode* mergeTwoLists(ListNode* l1, ListNode* l2) {
        ListNode dummy(0);
        ListNode* tail = &dummy;

        while (l1 != nullptr && l2 != nullptr) {
            if (l1->val <= l2->val) {
                tail->next = l1;
                l1 = l1->next;
            } else {
                tail->next = l2;
                l2 = l2->next;
            }
            tail = tail->next;
        }

        // Attach the remaining non-empty list
        if (l1 != nullptr) {
            tail->next = l1;
        } else {
            tail->next = l2;
        }

        return dummy.next;
    }

public:
    ListNode* mergeKLists(std::vector<ListNode*>& lists) {
        if (lists.empty()) {
            return nullptr;
        }

        ListNode* mergedHead = lists[0];
        for (size_t i = 1; i < lists.size(); ++i) {
            mergedHead = mergeTwoLists(mergedHead, lists[i]);
        }

        return mergedHead;
    }
};
```


### 3. Optimal Solution & Complexities

### Best Approach
Divide and Conquer processes the array of lists by recursively or iteratively splitting them into halves, merging each half, and combining the results. It matches the $O(M \log K)$ time of the heap solution, but achieves $O(1)$ auxiliary space if done iteratively.

### Algorithm (Iterative Divide and Conquer)
1. If the list array is empty, return `nullptr`.
2. Let $K$ be the number of lists.
3. While $K > 1$:
   * Loop through the lists in pairs: merge list `i` and list `i + interval` and store the result in list `i`.
   * The `interval` starts at `1` and doubles at each step (`interval *= 2`).
   * Each pass reduces the number of active lists to merge by half.
4. Return `lists[0]`.

### Why it is Optimal
* **Time**: It achieves the lower bound of $O(M \log K)$ time since we perform $\log K$ merge phases, and in each phase, we process all $M$ nodes.
* **Space**: By doing this iteratively in-place, we eliminate the call-stack of recursion, leading to $O(1)$ auxiliary space. This is strictly better than the priority queue solution which requires $O(K)$ extra space.

### C++17 Code
```cpp
#include <vector>

class OptimalMergeKSorted {
private:
    // Helper function to merge two sorted lists in-place
    ListNode* mergeTwoLists(ListNode* l1, ListNode* l2) {
        ListNode dummy(0);
        ListNode* tail = &dummy;

        while (l1 != nullptr && l2 != nullptr) {
            if (l1->val <= l2->val) {
                tail->next = l1;
                l1 = l1->next;
            } else {
                tail->next = l2;
                l2 = l2->next;
            }
            tail = tail->next;
        }

        tail->next = (l1 != nullptr) ? l1 : l2;
        return dummy.next;
    }

public:
    ListNode* mergeKLists(std::vector<ListNode*>& lists) {
        if (lists.empty()) {
            return nullptr;
        }

        int size = lists.size();
        int interval = 1;

        // Perform merge pairwise iteratively
        while (interval < size) {
            for (int i = 0; i + interval < size; i += interval * 2) {
                lists[i] = mergeTwoLists(lists[i], lists[i + interval]);
            }
            interval *= 2;
        }

        return lists[0];
    }
};
```


#### Complexity Summary

| Approach | Time Complexity | Space Complexity | Recommendation |
|:---|:---:|:---:|:---|
| **Brute Force (Sequential)** | $O(N \cdot K^2)$ | $O(1)$ | Bad. Suffers from quadratic performance. |
| **Better (Min-Heap)** | $O(M \log K)$ | $O(K)$ | Good. Great for streaming data scenarios. |
| **Optimal (Divide & Conquer)** | $O(M \log K)$ | $O(1)$ | Best. Highly efficient time & space limits. |

* **Time Complexity**: Every node in the system ($M$ nodes total) is merged exactly $\log_2 K$ times. Since merging a node takes $O(1)$ operations per level, the time complexity is strictly $O(M \log K)$.
* **Space Complexity**: The iterative divide and conquer approach updates node pointers in-place and uses only a few pointers for iteration. Thus, the auxiliary space complexity is $O(1)$.


---

## Q08 Palindrome LL

### 1. Problem Explanation

Given the head of a singly linked list, determine if it is a palindrome. A linked list is a palindrome if it reads the same forward and backward.

### Input
- `head`: A head pointer to a singly linked list.

### Output
- A boolean: `true` if the list is a palindrome, otherwise `false`.

### Constraints
- The number of nodes in the list is in the range $[1, 10^5]$.
- $0 \le \text{Node.val} \le 9$.

### Observations
1. Palindromes read identical from left-to-right and right-to-left.
2. In a singly linked list, we can only traverse forward easily. We cannot traverse backward from the tail node because there are no backward pointers.
3. If the list is empty or has a single node, it is trivially a palindrome.


### 2. Brute Solution

### Algorithm
1. Traverse the linked list and push every node's value into a vector.
2. Initialize two pointers: `left = 0`, `right = vector.size() - 1`.
3. While `left < right`:
   * If `vector[left] != vector[right]`, return `false`.
   * Increment `left`, decrement `right`.
4. Return `true`.

### Dry Run
Let list be `1 -> 2 -> 3 -> 2 -> 1`.
* Traverse: Vector `v = [1, 2, 3, 2, 1]`.
* Two pointers: `left = 0`, `right = 4`.
  * `v[0] == v[4]` (1 == 1) $\rightarrow$ `left = 1`, `right = 3`.
  * `v[1] == v[3]` (2 == 2) $\rightarrow$ `left = 2`, `right = 2`.
* Loop terminates because `left < right` is false.
* Return `true`.

### Complexity Analysis
* **Time Complexity**: $O(N)$ where $N$ is the number of nodes. We traverse the list once and the array once.
* **Space Complexity**: $O(N)$ to store the node values in the vector.

### Pros and Cons
* **Pros**: Simple, highly readable, robust. No mutation of the original list.
* **Cons**: Uses linear $O(N)$ extra memory, which is inefficient for large datasets.

### Commented C++17 Code
```cpp
#include <vector>

// Definition for singly-linked list.
struct ListNode {
    int val;
    ListNode *next;
    ListNode() : val(0), next(nullptr) {}
    ListNode(int x) : val(x), next(nullptr) {}
    ListNode(int x, ListNode *next) : val(x), next(next) {}
};

class BrutePalindromeLL {
public:
    bool isPalindrome(ListNode* head) {
        std::vector<int> values;
        ListNode* current = head;
        
        // Copy list values to array
        while (current != nullptr) {
            values.push_back(current->val);
            current = current->next;
        }

        // Two pointer comparison
        int left = 0;
        int right = values.size() - 1;
        while (left < right) {
            if (values[left] != values[right]) {
                return false;
            }
            left++;
            right--;
        }
        return true;
    }
};
```


### 3. Optimal Solution & Complexities

### Best Approach
This method halves the space requirement to $O(1)$ by finding the midpoint of the linked list, reversing the second half in-place, performing a comparison, and then reversing it back to restore the original list structure.

### Algorithm
1. **Find Middle**: Initialize `slow` and `fast` pointers at the `head`. Move `slow` by 1 step and `fast` by 2 steps. When `fast->next` or `fast->next->next` is `nullptr`, `slow` points to the end of the first half.
2. **Reverse Second Half**: Reverse the list starting from `slow->next`. Let the head of this reversed second half be `secondHalfHead`.
3. **Compare**: Compare nodes from `head` and `secondHalfHead` one by one. If any values mismatch, set a flag `isPalin = false`.
4. **Restore**: Reverse the second half again to restore it, reconnecting it back to `slow->next`.
5. **Return**: Return `isPalin`.

### Why it is Optimal
It runs in $O(N)$ time with no extra memory allocation, fulfilling the $O(1)$ auxiliary space constraint. No recursive stack frames or temporary arrays are created.

### C++17 Code
```cpp
class OptimalPalindromeLL {
private:
    // Helper function to reverse a linked list in-place
    ListNode* reverseList(ListNode* head) {
        ListNode* prev = nullptr;
        ListNode* curr = head;
        while (curr != nullptr) {
            ListNode* nextNode = curr->next;
            curr->next = prev;
            prev = curr;
            curr = nextNode;
        }
        return prev;
    }

public:
    bool isPalindrome(ListNode* head) {
        if (head == nullptr || head->next == nullptr) {
            return true;
        }

        // 1. Find the end of the first half (slow pointer)
        ListNode* slow = head;
        ListNode* fast = head;
        while (fast->next != nullptr && fast->next->next != nullptr) {
            slow = slow->next;
            fast = fast->next->next;
        }

        // 2. Reverse the second half
        ListNode* secondHalfHead = reverseList(slow->next);

        // 3. Compare values
        ListNode* p1 = head;
        ListNode* p2 = secondHalfHead;
        bool isPalin = true;
        while (p2 != nullptr) {
            if (p1->val != p2->val) {
                isPalin = false;
                break;
            }
            p1 = p1->next;
            p2 = p2->next;
        }

        // 4. Restore the original list structure
        slow->next = reverseList(secondHalfHead);

        return isPalin;
    }
};
```


#### Complexity Summary

| Approach | Time Complexity | Space Complexity | Description |
|:---|:---:|:---:|:---|
| **Brute Force (Array)** | $O(N)$ | $O(N)$ | Simple but uses linear memory. |
| **Better (Recursive Stack)** | $O(N)$ | $O(N)$ | Avoids array but uses stack space. |
| **Optimal (In-place Reversal)** | $O(N)$ | $O(1)$ | Constant memory, restores original list. |

* **Time Complexity**: $O(N)$ since finding the middle takes $N/2$ steps, reversing takes $N/2$ steps, comparison takes $N/2$ steps, and restoration takes $N/2$ steps. The sum of these parts is $2N = O(N)$.
* **Space Complexity**: $O(1)$ because we only use a constant number of pointers (`slow`, `fast`, `prev`, `curr`, `nextNode`, `p1`, `p2`) and no extra memory.


---

## Q09 Delete Without Head

### 1. Problem Explanation

You are given a pointer to a node `node` in a singly linked list. You do not have access to the `head` of the list. You must delete the given node from the list.

### Input
- `node`: A pointer to the node to be deleted.

### Output
- No return value. The node must be deleted in-place by mutating the structure of the linked list.

### Constraints
- The list will contain at least two elements.
- The node to be deleted is guaranteed to be in the list and is **not** the last node of the list.
- node value is an integer.

### Observations
1. In a singly linked list, to delete a node $X$, we typically need to modify the `next` pointer of the node preceding $X$ ($X_{\text{prev}}$) to point to the node succeeding $X$ ($X_{\text{next}}$).
2. Since we do not have the head pointer, we cannot traverse the list from the beginning to find $X_{\text{prev}}$.
3. We are guaranteed that the node is not the last node. This means `node->next` is never `nullptr`.


### 2. Brute Solution

> [!NOTE]
> Since we do not have access to the head pointer, there is no way to traverse the list from the beginning to find the predecessor node. Thus, a traditional traversal-based brute force solution is impossible. The brute force jumps directly to the optimal trick-based solution.


### 3. Optimal Solution & Complexities

### Best Approach
Copy the data from the next node into the current node, then delete the next node by setting the current node's `next` pointer to point to the next node's `next` pointer.

### Algorithm
1. Store a temporary pointer `temp` pointing to the next node: `ListNode* temp = node->next;`.
2. Copy the value of the next node to the current node: `node->val = temp->val;`.
3. Link the current node to the node after the next node: `node->next = temp->next;`.
4. Free/delete `temp` to prevent memory leaks: `delete temp;`.

### Why it is Optimal
* **Time**: It requires exactly $O(1)$ operations because we only look at the node and its immediate successor.
* **Space**: $O(1)$ auxiliary space since we only use one temporary pointer variable.

### C++17 Code
```cpp
// Definition for singly-linked list.
struct ListNode {
    int val;
    ListNode *next;
    ListNode(int x) : val(x), next(nullptr) {}
};

class OptimalDeleteNode {
public:
    void deleteNode(ListNode* node) {
        // Since we are guaranteed the node is not the last node,
        // node->next is never nullptr.
        ListNode* temp = node->next;
        
        // Copy the value of the next node into the current node
        node->val = temp->val;
        
        // Skip the next node
        node->next = temp->next;
        
        // Free the memory of the skipped node
        delete temp;
    }
};
```


#### Complexity Summary

* **Time Complexity**: $O(1)$. We perform a constant number of operations: one value assignment, one pointer assignment, and one memory deallocation.
* **Space Complexity**: $O(1)$ auxiliary space as we only use a single temporary pointer `temp`.


---

## Q10 Flatten Multilevel

### 1. Problem Explanation

Given a linked list where in addition to the `next` pointer, each node has a `child` pointer, which may point to a separate singly linked list. These child lists may also have one or more children of their own, and so on, creating a multilevel data structure.

Flatten the list so that all the nodes appear in a single-level singly linked list. The flattening should follow a **Depth-First Search (DFS)** order: if a node has a child, the entire child list must be flattened and inserted after the current node before we continue with the next node in the original list.

### Input
- `head`: A head pointer to the multilevel linked list.

### Output
- A head pointer to the flattened single-level linked list. All `child` pointers must be set to `nullptr`.

### Constraints
- The number of nodes in the list is in the range $[0, 1000]$.
- $1 \le \text{Node.val} \le 10^5$.

### Observations
1. This is essentially a tree structure where `child` is like a left child and `next` is like a right child.
2. The traversal order is preorder DFS: visit current node, then recurse on child (left), then recurse on next (right).
3. We need to reset the `child` pointers of all nodes to `nullptr` in the final output.


### 2. Brute Solution

### Algorithm
1. Create a helper function `dfs(Node* node, vector<Node*>& nodes)`:
   * If `node` is null, return.
   * Push `node` to `nodes`.
   * Recurse on `node->child`.
   * Recurse on `node->next`.
2. In the main function, call `dfs(head, nodes)`.
3. Loop from `i = 0` to `nodes.size() - 2`:
   * Link `nodes[i]->next = nodes[i+1]`.
   * Set `nodes[i]->child = nullptr`.
4. Set the `next` and `child` of the last node to `nullptr`.
5. Return `head`.

### Complexity Analysis
* **Time Complexity**: $O(N)$ where $N$ is the total number of nodes in the multilevel list.
* **Space Complexity**: $O(N)$ for the vector and the recursion stack.

### Commented C++17 Code
```cpp
#include <vector>

// Definition for a Node.
struct Node {
    int val;
    Node* next;
    Node* child;
    Node(int x) : val(x), next(nullptr), child(nullptr) {}
};

class BruteFlattenMultilevel {
private:
    void dfs(Node* node, std::vector<Node*>& nodes) {
        if (node == nullptr) return;
        nodes.push_back(node);
        dfs(node->child, nodes);
        dfs(node->next, nodes);
    }

public:
    Node* flatten(Node* head) {
        if (head == nullptr) return nullptr;
        
        std::vector<Node*> nodes;
        dfs(head, nodes);
        
        // Relink all nodes sequentially
        for (size_t i = 0; i < nodes.size() - 1; ++i) {
            nodes[i]->next = nodes[i+1];
            nodes[i]->child = nullptr;
        }
        nodes.back()->next = nullptr;
        nodes.back()->child = nullptr;
        
        return head;
    }
};
```


### 3. Optimal Solution & Complexities

### Best Approach
Iterate through the list. Whenever a node has a child, find the tail of that child list, link the tail to the current node's next node, make the child node the current node's next node, and set the child pointer to null. Continue traversing forward.

### Algorithm
1. Initialize `curr` to `head`.
2. While `curr` is not null:
   * If `curr` has a `child`:
     * Find the tail of the child list: `Node* temp = curr->child;` and loop `while (temp->next) temp = temp->next;`.
     * Link the child tail's `next` to `curr->next`: `temp->next = curr->next;`.
     * Link `curr->next` to `curr->child`: `curr->next = curr->child;`.
     * Clear the child pointer: `curr->child = nullptr;`.
   * Move `curr` to `curr->next`.
3. Return `head`.

### Why it is Optimal
* **Time**: $O(N)$. Although we have nested loops, each node is visited at most twice (once during main traversal, and at most once when locating the tail of a child list).
* **Space**: $O(1)$ auxiliary space since we do not use recursion stack or stack data structures.

### C++17 Code
```cpp
class OptimalFlattenMultilevel {
public:
    Node* flatten(Node* head) {
        if (head == nullptr) return nullptr;

        Node* curr = head;
        while (curr != nullptr) {
            // If the current node has a child list
            if (curr->child != nullptr) {
                // Find the tail of the child list
                Node* childTail = curr->child;
                while (childTail->next != nullptr) {
                    childTail = childTail->next;
                }

                // Connect the tail of the child list to the next of current
                childTail->next = curr->next;

                // Make the child the next of current
                curr->next = curr->child;

                // Clear the child pointer
                curr->child = nullptr;
            }
            // Move to the next node
            curr = curr->next;
        }
        return head;
    }
};
```


#### Complexity Summary

| Approach | Time Complexity | Space Complexity | Description |
|:---|:---:|:---:|:---|
| **Brute Force** | $O(N)$ | $O(N)$ | Simple but uses linear memory. |
| **Better (Stack)** | $O(N)$ | $O(D)$ | Standard DFS, uses stack proportional to depth. |
| **Optimal (In-place)** | $O(N)$ | $O(1)$ | No extra memory, highly efficient. |

* **Time Complexity**: $O(N)$ where $N$ is the number of nodes. Even though we find the tail of child lists using a nested loop, we traverse each node link a constant number of times. Specifically, no node's `next` link is traversed more than twice.
* **Space Complexity**: $O(1)$ auxiliary space as we only redirect pointers.


---

## Q11 Add Two Numbers

### 1. Problem Explanation

You are given two non-empty linked lists representing two non-negative integers. The digits are stored in **reverse order**, and each of their nodes contains a single digit. Add the two numbers and return the sum as a linked list.

You may assume the two numbers do not contain any leading zero, except the number 0 itself.

### Input
- `l1`: Head pointer to the first singly linked list representing number 1.
- `l2`: Head pointer to the second singly linked list representing number 2.

### Output
- Head pointer to a new singly linked list representing the sum of the two numbers.

### Constraints
- The number of nodes in each linked list is in the range $[1, 100]$.
- $0 \le \text{Node.val} \le 9$.

### Observations
1. The digits are stored in reverse order. This is a major advantage! Why? Because standard addition starts from the least significant digit (ones place) and moves to the most significant digit. Since the heads of the lists represent the ones place, we can traverse both lists from left to right and perform addition naturally.
2. We must handle the carry digit that propagates from right to left.
3. The lists can be of different lengths.
4. There might be a carry left over after processing all nodes from both lists (e.g., $99 + 1 = 100$). We must create a new node for this final carry.


### 2. Brute Solution

### Algorithm
1. Traverse `l1`, compute the integer `num1` (reversing digits back).
2. Traverse `l2`, compute the integer `num2`.
3. Let `sum = num1 + num2`.
4. If `sum == 0`, return a node with value `0`.
5. Reconstruct the list by taking `sum % 10` as node values and setting `sum = sum / 10`.

### Complexity Analysis
* **Time Complexity**: $O(M + N)$ where $M, N$ are the lengths of the lists.
* **Space Complexity**: $O(1)$ auxiliary space.
* **Pros**: Simple arithmetic.
* **Cons**: Fails completely on overflow when list length $> 19$.

### Commented C++17 Code
```cpp
// Definition for singly-linked list.
struct ListNode {
    int val;
    ListNode *next;
    ListNode() : val(0), next(nullptr) {}
    ListNode(int x) : val(x), next(nullptr) {}
    ListNode(int x, ListNode *next) : val(x), next(next) {}
};

class BruteAddTwoNumbers {
public:
    ListNode* addTwoNumbers(ListNode* l1, ListNode* l2) {
        long long num1 = 0, num2 = 0;
        long long factor = 1;
        
        // Convert l1 to integer
        while (l1 != nullptr) {
            num1 += l1->val * factor;
            factor *= 10;
            l1 = l1->next;
        }
        
        // Convert l2 to integer
        factor = 1;
        while (l2 != nullptr) {
            num2 += l2->val * factor;
            factor *= 10;
            l2 = l2->next;
        }
        
        long long sum = num1 + num2;
        if (sum == 0) return new ListNode(0);
        
        ListNode dummy(0);
        ListNode* curr = &dummy;
        
        // Build the linked list
        while (sum > 0) {
            curr->next = new ListNode(sum % 10);
            curr = curr->next;
            sum /= 10;
        }
        
        return dummy.next;
    }
};
```


### 3. Optimal Solution & Complexities

### Best Approach
Instead of transferring values to intermediate strings or vectors, perform the addition in a single pass directly on the linked list nodes. We sum the matching digits and propagate the carry variable forward.

### Algorithm
1. Initialize a `dummy` node to simplify the creation of the output list.
2. Initialize `curr` to point to `dummy`.
3. Set `carry` to `0`.
4. Run a loop while `l1 != nullptr` or `l2 != nullptr` or `carry != 0`:
   * Determine digit values: `x = (l1 != nullptr) ? l1->val : 0;` and `y = (l2 != nullptr) ? l2->val : 0;`.
   * Compute sum: `sum = x + y + carry;`.
   * Update carry: `carry = sum / 10;`.
   * Create a new node with `sum % 10` and link it: `curr->next = new ListNode(sum % 10);`.
   * Advance `curr`.
   * Advance `l1` and `l2` pointers if they are not null.
5. Return `dummy.next`.

### Why it is Optimal
* **Time**: $O(\max(M, N))$ since we traverse the longer list exactly once.
* **Space**: $O(1)$ auxiliary space (excluding the output list) because we only keep a few pointers and a single `carry` variable.

### C++17 Code
```cpp
class OptimalAddTwoNumbers {
public:
    ListNode* addTwoNumbers(ListNode* l1, ListNode* l2) {
        ListNode dummy(0); // Dummy node to simplify head initialization
        ListNode* curr = &dummy;
        int carry = 0;

        // Loop until lists are exhausted AND there is no remaining carry
        while (l1 != nullptr || l2 != nullptr || carry != 0) {
            int sum = carry;

            if (l1 != nullptr) {
                sum += l1->val;
                l1 = l1->next; // Move to next node
            }
            if (l2 != nullptr) {
                sum += l2->val;
                l2 = l2->next; // Move to next node
            }

            carry = sum / 10;                     // Compute new carry
            curr->next = new ListNode(sum % 10);  // Store digit
            curr = curr->next;
        }

        return dummy.next;
    }
};
```


#### Complexity Summary

| Approach | Time Complexity | Space Complexity | Description |
|:---|:---:|:---:|:---|
| **Brute Force** | $O(M + N)$ | $O(1)$ | Fails due to arithmetic overflow. |
| **Better (Strings)** | $O(M + N)$ | $O(M + N)$ | Works for all, but duplicates digits in memory. |
| **Optimal (Direct)** | $O(\max(M, N))$ | $O(1)$ auxiliary | Best. Minimal memory overhead. |

* **Time Complexity**: $O(\max(M, N))$ because we perform a single loop that runs for a number of iterations equal to the length of the longer list, plus at most 1 additional iteration for a trailing carry.
* **Space Complexity**: $O(1)$ auxiliary space since we do not allocate any temporary structures (vectors/strings). The $O(\max(M, N))$ space to build the result list is required by the problem statement.


---

## Q12 Intersection Point

### 1. Problem Explanation

Given the heads of two singly linked lists `headA` and `headB`, return the node at which the two lists intersect. If the two linked lists have no intersection at all, return `nullptr`.

Note that the linked lists must retain their original structure after the function returns.

### Input
- `headA`: Head pointer to the first singly linked list.
- `headB`: Head pointer to the second singly linked list.

### Output
- Pointer to the intersection node, or `nullptr` if there is no intersection.

### Constraints
- The number of nodes of `listA` is $M$.
- The number of nodes of `listB` is $N$.
- $1 \le M, N \le 3 \times 10^4$.
- $1 \le \text{Node.val} \le 10^5$.
- The intersection node must exist by reference, meaning the same memory node is shared. (It is not enough for values to be equal; the actual nodes must be identical).

### Observations
1. Once two singly linked lists intersect, they share all subsequent nodes because a node in a singly linked list only has one `next` pointer. This results in a "Y-shaped" structure, not an "X-shaped" one.
2. The lengths of `listA` and `listB` can be different.
3. If they intersect, the last node of both lists must be the same node.


### 2. Brute Solution

### Algorithm
1. Outer loop: Initialize `currA = headA`. While `currA != nullptr`:
   * Inner loop: Initialize `currB = headB`. While `currB != nullptr`:
     * If `currA == currB`, return `currA`.
     * Move `currB = currB->next`.
   * Move `currA = currA->next`.
2. Return `nullptr` if no match is found.

### Complexity Analysis
* **Time Complexity**: $O(M \times N)$ where $M, N$ are lengths of the two lists.
* **Space Complexity**: $O(1)$ auxiliary space.

### Commented C++17 Code
```cpp
// Definition for singly-linked list.
struct ListNode {
    int val;
    ListNode *next;
    ListNode(int x) : val(x), next(nullptr) {}
};

class BruteIntersection {
public:
    ListNode *getIntersectionNode(ListNode *headA, ListNode *headB) {
        ListNode* currA = headA;
        while (currA != nullptr) {
            ListNode* currB = headB;
            while (currB != nullptr) {
                // Check if they are the exact same node (by address)
                if (currA == currB) {
                    return currA;
                }
                currB = currB->next;
            }
            currA = currA->next;
        }
        return nullptr;
    }
};
```


### 3. Optimal Solution & Complexities

### Best Approach
Traverse both lists simultaneously with two pointers. When a pointer reaches the end of a list, redirect it to the head of the other list. They will meet at the intersection node or `nullptr`.

### Algorithm
1. If `headA == nullptr` or `headB == nullptr`, return `nullptr`.
2. Initialize `pA = headA` and `pB = headB`.
3. While `pA != pB`:
   * Move `pA`: if `pA` is `nullptr`, set `pA = headB`; otherwise, `pA = pA->next`.
   * Move `pB`: if `pB` is `nullptr`, set `pB = headA`; otherwise, `pB = pB->next`.
4. Return `pA` (which is either the intersection node or `nullptr`).

### Why it is Optimal
* **Time**: $O(M + N)$ since each pointer traverses at most $M + N$ steps.
* **Space**: $O(1)$ auxiliary space as we only use two pointer variables, without allocating any extra structures.

### C++17 Code
```cpp
class OptimalIntersection {
public:
    ListNode *getIntersectionNode(ListNode *headA, ListNode *headB) {
        if (headA == nullptr || headB == nullptr) {
            return nullptr;
        }

        ListNode* pA = headA;
        ListNode* pB = headB;

        // Traverse together. They will either meet at the intersection point,
        // or both reach nullptr (meaning no intersection).
        while (pA != pB) {
            // Redirect to the head of the other list upon reaching the end
            pA = (pA == nullptr) ? headB : pA->next;
            pB = (pB == nullptr) ? headA : pB->next;
        }

        return pA;
    }
};
```


#### Complexity Summary

| Approach | Time Complexity | Space Complexity | Description |
|:---|:---:|:---:|:---|
| **Brute Force** | $O(M \times N)$ | $O(1)$ | Double loop, slow. |
| **Better (Hash Set)** | $O(M + N)$ | $O(M)$ | Fast, but uses extra linear memory. |
| **Optimal (Two Pointers)** | $O(M + N)$ | $O(1)$ | Perfect time and space complexity. |

* **Time Complexity**: $O(M + N)$. In the worst case, each pointer travels at most $M + N$ steps (traversing one list and then the other).
* **Space Complexity**: $O(1)$ auxiliary space since we only use two pointers.


---

## Q13 Reverse K Groups

### 1. Problem Explanation

Given the `head` of a singly linked list and an integer `k`, reverse the nodes of the list `k` at a time and return the modified list.

`k` is a positive integer and is less than or equal to the length of the linked list. If the number of nodes is not a multiple of `k` then left-out nodes, in the end, should remain as they are (i.e., not reversed).

You may not alter the values in the list's nodes; only nodes themselves may be changed.

### Input
- `head`: A pointer to the head of a singly linked list.
- `k`: An integer representing the group size.

### Output
- A pointer to the head of the modified linked list.

### Constraints
- The number of nodes in the list is `n`.
- $1 \le k \le n \le 5000$
- $0 \le \text{Node.val} \le 1000$

### Observations
1. If $k = 1$, the list remains unchanged.
2. If $n < k$, no nodes are reversed; the list is returned as-is.
3. Only full groups of size `k` are reversed. Any remaining nodes at the end that form a group of size $< k$ must remain in their original order.
4. The relative connections between the reversed groups must be maintained correctly.


### 2. Brute Solution

### Algorithm
The simplest approach is to use a recursive formulation:
1. Count if there are at least `k` nodes left starting from the current node `head`.
2. If there are fewer than `k` nodes, return `head` as-is.
3. Otherwise, reverse the first `k` nodes using the standard iterative reversal technique.
4. After reversing, the original `head` is now the tail of this segment.
5. Recursively call `reverseKGroup` on the $(k+1)$-th node (which is the node immediately following the original segment).
6. Connect the original `head->next` to the result of the recursive call.
7. Return the new head of this reversed segment (which was the $k$-th node before reversal).

### Dry Run
List: `1 -> 2 -> 3 -> 4 -> 5`, `k = 2`
1. Call on node `1`. Count $2$ nodes: `1`, `2`. Eligible.
2. Reverse `1 -> 2` to get `2 -> 1`.
3. Node `1` is the new tail.
4. Recursively call on node `3`.
5. Call on node `3`. Count $2$ nodes: `3`, `4`. Eligible.
6. Reverse `3 -> 4` to get `4 -> 3`.
7. Node `3` is the new tail.
8. Recursively call on node `5`.
9. Call on node `5`. Count: only 1 node remaining. Less than `k=2`. Return `5`.
10. Unwinding recursion:
    - Node `3->next = 5`. Segment becomes `4 -> 3 -> 5`. Return `4`.
    - Node `1->next = 4`. Segment becomes `2 -> 1 -> 4 -> 3 -> 5`. Return `2`.

### Complexity Analysis
- **Time Complexity:** $O(N)$ because we traverse each node at most twice (once for counting, once for reversing).
- **Space Complexity:** $O(N/k)$ recursion stack space, where $N$ is the total number of nodes.

### Pros and Cons
- **Pros:** Extremely clean code, simple to explain and implement.
- **Cons:** Uses stack space proportional to $N/k$, which is not $O(1)$ auxiliary space.

### C++17 Code
```cpp
#include <iostream>

struct ListNode {
    int val;
    ListNode* next;
    ListNode() : val(0), next(nullptr) {}
    ListNode(int x) : val(x), next(nullptr) {}
    ListNode(int x, ListNode* next) : val(x), next(next) {}
};

class SolutionRecursive {
public:
    ListNode* reverseKGroup(ListNode* head, int k) {
        // Base case: check if we have at least k nodes left
        ListNode* curr = head;
        int count = 0;
        while (curr != nullptr && count < k) {
            curr = curr->next;
            count++;
        }
        
        // If we have fewer than k nodes, return head as-is
        if (count < k) {
            return head;
        }
        
        // Reverse first k nodes
        ListNode* prev = nullptr;
        curr = head;
        for (int i = 0; i < k; ++i) {
            ListNode* next_node = curr->next;
            curr->next = prev;
            prev = curr;
            curr = next_node;
        }
        
        // At this point, prev is the new head of the reversed segment,
        // and head is the tail of the reversed segment.
        // Link the tail to the result of the next recursive call.
        head->next = reverseKGroup(curr, k);
        
        return prev;
    }
};
```


### 3. Optimal Solution & Complexities

### Algorithm
To reverse the list in groups of `k` iteratively with $O(1)$ extra space, we need to carefully track the connections between the boundaries of consecutive groups.
1. Create a `dummy` node and set `dummy->next = head`.
2. Maintain `group_prev` pointing to the node right before the current group being processed. Initially, `group_prev = dummy`.
3. In a loop:
   - Find the $k$-th node from `group_prev`. Let this be `group_end`. If there are fewer than $k$ nodes left, we break and stop.
   - Keep a reference to the node after the group: `next_group_start = group_end->next`.
   - Reverse the current group of $k$ nodes. To do this, we can detach the group temporarily (set `group_end->next = nullptr`) or reverse it in place. Let's reverse it in place:
     - Set `prev = next_group_start` (so the tail of the reversed group points directly to the next group).
     - Set `curr = group_prev->next` (start of the current group).
     - Standard loop to reverse: `next_node = curr->next`, `curr->next = prev`, `prev = curr`, `curr = next_node`. Run this $k$ times.
   - Connect the preceding part: `group_prev->next` must now point to the new head of the reversed group (which is `group_end`).
   - Move `group_prev` to the tail of the current reversed group. The tail of the current reversed group is the node that was the head before reversal. We can find this by holding a reference to `group_prev->next` before changing it, or since we know it, we save `temp = group_prev->next` before pointer updates, then set `group_prev = temp`.

### Why Optimal?
- Reuses existing nodes without any extra heap allocation.
- Iterative execution ensures $O(1)$ stack space.

### C++17 Code
```cpp
#include <iostream>

class Solution {
public:
    ListNode* reverseKGroup(ListNode* head, int k) {
        if (head == nullptr || k == 1) {
            return head;
        }
        
        // Dummy node to handle head updates easily
        ListNode* dummy = new ListNode(0);
        dummy->next = head;
        
        ListNode* group_prev = dummy;
        
        while (true) {
            // Find the k-th node
            ListNode* group_end = group_prev;
            int count = 0;
            while (group_end != nullptr && count < k) {
                group_end = group_end->next;
                count++;
            }
            
            // If fewer than k nodes left, we are done
            if (group_end == nullptr) {
                break;
            }
            
            // Node right after the current group
            ListNode* next_group_start = group_end->next;
            
            // Reverse the group of size k.
            // We initialize 'prev' to 'next_group_start' so the original head 
            // of the group will automatically link to the next group after reversal.
            ListNode* prev = next_group_start;
            ListNode* curr = group_prev->next;
            ListNode* group_start = curr; // Save reference to the start of this group
            
            for (int i = 0; i < k; ++i) {
                ListNode* next_node = curr->next;
                curr->next = prev;
                prev = curr;
                curr = next_node;
            }
            
            // Connect group_prev to the new head of this reversed group (which is 'prev')
            group_prev->next = prev;
            
            // Update group_prev to be the end of the newly reversed group (which is 'group_start')
            group_prev = group_start;
        }
        
        ListNode* new_head = dummy->next;
        delete dummy; // Clean up memory
        return new_head;
    }
};
```


#### Complexity Summary

| Solution | Time Complexity | Space Complexity | Details |
| :--- | :--- | :--- | :--- |
| **Recursive** | $O(N)$ | $O(N/k)$ | One traversal for check + one traversal for reverse. Recursion stack size $\approx N/k$. |
| **Iterative (Optimal)** | $O(N)$ | $O(1)$ | Same traversal behavior, but pointer manipulations are done inline without stack allocations. |


---

## Q14 Sort LL MergeSort

### 1. Problem Explanation

Given the `head` of a singly linked list, return the list after sorting it in ascending order. The solution must run in $O(N \log N)$ time and use $O(\log N)$ stack space (or $O(1)$ auxiliary space if done iteratively).

### Input
- `head`: A pointer to the head of a singly linked list.

### Output
- A pointer to the head of the sorted singly linked list.

### Constraints
- The number of nodes in the list is in the range $[0, 5 \cdot 10^4]$.
- $-10^5 \le \text{Node.val} \le 10^5$

### Observations
1. Unlike arrays, we cannot access elements of a linked list in $O(1)$ time by index.
2. Standard sorting algorithms like Quick Sort or Heap Sort rely heavily on random access, making them less suitable or harder to optimize for linked lists.
3. Merge Sort fits linked lists perfectly because merging two sorted lists can be done in-place by simply adjusting pointers without allocating new nodes.


### 2. Brute Solution

### Algorithm
1. Traverse the linked list and copy all node values into a dynamic array (like `std::vector<int>`).
2. Sort the array using the built-in library sorting function (`std::sort`), which uses IntroSort ($O(N \log N)$ time).
3. Traverse the list again, overwriting the values of the nodes with the sorted values from the array.
4. Return the head of the list.

### Dry Run
List: `4 -> 2 -> 1 -> 3`
1. Array: `[4, 2, 1, 3]`
2. Sort Array: `[1, 2, 3, 4]`
3. Rewrite values:
   - Node 1 value becomes `1`.
   - Node 2 value becomes `2`.
   - Node 3 value becomes `3`.
   - Node 4 value becomes `4`.
4. Output: `1 -> 2 -> 3 -> 4`

### Complexity Analysis
- **Time Complexity:** $O(N \log N)$ to sort the array of size $N$, plus $O(N)$ for copying.
- **Space Complexity:** $O(N)$ auxiliary space to store the node values in the array.

### Pros and Cons
- **Pros:** Extremely easy to implement.
- **Cons:** High space complexity ($O(N)$) and does not actually sort the nodes by modifying pointers (violates the spirit of linked list manipulation).

### C++17 Code
```cpp
#include <vector>
#include <algorithm>

struct ListNode {
    int val;
    ListNode* next;
    ListNode() : val(0), next(nullptr) {}
    ListNode(int x) : val(x), next(nullptr) {}
    ListNode(int x, ListNode* next) : val(x), next(next) {}
};

class SolutionBrute {
public:
    ListNode* sortList(ListNode* head) {
        if (head == nullptr || head->next == nullptr) {
            return head;
        }
        
        std::vector<int> values;
        ListNode* curr = head;
        while (curr != nullptr) {
            values.push_back(curr->val);
            curr = curr->next;
        }
        
        std::sort(values.begin(), values.end());
        
        curr = head;
        for (int val : values) {
            curr->val = val;
            curr = curr->next;
        }
        
        return head;
    }
};
```


### 3. Optimal Solution & Complexities

### Why is Merge Sort preferred over Quick Sort for Linked Lists?
1. **No Random Access:** Quick Sort relies heavily on random access ($O(1)$ index access) to partition the array and select pivots. In linked lists, traversing to a specific index takes $O(N)$ time, which degrades Quick Sort's performance.
2. **$O(1)$ Merge Overhead:** Merging two sorted arrays requires an auxiliary array of size $O(N)$ to copy elements. However, merging two sorted linked lists can be done by simply changing pointers, requiring $O(1)$ auxiliary space.
3. **Worst-Case Guarantee:** Merge Sort guarantees $O(N \log N)$ time complexity in all cases. Quick Sort can degrade to $O(N^2)$ time if pivot selection is poor (e.g., already sorted lists).

### Algorithm
1. **Base Case:** If the list is empty or has only one node, return `head`.
2. **Find Middle:** Use the slow and fast pointer approach to find the midpoint of the list.
   - To split correctly, we want the slow pointer to stop at the node *before* the middle or at the middle.
   - Specifically: `slow` starts at `head`, `fast` starts at `head->next` (or we track `prev` of `slow`).
   - Move `slow` by 1 step and `fast` by 2 steps.
3. **Split List:**
   - The middle node is `slow`.
   - The head of the right sublist is `right_head = slow->next`.
   - Disconnect the lists: `slow->next = nullptr`.
4. **Sort Recursively:**
   - Sort the left half: `left = sortList(head)`.
   - Sort the right half: `right = sortList(right_head)`.
5. **Merge:** Merge the two sorted lists `left` and `right` and return the merged list's head.

### C++17 Code
```cpp
class Solution {
public:
    ListNode* sortList(ListNode* head) {
        // Base case: if list has 0 or 1 elements, it is already sorted
        if (head == nullptr || head->next == nullptr) {
            return head;
        }
        
        // Find the middle of the list
        ListNode* mid = getMid(head);
        ListNode* right_head = mid->next;
        mid->next = nullptr; // Split the list into two halves
        
        // Recursively sort the two halves
        ListNode* left = sortList(head);
        ListNode* right = sortList(right_head);
        
        // Merge the sorted halves
        return merge(left, right);
    }

private:
    // Helper function to find the midpoint
    ListNode* getMid(ListNode* head) {
        ListNode* slow = head;
        ListNode* fast = head->next; // Start fast one step ahead
        
        while (fast != nullptr && fast->next != nullptr) {
            slow = slow->next;
            fast = fast->next->next;
        }
        return slow; // slow will point to the middle node (or first middle in even length)
    }
    
    // Helper function to merge two sorted lists
    ListNode* merge(ListNode* list1, ListNode* list2) {
        ListNode dummy(0);
        ListNode* curr = &dummy;
        
        while (list1 != nullptr && list2 != nullptr) {
            if (list1->val < list2->val) {
                curr->next = list1;
                list1 = list1->next;
            } else {
                curr->next = list2;
                list2 = list2->next;
            }
            curr = curr->next;
        }
        
        // Append remaining nodes
        if (list1 != nullptr) {
            curr->next = list1;
        } else {
            curr->next = list2;
        }
        
        return dummy.next;
    }
};
```


#### Complexity Summary

| Scenario | Time Complexity | Space Complexity | Reasoning |
| :--- | :--- | :--- | :--- |
| **Worst Case** | $O(N \log N)$ | $O(\log N)$ | Always splits the list in half. Recursive stack depth is $\log N$. |
| **Average Case** | $O(N \log N)$ | $O(\log N)$ | Standard behavior of Merge Sort. |
| **Best Case** | $O(N \log N)$ | $O(\log N)$ | Even if the list is already sorted, we split and merge. |

*Note: It is possible to implement Merge Sort iteratively using bottom-up merging to achieve $O(1)$ space, but it is highly complex and rarely asked to be coded in full during a 45-minute interview.*


---

## Q15 Clone Random Pointer

### 1. Problem Explanation

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


### 2. Brute Solution

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


### 3. Optimal Solution & Complexities

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


#### Complexity Summary

| Solution | Time Complexity | Space Complexity | Details |
| :--- | :--- | :--- | :--- |
| **Hash Map (Brute)** | $O(N)$ | $O(N)$ | Map size scales linearly with list length. |
| **Interleaving (Optimal)** | $O(N)$ | $O(1)$ | No extra structures. Inline pointer restructuring only. |


---

## Q16 Segregate Even Odd LL

### 1. Problem Explanation

Given the `head` of a singly linked list, rearrange the nodes such that all even-valued nodes appear before all odd-valued nodes, while maintaining the relative order of the even and odd nodes as they appeared in the original list.

### Input
- `head`: A pointer to the head of a singly linked list.

### Output
- A pointer to the head of the rearranged linked list.

### Constraints
- The number of nodes in the list is in the range $[0, 10^5]$.
- $-10^5 \le \text{Node.val} \le 10^5$

### Observations
1. We must segregate based on the **values** of the nodes (i.e. whether `val` is even or odd), not their indices.
2. The relative order of the elements within the even group and within the odd group must remain unchanged. This implies the segregation must be **stable**.
3. Reordering should be done in-place by changing node pointers rather than creating new nodes or copying values.


### 2. Brute Solution

### Algorithm
1. Traverse the linked list and store even-valued nodes in an array/list, and odd-valued nodes in another array/list.
2. Create a new linked list by iterating through the even array first, then the odd array.
3. Return the new list.

### Complexity Analysis
- **Time Complexity:** $O(N)$ because we traverse the list and the arrays.
- **Space Complexity:** $O(N)$ auxiliary space to store the nodes in the arrays.

### C++17 Code
```cpp
#include <vector>

struct ListNode {
    int val;
    ListNode* next;
    ListNode() : val(0), next(nullptr) {}
    ListNode(int x) : val(x), next(nullptr) {}
    ListNode(int x, ListNode* next) : val(x), next(next) {}
};

class SolutionBrute {
public:
    ListNode* segregateEvenOdd(ListNode* head) {
        if (head == nullptr || head->next == nullptr) {
            return head;
        }
        
        std::vector<ListNode*> evens;
        std::vector<ListNode*> odds;
        
        ListNode* curr = head;
        while (curr != nullptr) {
            if (curr->val % 2 == 0) {
                evens.push_back(curr);
            } else {
                odds.push_back(curr);
            }
            curr = curr->next;
        }
        
        ListNode dummy(0);
        ListNode* tail = &dummy;
        
        for (auto* node : evens) {
            tail->next = node;
            tail = node;
        }
        for (auto* node : odds) {
            tail->next = node;
            tail = node;
        }
        tail->next = nullptr; // Terminate list
        
        return dummy.next;
    }
};
```


### 3. Optimal Solution & Complexities

### Algorithm
1. Initialize two dummy nodes: `even_dummy` and `odd_dummy`.
2. Keep two pointers: `even_tail` pointing to `even_dummy`, and `odd_tail` pointing to `odd_dummy`.
3. Traverse the list using a `curr` pointer:
   - If `curr->val % 2 == 0`, append to the even list: `even_tail->next = curr`, then `even_tail = even_tail->next`.
   - Else, append to the odd list: `odd_tail->next = curr`, then `odd_tail = odd_tail->next`.
   - Move `curr = curr->next`.
4. Connect the two lists: `even_tail->next = odd_dummy.next`.
5. Terminate the odd list tail: `odd_tail->next = nullptr`.
6. Return `even_dummy.next` if it is not null; otherwise, return `odd_dummy.next`.

### C++17 Code
```cpp
class Solution {
public:
    ListNode* segregateEvenOdd(ListNode* head) {
        if (head == nullptr || head->next == nullptr) {
            return head;
        }
        
        ListNode even_dummy(0);
        ListNode odd_dummy(0);
        
        ListNode* even_tail = &even_dummy;
        ListNode* odd_tail = &odd_dummy;
        
        ListNode* curr = head;
        while (curr != nullptr) {
            // Check for even numbers (works correctly for negative integers too)
            if (curr->val % 2 == 0) {
                even_tail->next = curr;
                even_tail = even_tail->next;
            } else {
                odd_tail->next = curr;
                odd_tail = odd_tail->next;
            }
            curr = curr->next;
        }
        
        // Stitch the lists together
        even_tail->next = odd_dummy.next;
        odd_tail->next = nullptr; // Crucial: break any cycles
        
        return even_dummy.next;
    }
};
```


#### Complexity Summary

| Metric | Complexity | Reasoning |
| :--- | :--- | :--- |
| **Time Complexity** | $O(N)$ | We traverse the list exactly once. |
| **Space Complexity** | $O(1)$ | Stack-allocated dummy nodes, no dynamic heap allocations. |


---

## Q17 Rotate LL

### 1. Problem Explanation

Given the `head` of a linked list, rotate the list to the right by `k` places.

### Input
- `head`: A pointer to the head of a singly linked list.
- `k`: A non-negative integer representing the number of rotations.

### Output
- A pointer to the head of the rotated linked list.

### Constraints
- The number of nodes in the list is in the range $[0, 500]$.
- $-100 \le \text{Node.val} \le 100$
- $0 \le k \le 2 \cdot 10^9$

### Observations
1. If $k = 0$, or the list has 0 or 1 nodes, the list remains unchanged.
2. Rotating a list of length $L$ by $L$ times results in the same list. Hence, the effective number of rotations is $k \bmod L$.
3. Rotation involves shifting the tail nodes to the front. The last $k \bmod L$ nodes become the first nodes of the new list.


### 2. Brute Solution

### Algorithm
1. If $k = 0$, return `head`.
2. For each rotation from $1$ to $k$:
   - Traverse the list to find the second-to-last node and the tail node.
   - Set the tail's next to the current head.
   - Set the second-to-last node's next to `nullptr`.
   - Update the head to be the tail.
3. Return the new head.

### Complexity Analysis
- **Time Complexity:** $O(k \times N)$ where $N$ is the number of nodes. Since $k$ can be up to $2 \cdot 10^9$, this will result in a Time Limit Exceeded (TLE) error.
- **Space Complexity:** $O(1)$ auxiliary space.

### C++17 Code
```cpp
struct ListNode {
    int val;
    ListNode* next;
    ListNode() : val(0), next(nullptr) {}
    ListNode(int x) : val(x), next(nullptr) {}
    ListNode(int x, ListNode* next) : val(x), next(next) {}
};

class SolutionBrute {
public:
    ListNode* rotateRight(ListNode* head, int k) {
        if (head == nullptr || head->next == nullptr || k == 0) {
            return head;
        }
        
        ListNode* curr = head;
        int length = 0;
        while (curr != nullptr) {
            length++;
            curr = curr->next;
        }
        
        int effective_k = k % length;
        for (int i = 0; i < effective_k; ++i) {
            ListNode* temp = head;
            while (temp->next->next != nullptr) {
                temp = temp->next;
            }
            ListNode* tail = temp->next;
            temp->next = nullptr;
            tail->next = head;
            head = tail;
        }
        
        return head;
    }
};
```


### 3. Optimal Solution & Complexities

### Algorithm
1. **Handle Base Cases:** If `head` is null, has one node, or $k = 0$, return `head`.
2. **Compute Length:** Traverse the list to find the tail node and compute the length $L$.
3. **Handle Cycle:** Connect `tail->next = head` to form a circular list.
4. **Determine Cut Point:** Calculate effective rotations: $k = k \bmod L$. If $k = 0$, disconnect the tail and return `head`.
5. **Locate New Tail:** Traverse $L - k - 1$ steps from the head to reach the node that will become the new tail.
6. **Break Circle:**
   - The `new_head` will be `new_tail->next`.
   - Break the circular connection: `new_tail->next = nullptr`.
7. **Return:** Return `new_head`.

### C++17 Code
```cpp
class Solution {
public:
    ListNode* rotateRight(ListNode* head, int k) {
        // Base cases
        if (head == nullptr || head->next == nullptr || k == 0) {
            return head;
        }
        
        // Step 1: Find the length of the list and the tail node
        ListNode* tail = head;
        int length = 1;
        while (tail->next != nullptr) {
            tail = tail->next;
            length++;
        }
        
        // Step 2: Calculate effective rotation index
        k = k % length;
        if (k == 0) {
            return head; // No rotation needed
        }
        
        // Step 3: Make the list circular
        tail->next = head;
        
        // Step 4: Find the node before the new head (new tail)
        // We need to move (length - k - 1) steps from head
        ListNode* new_tail = head;
        for (int i = 0; i < length - k - 1; ++i) {
            new_tail = new_tail->next;
        }
        
        // Step 5: Update the head and break the circular connection
        ListNode* new_head = new_tail->next;
        new_tail->next = nullptr;
        
        return new_head;
    }
};
```


#### Complexity Summary

| Solution | Time Complexity | Space Complexity | Reasoning |
| :--- | :--- | :--- | :--- |
| **Brute Force** | $O(k \times N)$ | $O(1)$ | Moves the tail node one-by-one $k$ times. |
| **Circular Linkage (Optimal)** | $O(N)$ | $O(1)$ | Performs two partial passes over the list. |


---

## Q18 Delete Key Occurrences

### 1. Problem Explanation

Given the `head` of a linked list and an integer `val`, remove all the nodes of the linked list that has `Node.val == val`, and return the new head of the modified linked list.

### Input
- `head`: A pointer to the head of a singly linked list.
- `val`: An integer representing the target key to be deleted.

### Output
- A pointer to the head of the modified linked list.

### Constraints
- The number of nodes in the list is in the range $[0, 10^4]$.
- $1 \le \text{Node.val} \le 50$
- $0 \le val \le 50$

### Observations
1. The target value `val` might appear at the head of the list, possibly multiple times consecutively (e.g., `2 -> 2 -> 3`, deleting `2`).
2. The target value might appear at the tail of the list.
3. The list might become empty after deleting all nodes (e.g., `2 -> 2`, deleting `2`).
4. In languages with manual memory management like C++, we should properly deallocate (delete) the removed nodes to prevent memory leaks.


### 2. Brute Solution

### Algorithm
1. Traverse the linked list and copy the values of all nodes that are NOT equal to `val` into an array/vector.
2. Create a new linked list using the values from the array.
3. Delete the original linked list to prevent memory leaks.
4. Return the head of the new list.

### Complexity Analysis
- **Time Complexity:** $O(N)$ because we traverse the list and rebuild it.
- **Space Complexity:** $O(N)$ auxiliary space to store the values.

### C++17 Code
```cpp
#include <vector>

struct ListNode {
    int val;
    ListNode* next;
    ListNode() : val(0), next(nullptr) {}
    ListNode(int x) : val(x), next(nullptr) {}
    ListNode(int x, ListNode* next) : val(x), next(next) {}
};

class SolutionBrute {
public:
    ListNode* removeElements(ListNode* head, int val) {
        std::vector<int> remaining_vals;
        ListNode* curr = head;
        
        // Step 1: Collect non-target values
        while (curr != nullptr) {
            if (curr->val != val) {
                remaining_vals.push_back(curr->val);
            }
            curr = curr->next;
        }
        
        // Free original list memory
        curr = head;
        while (curr != nullptr) {
            ListNode* temp = curr;
            curr = curr->next;
            delete temp;
        }
        
        // Step 2: Reconstruct list
        ListNode dummy(0);
        ListNode* tail = &dummy;
        for (int v : remaining_vals) {
            tail->next = new ListNode(v);
            tail = tail->next;
        }
        
        return dummy.next;
    }
};
```


### 3. Optimal Solution & Complexities

### Algorithm
1. Create a `dummy` node on the stack or heap: `ListNode dummy(0); dummy.next = head;`.
2. Initialize `prev = &dummy` and `curr = head`.
3. Loop while `curr != nullptr`:
   - If `curr->val == val`:
     - Connect the previous node to the next: `prev->next = curr->next`.
     - Save the next node: `ListNode* next_node = curr->next`.
     - Delete current node: `delete curr`.
     - Move current pointer: `curr = next_node`.
   - If `curr->val != val`:
     - Advance `prev = curr`.
     - Advance `curr = curr->next`.
4. Return `dummy.next`.

### Why Optimal?
- Cleans up edge cases (deleting head, deleting all nodes) without nested loops or custom head modifications.
- Performs deletion in a single pass ($O(N)$ time) with zero heap-allocation overhead ($O(1)$ space).

### C++17 Code
```cpp
class Solution {
public:
    ListNode* removeElements(ListNode* head, int val) {
        // Dummy node initialized on stack to avoid memory leak
        ListNode dummy(0);
        dummy.next = head;
        
        ListNode* prev = &dummy;
        ListNode* curr = head;
        
        while (curr != nullptr) {
            if (curr->val == val) {
                // Link prev node to curr's next node
                prev->next = curr->next;
                
                // Store curr for deletion, then advance
                ListNode* to_delete = curr;
                curr = curr->next;
                
                // Deallocate memory
                delete to_delete;
            } else {
                // Move prev and curr forward
                prev = curr;
                curr = curr->next;
            }
        }
        
        return dummy.next;
    }
};
```


#### Complexity Summary

| Solution | Time Complexity | Space Complexity | Reasoning |
| :--- | :--- | :--- | :--- |
| **Vector Copy** | $O(N)$ | $O(N)$ | Extra memory of size $N$ is allocated. |
| **Better (No Dummy)** | $O(N)$ | $O(1)$ | No extra memory, but complex head traversal. |
| **Optimal (Dummy)** | $O(N)$ | $O(1)$ | No extra memory, clean single pass. |


---

## Q19 LRU Cache

### 1. Problem Explanation

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


### 2. Brute Solution

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


### 3. Optimal Solution & Complexities

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


#### Complexity Summary

| Operation | Time Complexity (Average) | Space Complexity | Reasoning |
| :--- | :--- | :--- | :--- |
| **`get(key)`** | $O(1)$ | $O(1)$ | Hash Map lookup is $O(1)$, moving node in DLL is $O(1)$. |
| **`put(key, value)`**| $O(1)$ | $O(1)$ | Hash Map insertion/deletion is $O(1)$, adding/deleting DLL node is $O(1)$. |
| **Overall Cache** | - | $O(C)$ | $C$ is the capacity. At most $C$ nodes and map entries are maintained. |


---
