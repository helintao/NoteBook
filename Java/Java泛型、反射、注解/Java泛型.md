# 引入泛型的目的
Java 泛型就是把一种语法糖，通过泛型使得在编译阶段完成一些类型转换的工作，避免在运行时强制类型转换而出现ClassCastException，即类型转换异常。

# 泛型的好处
- 类型安全。类型错误现在在编译期间就被捕获到了，而不是在运行时当作java.lang.ClassCastException展示出来，将类型检查从运行时挪到编译时有助于开发者更容易找到错误，并提高程序的可靠性。

- 消除了代码中许多的强制类型转换，增强了代码的可读性。

- 为较大的优化带来了可能。

# 泛型的实质
允许在定义接口、类时声明类型形参，类型形参在整个接口、类体内可当成类型使用，几乎所有可使用普通类型的地方都可以使用这种类型形参。

# 泛型的使用
## 泛型类和泛型接口
```java
//定义接口时指定了一个类型形参，该形参名为E
public interface List<E> extends Collection<E> {
   //在该接口里，E可以作为类型使用
   public E get(int index) {}
   public void add(E e) {} 
}

//定义类时指定了一个类型形参，该形参名为E
public class ArrayList<E> extends AbstractList<E> implements List<E> {
   //在该类里，E可以作为类型使用
   public void set(E e) {
   .......................
   }
}
```
在JDK 1.7 增加了泛型的“菱形”语法：Java允许在构造器后不需要带完成的泛型信息，只要给出一对尖括号（<>）即可，Java可以推断尖括号里应该是什么泛型信息。
如下所示：
```java
Container<String,String> c1=new Container<>("name","hello");
Container<String,Integer> c2=new Container<>("age",22);
```

**泛型类派生子类:**  
当创建了带泛型声明的接口、父类之后，可以为该接口创建实现类，或者从该父类派生子类，需要注意：使用这些接口、父类派生子类时不能再包含类型形参，需要**传入具体的类型**。

错误的方式:public class A extends Container<K, V>{}

正确的方式:public class A extends Container<Integer, String>{}

也可以不指定具体的类型:public class A extends Container{}

## 泛型的方法
```java
public class Container<K, V>
 {
    //........................
    public K getkey() {
        return key;
    }
    public void setKey() {
        this.key = key;
    }
    //....................
}
```

但在另外一些情况下，在类、接口中没有使用泛型时，定义方法时想定义类型形参，就会使用泛型方法。如下方式：
```java
public class Main{
  public static <T> void out(T t){
       System.out.println(t);
  }
  public static void main(String[] args){
       out("hansheng");
       out(123);
  }
}
/**修饰符<T,S> 返回值类型 方法名 (形参列表){
    方法体
}*/
```

**方法声明中定义的形参只能在该方法里使用，而接口、类声明中定义的类型形参则可以在整个接口、类中使用。**

## 泛型构造器
```java
public class Person {
    public <T> Person(T t) {
        System.out.println(t);
    }
}

public static void main(String[] args){
        //隐式
        new Person(22);
        //显示
        new<String> Person("hello");
}
```

# 类型通配符
匹配任意类型的类型实参。

类型通配符是一个问号（？)，将一个问号作为类型实参传给List集合，写作：List<?>（意思是元素类型未知的List）。这个问号（？）被成为通配符，它的元素类型可以匹配任何类型。

# 带限通配符
## 上限通配符
如果想限制使用泛型类别时，只能用某个特定类型或者是其子类型才能实例化该类型时，可以在定义类型时，使用extends关键字指定这个类型必须是继承某个类，或者实现某个接口，也可以是这个类或接口本身。
```java
//它表示集合中的所有元素都是Shape类型或者其子类
List<? extends Shape>
```

## 下限通配符
如果想限制使用泛型类别时，只能用某个特定类型或者是其父类型才能实例化该类型时，可以在定义类型时，使用super关键字指定这个类型必须是是某个类的父类，或者是某个接口的父接口，也可以是这个类或接口本身。
```java
//它表示集合中的所有元素都是Circle类型或者其父类
List <? super Circle>
```

# 类型擦除
```java
Class c1=new ArrayList<Integer>().getClass();
Class c2=new ArrayList<String>().getClass();
System.out.println(c1==c2);
```

程序输出：
```
true
```

这是因为不管为泛型的类型形参传入哪一种类型实参，对于Java来说，它们依然被当成同一类处理，在内存中也只占用一块内存空间。从Java泛型这一概念提出的目的来看，其只是作用于代码编译阶段，在编译过程中，对于正确检验泛型结果后，会将泛型的相关信息擦出，也就是说，成功编译过后的class文件中是不包含任何泛型信息的。泛型信息不会进入到运行时阶段。

**在静态方法、静态初始化块或者静态变量的声明和初始化中不允许使用类型形参。由于系统中并不会真正生成泛型类，所以instanceof运算符后不能使用泛型类。**