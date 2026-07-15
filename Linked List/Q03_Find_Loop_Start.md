# Find Start of Loop in Linked List

## 1. Problem Statement
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

---

## 2. Interview Clarification Questions
1. **Can we modify the node values or structure?**
   - *Answer:* No, the list should remain unmodified.
2. **What should we return if there is no cycle?**
   - *Answer:* Return `nullptr`.
3. **Is it guaranteed that the cycle has only one entry point?**
   - *Answer:* Yes, since it is a singly linked list, each node has at most one `next` pointer, meaning there can be at most one entry point.

---

## 3. Thinking Process
- **First Instinct (Hashing):** Traverse the list and store each node's address in a hash set. The first node we encounter that is already in the hash set is the start of the cycle. This takes $O(N)$ time and $O(N)$ space.
- **Space Optimization:** We want to achieve $O(1)$ auxiliary space.
- **Two Pointers:** We can use Floyd's Tortoise and Hare algorithm to detect if a cycle exists. Let the meeting point of `slow` and `fast` be `meet`.
- **Finding the Start:** Once they meet at `meet`, reset `slow` to `head`. Move both `slow` and `meet` (which is tracked by `fast`) at the same speed of 1 step per iteration. The node where they meet next will be the start of the loop.
- Let's prove this mathematically to ensure it is always correct.

---

## 4. Pattern Recognition
- **Pattern:** Floyd's Cycle Detection + Mathematical Distance Relations.
- **Why this topic?** This is a classic follow-up to cycle detection. It shows how relative speed and cyclic arithmetic can locate a specific structural boundary in $O(1)$ space.

---

## 5. Brute Force Solution (Hash Set)
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

---

## 6. Better Solution
> [!NOTE]
> Like the loop detection problem, there is no intermediate solution between the hash set approach and the optimal Floyd-based pointer math approach. We jump directly to the optimal solution.

---

## 7. Optimal Solution (Floyd's Algorithm with Math Proof)

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

---

## 8. Code Walkthrough
- **Cycle Detection:** `slow` moves by 1, `fast` moves by 2. If they meet, `hasCycle` is set to `true` and we exit the loop.
- **Check Validity:** If `hasCycle` is `false`, it means `fast` hit the end of the list; we return `nullptr`.
- **Start Search:** Reset `slow` to `head`.
- **Same-Speed Traversal:** While `slow != fast`, we advance both pointers by 1 step.
- **Meeting Point:** Once the condition is met, both pointers are at the start of the loop. We return `slow`.

---

## 9. Dry Run
Input: `3 -> 2 -> 0 -> -4` (where `-4` links to `2`)
- $L_1 = 1$ (distance from node `3` to node `2`)
- $L_2 = 2$ (distance from node `2` to node `-4`)
- Cycle length $C = 3$ (nodes: `2`, `0`, `-4`)

### Phase 1: Cycle Detection
- Start: `slow = 3`, `fast = 3`
- Step 1: `slow = 2`, `fast = 0`
- Step 2: `slow = 0`, `fast = 2`
- Step 3: `slow = -4`, `fast = -4` (Meet!)

### Phase 2: Find Start
- Reset: `slow = 3` (head), `fast = -4` (meeting point)
- Step 1:
  - `slow = slow->next` -> Node `2`
  - `fast = fast->next` -> Node `2`
- Loop terminates because `slow == fast` (both point to Node `2`).
- Returns Node `2`.

---

## 10. Complexity Analysis

| Approach | Time Complexity | Space Complexity | Reasoning |
| :--- | :--- | :--- | :--- |
| **Hash Set** | $O(N)$ | $O(N)$ | Single pass traversal. Hash set stores up to $N$ pointers. |
| **Floyd's Algorithm**| $O(N)$ | $O(1)$ | Phase 1 takes $\le N$ steps. Phase 2 takes $L_1 \le N$ steps. Total operations are linear. No auxiliary memory allocated. |

---

## 11. Common Mistakes
- **Forgetting to check if a cycle exists first:** Resetting `slow` and running the second loop without checking if `fast` actually detected a cycle can lead to infinite loops.
- **Moving `fast` at double speed in Phase 2:** In the second phase, both pointers MUST move at 1 step per iteration. If `fast` continues to move at 2 steps, they may never meet or meet at the wrong node.
- **Incorrect null pointer checks:** Not checking `fast` and `fast->next` inside the loop condition.

---

## 12. Edge Cases

| Edge Case | Input Example | Expected Output | Why it matters |
| :--- | :--- | :--- | :--- |
| **No cycle** | `1 -> 2 -> nullptr` | `nullptr` | Code must exit early without running Phase 2. |
| **Entire list is cycle**| `1 -> 2 -> 1` | Node `1` (head) | $L_1 = 0$. Meeting point will be node `1`. Resetting `slow` to `head` means `slow == fast` immediately. |
| **Self loop at tail** | `1 -> 2 -> 2` | Node `2` | Meeting point will be node `2`. In Phase 2, `slow` travels 1 step to `2`, `fast` stays at `2`. |

---

## 13. Interview Explanation
> "To find the starting node of a cycle in a linked list, I use Floyd's cycle detection algorithm. First, I use slow and fast pointers to detect if a cycle exists. If they meet, we confirm a cycle is present. Mathematically, the distance from the head to the loop start is equal to the distance from the meeting point to the loop start, plus some number of full cycle laps. Therefore, by resetting the slow pointer to the head of the list and keeping the fast pointer at the meeting node, and then advancing both pointers one step at a time, they are guaranteed to meet exactly at the starting node of the cycle. This approach requires $O(N)$ time and $O(1)$ auxiliary space."

---

## 14. Follow-up Questions
1. **Can you prove the math on a whiteboard?**
   - *Yes, draw the linear part $L_1$, the cyclic part loop, label the meeting point $L_2$, and write out the distance equations showing $L_1 = (k-1)C + (C - L_2)$ as shown in the proof.*
2. **What if the list has a cycle but you want to remove it?**
   - *Once the start of the loop is found, traverse the loop using a temporary pointer until `temp->next` is the start node. Then set `temp->next = nullptr` to break the cycle.*

---

## 15. Variations
- **Find the Duplicate Number (LeetCode 287):** An array of size $n+1$ with elements in $[1, n]$ contains a duplicate. Treating indices as nodes and values as next pointers forms a cycle. The duplicate value is the entry point of the cycle.

---

## 16. Revision Notes
- **Key Equation:** $L_1 = (k-1)C + (C - L_2)$
- **Algorithm flow:**
  1. Detect loop (slow: 1 step, fast: 2 steps).
  2. If they meet, reset `slow = head`.
  3. Move both at 1 step until they meet again.
  4. Return the meeting node.

---

## 17. Memorization Version (< 1 minute read)
- **Goal:** Find cycle start node in $O(1)$ space.
- **Steps:**
  1. Find meeting node `meet` using slow (1 step) and fast (2 steps) pointers.
  2. Reset `slow = head`, keep `fast = meet`.
  3. Move both 1 step at a time: `slow = slow->next`, `fast = fast->next`.
  4. Stop when `slow == fast`. Return `slow`.
- **Complexity:** $O(N)$ Time, $O(1)$ Space.
