# 🧵 Java String — Complete Tutorial

> A comprehensive guide to `String`, `StringBuilder`, `StringBuffer`, memory management, internals, and all methods.

---

## 📌 Table of Contents

1. [What is a String?](#1-what-is-a-string)
2. [String Memory Management & JVM Architecture](#2-string-memory-management--jvm-architecture)
3. [String Pool (Intern Pool)](#3-string-pool-intern-pool)
4. [Immutability & `final` Keyword](#4-immutability--final-keyword)
5. [String Properties](#5-string-properties)
6. [String Initialization & Constructors](#6-string-initialization--constructors)
7. [== vs .equals() — Deep Dive](#7--vs-equals--deep-dive)
8. [Overridden Methods in String](#8-overridden-methods-in-string)
9. [All String Methods — Detailed](#9-all-string-methods--detailed)
10. [Type Conversion — String ↔ Other Types](#10-type-conversion--string--other-types)
11. [char, char[], and String Internals](#11-char-char-and-string-internals)
12. [String to char Array & Back](#12-string-to-char-array--back)
13. [Compile-time vs Runtime Strings](#13-compile-time-vs-runtime-strings)
14. [Reading String Input in Java](#14-reading-string-input-in-java)
15. [Spaces, Whitespace Handling](#15-spaces-whitespace-handling)
16. [Compile Left-to-Right & Right-to-Left](#16-compile-left-to-right--right-to-left)
17. [String Interning Guidelines](#17-string-interning-guidelines)
18. [StringBuilder — Complete Guide](#18-stringbuilder--complete-guide)
19. [StringBuffer — Complete Guide](#19-stringbuffer--complete-guide)
20. [StringBuilder vs StringBuffer vs String](#20-stringbuilder-vs-stringbuffer-vs-string)

---

## 1. What is a String?

A `String` in Java is a **sequence of characters** represented as an object of the `java.lang.String` class.

```java
String s = "Hello, World!";
```

### Key Facts:
- Strings are **objects**, not primitives.
- Stored internally as a `char[]` (before Java 9) or `byte[]` with encoding flag (Java 9+, **Compact Strings**).
- `String` class is in `java.lang` — **auto-imported**.
- Strings are **immutable** — once created, cannot be modified.

### Java 9+ Compact Strings:
```
// Before Java 9:
private final char[] value;

// Java 9+:
private final byte[] value;
private final byte coder; // LATIN1 = 0, UTF16 = 1
```
This saves ~50% memory for Latin-1 (ASCII) strings.

---

## 2. String Memory Management & JVM Architecture

### JVM Memory Areas

```
┌──────────────────────────────────────────────┐
│                   JVM Memory                  │
│                                               │
│  ┌────────────┐   ┌──────────────────────┐   │
│  │   Stack    │   │        Heap          │   │
│  │            │   │  ┌───────────────┐   │   │
│  │ s1 ──────────────► │ String Object │   │   │
│  │ s2 ──────────────► │ String Object │   │   │
│  │            │   │  └───────────────┘   │   │
│  │            │   │  ┌───────────────┐   │   │
│  │            │   │  │  String Pool  │   │   │
│  │            │   │  │  (PermGen /   │   │   │
│  │            │   │  │   Metaspace)  │   │   │
│  │            │   │  └───────────────┘   │   │
│  └────────────┘   └──────────────────────┘   │
└──────────────────────────────────────────────┘
```

### How Strings are Stored:

| Creation Method | Location | New Object? |
|----------------|----------|-------------|
| `"hello"` literal | String Pool | Only if not exists |
| `new String("hello")` | Heap | Always new |
| `.intern()` | String Pool | Only if not exists |

### Garbage Collection & Strings:
- **String Pool** entries are not GC'd easily (held by class loader).
- **Heap Strings** (via `new`) are GC'd when no references exist.
- From Java 7+, String Pool moved from **PermGen → Heap** (making it GC-eligible).

```java
String s1 = "hello";          // Goes to String Pool
String s2 = new String("hello"); // Goes to Heap
String s3 = s2.intern();      // Returns Pool reference

System.out.println(s1 == s3); // true  — same pool object
System.out.println(s1 == s2); // false — different locations
```

### Why Immutability Helps Memory:
- Same String literal can be **shared** safely across threads and references.
- No need to copy — just share the reference.
- Enables **String Pool optimization**.

---

## 3. String Pool (Intern Pool)

The **String Constant Pool** (SCP) is a special memory region that stores unique string literals to avoid duplication.

```java
String a = "Java";
String b = "Java";
String c = new String("Java");

// Memory:
// Pool: ["Java"] ← a and b point here
// Heap: [String@xyz] ← c points here

System.out.println(a == b);         // true
System.out.println(a == c);         // false
System.out.println(a == c.intern()); // true
```

### How Pool Works:
1. JVM checks if `"Java"` exists in pool.
2. If yes → return existing reference.
3. If no → create new entry, return reference.

---

## 4. Immutability & `final` Keyword

### String is Immutable because:
1. `private final char[] value` — the array is final (can't reassign).
2. No setter methods are exposed.
3. The class itself is `final` — cannot be subclassed.

```java
String s = "Hello";
s.concat(" World"); // Creates a NEW string — s is unchanged
System.out.println(s); // Still "Hello"

s = s.concat(" World"); // Now s points to NEW object
System.out.println(s); // "Hello World"
```

### final vs immutable:
```java
final String s = "Hi";
s = "Bye"; // ❌ Compile error — final reference

// But:
final StringBuilder sb = new StringBuilder("Hi");
sb.append(" Bye"); // ✅ OK — object is mutable, reference is final
```

### Security Benefits of Immutability:
- Safe to use as **HashMap keys** (hashCode never changes).
- **Thread-safe** — no synchronization needed.
- Class loading security (class name can't be tampered with).
- Safe for **network connections**, **file paths** — can't be changed after validation.

---

## 5. String Properties

| Property | Value |
|----------|-------|
| Package | `java.lang` |
| Superclass | `Object` |
| Interfaces | `Serializable`, `Comparable<String>`, `CharSequence` |
| Mutability | **Immutable** |
| Thread Safety | **Yes** (immutable) |
| Storage (Java 9+) | `byte[]` with coder |
| Pool Support | **Yes** |
| Nullable | **Yes** (reference type) |

```java
// Implements:
public final class String
    implements java.io.Serializable,
               Comparable<String>,
               CharSequence {
    private final byte[] value; // Java 9+
    private final byte coder;
    private int hash; // cached hashCode
}
```

---

## 6. String Initialization & Constructors

### All Ways to Create a String:

```java
// 1. String Literal (most common — uses pool)
String s1 = "Hello";

// 2. new keyword (heap — avoid unless needed)
String s2 = new String("Hello");

// 3. From char array
char[] chars = {'H', 'e', 'l', 'l', 'o'};
String s3 = new String(chars);

// 4. From char array with offset and count
String s4 = new String(chars, 1, 3); // "ell"

// 5. From byte array
byte[] bytes = {72, 101, 108, 108, 111};
String s5 = new String(bytes); // "Hello"

// 6. From byte array with charset
String s6 = new String(bytes, java.nio.charset.StandardCharsets.UTF_8);

// 7. From StringBuilder / StringBuffer
StringBuilder sb = new StringBuilder("Hello");
String s7 = new String(sb);

// 8. From another String
String s8 = new String("Hello");

// 9. Using valueOf
String s9 = String.valueOf(123);       // "123"
String s10 = String.valueOf(3.14);     // "3.14"
String s11 = String.valueOf(true);     // "true"
String s12 = String.valueOf('A');      // "A"
String s13 = String.valueOf(chars);    // "Hello"

// 10. Concatenation
String s14 = "Hel" + "lo"; // Compile-time constant → pool
String s15 = s1 + " World"; // Runtime → heap

// 11. String.format
String s16 = String.format("Hello %s, you are %d", "Yash", 21);
```

---

## 7. == vs .equals() — Deep Dive

### `==` — Reference Comparison
Checks if **both references point to the same object** in memory.

### `.equals()` — Content Comparison
Checks if **both strings have the same character sequence**.

```java
String s1 = "hello";
String s2 = "hello";
String s3 = new String("hello");
String s4 = new String("hello");

System.out.println(s1 == s2);         // true  (same pool object)
System.out.println(s1 == s3);         // false (different memory)
System.out.println(s3 == s4);         // false (different heap objects)

System.out.println(s1.equals(s2));    // true
System.out.println(s1.equals(s3));    // true
System.out.println(s3.equals(s4));    // true

// equalsIgnoreCase
System.out.println("HELLO".equalsIgnoreCase("hello")); // true

// compareTo (lexicographic)
System.out.println("apple".compareTo("banana")); // negative (a < b)
System.out.println("banana".compareTo("apple")); // positive
System.out.println("apple".compareTo("apple"));  // 0
```

### Memory Diagram:

```
Pool:  ["hello"] ←── s1, s2
Heap:  [String@100] ←── s3
       [String@200] ←── s4

s1 == s2  → compare 0x001 == 0x001 → true
s1 == s3  → compare 0x001 == 0x100 → false
s1.equals(s3) → compare content "hello" == "hello" → true
```

### ⚠️ Common Mistake:
```java
// NEVER compare strings with == in production
if (userInput == "admin") { } // ❌ WRONG — almost always false

if (userInput.equals("admin")) { } // ✅ CORRECT

// Null-safe comparison:
if ("admin".equals(userInput)) { } // ✅ Best practice — avoids NPE
```

---

## 8. Overridden Methods in String

`String` overrides these methods from `Object`:

```java
// 1. equals(Object obj)
String s = "hello";
s.equals("hello"); // true — compares content

// 2. hashCode()
// Returns consistent hash based on content
"hello".hashCode(); // always same value
new String("hello").hashCode(); // same value

// 3. toString()
String s2 = "world";
s2.toString(); // returns itself — "world"

// 4. compareTo(String another) — from Comparable<String>
"apple".compareTo("banana"); // -1 (negative)

// How equals is implemented internally:
public boolean equals(Object anObject) {
    if (this == anObject) return true;
    if (anObject instanceof String aString) {
        // compare byte arrays
        return StringLatin1.equals(value, aString.value);
    }
    return false;
}

// How hashCode is cached:
public int hashCode() {
    int h = hash;
    if (h == 0 && !hashIsZero) {
        h = isLatin1()
            ? StringLatin1.hashCode(value)
            : StringUTF16.hashCode(value);
        if (h == 0) hashIsZero = true;
        else hash = h;
    }
    return h;
}
```

---

## 9. All String Methods — Detailed

### 📏 Length & Checking

```java
String s = "Hello, World!";

s.length();           // 13 — number of chars
s.isEmpty();          // false — length == 0?
s.isBlank();          // false (Java 11+) — only whitespace?
s.charAt(0);          // 'H' — char at index
s.indexOf('o');       // 4 — first occurrence
s.indexOf('o', 5);    // 8 — first occurrence after index 5
s.lastIndexOf('o');   // 8 — last occurrence
s.lastIndexOf('o', 7); // 4 — last occurrence before index 7
```

### ✂️ Substring & Extraction

```java
String s = "Hello, World!";

s.substring(7);       // "World!" — from index to end
s.substring(7, 12);   // "World" — from 7 (inclusive) to 12 (exclusive)
s.charAt(1);          // 'e'
s.codePointAt(0);     // 72 (Unicode of 'H')
s.chars();            // IntStream of char values
```

### 🔍 Searching & Matching

```java
String s = "Hello, World!";

s.contains("World");         // true
s.startsWith("Hello");       // true
s.startsWith("World", 7);    // true — starts with "World" at index 7
s.endsWith("!");             // true
s.matches("Hello.*");        // true — regex match (full string)
s.regionMatches(7, "World!", 0, 5); // true — compare regions
s.regionMatches(true, 7, "world!", 0, 5); // true — ignoreCase version
```

### 🔄 Transformation

```java
String s = "  Hello, World!  ";

s.trim();                    // "Hello, World!" — removes ASCII whitespace
s.strip();                   // "Hello, World!" (Java 11+) — Unicode-aware
s.stripLeading();            // "Hello, World!  "
s.stripTrailing();           // "  Hello, World!"
s.toLowerCase();             // "  hello, world!  "
s.toUpperCase();             // "  HELLO, WORLD!  "
s.toLowerCase(Locale.US);    // locale-aware
"hello".toUpperCase(Locale.TURKEY); // careful — 'i' → 'İ'

// Replace
"aabbcc".replace('a', 'z');         // "zzbbcc" — char replace
"aabbcc".replace("aa", "zz");       // "zzbbcc" — string replace
"a1b2c3".replaceAll("[0-9]", "#");  // "a#b#c#" — regex replace
"a1b2c3".replaceFirst("[0-9]", "#"); // "a#b2c3"

// Reverse — no direct method; use StringBuilder
new StringBuilder("hello").reverse().toString(); // "olleh"
```

### 🔗 Joining & Splitting

```java
// split
String csv = "a,b,c,d";
String[] parts = csv.split(",");        // ["a","b","c","d"]
String[] parts2 = csv.split(",", 2);   // ["a","b,c,d"] — limit
"  hello  ".split("\\s+");             // ["", "hello"]

// join (Java 8+)
String.join(", ", "a", "b", "c");      // "a, b, c"
String.join("-", List.of("x","y","z")); // "x-y-z"

// concat
"Hello".concat(" World");              // "Hello World"

// repeat (Java 11+)
"ha".repeat(3);                        // "hahaha"
```

### 📊 Comparison

```java
"abc".compareTo("abd");         // -1 (c < d)
"abc".compareToIgnoreCase("ABC"); // 0
"abc".equals("abc");            // true
"abc".equalsIgnoreCase("ABC");  // true
"abc".contentEquals("abc");     // true — works with CharSequence
"abc".contentEquals(new StringBuilder("abc")); // true
```

### 🔢 Conversion

```java
// String → char[]
char[] arr = "hello".toCharArray();

// char[] → String
String s = new String(arr);

// String → bytes
byte[] b = "hello".getBytes();
byte[] b2 = "hello".getBytes(StandardCharsets.UTF_8);

// String → int/double etc.
int i = Integer.parseInt("42");
double d = Double.parseDouble("3.14");

// Primitive → String
String.valueOf(42);     // "42"
String.valueOf(3.14);   // "3.14"
Integer.toString(42);   // "42"
42 + "";                // "42" (concatenation trick)

// formatted (Java 15+)
"Hello %s".formatted("World"); // "Hello World"
```

### 🔐 Other Useful Methods

```java
// intern
String s = new String("hello").intern(); // returns pool reference

// hashCode
"hello".hashCode(); // consistent integer

// getChars — copy chars into array
char[] dest = new char[5];
"Hello World".getChars(6, 11, dest, 0); // dest = ['W','o','r','l','d']

// copyValueOf (same as new String(char[]))
String.copyValueOf(new char[]{'H','i'}); // "Hi"

// chars() and codePoints() — streams
"hello".chars().forEach(c -> System.out.print((char)c));
```

---

## 10. Type Conversion — String ↔ Other Types

```java
// ── String → Primitive ──
int    i  = Integer.parseInt("42");
long   l  = Long.parseLong("9999999999");
double d  = Double.parseDouble("3.14");
float  f  = Float.parseFloat("2.5");
boolean b = Boolean.parseBoolean("true");
char   c  = "A".charAt(0);

// ── Primitive → String ──
String s1 = String.valueOf(42);
String s2 = Integer.toString(42);
String s3 = "" + 42;              // concatenation
String s4 = String.format("%d", 42);

// ── String → Wrapper ──
Integer obj = Integer.valueOf("42");
Double  dbl = Double.valueOf("3.14");

// ── Wrapper → String ──
Integer num = 42;
String s = num.toString();

// ── String → char[] ──
char[] arr = "hello".toCharArray();

// ── char[] → String ──
String s = new String(arr);
String s = String.valueOf(arr);

// ── String → byte[] ──
byte[] bytes = "hello".getBytes(StandardCharsets.UTF_8);

// ── byte[] → String ──
String s = new String(bytes, StandardCharsets.UTF_8);
```

---

## 11. char, char[], and String Internals

### char in Java:
- `char` is a **16-bit unsigned integer** (0–65535).
- Represents a **Unicode code unit** (UTF-16).
- `char c = 'A';` → stores `65`.

```java
char c = 'A';
int code = c;           // 65
char next = (char)(c + 1); // 'B'

// char arithmetic
char ch = 'a';
ch - 'a'; // 0
ch - '0'; // used for digit conversion: '5' - '0' = 5

// char in switch
switch (c) {
    case 'A': System.out.println("Letter A"); break;
}
```

### char[] vs String:

| Feature | `char[]` | `String` |
|--------|----------|----------|
| Mutable | ✅ Yes | ❌ No |
| Clearable | ✅ Yes (security) | ❌ No |
| Has methods | ❌ Limited | ✅ Rich API |
| Pool | ❌ No | ✅ Yes |
| Use case | Passwords | General text |

```java
// ✅ Secure password handling:
char[] password = {'s','e','c','r','e','t'};
// use it...
Arrays.fill(password, '\0'); // clear from memory — not possible with String
```

---

## 12. String to char Array & Back

```java
// String → char[]
String s = "hello";
char[] arr = s.toCharArray();     // ['h','e','l','l','o']

// char[] → String
String s2 = new String(arr);
String s3 = String.valueOf(arr);
String s4 = String.copyValueOf(arr);

// Partial char[] → String
String s5 = new String(arr, 1, 3); // "ell" — offset=1, count=3

// Iterate chars
for (char c : "hello".toCharArray()) {
    System.out.print(c + " ");
}

// Using index
String s = "hello";
for (int i = 0; i < s.length(); i++) {
    System.out.print(s.charAt(i));
}

// Stream way
"hello".chars().mapToObj(c -> String.valueOf((char)c))
       .forEach(System.out::print);
```

---

## 13. Compile-time vs Runtime Strings

### Compile-time Constants (go to String Pool):
```java
String s1 = "Hello";
String s2 = "Hel" + "lo";    // compile-time concat → pool
String s3 = "Hello";

System.out.println(s1 == s2); // true — same pool object
System.out.println(s1 == s3); // true
```

### Runtime Strings (go to Heap):
```java
String part = "Hel";
String s4 = part + "lo";      // runtime concat → new Heap object

System.out.println(s1 == s4); // false — different objects
System.out.println(s1.equals(s4)); // true
```

### Why?
- Compiler resolves `"Hel" + "lo"` at compile time → `"Hello"` literal.
- `part + "lo"` involves a variable → resolved at **runtime** → `new StringBuilder().append(part).append("lo").toString()`.

---

## 14. Reading String Input in Java

```java
import java.util.Scanner;

Scanner sc = new Scanner(System.in);

// Read word (stops at whitespace)
String word = sc.next();

// Read full line
String line = sc.nextLine();

// ⚠️ Common pitfall: nextInt() + nextLine()
int n = sc.nextInt();
sc.nextLine(); // consume leftover newline
String s = sc.nextLine(); // now reads correctly

// Read char from string input
char c = sc.next().charAt(0);

// Read multiple strings
while (sc.hasNext()) {
    String token = sc.next();
}

// BufferedReader (faster for competitive programming)
import java.io.*;
BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
String line = br.readLine();
String[] parts = line.split(" ");
```

---

## 15. Spaces, Whitespace Handling

```java
String s = "  Hello   World  ";

s.trim();          // "Hello   World"  — strips ASCII space/tab/newline
s.strip();         // "Hello   World"  (Java 11+) — Unicode whitespace
s.stripLeading();  // "Hello   World  "
s.stripTrailing(); // "  Hello   World"
s.isBlank();       // false — only whitespace chars?

// Check for whitespace
Character.isWhitespace(' ');  // true
Character.isWhitespace('\t'); // true
Character.isWhitespace('\n'); // true
Character.isSpaceChar(' ');   // true (only space, not tab/newline)

// Normalize internal spaces
s.replaceAll("\\s+", " ").trim(); // "Hello World"

// Split on any whitespace
"hello   world".split("\\s+"); // ["hello", "world"]

// Count spaces
(int) "hello world".chars().filter(c -> c == ' ').count(); // 1
```

---

## 16. Compile Left-to-Right & Right-to-Left

### Left-to-Right (Default — String Concatenation):
```java
// Evaluated strictly left to right
System.out.println(1 + 2 + "3");    // "33" → (1+2)=3 → "3"+"3"="33"
System.out.println("1" + 2 + 3);    // "123" → "1"+2="12" → "12"+3="123"
System.out.println("1" + (2 + 3));  // "15"  → 2+3=5 first (parentheses)
```

### Right-to-Left (Reverse / StringBuilder):
```java
// Reverse a string
String reversed = new StringBuilder("hello").reverse().toString();

// Reverse manually
String s = "hello";
String rev = "";
for (int i = s.length() - 1; i >= 0; i--) {
    rev += s.charAt(i); // inefficient — use StringBuilder
}

// Efficient reverse
StringBuilder sb = new StringBuilder();
for (int i = s.length() - 1; i >= 0; i--) {
    sb.append(s.charAt(i));
}
String rev = sb.toString(); // "olleh"

// Right-to-left search
"hello".lastIndexOf('l'); // 3 — scans from right
```

---

## 17. String Interning Guidelines

```java
// Intern manually when:
// 1. You have many duplicate strings from external sources
String fromDb = getFromDatabase(); // not interned
String interned = fromDb.intern(); // now in pool

// 2. You need == comparison for performance
// 3. Memory optimization (many duplicates)

// Avoid intern when:
// - Strings are unique (wastes pool memory)
// - High churn strings (many short-lived strings)

// Java auto-interns:
// - All string literals
// - Compile-time constant expressions
```

---

## 18. StringBuilder — Complete Guide

`StringBuilder` is a **mutable**, **non-thread-safe** sequence of characters. Use it for single-threaded string manipulation.

### Why StringBuilder?
```java
// BAD — creates many intermediate String objects:
String result = "";
for (int i = 0; i < 1000; i++) {
    result += i; // new String each iteration!
}

// GOOD — single mutable buffer:
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 1000; i++) {
    sb.append(i);
}
String result = sb.toString();
```

### Constructors:
```java
new StringBuilder()           // capacity 16
new StringBuilder(50)         // capacity 50
new StringBuilder("hello")   // "hello", capacity 16+5=21
new StringBuilder(charSeq)   // from CharSequence
```

### All StringBuilder Methods:

```java
StringBuilder sb = new StringBuilder("Hello");

// ── Append ──
sb.append("!");           // Hello!
sb.append(42);            // Hello!42
sb.append(3.14);          // Hello!423.14
sb.append(true);          // Hello!423.14true
sb.append('A');           // Hello!423.14trueA
sb.append(new char[]{'X','Y'}); // append char array
sb.append("World", 1, 4); // append "orl" — substring of arg

// ── Insert ──
sb.insert(5, " World");   // insert at index 5
sb.insert(0, ">>>");      // insert at start

// ── Delete ──
sb.delete(0, 5);          // delete index 0..4 (exclusive 5)
sb.deleteCharAt(0);       // delete single char

// ── Replace ──
sb.replace(0, 5, "Hi");   // replace index 0..4 with "Hi"

// ── Reverse ──
sb.reverse();             // reverse entire content

// ── Index & Search ──
sb.indexOf("ll");         // first occurrence
sb.indexOf("ll", 3);      // from index 3
sb.lastIndexOf("l");      // last occurrence

// ── Char Operations ──
sb.charAt(0);             // char at index
sb.setCharAt(0, 'h');     // modify char at index
sb.codePointAt(0);        // Unicode code point

// ── Substring ──
sb.substring(2);          // from index to end (returns String)
sb.substring(2, 5);       // index 2..4 (returns String)
sb.subSequence(2, 5);     // returns CharSequence

// ── Capacity & Length ──
sb.length();              // current number of chars
sb.capacity();            // current buffer capacity
sb.ensureCapacity(100);   // ensure capacity >= 100
sb.trimToSize();          // shrink capacity to length
sb.setLength(3);          // truncate or pad with '\0'

// ── Convert ──
sb.toString();            // to immutable String
sb.chars();               // IntStream

// ── Chaining ──
String result = new StringBuilder()
    .append("Hello")
    .append(", ")
    .append("World")
    .append("!")
    .reverse()
    .toString(); // "!dlroW ,olleH"
```

### Capacity Growth:
```
Initial: 16
When full: new capacity = (old capacity * 2) + 2
So: 16 → 34 → 70 → 142 → ...
```

---

## 19. StringBuffer — Complete Guide

`StringBuffer` is identical to `StringBuilder` but **thread-safe** (all methods are `synchronized`).

```java
StringBuffer sf = new StringBuffer("Hello");

// ALL methods are same as StringBuilder:
sf.append(" World");
sf.insert(5, ",");
sf.delete(0, 3);
sf.reverse();
sf.replace(0, 2, "Hi");
sf.toString();
// ... all same methods

// Thread-safe example:
StringBuffer shared = new StringBuffer();
// Multiple threads can safely call shared.append() simultaneously

// But StringBuilder would have race conditions in multi-thread:
StringBuilder unsafe = new StringBuilder();
// NOT safe for concurrent access
```

### Internal Synchronization:
```java
// Every method in StringBuffer is like:
public synchronized StringBuffer append(String str) {
    // thread-safe operation
    return this;
}
```

---

## 20. StringBuilder vs StringBuffer vs String

| Feature | `String` | `StringBuilder` | `StringBuffer` |
|---------|----------|-----------------|----------------|
| Mutability | Immutable | Mutable | Mutable |
| Thread Safe | Yes (immutable) | ❌ No | ✅ Yes |
| Performance | Slow (new objects) | **Fastest** | Slower than SB |
| Pool Support | ✅ Yes | ❌ No | ❌ No |
| Use Case | Literals, keys, constants | Single-thread ops | Multi-thread ops |
| Memory | Shared (pool) | Own buffer | Own buffer |
| Synchronized | N/A | ❌ | ✅ |

### Decision Guide:
```
Need string manipulation?
    ├── Single thread?
    │       └── Use StringBuilder ✅
    └── Multiple threads?
            └── Use StringBuffer ✅

Just storing/passing text?
    └── Use String ✅

Using as HashMap key?
    └── Use String ✅ (immutable hashCode)
```

### Performance Benchmark Concept:
```java
// String concatenation in loop — O(n²) total copies
String s = "";
for (int i = 0; i < n; i++) s += i; // SLOW

// StringBuilder — O(n)
StringBuilder sb = new StringBuilder();
for (int i = 0; i < n; i++) sb.append(i); // FAST
String s = sb.toString();
```

---

## 📚 Quick Reference Card

```
String Methods Summary:
─────────────────────────────────────────────────
length()          charAt()         indexOf()
lastIndexOf()     substring()      contains()
startsWith()      endsWith()       equals()
equalsIgnoreCase()compareTo()      replace()
replaceAll()      split()          join()
trim()            strip()          toUpperCase()
toLowerCase()     toCharArray()    valueOf()
format()          intern()         isEmpty()
isBlank()         repeat()         matches()
─────────────────────────────────────────────────

StringBuilder / StringBuffer Methods:
─────────────────────────────────────────────────
append()          insert()         delete()
deleteCharAt()    replace()        reverse()
charAt()          setCharAt()      indexOf()
lastIndexOf()     substring()      length()
capacity()        ensureCapacity() toString()
─────────────────────────────────────────────────
```

---

*Made with ❤️ for Java learners | Star ⭐ if this helped!*
