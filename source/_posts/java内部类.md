---
title: java内部类
typora-root-url: ../
date: 2022-11-24 19:02:47
tags: 
 - java
 - 内部类
categories:
 - java
---





## 成员内部类

```java
public class Outer {//外部类
    class Inner {//内部类
    }
}
```

成员内部类可以拥有 private 访问权限、protected 访问权限、public 访问权限及包访问权限。

如果成员内部类 Inner 用 private 修饰，则只能在外部类的内部访问，如果用 public 修饰，则任何地方都能访问；如果用 protected 修饰，则只能在同一个包下或者继承外部类的情况下访问；如果是默认访问权限，则只能在同一个包下访问。这一点和外部类有一点不一样，外部类只能被 public 和包访问两种权限修饰。由于成员内部类看起来像是外部类的一个成员，所以可以像类的成员一样拥有多种权限修饰。

```java
public class Outer {

    private String name;

    public Outer(String name) {
        this.name = name;
    }

    public void out_mem_fuc(){
        System.out.println("外部类成员方法");
    }
    // 成员内部类 成员
    class MemberInner{
        void mem_fuc(){
            System.out.println("this: "+ this.getClass());
            System.out.println("name:" + name);
            name = "外部类(修改后)";
            System.out.println("name:" + name);
            out_mem_fuc();
        }
    }

    public static void main(String[] args) {
        new Outer("外部类").new MemberInner().mem_fuc();
    }
}
```



```txt
this: class com.qibria.inner.Outer$MemberInner
name:外部类
name:外部类(修改后)
外部类成员方法
```





## 局部内部类

```java
public class Outer {
    public void out_local_fuc(){
        class LocalInner{
        }
    }
}
```

局部内部类就像是方法里面的一个局部变量一样，是不能有 public、protected、private 以及 static 修饰符的。

（1）局部内部类可以直接访问外部类的所有成员，包含私有的；

（2）不能添加修饰符，因为它是一个局部变量，局部变量是不能使用修饰符的，但是可以用final修饰，因为局部变量是可以使用final修饰的。

（3）作用域仅仅在定义它的方法或者代码块中。

（4）局部内部类访问外部类的成员属性或者方法时，直接访问；

（5）外部类访问局部内部类的成员属性或者方法时，通过先创建对象，再访问，且必须要在作用域内。

```java
public class Outer {

    private String name;

    public Outer(String name) {
        this.name = name;
    }

    public void out_mem_fuc(){
        System.out.println("外部类成员方法");
    }

    public void out_local_fuc(){
        class LocalInner{
            void local_fuc(){
                System.out.println("this: " + this);
                System.out.println("修改前的外部类-->name: "+Outer.this.name);
                Outer.this.name = "外部类(修改)";
                System.out.println("修改后外部类-->name: "+Outer.this.name);
                out_mem_fuc();
            }
        }
        new LocalInner().local_fuc();
    }


    public static void main(String[] args) {
        new Outer("外部类").out_local_fuc();
    }

}
```

```txt
this: com.qibria.inner.Outer$1LocalInner@5e2de80c
修改前的外部类-->name: 外部类
修改后外部类-->name: 外部类(修改)
外部类成员方法
```



## 匿名内部类

Java 中可以实现一个类中包含另外一个类，且不需要提供任何的类名直接实例化。主要是用于在我们需要的时候创建一个对象来执行特定的任务，可以使代码更加简洁。匿名类是不能有名字的类，它们不能被引用，只能在创建时用 **new** 语句来声明它们。



匿名内部类也是不能有访问修饰符和 static 修饰符的。

匿名内部类是唯一一种没有构造器的类。正因为其没有构造器，所以匿名内部类的使用范围非常有限，大部分匿名内部类用于接口回调。

匿名内部类在编译的时候由系统自动起名为 Outer$1.class。

匿名内部类主要用于继承其他类或是实现接口，并不需要增加额外的方法，只是对继承方法的实现或是重写。

匿名内部类使用一次，就不再使用。

内部类可以直接访问外部属性，但是不要进行修改，如果真的需要修改请用final 数组，通过修改数组内部属性达到修改目的

```java
    public static void main(String[] args) {
        int  num = 123;
        new Consumer<Integer>() {
            @Override
            public void accept(Integer o) {
                System.out.println("this: " + this);
                System.out.println("修改前-->num: "+num);
                //num = 456; // 报错--> java: 从内部类引用的本地变量必须是最终变量或实际上的最终变量
                System.out.println("修改后-->num: " + num);
            }
        }.accept(num);
    }
```

```txt
this: com.qibria.inner.Outer$1@1d44bcfa
修改前-->num: 123
修改后-->num: 123
```





## 静态内部类

静态内部类也是定义在另一个类里面的类，只不过在类的前面多了一个关键字static。静态内部类是不需要依赖于外部类的，这点和类的静态成员属性有点类似，并且它不能使用外部类的非static成员变量或者方法，这点很好理解，因为在没有外部类的对象的情况下，可以创建静态内部类的对象，如果允许访问外部类的非static成员就会产生矛盾，因为外部类的非static成员必须依附于具体的对象。可以添加任意的访问修饰符public、protected、private 以及默认，因为它就是类的成员。作用域和其他类成员一样，为整个类体。

```java
public class Outer {//外部类
    static class Inner {//内部类
    }
}
```



```java
package com.qibria.reflect.demo_01;

import java.util.function.Consumer;

public class Outer {

    private String name;

    public Outer(String name) {
        this.name = name;
    }

    public void out_mem_fuc(){
        System.out.println("外部类成员方法");
    }

    public static void out_static_fuc(){
        System.out.println("外部类静态方法");
    }
    
    // 成员内部类 成员
    static int static_num  = 1;
    static class StaticInner{
        void static_fuc(){
            System.out.println("this: "+ this.getClass());
            System.out.println("修改前-->num: "+static_num);
            static_num = 2;
            System.out.println("修改后-->num: "+static_num);
            out_static_fuc();
        }
    }

    public static void main(String[] args) {
        new Outer.StaticInner().static_fuc();
    }
}
```

```txt
this: class com.qibria.inner.Outer$StaticInner
修改前-->num: 1
修改后-->num: 2
外部类静态方法
```

