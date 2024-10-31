# Mandali - ThreadSafetyAnalyzer

**Mandali** is a Java library designed to detect potential thread safety issues within Java classes. The name **Mandali** is inspired by the Indonesian words "Man" (safe) and "Dali" (controlled), symbolizing **safety under control**. This library helps identify the use of collections that may not be thread-safe in multi-threaded environments and automatically detects deadlocks when running.


## Features

- **Unsafe Collection Detection**: Identifies commonly unsafe collections in multi-threaded contexts, such as `ArrayList`, `HashMap`, and `StringBuilder`.
- **Thread-safe Alternatives Recommendation**: Suggests safe alternatives to replace unsafe collections or objects.
- **Deadlock Detection**: Automatically detects deadlocks in the system and displays detailed information about the locked threads.

## Installation
### In Gradle

```groovy
repositories {
    maven { url 'https://repo.repsy.io/mvn/hangga/repo' }
}

dependencies {
    implementation 'io.mandali:mandali:1.0.4-SNAPSHOT'
}
```
### In Maven
```xml
<repositories>
    <repository>
        <id>mandali-repo</id>
        <url>https://repo.repsy.io/mvn/hangga/repo</url>
    </repository>
</repositories>

<dependencies>
    <dependency>
        <groupId>io.mandali</groupId>
        <artifactId>mandali</artifactId>
        <version>1.0.4-SNAPSHOT</version>
    </dependency>
</dependencies>
```

## Usage

1. **Initialize the Class**: Create an instance of `ThreadSafetyAnalyzer` with the class you want to analyze.
2. **Run Analysis**: Call the `start()` method to initiate analysis on the specified class.

### Example Usage

```kotlin
private var threadSafetyChecker = ThreadSafetyAnalyzer(this.javaClass)

@Test
fun `example of deadlock`() {
    val account1 = Account("Hangga", 1000)
    val account2 = Account("John", 1000)
    val account3 = Account("Alice", 2000)

    // Transfer from account1 to account2
    thread {
        account1.transfer(account2, 100)
    }.join(10) // as a simulation of the time required

    // Transfer from account2 to account1
    thread {
        account2.transfer(account1, 200)
    }.join(20)

    // Transfer from account3 to account1
    thread {
        account3.transfer(account1, 1000)
    }.join(500)

    threadSafetyChecker.start()
}

val list = ArrayList<Int>()

@Test
fun `example thread-unsafe using HashMap`() {
    val map = HashMap<Int, Int>()

    val threads = List(10) { index ->
        thread {
            for (i in 0 until 1000) {
                map[i] = index
            }
        }
    }

    threads.forEach { it.join() }
    threadSafetyChecker.start()
}
```

### Sample Output

The library will display warnings and thread-safe usage suggestions for identified unsafe objects, such as:
```plaintext
[Mandali-ThreadSafetyAnalyzer]: Found HashMap on (MyClass.java:12)
[Mandali-ThreadSafetyAnalyzer]: Consider using ConcurrentHashMap or Collections.synchronizedMap(new HashMap<>())
```

If a deadlock is detected, detailed information is displayed:
```plaintext
[Mandali-ThreadSafetyAnalyzer]: Deadlock detected: [id:14, name:Thread-2, owner:Thread-1]
```

## API

### Constructor
- `ThreadSafetyAnalyzer(Class<?> clazz)`: Creates a new instance to analyze the specified class.

### Methods
- `void start()`: Starts the analysis on the given class, checking for thread-unsafe collections and detecting deadlocks.
- `void detectDeadlock()`: Explicitly runs deadlock detection in the system.

## Limitations

- Currently, `ThreadSafetyAnalyzer` only supports deadlock detection and basic analysis of thread-unsafe collection usage.
- It detects source code files based on standard directory structures (`src/main/java` or `src/test/java`). Other directory structures may require adjustments.

## License

ThreadSafetyAnalyzer is a proprietary, closed-source library. Redistribution and modification are restricted. For further information, please contact the development team.
```

This version acknowledges its proprietary nature and provides guidance for internal access and use.