###android studio 下配置java8环境
首先，你的java版本得是java8，在build.gradle下配置java8
```
compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
}
```
如果你的项目中结合了Rxjava,需要配置
```
jackOptions {
       enabled true
}
```
jack工具链是android在6.0版本添加的编译器，也就是说android6.0版本的代码都是jack工具链编译的，但是安卓官方开发者网站已经放出blog，在以后的android studio中，开始支持java8，开始废弃jack(具体什么时间，引用官网的原话：Moving forward, Java 8 language features will be natively supported by the Android build system。)，但是之前使用的jack风格的代码依然支持，附上[官网链接](https://android-developers.googleblog.com/2017/03/future-of-java-8-language-feature.html)
***
###Lambda表达式
要理解lambda表达式，首先要了解的是函数式接口（functional interface）。简单来说，函数式接口是只包含一个抽象方法的接口。比如Java标准库中的[java.lang.Runnable](http://download.java.net/jdk8/docs/api/java/lang/Runnable.html)和[java.util.Comparator](http://download.java.net/jdk8/docs/api/java/util/Comparator.html)都是典型的函数式接口。对于函数式接口，除了可以使用Java中标准的方法来创建实现对象之外，还可以使用lambda表达式来创建实现对象。这可以在很大程度上简化代码的实现。在使用lambda表达式时，只需要提供形式参数和方法体。由于函数式接口只有一个抽象方法，所以通过lambda表达式声明的方法体就肯定是这个唯一的抽象方法的实现，而且形式参数的类型可以根据方法的类型声明进行自动推断。
在工作中创建一个线程的写法如下：
```
/**
demo1
**/
public void runThread() {
    new Thread(new Runnable() {
        public void run() {
            System.out.println("test");
        }
    }).start();
}
```
java8中lambda表达式一般格式
>(argument) -> {body}

argument表示的是方法中的形式参数，如果没有直接放空，后面的body是方法体
所以demo1中的代码可以简化如下
```
/**
demo2
**/
public void runThread() {
    new Thread(
         () -> {System.out.println("test");}
    ).start();
}
```
方法体总只有一句代码所以可以继续简化
```
/**
demo3
**/
public void runThread() {
    new Thread(
         () -> System.out.println("test");
    ).start();
}
```

下面是一些常见的lambda表达式，可以加上参数类型
```
(int a, int b) -> {  return a + b; }
() -> System.out.println("Hello World");
(String s) -> { System.out.println(s); }
() -> 42
() -> { return 3.1415 };
a -> return a * a; // 形式参数中只有a
```

你也可以自己编写函数式接口

```
@FunctionalInterface
public interface Annimal {
    public abstract void play();
}
```
@FunctionalInterface是 Java 8 新加入的一种接口，用于指明该接口类型声明是根据 Java 语言规范定义的函数式接口。Java 8 还声明了一些 Lambda 表达式可以使用的函数式接口，当你注释的接口不是有效的函数式接口时，可以使用 @FunctionalInterface 解决编译层面的错误。
另外，在java8中接口支持方法的实现，对函数式接口并不影响
```
@FunctionalInterface
@RequiresApi(api = Build.VERSION_CODES.N)
public interface Annimal {
    public abstract void play();
    default void fly(){
      System.out.println("fly");
    }
    static void eat(){
      System.out.println("eat");
    }
}
```
上面的书写并不会编译报错，也是符合规范的，但是如果添加普通的方法就会报错，所以最好在接口上使用注解@FunctionalInterface进行声明，以免团队的其他人员错误地往接口中添加新的方法。
###Lambda 表达式与匿名类的区别
使用匿名类与 Lambda 表达式的一大区别在于关键词的使用。对于匿名类，关键词?this
?解读为匿名类，而对于 Lambda 表达式，关键词?this
?解读为写就 Lambda 的外部类。
Lambda 表达式与匿名类的另一不同在于两者的编译方法。Java 编译器编译 Lambda 表达式并将他们转化为类里面的私有函数，它使用 Java 7 中新加的?invokedynamic
?指令动态绑定该方法，关于 Java 如何将 Lambda 表达式编译为字节码，Tal Weiss 写了一篇[很好的文章](http://www.takipiblog.com/2014/01/16/compiling-Lambda-expressions-scala-vs-java-8/)。