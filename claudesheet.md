# COMP 202 — DSA Cheat Sheet

Quick-revision sheet covering Ch1–Ch8 of the syllabus. Code is C.

## Ch1: Basics

### Algorithm characteristics

- **Finiteness**: terminates after a finite number of steps
- **Definiteness**: every step is precise and unambiguous
- **Input**: zero or more well-defined inputs
- **Output**: at least one output
- **Effectiveness**: every step is basic enough to be carried out exactly (in principle, by hand)

### Arrays, pointers, structures

- Array: contiguous memory, O(1) index access, fixed size. `a[i]` is `*(a+i)`.
- Pointer: variable holding an address. `int *p = &x; *p = 5;`
- Dynamic memory: `malloc(n * sizeof(int))`, `free(p)`.
- Structure: groups different types. Use `->` with a pointer to a struct.

```c
struct Node { int data; struct Node *next; };
struct Node *n = malloc(sizeof(struct Node));
n->data = 10; n->next = NULL;
```

### Asymptotic analysis

| Notation | Meaning | Bound |
| --- | --- | --- |
| **O(g)** | f(n) ≤ c·g(n) for n ≥ n₀ | upper (worst case) |
| **Ω(g)** | f(n) ≥ c·g(n) for n ≥ n₀ | lower (best case) |
| **Θ(g)** | c₁·g(n) ≤ f(n) ≤ c₂·g(n) | tight (both) |

Growth order: `1 < log n < n < n log n < n² < n³ < 2ⁿ < n!`

Rules: drop constants and lower-order terms. Nested loops multiply, sequential loops add. A loop where the counter doubles/halves is O(log n).

## Ch2: Stack (LIFO)

Operations: `push`, `pop`, `top/peek`, `isEmpty`, `isFull` (array version). All O(1).

```c
#define MAX 100
int st[MAX], top = -1;

int  isEmpty()   { return top == -1; }
int  isFull()    { return top == MAX - 1; }
void push(int x) { if (isFull()) { printf("Overflow"); return; } st[++top] = x; }
int  pop()       { if (isEmpty()) { printf("Underflow"); return -1; } return st[top--]; }
int  peek()      { return st[top]; }
```

### Infix / prefix / postfix

| Infix | Prefix | Postfix |
| --- | --- | --- |
| A+B | +AB | AB+ |
| (A+B)\*C | \*+ABC | AB+C\* |
| A+B\*C | +A\*BC | ABC\*+ |

Precedence (high → low): `^` (right-assoc) → `* /` → `+ -`.

### Postfix evaluation

1. Scan left to right.
2. Operand: push.
3. Operator: pop `b`, pop `a`, push `a op b` (order matters: `a` is the second pop).
4. Final stack top is the answer.

Example: `2 3 4 * +` → push 2,3,4 → `3*4=12` → `2+12=14`.

### Infix → postfix

1. Operand: add to output.
2. `(`: push.
3. `)`: pop to output until `(`, discard `(`.
4. Operator: while stack top has precedence ≥ current (> for right-assoc `^`), pop to output; then push current.
5. End: pop everything to output.

Example: `A+B*C` → output `ABC*+`.

(For infix → prefix: reverse infix, swap brackets, convert, reverse result.)

## Ch3: Queue (FIFO)

Operations: `enqueue` (rear), `dequeue` (front), `isEmpty`, `isFull`. All O(1).

**ADT spec**: elements ordered by arrival; insert only at rear, delete only at front; `create`, `enqueue`, `dequeue`, `front`, `isEmpty`, `isFull`.

```c
#define MAX 100
int q[MAX], front = 0, rear = -1;

void enqueue(int x) { if (rear == MAX-1) return; q[++rear] = x; }
int  dequeue()      { if (front > rear) return -1; return q[front++]; }
int  isEmpty()      { return front > rear; }
```

Linear queue problem: freed front slots are never reused.

### Circular queue

Fixes wasted space using modulo.

```c
enqueue: if ((rear+1) % MAX == front) full;  rear = (rear+1) % MAX; q[rear] = x;
dequeue: if (empty) return;  x = q[front];  front = (front+1) % MAX;
```

Full: `(rear+1) % MAX == front`. Empty: `front == -1` (or keep a count; one slot is kept empty if using only front/rear).

### Priority queue

- **Ascending**: smallest element removed first.
- **Descending**: largest element removed first.
- Array version: insert in O(n) keeping order (or O(1) insert + O(n) delete scan). Heap version: O(log n) both.

## Ch4: Linked lists

```c
struct Node { int data; struct Node *next; };
```

### Singly linked list

```c
// insert at head — O(1)
n->next = head; head = n;

// insert at tail — O(n) (O(1) with tail pointer)
struct Node *t = head; while (t->next) t = t->next; t->next = n;

// insert after node p — O(1)
n->next = p->next; p->next = n;

// delete head — O(1)
tmp = head; head = head->next; free(tmp);

// delete node with key k — O(n)
prev = NULL; cur = head;
while (cur && cur->data != k) { prev = cur; cur = cur->next; }
if (cur) { if (prev) prev->next = cur->next; else head = cur->next; free(cur); }

// traverse
for (t = head; t; t = t->next) printf("%d ", t->data);
```

### Linked stack and queue

- **Stack**: push = insert at head, pop = delete at head. Both O(1), no overflow limit.
- **Queue**: keep `front` and `rear` pointers. Enqueue at rear (`rear->next = n; rear = n;`), dequeue at front. Both O(1). Handle the empty-queue case (`front = rear = n`).

### Doubly linked list

```c
struct DNode { int data; struct DNode *prev, *next; };
```

- Traverse both ways; delete a node in O(1) if you hold a pointer to it.
- Insert after p: `n->next = p->next; n->prev = p; if (p->next) p->next->prev = n; p->next = n;`
- Delete p: `p->prev->next = p->next; p->next->prev = p->prev; free(p);` (check NULLs at ends)
- Cost: extra pointer per node.

### Circular linked list

- Last node's `next` points back to head. Loop condition: `do { ... } while (t != head);`
- Good for round-robin scheduling. Keeping a pointer to the *tail* gives O(1) insert at both ends.

## Ch5: Trees

### Terminology

- **Root**: top node. **Leaf**: no children. **Father/parent**, **son/child**, **brother/sibling** (same parent).
- **Degree** of node: number of children. **Degree of tree**: max node degree.
- **Level**: root is level 0 (or 1, check your notes). **Depth** of node: edges from root. **Height** of node: edges to deepest leaf. **Height of tree**: height of root.
- **Ancestor / descendant**: nodes on path toward root / away from root.

### Binary tree types

- **Binary tree**: each node has ≤ 2 children (left, right).
- **Strictly (full) binary**: every node has 0 or 2 children. With n leaves → 2n−1 total nodes.
- **Complete binary**: all levels full, last level filled left → right... (in many textbooks, *complete* = all levels full; *almost complete* = last level filled left to right). Use your lecture's definition.
- **Almost complete**: all levels full except possibly the last, which is filled left to right (this is the heap shape).
- Max nodes at level L: 2ᴸ. Max nodes in height h: 2ʰ⁺¹ − 1. Min height with n nodes: ⌊log₂ n⌋.

```c
struct TNode { int data; struct TNode *left, *right; };
```

### Traversals (T = O(n))

```c
void preorder(struct TNode *r)  { if (!r) return; visit(r); preorder(r->left);  preorder(r->right); }
void inorder(struct TNode *r)   { if (!r) return; inorder(r->left);  visit(r);  inorder(r->right); }
void postorder(struct TNode *r) { if (!r) return; postorder(r->left); postorder(r->right); visit(r); }
```

| Order | Sequence | Notes |
| --- | --- | --- |
| Pre | Root, L, R | copy a tree; prefix expression |
| In | L, Root, R | **sorted output for a BST** |
| Post | L, R, Root | delete a tree; postfix expression |

To rebuild a tree you need in-order plus either pre- or post-order. Pre-order's first element and post-order's last element are the root.

### Binary Search Tree (BST)

Rule: left subtree < node < right subtree. Avg O(log n), worst O(n) (skewed).

```c
// search
struct TNode *search(struct TNode *r, int k) {
  if (!r || r->data == k) return r;
  return k < r->data ? search(r->left, k) : search(r->right, k);
}

// insert
struct TNode *insert(struct TNode *r, int k) {
  if (!r) { r = malloc(sizeof *r); r->data = k; r->left = r->right = NULL; return r; }
  if (k < r->data) r->left = insert(r->left, k);
  else if (k > r->data) r->right = insert(r->right, k);
  return r;
}
```

**Delete** (three cases):

1. **Leaf**: remove it.
2. **One child**: replace node with its child.
3. **Two children**: replace its value with the **in-order successor** (smallest in right subtree) or in-order predecessor (largest in left subtree), then delete that successor node.

## Ch6: Graphs

- G = (V, E). Directed or undirected, weighted or unweighted.
- Degree, path, cycle, connected, complete graph (max edges: n(n−1)/2 undirected).

### Representation

|  | Adjacency matrix | Adjacency list |
| --- | --- | --- |
| Space | O(V²) | O(V + E) |
| Edge check | O(1) | O(deg) |
| Iterate neighbours | O(V) | O(deg) |
| Best for | dense graphs | sparse graphs |

Matrix: `adj[i][j] = 1` (or weight) if edge i→j. Undirected → symmetric.

### DFS (stack / recursion) — O(V + E) with list, O(V²) with matrix

```c
void dfs(int v) {
  visited[v] = 1; printf("%d ", v);
  for (int u = 0; u < n; u++)
    if (adj[v][u] && !visited[u]) dfs(u);
}
```

### BFS (queue) — same complexity

```c
void bfs(int s) {
  enqueue(s); visited[s] = 1;
  while (!isEmpty()) {
    int v = dequeue(); printf("%d ", v);
    for (int u = 0; u < n; u++)
      if (adj[v][u] && !visited[u]) { visited[u] = 1; enqueue(u); }
  }
}
```

BFS finds shortest path in unweighted graphs. DFS goes deep first (stack); BFS goes level by level (queue).

### Minimum spanning tree (MST)

Spanning tree: connects all V vertices with V−1 edges, no cycle. MST: minimum total weight.

**Kruskal's** (edge-based, greedy)

1. Sort all edges by weight ascending.
2. Take the next edge; add it if it does **not form a cycle** (check with union-find).
3. Stop at V−1 edges.

- Time: O(E log E). Good for sparse graphs.

**Prim's** (vertex-based, greedy)

1. Start from any vertex.
2. Repeatedly pick the minimum-weight edge connecting the tree to a vertex **not yet in the tree**.
3. Stop when all vertices are included.

- Time: O(V²) with matrix; O(E log V) with heap. Good for dense graphs.

## Ch7: Sorting

| Algorithm | Best | Average | Worst | Space | Stable |
| --- | --- | --- | --- | --- | --- |
| Selection | n² | n² | n² | 1 | No |
| Insertion | **n** | n² | n² | 1 | Yes |
| Quick | n log n | n log n | **n²** | log n | No |
| Merge | n log n | n log n | n log n | **n** | Yes |
| Heap | n log n | n log n | n log n | 1 | No |

**Selection**: find the min of the unsorted part, swap into position. Always n²; at most n−1 swaps.

**Insertion**: take the next element, shift larger sorted elements right, drop it in. Great for nearly sorted data.

```c
for (i = 1; i < n; i++) {
  key = a[i]; j = i - 1;
  while (j >= 0 && a[j] > key) { a[j+1] = a[j]; j--; }
  a[j+1] = key;
}
```

**Quick**: pick a pivot, partition (smaller left, larger right), recurse on both sides. Worst case on already sorted input with first/last pivot.

```c
int partition(int a[], int lo, int hi) {
  int p = a[hi], i = lo - 1;
  for (int j = lo; j < hi; j++)
    if (a[j] <= p) { i++; swap(&a[i], &a[j]); }
  swap(&a[i+1], &a[hi]);
  return i + 1;
}
void quick(int a[], int lo, int hi) {
  if (lo < hi) { int p = partition(a, lo, hi); quick(a, lo, p-1); quick(a, p+1, hi); }
}
```

**Merge**: divide in half, sort each, merge two sorted halves with two pointers. Recurrence: T(n) = 2T(n/2) + n.

**Heap sort**: build a max-heap (O(n)), then repeatedly swap root with the last element, shrink the heap, heapify down (O(log n) each).

- Heap is an almost complete binary tree in an array. For index i (0-based): parent `(i-1)/2`, left `2i+1`, right `2i+2`.
- Max-heap: parent ≥ children.

## Ch8: Searching and hashing

### Sequential (linear) search

Check each element in turn. Works on unsorted data. Best O(1), average/worst **O(n)**.

### Binary search

Requires **sorted** array. Halve the range each step. Worst **O(log n)**.

```c
int lo = 0, hi = n - 1;
while (lo <= hi) {
  int mid = lo + (hi - lo) / 2;
  if (a[mid] == k) return mid;
  else if (a[mid] < k) lo = mid + 1;
  else hi = mid - 1;
}
return -1;
```

### Hashing

Map a key to a table index for \~O(1) average search/insert/delete.

**Hash functions**

- **Division**: `h(k) = k % m` (pick m prime, not a power of 2)
- **Mid-square**: square the key, take middle digits
- **Folding**: split the key into parts, add them, then mod m
- **Multiplication**: `h(k) = floor(m * frac(k * A))`, A ≈ 0.618

Good hash: fast, uniform spread, fewer collisions. **Load factor** α = n / m.

**Collision resolution**

| Method | How | Downside |
| --- | --- | --- |
| **Chaining** (open hashing) | Each slot holds a linked list of colliding keys | extra memory for pointers |
| **Linear probing** | Try `(h+1)%m, (h+2)%m, ...` | **primary clustering** |
| **Quadratic probing** | Try `(h+i²)%m` | secondary clustering |
| **Double hashing** | `(h1(k) + i·h2(k)) % m` | needs a second good hash |

Open addressing (probing) stores everything in the table itself, so n ≤ m. Deleting needs a "deleted" marker so probe chains don't break. Worst case search is O(n) if everything collides.

## Master complexity table

| Structure | Access | Search | Insert | Delete |
| --- | --- | --- | --- | --- |
| Array | O(1) | O(n) | O(n) | O(n) |
| Stack / Queue | — | O(n) | O(1) | O(1) |
| Singly linked list | O(n) | O(n) | O(1) at head | O(1) at head, O(n) by key |
| BST (avg / worst) | — | log n / n | log n / n | log n / n |
| Hash table (avg / worst) | — | 1 / n | 1 / n | 1 / n |
| Heap | — | n | log n | log n (root) |

## Last-minute reminders

- Always check **overflow/underflow** (stack) and **full/empty** (queue).
- Linked list: update pointers in the right order, and handle the **head / NULL** cases.
- BST in-order = sorted. Skewed BST = O(n).
- DFS = stack, BFS = queue.
- Kruskal sorts edges; Prim grows from a vertex.
- Quick sort worst case is n²; merge sort needs extra O(n) space; insertion sort best case is O(n).
- Binary search only on sorted data.
