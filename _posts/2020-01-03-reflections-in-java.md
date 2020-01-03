---
layout:     post
title:      Using Reflection in Java
date:       2020-01-03 16:00:00
summary:    How to use Reflections to access private data
categories: coding
thumbnail: fab fa-java
tags:
 - modding
 - code
 - java
 - reflection
---

> In today's world, "private" doesn't mean what it used to. The same goes for Java. Introducing an odd but very useful concept: Reflection.

[Reflection](https://www.oracle.com/technical-resources/articles/java/javareflection.html) is a way of accessing variables and methods declared "private" from other classes in Java. This can be especially useful when the class you are trying to access is from someone else's library.

Here's an exaple of a Java class, with a private method and a private variable:
```java
public class TestClass() {

    private static float privateFloat = 360.247f;

    private static void privateMethod() {
        System.out.println("lol you can't access me from other classes!!");
    }

}
```

Here's what we're going to start with.
```java
import java.lang.reflect.Method;
import java.lang.reflect.Field;

private static void reflectPrivateMethod() {
    // This is where we write our code
}
```

To start reflecting, we need to get an **instance** of our `TestClass`. You can make this `private static final` and put it at the top of your class, like so:
```java
private static final TestClass testclass = new TestClass();
```

But I'm going to put it in my `reflectPrivateMethod()` method for now, for simplicity. Then, we need to find the method by name using `getDeclaredMethod()`, and set it to be **accessible**, which avoids any errors.
```java
import java.lang.reflect.Method;

private static void reflectPrivateMethod() {

    TestClass instance = new TestClass();
    Method relflectedMethod = instance.getClass().getDeclaredMethod("privateMethod");
    reflectedMethod.setAccessible(true);

}
```
We then need to invoke our method. If the method is not void, for example, if it returns a boolean, then we can store the result of this in a variable by casting it:
```java
boolean invoke = (boolean) reflectedMethod.invoke();
```

However, our private method in this case is void, so we will just use `reflectedMethod.invoke()`.

Methods aren't the only thing you can reflect: you can also reflect **Fields** *(fancy name for variables)*. We do this in a very similar way:
```java
Field reflectedField = instance.getClass().getDeclaredField("privateFloat");
reflectedField.setAccessible(true);

float invoke = reflectedField.getFloat(null);
```

You can replace `getFloat` based on your data type, or use casting. Here we pass `null` as an argument, but you can use any object if the method or variable isn't `static`. For now, it's nothing to worry about.

## And now you know how Reflection works! It really is that simple.
---

# Helpful links
**Want to know more? Check out these links:**

[W3Schools Java Course](https://www.w3schools.com/java/default.asp)

[TutorialsPoint basic Java tutorials](https://www.tutorialspoint.com/java/)

[CodeAcademy Java course](https://www.codecademy.com/learn/learn-java)