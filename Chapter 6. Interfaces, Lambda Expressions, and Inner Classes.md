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

A **lambda expression** is a block of code that you can pass around so it can be executed later.

In Java (OOP), just like 6.1.8 Comparator interface, but **lambda expression save you life here.**

1. you have to create a class implementing a functional interface,
2. write the code block by overriding the method of the functional interface
3. create the object, and pass it.

## 6.2.2 The Syntax of Lambda Express

## 6.2.3 Functional Interfaces

## 6.2.4 Method References

## 6.2.5 Constructor References

## 6.2.6 Variable Scope

## 6.2.7 Processing Lambda Expressions

## 6.2.8 More about Comparators

# 6.3 Inner Classes

- Inner class can be hidden from other classes in the same package.
- Inner class methods can access the data from the scope in which they are defined
  - including data that would otherwise be **private**.

## 6.3.1 Use of an Inner Class to Access Object State

- An object that comes from an inner class has **an implicit reference** to the outer class object that instantiated it.

  - in listing6.7, directly use `beep` field of outer class in the inner class.
  - Only inner class can be `private`, other class always have either `package` or `public` class.
  - `static` inner classes **do not** have this added pointer. (implicit reference)

- (6.7) innerClass/InnerClassTest.java

## 6.3.2 Special Syntax Rules for Inner Classes

- Proper syntax of outer reference is `OuterClass.this`
- Write inner object constructor explicit `outerObject.new InnerClass(consctruction parameters)`
- refer to an inner class as `OuterClass.InnerClass` when it occurs outside the scope of the outer class.

- Any `static` fields declared in an inner class must be `final` and initialized with a `compile-time` constant.
- An inner class cannot have `static` methods.

## 6.3.3 Are Inner Classes Useful? Actually Necessary? Secure?

❓

## 6.3.4 Local Inner Classes

```Java
public void start() {
    class TimePrinter implements ActionListener {
        @Override
        public void actionPerformed(ActionEvent event) {
            System.out.println("At the tone, the time is "
                + Instant.ofEpochMilli(event.getWhen()));
            if (beep) Toolkit.getDefaultToolkit().beep();
        }
    }
    var listener = new TimePrinter();
    var timer = new Timer(interval, listener);
    timer.start();
}
```

- we can also define the class **locally in a single method.**
- local classes are never declared with an access specifier (Neither `public` nor `private`)
- the scope is always restricted to the block in which they are declared.
- completely hidden from the outside world。
  - No method except `start` has any knowlege of the TimePrinter class.

## 6.3.5 Accessing Variables from Outer Methods

- local classes can even access local variables
  - these local variables must be **effective final**, they may never change once they have been assgined.
    - when the method finished its code and exit, the local variable will no long exit.

## 6.3.6 Anonymous Inner Classes

- for just one object of inner class, not necessary to name it. (**an anonymous inner class**)

```Java
public void start(int interval, boolean beep) {
    // create a new object of a class that implements ActionListener interface
    var listener = new ActionListener() {
        @Override
        public void actionPerformed(ActionEvent event) {
            System.out.println("At the tone, the time is "
                + Instant.ofEpochMilli(event.getWhen()));
            if (beep) Toolkit.getDefaultToolkit().beep();
        }
    }
    var timer = new Timer(interval, listener);
    timer.start();
}
```

- Syntax is following

```Java
// the superType can be either interface or class
new SuperType(construction parameters) {
  //inner class methods and data
}
```

- An anonymous inner class **can't have constructors**, since it doesn't have a name

  - construction parameters are given to the superclass constructor.
  - if SuperType is an interface, can't have consctruction parameters.

- double brace initialization **rarely useful**

```Java
// option 1, traditional
var friends = new ArrayList<String>()
friends.add("Harry");
friends.add("Tony");
invite(friends);
// option 2: double brace initialization
// outer braces make an anonymous subclass
// inner braces are an object initialization block
invite(new ArrayList<String>() {{
  add("Harry");
  add("Tony");
}});
// option 3: use built-in method ✅
invite(List.of("Harry", "Tony"));
```

- be careful with `equals()` method

  - An anonymous subclass will fail `if (getClass() != other.getClass()) return false;`
  - 这里很好理解，因为这样的匿名函数没有名字,

- an static method has no `this`, can't use `this.getClass()`

  - `new Object().getClass().getEnclosingClass()` // gets class of static method
  - create an anonymous object of an anonymous subclass of `Object`, and `getEnclosingClass()` gets the class containing the static method.

- (6.8) anonymousInnerClass/AnonymousInnerClassTest.java

## 6.3.7 Static Inner Classes

- only inner classes can be declared `static`, also called nested class.
  - static inner class has only one difference: **no reference to the outer class object.**
  - static inner class can have static fields and methods.
- Inner class declared inside an interface are automatically **static** and **public**

- (6.9) staticInnerClass/StaticInnerClassTest.java

# 6.4 Service Loaders

# 6.5 Proxies

## 6.5.1 When to Use Proxies

## 6.5.2 Creating Proxy Objects
