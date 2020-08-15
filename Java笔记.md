# 异常

## 1 Java的异常

### 1.1 前言

计算机在运行期间会出现各种各样的错误，有一些错误是用户造成的，例如：

- 用户输入类型错误，或者合法性问题
- 程序读写文件时，发现用户已经将文件删除
- ......

还有一些错误是不知道什么时候会出现的，而且永远不可能避免，例如：

- 网络中断，与服务器连接终止
- 内存溢出
- 用户点“打印”， 但是根本没有打印机
- ......

所以，在编写程序的时候，要考虑到种种情况，尽可能多地处理各种各样的错误，这样才能保证程序的健壮性。



### 1.2 表现错误的方式

- 约定返回错误码
  - 在我感觉， 这在C语言编程中可能较为常见，可以自行设计一些处理逻辑的函数，告诉调用者不同情况会返回不同的数字或者字符来表示出现了不同的问题
  - 我在写前端向后端发送请求的时候，后端也会通过response结果的status来告诉我http请求是否成功。
- Java身为面向对象的语言，给我们提供了异常处理机制。异常作为一种类，包含了异常的信息，可以在任何地方抛出，然后调用方进行捕获和处理。就能够和方法调用分离。
  - **调用栈的概念应该都熟悉， Java在发生错误的时候会打印调用栈，会从捕获异常的函数，一直追溯到抛出异常的函数。我们可以选择一直沿着调用链抛到main函数，也可以选择在最开始抛出异常的时候就将异常处理掉，所以该在什么地方捕获异常并处理异常？又该在什么地方捕获异常并且把异常抛出给自己的调用方，我还是有些迷惑，所以先记录下来。说不定等我有更多经验之后，就能够回答我现在的问题。 **



### 1.3 Java 异常类

![异常继承树](C:\Users\aira\AppData\Roaming\Typora\typora-user-images\1590394105273.png)

Throwable是异常体系的根， 它下面还分为Error和Exception两部分：

- Error： 表示严重的错误，程序一般没有办法解决这样的问题，例如 
  - OutOfMemoryError： 内存耗尽
  - NoClassDefFoundError： 无法加载某个类
  - StackOverflowError：栈溢出
- Exception：则是一些由于外部设备、使用人员或者代码逻辑导致的错误，可以被捕获并且解决。
  - 然而Exception又分两类
    - RuntimeException及其子类，这个类型的异常一般都是由于代码逻辑问题导致的异常，通过修改代码都能解决
    - 除了上述类别以外的所有类别，这个类别的异常不是因为代码逻辑问题导致的，而是因为外部原因，比如文件丢失、网络错误等导致的。



**Java需要捕获的异常是Exception及其子类，但是不包括RuntimeException及其子类**

**Java不需要捕获的异常是Error及其子类、RuntimeException及其子类**



**前者为什么可以捕获呢？因为我们可以捕获之后给客户进行一些提示，比如：网络断开啦、没有检测到外部设备啦，这样的，然后代码别的逻辑可以继续运行下去**

**后者为什么不需要捕获呢？ Error类型的你捕获了有用吗？没有啊~你也处理不了。。。 RuntimeException 你捕获了有用吗？ 有用啊！ 你可以该代码让这个问题不发生呀**





## 2 处理异常

要说捕获异常，那还是最常用的try/catch语句块了。

### 2.1 try/catch

```java
try{
   //可能会抛出异常的代码                
}catch(Exception e){
	e.printStackTrace();
}
```



### 2.2 多重catch语句

```java
try{
	//某代码
}catch (Exception1 e){

}catch (Exception2 e){

}catch (Exception3 e){

}catch (Exception4 e){

}
```

按照规则，我们应该将子类放到好多层catch块中较为靠上的地方，也就是说 Exception1 < Exception2 < Exception3 < Exception4。

我们在这里将每一个类型的Exception比作一个碗，在继承树上越靠上的位置，碗越大，如果大碗放在上面的话，下面的类型就不会捕获到任何异常，因为会被上面的父类全部捕获了。



### 2.3 捕获多种异常

如果某些异常类型之间没有继承关系，并且处理逻辑相同，就可以将两种异常类型的catch块用 ” | “ 进行合并

```java
public static void main(String[] args) {
    try {
        process1();
        process2();
        process3();
    } catch (IOException | NumberFormatException e) { // IOException或NumberFormatException
        System.out.println("Bad input");
    } catch (Exception e) {
        System.out.println("Unknown error");
    }
}
```



### 2.4 finally语句

finally是跟在try/catch语句后的，它是不管是否抛出异常，都会执行的部分

注意：

- finally语句是可选的，可加可不加

- 如果没有发生异常的话，finally就会是最后执行的模块

- 如果发生异常的话，会在发生异常的地方终止，然后先去执行finally语句块，最后再回到之前终止的地方执行

- 可以只写 try/finally块，不写catch块，**前提是，你要在函数头部声明你可能会抛出的异常类型**

  ```java
  void process(String file) throws IOException { //就是这个 throws IOException
      try {
          ...
      } finally {
          System.out.println("END");
      }
  }
  ```

  - 借此可以说明一下我们捕获到异常之后该怎么办，我们可以有两种方案：
    - 捕获之后进行处理，解决异常
    - 不进行处理，直接抛给调用自己的人（就是上面说的那个 从函数头部声明抛出异常类型），这样就会把异常抛出给调用方。
    - 调用方在捕获异常之后同样有两种选项，要么处理，要么继续往上抛。 所以什么时候该处理，什么时候该抛，这可能是一个经验问题，也是我上面提出的疑惑。。。







**finally还有别的用处： 异常屏蔽**

我们知道，如果catch语句捕获异常之后，会先去执行finally块，如果我们在finally块抛出了异常的话，catch块的异常就不会被抛出了，这就是异常屏蔽

不过有个骚操作， 但是用的很少，我也就不说了， 都在廖雪峰老师的博客中，到时候自己去看吧， 具体地方为**异常处理/抛出异常章节**





### 2.5 异常类型转换

如果我们接收到异常之后，又抛出了另外一种类型的异常，那么异常就发生了类型转换，丢失原有的异常信息。

所以在构造异常的时候，把原始的Exception实例传进去，新的Exception就可以保存原有的Exception信息。

```Java
public class Main {
    public static void main(String[] args) {
        try {
            process1();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    static void process1() {
        try {
            process2();
        } catch (NullPointerException e) {
            throw new IllegalArgumentException(e);
        }
    }

    static void process2() {
        throw new NullPointerException();
    }
}

```



## 3 日志系统







# 注解与反射

## 注解

### 什么是注解？

Annotation是从JDK5.0开始引入的新技术



作用：

- 不是程序本身，可以对程序作出解释（这一点和注释comment没什么区别）
- **可以被其他程序（如编译器）读取**



Annotation格式：

- 注解是以“@注释名”在代码重尊在的，还可以添加一些参数值，例如：@SuppressWarnings（value="unchecked"）



Annotation在哪里使用？

- 可以附加在package，class，method，filed等上面，相当给他们添加了额外的辅助信息，我们可以通过反射机制编程实现对这些元数据的访问



### 内置注解

- @Override：定义在java.lang.Override中， 此注解只适用于修辞方法，表示一个方法打算重写超类中的另一个方法声明
- @Deprecated：定义在java.lang.Deprecated中，此注解可以用于修辞方法、属性、类，表示不鼓励程序员使用这样的元素，通常是因为他很危险或者存在更好的选择。
- @SuppressWarnings：定义在java.lang.SuppressWarnings中，用来抑制编译时的警告信息，
  - 与前两个不同，需要添加一个参数才能够正确使用，这些参数都是已经定义好了的，我们选择性的使用就好了
  - @SupressWarnings（“all”）
  - @SupressWarnings("unchecked")
  - @SupressWarnings(value = { "unchecked", " deprecation"})



### 元注解

- 元注解的作用是负责注解其他注解，Java定义了4个标准的meta-annotation类型，它们被用来提供对其他annotation类型作说明
- 这些类型和他们所支持的类在java.lang.annotation包中可以找到。（@Target， @Retention， @Documented， @Inherited）
  - **@Target**：用于描述注解的使用范围（即：被描述的注解可以用在什么地方）
    - 类或接口：`ElementType.TYPE`；
    - 字段：`ElementType.FIELD`；
    - 方法：`ElementType.METHOD`；
    - 构造方法：`ElementType.CONSTRUCTOR`；
    - 方法参数：`ElementType.PARAMETER`。
  - **@Retention**：表示需要在什么级别保存该注释信息，用于描述注解的声明周期
    - （SOURCE < CLASS < **RUNTIME**）
    - 仅编译期：`RetentionPolicy.SOURCE`；
    - 仅class文件：`RetentionPolicy.CLASS`；
    - 运行期：`RetentionPolicy.RUNTIME`。
  - @Documented：说明该注解将被包含在javadoc中
  - @Inherited：说明子类可以继承父类中的注解，仅针对`@Target(ElementType.Type)`类型的注解有效，且仅针对class的继承，对interface的继承无效。





### 自定义注解

- 使用@interface自定义注解时，自动继承了java.lang.annotation.Annotation接口
- 分析
  - @interface用来声明一个注解，格式： public @interface 注解名 {定义内容}
  - 其中的每一个方法实际上是声明了一个配置参数
  - 方法的名称就是参数的名称
  - 返回值类型就是参数的类型（返回值只能是基本类：**Class**（不是class）, String, enum）
  - 可以通过default来声明参数的默认值
  - 如果只有一个参数成员，一般参数名为value
  - 注解元素必须要有值，我们定义注解元素时，经常使用空字符串，0作为默认值。

```java
import java.lang.annotation.*;

@MyAnnotation
public class Test01 {

    /*
    * 如果只有一个参数的话，我们如果将参数名设置为value，那么我们在复制的时候
    * 可以省略key，直接写value。否则不可以省略key。
    * */
    @MyAnnotation3("wo")
    public void test3(){}

    /*
    * 注解如果有默认值的话， 那我们可以不进行赋值，也可以进行显式赋值
    * 如果没有默认值的话，我们就必须要进行赋值
    * */
    @MyAnnotation2(woc = 231.2f)//多参数注解
    public void test2(){}

    @MyAnnotation
    public void test(){}
}


@Target(value =  {ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@interface MyAnnotation3{
    String value();
}


@Target(value =  {ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@interface MyAnnotation2{
    // 注释的参数 ： 参数类型 + 参数名 ();
    String name() default "";
    int age() default 0;
    int id() default -1;  //如果默认值为 -1， 代表不存在
    String[] schools() default {"清华大学"};
    float woc();
}


@Target(value = {ElementType.METHOD, ElementType.TYPE})
@Retention(value = RetentionPolicy.RUNTIME)
@Documented
@Inherited
@interface MyAnnotation{

}
```



## 反射机制

### 反射概述

#### 静态语言 VS 动态语言

- 动态语言
  - 是一类在运行时，可以改变其结构的语言：例如新的函数、对象、甚至代码可以被引进，已有的函数可以被删除或是其他结构上的变化。通俗点说就是在运行时代码可以根据某些条件改变自身结构。
  - 主要动态语言：Object-C， C#， JavaScript， PHP， Python等
- 静态语言
  - 与动态语言对应，运行时结构不可变的语言就是静态语言，如C，C++, Java;
  - Java不是动态语言，但Java可以称为“准动态语言”。即Java有一定的动态性，我们可以利用反射机制获得类似动态语言的特性。Java的动态性让Java编程的时候更加灵活。 Java获取了一定的动态性，作为代价就失去了一部分的安全性。



#### Java Reflection

- Reflection 是Java被视为动态语言的关键，反射机制允许程序在执行期间借助Reflection API取得任何类的内部信息，并且能直接操作任意对象的内部属性及方法。

  ```java
  Class c = Class.forName("java.lang.String");
  ```

- 记载完类之后，在堆内存的方法区中就产生了一个Class类型的对象（一个类只有一个Class对象），这个对象就包含了完整的类的结构信息。我们可以通过这个对象看到类的结构。这个对象就像一面镜子，透过这个镜子看到类的结构。所以，我们形象的称之为：反射。

![1590416574695](C:\Users\aira\AppData\Roaming\Typora\typora-user-images\1590416574695.png)



#### Java反射机制研究及应用

- Java反射机制提供的功能
  - 在运行时判断任意一个对象所属的类
  - 在运行时构造任意一个类的对象
  - 在运行时判断任意一个类所具有的成员变量和方法
  - 在运行时获取泛型信息
  - 在运行时调用任意一个对象的成员变量和方法
  - 生成动态代理
  - ......
- 反射优缺点
  - 优点：可以实现动态创建对象和编译，体现出很大的灵活性
  - 缺点：对性能有影响。使用反射基本上是一种解释性操作，我们可以告诉JVM，我们希望做什么并且它会满足我么的呢需求，这类操作总是慢于直接执行相同的操作。







### 取得反射对象

#### 反射 API

- **Java.lang.Class**：代表一个类
- java.lang.reflect.Method：代表类的方法
- java.lang.reflect.Field：代表类的成员变量
- java.lang.reflect.Constructor：代表类的构造器



#### Class类

在Object类中定义了以下方法，此方法被所有的子类继承

```java
public final Class getClass();
```

该方法返回值的类型是Class类，此类是Java反射的源头，实际上所谓的反射从程序的运行结果来看也很好理解，即：可以通过对象反射求出类的名称



反射对象可以得到的信息

- 某个类的属性、方法和构造器
- 某个类到底实现了哪个接口

对于每个类而言，JRE都为其保留了一个不变的Class对象。一个Class对象包含了特定某个结构（class/interface/enum/annotation/primitive type/void/[]）的有关信息：

- Class对象本身也是一个类
- Class对象只能由系统建立对象
- 一个加载的类在JVM中只会有一个Class实例
- 一个Class对象对应的是一个加载到JVM中的一个.class文件
- 每个类的实例都会记得自己是由哪个Class实例所生成
- 通过Class可以完整地得到一个类中的所有被加载的结构
- Class类是Reflection的根源，针对任何你想动态加载、运行的类，唯有先获得的相应的Class对象



#### 得到Class类的实例

- 若已知具体的类，通过类的class属性获取，该方法最为安全可靠，程序性能最高：

  ```java
  Class class = Person.class;
  ```

- 已知某个各类的实例，调用该实例的getClass()方法获取Class对象：

  ```java
  Class class = person.getClass();
  ```

- 已知一个类的全类名，且该类在类路径下，可通过Class类的静态方法forName（）获取，可能抛出ClassNotFoundException

  ```java
  Class class = Class.forName("demo01.Student");
  ```

- 内置基本数据类型可以直接用类名.Type

- 还可以利用ClassLoader我们之后讲解。



#### 哪些类可以有Class对象

- 外部类，成员（成员内部类，静态内部类），局部内部类，匿名内部类。
- interface：接口
- []：数组  **只要元素类型和维度一样， 就是同一个Class**
- enum：枚举
- annotation：注解@interface
- primitive type：基本数据类型
- void



### 类的加载

当程序主动使用某个类的时候，如果该类还没有被加载到内存中，则系统会通过如下三个步骤对该类进行初始化。

- 加载：将class字节码内容加载到内存中，并将这些静态数据转换成方法区的运行时数据结构，然后生成一个代表这个类的java.lang.Class对象
- 链接：将Java类的二进制代码合并到JVM的运行状态之中的过程。
  - 验证：确保加载的类信息符合JVM规范，没有安全方面的问题
  - 准备：省市委类变量（static）分配内存并设置类变量默认值的阶段，这些内存都将在方法区中进行分配。
  - 解析：虚拟机常量池内的符号引用（常量名）替换为直接引用（地址）的过程。
- 初始化：
  - 执行类构造器\<clinit\>()方法的过程。类构造器\<clinit\>()方法是有编译期自动手机类中所有类变量的赋值动作和静态代码块中的语句合并产生的。（类构造器是构造信息的，不是构造该类对象的构造器）。
  - 当初始化一个类的时候，如果发现其父类还没有进行初始化，则需要先触发父类的初始化。
  - 虚拟机会保证一个类的\<clinit\>()方法在多线程环境中被正确的加锁和同步



### 类什么时候会发生初始化？

```java
public class Test06 {
    static {
        System.out.println("Main类被加载");
    }

    public static void main(String[] args) throws ClassNotFoundException {
		//下面每个部分都会添加一段代码在这个部分进行执行
    }
}

class Father{
    static{
        System.out.println("Father类被加载");
    }

    static int b = 2;
}

class Son extends Father{
    static {
        System.out.println("Son类被加载");
    }

    static int m = 100;

    static final int M = 1;
}
```

- 类的主动引用（一定会发生类的初始化）

  - 当虚拟机启动，先初始化main方法所在类

    ```java
    主类中的静态代码块就会被执行
    ```

  - new一个类的对象

    ```java
    Son son = new Son();
    /*
    Main类被加载
    Father类被加载
    Son类被加载
    */
    ```

  - 调用类的静态成员（除了final常量）和静态方法

    ```java
    System.out.println(Son.m);
    /*
    Main类被加载
    Father类被加载
    Son类被加载
    100
    */
    ```

  - 使用java.lang.reflect包的方法对类进行反射调用

    ```java
    Class.forName("javalearning.reflection.Son");
    /*
    Main类被加载
    Father类被加载
    Son类被加载
    */
    ```

  - 当初始化一个类，如果其父类没有被初始化，则会先初始化它的父类。

    ```java
    //上述Demo都是在加载子类之前先加载了父类
    ```

- 类的被动引用（不会发生类的初始化）

  - 当访问一个静态域时，只有真正声明这个域的类才会被初始化。如：当通过子类引父类的静态变量，不会导致子类初始化。

    ```java
    System.out.println(Son.b);
    /*
    Main类被加载
    Father类被加载
    2
    
    因为是父类声明了静态变量b，子类只是从父类身上继承了下来，子类并不是声明该字段的类，所以会去找子类的爸爸，和子类没太大瓜西（子类算是个中介吧）
    */
    ```

    

  - 通过数组定义引用，不会触发此类的初始化。

    ```java
    Son[] son = new Son[10];
    /*
    Main类被加载
    
    只是开辟了一个这个类型的空间，并没有用到这个类做任何事情
    */
    ```

  - 引用常量不会触发此类的初始化（常量在链接阶段就存入调用类的常量池中了）

    ```java
    System.out.println(Son.M);
    /*
    Main类被加载
    1
    */
    ```

    





### 类加载器的作用

- **类加载的作用**：将class文件字节码内容加载到内存中，并将在这些静态数据转换成方法去的运行时数据结构，然后在堆中生成一个代表这个类的java.lang.Class对象，作为方法区中类数据的访问入口。

- 类缓存：标准的JavaSE类加载器可以按要求查找类，但一旦某个类被加载到类加载中，它将维持加载（缓存）一段时间。不过JVM GC机制可以回收这些Class对象

  ![1590482901458](C:\Users\aira\AppData\Roaming\Typora\typora-user-images\1590482901458.png)

- 类加载作用是用来把类（class）装载进内存的。JVM规范定义了如下类型的类加载器

  - 引导类加载器（Bootstap Classloader）：用C++编写的，是JVM自带的类加载器，**负责Java平台核心库（rt.jar）**，用来装载核心类库。该加载去无法直接获取

  - 拓展类加载器（Extension Classloader）：负责jre/lib/ext目录下的jar包或 -D java.ext.dirs指定目录下的jar包 装入工作库

  - 系统类加载器（System Classloader / Application Classloader）：负责 java -classpath 或 -D java.class.path所指的目录下的类与jar包装入工作，是最常用的加载器。

    ![1590483257158](C:\Users\aira\AppData\Roaming\Typora\typora-user-images\1590483257158.png)

  - 双亲委派机制：如果java需要加载一个类，例如java.lang.String ，然鹅你自己也想写一个同名的。他会从低级类加载器到高级类加载器逐渐寻找，如果在高层找到了那个类，那么你手写的同名包就不会被调用。（你不配重写我的类 ----Sun公司）

    ```java
    public class Test07 {
        public static void main(String[] args) throws ClassNotFoundException {
            // 获取系统类加载器
            ClassLoader systemClassLoader = ClassLoader.getSystemClassLoader();
            System.out.println(systemClassLoader);
    
            // 获取系统类加载器的父类加载器 ---》 拓展类加载器
            ClassLoader parent = systemClassLoader.getParent();
            System.out.println(parent);
    
            // 获取拓展类加载器的父类加载器 ---》 根加载器（C / C++）
            ClassLoader parent1 = parent.getParent();
            System.out.println(parent1);
    
            // 查看当前类是哪个加载器加载的
            ClassLoader classLoader = Class.forName("javalearning.reflection.Test07").getClassLoader();
            System.out.println(classLoader);
    
            // 测试JDK内置的类是谁加载的
            classLoader = Class.forName("java.lang.Object").getClassLoader();
            System.out.println(classLoader);
    
            //如何获取系统类加载器可以加载的路径
            System.out.println(System.getProperty("java.class.path"));
    
            /*
            * E:\ProjectFiles\IJproject\JavaLearning\out\production\JavaLearning;
            * E:\ProjectFiles\IJproject\JavaLearning\lib\javax.transaction.jar;
            * E:\ProjectFiles\IJproject\JavaLearning\lib\javax.persistence.jar;
            * E:\ProjectFiles\IJproject\JavaLearning\lib\javax.resource.jar;
            * E:\ProjectFiles\IJproject\JavaLearning\lib\javax.ejb.jar;
            * E:\ProjectFiles\IJproject\JavaLearning\lib\javax.servlet.jsp.jar;
            * E:\ProjectFiles\IJproject\JavaLearning\lib\javax.servlet.jar;
            * E:\ProjectFiles\IJproject\JavaLearning\lib\javax.jms.jar;
            * E:\ProjectFiles\IJproject\JavaLearning\lib\javax.annotation.jar;
            * E:\ProjectFiles\IJproject\JavaLearning\lib\javax.servlet.jsp.jstl.jar;
            *
             * */
        }
    }
    
    /*
    jdk.internal.loader.ClassLoaders$AppClassLoader@3fee733d
    jdk.internal.loader.ClassLoaders$PlatformClassLoader@506e1b77
    null
    jdk.internal.loader.ClassLoaders$AppClassLoader@3fee733d
    null //根加载器我们JVM无法读取， 所以是null
    */
    ```

    

### 创建运行时类的完整结构

通过反射获取运行时类的完整结构：

```java
Field、Method、Constructor、Superclass、Interface、Annotation
```

- 实现的全部结构

- 所继承的父类

- 全部的构造器

- 全部的方法

- 全部的Field

- 注解

- ......

  ```java
  import java.lang.reflect.Constructor;
  import java.lang.reflect.Field;
  import java.lang.reflect.Method;
  
  // 获得类的信息
  public class Test08 {
      public static void main(String[] args) throws ClassNotFoundException, NoSuchFieldException, NoSuchMethodException {
          Class c1 = Class.forName("javalearning.reflection.User");
  
          //获得类的名字
          System.out.println(c1.getName());      // 包名 + 类名
          System.out.println(c1.getSimpleName());// 类名
  
          //获得类的属性
          System.out.println("===================================");
          Field[] fields = c1.getFields(); //只能获取自身以及父类的公共属性
          for (Field field : fields){
              System.out.println(field);
          }
  
          System.out.println("===================================");
          fields = c1.getDeclaredFields(); //能获取自身所有属性
          for (Field field : fields){
              System.out.println(field);
          }
  
  
          //获取指定的属性的值
          System.out.println("======================================");
          Field name = c1.getDeclaredField("name");
          System.out.println(name);
  
          System.out.println("======================================");
          Method[] methods = c1.getMethods(); //获得本类及其父类的所有public方法
          for (Method method : methods){
              System.out.println(method);
          }
  
          methods = c1.getDeclaredMethods(); //获得本类的所有方法
          for (Method method : methods){
              System.out.println(method);
          }
  
          System.out.println("======================================");
          // 获取指定的方法
          //重载 通过参数类型判断是哪个方法
          Method getName = c1.getMethod("getName", null);
          Method setName = c1.getMethod("setName", String.class);
          System.out.println(getName);
          System.out.println(setName);
  
          System.out.println("======================================");
          //获得构造器
          Constructor[] constructors = c1.getConstructors();
          for (Constructor constructor : constructors){
              System.out.println(constructor);
          }
  
          constructors = c1.getDeclaredConstructors();
          for (Constructor constructor : constructors){
              System.out.println("#"+ constructor);
          }
  
          System.out.println("指定构造器" + c1.getDeclaredConstructor(String.class, int.class, int.class));
      }
  }
  ```



### 小结

- 在实际操作中，取得类的信息的操作代码，并不会经常开发
- 一定要熟悉java.lang.reflect包的作用，反射机制。
- 如何取得属性、方法、构造器的名称、修饰符等



#### 有了Class对象之后，能做什么？

- 创建类的对象：调用Class对象的newInstance（）方法
  - 类必须有一个无参数的构造器
  - 类的构造器的访问权限需要足够
- **思考？**：难道没有午餐的构造器就不能创建对象了吗？只要在操作的时候明确的调用类中的构造器，并将参数传递进去之后，才可以实例化操作
- 步骤如下：
  - 通过Class的 getDeclaredConstructor（Class ... parameterTypes）取得本类的指定形参类型的构造器
  - 向构造器的形参中传递一个对象数组进去，里面包含了构造器中所需要的各个参数
  - 通过Constructor实例化对象



#### 调用指定的方法

通过反射，调用类中的方法，通过Method类完成。

- 通过Class类的getMethod（String name， Class ... parameterTypes）方法取得一个Method对象，并设置此方法操作时所需要的参数类型。

- 之后使用Object invoke（Object obj， Object[] args）进行调用，并向方法中传递要设置的obj对象的参数信息

  ![1590489423316](C:\Users\aira\AppData\Roaming\Typora\typora-user-images\1590489423316.png)

  ```java
  Object invoke(Object obj, Obj ... args)
  ```

  - Object对方法的返回值，若原方法无返回值，此时返回null
  - 若原方法为静态方法，此时形参Object obj可为null
  - 若原方法形参列表为空，则Object[] args 为null
  - 若原方法声明为private，则需要再调用此invoke（）方法钱，显式调用方法对象的setAccessible（true）方法，将可访问private的方法。
  - setAccessible
    - Method和Field、Constructor对象都有setAccessible（）方法
    - setAccesible作用是启动和禁用访问安全检查的开关。
    - 参数值为true， 指示反射的对象在使用时应该取消Java语言访问检查。
      - 提高反射的效率。如果代码中必须使用反射，而该句代码需要频繁的被调用，那么请设置为true
      - 使得原本无法访问的私有成员也可以访问
    - 参数值为false，指示反谁的对象应该是是java语言的访问检查

```java
import java.lang.reflect.Constructor;
import java.lang.reflect.Field;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;

public class Test09 {
    public static void main(String[] args) throws Exception {
        Class c1 = Class.forName("javalearning.reflection.User");


//        User user = (User) c1.newInstance(); //本质是调用了类的无参构造器
//        System.out.println(user);

        //通过构造器创建对象
        Constructor constructor = c1.getDeclaredConstructor(String.class, int.class, int.class);
        System.out.println(constructor);
        User user1 = (User) constructor.newInstance("岛市老八", 1, 18);
        System.out.println(user1);


        //通过反射调用普通方法
        //获取一个对象
        User user2 = (User)c1.newInstance();
        //通过反射获取一个方法
        Method setName = c1.getDeclaredMethod("setName", String.class);

        //invoke： 激活
        // (对象， "方法的值")
        setName.invoke(user2, "奥里给");
        System.out.println(user2.getName());

        //通过变成操作属性
        User user3 = (User) c1.newInstance();
        Field name = c1.getDeclaredField("name");


        //取消私有属性的安全检测
        name.setAccessible(true);
        name.set(user3, "wocao");
        System.out.println(user3.getName());
    }
}
```

#### 性能分析

```java
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;

//分析性能问题
public class Test10 {
    public static void main(String[] args) throws NoSuchMethodException, IllegalAccessException, InvocationTargetException {
        test01();
        test02();
        test03();
    }
    //普通方式调用

    public static void test01(){
        User user = new User();
        long startTime = System.currentTimeMillis();

        for (int i = 0; i < 100000000; i++){
            user.getName();
        }

        long endTime = System.currentTimeMillis();

        System.out.println("普通方法: " + (endTime - startTime) + " ms");
    }

    //反射方法调用
    public static void test02() throws NoSuchMethodException, InvocationTargetException, IllegalAccessException {
        User user = new User();
        Class c1 = user.getClass();
        Method getName = c1.getDeclaredMethod("getName", null);
        long startTime = System.currentTimeMillis();

        for (int i = 0; i < 100000000; i++){
            getName.invoke(user, null);
        }

        long endTime = System.currentTimeMillis();

        System.out.println("反射方法: " + (endTime - startTime) + " ms");
    }

    //反射方式调用 关闭检测
    public static void test03() throws NoSuchMethodException, InvocationTargetException, IllegalAccessException {
        User user = new User();
        Class c1 = user.getClass();
        Method getName = c1.getDeclaredMethod("getName", null);
        getName.setAccessible(true);
        long startTime = System.currentTimeMillis();

        for (int i = 0; i < 100000000; i++){
            getName.invoke(user, null);
        }

        long endTime = System.currentTimeMillis();

        System.out.println("关闭检测方法: " + (endTime - startTime) + " ms");
    }
}

/*
普通方法: 4 ms
反射方法: 330 ms
关闭检测方法: 159 ms
*/
```

反射确实会降低程序执行的性能，关闭安全检查可以较为显著地提升性能。







### 反射操作泛型

- Java采用泛型擦除机制来引入泛型，Java中的泛型仅仅是给编译器javac使用的，确保数据的安全性和免去强制类型转换问题。但是，一旦编译完成，所有和泛性有关的类型全部擦除。
- 为了通过反射操作这些类型，Java新增了ParameterizedType，GenericArrayType，TypeVariable和WildcardType 几种类型来代表不能被归一到Class类中的类型但是又和原始类型齐名的类型
  - ParameterizedType：表示一种参数化类型，比如Collection\<String\>
  - GenericArrayType：表示一种元素类型是参数化类型或者类型变量的数组类型
  - TypeVariable：是各种类型变量的公共父接口
  - WildcardType：代表一种通配符类型表达式

```java
import java.lang.reflect.Method;
import java.lang.reflect.ParameterizedType;
import java.lang.reflect.Type;
import java.util.List;
import java.util.Map;

//通过反射获取泛型
public class Test11 {

    public static void main(String[] args) throws NoSuchMethodException {
        Method method = Test11.class.getMethod("test01", Map.class, List.class);
        //返回函数的参数列表
        Type[] genericExceptionTypes = method.getGenericParameterTypes(); 
        //遍历每一项参数
        for (Type genericExceptionType : genericExceptionTypes){          
            System.out.println("#"+ genericExceptionType);
            //判断每一项的泛型的参数类型是否为结构化参数类型
            if (genericExceptionType instanceof ParameterizedType){       
                //强制转换为结构化参数类型，然后用函数获得其真实类型数组
                Type[] actualTypeArguments = ((ParameterizedType) genericExceptionType).getActualTypeArguments();
                for (Type actualTypeArgument : actualTypeArguments){
                    System.out.println(actualTypeArgument);
                }
            }
        }

        method = Test11.class.getDeclaredMethod("test02");
        //这个方法获取的是返回值类型，别的和上面的逻辑没啥区别
        Type genericReturnType = method.getGenericReturnType();  
        if (genericReturnType instanceof ParameterizedType){
            Type[] actualTypeArguments = ((ParameterizedType) genericReturnType).getActualTypeArguments();
            for (Type actualTypeArgument : actualTypeArguments){
                System.out.println(actualTypeArgument);
            }
        }


    }

    public void test01(Map<String, User> map, List<User> list){
        System.out.println("test01");
    }

    public Map<String,User> test02(){
        System.out.println("test02");
        return null;
    }
}
```



### 反射操作注解

- getAnnotations
- getAnnotation

```java
import java.lang.annotation.*;
import java.lang.reflect.Field;

//联系反射操作注解
public class test12 {
    public static void main(String[] args) throws ClassNotFoundException, NoSuchFieldException {
        Class c1 = Class.forName("javalearning.reflection.Student2");
        
        //通过反射获取注解
        Annotation[] annotations = c1.getAnnotations();
        for (Annotation annotation : annotations){
            System.out.println(annotation);
        }
        /*
        @javalearning.reflection.TableName(value="db_student")
        */

        //获取注解value的值
        //获取指定注解， 传入注解的Class 返回类型是Annotation类型的，然后强转为目标类型
        TableName tableName = (TableName) c1.getAnnotation(TableName.class);
        String value = tableName.value(); //通过.value()就能获取了
        System.out.println(value);
		/*
		db_student
		*/

        // 获取类指定的注解
        Field field = c1.getDeclaredField("name");
        FieldName annotation = field.getAnnotation(FieldName.class);
        System.out.println(annotation.columnName());
        System.out.println(annotation.type());
        System.out.println(annotation.length());
        
        /*
        db_name
		varchar
		3
		*/
    }

}

@TableName("db_student")
class Student2{
    @FieldName(columnName = "db_id", type="int", length=10)
    private int id;
    @FieldName(columnName = "db_age", type="int", length=10)
    private int age;
    @FieldName(columnName = "db_name", type="varchar", length=3)
    private String name;

    public Student2() {
    }

    public Student2(int id, int age, String name) {
        this.id = id;
        this.age = age;
        this.name = name;
    }

    public void setId(int id) {
        this.id = id;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getId() {
        return id;
    }

    public int getAge() {
        return age;
    }

    public String getName() {
        return name;
    }

    @Override
    public String toString() {
        return "Student2{" +
                "id=" + id +
                ", age=" + age +
                ", name='" + name + '\'' +
                '}';
    }

}

//类名的注解
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@interface TableName{
    String value();
}

//表属性的注解
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
@interface FieldName{
    String columnName();
    String type();
    int length();
}
```



## 小结

这部分，说实话学完下来，只是知道了怎么去操作，但是为什么去操作，什么地方应该去操作，还是不知道。以后会在框架底层反复巩固，其中也涉及到了JVM的知识。在这里就当作是提前学习一下，打一下基础，希望以后深入学习框架的时候能够得心应手。

感谢B站狂神说java，这一章两个小时的视频我看了很久才吸收完。继续学习吧！加油qwq









# 线程

## 线程简介

### 多任务

现实中很多同时做很多件事情的例子，看起来是多个任务都在做， 其实本质我们的大脑在同一时间依旧只做了一件事情，只是用非常快的速度进行交替。

### 多线程

![1590549703658](C:\Users\aira\AppData\Roaming\Typora\typora-user-images\1590549703658.png)

多线程拥有更高的执行效率

### 程序-进程-线程

每个程序就是一个进程，每个进程可以有很多线程，不同的线程负责不同的方面

- 程序是指令和数据的有序结合，其本身没有任何运行的含义，是一个静态的概念
- 进程是执行程序的一次执行过程，这是一个动态的概念。是系统资源分配的单位
- 通常在一个进程中可以包含若干个线程，一个进程中至少有一个线程（Main主线程），不然没有存在的意义。线程是CPU调度和执行的单位。
-  **注意**：在一个CPU的情况下， 同一时间点，CPU只能执行一个代码，因为切换的非常快，所以才有同时执行的错觉。只有多核CPU才能同时实现真正的多线程。



### 本章核心概念

- 线程就是独立的执行路径
- 在程序运行时，即使自己没有创建线程，后台也会有多个进程。如主线程，GC线程；
- main()称之为主线程，是系统的入口，用于执行整个程序
- 在一个进程中，如果开辟了多个线程，线程的运行由调度器安排调度，调度器是与操作系统紧密相关的，先后顺序是不能人为干预的。
- 对一份资源操作时，会存在资源抢夺的问题，需要加入并发控制
- 线程会带来额外的开销，如CPU调度时间，并发控制开销。
- 每个线程在自己的工作内存交互，内存控制不当会造成数据不一致





## 线程创建（重点）

### 三种创建方式

- Thread class ： 继承Thread类
  - 自定义线程类继承Thread类
  - 重写run（）方法，编写线程执行体
  - 创建线程对象，调用start（）方法启动线程
  - `子类对象.start();`
  - 线程开启后不一定立即执行，要看CPU调度
  - ==不推荐使用：避免OOP单继承局限性==
- Runnable接口：实现Runnable接口
  - 定义MyRunnable类实现Runnable接口
  - 实现run()方法，编写线程执行体
  - 创建线程对象，调用start()方法启动线程
  - `其实没啥区别，第二种方法在启动现成的时候需要通过把线程任务传入到线程类的构造函数中： new Thread(传进来).start`
  - ==避免单继承局限性，灵活方便，方便同一个对象被多个线程使用==
- Callable接口：实现Callable接口
  - 实现Callable接口，需要返回值类型
  - 重写call方法，需要抛出异常
  - 创建目标对象
  - 创建执行服务：`ExecutorService ser = Executors.newFixedThreadPool(1);`
  - 提交执行：`Future<Boolean> result1 = ser.submit(t1);`
  - 获取结果：`boolean r1 = result1.get();`
  - 关闭服务：`ser.shutdownNow();`
  - 好处
    - 可以定义返回值
    - 可以抛出异常



## 静态代理

静态代理对象 和 甚至对象都要实现同一个接口

代理对象要代理真实角色



好处：

- 代理对象可以做很多真实对象做不了的事情
- 真实对象专注与自己的事情

```java
public class test1 {
    public static void main(String[] args) {
        WeddingCompany weddingCompany = new WeddingCompany(new You());
        weddingCompany.HappyMarry();
    }

}

interface Marry{
    void HappyMarry();
}

// 真实角色
class You implements Marry{

    @Override
    public void HappyMarry() {
        System.out.println("秦老师要结婚了，超开心");
    }
}

// 代理角色
class WeddingCompany implements Marry{
    private Marry target;

    public WeddingCompany(Marry target){
        this.target = target;
    }

    @Override
    public void HappyMarry() {
        System.out.println("布置现场");
        this.target.HappyMarry();
        System.out.println("收尾款");
    }
}

```

**多线程对比静态代理**

```java
new Thread( () -> System.out.println("我爱你")).start();
new WeddingCompany(new You()).HappyMarry();
```

`Thread对象也实现了Runnable接口，所以第二种创建线程的方法，也可以换成第一种方法来做，但是第二种方法就相当于是让Thread对象代理了线程任务。线程任务专注于自己的任务，Thread类用来做线程调度。。`





## Lamda表达式

- 理解Functional Interface（函数式接口）是学习Java8 lambda表达式的关键所在。

- 函数式接口的定义：

  - 任何接口，如果只包含唯一一个抽象方法，那么它就是一个函数式接口。

    ```java
    public interface Runnable{
    	public abstract void run();                    
    }
    ```

  - 对于函数式接口，我们可以通过lambda表达式来创建该接口的对象



- lambda表达式只有在一行的情况下才能简化为一行，如果有多行，那么就用代码块包裹
- 前提是接口为函数式接口
- 如果 多参数 要省略类型，就全部省略。



## 线程状态

![1590575383252](C:\Users\aira\AppData\Roaming\Typora\typora-user-images\1590575383252.png)





### 线程停止

- 不推荐使用JDK提供的 `stop()、destroy()`方法；
- 推荐让线程自己停下来
- 建议使用一个标志位进行终止变量，当`flag = false`，则程序终止。

```java
// 1.建议线程正常停止 ----> 利用次数，不建议死循环
// 2.建议使用标志位 -----》 设置一个标志位
// 3.不要使用stop或destroy等过时或者JDK不建议使用的方法

public class teststop implements Runnable{
    private boolean flag = true;

    @Override
    public void run() {
        int i = 0;
        while(flag){
            System.out.println("run ---- Thread" + i ++);
        }
    }

    public void stop(){
        flag = false;
    }

    public static void main(String[] args) {
        teststop teststop = new teststop();
        new Thread(teststop).start();

        for (int i = 0; i < 1000; i++) {
            System.out.println("main:" + i);
            if (i == 900){
                teststop.stop();
                System.out.println("终止线程");
            }
        }
    }
}
```



### 线程休眠 sleep

- sleep指定当前线程阻塞的毫秒数
- sleep存在异常InterruptedException
- sleep时间达到后线程进入就绪状态
- sleep可以模拟网络延时，倒计时等
- 每个对象都有一个锁，sleep不会释放锁



### 线程礼让 yield

- 礼让线程，让当前正在执行的线程暂停，但不阻塞
- 让线程从运行状态转为就绪状态
- 让CPU重新调度，礼让不一定成功，看CPU心情

```java
public class TestYield {
    public static void main(String[] args) {
        MyYield myYield = new MyYield();

        new Thread(myYield, "a").start();
        new Thread(myYield, "b").start();
    }
}

class MyYield implements Runnable{

    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName() + "线程开始执行");
        Thread.yield();
        System.out.println(Thread.currentThread().getName() + "线程停止执行");
    }
}
```

### 线程强制执行 join

- join合并线程，待此线程执行完成后，再执行其他线程，其他线程阻塞
- 可以想象成阻塞

```java
public class TestJoin implements Runnable {
    @Override
    public void run() {
        for (int i = 0; i < 1000; i++) {
            System.out.println("线程VIp来了" + i);
        }
    }

    public static void main(String[] args) throws InterruptedException {
        TestJoin testJoin = new TestJoin();
        Thread thread = new Thread(testJoin);
        thread.start();


        for (int i = 0; i < 1000; i++) {
            if (i == 200){
                thread.join();
            }
            System.out.println("main " + i);
        }
    }
}
// 主线程执行到200之后，等到VIP线程执行结束后主线程才能执行，之前主线程会一直在阻塞状态
```



### 观测线程状态

- Thread.State 枚举类型
  - NEW: 尚未启动的线程处于此状态
  - RUNNABLE：在JVM中执行的线程处于此状态
  - BLOCKED：被阻塞等待监视器锁定的线程处于此状态
  - WAITING：正在等待另一个线程执行特定动作的线程处于此状态
  - TIMED_WAITING：正在等待另一个线程执行动作达到指定等待时间的线程处于此状态
  - TERMINATED：一经推出的线程处于此状态
  - 一个线程可以在给定时间点处于一个状态。这些状态是不反应任何操作系统线程状态的虚拟机状态

```java
//观察测试线程状态
public class TestState {

    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(() -> {
            for (int i = 0; i < 5; i++) {
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            System.out.println("////////");
        });

        //观察状态
        Thread.State state = thread.getState();
        System.out.println(state);    //NEW

        thread.start();
        state = thread.getState();
        System.out.println(state);     //RUNNABLE

        while (state != Thread.State.TERMINATED){   //只要线程终止，就一直输出状态
            Thread.sleep(100);
            state = thread.getState();              // 更新线程状态
            System.out.println(state);              // 输出线程状态
         }
    }
}
/*
NEW
RUNNABLE
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
////////
TERMINATED
*/
```





## 线程优先级

- Java提供一个线程调度器来监控程序中启动后进入就绪状态的所有线程，线程调度器按照优先级决定该调度哪个线程执行
- 线程的优先级用十个数字表示，范围从1~10；
  - `Thread.MIN_PRIORITY = 1;`
  - `Thread.MAX_PRIORITY = 10;`
  - `Thread.NORM_PRIORITY = 5;`
- 使用以下方法改变或获取优先级
  - `getPriority()、setPriority(int xxx)`
  - 优先级的设定应该在`start()`调度前

```java
//测试线程优先级
public class TestPriority {
    public static void main(String[] args) {
        //主线程默认优先级
        System.out.println(Thread.currentThread().getName() + "-------->" + Thread.currentThread().getPriority());

        MyPriority myPriority = new MyPriority();

        Thread t1 = new Thread(myPriority);
        Thread t2 = new Thread(myPriority);
        Thread t3 = new Thread(myPriority);
        Thread t4 = new Thread(myPriority);
        Thread t5 = new Thread(myPriority);
        Thread t6 = new Thread(myPriority);

        t1.start();

        t2.setPriority(2);
        t2.start();

        t3.setPriority(4);
        t3.start();

        t4.setPriority(6);
        t4.start();

        t5.setPriority(8);
        t5.start();

        t6.setPriority(Thread.MAX_PRIORITY);
        t6.start();
    }
}

class MyPriority implements Runnable{

    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName() + "-------->" + Thread.currentThread().getPriority());
    }
}
/*
main-------->5
Thread-0-------->5
Thread-1-------->2
Thread-3-------->6
Thread-4-------->8
Thread-2-------->4
Thread-5-------->10
*/ 
//还是看JVM心情
```

**优先级低只是意味着获得调度的概率低，并不是一定会后被调用，一切都要看CPU调度**



## 守护线程

- 线程分为用户线程和守护线程
- 虚拟机必须确保用户的线程执行完毕
- 虚拟机不用等待守护线程执行完毕
- 如，后台记录操作日志，监控内存，垃圾回收等待

```java
//测试守护线程
//上帝守护你
public class TestDeamon {
    public static void main(String[] args) {
        God god = new God();
        You1 you = new You1();

        Thread thread = new Thread(god);
        thread.setDaemon(true); //默认为false表示用户线程
        thread.start();

        new Thread(you).start();
    }
}


//上帝
class God implements Runnable{

    @Override
    public void run() {
        while (true){
            System.out.println("上帝保佑着你");
        }
    }
}


//你
class You1 implements Runnable{

    @Override
    public void run() {
        for (int i = 0; i < 36500; i++) {
            System.out.println("你一生都获得很开心");
        }
        System.out.println("======GoodBye !  world");
    }
}
/*
执行结果太长了， 总之就是用户线程结束之后，JVM就终止输出了，不管守护线程结束了没有
*/
```

## 线程同步（重点）

处理多线程问题时，多个线程访问同一个对象，并且某些线程还想修改这的对象，这时候我们就需要线程同步。

线程同步其实就是一种等待机制，多个需要访问此对象的线程进入这个对象的等待池形成队列，等待前面的线程使用完毕，下一个线程再使用



### 并发

同一对象被多个线程同时操作



### 队列和锁

两者结合就能确保线程的安全性





- 由于同一进程的多个线程共享同一块存储空间，再带来方便的同时，也带来了访问的冲突问题，为了保证数据在方法中被访问时的正确性，再访问时加入锁机制`synchronized`，当一个县城获得对象的排他锁，独占资源，其他线程必须等待，使用后释放锁即可。存在如下问题：
  - 一个线程持有锁会导致其他所有需要此锁的线程挂起
  - 在多线程竞争下，加锁，释放锁会导致比较多的上下文切换和调度演示，引起性能问题
  - 如果一个优先级高的线程等待一个优先级低的线程释放锁 会导致性能优先级倒置，引起性能问题



### 同步方法

- 由于我们要通过private 关键字来保证数据对象只能拿通过方法访问，所以我们只需要针对方法提出一套机制，者套机制就是`synchronized`关键字，它还包括两种用法：`synchronized`方法和`synchronized`块

  ```java
  public synochronized void method(int args){}; //同步方法
  ```

- `synochronized`方法控制着对象的访问，每个对象对应一把锁，每个`synochronized`方法阿斗必须调用该方法对象的锁才能执行，否则线程会阻塞，方法一旦执行，就会独占锁，知道方法返回才释放锁，后面被阻塞的线程才能获得这个锁，继续执行。

- **若将一个大方法声明为`synochronized`将会影响效率**

  - 方法里面有需要修改的内容才需要锁，锁的太多，浪费资源。



### 同步块

- 同步块：`synchronized(Obj) {}`
- Obj 称之为同步监视器
  - Obj可以是任何对象，但是推荐使用共享资源作为同步监视器
  - 同步方法中无需指定同步监视器，因为同步方法的同步监视器就是this，就是这个对象本身，或者是Class；
- 同步监视器的执行过程
  - 第一个线程访问，锁定同步监视器，执行其中代码
  - 第二个线程访问，发现同步监视器被锁定，无法访问
  - 第一个线程访问完毕，解锁同步监视器
  - 第二个线程访问，发现同步监视器没有锁，然后锁定并访问



### 死锁

多个线程各自占有一些共享资源，并且相互等待其他线程占有的资源才能运行，而导致两个或多个线程都在等待对方释放资源，都停止执行的情形。某一个同步块同时拥有==两个以上对象的锁==时，就可能会发生死锁



#### 避免方法

- 产生死锁的四个必要条件
  - 互斥条件：一个资源每次只能被一个进程使用
  - 请求与保持条件：一个进程因为请求资源而阻塞时，对以获得的资源保持不放
  - 不剥夺条件：进程以获得的资源，在未使用完之前，不能强行剥夺
  - 循环等待条件：若干进程之间形成一种头尾相接的循环等待资源关系
- 破解其中一个或多个就可以避免死锁发生





### Lock（锁）

![1590637178737](C:\Users\aira\AppData\Roaming\Typora\typora-user-images\1590637178737.png)



#### `synchronized`和`Lock`的对比

![1590637799115](C:\Users\aira\AppData\Roaming\Typora\typora-user-images\1590637799115.png)



## 线程协作通信（生产者消费者模式）



### 线程通信

![1590638062235](C:\Users\aira\AppData\Roaming\Typora\typora-user-images\1590638062235.png)



### 线程通信--分析

![1590638178154](C:\Users\aira\AppData\Roaming\Typora\typora-user-images\1590638178154.png)



![1590638216797](C:\Users\aira\AppData\Roaming\Typora\typora-user-images\1590638216797.png)



### 解决方式1 -- 》管程法

![1590638724586](C:\Users\aira\AppData\Roaming\Typora\typora-user-images\1590638724586.png)

```java
//测试 ： 生产者消费者模型 --》 利用缓冲区解决 ：管程法
public class TestPC {
    public static void main(String[] args) {
        SynContainer container = new SynContainer();
        new Productor(container).start();
        new Consumer(container).start();
    }
}

//生产者
class Productor extends Thread{
    SynContainer container;

    public Productor(SynContainer container){
        this.container = container;
    }
    //生产鸡
    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            container.push(new Chicken(i));
            System.out.println("生产了" + i + "只鸡");
        }
    }
}

//消费者
class Consumer extends Thread{
    SynContainer container;

    public Consumer(SynContainer container){
        this.container = container;
    }

    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            System.out.println("消费了"+ container.pop().id);
        }
    }
}

class Chicken{
    public int id;

    public Chicken(int i) {
        this.id = i;
    }
}

//缓冲区
class SynContainer{
    //需要容器大小
    private Chicken[] chickens = new Chicken[10];
    private int count = 0;
    //生产者放入产品
    public synchronized void push(Chicken chicken){
        //如果容器满了就要等待消费
        if (count==chickens.length){
            //通知消费者，生产等待
            try {
                this.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        //如果没有满，就丢入产品
        chickens[count] = chicken;
        count++;

        //通知消费者消费
        this.notifyAll();
    }

    //消费者消费产品

    public synchronized Chicken pop(){
        //判断能否消费
        if (count == 0){
            //等待生产
            try {
                this.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

        //可以消费
        count--;
        Chicken chicken = chickens[count];

        //通知生产者生产
        this.notifyAll();

        return chicken;
    }
}
```



### 解决方式2 ---》信号灯法

- 并发协作模型”生产者/消费者模式“ --->信号灯法

```java

//测试消费者-生产者模型 ： 标志位法
public class TestPC2 {
    public static void main(String[] args) {
        TV tv = new TV();
        new Player(tv).start();
        new Watch(tv).start();
    }
}


//生产者 ---》 演员
class Player extends Thread{
    TV tv;
    public Player(TV tv){
        this.tv = tv;
    }

    @Override
    public void run() {
        for (int i = 0; i < 20; i++) {
            if (i % 2 == 0){
                this.tv.play("快乐大本营");
            }else {
                this.tv.play("bilibili!");
            }
        }
    }
}

//消费者  ---》 观众
class Watch extends Thread{
    TV tv;
    public Watch(TV tv){
        this.tv = tv;
    }

    @Override
    public void run() {
        for (int i = 0; i < 20; i++) {
            tv.watch();
        }
    }
}

//产品 -》》 节目
class TV{
    //演员表演的时候，观众等待 T
    //观众观看，演员等待       F
    String voice;
    boolean flag = true;

    //表演
    public synchronized void play(String voice){
        if (!flag){
            try {
                this.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        System.out.println("表演了：" + voice);

        this.notifyAll();
        this.voice = voice;
        this.flag = !this.flag;
    }
    //观看
    public synchronized void watch(){
        if (flag){
            try {
                this.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        System.out.println("观看了" + voice);

        //通知演员表演
        this.notifyAll();
        this.flag = !this.flag;
    }
}
/*
表演了：快乐大本营
观看了快乐大本营
表演了：bilibili!
观看了bilibili!
...重复十次
*/
```



## 线程池

### 使用线程池

![1590648774257](C:\Users\aira\AppData\Roaming\Typora\typora-user-images\1590648774257.png)

![1590648949335](C:\Users\aira\AppData\Roaming\Typora\typora-user-images\1590648949335.png)

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

//测试线程池
public class TestPool {
    public static void main(String[] args) {
        //创建服务，创建线程池
        //newFixedThreadPool 参数未线程池大小
        ExecutorService service = Executors.newFixedThreadPool(10);

        //执行
        service.execute(new MyThread());
        service.execute(new MyThread());
        service.execute(new MyThread());
        service.execute(new MyThread());
        service.execute(new MyThread());

        //关闭连接
        service.shutdownNow();
    }
}

class MyThread implements Runnable {

    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName());
    }
}

```

