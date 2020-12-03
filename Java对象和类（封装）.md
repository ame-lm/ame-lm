

# Java对象和类（封装）



## 面向对象

传统的程序设计方式是面向过程的，然而面向过程方式设计程序有很多缺点：

1. 在某些场景，面向过程不符合物理世界的表现方式；而面向对象更加符合。
2. 在大规模开发中，面向过程设计的程序有很多过程，操作很多全局变量，难以调试；而采用面向对象，可以将不同的过程、变量封装到不同的类中，降低程序耦合性，便于协作开发、调试。

## 类和对象

1. 类是一组数据概念以及定义在其上的操作的集合，类没有实体，只存在于概念上。

2. 对象是类的实体，类的一个特例，比如人是类概念，小明这个人是人的实体对象。

3. 程序设计中识别类：在分析问题的过程中出现的名词，很多都对应类（或者是基本数据类型），而对应的动词则对应了对应的操作。
4. 类之间的关系：依赖（use-a）、聚合（has-a）、继承（is-a）

## 预定义类和对象的使用

包括创建、修改、丢弃（gc自动回收）、setter、getter等。

## 自定义类

类一般由构造、属性、方法组成。

### 构造

构造的特点：

1. 构造与类同名
2. 构造无返回值
3. 构造可以有任意多个参数
4. 构造可以重载
5. 构造伴随new关键字

#### 重载

构造可以重载。

如果没有显示定义构造，则java自动生成一个默认无参构造，将所有未初始化的域设置为默认值。

然而一旦显示定义了一个构造，不管是不是无参构造，java都**不会再提供默认构造**。

#### 初始化代码块

在类声明时，可以定义一个或者多个初始化代码块，只要构造对象，初始化代码块就会执行。如：

``` java
package com.ame;

public class Employee {
    private static int nextId;

    private int id;
    private String name;
    private double salary;

    //初始化代码块
    {
        id = nextId;
        nextId++;
    }
    
    public Employee(String name, double salary) {
        this.name = name;
        this.salary = salary;
    }

    public Employee() {
        name = "";
        salary = 0;
    }
}

```

不论使用哪个构造，都会在构造执行之前初始化代码块。如果有多个代码块，则按序执行。

初始化代码块可以用static修饰，用于类加载时执行（只执行一次），初始化静态域。如：

``` java
package com.ame;

public class Employee {
    private static int nextId;

    private int id;
    private String name;
    private double salary;

    static {
        nextId = 2;
        System.out.println("static.");
    }

    //初始化代码块
    {
        id = nextId;
        nextId++;
        System.out.println("id:" + id);
    }

    public Employee(String name, double salary) {
        this.name = name;
        this.salary = salary;
    }

    public Employee() {
        name = "";
        salary = 0;
    }

    public static void main(String[] args) {
        new Employee();
        new Employee();
        new Employee("", 0);
    }
}

```

执行结果：

``` shell
static.
id:2
id:3
id:4
```

#### 析构

C++中有显示析构方法，当对象被弃用的时候进行清理。

然而java有垃圾回收机制，不提供析构。

但是考虑到当对象不再使用时可能需要释放一些外部资源（如打开的文件），因此在java中，可以为类添加finalize方法，finalize将在垃圾回收前调用，但是具体的调用时间不确定（因此尽量不要依赖fnialize方法进行回收）。

### 方法

方法的参数分为隐式参数和显示参数。

隐式参数即this指针，this指向调用该方法的对象。

显示参数即为方法声明时显示定义的参数。

方法调用的参数为值传递方式，也就是说在传参时会进行一次拷贝。

​	注意：引用类型传参的时候只是将变量值（指针或地址）进行拷贝了，但拷贝后的指针仍然指向最初的对象。

所有的方法可以进行重载，重载只要求同名不同参。

### 属性

1. 强烈建议类中的属性设为私有，然后为私有的属性提供setter和getter方法。

   这样做有很多好处，例如可以在setter方法中对操作的合法性进行校验。

2. 尽量不要返回对象私有属性的引用。而应该返回其拷贝（clone方法）。

3. 不可变的属性应该用final修饰。

   final域只能在初始化时被赋值一次。

   注意：final域变量不可变并不意味这所指向的对象不可变。

4. 属性初始化一般有三种方式：默认值（尽量不要这样）、声明时赋值、构造赋值、静态初始化代码赋值。

### 基于类的访问权限

方法可以访问类**所有的对象**的私有数据。如：

``` java
package com.ame;

import java.util.Objects;

public class Person {
    private int id = 0;

    public Person(int id) {
        this.id = id;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Person person = (Person) o;
        return id == person.id;//这里通过person引用访问了私有属性id
    }

    @Override
    public int hashCode() {
        return Objects.hash(id);
    }
}
```

### 静态域和静态方法

静态域和静态方法用static修饰，静态成员由类所拥有，不属于具体的对象，通过类名访问。

静态域有静态变量、静态常量。

静态方法和普通的方法相比，没有this指针这个隐式参数，但是可以访问自身所在类的静态成员。

一般的main方法是一个常见的静态方法，也是java代码执行的入口。

### 包

java使用包组织类，将类按照包放在一起，不同包中的类可以同名，对于java编译器来说，嵌套的包是两个包，如java.util和java.util.jar是两个无关联的包（只是名字有重叠而已）。

#### 包的使用

略。

#### 包的导入

使用一个包中的类，可以通过全限定名访问，也可以通过import引入。如：

``` java
package com.ame;

import java.util.Random;//导入Random

public class Main {
    public static void main(String[] args) {
        java.util.Date d = new java.util.Date();//全限定名直接访问
        System.out.println(d);

        Random random = new Random();//类名访问
        System.out.println(random.nextInt());
    }
}

```

执行结果：

``` shell
Fri Jul 31 09:27:52 GMT+08:00 2020
400450209
```

假如两个包中有同名类，比如java.util.Date和java.sql.Date，那么一般按照下面所述进行处理：

1. 两者若导入的形式都是import .*，直接使用类名Date会发生冲突，无法编译，因为根据类名Date匹配到了两个类（不用Date则不会错）。

   ``` java
   import java.util.*;
   import java.sql.*;
   /*
   * 不使用Date类就不会报错，用了Date类才会报错
   */
   ```

2. 若两者都是import .Date形式，则由于引入的类名冲突，编译错误（还没用Date就错了）。

   ``` java
   import java.util.Date;
   import java.sql.Date;
   /*
   * 两个import冲突，引入时就会错。
   */
   ```

3. 如两者导入形式不同（一个import .*，一个import .Date（也可以加上import .\*））则后者优先级更高，可以使用类名Date（后者）

   ``` java
   import java.util.*;//可要可不要
   import java.sql.*;
   import java.util.Date;
   /*
   * 通过类名Date只能使用java.util.Date
   * 但仍然可以通过全限定名访问java.sql.Date访问java.sql.Date类。
   */
   ```

4. 若两者都需要使用，则只能最多显示引入（通过全限定名引入）一个，然后对没有显示引入的类使用全限定名进行访问。

#### 静态引入

import语句除了引入类，还可以用来引入类中的静态成员，如：

``` java
package com.ame;

import static java.lang.Math.*;

public class Main {
    public static void main(String[] args) {
        System.out.println(PI);//没有通过Math类名，直接使用了Math.PI
    }
}

```

执行结果：

``` java
3.141592653589793
```

#### 包作用域

对于类、类中成员，如果使用public修饰，代表完全公开，所有类的所有成员都可以访问的；

如果使用private修饰（类不能用private，仅限于类中成员），代表完全私有，只有定义他们的类可以访问；

如果不使用修饰符或使用protected（默认修饰符）修饰，则代表受保护，仅同一个包中的类中方法可以访问到该成员，而包以外的不能访问。