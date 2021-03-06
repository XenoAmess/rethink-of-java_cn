---
description: 众所周知，Java中存在数组。不那么众所周知，Java中的数组以及数组相关定义像屎一样。
---

# 1.1 Java中的数组

## 1.1.1 数组的多继承问题

对于每一个类，都有唯一与其对应的数组类。

先上代码：

```text
package a1.b1.c1;

class A {

}

class B extends A {

}

/**
 * @author XenoAmess
 */
public class Codes0 {
    public static void main(String[] args) {
        System.out.println(A.class);
        System.out.println(A[].class);
        System.out.println(B.class);
        System.out.println(B[].class);
        System.out.println(A[].class.equals(B[].class));
        System.out.println(new A[0] instanceof B[]);
        System.out.println(new B[0] instanceof A[]);
        System.out.println(A[].class.isAssignableFrom(B[].class));
        System.out.println(B[].class.isAssignableFrom(A[].class));
        System.out.println(new A[0].getClass().isAssignableFrom(new B[0].getClass()));
        System.out.println(new B[0].getClass().isAssignableFrom(new A[0].getClass()));
    }
}
```

运行上述代码，你会得到：

```text
C:\jdk8u252-b09\bin\java.exe -Dvisualvm.id=574497287560000 -javaagent:C:\Users\xenoa\AppData\Local\JetBrains\Toolbox\apps\IDEA-U\ch-0\202.6397.20\lib\idea_rt.jar=1815:C:\Users\xenoa\AppData\Local\JetBrains\Toolbox\apps\IDEA-U\ch-0\202.6397.20\bin -Dfile.encoding=UTF-8 -classpath C:\jdk8u252-b09\jre\lib\charsets.jar;C:\jdk8u252-b09\jre\lib\ext\access-bridge-64.jar;C:\jdk8u252-b09\jre\lib\ext\cldrdata.jar;C:\jdk8u252-b09\jre\lib\ext\dnsns.jar;C:\jdk8u252-b09\jre\lib\ext\jaccess.jar;C:\jdk8u252-b09\jre\lib\ext\localedata.jar;C:\jdk8u252-b09\jre\lib\ext\nashorn.jar;C:\jdk8u252-b09\jre\lib\ext\sunec.jar;C:\jdk8u252-b09\jre\lib\ext\sunjce_provider.jar;C:\jdk8u252-b09\jre\lib\ext\sunmscapi.jar;C:\jdk8u252-b09\jre\lib\ext\sunpkcs11.jar;C:\jdk8u252-b09\jre\lib\ext\zipfs.jar;C:\jdk8u252-b09\jre\lib\jce.jar;C:\jdk8u252-b09\jre\lib\jsse.jar;C:\jdk8u252-b09\jre\lib\management-agent.jar;C:\jdk8u252-b09\jre\lib\resources.jar;C:\jdk8u252-b09\jre\lib\rt.jar;D:\workspace\rethink-of-java_cn\target\classes a1.b1.c1.Codes0
class a1.b1.c1.A
class [La1.b1.c1.A;
class a1.b1.c1.B
class [La1.b1.c1.B;
false
false
true
true
false
true
false

Process finished with exit code 0
```

注意：

```text
System.out.println(A[].class.isAssignableFrom(B[].class));
```

的结果为true。

我们尝试用Java的逻辑去理解这个事情。

第一步，A\[\]是一个类，B\[\]也是一个类。

更严谨的说法可能是， new A\[0\]语法创建出的对象所属于的类是一个类。我们将这个类成为“A的数组类”。

第二步，A的数组类是B的数组类的父类。

另一个佐证是，

```text
System.out.println(new B[0] instanceof A[]);
```

的结果为true。

到目前为止，一切还在Java的逻辑框架内，并且看起来似乎并没有什么问题。

但是，Java中存在一个叫接口的东西。

先上代码：

```text
package a1.b1.c1;

/**
 * @author XenoAmess
 */
public class Codes1 {

    static class A {

    }

    static class B extends A implements C, D {

    }

    interface C {

    }

    interface D extends C {

    }

    public static void main(String[] args) {
        System.out.println(A[].class.isAssignableFrom(B[].class));
        System.out.println(C[].class.isAssignableFrom(B[].class));
        System.out.println(D[].class.isAssignableFrom(B[].class));
        System.out.println(C[].class.isAssignableFrom(D[].class));
    }
}
```

运行上述代码，你会得到：

```text
true
true
true
true

Process finished with exit code 0
```

这就产生了一个问题。

B的数组类是A的数组类的子类，并且B的数组类是C的数组类的子类。

显然，A的数组类与C的数组类并没有任何继承关系，所以B的数组类可以至少同时存在两套父类。注意，并不是两套父接口，而是两套可以产生实例的父类。（事实上，如果你愿意的话，增加接口E,F,G,H...用同样的方法你可以使B的数组类拥有无数个父类）。

动动手，动动脑：下面请自行尝试用Java的语法去做类似的事情，即编写几个类（必须是能够产生实例的类，而不是接口），让他们拥有与上述的几个数组类相同的继承关系。

……

很明显这是做不到的。

在Java的逻辑中，虽然接口可以多继承，但是任何的实例（即对象）都只能拥有一套父类，即只允许单继承，不允许多继承。

而在Java的数组系统中，设计人员突破了这种逻辑。他们随意地让一个数组类可以拥有任意多个父亲。这让我想起中国著名作家琼瑶女士。

这种多继承模式本身是否好并不是这里的重点，这里的重点还是，既然Java对于用户提出了“可构建实例的类必须单继承”的要求，那么自身在构建内部类的时候也必须满足该要求。只准州官放火，不许百姓点灯的行为是可耻的。一边宣称多继承多么不好并且禁止用户使用，一边自己用得欢，这种行为也是可耻的。

事实上这种行为的本质和某些自己类库里满天都是goto，却禁止别人用goto的老C++程序员，一点区别也没有。

## 1.1.2 数组的继承与功能窄化问题

按照Java的面向对象的常理，在通常情况下，子类应该是父类的扩展与细化。

一个业务，如果父类可以处理，那么子类也应该可以处理（此处存在一些例外，如Java的官方容器接口的一些Immutable实现，这个问题先不在这里讨论）。而一个业务如果子类不能处理，那么父类也理应不能处理（此处基本上不存在例外），即，子类理应比父类实现更多的功能，也更加好用。

举个例子，以apache-commons-lang（[https://github.com/apache/commons-lang](https://github.com/apache/commons-lang)）为例，org.apache.commons.lang3.tuple中，抽象类Pair类有两个子类ImmutablePair及MutablePair。

Pair类代表的概念是“对”，而ImmutablePair代表“不可变的对”，MutablePair代表“可变的对”。

在定义上，这两个类是比其父类更加细化。

而在功能上，这两个类比其父类做出了扩展，具体在此处就是：1，实现了父类的抽象函数，使得实例真正可用（使得可创建实例）。2，扩展了新的功能。例如，MutablePair中，添加了setLeft及setRight函数，用于设置左值和右值，这是父类所不能提供的。总之，做到了子类比父类实现更多的功能，也更加好用。

而在数组系统中，很奇妙地，他们并没有遵循相应的设计逻辑。

还是以1.1.1.Codes0为例，B的数组类B\[\]是A的数组类A\[\]的子类，按照逻辑，子类应该是父类的扩展，所以子类应该提供比父类更多更强的功能，而不是更少更弱的功能。

而我们很容易就能发现，实际上父类A\[\]比子类B\[\]的功能是更强的。一个A实例的可以放入A\[\]实例，却不能放入B\[\]实例。

这就造成了一个有趣的现象：子类反而比父类更加弱，即功能更加窄化。

对于有多个父亲的数组类，其比其每一个父亲都弱。

这是很反逻辑的事情。

## 1.1.3 解决方法

上面提到的这些问题其实有很多的解决方法。在尝试构建新语言时，读者们可以试一试下面几个方法，或者想一个新的，但是请务必吸取Java的教训。

### 1.1.3.1 取消各个数组类间的继承关系

这大概是最简单爽快的处理方式了。

既然数组类间的继承关系这么反人类，那我把他们都杀了，不就完事了。

并且这个解决方法实际上并不会带来什么大问题。

就我的Java经验来看，我根本没用到过数组类间的继承关系过。

并且我也不认为数组类间的继承关系有什么用处。

如果有哪位读者有对数组类间的继承关系很好地应用，或者非其不可的项目，请发来让我也长长见识。

### 1.1.3.2 不再给每个类型分配一个数组类，大家一起用一个

这也是很简单爽快的处理方式了。

既然数组类间的继承关系这么反人类，那我把他们都杀了，不就完事了。

仔细一想这么多数组类留着干嘛，何不一起也都杀了，就留一个Object\[\]，然后用语法糖模拟出多个数组类，省的再搞事。

这个方法是被证实了可行的，因为Java的容器类泛型实际上和这个解决方案有点类似。

ArrayList&lt;B&gt;和ArrayList&lt;A&gt;事实上都是ArrayList类，只是使用语法糖使编辑器可以区分。

这套逻辑不管怎么说，这么些年用下来了，至少是保证能用的。

（虽然伪泛型的问题也不少，但是那是另一个问题了）

