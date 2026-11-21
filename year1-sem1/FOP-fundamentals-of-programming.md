# Fundamentals of Programming (WIX1002), Java

> Learning to give a computer precise, step-by-step instructions, using Java.

**Topics:** [Problem Solving](#1-problem-solving-pseudocode--flowcharts) · [Variables & Operators](#2-variables-operators-strings--io) · [Selection](#3-selection-if-switch) · [Loops](#4-loops) · [Arrays](#5-arrays-sorting--searching) · [Methods](#6-methods) · [File I/O](#7-file-io) · [Classes & Objects](#8-classes--objects) · [Inheritance](#9-inheritance) · [Polymorphism, Abstract & Interfaces](#10-polymorphism-abstract-classes--interfaces) · [Exceptions](#11-exception-handling)

---

## 1. Problem Solving, Pseudocode & Flowcharts

Before coding, **understand the problem** and break it into smaller pieces (**decomposition**).

> 🍰 **Analogy:** a recipe. Programming is writing a recipe so precise that a robot with zero common sense could follow it.

**IPO model:** every program has **I**nput → **P**rocess → **O**utput.
*Example: compute a circle's area.* Input: radius. Process: `area = π·r²`. Output: area.

**Pseudocode:** structured English, not real code.
```
START
  READ radius
  area ← 3.14159 × radius × radius
  PRINT area
END
```

**Flowchart symbols:** oval = start/end, parallelogram = input/output, rectangle = process, diamond = decision, arrows = flow.

### Your first Java program
```java
public class Hello {                       // class name must match the file name Hello.java
    public static void main(String[] args) { // program starts here
        System.out.println("Hello, World!");
    }
}
```
Java source (`.java`) → compiled by `javac` into bytecode (`.class`) → run by the **JVM**. That's why Java is "write once, run anywhere".

---

## 2. Variables, Operators, Strings & I/O

A **variable** is a named box that holds a value. In Java every box has a **type**.

> 📦 **Analogy:** labelled storage boxes. You can't pour soup (a `double`) into an egg carton (an `int`) without some loss.

| Type | Holds | Example |
|---|---|---|
| `int` | whole numbers | `int age = 19;` |
| `long` | big whole numbers | `long pop = 8_000_000_000L;` |
| `double` | decimals | `double gpa = 3.75;` |
| `char` | one character | `char grade = 'A';` |
| `boolean` | true/false | `boolean passed = true;` |
| `String` | text (an object) | `String name = "Aisyah";` |

`final double PI = 3.14159;` declares a **constant** (can't change).

### Operators
```java
int a = 7, b = 2;
a + b   // 9
a / b   // 3   ← integer division drops the decimal!
a % b   // 1   ← remainder (modulus)
7.0 / 2 // 3.5
a++     // use a, then add 1
++a     // add 1, then use a
a += 3  // a = a + 3
```

### Type casting
```java
double d = 9.7;
int i = (int) d;        // 9 (explicit narrowing, the decimal is chopped)
double x = 5;           // 5.0 (automatic widening)
int avg = (int) Math.round(sum / (double) count);
```

### Strings
```java
String s = "Hello";
s.length();          // 5
s.charAt(1);         // 'e'
s.toUpperCase();     // "HELLO"
s.substring(1, 3);   // "el"  (end index is exclusive)
s.indexOf('l');      // 2
s.equals("hello");   // false, use equals(), NOT ==, to compare strings
s.equalsIgnoreCase("hello"); // true
"Age: " + 19         // "Age: 19" (concatenation)
```

### Input and output
```java
import java.util.Scanner;
Scanner in = new Scanner(System.in);
System.out.print("Enter your name: ");
String name = in.nextLine();
int age = in.nextInt();
System.out.printf("%s is %d years old, GPA %.2f%n", name, age, 3.756);  // GPA 3.76
```
⚠️ After `nextInt()`, call `in.nextLine()` once to consume the leftover Enter before reading another line.

### Random numbers
```java
int dice = (int)(Math.random() * 6) + 1;   // 1–6
Random r = new Random();  int n = r.nextInt(10);  // 0–9
```
Comments: `// one line`, `/* many lines */`, `/** Javadoc */`.

---

## 3. Selection (if, switch)

**Relational operators:** `== != < > <= >=`
**Logical operators:** `&&` (and), `||` (or), `!` (not). `&&` and `||` **short-circuit**: they stop as soon as the answer is known.

```java
if (mark >= 80)       grade = 'A';
else if (mark >= 60)  grade = 'B';
else if (mark >= 40)  grade = 'C';
else                  grade = 'F';
```
> 🚦 **Analogy:** a multi-way if is like road signs at a junction. You take the **first** road whose condition matches and ignore the rest.

```java
switch (day) {
    case 1:  System.out.println("Mon"); break;
    case 2:  System.out.println("Tue"); break;
    default: System.out.println("Other");
}   // forget `break` → execution "falls through" into the next case!

String result = (mark >= 40) ? "Pass" : "Fail";   // ternary operator
```

**Common errors:** `if (x = 5)` (assignment instead of `==`), a stray semicolon `if (x > 0);`, comparing doubles with `==` (use `Math.abs(a-b) < 1e-9`).

---

## 4. Loops

| Loop | Checks condition | Use when |
|---|---|---|
| `while` | **before** each pass | you don't know how many times |
| `do-while` | **after** each pass (runs at least once) | menus, input validation |
| `for` | before each pass | you know the count |

```java
int i = 1;
while (i <= 5) { System.out.print(i + " "); i++; }

int n;
do { n = in.nextInt(); } while (n < 0);   // keep asking until non-negative

for (int k = 0; k < 5; k++) System.out.print(k);

int sum = 0;
for (int k = 1; k <= 100; k++) sum += k;   // 5050
```
- `break`: exit the loop now. `continue`: skip to the next iteration.
- **Labels** break out of nested loops: `outer: for(...) { for(...) { if (found) break outer; } }`

> 🏃 **Analogy:** `while` = "keep running laps *while* you have energy" (might do 0 laps). `do-while` = "run one lap, *then* check if you can do another".

**Common errors:** infinite loops (forgetting `i++`), off-by-one (`<=` vs `<`).

---

## 5. Arrays, Sorting & Searching

An **array** is a fixed-size row of boxes of the same type, indexed from **0**.

> 🏨 **Analogy:** a row of numbered lockers. Locker 0 is the first one.

```java
int[] marks = {70, 85, 90};
int[] a = new int[5];            // {0,0,0,0,0}
marks[1] = 88;                   // update an element
marks.length                     // 3 (no brackets for arrays)
for (int m : marks) System.out.println(m);   // enhanced for

int[][] grid = new int[3][4];    // 3 rows, 4 columns
grid[1][2] = 7;
```
⚠️ `marks[3]` → `ArrayIndexOutOfBoundsException`.

### Bubble sort
Repeatedly swap neighbours that are out of order. The biggest value "bubbles" to the end each pass.
```java
for (int i = 0; i < a.length - 1; i++)
    for (int j = 0; j < a.length - 1 - i; j++)
        if (a[j] > a[j+1]) { int t = a[j]; a[j] = a[j+1]; a[j+1] = t; }
```

### Linear search: check one by one, O(n)
```java
for (int i = 0; i < a.length; i++) if (a[i] == key) return i;
return -1;
```

### Binary search: halve the search space each step, O(log n), **array must be sorted**
```java
int lo = 0, hi = a.length - 1;
while (lo <= hi) {
    int mid = (lo + hi) / 2;
    if (a[mid] == key) return mid;
    else if (a[mid] < key) lo = mid + 1;
    else hi = mid - 1;
}
return -1;
```
> 📖 **Analogy:** linear search = reading a dictionary page by page. Binary search = opening it in the middle and deciding which half to keep.

---

## 6. Methods

A **method** is a named, reusable block of code.

> 🔧 **Analogy:** a kitchen appliance. Put ingredients in (parameters), it does its job, and gives something back (return value).

```java
public static double area(double r) {     // header: modifiers, return type, name, parameters
    return Math.PI * r * r;
}
double a = area(2.5);                     // call it

public static void greet(String name) {   // void = returns nothing
    System.out.println("Hi " + name);
}
```
- **static** methods belong to the class and are called without creating an object.
- **Pass-by-value:** Java copies the argument. For primitives, changes inside the method don't affect the caller. For **reference types** (arrays, objects) the *reference* is copied, so the method can change the object's contents.

```java
static void change(int x, int[] arr) { x = 99; arr[0] = 99; }
int n = 1; int[] a = {1};
change(n, a);   // n is still 1, but a[0] is now 99
```
> 🏠 **Analogy:** passing a primitive = giving someone a photocopy. Passing an array = giving someone your house address: they can go and rearrange your furniture.

---

## 7. File I/O

```java
import java.io.*;

// Write text
try (PrintWriter out = new PrintWriter(new FileWriter("data.txt"))) {
    out.println("Ali,85");
}

// Read text
try (Scanner sc = new Scanner(new File("data.txt"))) {
    while (sc.hasNextLine()) {
        String[] parts = sc.nextLine().split(",");
        System.out.println(parts[0] + " scored " + parts[1]);
    }
} catch (FileNotFoundException e) { System.out.println("No file!"); }

// Binary files: raw bytes, smaller and faster, not human-readable
try (DataOutputStream d = new DataOutputStream(new FileOutputStream("n.dat"))) { d.writeInt(42); }
try (DataInputStream  d = new DataInputStream(new FileInputStream("n.dat")))  { int x = d.readInt(); }

// File class: info about files and folders
File f = new File("data.txt");
f.exists(); f.length(); f.delete();
for (File g : new File("myFolder").listFiles()) System.out.println(g.getName());
```
> 📝 **Text vs binary:** a text file is like a handwritten letter (anyone can read it). A binary file is like a sealed USB drive (compact, but you need the right program to read it).

`try (...)` = **try-with-resources**: it closes the file automatically.

---

## 8. Classes & Objects

A **class** is a blueprint; an **object** is a thing built from that blueprint.

> 🏗️ **Analogy:** the class `Car` is the factory design. `myCar` and `yourCar` are actual cars with their own colour and mileage.

```java
public class Student {
    private String name;          // instance variables (each object has its own)
    private double gpa;
    private static int count = 0; // static: ONE shared copy for the whole class

    public Student(String name, double gpa) {   // constructor: runs on `new`
        this.name = name;                       // `this` = the current object
        this.gpa = gpa;
        count++;
    }
    public Student() { this("Unknown", 0.0); }  // overloaded constructor

    public String getName() { return name; }    // getter (accessor)
    public void setGpa(double g) {              // setter (mutator) with validation
        if (g >= 0 && g <= 4) gpa = g;
    }
    public static int getCount() { return count; }   // class method
    @Override public String toString() { return name + " (" + gpa + ")"; }
}

Student s = new Student("Mei", 3.8);
System.out.println(s);           // Mei (3.8)
```

- **Encapsulation:** keep fields `private` and expose them through methods. You control how data changes (e.g. no GPA of 5.0).
  > 💊 Like a capsule: the medicine is inside and you only interact with the outside.
- **Method overloading:** same name, different parameter lists (`add(int,int)`, `add(double,double)`).
- **Wrapper classes:** object versions of primitives (`Integer`, `Double`, `Character`…). Needed for collections like `ArrayList<Integer>`. Java converts automatically (**autoboxing/unboxing**).
- **Packages:** folders that group related classes (`package my.app;`, `import java.util.*;`).

---

## 9. Inheritance

A **subclass** inherits fields and methods from a **superclass** and can add or change behaviour. It's an "**is-a**" relationship.

> 🐾 **Analogy:** a `Dog` *is an* `Animal`. Every animal can `eat()`, and a dog also `bark()`s.

```java
class Animal {
    protected String name;          // protected: visible to subclasses (and the same package)
    Animal(String name) { this.name = name; }
    void speak() { System.out.println("..."); }
}
class Dog extends Animal {
    Dog(String name) { super(name); }          // call the parent constructor (must be first)
    @Override void speak() { System.out.println(name + ": Woof!"); }   // overriding
}
```

| Access | Same class | Same package | Subclass | Anywhere |
|---|---|---|---|---|
| `private` | ✅ | ❌ | ❌ | ❌ |
| (default/package) | ✅ | ✅ | ❌ | ❌ |
| `protected` | ✅ | ✅ | ✅ | ❌ |
| `public` | ✅ | ✅ | ✅ | ✅ |

- **Overriding** (same signature, in the subclass) ≠ **overloading** (same name, different parameters).
- Every class ultimately extends **`Object`**, which gives `toString()`, `equals()`, `hashCode()`.

---

## 10. Polymorphism, Abstract Classes & Interfaces

### Polymorphism ("many forms")
A superclass reference can point to a subclass object; the **actual object's** method runs (dynamic binding).
```java
Animal a = new Dog("Rex");
a.speak();     // "Rex: Woof!" (Dog's version)
Animal[] zoo = { new Dog("Rex"), new Cat("Tom") };
for (Animal x : zoo) x.speak();   // each speaks its own way
```
> 🎮 **Analogy:** pressing "attack" on a game controller. A knight swings a sword, a wizard casts a spell. Same button, different behaviour.

### Abstract classes
A class that can't be instantiated. It can have **abstract methods** (no body) that subclasses **must** implement.
```java
abstract class Shape {
    abstract double area();
    void describe() { System.out.println("Area = " + area()); }
}
class Circle extends Shape { double r; Circle(double r){this.r=r;} double area(){ return Math.PI*r*r; } }
```

### Interfaces
A pure "contract": a list of methods a class promises to have. A class can implement **many** interfaces (but extend only one class).
```java
interface Payable { double pay(); }
class Employee implements Payable { public double pay() { return 3000; } }
```
**Comparable** lets objects be sorted:
```java
class Student implements Comparable<Student> {
    double gpa;
    public int compareTo(Student o) { return Double.compare(gpa, o.gpa); }
}
Arrays.sort(students);   // now works!
```

| | Abstract class | Interface |
|---|---|---|
| Fields | any | only constants |
| Constructors | yes | no |
| Inheritance | `extends` one | `implements` many |
| Use for | shared code in an "is-a" family | a capability ("can-do") |

- **`clone()`**: makes a copy of an object (implement `Cloneable`). A shallow copy shares inner objects; a deep copy copies them too.
- **Inner class:** a class defined inside another class. It can access the outer class's private members.

---

## 11. Exception Handling

An **exception** is an error that happens while the program runs (dividing by zero, a missing file, bad input).

> 🪂 **Analogy:** `try` is doing a skydive, `catch` is your parachute, `finally` is packing up your gear whether the jump went well or not.

```java
try {
    int x = Integer.parseInt(input);
    System.out.println(10 / x);
} catch (NumberFormatException e) {
    System.out.println("Not a number!");
} catch (ArithmeticException e) {
    System.out.println("Can't divide by zero!");
} finally {
    System.out.println("Always runs");
}
```
- **Checked exceptions** (e.g. `IOException`): the compiler forces you to handle or declare them.
- **Unchecked exceptions** (`RuntimeException`, e.g. `NullPointerException`, `ArrayIndexOutOfBoundsException`): usually bugs.

**Throwing and declaring:**
```java
static void withdraw(double amt) throws InsufficientFundsException {
    if (amt > balance) throw new InsufficientFundsException("Not enough money");
    balance -= amt;
}
class InsufficientFundsException extends Exception {
    InsufficientFundsException(String msg) { super(msg); }
}
```

---

### ✅ Quick self-check
1. What does `7 / 2` print in Java? *(3)*
2. Why use `.equals()` for strings? *(`==` compares references, not contents)*
3. Which loop always runs at least once? *(do-while)*
4. What must be true before using binary search? *(the array is sorted)*
5. Difference between overloading and overriding? *(different parameters in the same class vs the same signature in a subclass)*
