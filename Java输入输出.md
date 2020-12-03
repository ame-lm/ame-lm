# Java输入输出

java的输入输出相关的类，几乎全部都集中在java.io包中

Java输入输出流总览：

![](https://gitee.com/ame-lm/PicBed/raw/master/img/20200802230827)

java中的输入输出类库采用了装饰者模式（关于装饰者模式不再赘述

）：

## InputStream

![image-20200803081519220](https://gitee.com/ame-lm/PicBed/raw/master/img/20200803081520.png)

Component：InputStream。

Concrete Component：ByteArray、File、Piped、StringBufferInput。

Decorator：FilterInputStream子类、Object、SequenceInputStream。

**InputStream**：

| 方法      | 解释                                       |
| --------- | ------------------------------------------ |
| available | 返回可提供字节数                           |
| close     | 关闭字节流                                 |
| read      | 从字节流中读出                             |
| reset     | 重置到上一个标记位                         |
| skip      | 跳过                                       |
| mark      | 标记（参数代表标记失效前最大能读的字节数） |

1. 部分InputStream（流来自于内存）的close方法没有意义，如ByteArrayInputStream
2. mark的参数指定的字节数之后，可能失效，也可能有效（ByteArrayInputStream）。最好按失效处理。

**具体部件：**

| 具体部件（省略InputStream） | 解释                                                   |
| --------------------------- | ------------------------------------------------------ |
| ByteArray                   | 以字节数组为源                                         |
| File                        | 以文件为源                                             |
| Piped                       | 以PipedOutputStream为源、常用于多线程、提供connect方法 |
| StringBuffer                | 弃用                                                   |

**装饰类：**

| 装饰类（省略InputStream） | 解释                                                         |
| ------------------------- | ------------------------------------------------------------ |
| Filter                    | 过滤。将InputStream作为源流，装饰后添加其他功能。            |
| Buffered                  | 使用缓冲区进行读取（没有改变InputStream语义，只是实现不同）。 |
| Data                      | 提供了读取各种基本类型（int、byte、甚至utf）的功能。         |
| LineNumber                | 弃用。                                                       |
| Pushback                  | 提供unread功能。                                             |
| Object                    | 提供readObject功能（反序列化）。                             |
| Sequence                  | 提供将两个流合并的功能。                                     |

## OutputStream

![image-20200803005303441](https://gitee.com/ame-lm/PicBed/raw/master/img/20200803005305.png)

Component：OutputStream

Concrete Component：ByteArray、File、PipedOutputStream。

Decorator：FilterInputStream子类、ObjectOutputStream。

**OutputStream：**

| 方法  | 解释                         |
| ----- | ---------------------------- |
| write | 向流写入                     |
| close | 关闭流                       |
| flush | 刷新流（将buffer写入目的地） |

**具体部件：**

| 具体部件（省略OutputStream） | 解释                     |
| ---------------------------- | ------------------------ |
| ByteArray                    | 以字节数组为汇集         |
| File                         | 以文件为汇集             |
| Piped                        | 以PipedInputString为汇集 |

**装饰类：**

| 装饰类（省略OutputStream） | 解释                                      |
| -------------------------- | ----------------------------------------- |
| Filter                     | 过滤，将输出流过滤添加新的特性            |
| Buffered                   | 不改变OutputStream的语义，只是实现不同    |
| Data                       | 提供了各种写基本类型的方法（甚至包括UTF） |
| PrintStream                | 提供各类print方法                         |
| Object                     | 提供了writeObject方法                     |

## Reader

![image-20200803005329960](https://gitee.com/ame-lm/PicBed/raw/master/img/20200803005331.png)

Component：Reader。

Concrete Component：CharArray、InputStreamReader、PipedReader、StringReader。

Decorator：BufferedReader子类、FilterReader子类。

**Reader：**

| 方法  | 解释           |
| ----- | -------------- |
| read  | 读             |
| reset | 重置标记       |
| ready | 测试是否准备好 |
| close | 关闭           |
| mark  | 标记           |
| skip  | 跳过           |

**具体部件：**

| 部件        | 解释              |
| ----------- | ----------------- |
| CharArray   | 以字符数组为源    |
| InputStream | 以InputStream为源 |
| File        | 以文件为源        |
| Piped       | 以PipedWriter为源 |
| String      | 以String为源      |

**装饰类：**

| 装饰       | 解释                               |
| ---------- | ---------------------------------- |
| Buffered   | 通过缓存行实现，提供行读取方法     |
| LineNumber | 继承BuffedReader，提供行号相关功能 |
| Pushback   | 提供pushback                       |

## Writer

![image-20200803005341345](https://gitee.com/ame-lm/PicBed/raw/master/img/20200803005342.png)

Component：Writer。

Concrete Component：CharArray、OutputStream、Piped、StringWriter。

Decorator：Buffered、Filter、PrintWriter。

**Writer：**

| 方法   | 解释                              |
| ------ | --------------------------------- |
| append | 追加、返回Writer、和write无甚区别 |
| close  | 关闭流                            |
| flush  | 刷新流（buffer刷新）              |
| write  | 写入流                            |

**具体部件：**

| 部件         | 解释                 |
| ------------ | -------------------- |
| CharArray    | 以字符数组为汇集     |
| OutputStream | 以OutputStream为汇集 |
| File         | 以文件为汇集         |
| Piped        | 以PipedReader为汇集  |
| String       | 以String为汇集       |

**装饰类：**

| 装饰        | 解释                                |
| ----------- | ----------------------------------- |
| Buffered    | 提供缓冲写，按行缓存提供newLine方法 |
| Filter      | 抽象类。过滤流                      |
| PrintWriter | 打印流。提供一系列print方法。       |

## Scanner

Scanner类是java.util包中封装的一个读取输入数据的工具类。

它是一个可以使用正则表达式来解析基本类型和字符串的简单文本扫描器。

提供以下方法：

| 方法                           | 解释                                     |
| ------------------------------ | ---------------------------------------- |
| close                          | 关闭                                     |
| delimiter                      | 返回分隔符                               |
| useDelimiter                   | 设置分隔符                               |
| locale                         | 返回语言环境                             |
| useLocale                      | 设置语言环境                             |
| radix                          | 返回进制                                 |
| useRadix                       | 设置进制                                 |
| match                          | 正则匹配                                 |
| toString                       | 返回String                               |
| next、next*、nextLine          | 读取下一个，紧接着的文本不匹配会抛出异常 |
| hasNext、hasNext*、hasNextLine | 检测下一个                               |
| remove                         | 不支持（来自于迭代器接口）               |
| findInLine、findWithinHorizon  | 本行匹配正则、horizon（右边界）之前匹配  |
| skip                           | 跳过                                     |
| reset                          | 重置                                     |

## Reference

https://www.cnblogs.com/wxgblogs/p/5649933.html

https://blog.csdn.net/u010145219/article/details/89792877

https://blog.csdn.net/twelvechenlin/article/details/72599130