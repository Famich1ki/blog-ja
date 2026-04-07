---
title: Java Enum Design & Generic Utility
date: 2026-04-07 23:41:50
tags:
  - Java
  - Enumeration
categories:
  - [Java, Enumeration]
cover: https://pics.findfuns.org/java-enumeration.png
---


# Java Enum Design & Generic Utility – Study Notes

## 1. Problem Background

In many business scenarios, enums need to convert from a `String` value:

```java
StudentStatus.from("enrolled");
```

Without abstraction, every enum repeats the same logic:

```java
for (StudentStatus s : StudentStatus.values()) {
    if (s.getValue().equals(value)) {
        return s;
    }
}
```

This leads to **code duplication** and poor maintainability. We need to keep code **dry**

## 2. Goal

- Eliminate repeated `from()` logic

- Keep type safety

- Maintain clean and readable API

- Make the solution reusable across all enums

## 3. Final Design (Best Practice)

### 3.1 Base Interface

```java
public interface BaseEnum {
    String getValue();
}
```

Purpose:

- Defines a **contract**
- Ensures all enums expose `getValue()`

## 3.2 Generic Utility Class

```java
public class EnumUtil {

    public static <T extends Enum<T> & BaseEnum> T fromValue(Class<T> enumClass, String value) {
        for (T constant : enumClass.getEnumConstants()) {
            if (constant.getValue().equals(value)) {
                return constant;
            }
        }
        throw new RuntimeException("Invalid value: " + value);
    }
}
```

## 3.3 Enum Implementation

```java
@Getter
public enum StudentStatus implements BaseEnum {

    ENROLLED("enrolled"),
    ABSENT("absent"),
    COMPLETED("completed");

    private final String value;

    StudentStatus(String value) {
        this.value = value;
    }

    public static StudentStatus from(String value) {
        return EnumUtil.fromValue(StudentStatus.class, value);
    }
}
```

# 4. Key Concepts Explained

### 4.1 Why Use an Interface?

Without `BaseEnum`:

```java
<T extends Enum<T>>
```

The compiler only knows it's an enum
It does NOT know `getValue()` exists

With `BaseEnum`:

```java
<T extends Enum<T> & BaseEnum>
```

Now the compiler guarantees:

- `getValue()` exists

### 4.2 Lombok vs Interface

| Feature                  | Lombok | Interface |
| ------------------------ | ------ | --------- |
| Generate getter          | ✅      | ❌         |
| Enforce method existence | ❌      | ✅         |

Lombok helps reduce code
Interface ensures **type safety**

### 4.3 Understanding Generics

**<T>**

- A **named type parameter**

- Can be used throughout the method

`?`

- An **anonymous wildcard**

- Cannot be used for operations

### 4.4 Difference: `T` vs `?`

| Feature                    | `T`  | `?`  |
| -------------------------- | ---- | ---- |
| Has a name                 | ✅    | ❌    |
| Can be used as return type | ✅    | ❌    |
| Can be modified            | ✅    | ❌    |
| Read-only usage            | ✅    | ✅    |

### 4.5 `Class<T>` vs `Class<?>`

| Type       | Meaning             |
| ---------- | ------------------- |
| `Class<T>` | Known specific type |
| `Class<?>` | Unknown type        |

Example:

```java
Class<StudentStatus> clazz = StudentStatus.class; // specific
Class<?> clazz = StudentStatus.class;             // generic
```

#  5. Why Not Skip the Interface?

## Option 1: Use Lombok only

Problem:

- Compiler cannot confirm `getValue()`
- Generic method fails

## Option 2: Use `name()`

```java
constant.name().equals(value)
```

Problems:

- Not flexible
- Tightly coupled to enum names
- Not suitable for real business cases

## Option 3: Use Reflection

```java
constant.getClass().getMethod("getValue")
```

Problems:

- Slow
- Unsafe
- Hard to maintain

#  6. Why Keep `from()` Inside Enum?

### Better readability:

```java
StudentStatus.from("enrolled");
```

vs

```java
EnumUtil.fromValue(StudentStatus.class, "enrolled");
```

Enum version is more expressive and domain-oriented

# 7. Design Philosophy

### Key Idea:

> Interfaces are not for writing code — they are for enforcing contracts.

### Good Design =

- Simple usage
- Strong constraints internally
- Reusable components

# 8. Final Takeaways

- Use **interface + generic utility + enum wrapper**

- Lombok is for convenience, not for type safety

- Generics (`T`) enable reusable and safe design

- Avoid shortcuts that sacrifice maintainability