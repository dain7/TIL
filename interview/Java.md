# Basic 

##### 1. What’s the difference between String, StringBuffer, and StringBuilder?
**String** is an immutable class. In older JDKs, the recommandation when programmatically building a **String** was to use **StringBuffer** since this was optimized to concatenate multiple Strings together.
**StringBuffer** is synchronized, so it's thread-safe but slower. In contrast, **StringBuilder** is not synchronized, making it faster in single-threaded environments.

##### 2. What is the difference between final, finalize and finally?
**final** is a keyword that indicates a class, method or field cannot be modified or overridden.
**finalize** is a method that is called by the garbage collector before an object is removed from memory.
**finally** is a block used in excetpion handling, and it excutes regardless of whether an exception is thrown or not.

##### 3. How does Garbage Collection prevent a Java application from going out of memory?
**Garbage Collection** automatically removes objects that are no longer needed.
it helps clean up the heap and prevents OutOfMemoryError by freeing up space for new objects.

##### 4. What’s the difference between a ClassNotFoundException and NoClassDefFoundError?
**ClassNotFoundExction** occurrs when a class is not found in the classpath during runtime.
**NoClassDefFoundError** occurrs when a class was avaliable(existed) at compile time but is missing during runtime.

##### 5. Why isn’t String‘s .length() accurate?
The length() methods returns the number of UTF-16 code units, not the actual number of characters.
So, if your string contains emoji or certain characters from Asian languages, the count maybe different from what you visually expect.
If you want to get the actual number of characters, you can use codePointCount() instead of length().

##### 6. Given two double values d1, d2, why isn’t it reliable to test their equality using: d1 == d2
It's not reliable to compare double values using ==(equals equals) because of floating-point precision issues.
The most reliable way is to use 'Double.compare(d1, d2) == 0'.

##### 7. What is the problem with this code: final byte[] bytes = someString.getBytes();
The code relies on the default JVM charset, which can vary depending on the operating system.
For example, on most Windows installations, the default charset is CP1252, whereas on Linux, it is typically UTF-8.
As a result, even a simple string like “é” can produce different byte arrays depending on whether the code is run on Windows or Linux.

```java
// right code
final byte[] bytes = someString.getBytes(StandardCharsets.UTF_8);
```

##### 8. What is the JIT (just in time compiler)?
JIT, or Just-In-Time compiler, is a feature of the JVM that improves performance by converting bytecode into machine code at runtime. This allows frequently executed code to run much faster.

##### 9. How do you make this code print 0.5 instead of 0?: final double d = 1 / 2;
In the code, both values are integers, so the division results in an integer. 
To get a decimal result, you need to make at least one of them a floating-point number like double.

##### 10. what is the inferred type of the method reference System.out::println?
Consumer: 입력값을 받아 뭔가를 소비하고 결과는 반환X
It is an IntConsumer.

##### 11. What is the problem with this code?
```java
final Path path = Paths.get(...);
Files.lines(path).forEach(System.out::println);
```
Files.lines() opens the file and keeps it open while streaming lines.
If you don't close the stream properly, it can lead to a resource leak.

To avoid this, you should use a try-with-resources block