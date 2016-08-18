### Java8-Stream
流是Java8 API的新成员，它允许你以声明方式处理数据集合，你可以把它看成遍历数据集的高级迭代器；此外，流还可以透明地的并行处理，让你的代码性能更好。

### 构建流
1. 由值创建流
使用静态方法Stream.of，通过显式创建一个流。它可以接受任意数量的参数。
```
Stream<String> stream = Stream.of("java8","name","passWord");
```
2. 由数组创建流
```
int[] numbers = new int[]{1,2,6,4,8,3,10};
```
3. 由文件生成流
```
try (Stream<String> lines = Files.lines(Paths.get("Test.java"), Charset.defaultCharset())) {

        } catch (IOException e) {
            e.printStackTrace();
        }
```
4. 由函数生成流：创建无线流
Stream API提供了两个静态方法来从函数生成流：Stream.iterate和Stream.generate。这两个操作可以创建所谓的无限流。一般来说应该使用limit(n)对这种流加以限制。
	1. 迭代
    ```
    Stream<Integer> stream = Stream.iterate(0,n->n+2).limit(10);
	```
	2. 生成
	```
    Stream<Double> stream = Stream.generate(Math::random).limit(10);
	```

   相比较而言，interate是依次生成一系列值，生成的时候依赖于前值；比如例子中的生成偶数，每次计算对上一个值应用相加函数；而generate不是依次对每个新生成的值应用函数，它接受一个函数Supplier<T>产生新的值（Supplier<T>产生一个对象）。

5. 数值流特有的生成一定范围数值的流
```
IntStream stream = IntStream.range(1,100);
IntStream stream = IntStream.rangeClose(1,100);
```
这两个方法都接收起始值，但range不接收尾值，而rangeClose接收尾值。

### 流操作
有了数据流，我们可以对流进行各种操作；操作前，你需要了解些Lambda和方法引用的内容。
#### Lambda
Lambda表达式有三个部分：
* 参数列表
* 箭头
* Lambda主体

Lambda表达式实际上表达了函数式接口（只定义一个抽象方法的接口，Java8中为了更好的辨别此类接口使用了@FunctionalInterface的注解）的一个实例，相比匿名类来说，更简单更高效。
比如我们启动一个线程
```
new Thread(() -> System.out.println("Hello World")).start();
```
Thread接收一个Runnable的实例，这里简单的写成了`() -> System.out.println("Hello World")`,这个Lambda表达式就表达了一个Runnable的实例，事实上`Runnable r = ()->System.out.println("Hello World");`,两者是相等的。

注意：Lambda表达式的签名要和函数式接口的抽象方法签名一致

#### 方法引用
方法引用可以让你重复使用现有的方法定义，并像Lambda一样传递它们；方法引用可以被看做仅仅调用特定方法的Lambda的一种快捷键写法。如果一个Lambda代表的只是“直接调用这个方法”，那最好还是用名称来调用它。
比如上面的示例改成
```
new Thread(() -> System.out::println).start();
```
但是输出为空；常见的对象比较、对象转化我们都可以用这种方式来简化书写；下面的示例会有具体体现。

#### 中间操作
中间操作，是对流的操作处理，产生的也为一个流；重要的是如果流没有触发中断操作，中间操作不会执行任何处理。为了更好的演示，下面我们创建一个类，以便于具体场景下解释
```
class Dish {
        private final String name;// 名称
        private final boolean vegetarian;// 是否素食
        private final int calories;// 热量
        private final Type type;// 类型（肉、鱼、其他）

        public Dish(String name, boolean vegetarian, int calories, Type type) {
            this.name = name;
            this.vegetarian = vegetarian;
            this.calories = calories;
            this.type = type;
        }

        public String getName() {
            return name;
        }

        public boolean isVegetarian() {
            return vegetarian;
        }

        public int getCalories() {
            return calories;
        }

        public Type getType() {
            return type;
        }

        public enum Type {MEAT, FISH, OTHER}
    }
```
创建一个菜单
```
List<Dish> menu = Arrays.asList(
                new Dish("duck", false, 800, Dish.Type.MEAT),// 鸭肉
                new Dish("beef", false, 700, Dish.Type.MEAT),// 牛肉
                new Dish("chicken", false, 400, Dish.Type.MEAT),// 鸡肉
                new Dish("french fries", true, 530, Dish.Type.OTHER),// 炸薯条
                new Dish("rice", true, 350, Dish.Type.OTHER),// 米
                new Dish("season fruit", true, 120, Dish.Type.OTHER),// 季节水果
                new Dish("pizza", true, 550, Dish.Type.OTHER),// 比萨饼
                new Dish("prawns", false, 300, Dish.Type.FISH),// 虾
                new Dish("salmon", false, 450, Dish.Type.FISH)// 鲑鱼
        );
```
##### filter
该操作接受一个谓词（一个返回boolean的函数）作为参数，并返回一个包含所有符合谓词的元素的流。
比如获取素食
```
Stream<Dish> stream = menu.stream().filter(Dish::isVegetarian);
```
这里`Dish::isVegetarian`就是Lambda`a -> a.isVegetarian()`的简写，`a`由Java的类型推导会自动识别为`Dish`类型。
##### distinct
该操作会返回一个元素各异（根据流所生成元素的hashCode/equals方法实现）的流。
比如下面获取不重复的数值流
```
IntStream intStream =IntStream.of(1,2,1,2,4,5,6).distinct();
```
##### skip
该操作返回一个扔掉了前n个元素的流
比如菜单中出去前两个的菜品
```
Stream<Dish> stream = menu.stream().skip(2);
```
##### limit
该操作会返回一个不超过给定长度的流。
比如前5个菜品
```
Stream<Dish> stream = menu.stream().limit(5);
```
##### map
映射操作，即流接受一个函数作为参数，这个函数会应用到每个元素上，并将其映射成一个新的元素。
比如获取全部的菜名流
```
Stream<String> stream = menu.stream().map(Dish::getName);
```
##### flatMap
此方法把一个流中的每个值都换成另一个流，然后把所有的流链接起来成为一个流。
比如获取字符串数组的全部字母流
```
Arrays.stream(new String[]{"abc","defg"}).map(a->a.split(""));
```
此操作将每个元素进行`sprit`操作获取到Stream<String[]>，要想获取Stream<String>则需要进一步转换
```
Stream<String> stringStream = Arrays.stream(new String[]{"abc","defg"}).map(a->a.split("")).flatMap(Arrays::stream);
```
##### sorted
此方法可以将流按照一定的规则进行元素排列，参数为空时按照对象的默认比较器进行排序，同时你也可以传入自定义的比较器来定制排序规则。
比如按照热量对菜品进行排序
```
Stream<Dish> stream = menu.stream().sorted((a,b)-> a.getCalories()>b.getCalories()?1:0);

```
也可以简写为
```
Stream<Dish> stream = menu.stream().sorted(Comparator.comparing(Dish::getCalories));
```
#### 终端操作
终端操作消费一个流，获取一个具体的对象结果。
##### anyMatch/noneMatch/allMatch
此三个方法为判断流中元素是否匹配给定的谓词，返回一个boolean值：anyMathch任意匹配；noneMathch全不匹配;allMathch都匹配。
如下是否有素食
```
boolean flag = menu.stream().anyMatch(Dish::isVegetarian);
```
##### findAny/findFirst
返回当前流的任意/第一个元素，返回一个`Optional<T>`对象
```
Optional optional = menu.stream().findFirst();
```
`Otpional<T>`类是一个容器类，代表一个值存在或者不存在；Java8引入`Optional<T>`，这样就不会出现NUll的问题了，比如上面如果菜单为空，如何获取第一个元素。
`Optional<T>`提供了以下方法让你处理值不存在的情形：
* `isPresent()`将在`Optaional`包含值的时候返回为true否则false
* `isPresent(Consumer<T> block)`会在值存在的时候指定给定代码
* `T get()`会在值存在时返回值，否则抛出NoSuchElement异常
* `T orElseGet(Object other)`会在值不存在时返回一个默认值
* `T orElseGet(Supplier other)`会在值不存在时生成一个默认值

##### foreach
此方法对流中的每个元素应用一个`Comsumer<T>`的函数，返回一个void类型；常常我们用来输出。
如下打印全部的菜单名称
```
menu.stream().forEach(a -> System.out.println(a.getName()));
```
##### collect
将流 转换成其他形式，你可以把collect看做能够接受各种方案作为参数，并将流中的元素累积成一个汇总结果的操作。
比如将全部菜品收集到List中
```
menu.stream().map(Dish::getName).collect(Collectors.toList());
```
collect实际上也是一种归约操作，接收Collecttors参数可以实现更高级的汇总，比如实现菜单按照素食非素质归类，按照热量范畴归类等等。见 [Java8-Collectors]()


##### reduce
reduce即归约操作，用来表达更复杂的查询，比如“计算菜单中的总热量”
```
menu.stream().map(Dish::getCalories).reduce(0,Integer::sum);
```
reduce接收两个参数：
1 一个初始值
2 一个`BinaryOperator<T>`来将两个元素结合起来产生一个新值，这里我们使用的是lambda (a,b) -> a+b，用方法引用简写了；
reduce还有一个变体，不接收初始值，返回一个`Optional`对象，如
```
menu.stream().map(Dish::getCalories).reduce(Integer::sum);
```
但是这种写法有性能问题，它有一个暗含的装箱成本，每个Integer必须拆分成一个原始类型再进行求和。Stream API提供了原始类型流特化来专门支持处理数值流的方法。见下。

##### 原始类型特化
Java8提供了3个原始类型流接口：IntStream/DoubleStream/LongStream，分别将流中的元素特化为int/long/double，从而避免暗含的装箱成本。每个接口都提供了常用的规约方法，比如max/sum/min等。
比如上面实例可以写成
```
menu.stream().mapToInt(Dish::getCalories).sum();
```
注意：sum的默认值为0，如果为max()操作的话，将返回`OptionalInt`。

如果要将数值流转换回对象流，则需要`box()`操作，比如
```
Stream<Integer> stream = menu.stream().mapToInt(Dish::getCalories).boxed();
```

##### count
count即计数，计算流中元素的数量；实际上通过转化LongStream进行reduce操作获取的。
比如计算菜品总数
```
menu.stream().count();
```





