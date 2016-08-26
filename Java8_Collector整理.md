### 收集器
Stream中的collect可以将流中的元素累积成一个汇总结果，使用收集器可以简洁而灵活地定义collect用来生成结果集合的标准。更具体的来说，对流调用collect方法将对流中的元素触发一个归约操作，由Collector来参数化。一般来说，Collector会对元素应用一个转换函数（很多时候是不体现任何效果的恒等转换，例如toList()），并将结果累积在一个数据结构中，从而产生这一过程的最终输出。   
Collector主要提供3大功能：   

* 将流元素归约和汇总成为一个值   
* 元素分组   
* 元素分区   

###  元素归约和汇总
#### toList
*toList把流中的项目收集到一个List*   
使用示例获取菜单的全部列表：
```
menu.stream().map(Dish::getName).collect(Collectors.toList());
```

#### toSet
*toSet把流中所有项目收集到一个Set，删除重复项*
使用示例获取全部的菜品种类：
```
menu.stream().map(Dish::getType).collect(Collectors.toSet());
```

#### toCollect
*toCollect把流中所有项目收集到给定的供应源创建的集合*
使用示例获取菜单的全部集合：
```        menu.stream().map(Dish::getName).collect(Collectors.toCollection(TreeSet::new));
```

#### counting
*counting计算流中的元素的个数*   
使用示例获取菜单的菜品数量：
```
menu.stream().map(Dish::getName).distinct().collect(Collectors.counting());
```

##### minBy/maxBy
*查询流中的最大元素和最小元素*
使用示例获取最大热量/最小热量的菜品
```
menu.stream().collect(Collectors.maxBy(Comparator.comparing(Dish::getCalories)));
```

##### summingInt/averagingInt
*计算数值流中的元素总和/平均值*
使用示例获取全部菜品的热量总和/平均值
```
menu.stream().collect(Collectors.summingInt(Dish::getCalories));        menu.stream().collect(Collectors.averagingInt(Dish::getCalories));
```
同理还有sumingDouble/averagingDouble、sumingLong/averagingLong

##### summarizingInt
*sumarizingInt计算数值流中的元素总和/平均值/最大值/最小值*
使用示例获取菜品的热量总和/平均值/最大值/最小值/个数
```       
IntSummaryStatistics summaryStatistics = menu.stream().collect(Collectors.summarizingInt(Dish::getCalories));
```
通过`summaryStatistics.getCount(）`来获取具体的值。     

##### joining
*joining连接对流中每个项目调用toString方法所生成的字符串*
```       menu.stream().map(Dish::getName).collect(Collectors.joining(","));
```

##### reducing 
*reducing从一个作为累加器的初始值开始，利用BinaryOperator与流中的元素逐个结合，从而将流归约为单个值。*事实上，我们讨论的所有收集器都是一个可以用reducing工厂方法定义的归约过程的特殊情况而已。Collectors.reducing工厂方法是所有这些特殊情况的一般化。   
reducing需要3个参数：

* 第一个参数是归约操作的起始值，也是流中没有元素时的返回值
* 第二个参数是转换函数，即获取需要操作的对象属性
* 第三个参数是一个BinaryOperator，将两个项目累积成一个同类型的值。

使用实例，获取热量最高的菜品：

```    menu.stream().collect(Collectors.reducing(null,Dish::getCalories,(a,d)->a>d?a:d));
menu.stream().collect(Collectors.reducing(null,(d1,d2)->d1.getCalories()>d2.getCalories()?d1:d2));

```
如果不设初始值的话，则将返回一个`Optional<T>`的对象

```
menu.stream().collect(Collectors.reducing((d1,d2)->d1.getCalories()>d2.getCalories()?d1:d2));
```

### 分组
#### groupingBy
*根据项目的一个属性的值对流中的项目作分组，将属性值作为结果Map的键*
使用示例，根据菜品的类型分组：
```
        Map<Dish.Type,List<Dish>> map = menu.stream().collect(Collectors.groupingBy(Dish::getType));
```
结果为：
```
{FISH=[prawns, salmon], OTHER=[french fies, rice, season fruit, pizza], MEAT=[pork, beef, chicken]}
```
根据菜品热量分组：
```
Map<CaloricLevel,List<Dish>> map2 = menu.stream().collect(
          groupingBy(dish->{
              if(dish.getCalories()<=400) return CaloricLevel.DIET;
              else if(dish.getCalories() <=700) return CaloricLevel.NORMAL;
              return CaloricLevel.FAT;
          })
        );
```
结果为：
```
{FAT=[pork], NORMAL=[beef, french fies, pizza, prawns, salmon], DIET=[chicken, rice, season fruit]}
```

##### 多集分组
groupingBy工厂方法创建的收集器，除了普通的分类函数之外，还可以接受collector类型的第二个参数。
使用示例，根据菜品类型分类，再根据热量分类
```
Map<Dish.Type, Map<CaloricLevel,List<Dish>>> map2 = menu.stream().collect(
                groupingBy(Dish::getType,
                        groupingBy(dish -> {
                            if (dish.getCalories() <= 400) return CaloricLevel.DIET;
                            else if (dish.getCalories() <= 700) return CaloricLevel.NORMAL;
                            return CaloricLevel.FAT;
                        }))
        );
```
结果为:
```
{FISH={NORMAL=[prawns, salmon]}, OTHER={DIET=[rice, season fruit], NORMAL=[french fies, pizza]}, MEAT={DIET=[chicken], FAT=[pork], NORMAL=[beef]}}
```

当然第二个收集器也可以为其他，比如counting计数这样我们就可以获取不同类型的菜品数量，也可以为maxBy/minBy，即获取不同类型的菜品的最大热量、最小热量；但这样会产生一些问题，比如maxBy/minBy时，会生成`Optional<T>`，但这对我们并没有什么用处。解决见下。

#### collectingAndThen
*包裹一个收集器，对其结果应用转换函数*
使用示例，根据菜品类型分类，获取最大的热量菜
```
Map<Dish.Type, Dish> map3 = menu.stream().
                collect(Collectors.groupingBy(Dish::getType, collectingAndThen(
                        maxBy(Comparator.comparing(Dish::getCalories)), Optional::get
                )));
```

#### mapping
与groupingBy联合使用的另一个收集器是mapping，这个方法接收两个参数：一个函数对流中的元素做变换，另一个则将变换的结果对象收集起来。
使用示例，根据菜品分类，获取热量类型
```
Map<Dish.Type, Set<CaloricLevel>> map4 = menu.stream().collect(
                groupingBy(Dish::getType,mapping(dish -> {
                            if (dish.getCalories() <= 400) return CaloricLevel.DIET;
                            else if (dish.getCalories() <= 700) return CaloricLevel.NORMAL;
                            return CaloricLevel.FAT;
                        },toSet())));
```
结果为：
```
{FISH=[NORMAL], OTHER=[DIET, NORMAL], MEAT=[DIET, FAT, NORMAL]}
```

#### 分区
##### partitioningBy
分区是分组的特殊情况：由一个谓词（返回一个boolean的函数）作为分类函数；这意味着它最多分两组-true一组，false一组。
使用示例，根据是否素食分类
```
Map<Boolean,List<Dish>> map = menu.stream().collect(Collectors.partitioningBy(Dish::isVegetarian));
```
结果为：
```
{false=[duck, beef, chicken, prawns, salmon], true=[french fries, rice, season fruit, pizza]}
```
partitioningBy同样可以接收一个收集器的参数
比如按照素食分类再按照菜品类型分类
```
Map<Boolean,Map<Dish.Type,List<Dish>>> map = menu.stream().collect(Collectors.partitioningBy(Dish::isVegetarian,groupingBy(Dish::getType)));
```
当然亦可以接收其他的收集器比如count/maxBy等等，用法跟groupingBy一致

### 自定义收集器

#### 实现Collector接口方式
我们可以通过Collector接口提供自己的实现，从而自由地创建自定义归约操作。
首先我们看看Collector接口的定义
```
public interface Collector<T, A, R> {// T是要收集的项目的泛型A是累加器的类型，累加器是在收集过程中用于累积部分结果的对象，R是收集操作得到的对象的类型
    Supplier<A> supplier();// 建立新的结果容器

    BiConsumer<A, T> accumulator();// 将元素添加到容器

    Function<A, R> finisher();// 对结果应用最终转换

    BinaryOperator<A> combiner();// combiner方法会返回一个供归约操作使用的函数，它定义了对流的各个字部分进行并行处理时，各个子部分归约所得的累加器要如何合并

    Set<Characteristics> characteristics();// 返回一个不可变的集合，定义了收集器的行为-尤其是关于流是否可以并行归约 
	// 1，UNORDERED 归约结果不受流中项目的遍历和累积顺序的影响  
	// 2，CONCURRENT accumulator函数可以从多个线程同时调用，且该收集器可以并行归约流。如果收集器没有标为UNORDERED，那它们尽在用于无序数据源时才可以并行归约 
	// 3，IDENTITY_FINISH 这表明完成器方法返回的函数是一个恒等函数，可以跳过。这种情况下，累加器对象将会直接用作归约过程的最终结果。
```

接下来，我们定义一个收集器，把菜单中的素食和非素质收集起来（用分组函数可以很好的实现）
```
public class Container {// 自定义容器
    private List<Dish> ok = new ArrayList<>();// 素食集合
    private List<Dish> no = new ArrayList<>();// 非素食集合

    public Container() {
    }

    public void addOk(Dish value) {
        ok.add(value);
    }

    public void addNo(Dish value) {
        no.add(value);
    }

    public List<Dish> getOk() {
        return ok;
    }

    public List<Dish> getNo() {
        return no;
    }
}
```
```
// 自定义收集器
public class ToMyCollector implements Collector<Dish, Container, Container> {
    @Override
    public Supplier<Container> supplier() {// 创建收集器
        return Container::new;
    }

    @Override
    public BiConsumer<Container, Dish> accumulator() {// 根据Dish的IsVegetarian添加到不同的集合
        return (c, t) -> {
            if (t.isVegetarian()) c.addOk(t);
            c.addNo(t);
        };
    }

    @Override
    public BinaryOperator<Container> combiner() {// 并行时小集合并入第一个集合中
        return (container1, container2) -> {
            container1.getNo().addAll(container2.getNo());
            container1.getOk().addAll(container2.getOk());
            return container1;
        };
    }

    @Override
    public Function<Container, Container> finisher() {// 收集好的数据不需要转换
        return Function.identity();
    }

    @Override
    public Set<Characteristics> characteristics() {// 恒等且可以多线程调用
        return Collections.unmodifiableSet(EnumSet.of(
                Characteristics.IDENTITY_FINISH, Characteristics.CONCURRENT
        ));
    }
}
```
最终结果：
```
Container c = menu.stream().collect(new ToMyCollector());
        System.out.println(c.getNo());
        System.out.println(c.getOk());
输出:
[duck, beef, chicken, french fries, rice, season fruit, pizza, prawns, salmon]
[french fries, rice, season fruit, pizza]
```
#### collect重载方法方式
```
 menu.stream().collect(Container::new, (Container c, Dish t1) -> {
            if (t1.isVegetarian()) c.getOk().add(t1);
            c.getNo().add(t1);
        }, (Container c1, Container c2) -> {
            c1.getOk().addAll(c2.getOk());
            c1.getNo().addAll(c2.getNo());
        });
```
这种写法比较简洁和紧凑，但是不易读。另外不能传Characteristics参数，永远是一个IDENTITY_FINISH和CONCURRENT但并非UNORDERED的收集器。


### 






