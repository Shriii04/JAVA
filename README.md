# Java Memory Management

---

## 📚 Table of Contents

1. [Introduction to Memory in Java](#introduction)
2. [Types of Memory: Stack vs Heap](#stack-vs-heap)
3. [Pass by Value in Java](#pass-by-value)
4. [The Final Keyword](#final-keyword)
5. [Escaping References](#escaping-references)
6. [Summary & Best Practices](#summary)

---

## 🎯 Introduction to Memory in Java 

### What is Memory Management?

Think of computer memory like a **warehouse** where your program stores its stuff:
- **Variables** (boxes containing data)
- **Objects** (complex items with multiple compartments)
- **Method calls** (temporary workspaces)

In Java, the **JVM (Java Virtual Machine)** automatically manages memory for you through a process called **Garbage Collection** - like having an automatic cleanup crew!

### Why Should You Care?

Understanding memory helps you:
- ✅ Write more efficient code
- ✅ Avoid memory leaks
- ✅ Debug mysterious bugs
- ✅ Build scalable applications

---

## 🏗️ Types of Memory: Stack vs Heap {#stack-vs-heap}

Java divides memory into two main areas:

| Aspect | Stack Memory | Heap Memory |
|--------|-------------|-------------|
| **What's stored** | Method calls, local variables, references | Objects, instance variables |
| **Speed** | Very fast | Slower |
| **Size** | Small (per thread) | Large (shared) |
| **Lifecycle** | Automatic (method scope) | Managed by Garbage Collector |
| **Access** | LIFO (Last In, First Out) | Random access |

### 📦 Stack Memory - The Quick Access Shelf

**Analogy:** Think of Stack like a **stack of plates** - you can only add or remove from the top!

```java
public class StackExample {
    
    public static void main(String[] args) {
        // 'args' reference goes on Stack
        // 'args' array object goes on Heap
        
        int age = 25;           // Primitive 'age' stored in Stack
        double salary = 50000;  // Primitive 'salary' stored in Stack
        
        calculateBonus(salary); // New stack frame created
        
        // When method returns, its stack frame is removed
        System.out.println("Age: " + age);
    }
    
    public static double calculateBonus(double baseSalary) {
        // 'baseSalary' is a COPY on this method's stack frame
        double bonus = baseSalary * 0.10; // 'bonus' on Stack
        return bonus;
        // When this method ends, 'baseSalary' and 'bonus' are removed
    }
}
```

**Visual Representation:**
```
┌─────────────────────────────────────┐
│           STACK MEMORY              │
├─────────────────────────────────────┤
│  calculateBonus() frame:            │  ← TOP (Current)
│    - baseSalary: 50000.0            │
│    - bonus: 5000.0                  │
├─────────────────────────────────────┤
│  main() frame:                      │
│    - args: [reference to Heap]      │
│    - age: 25                        │
│    - salary: 50000.0                │
└─────────────────────────────────────┘
```

### 🏢 Heap Memory - The Big Warehouse

**Analogy:** Think of Heap like a **warehouse** where you store big items with labels (references)!

```java
public class HeapExample {
    
    public static void main(String[] args) {
        
        // Object created in HEAP
        // Reference 'person1' stored in STACK
        Person person1 = new Person("Alice", 30);
        
        // Another object in HEAP
        Person person2 = new Person("Bob", 25);
        
        // String objects are also in HEAP
        String greeting = "Hello, World!";
        
        // Arrays are objects - stored in HEAP
        int[] numbers = {1, 2, 3, 4, 5};
        
        System.out.println(person1.getName());
    }
}

class Person {
    // Instance variables stored with object in HEAP
    private String name;
    private int age;
    
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    public String getName() {
        return name;
    }
}
```

**Visual Representation:**
```
┌──────────────────┐         ┌─────────────────────────────────────┐
│   STACK MEMORY   │         │           HEAP MEMORY               │
├──────────────────┤         ├─────────────────────────────────────┤
│ main() frame:    │         │                                     │
│                  │         │  ┌─────────────────────┐            │
│ person1: 0x100 ──┼────────►│  │ Person Object       │            │
│                  │         │  │ name: "Alice"       │            │
│ person2: 0x200 ──┼────┐    │  │ age: 30             │            │
│                  │    │    │  └─────────────────────┘            │
│ greeting: 0x300 ─┼──┐ │    │                                     │
│                  │  │ │    │  ┌─────────────────────┐            │
│ numbers: 0x400 ──┼┐ │ └───►│  │ Person Object       │            │
│                  ││ │      │  │ name: "Bob"         │            │
└──────────────────┘│ │      │  │ age: 25             │            │
                    │ │      │  └─────────────────────┘            │
                    │ │      │                                     │
                    │ └─────►│  ┌─────────────────────┐            │
                    │        │  │ String: "Hello..."  │            │
                    │        │  └─────────────────────┘            │
                    │        │                                     │
                    └───────►│  ┌─────────────────────┐            │
                             │  │ int[]: {1,2,3,4,5}  │            │
                             │  └─────────────────────┘            │
                             └─────────────────────────────────────┘
```

### 🔑 Key Takeaways

```java
public class MemoryDemo {
    public static void main(String[] args) {
        
        // PRIMITIVES: Value stored directly in Stack
        int x = 10;        // Stack: x = 10
        int y = x;         // Stack: y = 10 (copy of value)
        y = 20;            // Stack: y = 20 (x is still 10!)
        
        System.out.println("x = " + x);  // Output: x = 10
        System.out.println("y = " + y);  // Output: y = 20
        
        // OBJECTS: Reference in Stack, Object in Heap
        StringBuilder sb1 = new StringBuilder("Hello");
        StringBuilder sb2 = sb1;  // sb2 points to SAME object!
        
        sb2.append(" World");     // Modifies the shared object
        
        System.out.println("sb1 = " + sb1);  // Output: sb1 = Hello World
        System.out.println("sb2 = " + sb2);  // Output: sb2 = Hello World
    }
}
```

---

## 🔄 Passing Value in Java 

### The Big Myth: "Java has Pass by Reference"

**❌ WRONG!** Java is **ALWAYS Pass by Value**

But wait... it gets confusing because:
- For **primitives**: The value itself is copied
- For **objects**: The reference (memory address) is copied

### Example 1: Primitives - Pure Pass by Value

```java
public class PrimitivePassByValue {
    
    public static void main(String[] args) {
        int number = 10;
        
        System.out.println("Before: " + number);  // Output: 10
        
        modifyPrimitive(number);
        
        System.out.println("After: " + number);   // Output: 10 (unchanged!)
    }
    
    public static void modifyPrimitive(int num) {
        // 'num' is a COPY of 'number'
        num = 100;  // Only the copy is changed
        System.out.println("Inside method: " + num);  // Output: 100
    }
}
```

**What happens in memory:**
```
BEFORE method call:
┌─────────────────────┐
│ main():             │
│   number = 10       │
└─────────────────────┘

DURING method call:
┌─────────────────────┐
│ modifyPrimitive():  │
│   num = 10 → 100    │  ← Only this copy changes
├─────────────────────┤
│ main():             │
│   number = 10       │  ← Original unchanged
└─────────────────────┘

AFTER method returns:
┌─────────────────────┐
│ main():             │
│   number = 10       │  ← Still 10!
└─────────────────────┘
```

### Example 2: Objects - Pass by Value of Reference

```java
public class ObjectPassByValue {
    
    public static void main(String[] args) {
        Person person = new Person("Alice");
        
        System.out.println("Before: " + person.getName());  // Alice
        
        modifyObject(person);
        
        System.out.println("After: " + person.getName());   // Bob (changed!)
    }
    
    public static void modifyObject(Person p) {
        // 'p' is a COPY of the reference, but points to SAME object
        p.setName("Bob");  // Modifies the actual object in Heap
    }
}

class Person {
    private String name;
    
    public Person(String name) { this.name = name; }
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
}
```

**What happens in memory:**
```
┌─────────────────────┐      ┌─────────────────────┐
│ STACK               │      │ HEAP                │
├─────────────────────┤      ├─────────────────────┤
│ modifyObject():     │      │                     │
│   p: 0x100 ─────────┼──┐   │ ┌─────────────────┐ │
├─────────────────────┤  │   │ │ Person Object   │ │
│ main():             │  └──►│ │ name: "Alice"   │ │
│   person: 0x100 ────┼─────►│ │   → "Bob"       │ │
└─────────────────────┘      │ └─────────────────┘ │
                             └─────────────────────┘

Both 'person' and 'p' point to the SAME object!
Changes through either reference affect the same object.
```

### Example 3: Reassigning References

```java
public class ReferenceReassignment {
    
    public static void main(String[] args) {
        Person person = new Person("Alice");
        
        System.out.println("Before: " + person.getName());  // Alice
        
        tryToReplacePerson(person);
        
        System.out.println("After: " + person.getName());   // Alice (unchanged!)
    }
    
    public static void tryToReplacePerson(Person p) {
        // This creates a NEW object and assigns to local copy 'p'
        p = new Person("Charlie");  // Only local 'p' points to new object
        System.out.println("Inside: " + p.getName());  // Charlie
    }
}
```

**What happens:**
```
DURING method call:
┌─────────────────────┐      ┌─────────────────────────┐
│ STACK               │      │ HEAP                    │
├─────────────────────┤      ├─────────────────────────┤
│ tryToReplacePerson():│     │ ┌───────────────────┐   │
│   p: 0x200 ─────────┼─────►│ │ NEW Person Object │   │
│   (was 0x100,       │      │ │ name: "Charlie"   │   │
│    now reassigned)  │      │ └───────────────────┘   │
├─────────────────────┤      │                         │
│ main():             │      │ ┌───────────────────┐   │
│   person: 0x100 ────┼─────►│ │ Original Person   │   │
│   (unchanged!)      │      │ │ name: "Alice"     │   │
└─────────────────────┘      │ └───────────────────┘   │
                             └─────────────────────────┘

The original reference in main() was never changed!
```

### 📊 Summary: Pass by Value

| Type | What's Passed | Original Affected? |
|------|--------------|-------------------|
| Primitive | Copy of value | ❌ Never |
| Object (modify) | Copy of reference | ✅ Yes (same object) |
| Object (reassign) | Copy of reference | ❌ No (different objects) |

---

## 🔒 The Final Keyword 

The `final` keyword means **"this cannot be changed after initialization"**

### Three Uses of Final

```java
// 1. FINAL VARIABLE - Cannot reassign
final int MAX_SIZE = 100;
// MAX_SIZE = 200;  // ❌ Compilation Error!

// 2. FINAL METHOD - Cannot be overridden
// 3. FINAL CLASS - Cannot be extended
```

### Example 1: Final Variables (Constants)

```java
public class FinalVariables {
    
    // Final class variable (constant)
    public static final double PI = 3.14159;
    
    // Final instance variable - must be initialized
    private final String id;
    
    public FinalVariables(String id) {
        this.id = id;  // Can initialize in constructor
    }
    
    public static void main(String[] args) {
        
        // Final local variable
        final int maxRetries = 3;
        // maxRetries = 5;  // ❌ Error: cannot reassign
        
        System.out.println("PI: " + PI);
        System.out.println("Max retries: " + maxRetries);
    }
}
```

### Example 2: Final with Objects (Important!)

```java
public class FinalWithObjects {
    
    public static void main(String[] args) {
        
        // Final reference - cannot point to different object
        final Person person = new Person("Alice");
        
        // person = new Person("Bob");  // ❌ Error: cannot reassign reference
        
        // BUT... the object's CONTENTS can still change!
        person.setName("Charlie");  // ✅ This is allowed!
        
        System.out.println(person.getName());  // Output: Charlie
        
        // Same with collections
        final List<String> names = new ArrayList<>();
        // names = new ArrayList<>();  // ❌ Error
        names.add("Alice");            // ✅ Allowed!
        names.add("Bob");              // ✅ Allowed!
    }
}
```

**Visual:**
```
┌─────────────────┐      ┌──────────────────────────┐
│ STACK           │      │ HEAP                     │
├─────────────────┤      ├──────────────────────────┤
│                 │      │                          │
│ person: 0x100 ──┼─────►│ ┌────────────────────┐  │
│ [FINAL - can't  │   ╲  │ │ Person Object      │  │
│  change ref]    │    ╲ │ │ name: "Alice"      │  │
│                 │     ╳│ │   → "Charlie" ✅   │  │
│                 │    ╱ │ │ (contents CAN      │  │
│                 │   ╱  │ │  change!)          │  │
│                 │      │ └────────────────────┘  │
└─────────────────┘      └──────────────────────────┘

❌ Cannot do: person = new Person("Bob")
✅ Can do: person.setName("Charlie")
```

### Example 3: Final Methods

```java
class BankAccount {
    private double balance;
    
    public BankAccount(double balance) {
        this.balance = balance;
    }
    
    // Final method - cannot be overridden by subclasses
    public final double calculateInterest() {
        return balance * 0.05;  // 5% interest
    }
    
    public double getBalance() {
        return balance;
    }
}

class SavingsAccount extends BankAccount {
    
    public SavingsAccount(double balance) {
        super(balance);
    }
    
    // ❌ This would cause compilation error:
    // @Override
    // public double calculateInterest() { ... }
    
    // But we can override non-final methods
    @Override
    public double getBalance() {
        return super.getBalance() + 10;  // Add bonus
    }
}
```

### Example 4: Final Classes

```java
// Final class - cannot be extended
public final class ImmutablePoint {
    private final int x;
    private final int y;
    
    public ImmutablePoint(int x, int y) {
        this.x = x;
        this.y = y;
    }
    
    public int getX() { return x; }
    public int getY() { return y; }
}

// ❌ This would cause compilation error:
// class ColoredPoint extends ImmutablePoint { ... }
```
---

## ⚠️ Escaping References 

### What is an Escaping Reference?

An **escaping reference** occurs when an internal mutable object is exposed outside its class, allowing external code to modify the object's state directly.

**Analogy:** It's like giving someone the **master key** to your house instead of just letting them in when needed!

### The Problem: Broken Encapsulation

```java
// ❌ BAD EXAMPLE - Reference Escapes!
public class CustomerRecords {
    
    private Map<String, Customer> records = new HashMap<>();
    
    public void addCustomer(Customer customer) {
        records.put(customer.getId(), customer);
    }
    
    // 🚨 DANGER: This exposes internal map!
    public Map<String, Customer> getRecords() {
        return records;  // Reference escapes!
    }
}

// External code can now do this:
public class MaliciousCode {
    public static void main(String[] args) {
        CustomerRecords cr = new CustomerRecords();
        cr.addCustomer(new Customer("C001", "Alice"));
        cr.addCustomer(new Customer("C002", "Bob"));
        
        // Get the internal map
        Map<String, Customer> records = cr.getRecords();
        
        // Modify it directly - bypassing encapsulation!
        records.clear();  // 😱 All records deleted!
        records.put("HACKER", new Customer("X", "Evil"));
        
        // The internal state is now corrupted
    }
}
```

### More Escaping Reference Examples

```java
// ❌ BAD: Returning mutable object from getter
public class Order {
    private List<Item> items = new ArrayList<>();
    
    public List<Item> getItems() {
        return items;  // Escaping reference!
    }
}

// External code can:
order.getItems().clear();        // Delete all items
order.getItems().add(freeItem);  // Add items for free
```

```java
// ❌ BAD: Storing mutable parameter directly
public class Team {
    private List<String> members;
    
    public Team(List<String> members) {
        this.members = members;  // Escaping reference via constructor!
    }
}

// External code can:
List<String> myList = new ArrayList<>();
myList.add("Alice");
Team team = new Team(myList);
myList.add("Hacker");  // Team's internal list is modified!
```

---

### ✅ Solutions to Escaping References

### Solution 1: Return Defensive Copies

```java
public class CustomerRecords {
    
    private Map<String, Customer> records = new HashMap<>();
    
    public void addCustomer(Customer customer) {
        records.put(customer.getId(), customer);
    }
    
    // ✅ GOOD: Return a copy!
    public Map<String, Customer> getRecords() {
        return new HashMap<>(records);  // Return a copy
    }
}
```

### Solution 2: Return Unmodifiable Collections

```java
import java.util.Collections;

public class Order {
    private List<Item> items = new ArrayList<>();
    
    public void addItem(Item item) {
        items.add(item);
    }
    
    // ✅ GOOD: Return unmodifiable view
    public List<Item> getItems() {
        return Collections.unmodifiableList(items);
    }
}

// Now external code gets:
order.getItems().clear();  // ❌ Throws UnsupportedOperationException!
```

### Solution 3: Copy Constructor Parameters

```java
public class Team {
    private final List<String> members;
    
    // ✅ GOOD: Create defensive copy of parameter
    public Team(List<String> members) {
        this.members = new ArrayList<>(members);  // Copy!
    }
    
    public List<String> getMembers() {
        return Collections.unmodifiableList(members);
    }
}

// Now external changes don't affect Team:
List<String> myList = new ArrayList<>();
myList.add("Alice");
Team team = new Team(myList);
myList.add("Hacker");  // Team's list is NOT affected!
```

### Solution 4: Return Copies of Mutable Objects

```java
public class Employee {
    private String name;
    private Date startDate;  // Date is mutable!
    
    public Employee(String name, Date startDate) {
        this.name = name;
        // ✅ Defensive copy in constructor
        this.startDate = new Date(startDate.getTime());
    }
    
    public Date getStartDate() {
        // ✅ Defensive copy in getter
        return new Date(startDate.getTime());
    }
}
```

### 📊 Escaping References - Summary Table

| Problem | Solution | Example |
|---------|----------|---------|
| Returning internal collection | Return copy or unmodifiable | `Collections.unmodifiableList()` |
| Storing external collection | Create defensive copy | `new ArrayList<>(param)` |
| Returning mutable object | Return copy | `new Date(date.getTime())` |
| Internal mutable object | Use immutable alternative | `LocalDate` instead of `Date` |

---

## 📝 Summary & Best Practices {#summary}

### Memory Types Quick Reference

```
┌────────────────────────────────────────────────────────┐
│                    JVM MEMORY                          │
├─────────────────────┬──────────────────────────────────┤
│    STACK            │            HEAP                  │
├─────────────────────┼──────────────────────────────────┤
│ • Method calls      │ • Objects (new keyword)          │
│ • Local variables   │ • Instance variables             │
│ • Primitives        │ • Arrays                         │
│ • References        │ • Strings                        │
│ • Fast, LIFO        │ • Garbage collected              │
│ • Thread-specific   │ • Shared across threads          │
└─────────────────────┴──────────────────────────────────┘
```

### Pass by Value Golden Rules

1. **Java is ALWAYS pass by value**
2. For primitives: the value is copied
3. For objects: the reference is copied (not the object)
4. Reassigning a parameter never affects the original variable

### Final Keyword Checklist

- ✅ `final` variable: Cannot be reassigned
- ✅ `final` method: Cannot be overridden
- ✅ `final` class: Cannot be extended
- ⚠️ `final` object reference: Object contents CAN still change!

### Escaping References Prevention

```java
// ✅ DO:
return Collections.unmodifiableList(list);
return new ArrayList<>(list);
this.list = new ArrayList<>(param);
Use immutable classes

// ❌ DON'T:
return internalList;
this.list = param;
return mutableObject;
```

### Best Practices Summary

| Practice | Benefit |
|----------|---------|
| Use primitives for simple values | Better performance, Stack storage |
| Make classes immutable when possible | Thread-safe, no escaping references |
| Use defensive copies | Protect encapsulation |
| Prefer `final` for fields | Clearer intent, safer code |
| Understand reference vs value | Avoid subtle bugs |

---
