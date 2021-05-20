# 1.1 From Iterating to Stream Operations

```Java
// iterate on a List<String> words
int count = 0;
for (String w: words) {
    if (w.length() . 12) count++;
}
// same operations with stream
long count = words.stream().filter(w -> w.length() > 12).count();
```

- Streams follow the "What, not how" principle. in the example above, we describe what needs to be done: get the long words and count them.
- significant differences between `Stream` and `Collections`

  1. A Stream does not store its elements
     - stored in underlying colletion OR generated on demand.
  2. Stream operations don't mutate their source.
     - `filter` method yields a new stream
  3. Stream operations are **lazy** when possible, they are not executed until their result is need.

- the typical Workflow with streams 结合上面的例子来看

  1. Create a stream (1.2)
  2. Specify **intermediate operations** for transforming the initial stream into others, possibly in mutiple steps. (1.3, 1.4, 1.5)
  3. Apply **a terminal operation** to produce a result. (1.6, 1.8, 1.9)
     - this operation forces the execution of the lazy operations that precede it.

- (1.1) streams/CountLongWords.java

| `java.util.Stream<T>`                      | description |
| ------------------------------------------ | ----------- |
| `Stream<T> filter(Predicate<? super T> p)` |             |
| `long count()`                             |             |

| `java.util.Collection<E>`            | description |
| ------------------------------------ | ----------- |
| `default Stream<E> stream()`         |             |
| `default Stream<E> parallelStream()` |             |

# 1.2 Stream Creation

- turn any collection into stream with `stream` and `parallelStream` methods
- Use static `Stream.of` for an array
- Use `Arrays.stream(array, from, to)` to make a stream from a part of an array.
- Use `Stream.empty()` to make a stream with no elements.

```Java
// split returns a String[] array
Stream<String> words = Stream.of(contents.split("\\PL+"));

// of method has a varargs parameter
Stream<String> song = Stream.of("gently", "down", "the", "stream");
```

- two static method for making infinite streams.
  - `generate` method takes a function with no arguments (an object of `Supplier<T>` interface)
  - `iterate` method takes a "seed" value and a function (a `UnaryOperator<T>`) and repeatly applies the function to the previous result.
    - to produce a finite stream, add a `predicate` that specifies when the iteration should finish.

```Java
Stream<String> echos = Stream.generate(() -> "Echo");
Stream<String> randoms = Stream.generate(Math::random);

Stream<BigInteger> integers = Stream.iterate(BigInteger.ZERO, n -> n.add(BigInteger.ONE));

// finite stream
var limit = new BigIteger("10000000");
Stream<BigInteger> integers = Stream.iterate(BigInteger.ZERO,
                n -> n.compareTo(limit) < 0, // predicate
                n -> a.add(BigInteger.ONE));
```

- `Stream.ofNullable` makes a really short stream from an object

  - empty stream of the `object == nul`l`, otherwise, contains only one element just the `object`
  - useful in conjunction with `flatMap`

- Other API methods yielding streams

```Java
Stream<String> words = Pattern.compile("\\PL+").splitAsStream(contents);

// yiels a stream of tokens of a scanner
Stream<String> words = new Scanner(contents).tokens();

try (Stream<String> lines = Files.lines(path)) {
    // Process lines
}
```

- turn an `Iterable` that's not a collection into a stream
  - `StreamSupport.stream(iterable.spliterator(), false);`
- hava an `Iterator` and want a stream of its resutls

  - `StreamSupport.stream(Spliterators.spliteratorUnknownSize(iterator, Spliterator.ORDERED), false);`

- We should not modify the underlying collection of a stream
  - **noninterferences**

```Java
List<String> wordList = ...;
Stream<String> words = wordList.stream();
wordList.add("END"); // will work but NOT recommended
long n = words.distinct().count();

Stream<String> words = wordList.stream();
words.forEach(s -> if(s.length() < 12) wordList.remove(s)); // ERROR - interference
```

- (1.2) streams/CreatingStreams.java

# 1.3 The `filter`, `map` and `flatMap` Methods

- A **stream transformation** produces a stream whose elements are derived from those of another stream.

```Java
// filter
Stream<String> longWords = words.stream().filter(w -> w.length() > 12);

// map
Stream<String> lowercaseWords = words.stream().map(String::toLowerCase);
Stream<String> firstLetters = words.stream().map(s -> s.substring(0, 1));

// map and flatMap, suppose codePoints return Stream<String>
Stream<Stream<String>> result = words.stream().map(w -> codePoints(w));
Stream<String> result = words.stream().flatMap(w -> codePoints(w));
```

| `java.util.stream.Stream`                                                          | description                                                                               |
| ---------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------- |
| `Stream<T> filter(Predicate<? super T> predicate)`                                 | yields a new stream with those elements that fulfill the predicate.                       |
| `<R> Stream<R> map(Function<? super T, ? extends R> mapper)`                       | the mapper will be applied to each element and yields a new stream containing the results |
| `<R> Stream<R> flatMap(Function<? super T, ? extends Stream<? extends R>> mapper)` | flatten a stream of stream to a single stream                                             |

# 1.4 Extracting Substreams and Combining Streams

| `java.util.stream.Stream`                                                   | description                                                                            |
| --------------------------------------------------------------------------- | -------------------------------------------------------------------------------------- |
| `Stream<T> limit(long maxSize)`                                             | yields a stream with up to `maxSize` of the initial elements from this stream          |
| `Stream<T> skip(long n)`                                                    | skip the initial `n` elements of this stream                                           |
| `Stream<T> takeWhile(Predicate<? super T> predicate)`                       |                                                                                        |
| `Stream<T> dropWhile(Predicate<? super T> predicate)`                       |                                                                                        |
| `static <T> Stream<T> concat(Stream<? extends T> a, Stream<? extends T> b)` | yields a stream whose elements are the elements of `a` followed by the elements of `b` |

# 1.5 Other Stream Transformations

| `java.util.stream.Stream`                            | description                                                                                             |
| ---------------------------------------------------- | ------------------------------------------------------------------------------------------------------- |
| `Stream<T> distinct()`                               | yields a stream of the distinct elements of this stream.                                                |
| `Stream<T> sorted()`                                 | elements are instances of a class implementing `Comparable`                                             |
| `Stream<T> sorted(Comparator<? super T> comparator)` | yields as tream whose elements are the elements of this stream in sorted order.                         |
| `Stream<T> peek(Consumer<? super T> action)`         | yields a stream with the same elements as this stream, passing each element to `action` it is consumed. |

# 1.6 Simple Reductions

- **reductions**, terminal operations,
  - they reduce the stream to a nonstream value that can be used in your program.
  - these methods return an `Optional<T>` value that either wraps the answer or indicates that there is none.

```Java
Optional<String> largest = words.max(String::compareToIgnoreCase);
System.out.println("largest: " + largest.orElse(""));

// find first match
Optional<String> startsWithQ = words.filter(s -> s.startWith("Q")).findFirst();

// find any match, effective when you parallelize the stream
Optional<String> startsWithQ = words.parallel().filter(s -> s.startWith("Q")).findFirst();

//check if any match exists
boolean aWordStartsWithQ = words.parallel().anyMatch(s -> s.startsWWith("Q"));
```

| `java.util.stream.Stream`                           | description |
| --------------------------------------------------- | ----------- |
| `Optional<T> max(Comparator<? super T> comparator)` |             |
| `Optional<T> min(Comparator<? super T> comparator)` |             |
| `Optional<T> findFirst()`                           |             |
| `Option<T> findAny()`                               |             |
| `boolean anyMatch(Predicate<? super T> predicate)`  |             |
| `boolean allMatch(Predicate<? super T> predicate)`  |             |
| `boolean noneMatch(Predicate<? super T> predicate)` |             |

# 1.7 The Optional Type

An `Optional<T>` object is a wrapper for either an object of type `T` or no object.

## 1.7.1 Getting an Optional Value

- three ways to **produce an alternative** if no value is present

```Java
String  result = optionalString.orElse("");
String result = optionalString.orElseGet(() -> System.getProperty("myapp.default"));
String result = optionalString.orElseThrow(IllegalStateException::new);
```

| `java.util.Optional`                                                          | description |
| ----------------------------------------------------------------------------- | ----------- |
| `T orElse(T other)`                                                           |             |
| `T orElseGet(Supplier<? extends T> other)`                                    |             |
| `<X extends Throwable> T orElseThrow(Suppier<? extends X> exceptionSupplier)` |             |

## 1.7.2 Consuming an Optional Value

- one ways to **consume** the value only if it is present.
  - `optionalValue.ifPresent(v -> results.add(v));`
- one action for **presence**, one action for **absence**
  ```Java
  optionalValue.ifPresentOrElse(
    v -> System.out.println("Found " + v),
    () -> logger.warning("No match")
  );
  ```

## 1.7.3 Pipelining Optional Values

- transforming the value inside an `Optional` by using `map` method
  - `Optional<String> transformed = optionalString.map(String::toUpperCase);`
  - if the `optionalString` is empty, then `transformed` is also empty;
- we can add to a list if it's present
  - `optionalValue.map(results::add);`
  - if `optionalValue` is empty, nothing happens.
- use `filter` method before or after transfoming it
  - `Optional<String> transformed = optionalString.filter(s -> s.length() >= 8).map(String::toUpperCase);`
- substitute an alternative `Optional` for an empty `Optional` with `or` method
  - `Optional<String> result = optionalString.or(() -> alternatives.stream().findFirst());`
  - **computed lazily**
    - lambda will be executed only if `optionalString` is empty.

## 1.7.4 How Not to work with Optional Values

You should use `Optional` correctly

```Java
Optional<T> optionalValue = ...;
optionalValue.get().someMethod(); // not good
// no safer than
T value = ...;
value.someMethod();

if (optionalValue.isPresent()) optionalValue.get().someMethod();
// no safer than
if (value != null) value.someMethod();
```

- Tips:
  1. A variable of type `Optional` should **never** be **null**.
  2. Don't use fields of type `Optional`. The cost is an additional object.
     - Inside a class, using `null` for an absent field is manageable.
  3. Don't put `Optional` objects in a set, and don't use them as keys for a map. Collect the values instead.

## 1.7.5 Creating Optional Values

- static method `Optional.of(result)` and `Optional.empty()`
- `Optional.ofNullable(object)`
  - returns `Optional.of(object)` if object is not null
  - otherwise return `Optional.empty()`

```Java
public static Optional<Double> inverse(Double x) {
  return x == 0 ? Optional.empty() : Optional.of(1/x);
}
```

## 1.7.6 Composing Optional Value Functions with flatMap

- Suppose `s.f()` returns `Optional<T>`, then T.g() returns `Optional<U>`
  - `Optional<U> result = s.f().flatMap(T::g);`
  - if `s.f()` is present, `T.g()` will called. otherwise, `Optional<U>` is empty
- You can build a pipeline of steps, simply by chaining calls to `flatMap`, that will succeed only when all parts do.
  - `Optional<Double> result = Optional.of(-4.0).flatMap(Demo::inverse).flatMap(Demo::squareRoot);`

## 1.7.7 Turning an Optional into an Stream

- `stream()` method turns an `Optional<T>` into a `Stream<T>` with zero or one element.
  - `isPresent()` + `get()` for `Optional<T>` object is NOT recommended!

```Java
// Suppose Optional<User> lookup(String id) method
Stream<String> ids = ...;
Stream<User> users = ids.map(Users::lookup).filter(Optional::isPresent).map(Optional::get); ❌
Stream<User> users = ids.map(Users::lookup).flatMap(Optional::stream); ✅

// Suppose User lookup(String id) method, return null if no valid result.
Stream<User> users = ids.map(Users::lookup).filter(Objects::nonNull); // option 1
Stream<User> users = ids.flatMap(id -> Stream.ofNullable(Users.lookup(id))); // option 2
Stream<User> users = ids.map(Users::lookup).flatMap(Stream::ofNullable); // option 3
```

- (1.3) optional/OptionalTest.java

# 1.8 Collecting Results

- When doen with a stream, try to look at the results
- old-fashioned `iterator` to visit all elements (NOT recommended ❌)
- `forEach()` apply a function to each element: `stream.forEach(System.out::println)`
  - On a **parallel stream**, the `forEach` method traverses elements in arbitrary order, can use `forEachOrdered()` method instead
- use `toArray()` to get an array of the stream elements: `String[] result = stream.toArray(String[]::new)`
  - `Object[]` by default unless pass array constructor.
- ⚠️ `collect()` method taking an instance of the `Collector` interface. 最常用
  - A `collector` is an object that accumulates elements and produces a result.
  - `Collectors` class provides lots of factory methods for common collectors.
    - `Collectors.toList()`
    - `Collectors.toSet()`
    - `Collectors.toCollection(TreeSet::new)` - control specific
    - `Collectors.joining()` - collect all strings in a stream by concatenating them, **can add delimiter**
    - `Collectors.summarizing(Int|Long|Double)` methods return type `(Int|Long|Double)SummaryStatistics`

```Java
List<String> result = stream.collect(Collectors.toList());

Set<String> result = stream.collect(Collectors.toSet());

// specify the collection type
TreeSet<String> result = stream.collect(Collectors.toCollection(TreeSet::new));

result = noVowels().limit(10).collect(Collectors.joining(", "));

IntSummaryStatistics summary = noVowels().collect(Collectors.summarizingInt(String::length));
double averageWordLength = summary.getAverage();
double maxWordLength = summary.getMax();
```

- (1.4) collecting/CollectingResults.java

# 1.9 Collecting into Maps

- turn a stream into a map, use `Collectors.toMap()`
  - **required** first two functional arguments produce keys and values
    - when value is actual elements: `Function.identity()`
  - **optional** resolve the conflict of duplicate keys
    - `(existingValue, newValue) => { return modifiedValue; }`, otherwise throw `IllegalStateException`
  - **optional** specify the implementation, `TreeSet::new`

```Java


Map<String, Set<String>> countryLanguageSets = Locales.collect(Collectors.toMap(
    Locale::getDisplayCountry,
    l -> Collections.singleton(l.getDisplayLangauge()),
    (a, b) -> {
        var union = new HashSet<String>(a);
        union.addAll(b);
        return union; // union of a and b
    },
    TreeMap::new
));
```

- (1.5) collecting/CollectingIntoMaps.java

# 1.10 Grouping and Partitioning

- `groupingBy`: forming groups of values with same characteristics
  - The function `Locale::getCountry` is the **classifer function** of the grouping.

```Java
Map<String, List<Locale>> countryToLocales =
    locales.collect(Collectors.groupingBy(Locale::getCountry));
```

- When the classifier function is a predicate function (functions returning `bolean` values)
  - the stream elements are partitioned into two lists, `true` or `false` key
  - In such a case, use `paritioningBy` is more efficient

```Java
Map<Boolean, List<Locale>> englishAndOtherLocales = locales.collect(
  Collectors.partioningBy(l -> l.getLanguage().equals("en"));
);
List<Locale> englishLocales = englishAndOtherLocales.get(true);
```

# 1.11 Downstream Collectors

- `groupingBy` method yields a map whose value are lists.

  - You can supply a `downstream collector` to process those lists

  ```Java
  // when you want set instead of list
  Map<String, Set<Locale>> countryToLocaleSet =
      locales.collect(groupingBy(Locale::getCountry, toSet()));
  ```

- other collectors used here for reducing elements to numbers

  - `counting()`
  - `summing(Int|Long|Double)` produce a sum

  ```Java
  Map<String, Integer> countryToLocaleCounts = locales.collect(
    groupingBy(Locale::getCountry, summingInt(City::getPopulation))
  );
  ```

  - `maxBy` and `minBy` take a **comparator** and produce max and min of the downstream elements

  ```Java
  Map<String, Optional<City>> stateToLargestCity = locales.collect(
    groupingBy(City::getState, maxBy(Comparator.comparing(City::getPopulation)))
  );
  ```

- `collectingAndThen` collector adds **a final processing step** behind a collector.
  ```Java
  Map<Character, Integer> stringCountsByStartingLetter = strings.collect(
    groupingBy(s -> s.charAt(0), collectingAndThen(toSet(), Set::size))
  );
  ```
- `mapping` applies a function to each collected element and passes the results to a downstream collector.

  - `flatMapping` return streams

  ```Java
  Map<Character, Integer> stringLengthsByStartingLetter = strings.collect(
    groupingBy(s -> s.charAt(0), mapping(String::length, toSet()))
  );

  // gathering a set of all languages in a country
  Map<String, Set<String>> countryToLangauges = locales.collect(
    groupingBy(Locale::getDisplayCountry,
      mapping(Locale::getDisplayLanguage, toSet()))
  );
  ```

- `summarizing(Int|Long|Double)`, grouping or mapping function return `int`, `long`, `double`

  - return a summary statistics object providing, sum, count, average, min, max.

  ```Java
  Map<String, IntSummaryStatistics> stateToCityPopulationSummary = cities.collect(
    groupingBy(City::getState, summarizingInt(City::getPopulation))
  );
  ```

- `filtering`

  ```Java
  Map<String, Set<City>> largeCitiesByState = cities.collect(
    groupingBy(City::getState, filtering(c -> c.getPopulation() > 500_000, toSet()))
  );
  ```

  ...

- (1.6) collecting/DownstreamCollectors.java

# 1.12 Reduction Operations

- the `reduce` method

  - a general mechanism for computing a value from a stream.
  - the **simplest form** takes a binary function and keeps applying it, starting with the first two elements.
    - many operations in practice: sum, product, string concetenation, max and min, set union or intersection

  ```Java
  List<Integer> values = ...;
  Optional<Integer> sum = values.streams().reduce((x, y) -> x + y); // Integer::sum
  ```

- operations must be **associative** when using reduction with **parallel stream**
  - It shouldn't matter in which order you combine the elements.
- set the **start value** (aka identity) of the computation

  - no longer Optional, when the stream is empty, return **start value**

  ```Java
  Integer sum = value.streams().reduce(0, (x, y) -> x + y);
  ```

- to get the sum of length of an array of strings

  - you need to provide **an accumulator** first (parallel by default, so will generate multiple reuslts)
    - `BiFunction<T, U, T>` interface - T, U params, returns T
  - then you need to combine their results. (**an combiner**)
    - for **sequential stream**, combiner will **not be executed**.
    - `BinaryOperator<T> extends BiFunction<T,T,T>` interface - T, T params, returns T

  ```Java
  // int total, String word, so it's a accumulator
  int results = words.reduce(0, (total, word) -> total + word.length(), Integer::sum);

  // HOWEVER, usually, use map is more efficient and clearner
  int results = words.mapToInt(String::length).sum();
  ```

- sometimes, `reduce` is not general enough. ❗️

  - suppose you want to collect results in a **BitSet** parallelly. because reduce is not thread-safe, you should use **collect** instead with three arguments ❗️
    1. A **supplier** to make new instances of the target object
       - `Supplier<R>` interface, no params, return R
    2. An **accumlator** that adds an element to the target. (executed in parallel)
       - `BiConsumer<R, T>` interface, no returns, R absorb T
    3. A **combiner** that merges two objects into one.
       - for **sequential stream**, combiner will **not be executed**. 这里跟 reducer 类似
       - `BiConsumer<R, T>` interface, no returns, R absorb T

  ```Java
  BitSet result = stream.collect(BitSet::new, BitSet::set, BitSet::or);
  ```

# 1.13 Primitive Type Streams

- Stream library has specialized types `IntStream`, `LongStream`, `DoubleStream` that store primitive types directly, withou using wrappers.
  - store `short`, `char`, `byte`, `boolean`, use `IntStream`
  - store `float`, use `DoubleStream`
- besides `generate` and `iterate`, `IntStream` and `LongStream` have static methods `range` and `rangeClosed` to generate integer ranges with step size one.

```Java
IntStream zeroToNinetyNine = IntStream.range(0, 100); // Upper bound is excluded
IntStream zeroToHundred = IntStream.rangeClosed(0, 100); // Upper bound is included
```

- The `CharSequence` interface has methods `codePoints` and `chars` that yields an `IntStream` of the Unicode codes of the characters or of the code units in the UTF-16 encoding.

- can use `mapToInt`, `mapToLong`, `mapToDouble` methods to transform a object stream to primitive stream

```Java
Stream<String> words = ...;
IntStream lengths = words.mapToInt(String::length);
```

- use `boxed()` method to turn a primitive stream to a wrapper object stream

  - `Stream<Integer> integers = IntStream.rang(0, 100).boxed();`

- notable difference between primitve stream and object stream
  1. `toArray()` return primitive type arrays
  2. yields `OptionalInt`, `OptionalLong`, `OptionalDouble` instread of `Optional` class
     - they have `getAsInt`, `getAsLong`, `getAsDouble` instead of `get` method
  3. they have `sum`, `average`, `max`, `min`, not defined for object streams
  4. `summaryStatistics` method yields an object of `IntSummaryStatistics`, `LongSummaryStatistics`, `DoubleSummaryStatistics`
     - can report sum, count, average, min, max.
- (1.7) streams/PrimitiveTypeStreams.java

# 1.14 Parallel Streams

- parallel stream is easy for **bulk operation**.
- `paralleleStream()` turn any collection into a parallel stream
- `parallel()` turn any sequential stream into a parallel one.
- any functions passed to to parallel stream should be safe to execute in parallel.
  - best way is to stay away from mutatable state, otherwise **race condition** may occur
- `unordered()` indicating that you are not interested in ordering.
  - dropping orders may speed up some operations, `limit()`, `distinct()`
  - `Stream<String> sample = words.parallelStream().unordered().limit(n);`
- merging maps concurrently

  - `Collectors.groupingByConcurrent` method uses a shared concurrent map.

  ```Java
  Map<Integer, List<String>> result = words.parallelStream().collect(
    // values aren't ordered in stream order
    Collectors.groupingByConcurrent(String::length)
  );

  Map<Integer, Long> wordCounts = words.parallelStream().collect(
    // use a downstream collector independent of the ordering
    groupingByConcurrent(String::length, counting())
  );
  ```

- keep in mind (don't turn all streams into parallel)

  1. **a substantial overhead** to parallelization that will only pay off for **very large data sets**.
  2. Parallelizing a stream is only a win if the underlying data source can be effectively split into multiple parts
  3. The thread pool that is used by parallel streams can be starved by blocking operations such as file I/O or network access.

- Parallel streams work best with **huge in-memory collections of data** and **computationally intensive processing**.

- if wanting to parallelize stream based on random numbers

  - don't use streams from `Random.ints`, `Random.longs`, or `Random.doubles`, these don't split
  - use `ints`, `longs`, `doubles` methods of `SplittableRandom` class

- （1.8) parallel/ParallelStreams.java
