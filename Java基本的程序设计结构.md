# Java基本的程序设计结构

## 注释

注释有两种。

一种是行注释：

```java
//这是注释
```

另一种是块注释

```java
/**
* 这是块注释
*/
```

## 数据类型

java中一共有8种基本数据类型：整型、浮点型、布尔型、字符类型。

### 整型

| 类型  | 大小（字节） | 取值           |
| ----- | ------------ | -------------- |
| byte  | 1            | -2^7^~2^7^-1   |
| short | 2            | -2^15~^2^15^-1 |
| int   | 4            | -2^31^~2^31^-1 |
| long  | 8            | -2^63^~2^63^-1 |

1. 长整数类型后面跟有一个后缀L，如：1234424242L。
2. 十六进制数值有一个前缀0x，如：0x123。
3. java没有任何无符号类型。
4. 不加后缀的数字（即便有0x前缀）默认为int类型。

### 浮点类型

| 类型   | 大小（字节） | 取值          |
| ------ | ------------ | ------------- |
| float  | 4            | 有效数字6-7位 |
| double | 8            | 有效数字15位  |

1. double类型后缀为D。
2. float类型后缀为F。
3. 浮点值默认为double。
4. 可以用“尾数e指数”或“尾数p指数”形式表达浮点数。
   1. 尾数e指数：指数基数是10；用于十进制尾数。
   2. 尾数p指数：指数基数是2；用于十六进制尾数。
5. 浮点运算遵循IEEE 754规范，用正无穷、负无穷、NaN来表示溢出和出错。
   1. 在Float和Double类型中用常量POSITIVE_INFINITY、NEGATIVE_INFINITY、NaN表示上述三个值。
   2. 对NaN值的判定不应用“==”，而要用isNaN等静态方法；但是对无穷的判定可以用“\==”；如下。

``` java
    //验证java中浮点值NaN和无穷值
    public static void verifyNaN() {
        //NaN
        System.out.println(Double.NaN == Double.NaN);
        System.out.println(Double.isNaN(Double.NaN));
        //POSITIVE_INFINITY
        System.out.println(Double.POSITIVE_INFINITY == Double.POSITIVE_INFINITY);
        System.out.println(Double.POSITIVE_INFINITY + 1 == Double.POSITIVE_INFINITY);
    }
```

调用结果：

``` shell
false
true
true
true
```

### 字符类型

java中的字符类型和C中的字符类型不同。

1. C中字符大小为单字节，而java中字符大小为双字节。

2. C的字符类型表示Ascii字符。java的字符类型表示Unicode字符类型（可以表示中文）。
3. java中字符的三种形式：字符本身、转义字符、Unicode编码；如下。

``` java
    //java字符的三种表示
    public static void charFormat() {
        //字符本身
        System.out.println("字符本身：");
        System.out.println('哈');
        System.out.println('A');
        //转义字符
        System.out.println("转义字符：");
        System.out.print('A');
        System.out.print('\t');
        System.out.println('B');
        //Unicode编码
        System.out.println("Unicode编码：");
        System.out.println('\u03C0');//pai
    }
```

调用结果：

``` shell
字符本身：
哈
A
转义字符：
A	B
Unicode编码：
π
```

4. Unicode编码的目的是统一各种编码格式，由于最初的Unicode字符集并不大，因此java采用了双字节字符类型。然而后来Unicode字符集急剧扩充，双字节已经不够用了，下面解释java解决这个问题的办法。
   1. Unicode将所有字符编码，将这个编码值称为代码点（codePoint）。
   2. 对于0x0000-0xffff直接的字符，可以用双字节的char表示，因此对应字符集合称为基本多语言平面（BMP），而0xffff以上的成为增补字符集，增补字符集无法用char表示，需要用String类、char[]等表示。
   3. 代码点是指一个Unicode字符，而代码单元是指一个内存中的双字节char，对于bmp字符，一个代码点对应一个代码单元，而对于增补字符，一个代码点对应两个代码单元。
   4. String.length()，返回的是代码单元的数量，而不是代码点的数量。

``` java
    //java中的代码点、代码单元
    public static void codePoint() {
        char ch = '\u0041';//bmp字符
        //ch = '\u1d56b';//增补字符，这样写会报错，因为增补字符超出了char的表示范围

        String str = new String(Character.toChars(0x1d56b));//增补字符
        //length()方法返回的是代码单元(char)数量，一个增补字符占据两个代码单元；
        System.out.println(str + ":" + str.length());
    }
```

调用结果：

``` shell
𝕫:2
```

### 布尔类型

没什么好说的。

## 变量

1. java不区分变量声明和定义。而C区分（如int x;和extern int x;）。
2. java变量必须初始化才能使用，否则会报错。

### 常量

1. java中的常量用final关键字声明。是一种特殊的变量。
2. 常量只能被赋值一次，尽量在声明时赋值。
3. 常量一般全大写表示。
4. const是Java的保留字，但是并未被使用。

## 运算符

**算数运算符**：+、-、*、/、%及其对应的简写形式（+=...）

**自增运算符**：++、--；

**关系和boolean运算符**：==、||、&&、!、!=

**三目运算符：**略

**位运算符*：**&、|、^、~、<<、>>、>>>

### Math类

java内置了Math类。

Math类提供了各种数学函数，以及数学常量。

**数学函数：**幂运算、三角函数、反三角函数、指数函数、自然对数、取整等。

**数学常量：**Math.PI、Math.E。

## 枚举类型

枚举值的范围被约束在一个有限集合内，基本使用如下。

```	 java
    //java枚举类型的基本用法
    enum Size {
        L, S, M
    }

    public static void enumUsage() {
        Size s;
        s = Size.S;
    }
```



## 字符串（String类）

1. 字符串时Unicode字符序列。
2. String类通过substring方法进行字串提取。
3. String类支持+拼接。
4. 字符串本身是不可变的，对字符串的修改操作会产生一个新的字符串。
5. 不可变字符串可以多变量共享。
6. 代码点与代码单元（参加前面的关于字符类型的叙述）。

String类拥有很多方法，都很常用。这里就不列出来了，上网查即可。

### 可变字符串（StringBuilder、StringBuffer）

1. StringBuilder是线程不安全的、StringBuffer是线程安全的。
2. 通过toString方法生成字符串。
3. 简单使用如下：

``` java
    //java中StringBuilder的使用
    public static void stringBuilderUsage() {
        StringBuilder stringBuilder = new StringBuilder();
        System.out.println("start...");
        System.out.println(stringBuilder);

        stringBuilder.append("hello ");
        System.out.println(stringBuilder);

        stringBuilder.append(2020);
        System.out.println(stringBuilder);

        System.out.println(stringBuilder.toString());
    }
```

调用结果：

``` shell
start...

hello 
hello 2020
hello 2020
```

## 控制流程

if、while、for、switch等。

## 大数值类型

如果基本的数据类型不能满足需求，可以使用java.math包中的BigInteger和BigDecimal类。



## 数组

1. 一维数组两种创建方式、两种遍历方式；如下。

   ``` java
       //java中的一维数组
       public static void array1D() {
           //显示指定大小创建
           int[] arr1 = new int[3];
           arr1[0] = 0;
           arr1[1] = 1;
           arr1[2] = 2;
           //隐式创建
           int[] arr2 = new int[]{4, 5, 6};
   
           System.out.println("下标遍历：");
           for (int i = 0; i < arr1.length; i++) {
               System.out.print(arr1[i]);
           }
           System.out.println();
           System.out.println("for each遍历：");
           for (int x : arr2) {
               System.out.print(x);
           }
           System.out.println();
       }
   ```

   调用结果：

   ``` shell
   下标遍历：
   012
   for each遍历：
   456
   
   ```

   

2. 二维数组可以创建规则数组也可以创建不规则数组；如下。

   ``` java
       //java中的二维数组
       public static void array2D() {
           int[][] arr1 = new int[2][3];
           int[][] arr2 = new int[2][];
           arr2[0] = new int[5];
           arr2[1] = new int[3];
           System.out.println("arr1:");
           for (int i = 0; i < arr1.length; i++) {
               for (int j = 0; j < arr1[i].length; j++) {
                   System.out.print(arr1[i][j]);
                   System.out.print(",");
               }
               System.out.println();
           }
           System.out.println("arr2:");
           for (int i = 0; i < arr2.length; i++) {
               for (int j = 0; j < arr2[i].length; j++) {
                   System.out.print(arr2[i][j]);
                   System.out.print(",");
               }
               System.out.println();
           }
       }
   ```

   调用结果：

   ``` shell
   arr1:
   0,0,0,
   0,0,0,
   arr2:
   0,0,0,0,0,
   0,0,0,
   
   ```

   