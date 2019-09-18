---
title: Lombok使用入门
date: 2019-09-18T10:44:29+08:00
tags:
    - lombok, java, libraries
categories:
    - java-libraries
slug: java-libraries-lombok
description: >
    Lombok使用入门
---


平时定义类时，难免会写很多`Getter`, `Setter`，`toString`, `Constructor `等方法。虽然可以用IDE自带的代码生成，但是生成的代码仍然很多，看起来特别臃肿。有了[`Lombok`](https://projectlombok.org)这个库，用一个注解就能自动搞定这个问题。


<!--more-->

### 使用效果

没有对比就没有伤害。通过定义一个类，让我们来感受下`Lombok`的带来的便利。

不使用`Lombok`定义一个类`PersonWithoutLombok.java`, 虽然大部分代码都是IDE自动生成的，但是看起来仍然特别臃肿。

```java
public class PersonWithoutLombok {
    private String name;
    private int age;

    @Override
    public String toString() {
        return "PersonWithoutLombok{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }

    @Override
    public int hashCode() {
        return super.hashCode();
    }

    @Override
    public boolean equals(Object obj) {
        return super.equals(obj);
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public PersonWithoutLombok() {
    }
}
```

使用`Lombok`之后，让我们来看看`PersonWithLombok.java`

```java
import lombok.Data;

@Data
public class PersonWithLombok {
    private String name;
    private int age;
}
```

看看类结构

![看看类结构](http://images.wiseturtles.com/1568769332.png)

一样的效果，使用`Lombok`之后，整个代码看起来是不是更加清爽了呢?


除了`Data`之外。`Lombok`还提供了很多其它很多实用的注解，可以参考[官方文档](https://projectlombok.org/features/all)，每个注解的使用都配有使用示例，以及不使用注解的情况下，原始的Java代码该如何写，一对比非常容易理解。

### 安装配置

`Lombok`的使用也非常方便，可以直接在`命令行`，`Eclipse`, `IntelliJ`中使用。也方便和`Maven`, `Gradle`集成，`Android`开发中同样适用。具体支持如下：

![Setup](http://images.wiseturtles.com/1568769867.png)

下面主要介绍下在命令行中和`IntelliJ`中的使用方式，其它的可以直接参考[官方文档](https://projectlombok.org/)，都有非常详细的介绍


#### 命令行中使用

1. 下载最新版本的[lombock.jar](https://projectlombok.org/downloads/lombok.jar)
2. 创建`PersonWithLombok.java`类文件

    ```bash
    $ cat PersonWithLombok.java
    ```

    ```java
    import lombok.Data;


    @Data
    public class PersonWithLombok {
        private String name;
        private int age;
    }
    ```

3. 编译`PersonWithLombok.java`类文件

    ```bash
    $ javac -cp /path/to/lombok.jar PersonWithLombok.java
    ```

4. 查看生成的`class`文件

    ```bash
    $ javap PersonWithLombok.class
    ```

    ```java
    Compiled from "PersonWithLombok.java"
    public class PersonWithLombok {
      public PersonWithLombok();
      public java.lang.String getName();
      public int getAge();
      public void setName(java.lang.String);
      public void setAge(int);
      public boolean equals(java.lang.Object);
      protected boolean canEqual(java.lang.Object);
      public int hashCode();
      public java.lang.String toString();
    }
    ```

	可以看到自动生成了相关代码


#### [IntelliJ中使用](https://github.com/mplushnikov/lombok-intellij-plugin)

1. 安装插件

    `Preferences` > `Settings` > `Plugins` > `Browse repositories...` > `Search for "lombok"` > `Install Plugin`

    然后重启IDE

    ![安装插件](http://images.wiseturtles.com/1568770578.png)

2. 配置IDE的编译器，开启

    `Preferences` -> `Build, Execution, Deployment` -> `Compiler, Annotation Processors`. 点击`Enable Annotation Processing`，并勾选`Obtain processors from project classpath`

    ![配置IDE，启用Annotation Processing](http://images.wiseturtles.com/1568770670.png)


3. 添加`Lombok`到项目依赖。这里根据使用的构建工具不同，操作也不同。

    - 如果使用的是`Gradle`, 在`build.gradle`中添加如下内容

        ```groovy
        // 'compile' can be changed to 'compileOnly' for Gradle 2.12+
        // or 'provided' if using 'propdeps' plugin from SpringSource
        compile "org.projectlombok:lombok:1.18.8"
         ```

    - 如果使用的是`Maven`，直接在`pom.xml`中添加如下内容

        ```xml
        <dependencies>
            <dependency>
                <groupId>org.projectlombok</groupId>
                <artifactId>lombok</artifactId>
                <version>1.18.8</version>
                <scope>provided</scope>
            </dependency>
        </dependencies>
        ```

    - 如果使用的是`Ivy`, 在`ivy.xml`中添加如下内容

        ```xml
        <dependency org="org.projectlombok" name="lombok" rev="1.18.8" conf="build" />
        ```

    - 如果没有使用任何构建工具，那需要手动下载[lombock.jar](https://projectlombok.org/downloads/lombok.jar), 然后在`项目结构`中，把下载的`lombok.jar`添加到项目依赖的`Libraries`中

        ![添加lombok.jar到项目的classpath中](http://images.wiseturtles.com/1568771143.png)

4. 创建一个类`PersonWithLombok.java`，测试效果,如下图，从类结构中可以看到，IDE已经自动识别出使用`Lombok`后生成的代码

    ![效果图](http://images.wiseturtles.com/1568771270.png)


### 常用注解介绍

#### `val`

作用在局部变量，用于定义一个局部变量，并将它声明为`final`, 不用声明变量类型，它会根据初始赋值自动推动变量类型。**只能用于局部变量或loop循环中，不能用于类的属性**。

```java
import java.util.ArrayList;
import java.util.HashMap;
import lombok.val;

public class ValExample {
  public String example() {
    val example = new ArrayList<String>();
    example.add("Hello, World!");
    val foo = example.get(0);
    return foo.toLowerCase();
  }

  public void example2() {
    val map = new HashMap<Integer, String>();
    map.put(0, "zero");
    map.put(5, "five");
    for (val entry : map.entrySet()) {
      System.out.printf("%d: %s\n", entry.getKey(), entry.getValue());
    }
  }
}
```

等效于

```java
import java.util.ArrayList;
import java.util.HashMap;
import java.util.Map;

public class ValExample {
  public String example() {
    final ArrayList<String> example = new ArrayList<String>();
    example.add("Hello, World!");
    final String foo = example.get(0);
    return foo.toLowerCase();
  }

  public void example2() {
    final HashMap<Integer, String> map = new HashMap<Integer, String>();
    map.put(0, "zero");
    map.put(5, "five");
    for (final Map.Entry<Integer, String> entry : map.entrySet()) {
      System.out.printf("%d: %s\n", entry.getKey(), entry.getValue());
    }
  }
}
```

#### `var`

与`val`功能类似，除了变量不会标记为`final`

#### `@NonNull`

非空检查

```java
import lombok.NonNull;

public class NonNullExample extends Something {
  private String name;

  public NonNullExample(@NonNull Person person) {
    super("Hello");
    this.name = person.getName();
  }
}
```

等效于


```java

 import lombok.NonNull;

public class NonNullExample extends Something {
  private String name;

  public NonNullExample(@NonNull Person person) {
    super("Hello");
    if (person == null) {
      throw new NullPointerException("person is marked @NonNull but is null");
    }
    this.name = person.getName();
  }
}
```

#### `@Cleanup`

资源管理，自动调用`close()`方法

```java
import lombok.Cleanup;
import java.io.*;

public class CleanupExample {
  public static void main(String[] args) throws IOException {
    @Cleanup InputStream in = new FileInputStream(args[0]);
    @Cleanup OutputStream out = new FileOutputStream(args[1]);
    byte[] b = new byte[10000];
    while (true) {
      int r = in.read(b);
      if (r == -1) break;
      out.write(b, 0, r);
    }
  }
}
```

等价于

```java
import java.io.*;

public class CleanupExample {
  public static void main(String[] args) throws IOException {
    InputStream in = new FileInputStream(args[0]);
    try {
      OutputStream out = new FileOutputStream(args[1]);
      try {
        byte[] b = new byte[10000];
        while (true) {
          int r = in.read(b);
          if (r == -1) break;
          out.write(b, 0, r);
        }
      } finally {
        if (out != null) {
          out.close();
        }
      }
    } finally {
      if (in != null) {
        in.close();
      }
    }
  }
}
```

#### `@Getter @Setter`

自动生成`Getter`和`Setter`代码


* 默认情况下，生成的getter/setter方法被标记为`public`,除非指定了`AccessLevel`

* 同样可以加`@Getter`和`@Setter`注解作用到类上， 那样类中所有的非静态属性都会自动生成getter/setter代码

* 如果不想给某个属性自动生成getter/setter代码，可以标记类属性为`AccessLevel.NONE`。这样可以覆盖给类添加`@Getter`, `@Setter` `@Data`产生的行为。

```java
import lombok.AccessLevel;
import lombok.Getter;
import lombok.Setter;

public class GetterSetterExample {
  /**
   * Age of the person. Water is wet.
   *
   * @param age New value for this person's age. Sky is blue.
   * @return The current value of this person's age. Circles are round.
   */
  @Getter @Setter private int age = 10;

  /**
   * Name of the person.
   * -- SETTER --
   * Changes the name of this person.
   *
   * @param name The new value.
   */
  @Setter(AccessLevel.PROTECTED) private String name;

  @Override public String toString() {
    return String.format("%s (age: %d)", name, age);
  }
}
```

等价于

```java
public class GetterSetterExample {
  /**
   * Age of the person. Water is wet.
   */
  private int age = 10;

  /**
   * Name of the person.
   */
  private String name;

  @Override public String toString() {
    return String.format("%s (age: %d)", name, age);
  }

  /**
   * Age of the person. Water is wet.
   *
   * @return The current value of this person's age. Circles are round.
   */
  public int getAge() {
    return age;
  }

  /**
   * Age of the person. Water is wet.
   *
   * @param age New value for this person's age. Sky is blue.
   */
  public void setAge(int age) {
    this.age = age;
  }

  /**
   * Changes the name of this person.
   *
   * @param name The new value.
   */
  protected void setName(String name) {
    this.name = name;
  }
}
```


#### `@ToString`

自动添加`toSting()`方法，默认使用类名 + 所有的非静态属性值。

可以通过`@ToString.Exclude`来排除一些不想添加到toString()中的属性

```java
import lombok.ToString;

@ToString
public class ToStringExample {
  private static final int STATIC_VAR = 10;
  private String name;
  private Shape shape = new Square(5, 10);
  private String[] tags;
  @ToString.Exclude private int id;

  public String getName() {
    return this.name;
  }

  @ToString(callSuper=true, includeFieldNames=true)
  public static class Square extends Shape {
    private final int width, height;

    public Square(int width, int height) {
      this.width = width;
      this.height = height;
    }
  }
}
```

等价于

```java
import java.util.Arrays;

public class ToStringExample {
  private static final int STATIC_VAR = 10;
  private String name;
  private Shape shape = new Square(5, 10);
  private String[] tags;
  private int id;

  public String getName() {
    return this.getName();
  }

  public static class Square extends Shape {
    private final int width, height;

    public Square(int width, int height) {
      this.width = width;
      this.height = height;
    }

    @Override public String toString() {
      return "Square(super=" + super.toString() + ", width=" + this.width + ", height=" + this.height + ")";
    }
  }

  @Override public String toString() {
    return "ToStringExample(" + this.getName() + ", " + this.shape + ", " + Arrays.deepToString(this.tags) + ")";
  }
}
```

#### `@Data`

相当于`@ToString`, `@EqualsAndHashCode`, `@Getter`,`@Setter`,`@RequiredArgsConstructor`的组合

#### `@Log`

自动为类添加`log`属性，方便直接使用log

```java
import lombok.extern.java.Log;
import lombok.extern.slf4j.Slf4j;

@Log
public class LogExample {

  public static void main(String... args) {
    log.severe("Something's wrong here");
  }
}

@Slf4j
public class LogExampleOther {

  public static void main(String... args) {
    log.error("Something else is wrong here");
  }
}

@CommonsLog(topic="CounterLog")
public class LogExampleCategory {

  public static void main(String... args) {
    log.error("Calling the 'CounterLog' with a message");
  }
}
```

等价于

```java
public class LogExample {
  private static final java.util.logging.Logger log = java.util.logging.Logger.getLogger(LogExample.class.getName());

  public static void main(String... args) {
    log.severe("Something's wrong here");
  }
}

public class LogExampleOther {
  private static final org.slf4j.Logger log = org.slf4j.LoggerFactory.getLogger(LogExampleOther.class);

  public static void main(String... args) {
    log.error("Something else is wrong here");
  }
}

public class LogExampleCategory {
  private static final org.apache.commons.logging.Log log = org.apache.commons.logging.LogFactory.getLog("CounterLog");

  public static void main(String... args) {
    log.error("Calling the 'CounterLog' with a message");
  }
}

```


下面注解的使用，可以[参考官方文档](https://projectlombok.org/features/all)

#### `@Value`
#### `@Builder`
#### `@Synchronized`
#### `@SneakyThrows`
#### `@With`
#### `@EqualsAndHashCode`
#### `@NoArgsConstructor @RequiredArgsConstructor @AllArgsConstructor`
#### `@Getter(lazy=true)`


赶紧拿起你的电脑试试吧！
