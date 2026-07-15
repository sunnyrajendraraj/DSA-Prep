# Rotate Linked List by K

## 1. Problem Statement

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

---

## 2. Interview Clarification

Before coding, clarify:
1. **Q:** Can `k` be larger than the length of the list?
   **A:** Yes, $k$ can be up to $2 \cdot 10^9$. We must handle this efficiently without iterating $k$ times.
2. **Q:** What should we return if the list is empty?
   **A:** Return `nullptr` immediately.
3. **Q:** Can we modify the values inside the nodes?
   **A:** No, rotation must be done by modifying the pointer links.

---

## 3. Thinking Process

Let's build the intuition step-by-step:
1. **Visualizing Rotation:**
   Suppose the list is `1 -> 2 -> 3 -> 4 -> 5` and $k = 2$.
   - The length of the list $L = 5$.
   - $k \bmod 5 = 2$. We need to move the last 2 nodes (`4` and `5`) to the front.
   - The resulting list is `4 -> 5 -> 1 -> 2 -> 3`.
2. **Connecting Tail to Head:**
   - Notice that the old tail `5` now connects to the old head `1`.
   - This suggests that during our traversal to find the length, we can link `tail->next = head`, forming a temporary circular linked list.
3. **Finding the Splitting Point:**
   - In our example, the node `3` is the new tail. Its next pointer should be set to `nullptr`.
   - Node `4` is the new head.
   - Where is the new tail located?
     - 0-indexed position of the new tail from the start is $L - (k \bmod L) - 1$.
     - In our example: $5 - 2 - 1 = 2$. The 2nd node (0-indexed) is `3`.
   - We start at the `head` and walk $L - (k \bmod L) - 1$ steps. The node we stop at is the `new_tail`.
   - The `new_head` is `new_tail->next`.
   - We then break the loop: `new_tail->next = nullptr`.

---

## 4. Pattern Recognition

This problem uses the **Circular Linkage & Break Pattern**.
- **Why this pattern?** By temporarily joining the tail to the head, we simplify the movement of the start point of the list. We then break the circle at the calculated offset to restore the linear structure.

---

## 5. Brute Force Solution

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

---

## 6. Better Solution

There is no intermediate solution; the brute force jumps directly to the optimal $O(N)$ solution by calculating the length first.

---

## 7. Optimal Solution

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

---

## 8. Code Walkthrough

1. `ListNode* tail = head; int length = 1;`
   - Start traversing from the head. We initialize `length = 1` because we stop when `tail->next == nullptr` (at the last node).
2. `k = k % length;`
   - Removes redundant rotations. For example, if $L = 5$ and $k = 12$, $12 \bmod 5 = 2$.
3. `tail->next = head;`
   - Creates the circular loop. The list is now infinitely traversable.
4. `for (int i = 0; i < length - k - 1; ++i) { new_tail = new_tail->next; }`
   - Moves to the exact position of the new tail.
5. `ListNode* new_head = new_tail->next; new_tail->next = nullptr;`
   - Captures the new head pointer and severs the circular loop by setting the new tail's next to `nullptr`.

---

## 9. Dry Run

List: `1 -> 2 -> 3 -> 4 -> 5`, $k = 2$.

1. **Length calculation:**
   - Loop runs. `tail` stops at `5`. `length = 5`.
2. **Effective $k$:**
   - $k = 2 \bmod 5 = 2$.
3. **Make circular:**
   - `5->next = 1`.
4. **Locate new tail:**
   - Steps to move: $5 - 2 - 1 = 2$ steps.
   - Start `new_tail = 1`.
   - Step 1: `new_tail = 2`.
   - Step 2: `new_tail = 3`.
5. **Cut and Return:**
   - `new_head = new_tail->next` $\implies$ `new_head = 4`.
   - `new_tail->next = nullptr` $\implies$ `3->next = nullptr`.
   - Return `4`.
   - Result: `4 -> 5 -> 1 -> 2 -> 3`.

---

## 10. Complexity Analysis

| Solution | Time Complexity | Space Complexity | Reasoning |
| :--- | :--- | :--- | :--- |
| **Brute Force** | $O(k \times N)$ | $O(1)$ | Moves the tail node one-by-one $k$ times. |
| **Circular Linkage (Optimal)** | $O(N)$ | $O(1)$ | Performs two partial passes over the list. |

---

## 11. Common Mistakes

1. **Incorrect Length Calculation:**
   - Setting the initial length to `0` and looping while `curr != nullptr` will leave `tail` pointing to `nullptr` at the end, meaning we lose the reference to the last node.
2. **Modulo Divide by Zero:**
   - Not checking if the list is empty before performing `k % length`. If the list is empty, `length` is `0`, causing a runtime crash.
3. **Loop Bounds Off-By-One:**
   - Shifting the loop steps to $L - k$ instead of $L - k - 1$. This causes the split to occur one node too late or early.

---

## 12. Edge Cases

| Edge Case | Input | Expected Output | Why It Matters |
| :--- | :--- | :--- | :--- |
| **Empty list** | `nullptr`, $k = 5$ | `nullptr` | Prevent null pointer dereference. |
| **Single node** | `1`, $k = 9$ | `1` | Checks behavior when length is 1. |
| **k is multiple of length** | `1->2`, $k = 4$ | `1->2` | Resulting rotation brings the list back to original state. |
| **k = 0** | `1->2`, $k = 0$ | `1->2` | Guard code must return head immediately. |

---

## 13. Interview Explanation

"To rotate a linked list to the right by $k$ places efficiently, we first calculate the length of the list, say $L$, and locate the tail node. We then connect the tail node to the head of the list to make it circular. 

Next, we calculate the effective rotations as $k \bmod L$. The new tail node will be located at position $L - (k \bmod L) - 1$ from the original head. We traverse to this node, set the node immediately after it as our new head, and break the circular connection by setting the new tail's next pointer to `nullptr`. This achieves the rotation in $O(N)$ time and $O(1)$ space."

---

## 14. Follow-up Questions

1. **Q:** What if we want to rotate the list to the *left* instead of the right?
   **A:** Rotating to the left by $k$ places is equivalent to rotating to the right by $L - (k \bmod L)$ places.
2. **Q:** Can this be solved using a auxiliary data structure?
   **A:** Yes, we could copy nodes to an array, rotate the array, and rebuild the list, but it would use $O(N)$ extra space.

---

## 15. Variations

1. **Rotate Array by K elements:**
   - Standard array rotation. Uses three reverse operations: reverse first part, reverse second part, reverse whole array.

---

## 16. Revision Notes

- **Circular Trick:** Connecting `tail->next = head` simplifies the rotation problem into a simple traversal problem.
- **Formula:** New tail is at $L - (k \bmod L) - 1$ steps from head.
- **Modulo Constraint:** Crucial to execute `k % length` to avoid $O(K)$ time limits.

---

## 17. Memorization Version

```cpp
// 1. Traverse to find tail and length.
// 2. k = k % length; if (k == 0) return head;
// 3. Connect tail->next = head.
// 4. Move (length - k - 1) steps from head to find new_tail.
// 5. new_head = new_tail->next, new_tail->next = nullptr.
// 6. Return new_head.
```
