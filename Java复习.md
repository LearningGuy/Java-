## 4. 面向对象
### 4.1. 多态的表现形式 ###
1. 最牛叉的一句话：多态将做什么和怎么做解耦。
2. 从字面上说，对象在运行时可以表现出不同的行为。典型的就是将子类对象的引用赋值给父类或接口。
3. 使用多态特性的方法包括重写，实现抽象方法和实现接口方法。
4. `private`方法隐式地是`final`的，构造器方法隐式地是`static`的。两者，加上对属性的访问不具备多态性。
5. 封装指得是隐藏了类内部的实现细节，仅通过部分公开接口对外提供访问。典型地就是`private`属性和`setter/getter`方法。还有一系列的访问控制符。
6. `has-a`关系通常用组合和接口来表达，而`is-a`关系通常是用继承来表达。

### 4.2 重写与覆盖 ###
1. 切记：如何向上转型后，该引用只能调用父类中包括的访问，除此之外将无法通过编译器检查。
2. 重写涉及方法的动态绑定（JVM底层通过`invokevirtual`和`invokeinterface`实现），重载设计方法的静态绑定（底层通过`invokestatic`和`invokespecial`实现）
3. 细节见JVM吧


### 4.3. 内部类的一些细节 ###
1. 内部类对象拥有对于外部类属性和方法的全部访问权限。内部实现：编译器隐式地为内部类对象生成了一个指向外部对象的引用。
2. 内部类对象的创建一定在外部类对象创建之后，可以通过`outer.new.Inner()`实现，相反通过`Outer.this()`获得外部对象的引用。
3. 普通内部类和静态内部类在使用上的区别是静态内部类可以直接初始化：`Inner inner = new Outer.Inner()`。静态内部类的一个典型应用是构造器模式。
4. 重点：为什么需要内部类？
   - 内部类提供了更好的封装性。在集合框架内就存在大量的内部类来辅助集合的增删改查。当然在外部设计同样目标的实现类也是可以的。但这样语义上没有内部类好。（高内聚，低耦合）
   - 从某种角度说它实现了多重继承的功能。往往是说内部类最吸引人的地方在于每一个内部类都独立地继承了一个实现或者接口，外部类的继承情况对内部类没影响。当然，这是在多重继承的环境下说的。
   - 没有令人疑惑的`is a`关系。感觉这句话的意思还是在说内部类的作用在于辅助外部类。
5. 语法上需要注意的地方：匿名内部类和成员内部类中均不能有`static`修饰的方法和变量

### 4.4 Object的共有方法 ###
`equals()`，`hashcode()`，`toString()`，`clone()`等等

## 5. 异常 ##
1. `Throwable`为所有异常和错误的基类，下面分`Error`和`Exception`，其中，`Error`主要为分为`VirtualMahcineError`和`AWTError`，虚拟机异常主要为栈溢出和内存溢出错误。`Exception`分为`IOException`和`RuntimeException`
2. `IOException`和`RuntimeException`是最常见的，前者为`Checked Exception`，后者为`Unchecked Exception`。受检异常和非受检异常的区别在于Java的处理方式，受检异常必须捕获或者继续向上抛出。非受检异常则不必须这么做。
3. `error`和`exception`的区别在于前者指系统运行中出现的严重错误，出现了JVM会终止线程。后者一般表示程序上的设计或实现问题。
4. jdk 1.7开始支持`try-with-resources`语句，支持资源的安全打开和释放。

## 6. 泛型 ##
1. 泛型要解决的问题
   - 创建更加泛化的代码。通常可以使用多态特性，使用Object或者接口来实现泛化代码。但这面临`ClassCastException`的问题。
   - 类型安全，将这类问题的检查从运行时提前至编译期。
   - 泛型和集合简直是天作之合。
2. 引入泛型要面临的一个问题：向后兼容。解决方法：类型擦除
   - 一句话：在泛型代码内部，无法获得有关泛型参数的信息，但像集合里面的get方法中由编译期自动插入了强转
   - 在泛型代码里，初始化泛型参数对应类是不允许的。解决方法是传入一个工厂对象，最简单的就是`Class<T>`对象工厂，因为它包含`newInstance()`方法。
3. 泛型与数组
   - 一句话：不能实例化一个参数化类型数组，但可以参数化数组本身的类型。这种看似矛盾的地方在于数组本身也是一个对象。
   - 原因在于数组安全检查要求数组元素的类型都是确定的，但类型擦除后所有元素都自动成为它的边界了。
   - 解决方案
     - 使用通配符：`List<?>[] a = new List<?>[arr_len]` 。原因在于`?`表示未知类型，因此涉及操作不会与类型相关，所以可以通过编译期检查。
     - 在调用方法的时候可以将形参设置成泛型数组`T[]`，这其实隐含了对数组本身类型的强转
4. 通配符的使用：`?`、`? extends SuperClass`和`? super SubClass`
   - `? extends SuperClass`表示一个`SuperClass`的子类，因此向这类泛型的列表中直接添加元素是不允许的，因为编译器不知道会是啥类型
   - `? super SubClass`表示逆变，向`set()`传递`SuperClass`的子类是允许的，因为向上转型是自动的
   - 上述两种方法能限制边界，但也会限制了集合中的一些功能。比如分别对应`set()`和`get()`的失效
   - `?`
5. 使用泛型时的一些问题
    - 基础类型不能作为泛型参数。解决：使用包装类。依然存在问题：比如`Integer[]`不能自动拆箱成`int[]`
    - 一个类不能实现同一种泛型接口的两种变体
    - 对`(T)`和`instanceOf`的操作会失效
    - 重载时的参数类型为泛型的不同变体将不允许
    - 基类劫持接口，下面操作是不允许的
        ``` java
            class SuperClass implements Comparable<SuperClass> {
                public int compareTo(Obeject o) {
                    return 0;
                }
            }
            class SubClass extends SuperClass implements Comparable<SubClass> {

            }
        ```
6. 自限定泛型：`class SelfBounded<T extends SelfBounded<T>>`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  # Java-
# Java-
