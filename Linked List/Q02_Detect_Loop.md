# Detect Loop in Linked List

## 1. Problem Statement
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

---

## 2. Interview Clarification Questions
1. **Can we modify the structure of the linked list or values of the nodes?**
   - *Answer:* No, the list should be treated as read-only.
2. **Can we use auxiliary memory?**
   - *Answer:* We can start with a hash-set based approach ($O(N)$ space), but the interviewer will expect us to optimize it to $O(1)$ auxiliary space.

---

## 3. Thinking Process
- **First Instinct (Hashing):** As we traverse the linked list, we can store the memory addresses of visited nodes in a hash set. If we encounter a node that is already in the set, we have detected a cycle.
- **Space Optimization:** To avoid $O(N)$ space, we need a way to detect cycles using only a few pointers.
- **Analogy (The Race Track):** Imagine two runners on a circular track. If one runner runs faster than the other, the faster runner will eventually lap the slower runner. If the track is straight, the faster runner will just reach the end and stop.
- **Floyd's Cycle Detection (Tortoise and Hare):** 
  - Initialize two pointers: `slow` and `fast` at the `head`.
  - Move `slow` by 1 node at a time.
  - Move `fast` by 2 nodes at a time.
  - If a cycle exists, `fast` and `slow` will eventually point to the same node.
  - If no cycle exists, `fast` (or `fast->next`) will reach `nullptr` first.

---

## 4. Pattern Recognition
- **Pattern:** Fast and Slow Pointers (Floyd's Cycle-Finding Algorithm).
- **Why this topic?** This pattern is ideal for detection of cycles in linear data structures without allocating extra node markers or using hash tables.

---

## 5. Brute Force Solution (Hash Set)
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

---

## 6. Better Solution
> [!NOTE]
> The hash set approach is the standard brute force. The only intermediate approach involves modifying node values to a special sentinel value (e.g., `INT_MAX`) or breaking links as we traverse, but these destroy the input list. Thus, we jump directly to Floyd's Tortoise and Hare algorithm as the optimal, non-destructive approach.

---

## 7. Optimal Solution (Floyd's Tortoise and Hare)

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

---

## 8. Code Walkthrough
- **`if (!head || !head->next) return false;`**: Handles cases where there are fewer than 2 nodes, which cannot form a cycle (unless a single node points to itself, which `head->next` would catch).
- **`ListNode* slow = head; ListNode* fast = head;`**: Both start at the head.
- **`while (fast != nullptr && fast->next != nullptr)`**: Ensures that `fast->next->next` will not cause a null pointer dereference.
- **`slow = slow->next; fast = fast->next->next;`**: The slow pointer takes 1 step; the fast pointer takes 2 steps.
- **`if (slow == fast) return true;`**: Check if they have met. If yes, return `true`.

---

## 9. Dry Run
Input: `3 -> 2 -> 0 -> -4` (where `-4` links to `2`)

```
   3 ---> 2 ---> 0 ---> -4
          ^              |
          |______________|
```

| Step | `slow` position | `fast` position | `slow == fast`? |
| :--- | :--- | :--- | :--- |
| **0 (Init)**| Node `3` (val=3) | Node `3` (val=3) | - |
| **1** | Node `2` (val=2) | Node `0` (val=0) | No |
| **2** | Node `0` (val=0) | Node `2` (val=2) | No |
| **3** | Node `-4` (val=-4)| Node `-4` (val=-4)| Yes (Meet at `-4`) |

**Result:** Returns `true`.

---

## 10. Complexity Analysis

| Approach | Time Complexity | Space Complexity | Reasoning |
| :--- | :--- | :--- | :--- |
| **Hash Set** | $O(N)$ | $O(N)$ | Visits each node once. Stores up to $N$ addresses in the set. |
| **Floyd's Algorithm**| $O(N)$ | $O(1)$ | Slower pointer enters loop after $X$ steps and meets fast within $C$ steps. Total steps $\leq X + C = N$. Uses only two pointer variables. |

---

## 11. Common Mistakes
- **Incorrect loop guard:** Using `while (fast != nullptr)` instead of `while (fast != nullptr && fast->next != nullptr)`. This leads to a crash if the list has an odd length and no cycle.
- **Updating pointers incorrectly:** Skipping the update steps or updating `fast` before checking validity.
- **Comparing node values instead of addresses:** Writing `slow->val == fast->val`. Multiple nodes can have the same value; we must compare the pointer addresses (`slow == fast`).

---

## 12. Edge Cases

| Edge Case | Input Example | Expected Output | Why it matters |
| :--- | :--- | :--- | :--- |
| **Empty list** | `nullptr` | `false` | Prevents null dereference right at the start. |
| **Single node, no cycle**| `1 -> nullptr` | `false` | Loop guard `fast->next != nullptr` handles it immediately. |
| **Single node with cycle**| `1 -> (itself)` | `true` | `slow` and `fast` will meet on first step. |
| **Two nodes, no cycle**| `1 -> 2 -> nullptr` | `false` | `fast` reaches null in one jump. |

---

## 13. Interview Explanation
> "To detect if a linked list contains a cycle, the optimal approach is Floyd's Cycle Detection algorithm, also known as the Tortoise and Hare algorithm. We initialize two pointers, `slow` and `fast`, at the head of the list. We traverse the list, moving `slow` by one step and `fast` by two steps. If there is no cycle, `fast` will reach the end of the list (`nullptr`) and we return `false`. If there is a cycle, `fast` will enter the loop first, and because its relative speed is 1 step faster than `slow`, it will decrease the distance between them by 1 node per iteration, eventually meeting `slow` inside the loop. This algorithm runs in $O(N)$ time and uses $O(1)$ auxiliary space."

---

## 14. Follow-up Questions
1. **Can you find the length of the cycle?**
   - *Yes. Once `slow` and `fast` meet, keep `slow` stationary and advance `fast` one step at a time, counting the steps until `fast` meets `slow` again.*
2. **What if the fast pointer moves 3 steps instead of 2? Does it still meet?**
   - *Yes, the relative speed becomes 2 steps per iteration. The meeting is still guaranteed, but the math varies. The gap decreases by 2 (modulo cycle length) each step. 2 steps is optimal because it guarantees a meeting in the fewest operations without risking skipping.*

---

## 15. Variations
- **Happy Number (LeetCode 202):** Checking if digit square sums lead to 1 or cycle endlessly.
- **Find the Duplicate Number (LeetCode 287):** Treating the array values as pointers to construct a virtual linked list and finding the cycle.

---

## 16. Revision Notes
- **Mnemonic:** **"Hare runs twice as fast; if they run in a circle, the Hare must lap the Tortoise."**
- **Loop Guard:** `while (fast && fast->next)`
- **Key concept:** Relative speed is $1$. Gap shrinks by $1$ each step.

---

## 17. Memorization Version (< 1 minute read)
- **Goal:** Detect loop in $O(1)$ space.
- **Strategy:** Floyd's Tortoise and Hare.
  - Set `slow = head`, `fast = head`.
  - Loop: `while (fast && fast->next)`.
  - Update: `slow = slow->next`, `fast = fast->next->next`.
  - If `slow == fast`, return `true` (cycle exists).
- **Exit:** If loop finishes, return `false`.
- **Complexity:** $O(N)$ Time, $O(1)$ Space.
