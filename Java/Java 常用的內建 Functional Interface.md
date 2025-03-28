---
title: Java 常用的內建 Functional Interface
tags: [java]

---

# Java 常用的內建 Functional Interface

## Consumer<T>
```java=
public interface Consumer<T> {
    void accept(T t);
}
```

## Function<T,R>
```java=
public interface Function<T, R> {
    R apply(T t);
}
```

## Predicate<T>
```java=
public interface Predicate<T> {
    boolean test(T t);
}
```

## Supplier<T>
```java=
public interface Supplier<T> {

    T get();
}
```

###### tags: `java`