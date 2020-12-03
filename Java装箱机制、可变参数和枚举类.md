# Java装箱机制、可变参数和枚举类

## 装箱机制

java中基本数据类型（int、double等）不是对象，为了将基本数据类型转换成相应的对象，java中设计了一系列**包装类**，用来作为基本类型的对象版本替代。

包装类是不可变的。包装类自增时其实是改变了引用值。如下：

``` java
    //验证包装类的不可变
    public static void f1() {
        Integer a = 0;
        Integer b = a;
        System.out.println(a == b);
        a++;
        System.out.println(a == b);
    }
```

执行结果：

``` shell
true
false
```

java中，编译器会根据需要，随时在基本类型和包装类之间转换，这就是**装箱机制**。

装箱机制是编译器层面的而不是虚拟机层面的，也就是装箱、拆箱在编译的时候通过植入代码完成，而不是在运行时进行。

### 相等的判定

包装类既可以看做对象，又可以看做基本类型，因此包装类相等的判定和基本类型不太一样。

当执行装箱动作（int->Integer）时，-128到127之间的数字装箱后，结果引用会指向**固定的256个对象**中，而对于这个范围之外的数字，每次装箱都会新建一个包装类对象。

因此当用equals判定包装类之间的相等不会有问题，然而当用==判定时，可能会出现一些不同：

``` java
    //包装类相等判定验证
    public static void f2() {
        Integer a = 127;
        Integer b = 127;
        System.out.println("a==b:" + (a == b));
        System.out.println("a.equals(b):" + a.equals(b));
        a = 128;
        b = 128;
        System.out.println("a==b:" + (a == b));
        System.out.println("a.equals(b):" + a.equals(b));
    }
```

调用结果：

``` shell
a==b:true
a.equals(b):true
a==b:false
a.equals(b):true
```

由于上述原因，对包装类进行相等判定的时候，**最好使用equals方法**。



## 可变参数

在java SE5.0之前，java的所有方法都有固定数量的参数，然而java SE5.0新增了可变参数机制。

变参方法是指方法在调用时，参数数量可以不固定。比如printf方法。

定义和使用可变参数实例如下：

``` java
  //这是一个变参方法
    public static void f3(int... values) {
        System.out.println("start.")
        for (int item : values) {
            System.out.println(item);
        }
        System.out.println("end.")
    }
```

f3();调用结果：

```shell
start.
end.
```

f3(1);调用结果：

``` shell
start.
1
end.
```

f3(1,2,3);调用结果：

``` shell
start.
1
2
3
end.
```

可变参数是通过数组实现的，对于方法内部来说type...values和type[] values其实是一样的。只不过将参数包装成数组的步骤交给了编译器，而非用户。

另外需要注意的是，可变参数**只能有一个**，只能放在**末尾**，调用时长度**可以为0**.

## 枚举类

当一个类，他的取值仅有有限个（在某个特定范围内）时，可以将其定义为枚举类。

定义实例1：

``` java
enum Size{SMALL,MEDIUM,LARGE}
```

以上定义相当于定义了一个Size类，Size类有且仅有3个实例，通过Size.SMALL这样的方式获得。

定义实例2：

```java
public enum Size{
    SMALL("S"),MEDIUM("M"),LARGE("L");
    private Size(String s){
        System.out.println(s);
    }
}
```

当第一次试图获取枚举实例的时候，jvm会调用枚举类的构造器构造所有的实例（这里是3个），如果没有特别指定，调用默认构造器。

这里我自己定义了构造器，构造对应的实例时传入构造器的参数写在枚举值后面。

如果重载了多个构造器，括号里的参数值可以指定其中一个，当无参数时括号可以省略（实例1）；

当然也可以给枚举类添加其他属性、方法等，使用方式和类一般无二，但是无法自己创建实例，只能有jvm创建。

枚举引用比较时直接==即可，因为同一枚举值永远只有一个相应的对象实例。