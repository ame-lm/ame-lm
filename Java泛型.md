# Java泛型

[toc]

泛型机制是在Java SE5.0新增的。

泛型的类型由专门的类型变量来限定。

按照泛型的应用场景，分为泛型类和泛型方法。

## 泛型类

泛型类是指定义类时使用了一个或多个泛型变量。

语法：

``` java
class Test<T>{...}
```

## 泛型方法

泛型方法是指在定义方法时使用了一个或多个泛型变量。

语法：

``` java
public <T,U> void f(){...}
```

泛型方法调用时，如果可以根据上下文自动推导出泛型变量的指，可以不用指定泛型变量。

## 泛型限定符

有时候需要限定类型变量的范围。例如需要设计一个泛型方法，对传入的两个参数比较大小（compareTo），这个时候如果不加以限定，传入的参数可能是不可比较的（没有实现Comparable接口），就会引起预料之外的错误。此时需要将类型限定在“实现了Comparable接口的类型”中。

如果有必要，可以同时限定多个范围的交集（例如：实现了Comparable接口，且必须是Number的子类）。

语法：

``` java
public <T extends Comparable & Number> int f(){...}
```

## 泛型与虚拟机

在虚拟机中，并没有定义泛型机制，泛型机制是在编译器层面定义的。

从编码到运行泛型代码通常需要经过一些编译器层面的处理。

### 类型擦除

对于虚拟机来说，它只认常规类型，不认泛型，因此在编译时，编译器会做类型擦除，将泛型类用其**原始类型**来表示。原始类型中的泛型变量用**限定类型**来替代。

1. 如果没有限定符。限定类型是Object
2. 如果有单个限定范围。限定类型是限定符之后的类型。
3. 如果有多个限定范围。限定类型是限定符后第一个类型。

### 翻译泛型表达式

在运行泛型方法过程中，如果有表达式带有泛型定义的变量，编译器会自动对其插入强制类型转换语句。一般包括返回时、存取泛型变量时。

### 翻译泛型方法（桥方法）

编译器在编译泛型方法的时候，会对泛型方法进行类型擦除，而类型擦除之后，可能会和Java的 多态机制产生冲突。例如：

``` java
class T1<T> {
    public void f(T t) {
        System.out.println("T1:" + t);
    }
}

class T2 extends T1<Number> {
    public void f(Number t) {
        System.out.println("T2:" + t);
    }
}
```

对于T1，类型擦除之后剩下``public void f(Object t);``

对于T2，有``public void f(Number t);``（严格意义上是重载，而非重写）

​	以及从T1继承来的``public void f(Object t);``

当通过T1类型的指针调用T2对象的f方法时，多态要求我们调用``public void f(Object t);``而泛型的实现要求我们调用``public void f(Number t);``，这时，多态机制和泛型机制发生了冲突。

在Java中，引入了**桥方法**机制，来解决这个冲突，在T2中，有两个方法``public void f(Object t);``和``public void f(Number t);``。前一个是通过继承得来（并非T2重写）后一个泛型定义的，但是严格意义上来说他们只是重载而已，当触发多态机制时，根据多态机制，Java将调用``public void f(Object t);``方法，Java将这个通过继承得来的方法定义为**桥方法**，然后在这个桥方法中调用``public void f(Number t);``即：

``` java
@Override
public void f(Object t){
    f((Number)t);
}
```

这段代码是编译器生成的，并非用户自定义。

若是用户自己重写了该方法，那就不存在多态机制和泛型机制冲突了，也就不需要桥方法了。

**因此，桥方法的合成仅仅是为了保持多态性。**



## 泛型的局限性（几乎都是由类型擦除带来）

### 不支持基本类型

泛型的类型变量不能用int、double等，只能用Integer、Double等包装器对象，因为int匹配类型擦除后的原始类型。

### 运行时类型查询只适用于原始类型

``` java
//if(a instanceOf Pair<String>){...}//错误

if(a instanceOf Pair){...}//正确
```

因为类型擦除之后，泛型类的类型只有原始类型。

同理，getClass方法总是会返回原始类型。

### 不能创建参数化类型数组

``` java
T1<String> t1;//可以声明
//t1 = new T1<String>[];//不可以创建
```

可以声明这样的数组变量，但是不能创建。

原因是Java中数组会记住自己的元素类型，当存储元素时，会进行类型检查（编译期间）。对于参数化类型，由于类型擦擦，将会使得这一步无法有效进行，尽管编译时正确，但是运行时可能出错。例如：

``` java
T1 t1=new T1<String>[10];
t1[0]=new T1<Integer>();
```

这段代码无法编译，假如可以编译，那么当类型擦擦之后，t1会允许T1<Integer>类型的变量存储，然而这是不符合数组规范的，后续如果将T1<Integer>当作T1<String>使用，可能就会引发一些运行时错误。

出于这个原因就，Java禁止创建参数化类型数组。

### Varargs警告

上面说过，Java不能创建参数化类型数组，然而却可以声明。

当使用变参函数时，由于可变参数会自动转变成数组，倘若可变参数是参数化类型，那就会创建参数化类型数组。这是由编译器内部创建的，因此Java并没有禁止这个特性，而是会给出一个Varargs警告。

这个警告可以通过SafeVarargs注解消除。如下：

``` java
    //Varargs警告
    @SafeVarargs
    public static <T> void f7(T... values) {
        System.out.println("hello world");
    }
```

可以编译运行。

### 不能实例化类型变量

当用泛型的类型变量直接创建一个对象时，由于类型擦除，会直接进行原始类型创建，这显然不是我们想要的。因此，Java禁止实例化类型变量。如下：

``` java
T t;
//t = new T();//这句非法
```

**解决方案**：在外部，通过将T的构造方法（方法引用）传入，利用函数式接口进行创建。如下：

``` java
    //不能实例化类型变量
    public static <T> void f8(Supplier<T> supplier) {
        T t;
        //t = new T()非法
        t = supplier.get();//合法
    }
```

通过``f8(String::new);``这样调用。

### 不能创建泛型数组

由于类型擦擦，创建泛型数组其实是创建原始类型数组，如果数组仅仅是作为一个类的私有实例，外部无法直接访问，那么声明为原始类型不会有任何问题，只需要在使用的时候进行强制类型转换即可。（ArrayList<E>正是采用这样的做法）。

然而，如果该数组需要对外提供访问权限，再采用这样的作法就会出问题。如下：

``` java
    //不能创建泛型数组
    public static <T> T[] f9() {
        //转换时编译器不会报错。
        return (T[]) new Object[10];
        /*
        * 类型擦除之后，new T[10]等价于new Object[10];
        */
    }

    public static void main(String[] args) {
        String[] strings = f9();
        strings[0]="hello";
        /*
        * 这里试图向Object类型数组存放String实例，运行时出错。
        */
    }
```

由于上述原因，Java禁止创建泛型数组。

**解决方案：**由调用者在外部将泛型数组的构造器传入，通过函数式接口创建。

``` java

    //不能创建泛型数组
    public static <T> T[] f9(Function<Integer, T[]> function) {
        //return (T[]) new Object[10];
        return function.apply(10);
    }

    public static void main(String[] args) {
        String[] strings = f9(String[]::new);
        strings[0] = "hello";
    }
```

成功运行。

### 不能在静态域、静态方法中使用泛型

静态域和静态方法无法使用泛型。

因为类型擦除之后，静态域中的所有类型变量都是同一个值（原始类型）。

### 其他

除上述问题之外，还有一些由于泛型的类型擦除机制引发的局限性，都是可以根据类型擦除进行简单分析的，既不常用，也不在此赘述。

## 泛型与继承

![image-20200805091459995](https://gitee.com/ame-lm/PicBed/raw/master/img/20200805091502.png)

1. ``Class<T>``和``Class<S>``没有任何关系，**就算T和S有继承关系也一样**。

2. 继承时，会将类型变量一并继承。

   例如ClassA extends ClassB``<String>``，那么在ClassA中 类型变量也是String，不能是别的。


## 通配符

   使用固定的类型参数，有诸多不便。

   因此Java设计者发明了泛型的通配符机制。

   对于泛型通配符，和泛型T有很重要的区别：**在泛型T中，所有出现T的地方都代表了同一个类型，T f()、T g()会返回同样的类型；而在通配符中? extends Fruit f()、? extends Fruit g()可能会一个返回Apple，一个返回Banana，两者唯一的联系是有共同的上界**。

   为了便于后面的理解，需要了解几条规则：

   1. 如上。T和?的区别。
   2. null是所有类的子类、Object类是所有类的超类。
   3. 向上类型转换是安全的，向下类型转换是不安全的。
   4. ?是一个具体的类型，但是不确定到底是哪个类型，而不是一个类型族。

   ### 上边界通配符

   1. 使用extends限定上边界。

   2. **无法使用多重限定**。

   3. 上边界通配符限定的变量**不能写（可以写null）、只能读（读到上边界类型）**。

      这是因为，上面所说，在T item在通配符下，变为? extends Fruit，这是一个无法确定的**具体类型**，编译器知道这个类型可能是Fruit、也可能是Apple，因此无法赋值。

      假如编译器认为它是Fruit，于是给其赋值Fruit类型，但实际上它的类型是Apple，这样就会引发错误。

   例如：

   ``` java
   package com.ame;
   
   class Plane<T> {
       public T getItem() {
           return item;
       }
   
       public void setItem(T item) {
           this.item = item;
       }
   
       private T item;
   }
   
   class Fruit {
       public String name = "fruit";
   }
   
   class Apple extends Fruit {
       public String name = "apple";
   }
   
   public class Test {
       //extends
       public static void f1() {
           Plane<Fruit> plane = new Plane<>();
           Plane<? extends Fruit> plane1 = plane;
   
           plane.setItem(new Apple());
           // plane1.setItem(new Apple());
   
           System.out.println(plane1.getItem().name);
       }
   
       public static void main(String[] args) {
           f1();
       }
   }
   
   ```

   ### 下边界通配符

   1. 下边界通配符使用super进行限定。

   2. 下边界通配符无法多重限定。

   3. 下边界通配符**可以写（下边界）、但是不可以读（读到的是Object类型）**。

      这是因为，通配符? super Apple虽然无法知道具体是哪个Apple的超类型，但是无论哪个都可以容纳Apple类型的值，所以可以赋值。

      读的时候，因为无法确定具体类型，所以读到的只能是Object类型。

   ``` java
   package com.ame;
   
   import com.sun.xml.internal.ws.addressing.WsaActionUtil;
   
   class Plane<T> {
       public T getItem() {
           return item;
       }
   
       public void setItem(T item) {
           this.item = item;
       }
   
       private T item;
   }
   
   class Fruit {
       public String name = "fruit";
   }
   
   class Apple extends Fruit {
       public String name = "apple";
   }
   
   public class Test {
   	//super
       public static void f2() {
           Plane<Fruit> plane = new Plane<>();
           Plane<? super Apple> plane1 = plane;
   
           plane1.setItem(new Apple());
           //这里会报错
           //plane1.setItem(new Fruit());
           
           //这里会报错
           //System.out.println(plane1.getItem().name);
           System.out.println(plane.getItem().name);
       }
   
       public static void main(String[] args) {
           f2();
       }
   }
   
   ```

   ### 无边界通配符

   没有边界限定的通配符，**既不能写（可以写null），也不能读（可以读Object）**

## Reference

https://mp.weixin.qq.com/s/0_nbg-axlpngrunTa7mWzQ