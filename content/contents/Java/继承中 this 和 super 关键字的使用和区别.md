---
title: "继承中 This 和 Super 关键字的使用和区别"
date: 2024-01-17T22:24:20+08:00
categories: Java
draft: false
---
在面向对象模式开发中，经常会用到 this 和 super 两个关键字，在实体类的编写中，一般会用到较多的 this 关键字，本文通过举例说明来了解 this 和 super 关键字的用法和异同之处。

<!--more-->

## this 关键字

this 是自身的一个对象，代表该对象本身，也就是说这是指向对象本身的一个指针。在实体类的编写中，this 关键字常被用来区分形参和成员变量。

```java
class Student {
    private long serial = 1;
    private int grade;
    private String name;

    public Student(
        long serial, int grade, String name
    ) {
        this.serial = serial;
        this.grade = grade;
        this.name = name;
    }

    /* Getter & Setter */
}
```

在 Student 类的构造函数中使用 this 关键字可以对类中定义的成员变量进行直接访问。这是因为 this 关键字是自身的一个对象，可以使用对象来访问类內部的成员变量。

在新建对象的时候通常会使用 `Object obj = new Object()` 语法，new 之后的 `Object()` 调用该类中对应的构造方法。换言之，this 关键字也能够作为一个方法来执行，它调用该类中对应的构造函数。

```java
class Student {
    private long serial = 0;
    private String name;

    public Student() {
        this.serial = 1;
    }

    public Student(String name) {
        this();
        this.name = name;
    }

    public static void main(String[] args) {
        Student student = new Student("scholar7r");
        System.out.println("Serial: " + student.serial);
        System.out.println("Name: " + student.name);
    }
}

// --- Output
// Serial: 1
// Name: scholar7r
```

示例代码中包含了 `serial` 和 `name` 两个成员变量，`serial` 变量的默认值为 0，以及一个不需要传入任何参数的构造函数和一个和需要传入字符串的构造函数。无参数构造函数只要被执行，就会将成员变量 `serial` 的值改为 1，这个无参数的构造函数在包含字符串参数的构造函数中被调用。需要传入字符串的构造函数首先是使用 `this()` 来调用无参数的构造函数，然后将成员变量 `name` 的值修改为传入的字符串。在主函数中实例化 Student 对象，并且传入值 `scholar7r`。这时会调用需要字符串参数的构造函数，于是就完成了成员变量的初始化。

## super 关键字

super 关键字可以理解为指向超类（即父类）的指针，这个所指向的超类是继承级别最近的一个类。

```java
class Animal {
    private String name;

    public Animal(String name) {
        this.name = name;
    }

    public void makeSound() {
        System.out.println("Animal makes a sound");
    }

    public String getName() {
        return name;
    }
}

class Dog extends Animal {
    private String breed;

    public Dog(String name, String breed) {
        super(name);
        this.breed = breed;
    }

    @Override
    public void makeSound() {
        super.makeSound();
        System.out.println("Dog barks");
    }

    public String getBreed() {
        return breed;
    }
}

class Trigger extends Animal {
    public Trigger(String name) {
        super(name);
    }

    @Override
    public void makeSound() {
        super.makeSound();
        System.out.println("Trigger growls");
    }
}
```

`Dog` 类和 `Trigger` 继承 `Animal`，两者都调用了 super 关键字调用了父类的构造函数，并且传入字符串初始化成员变量。然后重载父类中的 `makeSound` 方法，输出不同的内容。super 关键字的作用就是访问父类的构造方法、字段或者方法。与 this 关键字的使用本质上是类似的，区别只在于所指向的对象不同。
