---
layout:     post
title:      "Java8归纳总结之Lambda"
subtitle:   " \"Lambda\""
date:       2019-04-15 12:00:00
author:     "edsel"
header-img: "img/2.png"
catalog: true
tags:
    - java8
---

> “Yeah It's on. ”

### Lambda表达式的使用

#### 无参函数的简写

##### Java7写法

```java
// JDK7 匿名内部类写法
new Thread(new Runnable(){// 接口名
    @Override
    public void run(){// 方法名
        System.out.println("Thread run()");
    }
}).start();
```

##### Lambda写法

```java
// JDK8 Lambda表达式写法
new Thread(
        () -> System.out.println("Thread run()")// 省略接口名和方法名
).start();

// JDK8 Lambda表达式代码块写法
new Thread(
        () -> {
            System.out.print("Hello");
            System.out.println(" Hoolee");
        }
).start();
```

#### 带参函数的简写

##### Java7写法

```java
// JDK7 匿名内部类写法
List<String> list = Arrays.asList("I", "love", "you", "too");
Collections.sort(list, new Comparator<String>(){// 接口名
    @Override
    public int compare(String s1, String s2){// 方法名
        if(s1 == null)
            return -1;
        if(s2 == null)
            return 1;
        return s1.length()-s2.length();
    }
});
```

##### Java8写法

```java
// JDK8 Lambda表达式写法
List<String> list = Arrays.asList("I", "love", "you", "too");
Collections.sort(list, (s1, s2) ->{// 省略参数表的类型
    if(s1 == null)
        return -1;
    if(s2 == null)
        return 1;
    return s1.length()-s2.length();
});
```

### 功能接口

能使用lambda表达式的必须有相对应的功能接口`FunctionalInterface`,所谓的功能接口就是说接口中必须**只**含有一个抽象方法声明。你可以给接口添加`@FunctionalInterface`注解，编译器知道这个注解，并会在尝试向接口添加第二个抽象方法声明的时候跑出编译器错误。（当然，不添加`@FunctionalInterface`也不会有错误）

```java
@FunctionalInterface
interface Converter<F, T> {
    T convert(F from);
}
```

```java
Converter<String, Integer> converter = (from) -> Integer.valueOf(from);
Integer converted = converter.convert("123");
System.out.println(converted);    // 123
```

#### 方法和构造函数引用

我们可以通过静态方法引用来进一步简化上面的示例代码：

```java
Converter<String, Integer> converter = Integer::valueOf;
Integer converted = converter.convert("123");
System.out.println(converted);   // 123
```

Java 8使您可以通过`::`关键字传递方法或构造函数的引用。上面的示例显示了如何引用静态方法。我们也可以引用对象方法：

```java
class Something {
    String startsWith(String s) {
        return String.valueOf(s.charAt(0));
    }
}
```

```java
Something something = new Something();
Converter<String, String> converter = something::startsWith;
String converted = converter.convert("Java");
System.out.println(converted);    // "J"
```

下面将演示如何在构造函数中使用`::`：

```java
class Person {
    String firstName;
    String lastName;

    Person() {}

    Person(String firstName, String lastName) {
        this.firstName = firstName;
        this.lastName = lastName;
    }
}
```

```java
interface PersonFactory<P extends Person> {
    P create(String firstName, String lastName);
}
```

```java
PersonFactory<Person> personFactory = Person::new;
Person person = personFactory.create("Peter", "Parker");
```

我们不是手动地实现工厂，而是通过构造函数引用将所有内容粘合在一起。我们通过创建对Person构造函数的使用`Person::new`。Java编译器通过匹配`PersonFactory.create`来自动选择正确的构造函数。

### Lambda作用域

#### 访问局部变量

我们可以从lambda表达式**读取**外部范围的**最终的**局部变量：

```java
final int num = 1;
Converter<Integer, String> stringConverter =
        (from) -> String.valueOf(from + num);

stringConverter.convert(2);     // 3
```

但与匿名对象不同的是，变量不一定要申明为`final`：

```java
int num = 1;
Converter<Integer, String> stringConverter =
        (from) -> String.valueOf(from + num);

stringConverter.convert(2);     // 3
```

但被使用的变量必须是隐式的`final`，如以下代码就会报错：

```java
int num = 1;
Converter<Integer, String> stringConverter =
        (from) -> String.valueOf(from + num);
num = 3;
```

报错信息为：`Variable used in lambda expression should be final or effectively final`

#### 访问字段和静态变量

与局部变量相比，我们对lambda表达式中的实例字段和静态变量都有读写访问权限。 

```java
class Lambda4 {
    static int outerStaticNum;
    int outerNum;

    void testScopes() {
        Converter<Integer, String> stringConverter1 = (from) -> {
            outerNum = 23;
            return String.valueOf(from);
        };

        Converter<Integer, String> stringConverter2 = (from) -> {
            outerStaticNum = 72;
            return String.valueOf(from);
        };
    }
}
```

### 内置功能接口

#### Predicate

Predicate是一个参数的布尔值函数，该接口包括多个多个默认的函数来组合多个predicates成为复杂的逻辑单元（and，or，negate）

```java
Predicate<String> predicate = (s) -> s.length() > 0;

predicate.test("foo");              // true
predicate.negate().test("foo");     // false

Predicate<Boolean> nonNull = Objects::nonNull;
Predicate<Boolean> isNull = Objects::isNull;

Predicate<String> isEmpty = String::isEmpty;
Predicate<String> isNotEmpty = isEmpty.negate();
```

#### Function

函数接受一个参数并生成结果。默认方法可用于将多个函数链接在一起（compose，andThen）。

```java
Function<String, Integer> toInteger = Integer::valueOf;
Function<String, String> backToString = toInteger.andThen(String::valueOf);

backToString.apply("123");     // "123"
```

#### Supplier

Supplier产生一个给定基本类型的结果，不像Functions，Suppliers不接受参数

```java
Supplier<Person> personSupplier = Person::new;
personSupplier.get();   // new Person
```

#### Consumers

Consumers表示对单一输入参数的操作。

```java
Consumer<Person> greeter = (p) -> System.out.println("Hello, " + p.firstName);
greeter.accept(new Person("Luke", "Skywalker"));
```

#### Comparators

Java8给原有的Comparators接口增加了新的默认方法。

```java
Comparator<Person> comparator = (p1, p2) ->
	p1.firstName.compareTo(p2.firstName);

Person p1 = new Person("John", "Doe");
Person p2 = new Person("Alice", "Wonderland");

comparator.compare(p1, p2);             // > 0
comparator.reversed().compare(p1, p2);  // < 0
```
#### Optional

Optional不是功能接口，而是一个用来有效防止`NullPointerException`的工具。

```java
Optional<String> optional = Optional.of("bam");

optional.isPresent();           // true
optional.get();                 // "bam"
optional.orElse("fallback");    // "bam"

optional.ifPresent((s) -> System.out.println(s.charAt(0)));     // "b"
```

## Lambda表达式和Java集合类

###Java8集合类接口变化

![1555225999234](https://github.com/zehuaiWANG/Test/tree/master/_posts/img/java8_1.png)

![1555225999235](https://github.com/zehuaiWANG/Test/tree/master/_posts/img/java8_2.png)

### Collection中的新方法

#### forEach()

该方法的签名为`void forEach(Consumer<? super E> action)`，作用是对容器中的每个元素执行`action`指定的动作，其中`Consumer`是个函数接口，里面只有一个待实现方法`void accept(T t)` .

需求：*假设有一个字符串列表，需要打印出其中所有长度大于3的字符串.* 

##### Java7

```java
// 使用曾强for循环迭代
ArrayList<String> list = new ArrayList<>(Arrays.asList("I", "love", "you", "too"));
for(String str : list){
    if(str.length()>3)
        System.out.println(str);
}
```

##### Java8

```java
// 使用forEach()结合匿名内部类迭代
ArrayList<String> list = new ArrayList<>(Arrays.asList("I", "love", "you", "too"));
list.forEach(new Consumer<String>(){
    @Override
    public void accept(String str){
        if(str.length()>3)
            System.out.println(str);
    }
});
```

```java
// 使用forEach()结合Lambda表达式迭代
ArrayList<String> list = new ArrayList<>(Arrays.asList("I", "love", "you", "too"));
list.forEach( str -> {
        if(str.length()>3)
            System.out.println(str);
    });
```

#### removeIf()

该方法签名为`boolean removeIf(Predicate<? super E> filter)`，作用是**删除容器中所有满足filter指定条件的元素**，其中`Predicate`是一个函数接口，里面只有一个待实现方法`boolean test(T t)`，同样的这个方法的名字根本不重要，因为用的时候不需要书写这个名字。

需求：*假设有一个字符串列表，需要删除其中所有长度大于3的字符串。*

我们知道如果需要在迭代过程冲对容器进行删除操作必须使用迭代器，否则会抛出`ConcurrentModificationException`，所以上述任务传统的写法是：

```java
// 使用迭代器删除列表元素
ArrayList<String> list = new ArrayList<>(Arrays.asList("I", "love", "you", "too"));
Iterator<String> it = list.iterator();
while(it.hasNext()){
    if(it.next().length()>3) // 删除长度大于3的元素
        it.remove();
}
```

现在使用`removeIf()`方法结合匿名内部类，我们可是这样实现:

```java
// 使用removeIf()结合匿名名内部类实现
ArrayList<String> list = new ArrayList<>(Arrays.asList("I", "love", "you", "too"));
list.removeIf(new Predicate<String>(){ // 删除长度大于3的元素
    @Override
    public boolean test(String str){
        return str.length()>3;
    }
});
```

上述代码使用`removeIf()`方法，并使用匿名内部类实现`Precicate`接口。 

```java
// 使用removeIf()结合Lambda表达式实现
ArrayList<String> list = new ArrayList<>(Arrays.asList("I", "love", "you", "too"));
list.removeIf(str -> str.length()>3); // 删除长度大于3的元素
```

#### replaceAll()

该方法签名为`void replaceAll(UnaryOperator<E> operator)`，作用是**对每个元素执行operator指定的操作，并用操作结果来替换原来的元素**。其中`UnaryOperator`是一个函数接口，里面只有一个待实现函数`T apply(T t)`。

需求：*假设有一个字符串列表，将其中所有长度大于3的元素转换成大写，其余元素不变。*

Java7及之前似乎没有优雅的办法：

```java
// 使用下标实现元素替换
ArrayList<String> list = new ArrayList<>(Arrays.asList("I", "love", "you", "too"));
for(int i=0; i<list.size(); i++){
    String str = list.get(i);
    if(str.length()>3)
        list.set(i, str.toUpperCase());
}
```

使用`replaceAll()`方法结合匿名内部类可以实现如下： 

```java
// 使用匿名内部类实现
ArrayList<String> list = new ArrayList<>(Arrays.asList("I", "love", "you", "too"));
list.replaceAll(new UnaryOperator<String>(){
    @Override
    public String apply(String str){
        if(str.length()>3)
            return str.toUpperCase();
        return str;
    }
});
```

上述代码调用`replaceAll()`方法，并使用匿名内部类实现`UnaryOperator`接口。我们知道可以用更为简洁的Lambda表达式实现： 

```java
// 使用Lambda表达式实现
ArrayList<String> list = new ArrayList<>(Arrays.asList("I", "love", "you", "too"));
list.replaceAll(str -> {
    if(str.length()>3)
        return str.toUpperCase();
    return str;
});
```

#### sort()

该方法定义在`List`接口中，方法签名为`void sort(Comparator<? super E> c)`，该方法**根据c指定的比较规则对容器元素进行排序**。`Comparator`接口我们并不陌生，其中有一个方法`int compare(T o1, T o2)`需要实现，显然该接口是个函数接口。

需求：*假设有一个字符串列表，按照字符串长度增序对元素排序。*

由于Java7以及之前`sort()`方法在`Collections`工具类中，所以代码要这样写：

```java
// Collections.sort()方法
ArrayList<String> list = new ArrayList<>(Arrays.asList("I", "love", "you", "too"));
Collections.sort(list, new Comparator<String>(){
    @Override
    public int compare(String str1, String str2){
        return str1.length() - str2.length();
    }
});
```

现在可以直接使用`List.sort()方法`，结合Lambda表达式，可以这样写： 

```java
// List.sort()方法结合Lambda表达式
ArrayList<String> list = new ArrayList<>(Arrays.asList("I", "love", "you", "too"));
list.sort((str1, str2) -> str1.length() - str2.length());
```

### Map中的新方法

####forEach()

该方法签名为`void forEach(BiConsumer<? super K,? super V> action)`，作用是**对Map中的每个映射执行action指定的操作**，其中`BiConsumer`是一个函数接口，里面有一个待实现方法`void accept(T t, U u)`。`BinConsumer`接口名字和`accept()`方法名字都不重要，请不要记忆他们。

需求：*假设有一个数字到对应英文单词的Map，请输出Map中的所有映射关系．*

Java7以及之前经典的代码如下：

```java
// Java7以及之前迭代Map
HashMap<Integer, String> map = new HashMap<>();
map.put(1, "one");
map.put(2, "two");
map.put(3, "three");
for(Map.Entry<Integer, String> entry : map.entrySet()){
    System.out.println(entry.getKey() + "=" + entry.getValue());
}
```

使用`Map.forEach()`方法，结合匿名内部类，代码如下：

```java
// 使用forEach()结合匿名内部类迭代Map
HashMap<Integer, String> map = new HashMap<>();
map.put(1, "one");
map.put(2, "two");
map.put(3, "three");
map.forEach(new BiConsumer<Integer, String>(){
    @Override
    public void accept(Integer k, String v){
        System.out.println(k + "=" + v);
    }
});
```

上述代码调用`forEach()`方法，并使用匿名内部类实现`BiConsumer`接口。当然，实际场景中没人使用匿名内部类写法，因为有Lambda表达式：

```java
// 使用forEach()结合Lambda表达式迭代Map
HashMap<Integer, String> map = new HashMap<>();
map.put(1, "one");
map.put(2, "two");
map.put(3, "three");
map.forEach((k, v) -> System.out.println(k + "=" + v));
}
```

####getOrDefault()

该方法跟Lambda表达式没关系，但是很有用。方法签名为`V getOrDefault(Object key, V defaultValue)`，作用是**按照给定的key查询Map中对应的value，如果没有找到则返回defaultValue**。使用该方法程序员可以省去查询指定键值是否存在的麻烦．

需求；*假设有一个数字到对应英文单词的Map，输出4对应的英文单词，如果不存在则输出NoValue*

```
// 查询Map中指定的值，不存在时使用默认值
HashMap<Integer, String> map = new HashMap<>();
map.put(1, "one");
map.put(2, "two");
map.put(3, "three");
// Java7以及之前做法
if(map.containsKey(4)){ // 1
    System.out.println(map.get(4));
}else{
    System.out.println("NoValue");
}
// Java8使用Map.getOrDefault()
System.out.println(map.getOrDefault(4, "NoValue")); // 2
```

#####putIfAbsent()

该方法跟Lambda表达式没关系，但是很有用。方法签名为`V putIfAbsent(K key, V value)`，作用是只有在**不存在key值的映射或映射值为null时**，才将`value`指定的值放入到`Map`中，否则不对`Map`做更改．该方法将条件判断和赋值合二为一，使用起来更加方便．

#####remove()

我们都知道`Map`中有一个`remove(Object key)`方法，来根据指定`key`值删除`Map`中的映射关系；Java8新增了`remove(Object key, Object value)`方法，只有在当前`Map`中**key正好映射到value时**才删除该映射，否则什么也不做．

####replace()

在Java7及以前，要想替换`Map`中的映射关系可通过`put(K key, V value)`方法实现，该方法总是会用新值替换原来的值．为了更精确的控制替换行为，Java8在`Map`中加入了两个`replace()`方法，分别如下：

- `replace(K key, V value)`，只有在当前`Map`中**key的映射存在时**才用`value`去替换原来的值，否则什么也不做．
- `replace(K key, V oldValue, V newValue)`，只有在当前`Map`中**key的映射存在且等于oldValue时**才用`newValue`去替换原来的值，否则什么也不做．

#### replaceAll()

该方法签名为`replaceAll(BiFunction<? super K,? super V,? extends V> function)`，作用是对`Map`中的每个映射执行`function`指定的操作，并用`function`的执行结果替换原来的`value`，其中`BiFunction`是一个函数接口，里面有一个待实现方法`R apply(T t, U u)`．不要被如此多的函数接口吓到，因为使用的时候根本不需要知道他们的名字．

需求：*假设有一个数字到对应英文单词的Map，请将原来映射关系中的单词都转换成大写．*

Java7以及之前经典的代码如下：

```java
// Java7以及之前替换所有Map中所有映射关系
HashMap<Integer, String> map = new HashMap<>();
map.put(1, "one");
map.put(2, "two");
map.put(3, "three");
for(Map.Entry<Integer, String> entry : map.entrySet()){
    entry.setValue(entry.getValue().toUpperCase());
}
```

使用`replaceAll()`方法结合匿名内部类，实现如下：

```java
// 使用replaceAll()结合匿名内部类实现
HashMap<Integer, String> map = new HashMap<>();
map.put(1, "one");
map.put(2, "two");
map.put(3, "three");
map.replaceAll(new BiFunction<Integer, String, String>(){
    @Override
    public String apply(Integer k, String v){
        return v.toUpperCase();
    }
});
```

上述代码调用`replaceAll()`方法，并使用匿名内部类实现`BiFunction`接口。更进一步的，使用Lambda表达式实现如下：

```java
// 使用replaceAll()结合Lambda表达式实现
HashMap<Integer, String> map = new HashMap<>();
map.put(1, "one");
map.put(2, "two");
map.put(3, "three");
map.replaceAll((k, v) -> v.toUpperCase());
```

####merge()

该方法签名为`merge(K key, V value, BiFunction<? super V,? super V,? extends V> remappingFunction)`，作用是：

1. 如果`Map`中`key`对应的映射不存在或者为`null`，则将`value`（不能是`null`）关联到`key`上；
2. 否则执行`remappingFunction`，如果执行结果非`null`则用该结果跟`key`关联，否则在`Map`中删除`key`的映射．

参数中`BiFunction`函数接口前面已经介绍过，里面有一个待实现方法`R apply(T t, U u)`．

`merge()`方法虽然语义有些复杂，但该方法的用方式很明确，一个比较常见的场景是将新的错误信息拼接到原来的信息上，比如：

```
map.merge(key, newMsg, (v1, v2) -> v1+v2);
```

####compute()

该方法签名为`compute(K key, BiFunction<? super K,? super V,? extends V> remappingFunction)`，作用是把`remappingFunction`的计算结果关联到`key`上，如果计算结果为`null`，则在`Map`中删除`key`的映射．

要实现上述`merge()`方法中错误信息拼接的例子，使用`compute()`代码如下：

```
map.compute(key, (k,v) -> v==null ? newMsg : v.concat(newMsg));
```

####computeIfAbsent()

该方法签名为`V computeIfAbsent(K key, Function<? super K,? extends V> mappingFunction)`，作用是：只有在当前`Map`中**不存在key值的映射或映射值为null时**，才调用`mappingFunction`，并在`mappingFunction`执行结果非`null`时，将结果跟`key`关联．

`Function`是一个函数接口，里面有一个待实现方法`R apply(T t)`．

`computeIfAbsent()`常用来对`Map`的某个`key`值建立初始化映射．比如我们要实现一个多值映射，`Map`的定义可能是`Map<K,Set<V>>`，要向`Map`中放入新值，可通过如下代码实现：

```java
Map<Integer, Set<String>> map = new HashMap<>();
// Java7及以前的实现方式
if(map.containsKey(1)){
    map.get(1).add("one");
}else{
    Set<String> valueSet = new HashSet<String>();
    valueSet.add("one");
    map.put(1, valueSet);
}
// Java8的实现方式
map.computeIfAbsent(1, v -> new HashSet<String>()).add("yi");
```

使用`computeIfAbsent()`将条件判断和添加操作合二为一，使代码更加简洁．

### computeIfPresent()

该方法签名为`V computeIfPresent(K key, BiFunction<? super K,? super V,? extends V> remappingFunction)`，作用跟`computeIfAbsent()`相反，即，只有在当前`Map`中**存在key值的映射且非null时**，才调用`remappingFunction`，如果`remappingFunction`执行结果为`null`，则删除`key`的映射，否则使用该结果替换`key`原来的映射．

这个函数的功能跟如下代码是等效的：

```java
// Java7及以前跟computeIfPresent()等效的代码
if (map.get(key) != null) {
    V oldValue = map.get(key);
    V newValue = remappingFunction.apply(key, oldValue);
    if (newValue != null)
        map.put(key, newValue);
    else
        map.remove(key);
    return newValue;
}
return null;
```
