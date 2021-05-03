- What is the diference between multiple processes and multiple threads?
  - The essential difference is that while each process has a complete set of its own variables, threads share the same data.

# 12.1 What Are Threads?

STEPS for running a task in a separate thread

1. Place code for tasks into run method of Runnable interface.

```Java
@functional // it's functional interface
public interface Runnable {
  void run();
}
// create an instance with lambda expression
Runnable r = () -> { task code };
```

2. Construct a `Thread` object from `Runnable` instance

```Java
var t = new Thread(r);
```

3. start the Thread

```Java
t.start();
```

- (12.1) threads/ThreadTest.java
- (12.2) threads/Bank.java

* `Thread.sleep()` amy throw a checked exception `InterruptedException`

### Other option (**Not Recommended**)

1. a forming a subclass of the Thread

```Java
class MyThread extends Thread {
  public void run() {
    task code
  }
}
```

2. create an instance of such subclass

```Java
Thread t = new MyThread();
```

3. start the Thread

```Java
t.start();
```

- Why not recommended?
  - tasks and thread should be separated, decoupled.
  - can use thread pool for many tasks.
- DON'T call the `run` method directly
  - it's will executes the task in the **same** thread. No new thread will be created.

# 12.2 Thread States

- Six states
  1. New
  2. Runnable
  3. Blocked
  4. Waiting
  5. Timed Waiting
  6. Terminated

## 12.2.1 New Threads

## 12.2.2 Blocked and Waiting Threads

## 12.2.3 Terminated Threads

# 12.3 Thread Properties

## 12.3.1 Interrupting Threads

## 12.3.2 Deamon Threads

## 12.3.3 Thread Names

## 12.3.4 Handlers for Uncaught Exceptions

## 12.3.5 Thread Priorities

# 12.4 Synchronization

## 12.4.1 An Example of a Race Condition

## 12.4.2 The Race Condition Explained

## 12.4.3 Lock Objects

## 12.4.4 Condition Objects

## 12.4.5 The synchronized keyword

## 12.4.6 Synchronized Blocks

## 12.4.7 The Monitor Concept

## 12.4.8 `Volatile` Fields

- a lock-free mechanism for synchronzing access to an instance field.
- when declere a field as `volatile`, then the compiler and the virtual machine take into account that the field may be concurrently updated by another thread.

```Java
private boolean done;
public synchronized boolean isDone() { return done; }
public synchronized void setDone() { done = true; }

// better
private volatile boolean done;
public boolean isDone() { return done; }
public void setDone() { done = true; }
```

- A warning, **no atomicity** provided from `volatile` fields

```Java
public void flipDone() { done = !done; } // not automic
// not guarantee to flip the value;
// no guarantee that reading, flipping, and writng is uninterrupted.
```

## 12.4.9 `Final` Variables

- one other situation in which it's safe to access a shared field, declared with `final`

```Java
final var accounts = new HashMap<String, Double>();
```

- Then other threads get to see the `accounts` variable after the constructor has finished.
- Without `final`, other threads would see `accounts` as null.

我想这里是因为 `constant` 的原因，`accounts` 跟 `new HashMap<>()`绑定在一起，但是 `Hashmap` 本身不是 thread-safe 的，需要用到 synchronization 或者 `ConcurrentHashMap`

## 12.4.10 Atomics

- a number of classes in the `java.util.concurrent.atomic` package that use efficient machine-level instructions to guarantee atomicity of other operations(expcet assignments) without using locks.
  - AtomicInteger
  - AtomicLong
- for more complex update, using `compareAndSet`, `updateAndGet`, `accumulateAndGet`, `getAndUpdate`, `getAndAccumulate`...

## 12.4.11 Deadlocks

## 12.4.12 Thread-Local Variables

## 12.4.13 Why the stop and suspend Methods Are Deprecated?

# 12.5 Thread-Safe Collections

## 12.5.1 Blocking Queues

## 12.5.2 Efficient Maps, Sets, and Queues

## 12.5.3 Atomic Update of Map Entries

## 12.5.4 Bulk Operations on Concurrent Hash Maps

## 12.5.5 Concurrent Set Views

## 12.5.6 Copy on Write Arrays

## 12.5.7 Parallel Array Algorithms

## 12.5.8 Older Thread-Safe Collections

# 12.6 Tasks and Thread Pools

# 12.7 Asynchronous Computations

# 12.8 Processes
