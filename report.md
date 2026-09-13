# SOFE5520U Lab 1 Report: Binary Calculator

**Name:** Cris Huynh 

**Student ID:** 101072055 

**Course:** ENGR-5520G-001 

**GitHub Repository:** https://github.com/CrisH2307/ENGR-5520G-001

**Video Demo:** [[Link]](https://drive.google.com/file/d/13AV0eTRX1rp3jySRgpcuk_ZUKBzwi8Hz/view?usp=drive_link)

---

## 1. Introduction

In this lab I worked with a Maven project called BinaryCalculator. The project already came with a `Binary` class that stores an unsigned binary number as a string and has an `add` function. For the design part, I had to add three new operations to the class:

1. `or`, which does a bitwise OR between two binary numbers
2. `and`, which does a bitwise AND between two binary numbers
3. `multiply`, which multiplies two binary numbers

I also updated `App.java` so it shows the result of the new functions, and I wrote three JUnit test cases for each new function in `BinaryTest.java`.

---

## 2. Source Code Discussion

### 2.1 How the Binary class stores numbers

Before writing anything new I looked at how the constructor works, because all my functions depend on it. The constructor takes a string and:

- sets the value to `"0"` if the string is null or empty
- sets the value to `"0"` if the string has any character other than `0` or `1`
- removes leading zeros, so `"00001001"` becomes `"1001"`
- makes sure a string of only zeros is stored as `"0"`

This was really useful for me. Since every function ends with `return new Binary(...)`, I don't have to clean up leading zeros myself. The constructor does it for me.

### 2.2 OR function

```java
public static Binary or(Binary num1, Binary num2) {
    int ind1 = num1.number.length() - 1;
    int ind2 = num2.number.length() - 1;
    StringBuilder result = new StringBuilder();

    while (ind1 >= 0 || ind2 >= 0) {
        char digit1 = (ind1 >= 0) ? num1.number.charAt(ind1) : '0';
        char digit2 = (ind2 >= 0) ? num2.number.charAt(ind2) : '0';

        // Perform OR operation
        char orResult = (digit1 == '1' || digit2 == '1') ? '1' : '0';
        result.insert(0, orResult);

        ind1--;
        ind2--;
    }

    return new Binary(result.toString());
}
```

I followed the same idea that the `add` function uses. I start from the last digit of each number (the least significant bit) and move to the left. The loop keeps going as long as one of the numbers still has digits left.

The tricky part is when the two numbers have different lengths, like `"1"` and `"1010"`. To handle that, if one index goes below 0, I just treat that digit as `'0'`. This is the same as padding the shorter number with zeros on the left, which doesn't change its value.

For each position, the result bit is `'1'` if at least one of the digits is `'1'`, otherwise it's `'0'`. I insert each bit at the front of a `StringBuilder` so the final string ends up in the right order. I used `StringBuilder` instead of adding strings together because it's cleaner to build the result one character at a time.

### 2.3 AND function

```java
public static Binary and(Binary num1, Binary num2) {
    int ind1 = num1.number.length() - 1;
    int ind2 = num2.number.length() - 1;
    StringBuilder result = new StringBuilder();

    while (ind1 >= 0 || ind2 >= 0) {
        char digit1 = (ind1 >= 0) ? num1.number.charAt(ind1) : '0';
        char digit2 = (ind2 >= 0) ? num2.number.charAt(ind2) : '0';

        // Perform AND operation
        char andResult = (digit1 == '1' && digit2 == '1') ? '1' : '0';
        result.insert(0, andResult);

        ind1--;
        ind2--;
    }

    return new Binary(result.toString());
}
```

The AND function has the same structure as OR. The only real difference is the condition: the result bit is `'1'` only when both digits are `'1'`.

One thing I noticed with AND is that the result can have a lot of leading zeros. For example, `1000 AND 0111` builds the string `"0000"`. That's why passing the result into the constructor matters here, because it turns `"0000"` into `"0"`. I made sure to test this case (see Section 3).

### 2.4 Multiply function

```java
public static Binary multiply(Binary num1, Binary num2) {
    Binary result = new Binary("0");
    int ind2 = num2.number.length() - 1;

    while (ind2 >= 0) {
        if (num2.number.charAt(ind2) == '1') {
            // Shift num1 left by the appropriate number of positions
            String shiftedNum1 = num1.number + "0".repeat(num2.number.length() - 1 - ind2);
            result = Binary.add(result, new Binary(shiftedNum1));
        }
        ind2--;
    }

    return result;
}
```

For multiplication I used the shift and add method, which is basically the same long multiplication we do by hand, but in base 2. The lab said we could reuse the `add` function, so I did.

Here is how it works:

1. I start with `result` equal to `"0"`.
2. I go through each digit of `num2` from right to left.
3. If the digit is `'1'`, I shift `num1` to the left by the position of that digit. In binary, shifting left means adding zeros at the end, so I use `"0".repeat(...)` to add the right number of zeros.
4. I add the shifted value to `result` using `Binary.add`.
5. If the digit is `'0'`, I skip it, since multiplying by 0 adds nothing.

For example, `101 x 11` (5 x 3):

```
      101      (digit 0 of 11 is 1, shift by 0)
   + 1010      (digit 1 of 11 is 1, shift by 1)
   ------
     1111      = 15
```

I liked this approach because `add` already handles carries correctly and is already tested, so I didn't have to write carry logic again. Also, if either number is `"0"`, the loop either never adds anything or only adds zeros, so the answer is `"0"` without needing a special case.

Note: `String.repeat()` needs Java 11 or newer. The project's `pom.xml` is set to Java 17, so this works fine.

### 2.5 Updates to App.java

I added the new operations to the `main` method so they print out when I run the app:

```java
Binary or = Binary.or(binary1, binary2);
System.out.println("Their OR operation result is " + or.getValue());
Binary and = Binary.and(binary1, binary2);
System.out.println("Their AND operation result is " + and.getValue());
Binary multiply = Binary.multiply(binary1, binary2);
System.out.println("Their multiplication result is " + multiply.getValue());
```

When I run it, this is the output:

```
First binary number is 10001000
Second binary number is 111000
Their summation is 11000000
Their OR operation result is 10111000
Their AND operation result is 1000
Their multiplication result is 1110111000000
```

I checked these by hand using decimal. The first number `10001000` is 136 and the second `111000` is 56.

| Operation | Decimal check | Binary output | Correct? |
|---|---|---|---|
| Add | 136 + 56 = 192 | 11000000 | Yes |
| OR | 10001000 OR 00111000 | 10111000 | Yes |
| AND | 10001000 AND 00111000 | 1000 | Yes |
| Multiply | 136 x 56 = 7616 | 1110111000000 | Yes |

The first input was given as `"00010001000"`, and the constructor removed the leading zeros before any math happened, which also shows the constructor is doing its job.

---

## 3. Testing Code Discussion

All tests are in `src/test/java/com/ontariotechu/sofe3980U/BinaryTest.java` and use JUnit 4. Each test creates two `Binary` objects, calls the function, and uses `assertTrue` to check the result string.

For each function I tried to pick three different kinds of cases instead of three that are almost the same:

- a **normal case** to check the basic logic
- a **zero case** to check what happens when one input is `"0"`
- an **edge case** that targets the part of the code most likely to break

### 3.1 OR tests

| Test | Input 1 | Input 2 | Expected | Why I chose it |
|---|---|---|---|---|
| `or1` | 1010 | 1100 | 1110 | Normal case, same length, covers all four bit combos (1/1, 0/1, 1/0, 0/0) |
| `or2` | 1010 | 0 | 1010 | OR with zero should give back the same number |
| `or3` | 1 | 1010 | 1011 | Different lengths, checks that the shorter number gets padded with zeros correctly |

```java
@Test
public void or3()
{
    Binary binary1 = new Binary("1");
    Binary binary2 = new Binary("1010");
    Binary binary3 = Binary.or(binary1, binary2);
    assertTrue(binary3.getValue().equals("1011"));
}
```

### 3.2 AND tests

| Test | Input 1 | Input 2 | Expected | Why I chose it |
|---|---|---|---|---|
| `and1` | 1010 | 1100 | 1000 | Normal case, only the bit where both are 1 stays |
| `and2` | 1010 | 0 | 0 | AND with zero should always be zero |
| `and3` | 1000 | 0111 | 0 | No bits overlap, so the raw result is `"0000"`. Checks that it becomes `"0"` |

```java
@Test
public void and3(){
    Binary binary1 = new Binary("1000");
    Binary binary2 = new Binary("0111");
    Binary binary3 = Binary.and(binary1, binary2);
    assertTrue(binary3.getValue().equals("0"));
}
```

I think `and3` is the most interesting one. If I had built the result string and returned it without going through the constructor, this test would fail because it would return `"0000"`.

### 3.3 Multiply tests

| Test | Input 1 | Input 2 | Expected | Why I chose it |
|---|---|---|---|---|
| `multiply1` | 101 | 11 | 1111 | Normal case, 5 x 3 = 15 |
| `multiply2` | 101 | 0 | 0 | Multiplying by zero gives zero |
| `multiply3` | 1010 | 1010 | 1100100 | 10 x 10 = 100, several shifts and carries in the add step |

```java
@Test
public void multiply3(){
    Binary binary1 = new Binary("1010");
    Binary binary2 = new Binary("1010");
    Binary binary3 = Binary.multiply(binary1, binary2);
    assertTrue(binary3.getValue().equals("1100100"));
}
```

At first, my third test for each function also used `"0"` as one of the inputs. When I looked at it again, I realized two zero tests per function don't really test much new, since the answer is basically obvious. So I swapped one of them for a harder case. For multiply, `1010 x 1010` is a better test because it has to shift more than once and the `add` calls have to carry bits.

### 3.4 Test results

I ran the tests with `mvn clean package`. All 20 tests passed (11 tests that came with the project plus my 9 new ones):

```
Tests run: 20, Failures: 0, Errors: 0, Skipped: 0
```

---

## 4. Build, Run, and Documentation

These are the commands I used, and they are also what I show in the video:

```bash
# build the project and run the tests
mvn clean package

# run the app
java -jar target/BinaryCalculator-1.0.0-jar-with-dependencies.jar

# generate the documentation
mvn javadoc:javadoc
```

The generated documentation is in `target/site/apidocs/index.html`. In the video I open that page and go to the `Binary` class to show the `or`, `and`, and `multiply` functions.

---

## 5. Conclusion

This lab helped me get more comfortable with Maven, JUnit, and Javadoc. For the design part, the main thing I learned was that reusing what's already there makes things easier. The OR and AND functions follow the same digit-by-digit loop as `add`, and `multiply` just calls `add` over and over. I also learned that having three test cases isn't enough on its own. The tests need to cover different situations, like different lengths or results with leading zeros, or else they don't really catch bugs.
