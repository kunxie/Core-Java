# 6.1 Interfaces

## 6.1.1 The Interface Concept

**Definition:** a set of requirements for the classes that want to conform to the interface

All methods(>= 1) of an interface are automatically public (No need to use public for interface methods)

Interface can also define constants(automatically `public static final`), but it can't have instance files.

Usage

1. delaraction: `class Employee implements Comparable`
2. implementation of interface methods

Comparable Interface (example, also functional interface)

```java
// negative this < other,
// 0 equal
// postive this > other, ASC order by default
# version 1, need casting 🉑️
public interface Comparable { int compareTo(Object other); }
# version 2, generic type ✅
public interface Comparable<T> { int compareTo(T other); }
```

Some principles for compareTo method:

- `x.compareTo(y) == 0`, `x.equals(y) == true`应该在相同的情况下产生, 除了`BigDecimal`精度不同都不能相等(1.00 and 1.0)
- `x.compareTo(y)`, `y.compareTo(x)`应该返回相反的答案，如果要报错应该都报错

List Example:

- (6.1) interface/EmployeeSortTest.java
- (6.2) interface/Employee.java

## 6.1.2 Properties of Interfaces

- can't use new operator to instantiate an interface
  - `x = new Comparable(...); // Error`
- can delcare interface variable
  - `Comparable x; // OK`
- use `instanceof` to check an interface as with subclass
  - `if (anObject instanceof Comparable) { ... }`
- can extend interface
  - `public interface interfaceA { ... }`
  - `public interface interfaceB extends interfaceA { ... }`

## 6.1.3 Interfaces and Abstract Classes

- each class can only extends ONE superclass (**single inheritance**), but implement MULTIPLE interface
  - `public class implements Cloneable, Comparable { ... }`
- interface affords most of the benefits of **multiple inheritance** while avoiding the complexities and inefficiencies

## 6.1.4 Static and Private Methods

- old version: interface/companion class(implementation)
  - Example: Collection/Collections, Path/Paths
- since Java 8, static method is allowed in interfaces
  - companion classes is no longer necessary.
- since Java 9, private method is allowed in interfaces
  - only used as helper methods inside the interfaces

## 6.1.5 Default Methods

- can supply a default implementation for any interface method, with `default` modifier
- a default method can call other abstract method
- why default implementation? **interface evolution**
  - When you add a method with default implementation to the interface, the classes implements the interface don't have to re-compile.

```java
public interface Collection {
    int size(); // an abstract method
    default boolean isEmpty() { return size() == 0; }
}
```

## 6.1.6 Resolving Default Methods Conflicts

- conflicts happened?
  - same methods defined as default one in a interface, and then again as a method of a superclass or another interface (no matter default or not)
- Rule:
  - Superclass win. Any default method from interface is simply ignored.
    - then, can't redefines methods in `Object` class. (will be ignored)
  - Interfaces clash: Must resolve the conflict by overriding that method in class.

```java
// interface 1
interface Person {
    default String getName() { return ""; }
}
// interface 2
interface Named {
    default String getName() { return getClass().getName(); }
}
// resolve the conflict
class Student implements Person, Named {
    ...
    // in the method, you can choose one of two conflicting method!
    public String getName() { return Person.super.getName(); }
}
```

## 6.1.7 Interfaces and Callbacks

**Callback:** specify the action that should occur whenever a particular event happes.

Java, as OOP, pass **objects** of some classes. then call certain methods of those objects.

Use **functional interface** (with only one method) to specify the method to be called.

## 6.1.8 The Comparator Interface

```java
public interface Comparator<T> {
    int compare(T first, T second);
}
// create comparator class
class LengthComparator<String> implements Comparator<String> {
    public int compare(String first, String second) {
        return first.length() - second.length();
    }
}
// use it
String[] friends = {"Peter", "Paul", "Mary"};
Arrays.sort(friends, new LengthComparator()); // lambda can be used here.
```

## 6.1.9 Object Cloning

其实也没多少人用 Cloneable, 只有 5% Standard Library 用它

- `Cloneable` interface, just a **tagging interface**
  - contains no methods, just allow the use of **instanceof** in type inquiry
- `clone` method is a `protected` method of `Object`.
  - a `clone` method used for deep cloning Object.
- `cloneNotSupportedException` throwed whenever clone is invoked on an object whose class does **NOT** implement `Cloneable` interface.
- alternative `clone`
  - **serialization** feature of Java, easy and safe mechanism but not efficient.

All array types have a clone method that is public.

```java
int[] luckyNumbers = { 2, 3, 5, 7, 11, 13 };
int[] cloned = luckNumbers.clone();
cloned[5] = 12; // doesn't change luckyNumbers[5];
```

- (6.4) clone/CloneTest.java
- (6.5) clone/Employee.java

# 6.2 Lambda Expressions

## 6.2.1 Why Lambdas?

## 6.2.2 The Syntax of Lambda Express

## 6.2.3 Functional Interfaces

## 6.2.4 Method References

## 6.2.5 Constructor References

## 6.2.6 Variable Scope

## 6.2.7 Processing Lambda Expressions

## 6.2.8 More about Comparators

# 6.3 Inner Classes

## 6.3.1 Use of an Inner Class to Access Object State

## 6.3.2 Special Syntax Rules for Inner Classes

## 6.3.3 Are Inner Classes Useful? Actually Necessary? Secure?

## 6.3.4 Local Inner Classes

## 6.3.5 Accessing Variables from Outer Methods

## 6.3.6 Anonymous Inner Classes

## 6.3.7 Static Inner Classes

# 6.4 Service Loaders

# 6.5 Proxies

## 6.5.1 When to Use Proxies

## 6.5.2 Creating Proxy Objects
