# 7.1 Dealing with Errors

- Java allows every method an **alternative exit path** if it's unable to complete its task in the normal way.
  - _throws_ an **object** the encapsulates the error information
  - exception-handling mechanism begins its search for **an exception handler** dealing with this error condition.

## 7.1.1 The Classification of Exceptions

- An exception object is always an instance of a class derived from `Throwable`, then immediately splites into two branches: `Error` and `Exception`
  - **Error:** internal errors and resource exhaustion situations in the runtime system. **should NOT throw an object of this type.** (Beyond my control)
  - **Exception:** also splits into two branches: `RuntimeException` or not.
    - `RuntimeException` such as (`ArrayIndexOutOfBoundsException`, `NullPointerException`) should be **completely avoid**. (Under my control)
    - Not `RuntimeException`, **checked exceptions**, should write handler for such Exceptions. (part of control)

## 7.1.2 Declaring Checked Exceptions

- An exception is thrown in any of the following four situations (前两个就需要 advertise the posibility):
  1. call a method that **throws a checked exception**.
  2. detect an error and **throw a checked exception** with the `throw` statement.
  3. make a programmer error, rising to an unchecked exception (`ArraysIndexOutOfBoundsException`)
  4. An internal error occurs in the virtual machine or runtime library.

```java
class MyAnimation {
    ...
    public Image loadImage(String s) throws FileNotFoundException, EOFException {
        ...
    }
}
```

- If you override a method from a superclass, the checked exceptions that **subclass** method declares **cannot be more general** than those of the **superclass** method.
- If the superclass method throws no chechked exception at all, neither can be subclass.

## 7.1.3 How to Throw an Exception

- Easy if one of the existing exception classes works.
  1. Find an appropriate exception class.
  2. Make an object of that class.
  3. Throw it.

```Java
String readData(Scanner in) throws EOFException {
    ...
    while (...) {
        if (!in.hasNext()) { // EOF encountered
            if (n < len) {
                String grip = "Content-length: " + len + ", Received: " + n;
                throw new EOFException(gripe);
            }
        }
        ...
    }
    return s;
}
```

## 7.1.4 Creating Exception Classes

- create custom Exception class, just derived it from `Exception`, or a child class of `Exception` such as `IOException`.

```Java
class FileFormatException extends IOException {
    public FileFormatException() {}
    public FileFormatException(String gripe) { super(gripe); }
}
```

# 7.2 Catching Exception

- For **un-caught exceptions**, the program will **terminate** and print a message to the console, giving the type of the exception and a stack trace.

## 7.2.1 Catching an Exception

```java
try {
    code
}
catch (ExceptionType e) {
   // handler for this type
   e.getMessages(); // get detailed error message
   e.getClass().getName(); // get the actual type of the exception object.
}
```

- The compiler strictly enforces the `throws` specifiers. If you call a method that throws a checked exception, you must either **handle it** or **pass it on**.
- If you are writing a method that overrrides a superclass method which throws no exceptions, then you must catch each checked exception in your method's code.

## 7.2.2 Catching Multiple Exceptions

```Java
try {
    // code that might throw exceptions
}
// catch multiple exceptions in same catch clause
// used when catching exception types that are not subclasses of one another.
catch (FileNotFoundException | UnknownHostException e) { ... }
catch (IOException e) { ... }
```

## 7.2.3 Rethrowing and Chaining Exceptions

- can throw an exception in a `catch` clause when trying to change the exception type.

```Java
try {
    // access the database
}
catch (SQLException original) {
    var e = ServletException("database error");
    e.initCause(original); // wrap the cause
    logger.log(level, message, e)); // update the logger
    throw e;
}
```

## 7.2.4 The finally Clause

- The code in `finally` clause executes whether or not an exception was caught.

```Java
try { ... }
catch (ExceptionType e) { ... }
finally { ... }
```

- you can use the `finally` clause without a `catch` clause.

```Java
InputStream in = ...;
try {
    try {
        // code that might throw exception
    }
    finally {
        in.close();
    }
}
catch (IOException e) {
    // show error message
    // errors in the finally clause are reported as well.
}
```

## 7.2.5 The try-with-Resource Statement

- Tranditional resource code pattern

```Java
// open a resource
try {
    // work with the resource
}
finally {
    // close the resource
}
```

- Java 7, a usefull shortcut for resouces belongs to a class implementing the `AutoCloseable` interface
  - `AutoCloseable` is a functional interface with a single method `void close() throws Exception`
  - `Closeable` interface is a subinterface of `AutoCloseable` with a method `void close() throws IOException`

```Java
try (Resource res = ...) {
    // work with res
}
```

**example**

```Java
try (var in = new Scanner(...)) {
    while (in.hasNext())
        System.out.println(in.next());
}
// in.close() will be called when the try block exits.
// no matter normally or with exception

// can specify multiple resources
try (
    var int = new Scanner(...);
    var out = new PrintWriter(...)
) {
    // work with resouces
}
// in.close(), out.close() will be called.
```

- Java 9, you can declared effectively final variables in a `try` header.

```Java
public static void printAll(String[] lines, PrintWriter out) {
    try(out) { // effectively final variable
        for (String line: lines)
            out.println(line);
    } // out.close() called here
}
```

- `try`-with-resources statement can have `catch` clauses and a `finally`. there are executed after closing the resources.

## 7.2.6 Analyzing Stack Trace Element

# 7.3 Tips for Using Exceptions

# 7.4 Using Assertions

## 7.4.1 The Assertion Concept

## 7.4.2 Assertion Enabling and Disabling

## 7.4.3 Using Assertions for Parameter Checking

## 7.4.4 Using Assertions for Documenting Assumptions

# 7.5 Logging

## 7.5.1 Basic Logging

## 7.5.2 Advanced Logging

## 7.5.3 Chaning the Log Manager

## 7.5.4 Localization

## 7.5.5 Handlers

## 7.5.6 Filters

## 7.5.6 Formatters

## 7.5.8 A Logging Recipe
