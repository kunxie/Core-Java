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

<img src="assets/threadStates.jpg" alt="drawing" width="400"/>

## 12.2.1 New Threads

- just create a thread with `new` operator, in the **new** state.
  - the program has not started executing code inside of it.
  - A certain amount of bookkeeping needs to be done before a thread can run.

## 12.2.2 Runnable Threads

- invoke `start()` method -> the thead is in the **runnable** state.
  - **may or may not** actually be running, ready to be executed by OS. (It's up to OS)
  - All modern desktop and server OS use **preemptive scheduling:**
    - each runnable thread can only perform its task in a slice of CPU time. (they take turns according to priority **12.3.5**)
  - small devices such as cell phones may use **cooperative scheduling:**
    - a thread loses control only when it calls the `yield` method, or when it blocked or waiting.

## 12.2.3 Blocked and Waiting Threads

- **blocked** or **waiting** state, thread is temporarily inactive, no code being executed, costing minimal resouces
  - waiting for **thread scheduler** to reactivate it.
  - in practice, no significant diference bewteen **blocked** and **waiting** state
- how inactive state was reached:
  1. When the thread tries to acquire an **instrinsic object lock** that is currently held by another thread, it becomes **blocked**.
     - becomes unblocked (**runnable state**) when acquired lock that relinquished by other threads.
  2. When the thread waits for another thread to notify the scheduler of a condition, it enters the **waiting** state.
     - `Object.wait()`, `Thread.join()`, or by waiting for a `Lock` or `Condition` in the `java.util.concurrenty` library.
  3. Several methods have a **timeout** parameters. calling them causes thread to enter **timed waiting** state.
     - This state persists either until the timeout expires or the appropriate notification received.
     - `Thread.sleep()`, the timed versions of `Object.wait()`, `Thread.join()`, `Lock.tryLock()`, `Condition.await()`

## 12.2.3 Terminated Threads

- A thread is terminated for one of two reasons:

  1. dies naturally when finished all codes in `run()` method.
  2. dies abruptly when an uncaught exception terminates the `run()` method.

- you can kill a thread by calling `stop()` method, which will throws a `ThreadDeath` erro object that kills the thread.
  - BUT, `stop()` method is deprecated, should **NEVER** be used.

# 12.3 Thread Properties

## 12.3.1 Interrupting Threads

- rather than deprecated `stop()` method, there is no way to **force** a thread to terminate
  - `interrupt()` method can be used to **request termination** of a thread.
  - check the interrupted status (a `Boolean` flag), use `isInterrupted()` method
  - ❗️ when `interrupt` method is called on a thread that blocks on a call (such as `sleep` or `wait`), the blocking call is terminated by an `InterruptedException`
- You can either handle the exception (for important threads) or terminates (most commonly)

  - if your loop calls `sleep()` method, don't check the interrupted status.
  - `sleep()` method will clear the interrupted status(if `true`) and throws an `InterruptedException`

  ```Java
  Runnable r = () -> {
    try {
      ...
      while (!Thread.currentThread().isInterrupted() && more work to do) {
        do more work
      }
    }
    catch (InterruptedException e) {
      // thread was interrupted during sleep or wait
    }
    finally {
      // cleanup, if required
    }
    // exiting the run method terminates the thread
  }
  ```

- Comparison
  | `interrupted()` method | `isInterrupted()` method |
  | ------------------------------------------ | ------------------------------------------ |
  | **static** method | **instance** method |
  | check the **current** thread | can check **any** thread |
  | will then **clear** the interrupted status | does **not change** the interrupted status |

- two way to deal with `Interrupted Exception`, DON'T ignore it

```Java
  // Option 1
  catch (InterruptedException e) {
    // set the interrupted status
    Thread.currentThread().interrupt();
  }

  // tag your method with `throws InterruptedException` and drop the try block.
```

## 12.3.2 Deamon Threads

- turn a thread into **daemon thread** by calling `t.setDaemon(true);`
- A daemon thread has no other role in life than to serve others.
  - ex: timer threads or threads that clean up stale cache entries.
  - When only daemon threads remain, the virtual machine exists.

## 12.3.3 Thread Names

- threads have catchy names by calling `t.setName("Web crawler");`
  - useful in thread dumps

## 12.3.4 Handlers for Uncaught Exceptions

- `run()` method **can't throw any checked exceptions**, terminated by an unchecked exception.
  - we CAN'T use `catch` clause to propagate exceptions.
  - just before the thread dies, the exception is passed to **a hander** for uncaught exception
- The handler must implements `Thread.UncaughtExceptionHandler` functional interface.
  - `void uncaughtException(Thread t, Throwable e)`
- set the handler into any thread with `setUncaughtExceptionHandler()` method
  - otherwise the thread's `ThreadGroup` object
- OR set default handler for all threads with static `setDefaultUncaughtExceptionHandler()` method

  - otherwise the default handler `null`

- actions taken by `uncaughtException()` method of `ThreadGroup` class
  1. if the thread group has a parent, call parent's `uncaughtException()` method
  2. **otherwise**, if `Thread.getDefaultUncaughtExceptionHandler` method return non-null handler, call it.
  3. **otherwise**, if the `Throwable` is an instance of `ThreadDeath`, nothing happens.
  4. **Otherwise**, the name of thread and the stack trace of the `Throwable` are printed on `System.err`

## 12.3.5 Thread Priorities

- In Java, every thread has a **priority**.
- By default, a thread inherits the priority of the thread that constructed it.
- explicitly set priority by calling `setPriority()` method.
  - constants: `MIN_PRIORITY` (defined as 1), `MAX_PRIORITY` (defined as 10), `NORM_PRIORITY` (defined as 5)
- JVM try to map Java Thread priorty to OS priority levels, with more or fewer levels
  - it's highly system-dependent
  - In Oracle JVM for Linux, thread priorities are ignored completely.
- ❗️ **You should not use priority nowadays.**

# 12.4 Synchronization

## 12.4.1 An Example of a Race Condition

- **a race condition**: the threads step on each other's toes.

- (12.3) unsynch/UnsynchBankTest.java

## 12.4.2 The Race Condition Explained

- omit

## 12.4.3 Lock Objects

- basic outline for protecting a code block with a `ReentrantLock`
  - `try`-with-resources statement will not work.
  -
  ```Java
  public class Bank {
    // each reentrantLock object for each Bank object
    private var bankLock = new ReentrantLock();
    ...
    public void transfer(int from, int to, int amount) {
      bankLock.lock(); // try to acquire the lock
      try {
        // code block
        ...
      }
      finally {
        // MUST release the lock even if an exception is thrown
        // otherwise, other threads wil be blocked forever. ❗️
        bankLock.unlock();
      }
    }
  }
  ```
- **reentrant** means a thread can repeatedly acquire a lock that it already owns
  - the Lock has a **hold count** that keeps track of the nested calls to the `lock` method.
  - The thread has to call `unlock()` for every call to `lock()` method to relinquish the lock
    - In `transfer` method, the `getTotalBalance` method is called, then `bankLock.lock()` has been called twice, the **count hold** becomes **2**
    - when the `transfer` method exists with **hold count == 0**, the thread reliquishes the lock.
- `ReentrantLock(boolean fair)`
  - a fair lock favors the thread that has been waiting for the longest time.
  - by default, locks are NOT fair
  - fair locks are **a lot slower** than regular locks.
    - 而且即使设置成了 fair, 最终还是要依靠 thread scheduler 来实现 fair，并不能一定保证

## 12.4.4 Condition Objects

- when a thread acquired a bankLock but found not enough money to be transfered.
  - `Condition` objects used to manage threads that have acquired a lock but cannot do useful work.
- Understand the **mechnism** under the hood ❗️ 结合例子(12.4)分析
  - A lock object can have one or more associated condition objects.
    - `sufficientFunds = bankLock.newCondition();`
    - when sufficient funs are NOT avaiable
      - `sufficientFunds.await();`, then thread deactivated and gives up the lock
  - difference bewteen `await()` method and `waiting` state.
    - after called `await()`, it enters a **wait set** for that condition. The thread **not** moade runnable when the lock is available. Instead, it stays deactivated until another threads call `signalAll()` method on the same condition.
    - `await()` 是不可以自己唤醒的，只能依靠其他 threads
  - `signalAll()`, reactivates all threads waiting for the condition. When the threads removed from the **wait set**, they are again runnable and the scheduler will eventually active them again.
    - **when to use** `signalAll()`, to call it whenever the state of object changes in a way advantageous to waiting threads.
    - when lock avalable, one of threads will **acquire the lock and continue where it left off**, returning from the call to await.
    - `signalAll()` does NOT immediately active a waiting thread. only unblocks the waiting threads so that they can compete for entry into object after the current thread has reliquished the lock
  - `signal()` unblocks only a single thread from the wait set, chose at random.
    - more efficient but more dangerous, may cause **deadlock**
- (12.4) synch/Bank.java

## 12.4.5 The synchronized keyword

- Ever since version 1.0, **every object** in Java has **an instrinsic lock**. If a method is declared with `synchronized` keyword, the object's lock protects the entire method.

  ```Java
  public synchronized void method() { ... }

  // is equivalent of
  public void method() {
    this.intrinsicLock.lock();
    try {
      // method body
    }
    finally { this.intrinsicLock.unlock(); }
  }
  ```

- similarly, `wait()` or `notifyAll()` or `notify()` methods in Object class.
  - since they are final methods, so conditions has to choose different names.
  - equivalent of `instrinsicCondition.await()`, `instrinsicCondition.signalAll()`, `instrinsicCondition.signal()` methods.
- declare **static** method `synchronized`
  - it acquires the instrinsic lock of the associated class object. ex, `Bank.class` object
- **limitations** for threads and conditions
  - Can't interrupt a thread that is trying to acquire a lock
  - **Can't specify a timeout** when trying to acquire a lock
  - Having a single condition per lock can be **inefficient**.
- **Recommendations** 使用的顺序
  - best to use **neither** `Lock/Condition` **nor** the `synchronized` keyword.
    - try to use one of the mechanisms of the `java.util.concurrent` package.
  - If the `synchronized` keyword works for your situation, use it.
  - Use `Lock/Condition` if you really need additional power.
- (12.5) synch2/Bank.java

## 12.4.6 Synchronized Blocks

- When a thread enters a synchronized block, it then acquires the lock for `obj`
  ```Java
  schcronized(obj) {
    critial section
  }
  ```
  - a "ad hoc" locks auch as following
  ```Java
  public class Bank {
    private var lock = new Object(); // created only to use the lock
    ...
    public void transfer(int from, int to, int amount) {
      synchronized(lock) {
        accounts[from] -= amount;
        accounts[to] += amount;
      }
      System.out.println(...);
    }
  }
  ```

## 12.4.7 The Monitor Concept

这里只是一个概念了解

- `Lock/Condition` are not very object-oriented.
- then **monitor concept** to make multithreading safe **without** thinking about explicit locks.
  - properties in **monitor concept**
    - A monitor is a class with only private fields
    - Each object of that class has an associated lock
    - All methods are locked by that lock.
      - if client class `obj.method()`, the lock automatically acquired at the beginning, and reliquished when the method returns.
    - The lock can have any number of associated conditions
- Java designer **loosely** adapted the monitor concept
  - using `synchronized` keyword and `wait/notifyAll/notify` method, it can act as a monitor.
  - a java object differs from a monitor in three important ways
    1. Fields are not required to be `private`
    2. Methods are not required to be `synchronized`
    3. The instrinsic lock is available to clients

## 12.4.8 `Volatile` Fields

- Why get wrong?
  - because many threads will temporarily hold memory values in register or local memory caches. For multithreading, threads in different processors may see different values for the same memory location.
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
  - should perform no operations other than assignment

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

  ```Java
  public static AtomicLong nextNumber = new AtomicLong();
  // operations of getting the value, adding 1,
  // setting it and producing the new value cannot be interrupted.
  // guaranteed correct value is computed and returned,
  // event if multiple threads access the same instance concurrently
  long id = nexNumber.incrementAndGet();
  ```

- for very large number of threads accessing the same atomic values
  - use `LongAdder` and `LongAccumulator` classes for `long` values
  - similar `DoubleAdder` and `DoubleAccumulator` for `double` values
- `LongAdder` mechanism
  - A `LongAdder` is composed of multiple variables whose collective sum is current value.
  - Different threads can update different summands
  - new summands are automatically provided when the number of threads increases.
- `LongAccumulator` mechanism
  - internally, the accumulator has variables a1, a2, ... an, each of them is initialized with **neutral element**
  - when calling `accumulative()` with value `v`, one of the atomically updated as a_i = a_i op v
  - result of `get()` method is a1 op a2 op ... op an.
- In general, the operation must be **associative** and **commutative**

  - final result independent of the order in which intermediate values were combined.

  ```Java
  // ex of LongAdder
  var adder = new LongAdder();
  for (...)
    pool.submit(() -> {
      while (...) {
        ...
        if (...) adder.increment();
      }
    });
  ...
  long total = adder.sum();

  // ex of LongAccumulator
  var accu = new LongAccumulator(Long::sum, 0);
  // in some thread...
  accu.accumulate(value);
  ```

## 12.4.11 Deadlocks

- **deadlock**
  - all threads get blocked because each of them are waiting for other threads to reliquish the locks or satisfy the condition.
- context gave some ways to get deadlock by modifying `sync/Bank.java` class
- `ctrl + \` to view the thread dump listing all threads.
  - each thread has a stack trace, telling you where it's currently blocked.
- ❗️ **there is nothing in Java to avoid or break these deadlocks. you must design your program to prevent deadlocks from happening.**

## 12.4.12 Thread-Local Variables

- You can give each thread its own instance, using `ThreadLocal` helper class.

  - example, `SimpleDateFormat` is not thread-safe

  ```Java
  public static final ThreadLocal<SimpleDateFormat> dateFormat
    = ThreadLocal.withInitial(() -> new SimpleDateFormat("yyyy-MM-dd"));

  // usage
  // the get() method will return the instance belonging to current thread.
  String dataStamp = dateFormat.get().format(new Date());
  ```

- Even though `java.util.Random` class is thread-safe, since Java 7, a convience class provided.
  ```Java
  // current() returns an instance of the Random class unique to the current thread.
  int random = ThreadLocalRandom.current().nextInt(upperBound);
  ```

## 12.4.13 Why the stop and suspend Methods Are Deprecated?

- `stop()` `suspend()` `resume()` NOW **deprecated**
  - both attemp to control the behavior of a given thread without the thread's cooperation.
  - `stop` inherently unsafe
  - `suspend` frequently leads to deadlocks
- `stop()`
  - terminates all pending methods, including the `run` method
  - When a thread is stopped, it **immediately gives up the locks** on all object that it has locked.
    - the objects may be in a inconsistent state, **damaged**
  - ✅ you **should interrupted** a thread and the interruped thread can stop when it's safe to do so.
- `suspend()`
  - if you suspend a thread that owns a lock, then the lock is **unavailable** until the thread is resumed.
  - cause no damage objects but deadlocks
  - ✅ if you want to **safely suspend a thread**, **introduce a variable `suspendRequested`** and test it in a safe place of your `run` method
    - when the thread finds the `suspendedRequested` variable has been set, it should keep waiting until it becomes available again.

# 12.5 Thread-Safe Collections

## 12.5.1 Blocking Queues

- Producer threads insert items into the queue, and consumer threads retrieven them.
  - The queue lets your safely hand over data from one thread to another
- A `blocking queue` causes a thread to block when **adding** an element to a **full** queue, or **removing** from a **empty** queue.
  - **illegal to insert `null` values**

| Blocking Queue methods | description                        | Actions                                  |
| ---------------------- | ---------------------------------- | ---------------------------------------- |
| `add`                  | Adds an element                    | Throws `IllegalStateException` if full   |
| `element`              | Return the head element            | throws `NoSuchElementException` if empty |
| `offer`                | Adds an element and returns `true` | returns `false` if full                  |
| `peek`                 | Return the head element            | returns `null` if empty                  |
| `poll`                 | removes & returns the head element | returns `null` if empty                  |
| `put`                  | Adds an element                    | **block** if full                        |
| `remove`               | removes & returns the head element | throws `NoSuchElementException` if empty |
| `take`                 | removes & returns the head element | **block** if the queue is empty          |

- for thread management, use `put/take` methods ✅

  - `add/remove/element` will throw an exception ❌
  - `offer/peek/pull` can also be used ina multithreading program. ✅

- You can also set a timeout to `offer/poll` methods
  ```Java
  boolean success = q.offer(x, 100, TimeUnit.MILLISECONDS);
  Object head = q.poll(100, TimeUnit.MILLISECONDS);
  ```
- concrete implementations

  - `LinkedBlockingQueue`
    - has no upper bound of capacity, but can specify one.
  - `ArrayBlockingQueue`
    - constructed with a given capacity and an optional fairness parameter.
    - if fairness specified, the longest-waiting threads are given preferential treatment.
      - fairness will slow performance, should be used carefully.
  - `PriorityBlockQueue` - not FIFO queue
    - elements are removed in order of their priority
    - unbounded capacity, but retrieval will block if the queue is empty.
  - `DelayQueue`
    - objects must implement the `Delayed` interface
    - element can only be removed if their delay has elapsed.
    - `DelayQueue` use `compareTo` method to sort the entries.
    ```Java
    interface Delayed extends Comparable<Delayed> {
      // returns the remaining delay of the object.
      // nagative value indicates the delay has elapsed.
      long getDelay(TimeUnit unit);
    }
    ```
  - `LinkedTransferQueue` implements `TransferQueue` interface.
    - allows a producer thread to **wait** until a consumer is ready to take on an item.
    - `q.transfer(item);` the producer blocks until another removes the item.

- (12.6) blockingQueue/BlockingQueueTest.java
  - ❗️❗️ understand how to use queue data structure as synchronization mechanism.

## 12.5.2 Efficient Maps, Sets, and Queues

- other implementations

  - `ConcurrentHashMap`, `ConcurrentSkipListMap`, `ConcurrentSkipListSet`, `ConcurrentLinkedQueue`

- `size()` methods of these classes **NOT O(1)**
  - or using `mappingCount()` method returns the size as long.
- these classes will return **weakly consistent** iterators

  - may or may not reflect all modifications made after they were constructed.
  - will not return a value twice
  - will not throw a **ConcurrentModificationException**

- For `ConcurrentHashMap`
  - efficiently support a large number of readers and a fixed number of writers
  - usually 16 simultaneous writer threads, if more than 16, some will temporarily blocked.

## 12.5.3 Atomic Update of Map Entries

- `put()` method is NOT thread-safe ❌

  - ConcurrentHashMap 里面不是所有的 method 都是 thread-safe 的，他们只是不会被多线程破坏内部结构

  ```Java
  Long oldValue = map.get(word);
  Long newValue = oldValue == null ? 1 : oldValue + 1;
  // ERROR -- might not replace oldValue due to race condition
  map.put(word, newValue);
  ```

- in old version of Java, using `replace()` method ⭕️

  ```Java
  do {
     Long oldValue = map.get(word);
     Long newValue = oldValue == null ? 1 : oldValue + 1;
  } while (!map.replace(word, oldValue, newValue));
  ```

- An alternative for `ConcurrentHashMap<String, AtomicLong>` ⭕️

  ```Java
  // a new AtomicLong will be constructed for each increment, very inefficient.
  map.putIfAbsent(word, new AtomicLong());
  map.get(word).incrementAndGet();
  ```

- nowadays, using `compute()` method ✅

  ```Java
  map.compute(word, (key, oldValue) -> oldValue == null ? 1 : oldValue + 1);
  ```

- can't have `null` values in a `ConcurrentHashMap`, an indication for absence.
- variants `computeIfPresent()` and `computeIfAbsent()` ✅

  ```Java
  // the constructor only called when necessary.
  map.computeIfAbsent(word, k -> new LongAdder()).increment();
  ```

- `merge()` method, combines the existing value and initial value ✅

  ```Java
  map.merge(word, 1L, (existingValue, newValue) -> existingValue + newValue);
  // more concise
  map.merge(word, 1L, Long::sum);
  ```

- (12.7) concurrentHashMap/CHMDemo.java

## 12.5.4 Bulk Operations on Concurrent Hash Maps

- Unless you happen to know that the map is not being modified while a bulk operation runs, you should treat its result as **an approximation** of the map's state.
- three kinds of operations
  1. `search`
  2. `reduce`
  3. `forEach`
- four versions for each operations
  1. `operationKyes`
  2. `operationsValues`
  3. `operation`: operates on keys and values
  4. `operationEntries`: operates on `Map.Entry` objects
- need to specify a `parallelism threshold` for each operations
  - if the map contains more elements than the threshold, the bulk operation is parallelized.
  - threshold of `1` -> use maximum number of threads
  - threshold of `Long.MAX_VALUE` -> single thread

### `search` methods

- `U search*(long threshold, BiFunction<? super K, ? extends U> f)`

### `forEach` methods

- two variants
  1. simply applies a `consumer`
  ```Java
  map.forEach(threshold, (k, v) -> System.out.println(k + " -> " + v));
  ```
  2. takes an additional `transformer` func, then pass the result to a `consumer`
     - `transformer` can be used as a **filter**: if transformer returns `null`, the value will be skipped.
  ```Java
  map.forEach(threshold,
    (k, v) -> k + " -> " + v, // transformer
    System.out::println);     // consumer
  ```

### `reduce` methods

- two variants
  1. simply applies a `accumulation` func
  ```Java
  Long sum = map.reduceValues(threshold, Long::sum);
  ```
  2. supply a `transformer` func
     - `transformer` can be used as a **filter**: if transformer returns `null`, the value will be skipped.
  ```Java
  Integer maxLength = map.reduceKeys(threshold,
    String::length,             // transformer
    Integer::max);              // accumulator
  ```
- if map is empty, `reduce` methods return `null`.
- if map has only one element, return transformation result, ignore accumulator.

### Primitive output suffixes

- three primitive output suffixes. **Optional**
  1. `opertaionVersionToInt`
  1. `opertaionVersionToLong`
  1. `opertaionVersionToDouble`
  ```Java
  long sum = map.reduceValuesToLong(threshold,
    Long::longValue,    // transformer to primitive type
    0,                  // default value for empty map
    Long::sum);         // primitive type accumulator
  ```

## 12.5.5 Concurrent Set Views

- for **a large, thread-safe set**, No such `ConcurrentHashSet` class
  - `newKeySet` method yields a `Set<K>`: a wrapper around a `ConcurrentHashMap<K, Boolean>` with values `Boolean.TRUE`
  ```Java
  Set<String> words = ConcurrentHashMap.<String>newkeySet();
  ```
  - for an existing map, use `keySet()` method.
    - mutable, but nonsense to add a key without value.
    - you can set default value with `keySet()`
  ```Java
  Set<String> words = map.keySet(1L);
  words.add("Java"); // its a Java-1L key-value pair in the map.
  ```

## 12.5.6 Copy on Write Arrays

- `CopyOnWriteArrayList` `CopyOnWriteArraySet` thread-safe collections.
  - all mutators make a copy of the underlying array.
  - for iterator constructed, the array later mutated, iterator still has the old array.

## 12.5.7 Parallel Array Algorithms

- The `Arrays` class has a number of parallelized operations
  - `Arrays.parallelSort`: sort an array of primitive values or objects
  - `parallelSetAll`: fills an array with values computed from a function.
    - the func receives the **index** and computes the value at that location.
  - `Arrays.parallelPrefix`: replaces each array element with the **accumulation** of the prefix for **a given associative operation**.
    - `log (n)`

```java
String[] words = ...;
Arrays.parallelSort(words);
// supply a Comparator
Arrays.parallelSort(words, Comparator.comparing(String::length));
// supply the bound of a range, e.g. the upper half
values.parallelSort(values.length/2, values.length);

// values[0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 0, 1, 2,....]
Arrays.parallelSetAll(values, i -> i % 10);

// before values [1, 2,   3,     4, ...]
// after  values [1, 1x2, 1x2x3, 1x2x3x4, ...]
Arrays.parallelPrefix(values, (x, y) -> x * y);
```

## 12.5.8 Older Thread-Safe Collections

- `Vector` and `HashTable` thread-safe class, considered **obsolete**
- Any collection class can be made thread-safe by means of a `synchronization wrapper`

  - methods protected by a lock

  ```Java
  List<E> synchArrayList = Collections.synchronizedList(new ArrayList<E>());
  Map<K, V> synchHashMap = Collections.synchronizedMap(new HashMap<K, V>());

  // no threads access the data structure
  // through the original unsynchronized methods.
  synchronized (synchHashMap) { // same codes for for-each loop
    Iterator<K> iter = synchHashMap.keySet().iterator();
    // throws ConcurrentyModificationException
    while (iter.hasNext()) ... ;
  }
  ```

- better use `java.util.concurrent` package ✅
- one exception is an array list frequently mutated.
  - a synchronized `ArrayList` outperform a `CopyOwnWriteArrayList`

# 12.6 Tasks and Thread Pools

- constructing a new threads is somewhat expensive. it's not a good a idea to create a large number of short-lived threads. use a **thread-pool**

## 12.6.1 Callables and Futures

- `Callable` is very similar to a `Runnable`, but it returns a value.
  ```Java
  public interface Callable<V> {
    V call() throws
  }
  ```
- A `Future` holds the **result** of an asynchronous computation
  - the owner of the `Future` object can object the result when it is ready.

| methods                              | description                                                                                       |
| ------------------------------------ | ------------------------------------------------------------------------------------------------- |
| `V get()`                            | blocks until the computation is finished.                                                         |
| `V get(long timeout, TimeUnit unit)` | blocks, but it throws a `TimeoutException` if the call timed out before the computation finished. |
| `void cancel(boolean mayInterrupt)`  | not guarantee, just a notification                                                                |
| `boolean isCancelled()`              |                                                                                                   |
| `boolean isDone()`                   | `false` if the computation is in progress, `true` if it's finished.                               |

- one way to execute a `Callable` is to use `FutureTask` implementing both the `Future` and `Runnable` interfaces.
  ```Java
  Callable<Integer> task = ...;
  var futureTask = new FutureTask<Integer>(task);
  var t = new Thread(futureTask); // It's a Runnable
  t.start();
  ...
  Integer result = task.get(); // It's a Future.
  ```

## 12.6.2 Executors

- The `Executors` class has a number of **static factory methods** for constructing thread pools

| Methods                            | Description                                                                                                                             |
| ---------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| `newCachedThreadPool`              | New threads are created as needed; idle threads are kept for 60 seconds                                                                 |
| `newFixedThreadPool`               | The pool contains a fixed set of threads; idle threads are kept indefinitely                                                            |
| `newWorkStealingPool`              | A pool suitable for **fork-join** tasks in which complex tasks are broken up into simpler tasks and idle threads "steal" simpler tasks. |
| `newSingleThreadExecutor`          | A "pool" with a single thread that executes the submitted tasks **sequentially**                                                        |
| `newScheduledThreadPool`           | A fixed-thread "pool" for scheduled execution.                                                                                          |
| `newSingleThreadScheduledExecutor` | A single-thread "pool" for scheduled execution.                                                                                         |

- Use `newCachedThreadPool` when you have threads short-lived or spend lots of time blocking
- For optimum speed, the number of concurrent threads is the number of processor cores. `newFixedThreadPool`
- The single-thread executor `newSingleThreadExecutor` is useful for performance analysis.
- three methods above return an object of `ThreadPoolExecutor` class implementing the `ExecutorService` interface.

- You can submit a `Runnable` or `Callable` to an `ExecutorService` with following methods
  - `Future<T> sumbit(Callable<T> task)`
  - `Future<?> sumbit(Runnable task)`
    - `get()` method will return `null`, other methods `isDone()`, `cancel()`, `isCancelled()` are fine.
  - `Future<T> sumbit(Runnable task, T result)`
    - `get()` method will return `result` object **upon completion**.
- done with a thread pool

  - `shutdown()`, An executor then accepts no new tasks. When all tasks are finished, the threads in the pool die.
  - `shutdownNow()`, the pool then cancels all tasks that have not yet begun.

- `ScheduledExecutorService` interface has methods for scheduled or repeated execution of tasks.
  - `newScheduledThreadPool` and `newSingleThreadScheduledExecutor` methods return objects implementing `ScheduledExecutorService`
  - `ScheduledFuture<V> schedule(Callable<V> task, long time, TimeUnit unit)`
  - `ScheduledFuture<V> schedule(Runnable task, long time, TimeUnit unit)`
  - `ScheduledFuture<V> scheduleAtFixedRate(Runnable task, long initialDelay, long period, TimeUnit unit)`
    - schedules the given task to run periodically, every `period` units, after the initial delay has elapsed.
  - `ScheduledFuture<V> scheduleWithFixedDelay(Runnable task, long initialDelay, long period, TimeUnit unit)`
    - schedules the given task to run periodically, with `Delay` units between completin of one invocation and the start of the next, after the initial delay has elapsed.

## 12.6.3 Controlling Groups of Tasks

- to control a group of related tasks

  - `shutdownNow()`: cancel all tasks in an executor.
  - `invokeAny()`: submits **all objects in a collection of `Callable` objects** and returns the result of a completed task.
  - `invokeAll()`: submits **all objects in a collection of `Callable` objects**, blocks until all of them complete, and returns a list of `Future` objects that represent the solutions to all tasks.

  ```Java
  List<Callable<T>> tasks = ...;
  List<Future<T>> results = executor.invokeAll(tasks);
  for (Future<T> result: results)
    processFurther(result.get());

  // more efficient way
  var service = new ExecutorCompletionService<T>(executor);
  for (Callable<T> task: tasks) service.submit(task);
  for (int i = 0; i < tasks.size(); i++)
    processFurther(service.take().get());
  ```

- (12.8) execution/ExecutorDemo.java

## 12.6.4 The Fork-Join Framework

- some app use a large number of threads that are mostly idle
  - web server uses one thread per connection
  - other app use one thread per processor core. (where **fork-join framework support**)
- suppose you decomposes a processing task into subtasks
  - extends `RecursiveTask<T>` or `RecursiveAction` class
  ```java
  class Counter extends RecursiveTask<Integer> {
    ...
    protected Integer compute() {
      if (to - from < THRESHOLD) {
        solve problem directly
      }
      else {
        int mid = (from + to) / 2;
        var first = new Counter(values, from, mid, filter);
        var second = new Counter(values, mid, to, filter);
        // recetives a number of tasks and blocks until all done.
        invokeAll(first, second);
        return first.join(). second.join(); // yields the results
      }
    }
  }
  ```
- fork-join framework uses an effective heuristic, called **work stealing**, for balancing the workload among available threads.
  - each work thread has a **deque** for tasks
  - A work thread pushes subtasks onto **head** of its own deque.
  - A idle thread will "steal" a tasks from the **tail** of another deque.
- (12.9) forkJoin/ForkJoinTest.java

# 12.7 Asynchronous Computations

- implement wait-free, or **asynchronous**, computations.

## 12.7.1 Completable Futures

- The `CompletableFuture` class implementing the `Future` interface. 类似 Promise

  - it also provides a second mechanism for obtaining the result.
  - register a `callback` that will be invoked (in some thread) with the result once available.

  ```Java
  CompletableFuture<String> f = ...;
  f.thenAccept(s -> Process the result string s);

  // example: fetch a web page asynchronously with the experimental HttpClient class
  HttpClient client = HttpClient.newHttpClient();
  HttpRequest request = HttpRequest.newBuilder(URI.create(urlString)).GET().build();
  CompletableFuture<HttpResponse<String>> f = client.sendAsync(request, BodyHandler.asString());
  ```

  - example of making a `CompletableFuture`
    - if omit the `executor`, task will be run on a default executor (ForkJoinPool.commonPool()) NOT RECOMMENDED ⭕️

  ```Java
  public CompletableFuture<String> readPage(URL url){
    return CompletableFuture.supplyAsync(() -> {
      try {
        return new String(url.openStream().readAllBytes(), "UTF-8");
      }
      catch (IOException e) {
        throw new UncheckedIOException(e);
      }
    }, executor);
  }
  ```

  - `CompletableFuture` can complete (`whenComplete()` method) in two ways: with a result, or with an uncaught exception

  ```Java
  f.whenComplete((s, t) -> {
    if (t == null) {
      Process the result s;
    }
    else {
      Process the Throwable t;
    }
  });
  ```

  - `complete()` and `completeExceptionally()` methods: manually set a completion value
    - safe to call `complete()` or `completeExceptionally()` on the same future in multiple threads. if the future is already completed, these calls has no effect.

  ```Java
  var f = new CompletableFuture<Integer>();
  executor.execute(() -> {
    int n = workHard();
    f.complete(n);
  });
  executor.execute(() -> {
    int n = workSmart();
    f.complete(n);
  });

  Throwable t = ...;
  f.completeExceptionally(t);
  ```

  - `isDone()` method tells whether a `Future` object has been complteted (both normally or with an exception).
  - Unlike a plain `Future`, `CompletableFuture` will not be interrupted by invoking `cancel()` method,
    - instead, it will be completed exceptional, with a `CancellationException`.

## 12.7.2 Composing Completable Futures

- The `CompletableFutre` class provides a mechanism for **composing** asynchronous tasks into a processing pipeline.

  ```Java
  CompletableFuture<String> contenst = readPage(url);
  CompletableFuture<List<URL>> imageURLs = contents.thenApply(this::getLinks);
  ```

- Adding an Action to a `CompletableFuture<T>` Object

  | Method            | Paramenter                 | Description |
  | ----------------- | -------------------------- | ----------- |
  | thenApply         | T -> U                     |             |
  | thenAccept        | T -> void                  |             |
  | thenCompose       | T -> CompletableFuture\<U> |             |
  | handle            | (T, Throwable) -> U        |             |
  | whenComplete      | (T, Throwable) -> void     |             |
  | exceptionally     | Throwable -> T             |             |
  | completeOnTimeout | T, long, TimeUnit          |             |
  | orTimeout         | long, TimeUnit             |             |
  | thenRunn          | Runnable                   |             |

  - for each method in the table, there are also two `Async` variants

    - one uses a shared `ForkJoinPool`
    - the other has an `Executor` paramter

- Combining Multiple Composition Objects

  | Method         | Paramter                          | Description |
  | -------------- | --------------------------------- | ----------- |
  | thenCombine    | CompletableFuture<U>, (T, U) -> V |             |
  | thenAcceptBoth |                                   |             |
  | runAfterBoth   |                                   |             |
  | applyToEither  |                                   |             |
  | acceptEither   |                                   |             |
  | runAfterEither |                                   |             |
  | static allOf   |                                   |             |
  | static anyOf   |                                   |             |

  - first three
  - next three
  - `allOf` and `anyOf`

- (12.10) completableFutures/CompletableFutureDemo.java

## 12.7.3 Long-Running Tasks in User Interface Callbacks

omit

# 12.8 Processes

## 12.8.1 Building a Process

## 12.8.2 Running a Process

## 12.8.3 Process Handles
