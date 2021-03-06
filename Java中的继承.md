# Java中的继承

## 超类和子类

当两个类之间存在is-a这样的关联的时候，我们称这种关系为继承关系。

例如猫is-a动物，于是猫类继承于动物类，猫类成为动物的子类，动物类成为猫类的超类。

java不支持多继承，但是支持接口，应当谨记，继承是属性（数据、概念）上具象化，而不是行为（功能）上的。例如鸟是动物，同时鸟是飞行物，但是鸟是动物，这是概念（物质）上决定的，而鸟是飞行物，这是由鸟会飞而决定的。所以这种情况，应当认为鸟是继承于动物，同时实现了飞行接口。

## 成员限定符

java类中成员的限定符有三种public、protected（默认）、private。

区别如下：

1. private：只能在类中访问，不能在类外访问（包括对象访问、子类super访问都不行）。
2. public：完全公有，可以通过所有方式访问。
3. protected：只能由同包、子类访问。

关于protected的解释：对于如下所示的结构，ClassC继承于ClassB，ClassB继承于ClassA。

![image-20200731140117902](https://gitee.com/ame-lm/PicBed/raw/master/img/20200731140119.png)

将除了本类中访问以外的访问方式分为两种：一种是通过对象指针访问，一种是子类通过super指针访问。

protected的作用归纳为：对象指针仅限同包访问、super可以跨包访问。

1. Main可以通过对象指针访问A、B的protected成员，不能访问C的protected成员。因为对象指针只能同包访问。
2. ClassC内部可以通过super访问A、B的protected成员，因为C是A、B的派生类。
3. 由于继承的关系，所以C的对象指针可以看做A、B的指针，因此实际上在Main中可以通过c.proA的形式访问A的protected成员。

## 覆盖与多态

在子类中可以覆盖超类的属性或方法。

覆盖属性：属性不能覆盖，属性只是新增一个同名属性（超类的属性还在，子类一共有两个属性，一个属于自己一个属于超类），根据指针的类型不同会指向不同的属性。

覆盖方法（重写）：方法可以覆盖，子类的同名方法会替换掉原来的方法（可以理解为超类的方法已经不存在了）。

示例如下：

``` java
//Main.java
package com.ame;

public class Main {
    public static void main(String[] args) {
        ClassC c = new ClassC();
        //三个x都存在
        System.out.println(c.x);//3
        System.out.println(((ClassB) c).x);//2
        System.out.println(((ClassB) c).x);//1

        //只存在一个f
        c.f();//3
        ((ClassB) c).f();//3
        ((ClassA) c).f();//3
    }
}
//ClassA.java
package com.ame;

public class ClassA {

    public int x = 1;

    public void f() {
        System.out.println(1);
    }
}
//ClassB.java
package com.ame;

public class ClassB extends ClassA {

    public int x = 2;

    @Override
    public void f() {
        System.out.println(2);
    }
}
//ClassC.java
package com.ame;

public class ClassC extends ClassB {

    public int x = 3;

    @Override
    public void f() {
        System.out.println(3);
    }
}
```

### 多态

多态是指通过对象指针进行方法调用的时候，并不是根据指针的类型去进行访问的，而是根据对象的实际类型进行调用。例如上面的例子。

这种现象可以叫做**动态绑定**，也就是并不会和变量（类型）关联，而是根据变量指向的对象动态地绑定方法。

#### java方法调用的理解

（自己查资料加上一些自己的理解，不一定对。）

当调用方法时，比如x.f(args)，x是C类型的一个变量。会发生以下步骤：

1. 编译器首先在C类（包括其超类的public方法）中找匹配的方法（名字匹配、参数匹配）。如果超类和C类中有相同的方法（包括名字和参数），则C类的会优先被匹配。

2. 匹配成功之后检查方法修饰符，如果是private、static、final、构造，那么编译器就可以直接调用了，这是**静态绑定**。

3. 如果是public方法，那么采用的是**动态绑定**。

   1. jvm为每个类维护了一张方法表，表中指向了每个方法的实际入口（如果子类重写了方法，那么子类的方法表中的入口会改变）。

   2. 当调用buplic方法时，编译器根据变量类型查方法表，找到要调用方法f在方法表中的位置（下标）。

   3. 当运行时根据变量指向的对象，加载对象对应的方法表（而不是指针所属类型的方法表），然后根据上一步找到的下标在这个方法表中找到方法入口，进行跳转。

   4. 因为方法表的组织原则是这样的（A继承于B，B继承于Object）：

      | Object类的表   | B类的表                   | A类的表                   |
      | -------------- | ------------------------- | ------------------------- |
      | Object类的方法 | Object类的方法            | Object类的方法            |
      |                | B类的方法（指向B的f方法） | B类的方法（指向A的f方法） |
      |                |                           | A类的方法                 |

      也就是超类的方法在子类的方法表中具有相同的下标，这样就可以通过下标进行动态绑定。

      比如B b=new A（），通过b.f()调用的时候。

      编译器根据指针b的类型B查B的方法表，下标为2（假定），然后运行的时候，加载对象（A类）的方法表，并取第2项，这就获得了指向A的f方法的指针。

4. 补充：编译的时候时不知道具体运行时的方法表是什么的，所以只能根据指针类型的方法表进行编译。这也印证了为什么B b=new A()，不能通过b.g()调用A特有（并不是从B继承来）的方法g；因为编译这一关就过不了，更别说什么动态绑定了。

## 阻止继承-final

被final修饰的类不允许被继承。

被final修饰的方法不允许被重写。

被final修饰的域不允许被修改。

被final修饰的方法，采用静态绑定，在编译时就可以确定地址，避免了动态绑定带来的开销。

按照传统的编译器思路，需要对所有的public方法都实施动态绑定。然而jvm中的即时编译器可以编译时实时检测public方法是否真的被重写了，如果没有被重写，那就按照静态绑定方式编译。

## 类型转换

类型转换可以分为隐式转换和显示转换。

子类->超类是隐式转换，即不需要写更多的代码，你可以直接把子类对象当做超类使用。

超类->子类是显示转换，需要显示地书写代码进行转换（语法略）。如果不是超类-子类的关系进行显示转换，jvm会抛出异常，最好在显示转换时使用instanceOf运算符进行判断。

## 抽象类

在自下而上的继承层次中，越是上层的类越具有通用性，更加抽象。

在OOP中所有的对象都是由类生成，然而并非所有的类都可以生成对象，如果类包含的信息不足以描述一个具体的对象，那这样的类就是抽象类。

我也说不清楚抽象类怎么定义，随便说说吧。。。

抽象类是一种高层次的抽象，是一个或者多个真实类的共性的抽象，仅凭这些共性无法确定一个具体的对象，然而又需要这样的抽象类来统筹其各个子类。比如我们需要一个容器，能存放猫、狗、猪等对象，容器不论定义成什么类型，都无法满足要求，只能寻求更高层次的抽象-动物类，让猫、狗都继承于动物，就可以用动物容器来放了。

总之抽象类最大的特点就是**无法被实例化**。

抽象类用abstract定义，抽象类包含零个或多个抽象方法，抽象方法没有方法体，由子类自行实现（如果子类也是抽象类则可不必）。

包含抽象方法的一定是抽象类，抽象类未必一定包含抽象方法。但是就算没有抽象方法，依然无法实例化。

抽象类可以包含具体的属性、方法、抽象方法。但是建议抽象类尽量不要包含具体的 方法，将具体的方法教给子类实现。

## Object类

java默认Object类是所有类的始祖类，所有的类都是Object类的子类，但是并不需要显示继承，如果没有显示继承，那默认继承自Object类。

C++中没有始祖类这个概念，但是所有的指针都可以转为void*指针。

Object类型的变量可以指向任意对象，因为它是始祖类。

Object类中有几个重要的方法。

### equals方法

该方法用于检测两个对象是否相等（两个对象是否具有相同的状态、作用，而非检测是否是同一个对象，当然同一个对象肯定想等啦）。

检测每一种对象是否相等（状态、作用）都应该有其特定的规则，因此当需要比较时，一般都需要重写该方法。

Object类中默认实现是仅仅比较是否是同一对象：比较双方是同一对象返回true，否则返回false。如下：

``` java
    public boolean equals(Object obj) {
        return (this == obj);
    }
```

在子类中重写equals方法时，一般先调用超类的equals，如果超类不等，则肯定不等；如果相等，则再执行子类的判断逻辑。

在java规范中equals方法遵循自反（自己等于自己）、对称、传递、一致（指针x和指针y一直没变，则他们比较的结果也不应改变）、null返回false的特性。

#### 两种实现思路

一般实现equals方法时，流程如下（其中**某些步骤可能是由超类提供**）：

1. 检测是否是同一对象。
2. 检测参数是否为空。
3. 检测是否同类（严格，继承算不同类）。
4. 强制类型转换，然后进行更进一步的比较。

在步骤3，对于比较双方不同类的处理很有争议。

一种做法是当两者不同类时返回false，通过.getClass()方法获取Class对象进行比较。

然而在AbstractSet类中，equals方法的语义是检测两个集合是否有相同的元素，对于HashSet和TreeSet，他们使用不同的算法实现查找操作，然而两个实现虽然不同类，但在语义上却可能相等，这时候如果采用上一种做法显然不对。

归纳如下：

若子类自己能够拥有相等概念，则由于对称性限制，需要采用比较Class对象的方式比较。

若相等概念由超类决定（所有的子类并无语义上的扩展，仅仅是实现不同），那么应在超类中定义equals方法，并使用instanceOf检测，这样可以在不同子类对象之间比较；超类的equals方法应当定义成final，不允许子类改变其语义。

注意到AbstractSet的equals方法并非是final的，这是预留给子类实现更高效的比较算法的（但是子类重写之后不应改变其语义，仅仅只需要改变其算法实现过程）。

#### 数组域的比较

数组本身没有实现equals方法，仅仅是从Object类继承而来，所以语义上是错误的，这时候可以用Arrays.equals()方法比较。



### hashCode方法

hashCode，译为散列值，也即摘要，是一个int值。一个对象的hashCode为一个对象的摘要，**散列值要求：相等对象应当具有相同的散列码，但是具有相同散列码的不一定是相等对象；散列值不同的对象不等**。

Object中的hashCode方法返回对象在内存中的地址，这和Object提供的equals方法不冲突：相等意味着相同，也就意味着地址相等；地址不等则不是同一个对象，于是不相等。

一般来说，如果重写了equals方法，那么也应该重写hashCode方法，保证hashCode满足相应的特性。这主要是为了方便用户将对象插入到散列数据结构中。

#### Objects.hash、Objects.hashCode方法

当重写hashCode方法时，很有可能需要计算对象域的hashCode，固然可以通过.hashCode的方式来调用，但是可以做得更好，建议使用null安全的java.util.Objects.hashCode(Object obj)和java.util.Objects.hash(Object...values)计算，对于null值，hash为0；

#### 数组域的hash

数组对象本身没有实现hashCode方法，仅仅是从Objects类继承而来，所以语义上是不对的，这时候可以用Arrays.hashCode(type[] a)计算hash。

### toString方法

toString方法用于将对象成字符串。大多数toString方法返回值遵循类名[key=value]的形式。

#### 数组的toString

数组对象本身没有实现toString方法，仅仅是继承自Object类，因此语义上不完整，这时候可以用Arrays.toString()方法。

