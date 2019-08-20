# String

## 概述

`String`类用于表示字符串，字符串的值在创建后不能被更改，即String对象是不可变的，他们可以在字符串常量池中被共享。

该类中提供了遍历字符串中每个字符、比较字符串、查找字符串、获取字符串子串以及提供一个由原字符串拷贝而来的大写或小写字符串。

此类为连接运算符`+`提供特殊支持，以实现字符串的连接以及将其他对象转换为字符串。字符串的连接是通过`StringBuilder`(`StringBuffer`)及其`append`方法实现的。

String类实现了三个接口，其继承关系如下：

[String类](..//img//String//String类.png)

该类API详见[java.lang.String](https://docs.oracle.com/javase/8/docs/api/)

> **String的不可变性如何体现？**
>
> 一旦一个String对象在堆中被创建，就无法被修改。String类中的方法都没有改变字符串本身的值，都是返回了一个新对象。故如果需要可修改的字符串，应使用StringBuffer或StringBuilder。否则每次修改时都会创建新的对象，会消耗大量时间在GC上。

> **为什么要把String设计为不可变的？**
>
> - 如果字符串可变时，当两个引用指向同一个字符串时，对其中一个做修改就会影响另一个。
>
> - 可以用于**缓存HashCode**。在String类的属性中，定义了一个整数hash来存储String的散列值，所以一旦对象被创建，该hash值也无法改变。当需要使用该对象的hashCode时，直接返回hash即可。这也就意味着每次在使用一个字符串的hashcode的时候不用重新计算一次，这样更加**高效**。关于这一点，可以参考[hashCode](#hashCode)方法的实现。
>
> - 如HashMap等经常会使用到字符串的散列码，字符串的不可变性可以保证hashCode永远保持一致，这样就可以避免一些问题。如以下这个例子:
>
>   ```java
>   public void testImmutable() {
>       Set<String> set = new HashSet<>();
>       set.add("a");
>       set.add("b");
>       set.add("c");
>       for (String s : set) {
>           String str = s;
>           str = "a";
>       }
>   }
>   ```
>
>   如果String类不是不可变的，那么以上的操作就会使得set中产生了重复的字符串，这显然是违反set的设计原则的。
>
> - **安全性**。String被广泛的使用在其他Java类中充当参数。比如网络连接、打开文件等操作。如果字符串可变，那么类似操作可能导致安全问题。因为某个方法在调用连接操作的时候，他认为会连接到某台机器，但是实际上并没有（其他引用同一String对象的值修改会导致该连接中的字符串内容被修改）。可变的字符串也可能导致反射的安全问题，因为他的参数也是字符串。
>
> - 不可变对象天生就是**线程安全**的。因为不可变对象不能被改变，所以他们可以自由地在多个线程之间共享。不需要任何同步处理。
>
> 总之，String被设计成不可变的主要目的是为了**安全和高效**

## 属性

```java
/** The value is used for character storage. */
// char类型的final数组，用来储存字符，其引用不可更改
private final char value[];

/** Cache the hash code for the string */
// 存储String的散列值，默认值为0
private int hash; // Default to 0

/** use serialVersionUID from JDK 1.0.2 for interoperability */
private static final long serialVersionUID = -6849794470754667710L;
```

由此可见，`String`底层的数据结构是一个`char`型数组`value`

> **为什么要将`value[]`声明为`final`?**
>
> 这是与String的不可变形相适应的。

## 构造器

### 空参构造器

```java
public String() {
    this.value = "".value;
}
```

该构造函数创建了一个空的String对象，但是并没有在常量池中创建`“”`，如以下代码：

```java
@Test
public void testConstructor() {
    String str = new String();
    str = "abc";
}
```

通过javap进行反编译，可以看出，其现在堆中创建了一个空的String对象，后在常量池中创建了一个`"abc"`对象，然后将其引用传递给空String对象：

```bash
...
Constant pool:
  ...
  #23 = Utf8               abc  // 在常量池中创建了"abc"
  #24 = Utf8               string/StringTest
  #25 = Utf8               java/lang/Object
{
  ...
  public void testConstructor();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=2, args_size=1
         0: new           #2                  // class java/lang/String new了一个String对象
         3: dup
         4: invokespecial #3                  // Method java/lang/String."<init>":()V
         7: astore_1
         8: ldc           #4                  // String abc 将引用指向常量池中的"abc"
...
```

### 通过String初始化

```java
public String(String original) {
    this.value = original.value;
    this.hash = original.hash;
}
```

该方法通过创建一个新的String对象，再将新对象的引用指向原对象在常量池中的引用：

```java
@Test
public void testConstructorString() {
    String str = "abc";
    String str1 = new String(str);
    System.out.println(str == str1);  // false
}
```

### 通过char[]初始化

```JAVA
public String(char value[]) {
    this.value = Arrays.copyOf(value, value.length);
}
```

其将char数组中的值复制到此对象的value数组中，使用的拷贝函数`Arrays.copyof`实际上调用的是`System.arraycopy`函数进行深拷贝。

```java
public String(char value[], int offset, int count) {
    if (offset < 0) {
        throw new StringIndexOutOfBoundsException(offset);
    }
    if (count <= 0) {
        if (count < 0) {
            throw new StringIndexOutOfBoundsException(count);
        }
        if (offset <= value.length) {
            this.value = "".value;
            return;
        }
    }
    // Note: offset or count might be near -1>>>1.
    if (offset > value.length - count) {
        throw new StringIndexOutOfBoundsException(offset + count);
    }
    this.value = Arrays.copyOfRange(value, offset, offset+count);
}
```

增加了偏移量和offset和要复制的长度count，若offset不合法，会抛出`StringIndexOutOfBoundsException`

### :question:使用int[]初始化

### :question:使用byte[]初始化

### 使用StringBuffer初始化

```java
public String(StringBuffer buffer) {
    synchronized(buffer) {
        this.value = Arrays.copyOf(buffer.getValue(), buffer.length());
    }
}
```

### 使用StringBuilder初始化

```java
public String(StringBuilder builder) {
    this.value = Arrays.copyOf(builder.getValue(), builder.length());
}
```

通过`StringBuffer`和`StringBuilder`都可以进行初始化，但是这两个类都定义了`toString`方法，更建议使用`toString`方法来创建String对象，若无需考虑线程安全，优先选择`StringBuilder`

### 直接赋值

要区分以下两种情况：

```java
@Test
public void testString() {
    String str1 = new String();
    String str2 = "";
}
```

对于使用空参构造器的情况，并没有在常量池中创建对象，但是在堆中创建了一个String对象

而直接赋值时，会在常量池中创建一个""空串的对象，且没有在堆中创建对象，而是将引用指向了常量池中的空串。

## 方法

### equals

```java
public boolean equals(Object anObject) {
    // 如果引用相同，返回true
    if (this == anObject) {
        return true;
    }
    // 若参数是String对象，则继续比较，否则返回false
    if (anObject instanceof String) {
        // 向下转型
        String anotherString = (String)anObject;
        int n = value.length;
        // 若参数对象value的长度与此对象value长度不相同，直接返回false，否则继续比较
        if (n == anotherString.value.length) {
            char v1[] = value;
            char v2[] = anotherString.value;
            int i = 0;
            // 比较两对象value数组中的值，若完全相同返回true
            while (n-- != 0) {
                if (v1[i] != v2[i])
                    return false;
                i++;
            }
            return true;
        }
    }
    return false;
}
```

### hashCode

```java
public int hashCode() {
    int h = hash;
    // 若哈希值为初始值为0且value数组长度大于0时计算哈希值
    if (h == 0 && value.length > 0) {
        char val[] = value;

        for (int i = 0; i < value.length; i++) {
            h = 31 * h + val[i];
        }
        hash = h;
    }
    return h;
}
```

由以上实现可以看出，由于String的不可变性，所以一旦使用过此函数，其散列值就不会再发生变化

### intern

```java
public native String intern();
```

调用此方法时，会先使用`equals`检查字符串常量池中是否有与调用对象相等的字面量对象，若有，返回常量池中对象的引用；若没有，则在常量池中添加一个该对象，然后再返回其引用。

由此可见，对于任何两个字符串`s`和`t` ，当且仅当`s.equals(t)`是`true`， `s.intern() == t.intern()`是`true`

请看下面这个例子：

```java
@Test
public void testIntern() {
    String s1 = "test";  
    String s2 = new String("test");
    String s3 = s1.intern();
    System.out.println(s1 == s2);  // false
    System.out.println(s1 == s3);  // true
}
```

以上结果可以这么解释：

`String s1 = "test"; `是在字符串常量池中，创建了一个常量`"test"`，并直接返回该常量的引用；

`String s2 = new String("test");`，其会先在堆中创建一个String对象，并检查字符串常量池中检查是否已有`"test"`常量，若有，则将此对象指向该常量的引用。

而`String s3 = s1.intern();`则是检查`"test"`是否存在于常量池中，若存在，则直接返回此常量值的引用。“如果存在就直接返回其引用”，指的是会把字面量对象的引用直接返回给定义的对象。这个过程是不会在Java堆中再创建一个String对象的。

### compareTo

该方法返回一个整型值，有三种情况：

- 如果参数字符串等于此字符串，返回0；
- 如果此字符串的字典序比字符串参数小，返回的值小于0；
- 如果此字符串的字典序比字符串参数大，返回的值大于0

```java
public int compareTo(String anotherString) {
    int len1 = value.length;
    int len2 = anotherString.value.length;
    // 获取此字符串长度与参数字符串长度的较小值
    int lim = Math.min(len1, len2);
    char v1[] = value;
    char v2[] = anotherString.value;

    int k = 0;
    // 遍历两个字符串的字符直到k == lim为止
    while (k < lim) {
        char c1 = v1[k];
        char c2 = v2[k];
        // 若当前位置上两字符串的字符不相同，返回此字符串与参数字符串当前字符的差值
        if (c1 != c2) {
            return c1 - c2;
        }
        k++;
    }
    // 若前lim位字符都相同，则返回两字符串长度的差值
    return len1 - len2;
}
```

### startWith

```java
public boolean startsWith(String prefix, int toffset) {
    // 此字符串的字符数组
    char ta[] = value;
    // 比较的起始索引
    int to = toffset;
    // 参数字符串的字符数组
    char pa[] = prefix.value;
    int po = 0;
    // 参数字符串的长度
    int pc = prefix.value.length;
    // Note: toffset might be near -1>>>1.
    // 若起始索引小于零或起始索引大于此字符串的长度与参数字符串长度的差值，返回false
    if ((toffset < 0) || (toffset > value.length - pc)) {
        return false;
    }
    // 将此字符串(从起始索引开始)与参数字符串(从0索引开始)逐一字符比较，若有字符不同，返回false
    while (--pc >= 0) {
        if (ta[to++] != pa[po++]) {
            return false;
        }
    }
    return true;
}
```



```java
public boolean startsWith(String prefix) {
    return startsWith(prefix, 0);
}
```

### endWith

```java
public boolean endsWith(String suffix) {
    return startsWith(suffix, value.length - suffix.value.length);
}
```

其实就是将`startsWith(String prefix, int toffset)`方法的起始索引位置设为`value.length - suffix.value.length`，设参数字符串长度为`n`，即从倒数第n个字符开始比较。

### concat

```java
public String concat(String str) {
    int otherLen = str.length();
    // 若参数字符串长度为0，直接返回调用该方法的字符串(不新建对象)
    if (otherLen == 0) {
        return this;
    }
    int len = value.length;
    // 创建一个长度等于待拼接的两字符串总长的字符数组，并将本字符串的值拷贝到此数组前0到len-1的位置上
    char buf[] = Arrays.copyOf(value, len + otherLen);
    // 将参数字符串的值拷贝到buf数组的len到(len + otherLen - 1)的位置
    str.getChars(buf, len);
    // 返回使用buf数组创建的一个新的String对象
    return new String(buf, true);
}
```

其中，`getChar`方法的实现如下：

```java
/**
 * Copy characters from this string into dst starting at dstBegin.
 * This method doesn't perform any range checking.
 */
// 即将字符串中的value[]拷贝到dst[]的dstBegin位置上
void getChars(char dst[], int dstBegin) {
    System.arraycopy(value, 0, dst, dstBegin, value.length);
}
```

### replace

该方法用于将替换原字符串中的指定字符，并返回一个替换后的新字符串

```java
public String replace(char oldChar, char newChar) {
    if (oldChar != newChar) {
        int len = value.length;
        int i = -1;
        char[] val = value; /* avoid getfield opcode */

        // 找到第一次出现oldChar的索引i
        while (++i < len) {
            if (val[i] == oldChar) {
                break;
            }
        }
        if (i < len) {
            // 将oldChar出现之前的位置的值复制到新建的数组buf[]中
            char buf[] = new char[len];
            for (int j = 0; j < i; j++) {
                buf[j] = val[j];
            }
            // 从i开始判断val各位置的字符是否为oldChar，若是，将newChar赋给相应位置的buf[]，否则，
            // 将val[]该位置上的值赋给buf[]
            while (i < len) {
                char c = val[i];
                buf[i] = (c == oldChar) ? newChar : c;
                i++;
            }
            return new String(buf, true);
        }
    }
    // 如果要替换的新旧字符相同，则直接返回原字符串
    return this;
}
```

### 其他常用方法

#### valueOf

#### trim

#### indexOf

#### lastIndexOf

#### substring



## 运算符“+”重载

```java
@Test
public void testConcat() {
    String str1 = "a" + "b";
    String str2 = str1 + "c";

}
```

使用CFR反编译后：

```java
@Test
public void testConcat() {
    String str1 = "ab";
    String str2 = new StringBuilder().append(str1).append("c").toString();
}
```

可见JVM对字符串的拼接做了一些优化

## 参考资料

[**《成神之路-基础篇》Java基础知识——String相关**](https://www.hollischuang.com/archives/1330)

[**Stringbuilder vs concatentation**](http://www.benf.org/other/cfr/stringbuilder-vs-concatenation.html)

[Why String is immutable in Java?](https://www.programcreek.com/2013/04/why-string-is-immutable-in-java/)

[Java_String](http://www.52dadudu.com/posts/3f00ad1.html)

### **拓展阅读**

[深入解析String#intern](https://tech.meituan.com/2014/03/06/in-depth-understanding-string-intern.html)

[Java 中new String("字面量") 中 "字面量" 是何时进入字符串常量池的?](https://www.zhihu.com/question/55994121)



# AbstractStringBuilder

## 属性

```java
/**
 * value数组用于存储字符
 */
char[] value;

/**
 * count用于表示当前字符串的长度
 */
int count;
```

应注意，与String不同，该类的底层数据结构并没有被final修饰，因而其引用可以被改变。

## 构造器

此类有两个构造方法，一个为空实现，另一个则将value初始化为制定容量的数组

```java
/**
 * This no-arg constructor is necessary for serialization of subclasses.
 */
AbstractStringBuilder() {
}

/**
 * Creates an AbstractStringBuilder of the specified capacity.
 */
AbstractStringBuilder(int capacity) {
    value = new char[capacity];
}
```

## 方法

### append

#### append(String): AbstractStringBuilder

以`append(String str)`为例，分析`AbstractStringBuilder`如何对字符串进行拼接

```java
public AbstractStringBuilder append(String str) {
    if (str == null)
        return appendNull();
    int len = str.length();
    // 确保容量足够，否则扩容
    ensureCapacityInternal(count + len);
    // 将str中的值拷贝到value[]中count索引以及之后的位置上
    str.getChars(0, len, value, count);
    // 更新当前字符串长度
    count += len;
    return this;
}
```

其中，`appendNull()`的实现如下，实际上就是在value字符数组中加上'n''u''l''l'四个字符

```java
private AbstractStringBuilder appendNull() {
    int c = count;
    ensureCapacityInternal(c + 4);
    final char[] value = this.value;
    value[c++] = 'n';
    value[c++] = 'u';
    value[c++] = 'l';
    value[c++] = 'l';
    count = c;
    return this;
}
```

注意到这两个方法都调用了同一个函数用来确保当前数组的容量足够存放新的数据

#### ensureCapacityInternal

该方法用于判断对于当前所需的容量是否应对数组进行扩容，如果需要则将value的引用指向一个容量为newCapacity的新数组。

```java
/**
* For positive values of {@code minimumCapacity}, this method
* behaves like {@code ensureCapacity}, however it is never
* synchronized.
* If {@code minimumCapacity} is non positive due to numeric
* overflow, this method throws {@code OutOfMemoryError}.
*/
private void ensureCapacityInternal(int minimumCapacity) {
    // overflow-conscious code
    // 如果当前所需的数组容量minimumCapacity大于当前value数组的容量时，
    // 调用copyOf方法创建一个扩容到容量为newCapacity的新数组并将value的值复制到该数组中
    if (minimumCapacity - value.length > 0) {
        value = Arrays.copyOf(value, newCapacity(minimumCapacity));
    }
}
```

#### newCapacity

```java
private int newCapacity(int minCapacity) {
    // overflow-conscious code
    // 先将数组容量设为原来的2倍+2
    int newCapacity = (value.length << 1) + 2;
    // 若此容量仍比所需的最小容量小，则将所需最小容量作为数组容量
    if (newCapacity - minCapacity < 0) {
        newCapacity = minCapacity;
    }
    // 判断更新后的数组容量是否为非正数
    // 或此容量是否大于规定的最大数组容量MAX_ARRAY_SIZE(其被设置为Integer.MAX_VALUE - 8)
    // 若是，返回hugeCapacity
    // 否则，将此容量作为新数组的容量用于扩容
    return (newCapacity <= 0 || MAX_ARRAY_SIZE - newCapacity < 0)
        ? hugeCapacity(minCapacity)
        : newCapacity;
}
```

从以上一系列方法，我们可以看出其与String的一些不同点：

- 当数组容量充足时，直接在原数组上进行添加的操作
- 当数组容量不足时，将value的引用指向另一个数组

#### append(char[]): AbstractStringBuilder

对于字符数组的拼接，实现上则有些不同：

```java
public AbstractStringBuilder append(char[] str) {
    int len = str.length;
    ensureCapacityInternal(count + len);
    // 将char型数组中的元素值复制到扩容后的value数组中
    System.arraycopy(str, 0, value, count, len);
    count += len;
    return this;
}
```

#### append(boolean) / append(char): AbstractStringBuilder

对于`boolean`和`char`型元素的添加，逻辑是相似的，即先判断value数组是否需要扩容，若需要则扩容，并直接在扩容后的数组写入相应的元素值：

```java
public AbstractStringBuilder append(boolean b) {
    if (b) {
        ensureCapacityInternal(count + 4);
        value[count++] = 't';
        value[count++] = 'r';
        value[count++] = 'u';
        value[count++] = 'e';
    } else {
        ensureCapacityInternal(count + 5);
        value[count++] = 'f';
        value[count++] = 'a';
        value[count++] = 'l';
        value[count++] = 's';
        value[count++] = 'e';
    }
    return this;
}

@Override
public AbstractStringBuilder append(char c) {
    ensureCapacityInternal(count + 1);
    value[count++] = c;
    return this;
}
```

#### append(int): AbstractStringBuilder 

添加int则是先判断该整数的位数(负数还应加上负号位)，然后再通过包装类`integer`的`getChar`方法，将该元素的值转为字符后复制到value数组中：

```java
public AbstractStringBuilder append(int i) {
    if (i == Integer.MIN_VALUE) {
        append("-2147483648");
        return this;
    }
    int appendedLength = (i < 0) ? Integer.stringSize(-i) + 1
        : Integer.stringSize(i);
    int spaceNeeded = count + appendedLength;
    ensureCapacityInternal(spaceNeeded);
    Integer.getChars(i, spaceNeeded, value);
    count = spaceNeeded;
    return this;
}
```

### insert

#### insert(int, String): AbstractStringBuilder 

```java
public AbstractStringBuilder insert(int offset, String str) {
    // 判断插入的位置是否越界，注意此处的length()返回的是字符串的长度而非字符数组的容量
    if ((offset < 0) || (offset > length()))
        throw new StringIndexOutOfBoundsException(offset);
    if (str == null)
        str = "null";
    int len = str.length();
    // 数组扩容
    ensureCapacityInternal(count + len);
    // 将原value的offset以及之后的已使用的位置(存有字符的位置)的值拷贝到offset+len后
    System.arraycopy(value, offset, value, offset + len, count - offset);
    // 再将str的值拷贝到扩容后的value数组的offset位置
    str.getChars(value, offset);
    count += len;
    return this;
}
```

其他insert的方法的思路大同小异

### :question:其他常用方法

#### reverse

# StringBuilder

## 概述

`StringBuilder`可以兼容`StringBuffer`的API，但是`StringBuilder`无法保证线程安全，故如果在没有线程安全问题的情况下，建议使用性能较高的`StringBuilder`，而在对线程安全有要求的情境下，使用`StringBuffer`

![StringBuilder](D:\Java\Notes\Java\我读源码\img\String\StringBuilder.png)



## 属性

以下两个属性是声明在其父类`AbstractStringBuilder`中

```java
/**
 * value数组用于存储字符
 */
char[] value;

/**
 * count用于表示当前字符串的长度
 */
int count;
```



## 构造器

```java
// 调用空参构造器时，会调用父类的构造器将value数组容量初始化为16
public StringBuilder() {
    super(16);
}

// 将字符数组容量初始化为指定值
public StringBuilder(int capacity) {
    super(capacity);
}

// 将字符数组容量初始化为参数字符串的长度 + 16
public StringBuilder(String str) {
    super(str.length() + 16);
    append(str);
}

public StringBuilder(CharSequence seq) {
    this(seq.length() + 16);
    append(seq);
}
```

## 方法

### append

实际上append方法调用的仍然是父类的append，详见`AbstractStringBuilder`中的[append](# append)

```java
@Override
public StringBuilder append(String str) {
    super.append(str);
    return this;
}
```

### insert

与append一样，该方法调用的仍然是父类的insert，详见`AbstractStringBuilder`中的[insert](#insert)

```
/**
* @throws StringIndexOutOfBoundsException {@inheritDoc}
*/
@Override
public StringBuilder insert(int offset, String str) {
super.insert(offset, str);
return this;
}
```



# :question:StringBuffer



> **String，StringBuilder，StringBuffer有什么区别？**
>
> String被final修饰，一旦创建无法更改，每次更改则是在新创建对象；StringBuilder和StringBuffer则是可修改的字符串 
>
> StringBuffer 被synchronized 修饰，同步，线程安全；StringBuilder并没有对方法进行加同步锁，所以是非线程安全的。如果程序不是多线程的，那么使用StringBuilder效率高于StringBuffer。
>
> String被final修饰，一旦被创建，无法更改
>
> String类的所有方法都没有改变字符串本身的值，都是返回了一个新的对象。
>
> 如果你需要一个可修改的字符串，应该使用StringBuilder或者 StringBuffer。
>
> 如果你只需要创建一个字符串，你可以使用双引号的方式，如果你需要在堆中创建一个新的对象，你可以选择构造函数的方式。
>
> 在使用StringBuilder时尽量指定大小这样会减少扩容的次数，有助于提升效率。
>
> 



https://www.jianshu.com/p/799c4459b808
