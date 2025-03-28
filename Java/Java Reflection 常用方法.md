---
title: Java Reflection 常用方法
tags: [java]

---

# Java Reflection 常用方法

## Get Class by name

```java=
Class c = Class.forName("com.package.MyClass");
```

## 抓取泛型的 class
```java=
(Class<T>) ((ParameterizedType) getClass()
                            .getGenericSuperclass()).getActualTypeArguments()[0];
```

## 取得extends後的 class list
```java=

// gradle implementation 'org.reflections:reflections:0.10.2'

        Reflections reflections = new Reflections(CommConstant.PACKAGE_PREFIX);
        Set<Class<? extends TaskAbs>> classes = reflections.getSubTypesOf(TaskAbs.class);
        List<Class<? extends TaskAbs>> runClazzs = classes.stream()
                .filter(c -> {
                    return c.getAnnotation(Component.class) != null;
                })
                .collect(Collectors.toList());
        return runClazzs;
```

## newInstance by Class
```java=
Class c = ...
var constructor = c.getConstructor();
            constructor.setAccessible(true);
            TaskAbs ta = constructor.newInstance(...);
```

## Get Class Name
```java=
// 一般
getName(); // my.MyObj
getgetCanonicalName(); // my.MyObj
getSimpleName();    // MyObj

// innerClass
getName(); // my.MyObj$InnerObj
getgetCanonicalName(); // my.MyObj
getSimpleName();    // InnerObj

```

###### tags: `java`