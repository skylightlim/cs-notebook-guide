# Data Structures (WIA1002), Java

> Ways of organising data so programs can store, find, and change it efficiently.

**Topics:** [OOP Revision](#1-oop-revision) · [Generics](#2-generics) · [ADT & Bag](#3-abstract-data-types-adt--bag) · [Linked List](#4-linked-list) · [Stack](#5-stack) · [Queue & Priority Queue](#6-queue--priority-queue) · [Graph](#7-graph) · [Recursion](#8-recursion) · [Searching & Sorting](#9-searching--sorting) · [Binary Search Tree](#10-binary-search-tree-bst) · [Big-O Cheat Sheet](#big-o-cheat-sheet)

---

## 1. OOP Revision

Quick recap (full detail in [FOP](../year1-sem1/FOP-fundamentals-of-programming.md#8-classes--objects)):

- **Class** = blueprint, **object** = instance. A **UML class diagram** shows name / fields / methods, with `-` for private and `+` for public.
- **Constructor** runs on `new`. If you write none, Java gives you a **default no-arg constructor**.
- **Reference variables** hold the *address* of an object; primitive variables hold the value itself.
  ```java
  Circle c1 = new Circle(5);
  Circle c2 = c1;   // c2 points to the SAME object, not a copy
  c2.radius = 10;   // c1.radius is now 10 too
  ```
- **Encapsulation:** `private` fields + public getters/setters.
- **Inheritance:** `class Student extends Person`. The superclass constructor **always runs first** (an implicit `super()` call).
- **Polymorphism:** `Person p = new Student();` calls Student's overridden methods (dynamic binding).
- **Overriding** = same signature in a subclass; **overloading** = same name, different parameters.

---

## 2. Generics

**Generics** let you write a class or method once and use it with any type, **type-safely**.

> 📦 **Analogy:** a labelled storage box. `Box<Book>` only accepts books, so you never reach in expecting a book and pull out a shoe.

```java
// Without generics: casting needed, errors only at runtime
ArrayList list = new ArrayList();
list.add("hello"); list.add(42);
String s = (String) list.get(1);   // 💥 ClassCastException at runtime

// With generics: the compiler catches mistakes
ArrayList<String> names = new ArrayList<>();
names.add("Ali");
// names.add(42);   ← compile error ✅
String n = names.get(0);           // no cast needed
```

**Array vs ArrayList:** an array has a fixed size and holds primitives or objects; an `ArrayList` grows automatically and holds objects only (`ArrayList<Integer>`, not `<int>`).

**Naming convention:** `E` element, `T` type, `K` key, `V` value, `N` number.

### Generic class and generic method
```java
public class GenericBox<T> {
    private T item;
    public void set(T item) { this.item = item; }
    public T get() { return item; }
}
GenericBox<Integer> b = new GenericBox<>();  b.set(5);

public static <E> void printArray(E[] arr) {
    for (E e : arr) System.out.print(e + " ");
}
```

### Bounded types and wildcards
```java
// Bounded: T must be a Number (or a subclass)
public static <T extends Number> double sum(T a, T b) { return a.doubleValue() + b.doubleValue(); }

// T must be comparable to itself, so we can find a max
public static <T extends Comparable<T>> T max(T a, T b) { return a.compareTo(b) > 0 ? a : b; }

// Wildcards
void printAll(List<?> list)                  // any type (unbounded)
double total(List<? extends Number> list)    // Number or a subclass (read from it)
void addInts(List<? super Integer> list)     // Integer or a superclass (write to it)
```
**Raw types** (`ArrayList` without `<>`) exist only for backward compatibility and are unsafe. Avoid them.

### Type erasure and restrictions
Generic info is checked at compile time, then **erased** (replaced by `Object` or the bound). So:
1. ❌ `new E()`: the runtime doesn't know what E is
2. ❌ `new E[10]`: use `(E[]) new Object[10]` instead
3. ❌ a generic type parameter in a static context
4. ❌ generic exception classes

---

## 3. Abstract Data Types (ADT) & Bag

An **ADT** describes **what** a data collection does (its operations), not **how** it's implemented.
A **data structure** is the concrete implementation (array, linked list…).

> 🥤 **Analogy:** a vending machine. You know the interface (insert coin, press button, get drink) without knowing the internal mechanics. You could swap the machinery inside and you'd still use it the same way.

**Advantages:** you can change the implementation without changing the code that uses it; it's easier to reason about; reusable.

### The Bag ADT
A **bag** is an unordered collection that allows **duplicates**.
> 🛍️ Like a shopping bag: you can put in 3 apples; order doesn't matter.

```java
public interface BagInterface<T> {
    int getCurrentSize();
    boolean isEmpty();
    boolean add(T newEntry);
    T remove();                  // removes an unspecified entry
    boolean remove(T anEntry);
    void clear();
    int getFrequencyOf(T anEntry);
    boolean contains(T anEntry);
    T[] toArray();
}
```
Design tools: **CRC cards** (Class, Responsibilities, Collaborations) and **UML** notation.

**Two implementations:** `ArrayBag` (fixed-size array, fast access, may fill up) and `LinkedBag` (chain of nodes, grows freely).

**Set** = like a bag but **no duplicates** (`java.util.Set`).

---

## 4. Linked List

A **linked list** is a chain of **nodes**; each node holds data and a reference to the next node.

> 🚂 **Analogy:** a train. Each carriage (node) carries passengers (data) and is coupled to the next carriage. To reach carriage 5, walk through 1–4.

```
head → [10|•] → [20|•] → [30|null] ← tail
```

### Array vs linked list
| | Array / ArrayList | Linked list |
|---|---|---|
| Access element i | **O(1)** | O(n), must walk |
| Insert/delete at front | O(n), shift everything | **O(1)** |
| Insert/delete at end | O(1) amortised | O(1) with a tail pointer (delete-last is O(n) for singly) |
| Memory | contiguous; may waste capacity | extra space for pointers |

### Implementation
```java
public class MyLinkedList<E> {
    private static class Node<E> { E element; Node<E> next; Node(E e) { element = e; } }
    private Node<E> head, tail;
    private int size;

    public void addFirst(E e) {
        Node<E> n = new Node<>(e);
        n.next = head; head = n; size++;
        if (tail == null) tail = head;
    }
    public void addLast(E e) {
        Node<E> n = new Node<>(e);
        if (tail == null) head = tail = n;
        else { tail.next = n; tail = n; }
        size++;
    }
    public void add(int index, E e) {
        if (index == 0) addFirst(e);
        else if (index >= size) addLast(e);
        else {
            Node<E> cur = head;
            for (int i = 1; i < index; i++) cur = cur.next;   // stop just before the position
            Node<E> n = new Node<>(e);
            n.next = cur.next; cur.next = n; size++;
        }
    }
    public E removeFirst() {
        if (size == 0) return null;
        E e = head.element; head = head.next; size--;
        if (head == null) tail = null;
        return e;
    }
    public E removeLast() {
        if (size <= 1) return removeFirst();
        Node<E> cur = head;
        while (cur.next != tail) cur = cur.next;   // find the second-last
        E e = tail.element; tail = cur; tail.next = null; size--;
        return e;
    }
    public void print() {                           // traversal
        for (Node<E> c = head; c != null; c = c.next) System.out.print(c.element + " ");
    }
}
```

### Variations
| Type | Description |
|---|---|
| **Singly** | each node → next |
| **Circular** | last node → first (playlists on repeat 🔁) |
| **Doubly** | each node has `next` **and** `prev`; traverse both ways (browser back/forward) |
| **Circular doubly** | both of the above |

**Doubly linked list insertion in the middle:**
```java
Node<E> n = new Node<>(e);
n.prev = cur; n.next = cur.next;
cur.next.prev = n; cur.next = n;
```
Downside: more memory per node and more pointers to update.

---

## 5. Stack

A **stack** is **LIFO**: **L**ast **I**n, **F**irst **O**ut.

> 🍽️ **Analogy:** a stack of plates. You add (push) and remove (pop) only at the top.

| Operation | Meaning |
|---|---|
| `push(e)` | add to the top |
| `pop()` | remove and return the top |
| `peek()` | look at the top without removing |
| `isEmpty()`, `getSize()` | |

**Uses:** undo/redo, browser back button, method-call stack, matching brackets, expression evaluation, DFS.

```java
public class GenericStack<E> {
    private ArrayList<E> list = new ArrayList<>();
    public void push(E e) { list.add(e); }
    public E pop()  { return list.remove(list.size() - 1); }
    public E peek() { return list.get(list.size() - 1); }
    public boolean isEmpty() { return list.isEmpty(); }
    public int getSize() { return list.size(); }
}
```

### Postfix evaluation
**Infix:** `(3 + 4) * 5` → **Postfix:** `3 4 + 5 *` (no brackets needed!)

Algorithm: scan left to right. Number → push. Operator → pop two, compute, push the result.
```
3 4 + 5 *
push 3        [3]
push 4        [3, 4]
+ → 3+4       [7]
push 5        [7, 5]
* → 7*5       [35]   ✅ answer
```
Errors: **too many operators** (stack empty when popping) or **too many operands** (more than one value left at the end).

```java
int evalPostfix(String expr) {
    Stack<Integer> s = new Stack<>();
    for (String t : expr.split(" ")) {
        if (t.matches("-?\\d+")) s.push(Integer.parseInt(t));
        else { int b = s.pop(), a = s.pop();
               s.push(switch (t) { case "+" -> a + b; case "-" -> a - b; case "*" -> a * b; default -> a / b; }); }
    }
    return s.pop();
}
```

---

## 6. Queue & Priority Queue

A **queue** is **FIFO**: **F**irst **I**n, **F**irst **O**ut.

> 🧍‍♂️🧍‍♀️ **Analogy:** a line at the bank. Join at the back (enqueue), leave from the front (dequeue).

| Operation | Meaning |
|---|---|
| `enqueue(e)` | add at the back |
| `dequeue()` | remove from the front |
| `peek()` | look at the front |

**Uses:** print queues, CPU scheduling, BFS, buffering (keyboard, network packets).

```java
public class GenericQueue<E> {
    private LinkedList<E> list = new LinkedList<>();   // O(1) at both ends
    public void enqueue(E e) { list.addLast(e); }
    public E dequeue() { return list.removeFirst(); }
    public E peek() { return list.getFirst(); }
    public boolean isEmpty() { return list.isEmpty(); }
}
```
(Java's built-in: `Queue<E> q = new LinkedList<>(); q.offer(x); q.poll(); q.peek();`)

### Priority queue
Elements leave in **priority order**, not arrival order. Java's `PriorityQueue` is a **min-heap** by default (the smallest comes out first).

> 🏥 **Analogy:** a hospital emergency room. The most urgent patient is treated first, regardless of arrival time.

```java
PriorityQueue<Integer> pq = new PriorityQueue<>();
pq.offer(5); pq.offer(1); pq.offer(3);
pq.poll();   // 1

PriorityQueue<String> maxQ = new PriorityQueue<>(Collections.reverseOrder());
PriorityQueue<Patient> er = new PriorityQueue<>((a, b) -> b.urgency - a.urgency);
```

---

## 7. Graph

(Theory in [CM: Graph Theory](../year1-sem1/CM-computing-mathematics.md#9-graph-theory).) Here we **implement** graphs.

> ✈️ **Example:** cities as vertices, flights as weighted edges (distance or price).

### Representing edges
| Representation | Space | Check edge (u,v) | List neighbours | Best for |
|---|---|---|---|---|
| **Adjacency matrix** `int[n][n]` | O(V²) | O(1) | O(V) | dense graphs |
| **Adjacency list** | O(V + E) | O(degree) | O(degree) | sparse graphs (most real ones) |

### A simple weighted, directed graph using linked lists
```java
class Edge<T> { Vertex<T> to; int weight; Edge<T> next; }
class Vertex<T> { T info; int inDeg, outDeg; Edge<T> firstEdge; Vertex<T> next; }

public class Graph<T extends Comparable<T>> {
    Vertex<T> head; int size;

    public boolean addVertex(T v) {
        if (hasVertex(v)) return false;
        Vertex<T> nv = new Vertex<>(); nv.info = v;
        if (head == null) head = nv;
        else { Vertex<T> t = head; while (t.next != null) t = t.next; t.next = nv; }
        size++; return true;
    }
    public boolean hasVertex(T v) {
        for (Vertex<T> t = head; t != null; t = t.next) if (t.info.compareTo(v) == 0) return true;
        return false;
    }
    public boolean addEdge(T from, T to, int w) {
        Vertex<T> s = find(from), d = find(to);
        if (s == null || d == null) return false;
        Edge<T> e = new Edge<>(); e.to = d; e.weight = w;
        e.next = s.firstEdge; s.firstEdge = e;       // insert at the front of the edge list
        s.outDeg++; d.inDeg++; return true;
    }
    public List<T> getNeighbours(T v) {
        List<T> out = new ArrayList<>();
        for (Edge<T> e = find(v).firstEdge; e != null; e = e.next) out.add(e.to.info);
        return out;
    }
    private Vertex<T> find(T v) {
        for (Vertex<T> t = head; t != null; t = t.next) if (t.info.compareTo(v) == 0) return t;
        return null;
    }
}
```

### Traversals
- **DFS** (stack/recursion): go deep, then backtrack. Uses: detecting cycles, finding paths, connected components, maze solving.
- **BFS** (queue): level by level. Uses: shortest path in an unweighted graph, "degrees of separation" in social networks.

(See the BFS/DFS code in [CM: Trees](../year1-sem1/CM-computing-mathematics.md#bfs-vs-dfs).)

---

## 8. Recursion

A **recursive method** calls itself to solve a smaller version of the same problem.

Every recursion needs:
1. **Base case:** when to stop (no recursive call).
2. **Recursive case:** call itself on a **smaller** problem that moves toward the base case.

> 🪆 **Analogy:** Russian nesting dolls. Open a doll, find a smaller doll, repeat until you reach the tiniest solid one (the base case).

```java
long factorial(int n) {
    if (n == 0) return 1;              // base case
    return n * factorial(n - 1);       // recursive case
}
```
**Trace of factorial(4)** (each call waits on the call stack):
```
factorial(4) = 4 * factorial(3)
                   3 * factorial(2)
                       2 * factorial(1)
                           1 * factorial(0) = 1
                       = 2
                   = 6
             = 24
```

**Fibonacci:** `fib(0)=0, fib(1)=1, fib(n)=fib(n-1)+fib(n-2)`
```java
long fib(int n) { return n < 2 ? n : fib(n - 1) + fib(n - 2); }   // O(2ⁿ), very slow for big n!
```
⚠️ **No base case (or one that's never reached)** → infinite recursion → `StackOverflowError`.

### Recursion vs iteration
| | Recursion | Iteration |
|---|---|---|
| Code | often shorter and more elegant | sometimes longer |
| Memory | each call uses stack space | constant |
| Speed | function-call overhead | usually faster |
| Natural for | trees, graphs, divide-and-conquer, directories | simple repetition |

**Thinking recursively:** "If I already had a solution for the smaller problem, how would I use it?"
```java
long directorySize(File f) {                      // naturally recursive!
    if (f.isFile()) return f.length();
    long total = 0;
    for (File sub : f.listFiles()) total += directorySize(sub);
    return total;
}
boolean isPalindrome(String s) {
    if (s.length() <= 1) return true;
    return s.charAt(0) == s.charAt(s.length()-1) && isPalindrome(s.substring(1, s.length()-1));
}
```

---

## 9. Searching & Sorting

### Searching
| Algorithm | Requirement | Time |
|---|---|---|
| **Linear search** | none | O(n) |
| **Binary search** | **sorted** array | O(log n) |

(Code in [FOP: Arrays](../year1-sem1/FOP-fundamentals-of-programming.md#5-arrays-sorting--searching).) Binary search on 1,000,000 items takes at most ~20 comparisons.

### Sorting algorithms
**Selection sort:** find the smallest remaining element and swap it to the front.
> 🃏 Like picking the lowest card from your hand each time and placing it in order on the table.
```java
for (int i = 0; i < a.length - 1; i++) {
    int min = i;
    for (int j = i + 1; j < a.length; j++) if (a[j] < a[min]) min = j;
    int t = a[i]; a[i] = a[min]; a[min] = t;
}
```

**Insertion sort:** take the next element and insert it into the correct place in the sorted part on the left.
> 🃏 Like sorting cards in your hand as you're dealt them.
```java
for (int i = 1; i < a.length; i++) {
    int cur = a[i], k = i - 1;
    while (k >= 0 && a[k] > cur) { a[k + 1] = a[k]; k--; }
    a[k + 1] = cur;
}
```

**Bubble sort:** swap adjacent out-of-order pairs; stop early if a pass makes no swaps (`needNextPass`).
```java
boolean needNextPass = true;
for (int k = 1; k < a.length && needNextPass; k++) {
    needNextPass = false;
    for (int i = 0; i < a.length - k; i++)
        if (a[i] > a[i+1]) { int t = a[i]; a[i] = a[i+1]; a[i+1] = t; needNextPass = true; }
}
```

**Merge sort:** divide the array in half, sort each half recursively, then **merge** the two sorted halves.
> 📚 Like two people each sorting half of a stack of exam papers, then merging their piles by repeatedly taking the smaller top paper.
```java
void mergeSort(int[] a) {
    if (a.length < 2) return;
    int[] left = Arrays.copyOfRange(a, 0, a.length / 2);
    int[] right = Arrays.copyOfRange(a, a.length / 2, a.length);
    mergeSort(left); mergeSort(right);
    int i = 0, j = 0, k = 0;
    while (i < left.length && j < right.length) a[k++] = (left[i] <= right[j]) ? left[i++] : right[j++];
    while (i < left.length) a[k++] = left[i++];
    while (j < right.length) a[k++] = right[j++];
}
```
```
[2 9 5 4 8 1 6 7]
[2 9 5 4] [8 1 6 7]
[2 9][5 4] [8 1][6 7]
[2 9][4 5] [1 8][6 7]
[2 4 5 9]  [1 6 7 8]
[1 2 4 5 6 7 8 9]
```

**Generic sort:** make it work for any `Comparable` type:
```java
public static <E extends Comparable<E>> void selectionSort(E[] list) { /* use list[j].compareTo(list[min]) < 0 */ }
```

| Algorithm | Best | Average | Worst | Notes |
|---|---|---|---|---|
| Selection | O(n²) | O(n²) | O(n²) | few swaps |
| Insertion | **O(n)** | O(n²) | O(n²) | great for nearly sorted data |
| Bubble | O(n) (with flag) | O(n²) | O(n²) | simple, slow |
| Merge | O(n log n) | O(n log n) | O(n log n) | needs O(n) extra space; stable |

---

## 10. Binary Search Tree (BST)

A **binary tree**: each node has at most 2 children (left, right).
A **BST**: for every node, **left subtree < node < right subtree**.

```
        60
       /  \
     55    100
    /  \   /  \
  45   57 67  107
```
> 📖 **Analogy:** a "guess my number" game. At each node: "Is it smaller or bigger?" and you go left or right, discarding half the tree.

**Terms:** root, leaf, parent, child, sibling, subtree, depth (distance from the root), height (longest root-to-leaf path), level.

### Search and insert
```java
class TreeNode<E> { E element; TreeNode<E> left, right; TreeNode(E e) { element = e; } }

public boolean search(E e) {
    TreeNode<E> cur = root;
    while (cur != null) {
        int c = e.compareTo(cur.element);
        if (c < 0) cur = cur.left; else if (c > 0) cur = cur.right; else return true;
    }
    return false;
}
public boolean insert(E e) {
    if (root == null) { root = new TreeNode<>(e); return true; }
    TreeNode<E> parent = null, cur = root;
    while (cur != null) {
        parent = cur;
        int c = e.compareTo(cur.element);
        if (c < 0) cur = cur.left; else if (c > 0) cur = cur.right; else return false; // no duplicates
    }
    if (e.compareTo(parent.element) < 0) parent.left = new TreeNode<>(e);
    else parent.right = new TreeNode<>(e);
    return true;
}
```

### Traversals
| Traversal | Order | Result for the tree above |
|---|---|---|
| **Inorder** | Left, **Node**, Right | 45 55 57 60 67 100 107 ← **sorted!** |
| **Preorder** | **Node**, Left, Right | 60 55 45 57 100 67 107 (copying a tree) |
| **Postorder** | Left, Right, **Node** | 45 57 55 67 107 100 60 (deleting a tree) |
| **Breadth-first** | level by level | 60 55 100 45 57 67 107 |

💡 *Memory trick:* the name says where the **node** goes: **pre** = before, **in** = in between, **post** = after.

```java
void inorder(TreeNode<E> n) { if (n == null) return; inorder(n.left); System.out.print(n.element + " "); inorder(n.right); }
```

### Deleting a node: three cases
1. **Leaf:** just remove it.
2. **One child:** connect the parent directly to that child.
3. **Two children:** replace the node's value with the **largest value in its left subtree** (rightmost node of the left subtree, its in-order predecessor), then delete that node. (Using the smallest in the right subtree also works.)

**Efficiency:** search/insert/delete are O(h), where h = height. Balanced tree → O(log n). Degenerate (inserting sorted data makes it a linked list) → O(n).

---

## Big-O cheat sheet

**Big-O** describes how running time grows as input size n grows.

| Big-O | Name | Example |
|---|---|---|
| O(1) | constant | array index, stack push/pop |
| O(log n) | logarithmic | binary search, balanced BST |
| O(n) | linear | linear search, traversing a list |
| O(n log n) | linearithmic | merge sort |
| O(n²) | quadratic | bubble/selection/insertion sort, nested loops |
| O(2ⁿ) | exponential | naive recursive Fibonacci |

| Structure | Access | Search | Insert | Delete |
|---|---|---|---|---|
| Array | O(1) | O(n) | O(n) | O(n) |
| Linked list | O(n) | O(n) | O(1) at head | O(1) at head |
| Stack / Queue | n/a | n/a | O(1) | O(1) |
| BST (balanced) | O(log n) | O(log n) | O(log n) | O(log n) |
| Priority queue (heap) | O(1) peek | O(n) | O(log n) | O(log n) |

---

### ✅ Quick self-check
1. Stack or queue for "undo"? *(stack)*
2. Evaluate postfix `5 1 2 + 4 * + 3 -`. *(14)*
3. Which traversal of a BST prints sorted order? *(inorder)*
4. What happens if recursion has no base case? *(StackOverflowError)*
5. Why can't you write `new E()` in a generic class? *(type erasure)*
