# 03 - Java Lambda表达式
为了可以更快、更清晰的编码，Java8引入了一些新的语言特性。凭借着被称为最重要的特性"Lambda Expressions"，Java打开了函数式编程的大门。Lambda表达式允许以最简单的方式实现和传递函数，而无需声明其它（匿名）类。

>注意：
Flink所有的Java API都支持使用lambda表达式，但是，无论什么时候lambda表达式使用泛型时，您都需要显示声明类型信息。

本文将会展示如何使用lambda表达式，以及目前存在什么限制。对于常规的Flink API介绍，可以查看编程手册。

## 样例和限制
下面的例子展示了如何使用lambda表达式来简单的使用一下map()函数。map() 函数的输入/输出参数类型不需要申明，因为Java编译器能够推测出来。

```
env.fromElements(1, 2, 3)
// returns the squared i
.map(i -> i*i)
.print();
```

Flink可以自动的从方法签名中获取结果类型信息，因为<B>OUT map(IN value)</B>中的IN类型是已经被注册的（传入参数的类型是确定的）但是返回的类型是不确定的。


不幸的是，拥有签名<B>void flatMap(IN value, Collector<OUT> out) </B>的flatMap()函数将会被Java编译器编译为<B>void flatMap(IN value, Collector out)</B>。这使得Flink无法自动的推测出输出类型信息。

Flink很有可能会抛出类似下面这种异常信息：

```
org.apache.flink.api.common.functions.InvalidTypesException: The generic type parameters of 'Collector' are missing.
    In many cases lambda methods don't provide enough information for automatic type extraction when Java generics are involved.
    An easy workaround is to use an (anonymous) class instead that implements the 'org.apache.flink.api.common.functions.FlatMapFunction' interface.
    Otherwise the type has to be specified explicitly using type information.
```

在这个例子中，需要显示的指定类型信息，否则输出类型将会被视为Object类型，这会导致无效的序列化。

```
import org.apache.flink.api.common.typeinfo.Types;
import org.apache.flink.api.java.DataSet;
import org.apache.flink.util.Collector;

DataSet<Integer> input = env.fromElements(1, 2, 3);

// collector type must be declared
input.flatMap((Integer number, Collector<String> out) -> {
    StringBuilder builder = new StringBuilder();
    for(int i = 0; i < number; i++) {
        builder.append("a");
        out.collect(builder.toString());
    }
})
// provide type information explicitly
.returns(Types.STRING)
// prints "a", "a", "aa", "a", "aa", "aaa"
.print();
```

使用返回类型包含泛型的map()函数时也会出现类似的问题。在下面的例子中，当方法签名为<B>Tuple2<Integer, Integer> map(Integer value)</B>的时候，将会被删除为Tuple2 map(Integer value)。

```
import org.apache.flink.api.common.functions.MapFunction;
import org.apache.flink.api.java.tuple.Tuple2;

env.fromElements(1, 2, 3)
    .map(i -> Tuple2.of(i, i))    // no information about fields of Tuple2
    .print();
```

通常，这类问题存在多种解决方式：

```
import org.apache.flink.api.common.typeinfo.Types;
import org.apache.flink.api.java.tuple.Tuple2;

// 方法一：调用".returns(...)"方法来明确返回类型
env.fromElements(1, 2, 3)
    .map(i -> Tuple2.of(i, i))
    .returns(Types.TUPLE(Types.INT, Types.INT))
    .print();

// 方法二：使用类来代替
env.fromElements(1, 2, 3)
    .map(new MyTuple2Mapper())
    .print();

public static class MyTuple2Mapper extends MapFunction<Integer, Tuple2<Integer, Integer>> {
    @Override
    public Tuple2<Integer, Integer> map(Integer i) {
        return Tuple2.of(i, i);
    }
}

//方法三：使用匿名类来代替
env.fromElements(1, 2, 3)
    .map(new MapFunction<Integer, Tuple2<Integer, Integer>> {
        @Override
        public Tuple2<Integer, Integer> map(Integer i) {
            return Tuple2.of(i, i);
        }
    })
    .print();

// 方法四：通过使用元组的子类来实现
env.fromElements(1, 2, 3)
    .map(i -> new DoubleTuple(i, i))
    .print();

public static class DoubleTuple extends Tuple2<Integer, Integer> {
    public DoubleTuple(int f0, int f1) {
        this.f0 = f0;
        this.f1 = f1;
    }
}
```





