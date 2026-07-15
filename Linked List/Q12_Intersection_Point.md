# Intersection Point of Two Linked Lists

## 1. Problem Statement

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

---

## 2. Interview Clarification

Before coding, clarify the following:
1. **Is it possible that there is no intersection?**
   * *Answer*: Yes, in that case, your function should return `nullptr`.
2. **Are the values of nodes guaranteed to be unique?**
   * *Answer*: No. Thus, comparing values is incorrect; we must compare node memory addresses.
3. **Should we restore the list structure?**
   * *Answer*: Yes, do not modify the nodes or their `next` pointers permanently.
4. **What are the constraints on time and space?**
   * *Answer*: The optimal solution should run in $O(M + N)$ time and use $O(1)$ auxiliary space.

---

## 3. Thinking Process

Let's think through how to solve this:
* **First Instinct (Brute Force)**:
  * For every node in list A, we can loop through all nodes in list B and check if they point to the exact same node (same memory address).
  * If we find a match, that is the intersection point.
  * This requires nested loops. The time complexity is $O(M \times N)$ and space is $O(1)$. This is too slow for large list sizes.
* **Better Solution (Hash Set)**:
  * To avoid nested loops, we can traverse list A and store the addresses of all its nodes in a hash set.
  * Then, we traverse list B. For each node in B, we check if its address exists in our hash set.
  * The first address that is present in the hash set is the intersection point.
  * This runs in $O(M + N)$ time. However, the space complexity is $O(M)$ to store the nodes of list A in the set.
* **Can we achieve O(1) space? (Optimal Insight)**:
  * Let's analyze the difference in lengths.
  * Suppose `listA` has length 5 and `listB` has length 6, and they intersect at a node.
  * Let the length of list A before intersection be $a$.
  * Let the length of list B before intersection be $b$.
  * Let the shared length after intersection be $c$.
  * Length of A = $a + c$.
  * Length of B = $b + c$.
  * If we could make both pointers start at the same distance from the intersection, we could just move them at the same speed, and they would meet at the intersection point!
  * **Length Difference Method**:
    * Calculate lengths of both lists.
    * Find the difference: $d = |len(A) - len(B)|$.
    * Move the pointer of the longer list forward by $d$ steps.
    * Now, both pointers are at the same distance from the intersection. Move them step-by-step together until they meet.
  * **Two-Pointer Swap Method (Cleaner)**:
    * Initialize two pointers: `pA = headA`, `pB = headB`.
    * In each step, move both pointers forward by one node.
    * If `pA` reaches `nullptr`, redirect it to `headB`.
    * If `pB` reaches `nullptr`, redirect it to `headA`.
    * They will eventually meet at the intersection node (or both reach `nullptr` at the same time if there is no intersection).
    * Let's prove mathematically why this works.

### Mathematical Proof for the Two-Pointer Swap Method
Let:
- $a$: non-overlapping length of List A.
- $b$: non-overlapping length of List B.
- $c$: overlapping length.

1. **Pointer A** travels:
   * Non-overlapping part of A ($a$ steps) + overlapping part ($c$ steps) $\rightarrow$ reaches end (`nullptr`).
   * Redirected to `headB` $\rightarrow$ travels non-overlapping part of B ($b$ steps).
   * Total distance traveled by Pointer A before meeting = $a + c + b$.

2. **Pointer B** travels:
   * Non-overlapping part of B ($b$ steps) + overlapping part ($c$ steps) $\rightarrow$ reaches end (`nullptr`).
   * Redirected to `headA` $\rightarrow$ travels non-overlapping part of A ($a$ steps).
   * Total distance traveled by Pointer B before meeting = $b + c + a$.

Since $a + c + b = b + c + a$, both pointers travel the exact same distance and will arrive at the start of the intersection at the same time!
* If there is no intersection ($c = 0$):
  * Pointer A travels $a + b$ steps to reach `nullptr`.
  * Pointer B travels $b + a$ steps to reach `nullptr`.
  * They meet at `nullptr` at the exact same step, terminating the loop and returning `nullptr`.

---

## 4. Pattern Recognition

This problem uses the **Two-Pointer / Length Alignment** pattern.
* When traversing two paths of different lengths to reach a common point, we can align their starting distances either by explicitly calculating lengths, or by redirecting each pointer to the other path's head upon completion.
* This is a fundamental pattern for resolving asymmetry in linked lists.

---

## 5. Brute Force Solution (Nested Loops)

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

---

## 6. Better Solution (Hash Set)

### Improvement over Brute Force
Reduces the time complexity to linear by storing visited nodes of one list in a Hash Set, reducing lookup time to $O(1)$.

### Algorithm
1. Create an unordered set of `ListNode*`: `std::unordered_set<ListNode*> visited`.
2. Traverse list A and insert all node pointers into `visited`.
3. Traverse list B. For each node, check if it is in `visited`.
4. If found, return that node (it's the first intersection point).
5. If the loop ends without finding a match, return `nullptr`.

### Complexity Analysis
* **Time Complexity**: $O(M + N)$. We traverse list A once ($M$ steps) and list B once ($N$ steps). Hash set insertions and lookups take $O(1)$ on average.
* **Space Complexity**: $O(M)$ to store the nodes of list A in the set.

### Commented C++17 Code
```cpp
#include <unordered_set>

class HashIntersection {
public:
    ListNode *getIntersectionNode(ListNode *headA, ListNode *headB) {
        std::unordered_set<ListNode*> visited;
        
        // Store all nodes of list A
        ListNode* currA = headA;
        while (currA != nullptr) {
            visited.insert(currA);
            currA = currA->next;
        }
        
        // Traverse list B and look for intersection
        ListNode* currB = headB;
        while (currB != nullptr) {
            if (visited.count(currB)) {
                return currB; // Found intersection
            }
            currB = currB->next;
        }
        
        return nullptr;
    }
};
```

---

## 7. Optimal Solution (Two-Pointer Swap Method)

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

---

## 8. Code Walkthrough

* `if (headA == nullptr || headB == nullptr) return nullptr;`
  * Guard clause: if either list is empty, there can be no intersection.
* `while (pA != pB)`
  * Loop runs as long as the pointers point to different memory addresses.
  * If there is an intersection, they will eventually point to the same node, breaking the loop.
  * If there is no intersection, they will both reach `nullptr` at the same time. Since `nullptr == nullptr` is true, the loop will terminate.
* `pA = (pA == nullptr) ? headB : pA->next;`
  * If `pA` reaches the end, it jumps to `headB` to start traversing list B. Otherwise, it moves to `pA->next`.
* `return pA;`
  * Returns the node pointer where they met (or `nullptr`).

---

## 9. Dry Run

Let's dry run the optimal solution on the following intersecting lists:
* List A: `1 -> 2 -> 3 -> 4 -> nullptr` (length = 4)
* List B: `5 -> 3 -> 4 -> nullptr` (length = 3)
* Intersection starts at node containing `3`.
* Here, $a = 2$ (`1 -> 2`), $b = 1$ (`5`), $c = 2$ (`3 -> 4`).

| Step | `pA` value | `pA` address / state | `pB` value | `pB` address / state |
|:---:|:---:|:---|:---:|:---|
| Setup | `1` | Start of list A | `5` | Start of list B |
| **1** | `2` | Moving on A | `3` | Intersection node |
| **2** | `3` | Intersection node | `4` | Moving on B |
| **3** | `4` | Moving on A | `nullptr`| End of B |
| **4** | `nullptr`| End of A | `1` | Redirected to `headA` |
| **5** | `5` | Redirected to `headB` | `2` | Moving on A |
| **6** | `3` | Intersection node | `3` | Intersection node (Match!) |

**Result**: Loop terminates because `pA == pB` (both pointing to node `3`). Return node `3`.

---

## 10. Complexity Analysis

| Approach | Time Complexity | Space Complexity | Description |
|:---|:---:|:---:|:---|
| **Brute Force** | $O(M \times N)$ | $O(1)$ | Double loop, slow. |
| **Better (Hash Set)** | $O(M + N)$ | $O(M)$ | Fast, but uses extra linear memory. |
| **Optimal (Two Pointers)** | $O(M + N)$ | $O(1)$ | Perfect time and space complexity. |

* **Time Complexity**: $O(M + N)$. In the worst case, each pointer travels at most $M + N$ steps (traversing one list and then the other).
* **Space Complexity**: $O(1)$ auxiliary space since we only use two pointers.

---

## 11. Common Mistakes

1. **Comparing Values instead of Pointers**: Comparing `pA->val == pB->val`. If two nodes have the same value but are at different positions, this incorrectly identifies them as an intersection.
2. **Infinite Loops**: Redirecting `pA` to `headB` when `pA->next` is `nullptr` instead of when `pA` is `nullptr`. If you redirect when `pA->next == nullptr`, you skip the `nullptr` node, which prevents the pointers from ever aligning when there is no intersection, causing an infinite loop.
3. **Modifying List Structures**: Attempting to reverse lists or add dummy nodes to find intersections and not restoring them, violating constraints.

---

## 12. Edge Cases

| Edge Case | Input Example | Expected Output | Why it Matters |
|:---|:---|:---|:---|
| **No Intersection** | `1 -> 2` and `3 -> 4` | `nullptr` | Verifies that pointers terminate at `nullptr` simultaneously. |
| **Full Intersection** | `1 -> 2` and `1 -> 2` (same list) | `1` (head node) | Tests case where heads are identical. |
| **One List is Sublist of Other** | A: `1 -> 2 -> 3`, B: `2 -> 3` | `2` | Tests overlap starting early in one list. |
| **Different Lengths** | A: 100 nodes, B: 2 nodes | Match node or `nullptr` | Tests pointer alignment over large length gaps. |

---

## 13. Interview Explanation

"To find the intersection point of two linked lists, we can't simply compare node values because nodes can have duplicate values. We must compare the node memory addresses.

A brute force nested loop approach takes $O(M \times N)$ time. We can improve this to $O(M + N)$ time using a hash set to store the nodes of the first list, but it requires $O(M)$ extra space.

To achieve $O(1)$ space, we use a two-pointer technique:
1. We place pointer A at the head of list A, and pointer B at the head of list B.
2. We traverse forward. When pointer A reaches the end of list A, we redirect it to the head of list B. Similarly, when pointer B reaches the end of list B, we redirect it to the head of list A.
3. By swapping paths, both pointers travel the exact same total distance ($a + b + c$ steps).
4. Thus, they are guaranteed to meet at the intersection point, or meet at `nullptr` if no intersection exists.
This optimal solution runs in $O(M + N)$ time and uses $O(1)$ space."

---

## 14. Follow-up Questions

1. **How does this relate to finding a cycle in a linked list?**
   * *Answer*: If we connect the tail of list A to the head of list A, it creates a cycle. The intersection point of list A and list B is equivalent to the start of the cycle. We could use Floyd's Cycle Detection Algorithm (Slow/Fast pointers) to find it.
2. **What is the comparison with the "Length Difference" method?**
   * *Answer*: The length difference method also takes $O(M + N)$ time and $O(1)$ space. However, it requires writing more lines of code (calculating lengths, calculating difference, moving one pointer forward, and then looping). The two-pointer swap method is much shorter, cleaner, and less error-prone to write during an interview.

---

## 15. Variations

1. **Linked List Cycle II (Find start of cycle)**:
   * *Relation*: Uses two pointers to detect a cycle, and then uses mathematical properties of distances to find the cycle's starting node.
2. **Find the Duplicate Number (LeetCode 287)**:
   * *Relation*: Uses index-pointer cycles to find duplicates in an array using Floyd's algorithm.

---

## 16. Revision Notes

* **Core Mnemonic**: "Swap heads at the end." Pointer A jumps to B, Pointer B jumps to A.
* **Math Proof Visual**:
  ```
  Path A: [--- a ---] [--- c ---]
  Path B: [--- b ---] [--- c ---]
  
  Traversed by A: a + c + b
  Traversed by B: b + c + a
  They meet at start of c.
  ```
* **Complexity summary**: Time: $O(M + N)$ | Space: $O(1)$ (Optimal) vs $O(M)$ (Better).

---

## 17. Memorization Version

```cpp
// 1-min Review: Intersection of Two LL O(1) Space
ListNode *getIntersectionNode(ListNode *headA, ListNode *headB) {
    if (!headA || !headB) return nullptr;
    ListNode *pA = headA, *pB = headB;
    while (pA != pB) {
        pA = (pA == nullptr) ? headB : pA->next;
        pB = (pB == nullptr) ? headA : pB->next;
    }
    return pA; // Returns intersection node or nullptr
}
```
