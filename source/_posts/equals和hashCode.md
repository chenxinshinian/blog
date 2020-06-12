---
title: equals和hashCode
date: 2020-05-29 20:27:06
img: https://chenxinshinian.oss-cn-beijing.aliyuncs.com/img/20200529204403.jpg
categories: 编程语言
tags:
    - Java
---

<img src="https://chenxinshinian.oss-cn-beijing.aliyuncs.com/img/20200529204403.jpg"></img>
Oject类是所有类的超类，hashCode和equals都是Object类中的方法，所以Java中所有类都默认继承了这两个方法。

Object类中的equals:
```java
    public boolean equals(Object obj) {
        return (this == obj);
    }
```



Object类中的hashCode:
```java
public native int hashCode();
```

#### ==和equals的区别
可以看到equals代码实质上也是调用了==，而==用来比较引用数据类型的话比较的是内存的地址也就是引用是否相等，如果==用来比较基本数据类型的话是直接比较值是否相等。
这是因为基本数据类型和变量的引用是直接存储在堆栈里面，而变量是存储在堆里面，然后引用再指向堆内存中的地址。对于比较基本数据类型来说在直接使用==即可。
如果要比较引用是否相同的同样使用==即可，但是对于大多数类来说比较引用相同没有什么意义，因为如果有两个雇员对象的姓名、薪水、和雇佣日期都一样的话我们就应该认为他们是相等的。
所以Java提供了equals方法，我们通过重写equlas就可以达到我们想要的效果。

#### 如何比较String
Java中字符串是一个比较特殊的引用数据类型，如果要比较他的话需要使用equals来比较。
因为String是存储在字符串常量池，String有两个特性：一个是不可变 二是编译器可以让字符串共享。
因为字符串共享的特性如果复制了一个字符串，原始字符串与复制字符串就指向相同常量池的地址。
```java
public static void main(String[] args) {
        String  a = "test";
        String b = a;
        System.out.println(a == b); // true

        String c = "test2";
        String d = "test2";
        System.out.println(c == d); //true
    }
```
这样看起来好像没什么问题，但是
```java
    public static void main(String[] args) {
        String greeting = "Hello";
        System.out.println(greeting == "Hello"); //true

        System.out.println(greeting.substring(0,3) == "Hel"); //false
    }
```
如果虚拟机是最终将两个相同的字符串共享，就可以使用==来检测是否相当。但实际上只有字符串常量是共享的，而+和substring等操作拼接或者截取的字符串不是共享的。
所以要避免使用==运算符来比较字符串的相等性。

String中已经重写来Object类中的equals：
```java
 public boolean equals(Object anObject) {
        if (this == anObject) {
            return true;
        }
        if (anObject instanceof String) {
            String anotherString = (String)anObject;
            int n = value.length;
            if (n == anotherString.value.length) {
                char v1[] = value;
                char v2[] = anotherString.value;
                int i = 0;
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
使用equlas来比较：
```java
public static void main(String[] args) {
        String greeting = "Hello";
        System.out.println(greeting == "Hello"); //true

        System.out.println(greeting.substring(0,3).equals("Hel") ); //true
    }
```

#### 如何定义equals
如果想要根据两个对象的状态相等来进行检测的话，如：雇员对象的姓名、薪水、雇佣日期一直的话就认为他是相等的

```java
package equals;

import java.time.*;
import java.util.Objects;

public class Employee
{
   private String name;
   private double salary;
   private LocalDate hireDay;
   public boolean equals(Object otherObject)
   {
      if (this == otherObject) return true;

      if (otherObject == null) return false;

      if (getClass() != otherObject.getClass()) return false;

      Employee other = (Employee) otherObject;

      return Objects.equals(name, other.name) && salary == other.salary && Objects.equals(hireDay, other.hireDay);
   }
}
```

因为引用数据可以是null，直接使用name.equals(other.name) 可能会报错，所以使用Objects.equals(name, other.name)

objecs.equals:
```java
 public static boolean equals(Object a, Object b) {
        return (a == b) || (a != null && a.equals(b));
    }
```


1. 显示参数命名为otherObject, 稍后需要将他转换成另一个叫做作other的变量
2. 检测this 与 otherObject是否引用同一个对象
`if(other == null) return true`
这条语句只是一个优化。实际上，这是一种经常采用的形式。因为计算这个等式要比一个一个地比较类中的域付出的代价小的多。
3. 检测otherObject是否为null，如果为null，返回false
4. 比较this与otherObject是否是同一个类。如果equals的语义在每个子类中有所改变，就用getClass检测：
`if (getClass() != otherObject.getClass()) return false;`
如果子类都拥有统一的语义，就用instanceof检测：
`if(!(otherObject instanceof ClassName)) return false`
5. 将otherObject转换为相应的类的类型变量：
`ClassName other = (ClassName)otherObject`
6. 现在开始对所有需要比较的域进行比较了。使用==比较基本类型，使用equals比较对象域。如果所有域都匹配返回true，否则返回false

如果在子类中重新定义equals，就要在其中包换调用super.equals(other)

```java
public class Manager extends Employee
{
   private double bonus;

public boolean equals(Object otherObject)
   {
      if (!super.equals(otherObject)) return false;
      Manager other = (Manager) otherObject;
      // super.equals checked that this and other belong to the same class
      return bonus == other.bonus;
   }
}
```


#### hashCode方法
散列码（hashCode）是由对象导出的一个整型值。
由于hashCode方法定义在Object类中，因此每个对象都有一个默认的散列码，其值为对象的存储地址。

String重写了hashCode，String的散列码是由内容导出的

```java
public static void main(String[] args) {
        String s = "OK";
        StringBuilder sb = new StringBuilder(s);
        String t = new String("OK");
        StringBuilder tb = new StringBuilder(t);
        System.out.println("s.hashCode:" + s.hashCode());
        System.out.println("t.hashCode:" + t.hashCode());
        System.out.println("sb.hashCode:" + sb.hashCode());
        System.out.println("tb.hashCode:" + tb.hashCode());
    }
```
输出：
```
s.hashCode:2524
t.hashCode:2524
sb.hashCode:1956725890
tb.hashCode:356573597
```

如果重新定义equals方法，就必须重新定义hashCode方法，以便用户将对象插入到<a href="https://baike.baidu.com/item/%E5%93%88%E5%B8%8C%E8%A1%A8">散列表</a>中

重新定义hashCode可以调用Objects.hash:
```java
    public static int hash(Object... values) {
        return Arrays.hashCode(values);
    }
```

示例:
```java
   public int hashCode()
   {
      return Objects.hash(name, salary, hireDay); 
   }

```