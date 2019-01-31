# 02 - Scala API扩展
为了保持Scala和Java API能够高度的统一，批处理和流处理的标准API将会忽略Scala中一些高级表达性的功能。

如果你希望能够使用完整的Scala特性，你可以选择使用隐式转换来增强Scala API的扩展。

为了能够使用所以可用的扩展，你只需要为DataSet API加入一个简单的导入：

```
import org.apache.flink.api.scala.extensions._
```

对于DataStream API则是：

```
import org.apache.flink.streaming.api.scala.extensions._
```

或者，你也可以导入单个你喜欢使用的扩展。

## Accept partial functions
通常，DataSet和DataStream API都不接收使用匿名模式匹配函数来解析元组、case class和集合，例如下面这些都不支持：

```
val data: DataSet[(Int, String, Double)] = // [...]
data.map {
  case (id, name, temperature) => // [...]
  // The previous line causes the following compilation error:
  // "The argument types of an anonymous function must be fully known. (SLS 8.5)"
}
```

此扩展在DataSet 和 DataStream Scala API中引入了新的方法，这些防范在扩展API中具有一一对应的关系。这些方法支持匿名模式匹配。

### DataSet API
<table class="table table-bordered">
  <thead>
    <tr>
      <th class="text-left" style="width: 20%">方法</th>
      <th class="text-left" style="width: 20%">原来的方法</th>
      <th class="text-center">示例</th>
    </tr>
  </thead>

  <tbody>
    <tr>
      <td><strong>mapWith</strong></td>
      <td><strong>map (DataSet)</strong></td>
      <td><code>
data.mapWith {<br/>
  case (_, value) => value.toString<br/>
}
</code>
      </td>
    </tr>
    <tr>
      <td><strong>mapPartitionWith</strong></td>
      <td><strong>mapPartition (DataSet)</strong></td>
      <td>
data.mapPartitionWith {
  case head #:: _ => head
}
      </td>
    </tr>
    <tr>
      <td><strong>flatMapWith</strong></td>
      <td><strong>flatMap (DataSet)</strong></td>
      <td>
data.flatMapWith {
  case (_, name, visitTimes) => visitTimes.map(name -> _)
}
      </td>
    </tr>
    <tr>
      <td><strong>filterWith</strong></td>
      <td><strong>filter (DataSet)</strong></td>
      <td>

data.filterWith {
  case Train(_, isOnTime) => isOnTime
}

      </td>
    </tr>
    <tr>
      <td><strong>reduceWith</strong></td>
      <td><strong>reduce (DataSet, GroupedDataSet)</strong></td>
      <td>

data.reduceWith {
  case ((_, amount1), (_, amount2)) => amount1 + amount2
}

      </td>
    </tr>
    <tr>
      <td><strong>reduceGroupWith</strong></td>
      <td><strong>reduceGroup (GroupedDataSet)</strong></td>
      <td>

data.reduceGroupWith {
  case id #:: value #:: _ => id -> value
}

      </td>
    </tr>
    <tr>
      <td><strong>groupingBy</strong></td>
      <td><strong>groupBy (DataSet)</strong></td>
      <td>

data.groupingBy {
  case (id, _, _) => id
}

      </td>
    </tr>
    <tr>
      <td><strong>sortGroupWith</strong></td>
      <td><strong>sortGroup (GroupedDataSet)</strong></td>
      <td>

grouped.sortGroupWith(Order.ASCENDING) {
  case House(_, value) => value
}

      </td>
    </tr>
    <tr>
      <td><strong>combineGroupWith</strong></td>
      <td><strong>combineGroup (GroupedDataSet)</strong></td>
      <td>

grouped.combineGroupWith {
  case header #:: amounts => amounts.sum
}

      </td>
    <tr>
      <td><strong>projecting</strong></td>
      <td><strong>apply (JoinDataSet, CrossDataSet)</strong></td>
      <td>

data1.join(data2).
  whereClause(case (pk, _) => pk).
  isEqualTo(case (_, fk) => fk).
  projecting {
    case ((pk, tx), (products, fk)) => tx -> products
  }

data1.cross(data2).projecting {
  case ((a, _), (_, b) => a -> b
}

      </td>
    </tr>
    <tr>
      <td><strong>projecting</strong></td>
      <td><strong>apply (CoGroupDataSet)</strong></td>
      <td>

data1.coGroup(data2).
  whereClause(case (pk, _) => pk).
  isEqualTo(case (_, fk) => fk).
  projecting {
    case (head1 #:: _, head2 #:: _) => head1 -> head2
  }
}

      </td>
    </tr>
    </tr>
  </tbody>
</table>

### DataStream API
<table class="table table-bordered">
  <thead>
    <tr>
      <th class="text-left" style="width: 20%">方法</th>
      <th class="text-left" style="width: 20%">原来的方法</th>
      <th class="text-center">示例</th>
    </tr>
  </thead>

  <tbody>
    <tr>
      <td><strong>mapWith</strong></td>
      <td><strong>map (DataStream)</strong></td>
      <td>

data.mapWith {
  case (_, value) => value.toString
}

      </td>
    </tr>
    <tr>
      <td><strong>mapPartitionWith</strong></td>
      <td><strong>mapPartition (DataStream)</strong></td>
      <td>

data.mapPartitionWith {
  case head #:: _ => head
}

      </td>
    </tr>
    <tr>
      <td><strong>flatMapWith</strong></td>
      <td><strong>flatMap (DataStream)</strong></td>
      <td>

data.flatMapWith {
  case (_, name, visits) => visits.map(name -> _)
}

      </td>
    </tr>
    <tr>
      <td><strong>filterWith</strong></td>
      <td><strong>filter (DataStream)</strong></td>
      <td>

data.filterWith {
  case Train(_, isOnTime) => isOnTime
}

      </td>
    </tr>
    <tr>
      <td><strong>keyingBy</strong></td>
      <td><strong>keyBy (DataStream)</strong></td>
      <td>

data.keyingBy {
  case (id, _, _) => id
}

      </td>
    </tr>
    <tr>
      <td><strong>mapWith</strong></td>
      <td><strong>map (ConnectedDataStream)</strong></td>
      <td>

data.mapWith(
  map1 = case (_, value) => value.toString,
  map2 = case (_, _, value, _) => value + 1
)

      </td>
    </tr>
    <tr>
      <td><strong>flatMapWith</strong></td>
      <td><strong>flatMap (ConnectedDataStream)</strong></td>
      <td>

data.flatMapWith(
  flatMap1 = case (_, json) => parse(json),
  flatMap2 = case (_, _, json, _) => parse(json)
)

      </td>
    </tr>
    <tr>
      <td><strong>keyingBy</strong></td>
      <td><strong>keyBy (ConnectedDataStream)</strong></td>
      <td>

data.keyingBy(
  key1 = case (_, timestamp) => timestamp,
  key2 = case (id, _, _) => id
)

      </td>
    </tr>
    <tr>
      <td><strong>reduceWith</strong></td>
      <td><strong>reduce (KeyedDataStream, WindowedDataStream)</strong></td>
      <td>

data.reduceWith {
  case ((_, sum1), (_, sum2) => sum1 + sum2
}

      </td>
    </tr>
    <tr>
      <td><strong>foldWith</strong></td>
      <td><strong>fold (KeyedDataStream, WindowedDataStream)</strong></td>
      <td>

data.foldWith(User(bought = 0)) {
  case (User(b), (_, items)) => User(b + items.size)
}

      </td>
    </tr>
    <tr>
      <td><strong>applyWith</strong></td>
      <td><strong>apply (WindowedDataStream)</strong></td>
      <td>

data.applyWith(0)(
  foldFunction = case (sum, amount) => sum + amount
  windowFunction = case (k, w, sum) => // [...]
)

      </td>
    </tr>
    <tr>
      <td><strong>projecting</strong></td>
      <td><strong>apply (JoinedDataStream)</strong></td>
      <td>

data1.join(data2).
  whereClause(case (pk, _) => pk).
  isEqualTo(case (_, fk) => fk).
  projecting {
    case ((pk, tx), (products, fk)) => tx -> products
  }

      </td>
    </tr>
  </tbody>
</table>

更多关于每个方法的语义信息，请查看DataSet 和 DataStream API API文档。

为了能够单独使用指定的语义，你可以添加下面的这些import：

- DataSet 扩展

```
import org.apache.flink.api.scala.extensions.acceptPartialFunctions
```

- DataStream 扩展

```
import org.apache.flink.streaming.api.scala.extensions.acceptPartialFunctions
```

下面这段脚本是如何在DataSet API中使用这些扩展的最小示例：

```
object Main {
  import org.apache.flink.api.scala.extensions._
  case class Point(x: Double, y: Double)
  def main(args: Array[String]): Unit = {
    val env = ExecutionEnvironment.getExecutionEnvironment
    val ds = env.fromElements(Point(1, 2), Point(3, 4), Point(5, 6))
    ds.filterWith {
      case Point(x, _) => x > 1
    }.reduceWith {
      case (Point(x1, y1), (Point(x2, y2))) => Point(x1 + y1, x2 + y2)
    }.mapWith {
      case Point(x, y) => (x, y)
    }.flatMapWith {
      case (x, y) => Seq("x" -> x, "y" -> y)
    }.groupingBy {
      case (id, value) => id
    }
  }
}
```



