---
title: java反射
typora-root-url: ../
date: 2022-11-24 19:03:06
tags:
 - java
 - 反射
categories:
 - java
---



## 前言

Reflection(反射)是Java被视为动态语言的关键，反射机制允许程序在执行期间借助于Reflection API取得任何类的内部信息，并能直接操作任意对象的内部属性及方法

```java
Class c = Class.forName("java.lang.String")
```

加载完类之后，在堆内存的方法区中就产生了一个Class类型的对象(一个类中只有一个Class对象), 这个对象就包含了完整的类的结构信息。 我们可以以通过这个对象看到类的内部结构这就称为反射



<br/>

## 获取Class类的实例的三种方法

若已知具体的类，通过类的class属性获取，该方法最为可靠，程序性能最高

```java
Class clazz = Person.class;
```



已知某个类的实例，调用该实例的`getClass()`方法获取Class对象

```java
Person person = new Person();
Class clazz = person.getClass();
```



已知一个类的全类名，且该类在类路径下，可通过`Class`类的静态方法`forName()`获取

```java
Class clazz = Class.forName("com.qibria.Person");
```



```java
    public static void main(String[] args) throws ClassNotFoundException {
        three_method();
    }

    private static void three_method() throws ClassNotFoundException {

        // 方法一
        Class<Student> Class_1 = Student.class;
        System.out.println("方法一-->Class: "+Class_1 +
                "\t hashcode: " + Class_1.hashCode());

        // 方法二
        Student student = new Student();
        Class<? extends Person> Class_2 = student.getClass();
        System.out.println("方法二-->Class: "+Class_2 +
                "\t hashcode: " + Class_2.hashCode());

        // 方法三
        Class<?> Class_3 = Class.forName("com.qibria.reflect.Student");
        System.out.println("方法三-->Class: "+Class_3 +
                "\t hashcode: " + Class_3.hashCode());
        
    }
```

```java
方法一-->Class: class com.qibria.reflect.Student	hashcode: 1580066828
方法二-->Class: class com.qibria.reflect.Student	hashcode: 1580066828
方法三-->Class: class com.qibria.reflect.Student	hashcode: 1580066828
```



<br/>

## Class类型

```java
class: 外部类，成员(成员内部类，静态内部类)，局部内部类，匿名内部类
interface: 接口
[]: 数组
enum: 枚举
annotation: 注解@interface
primitive type: 基本数据类型
void 空类型
```



```java
public static void main(String[] args){
    class_type();
}
private static void class_type(){
    Class c1 = Object.class; //类
    Class c2 = Comparable.class; //接口
    Class c3 = String[].class; //一维数组
    Class c4 = int[][].class; //二维数组
    Class c5 = Override.class; //注解
    Class c6 = ElementType.class; //枚举
    Class c7 = Integer.class; //基本数据类型
    Class c8 = void.class; //void
    Class c9 = Class.class; //Class
    
    System.out.println(c1);
    System.out.println(c2);
    System.out.println(c3);
    System.out.println(c4);
    System.out.println(c5);
    System.out.println(c6);
    System.out.println(c7);
    System.out.println(c8);
    System.out.println(c9);
}
```

```java
class java.lang.Object
interface java.lang.Comparable
class [Ljava.lang.String;
class [[I
interface java.lang.Override
class java.lang.annotation.ElementType
class java.lang.Integer
void
class java.lang.Class
```



<br/>

## 反射相关的主要API

```java
java.lang.Class: 代表一个类

java.lang.reflect.Method: 代表类的方法

java.lang.relfect.Field: 代表类的成员变量

java.lang.relfect.Contructor: 代表类的构造器
```



在Object类中定义了一下的方法，此方法将被所有的子类继承

```java
 public final Class getClass()
```



此方法返回值的类型是一个Class类，此类是Java反射的源头，实际上所谓的反射从程序的运行结果来看也很好理解，即：通过对象反射求出类的名称。

对象通过反射后可以得到的信息：某个类的属性、方法和构造器、某个类到底实现了那些接口。对于每个类而言，JRE都为其保留一个不变的Class类型的对象。一个Class对象包含了特定的某个结构`(class/interface/enum/annotation/primitive type/void[])`的有关信息。

- Class 本身也是一个类
- Class 对象只能由系统建立对象
- 一个加载的类在JVM中只有一个Class的实例
- 一个Class对象对应的是一个加载到JVM中的.class文件
- 每个类的实例都会记得自己是由哪个Class实例产生的
- 通过Class可以完整地得到一个类中地所有被加载的结构
- Class类是Reflection的根源，针对任何你想动态加载、运行的类，唯有想获得相应的Class对象



Class类中的常用方法

| 方法名                   | 功能说明                    |
| :----------------------- | --------------------------- |
| `Class.forName()`        | 动态加载类                  |
| `newInstance()`          | 根据对象的class新建一个对象 |
| `getSuperclass()`        | 获取继承的父类              |
| `getInterfaces()`        | 获取继承的接口              |
| `getDeclaredFields()`    | 获取字段名字                |
| `getDeclaredMethods()`   | 获取当前类的所有方法        |
| `getConstructors()`      | 获得所有的构造函数          |
| `getModifiers()`         | 反射中获得修饰符            |
| `getPackage()`           | 反射中获得package           |
| `getField(String name）` | 反射中获得域成员            |
| `getFields()`            | 获得域数组成员              |
| `isAnnotation()`         | 判断是否为注解类型          |
| `isPrimitive()`          | 判断是否为基本类型          |
| `isArray()`              | 判断是否为数组类型          |
| `isEnum()`               | 判断是否为枚举类型          |
| `getClassLoader()`       | 获得类的类加载器            |
| `getMethods()`           | 获得公共的方法              |



<br/>

## 获取类的运行时结构



准备工作

```java
package com.qibria.reflect;

public class Person {

    protected String name;

    protected Integer age;

    public Person() {
    }

    public Person(String name, Integer age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "People{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}
```

```java
package com.qibria.reflect;

public class Student extends Person {

    private String grade;

    public Student() {
        super();
    }

    public Student(String name, Integer age, String grade) {
        super(name,age);
        this.grade = grade;
    }

    public String getGrade() {
        return grade;
    }

    public void setGrade(String grade) {
        this.grade = grade;
    }

    @Override
    public String toString() {
        return "Student{" +
                "name='" + name + '\'' +
                ", age=" + age +
                ", grade='" + grade + '\'' +
                '}';
    }

    public static void main(String[] args) {
        Student student = new Student("张三", 18, "三");
        System.out.println(student);
    }
}
```



```java
package com.qibria.reflect;

import java.lang.annotation.ElementType;

public class ReflectMain {
    
    public static void main(String[] args) throws ClassNotFoundException {
        
    }
    
    private static void getRuntimeClassStructure() throws ClassNotFoundException {
        
        Class<?> aClass = Class.forName("com.qibria.reflect.Student");

    }

}

```



### 操作对象



1.获取class对象

```java
Class<?> aClass = Class.forName("com.qibria.reflect.Student");
```



2.获取Class的构造器

无参构造器

```java
Constructor<?> constructor = aClass.getConstructor();
```

全参构造器

```java
Constructor<?> constructor = aClass.getConstructor(String.class, Integer.class, String.class);
```



3.使用构造器创建对象实例

```java
Student student = (Student)constructor.newInstance("qibria", 19, "三");
```



### 操作属性

#### 获取所有属性

`getDeclaredFields()`获取所有属性，包括私有、静态、常量都可以获取到，但是无法获取到父类的属性，如果想要获取到父类的属性可以先获取父类Class再获取属性`Field[] declaredFields =  aClass.getSuperclass().getDeclaredFields();`

然后使用获取到的`Filed`对象可以操作相应的属性，如果操作私有属性要使用`setAccessible(true)`成员方法设置权限

注意点: 不要修改final修饰的属性，常量不可改，正常读取即可

```java
Field[] declaredFields = aClass.getDeclaredFields();
for (Field field : declaredFields) {
    field.setAccessible(true); // 获取private属性时候要设置权限, 否则会抛非法异常
    String name = field.getName();
    Object value = field.get(student);
    System.out.println(field.getName() + ": "+field.get(student));
}
```

```java
grade: 三
test01: null
test02: test02...
```



`getFields()` 只能获取到`public`的属性

```java
Field[] fields = aClass.getFields();
```



#### 获取指定的属性

`getDeclaredField("属性名")` 获取指定属性包括私有属性

`getField("属性名")`获取指定public属性

```java
Field grade = aClass.getDeclaredField("grade");
Field grade1 = aClass.getField("grade");
```

注意: 父类属性先获取父类Class再获取属性`aClass.getSuperclass().getDeclaredField("name");`

```java
String name = aClass.getSuperclass().getDeclaredField("name");
```



获取到Field对象后，接着就是怎么操作Field对象

这里只说两个方法，就是`get()`和set`("具体的对象名")`两个方法

get方法就是直接获取属性的值

set方法就是设置属性的值

下面就是操作`name`属性

```java
String string_01 = (String)name.get(student);
name.set(student,"七七七");
```



### 操作方法

获取所有方法，包括`private`和`static`方法，下面的操作和操作属性的方式大同小异，就不一一演示

```java
Method[] declaredMethods = aClass.getDeclaredMethods();
```



获取指定方法

```java
Method setGrade = aClass.getDeclaredMethod("setGrade", String.class);
```

第一个参数是方法名，后面的参数是方法的入参类型



获取到方法的`Method`对象后，接下来是怎么操作`Method`对象

使用`invoke`方法进行操作

  `invole`方法第一个参数为操作的对象示例，后面是方法的参数

```java
setGrade.invoke(student, "六");
```

