# Object类

## 概述



![1565620445056](C:\Users\Sin\AppData\Roaming\Typora\typora-user-images\1565620445056.png)



## 类方法

```java
private static native void registerNatives();

public final native Class<?> getClass();

public native int hashCode()

public boolean equals(Object obj) 
    
protected native Object clone() throws CloneNotSupportedException

public String toString()

public final native void notify()

public final native void notifyAll()

public final native void wait(long timeout) throws InterruptedException

public final void wait(long timeout, int nanos) throws InterruptedException

public final void wait() throws InterruptedException

protected void finalize() throws Throwable
```

## registerNatives()

该方法声明再静态代码块中，其在类初始化时执行

    static {
        registerNatives();
    }
## getClass()

```java
public final native Class<?> getClass();
```

getClass()是一个native方法，该方法声明为final，故不允许子类重写，返回当前**运行时类**的Class对象，

```java
@Test
public void getClassTest() {
    Number n = 10;
    Class<? extends Number> clazz = n.getClass();
    System.out.println(clazz.getName());  // java.lang.Integer
}
```

由以上例子可以看出，返回的Class类对象是多态的，其可以是调用者的子类。

## hashCode()

hashCode方法返回对象的哈希值，该哈希值通常用于散列表（如`HashSet`、`HashMap`）。覆盖equals方法时，通常都需要覆盖hashCode方法

```
public native int hashCode();
```

Object类中约定了hashCode的使用规范：

- 在应用程序执行期间，只要对象的equals方法的比较操作所用到的信息没有被更改，那么对同一个对象的多次调用，其返回的必须始终为同一个值。但在一个应用程序与另一个程序的执行过程中，执行hashCode方法所返回的值可以不一致
- 如果两个对象根据equals方法比较是相等的，那么调用这两个对象中的hashCode方法都必须产生同样的整数结果
- 如果两个对象根据equals方法比较是不相等的，那么调用这两个对象的hashCode方法，不一定要求该方法必须产生不同的结果。但是，给不相等的对象产生截然不同的整数结果，有可能提高散列表的性能。

关于如何提供一个适当的hashCode方法，可以参考[Effective Java中文版（第3版）](https://book.douban.com/subject/30412517/)中的*第11条：覆盖equals时总要覆盖hashCode*。

在散列码的计算过程中，应该将衍生域（即该域的值可由参与计算的其它域值计算出来）排除。

[Student类](# 测试类：Student)中由IDEA自动生成的hashCode()方法：

```java
@Override
public int hashCode() {
    int result = getId();
    result = 31 * result + (getName() != null ? getName().hashCode() : 0);
    return result;
}
```

以上代码中的乘法部分可以使得散列值依赖于域的顺序。而乘法中使用31，是因为其为一个奇素数，如果乘数是偶数，如果乘数是函数，其等价于移位运算，如果乘法溢出，信息就会丢失。而且31可以用移位和减法来代替乘法，即`31 * i == (i << 5) - i`，这种优化可以由虚拟机自动完成。

> **重写equals()为什么建议同时重写hashCode()？**
>
> 首先，不重写hashCode()违反了关键约定的第二条：相等的对象必须具有相等的散列值。违反该规定有可能导致程序不正常，如依赖于散列表的集合类可能无法保证其互异性
>
> 其次，重写hashCode()方法有利于在比较时提高性能，比如先比较散列值是否相等，若相等，再比较其内容是否相等。

```java
@Test
public void equalsAndHashCodeTest() {
    Set<Student> set = new HashSet<>();
    Student s1 = new Student(63, "Daddy", new Address("Shenzhen University, China"));
    Student s2 = new Student(63, "Daddy", new Address("Shenzhen University, China"));
    System.out.println(set.add(s1));
    System.out.println(set.add(s2));
}
```

​	

## equals()

```java
public boolean equals(Object obj) {    return (this == obj);}
```

应注意，默认equals()方法比较的是调用对象与参数obj的引用是否相同，即两个对象都指向同一块内存对象，即默认的equals()和==是等价的

如果希望对于内存不同但内容相同的两个对象调用此方法时返回true，则需要重写该方法，例如String类的equals()就重写了该方法（详见[String类](D:\Java\Notes\Java\我读源码\String类.md)）。

equals()方法实现了等价关系：

- **自反性**（reflexive）：对于任何非空的参考值`x` ， `x.equals(x)`应该返回`true` 。
- **对称性**（symmetric）：对于任何非空引用值`x`和`y` ， `x.equals(y)`应该返回`true`当且仅当`y.equals(x)`返回`true` 。
- **传递性**（transitive） ：对于任何非空引用值`x` ， `y`和`z` ，如果`x.equals(y)`返回`true`，且`y.equals(z)`回报`true` ，然后`x.equals(z)`应该返回`true` 。
- **一致性**（consistent）：对于任何非空引用值`x`和`y` ，只要`equals`的比较操作在对象中所用的信息没有被修改，多次调用`x.equals(y)`就会始终返回`true`或始终返回`false` 
- 对于任何非`null`的引用值`x`，`x.equals(null)`必须返回`false`

重写`equals`方法时应注意遵循以上的规定。

实现equals方法可以参考以下方法：

- 使用`==`操作符检查“参数是否为这个对象的引用”；
- 使用`instanceof`操作符来检查“参数是否为正确的类型”。正确的类型通常指该`equals`方法所在的类，但有时也可以指该类所实现的某个接口。如果类实现的接口改进了equals约定，运行在实现类之间进行比较，那么就使用接口。集合接口如`Set`、`List`、`Map`和`Map.Entry`具有这样的特性。
- 把参数转换成正确的类型

- 对于该类中的每个“关键”域，检查参数中的域是否域对象中对应的域相匹配。
- 覆盖equals时总要覆盖hashCode
- 不要将equals方法声明中的Object对象替换为其他类型，使用`@Override`注解可以避免这种错误

[Student类](# 测试类：Student)[][#测试类：Student]中由IDEA自动生成的equals()：

```java
@Override
public boolean equals(Object o) {
    // 如果两个对象的引用相同，那么必然相等
    if (this == o) return true;
    // 如果比较的对象不是Student或Student类的子类，则判断两者不相等
    if (!(o instanceof Student)) return false;

    // 向下转型
    Student student = (Student)o;

    // 判断属性是否相同
    if (getId() != student.getId()) return false;
    return getName() != null ? 
        getName().equals(student.getName()) : student.getName() == null;

}
```

其中，`instanceof`关键字是用来判断运行时对象是否为指定类或指定类子类的对象，如果希望继承于指定类型的类都可以依赖此方法进行比较，则可以采用`instanceof`，若希望此比较仅限于当前类型，可以使用`getClass()`方法代替。

## clone()

### 将一个对象的引用复制给另外一个对象的方法

#### 直接赋值

二者的引用是同一个对象，并**没有创建出一个新的对象**。可以把这种现象叫做引用的复制（或引用拷贝）

#### 浅拷贝

Java中默认的clone()方法都是浅拷贝。

浅拷贝是按位拷贝对象，它**会创建一个新对象**，这个对象有着原始对象属性值的一份精确拷贝。

如果属性是基本类型，拷贝的就是基本类型的**值**；

如果属性是内存地址（引用类型），拷贝的就是**内存地址**（即复制引用但不复制引用的对象） ，因此如果其中一个对象改变了这个地址，就会影响到另一个对象。

即**被拷贝对象的所有变量都含有与原来的对象相同的值，而所有的对其他对象的引用仍然指向原来的对象**。针对基本类型及其封装类都是将对应的基本类型值拷贝，对于其余对象，则仅是拷贝其引用，这导致最终里面的对象是同一个，更改一个，另一个的拷贝/原对象的对应值也随之更改。

**实现对象拷贝的类，必须实现Cloneable接口，并覆写clone()方法。**

![/clone-qian.png](https://segmentfault.com/img/remote/1460000010648519)

#### 深拷贝

被拷贝对象的所有变量都含有与原来的对象相同的值，除去那些引用其他对象的变量。那些引用其他对象的变量将指向被复制过的新对象，而不再是原有的那些被引用的对象。换言之，**深复制把要复制的对象所引用的对象都复制了一遍**。

**深拷贝在代码中，需要在clone方法中多书写调用这个类中其他类的变量的clone函数。**

![/clone-æ·±.png](https://segmentfault.com/img/remote/1460000010648520)

#### 利用序列化来做深复制

把对象写到流里的过程是序列化（Serilization）过程，其可以形象地称为“冷冻”或者“腌咸菜（picking）”过程；而把对象从流中读出来的反序列化（Deserialization）过程则叫做 “解冻”或者“回鲜(depicking)”过程。这里写到流中的对象则是原始对象的一个拷贝，因为原始对象还存在 JVM 中，所以我们可以利用对象的序列化产生克隆对象，然后通过反序列化获取这个对象。

注意每个需要序列化的类都要实现 Serializable 接口，如果有某个属性不需要序列化，可以将其声明为 transient，即将其排除在克隆属性之外。

### Java中的clone()

该方法限制所有调用 clone() 方法的对象，都必须实现 `Cloneable` 接口，否者将抛出 `CloneNotSupportedException` 这个异常。如何通过Java实现浅拷贝和深拷贝，详见[Student类](# 测试类：Student)

调用clone复制对象时，没有调用该调用类的构造函数。

```java
@Test
public void testClone() throws CloneNotSupportedException {
    Student student = new Student(57, "daddy", new Address("shenzhen"));
    // Deep Clone
    Student clone = (Student)student.clone();
    System.out.println("original: " + student);
    System.out.println("clone: " + clone);
    System.out.println("original == clone? " + (student == clone));
}

@Test
public void testCopyConstructor() {
    Student student = new Student(57, "daddy", new Address("shenzhen"));
    Student clone = new Student(student);
    System.out.println("original: " + student);
    System.out.println("clone: " + clone);
    System.out.println("original == clone? " + (student == clone));
}

@Test
public void testStaticCopyFactory() throws CloneNotSupportedException {
    Student student = new Student(57, "daddy", new Address("shenzhen"));
    Student clone = Student.copyStudent(student);
    System.out.println("original: " + student);
    System.out.println("clone: " + clone);
    System.out.println("original == clone? " + (student == clone));
}
```



## :question:notify() / notifyAll()

> **java 为什么wait(),notify(),notifyAll()必须在同步（Synchronized）方法/代码块中调用？**
>
> https://blog.csdn.net/qq_42145871/article/details/81950949

https://fangjian0423.github.io/2016/03/12/java-Object-method/#clone%E6%96%B9%E6%B3%95

## :question:wait()

sleep()和wait()的区别

## :question:finalize()

> final, finally, finalize

https://www.jb51.net/article/107058.htm

finalize()并非析构函数，也不是释放内存的方法。

## 测试类：Student

```java
package object;

public class Student implements Cloneable {
    int id;
    String name;
    Address address;

    public Student(int id, String name, Address address) {
        this.id = id;
        this.name = name;
        this.address = address;
        System.out.println("Construct Stuendent");
    }

    /**
     * 拷贝构造函数
     * @param student
     */
    public Student(Student student) {
        this.id = student.id;
        this.name = student.name;
        this.address = student.address;
    }

    /**
     * 用静态方法实现拷贝
     * @param student
     * @return
     * @throws CloneNotSupportedException
     */
    public static Student copyStudent(Student student) throws CloneNotSupportedException {
        if (!(student instanceof Cloneable)) {
            throw new CloneNotSupportedException();
        }
        return new Student(student.id, student.name, student.address);
    }

    // 此处省略getter和setter方法

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Student)) return false;

        Student student = (Student)o;

        if (getId() != student.getId()) return false;
        return getName() != null ? getName().equals(student.getName()) : student.getName() == null;

    }

    @Override
    public int hashCode() {
        int result = getId();
        result = 31 * result + (getName() != null ? getName().hashCode() : 0);
        return result;
    }

    @Override
    protected Object clone() throws CloneNotSupportedException {
        // 深拷贝
        Student student = (Student)super.clone();
        student.address = (Address)address.clone();
        return student;
        // 浅拷贝
        /* return super.clone(); */
    }

    @Override
    public String toString() {
        return "Student{" +
            "id=" + id +
            ", name='" + name + '\'' +
            ", address=" + address +
            '}';
    }
}

class Address implements Cloneable {
    String address;

    public Address(String address) {
        this.address = address;
    }

    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }

    // 不重写toString，直接打印地址值
    /*@Override
    public String toString() {
        return "Address{" +
                "address='" + address + '\'' +
                '}';
    }*/

    @Override
    protected Object clone() throws CloneNotSupportedException {
        return super.clone();
    }
}
```



## 参考资料

[**Effective Java中文版（第3版）**](https://book.douban.com/subject/30412517/)

[**Java clone – deep and shallow copy – copy constructors**](https://howtodoinjava.com/java/cloning/a-guide-to-object-cloning-in-java/)

[**Java Cloning: Copy Constructors vs. Cloning**](https://dzone.com/articles/java-cloning-copy-constructor-vs-cloning)

[Java根类Object的方法说明](https://fangjian0423.github.io/2016/03/12/java-Object-method/)

[JDK源码阅读（一）：Object源码分析](https://cloud.tencent.com/developer/article/1446940)	

[Java的深拷贝和浅拷贝](https://www.cnblogs.com/ysocean/p/8482979.html)

[Java漫谈-深拷贝与浅拷贝](https://cloud.tencent.com/developer/article/1343368)

[细说 Java 的深拷贝和浅拷贝](https://segmentfault.com/a/1190000010648514)

[Difference Between Shallow Copy Vs Deep Copy In Java](https://javaconceptoftheday.com/difference-between-shallow-copy-vs-deep-copy-in-java/)

[clone() Method Of java.lang.Object Class](https://javaconceptoftheday.com/clone-method-java-lang-object-class/)

**待阅读：**

[Everything You Need to Know About Java Serialization Explained](https://dzone.com/articles/what-is-serialization-everything-about-java-serial)

[clone() vs copy constructor vs factory method?](https://stackoverflow.com/questions/1106102/clone-vs-copy-constructor-vs-factory-method)

[Clone() vs Copy constructor- which is recommended in java ](https://stackoverflow.com/questions/2427883/clone-vs-copy-constructor-which-is-recommended-in-java)

