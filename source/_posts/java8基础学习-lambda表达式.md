###android studio ������java8����
���ȣ����java�汾����java8����build.gradle������java8
```
compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
}
```
��������Ŀ�н����Rxjava,��Ҫ����
```
jackOptions {
       enabled true
}
```
jack��������android��6.0�汾��ӵı�������Ҳ����˵android6.0�汾�Ĵ��붼��jack����������ģ����ǰ�׿�ٷ���������վ�Ѿ��ų�blog�����Ժ��android studio�У���ʼ֧��java8����ʼ����jack(����ʲôʱ�䣬���ù�����ԭ����Moving forward, Java 8 language features will be natively supported by the Android build system��)������֮ǰʹ�õ�jack���Ĵ�����Ȼ֧�֣�����[��������](https://android-developers.googleblog.com/2017/03/future-of-java-8-language-feature.html)
***
###Lambda���ʽ
Ҫ���lambda���ʽ������Ҫ�˽���Ǻ���ʽ�ӿڣ�functional interface��������˵������ʽ�ӿ���ֻ����һ�����󷽷��Ľӿڡ�����Java��׼���е�[java.lang.Runnable](http://download.java.net/jdk8/docs/api/java/lang/Runnable.html)��[java.util.Comparator](http://download.java.net/jdk8/docs/api/java/util/Comparator.html)���ǵ��͵ĺ���ʽ�ӿڡ����ں���ʽ�ӿڣ����˿���ʹ��Java�б�׼�ķ���������ʵ�ֶ���֮�⣬������ʹ��lambda���ʽ������ʵ�ֶ���������ںܴ�̶��ϼ򻯴����ʵ�֡���ʹ��lambda���ʽʱ��ֻ��Ҫ�ṩ��ʽ�����ͷ����塣���ں���ʽ�ӿ�ֻ��һ�����󷽷�������ͨ��lambda���ʽ�����ķ�����Ϳ϶������Ψһ�ĳ��󷽷���ʵ�֣�������ʽ���������Ϳ��Ը��ݷ������������������Զ��ƶϡ�
�ڹ����д���һ���̵߳�д�����£�
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
java8��lambda���ʽһ���ʽ
>(argument) -> {body}

argument��ʾ���Ƿ����е���ʽ���������û��ֱ�ӷſգ������body�Ƿ�����
����demo1�еĴ�����Լ�����
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
��������ֻ��һ��������Կ��Լ�����
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

������һЩ������lambda���ʽ�����Լ��ϲ�������
```
(int a, int b) -> {  return a + b; }
() -> System.out.println("Hello World");
(String s) -> { System.out.println(s); }
() -> 42
() -> { return 3.1415 };
a -> return a * a; // ��ʽ������ֻ��a
```

��Ҳ�����Լ���д����ʽ�ӿ�

```
@FunctionalInterface
public interface Annimal {
    public abstract void play();
}
```
@FunctionalInterface�� Java 8 �¼����һ�ֽӿڣ�����ָ���ýӿ����������Ǹ��� Java ���Թ淶����ĺ���ʽ�ӿڡ�Java 8 ��������һЩ Lambda ���ʽ����ʹ�õĺ���ʽ�ӿڣ�����ע�͵Ľӿڲ�����Ч�ĺ���ʽ�ӿ�ʱ������ʹ�� @FunctionalInterface ����������Ĵ���
���⣬��java8�нӿ�֧�ַ�����ʵ�֣��Ժ���ʽ�ӿڲ���Ӱ��
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
�������д��������뱨��Ҳ�Ƿ��Ϲ淶�ģ�������������ͨ�ķ����ͻᱨ����������ڽӿ���ʹ��ע��@FunctionalInterface���������������Ŷӵ�������Ա��������ӿ�������µķ�����
###Lambda ���ʽ�������������
ʹ���������� Lambda ���ʽ��һ���������ڹؼ��ʵ�ʹ�á����������࣬�ؼ���?this
?���Ϊ�����࣬������ Lambda ���ʽ���ؼ���?this
?���Ϊд�� Lambda ���ⲿ�ࡣ
Lambda ���ʽ�����������һ��ͬ�������ߵı��뷽����Java ���������� Lambda ���ʽ��������ת��Ϊ�������˽�к�������ʹ�� Java 7 ���¼ӵ�?invokedynamic
?ָ�̬�󶨸÷��������� Java ��ν� Lambda ���ʽ����Ϊ�ֽ��룬Tal Weiss д��һƪ[�ܺõ�����](http://www.takipiblog.com/2014/01/16/compiling-Lambda-expressions-scala-vs-java-8/)��