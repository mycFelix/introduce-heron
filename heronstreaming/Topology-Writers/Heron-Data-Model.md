# [Heron 数据模型](http://twitter.github.io/heron/docs/developers/data-model/)

Tuple 是 Heron 的核心数据类型。tuple 在 Heron 拓扑中起到关键桥梁作用，[spout](../Heron-Concepts/Heron-Topology.md#spouts) 负责生成 tuple，[bolt](../Heron-Concepts/Heron-Topology.md#bolts) 负责处理 tuple。

Heron 提供了 [Tuple API](http://twitter.github.io/heron/api/com/twitter/heron/api/tuple/Tuple.html) 用于具体处理数据。它可以存储各种类型的数据，并提供了下标和名称两种访问方式。

## 使用 Tuple (Using Tuples)

可以参考 [Tuple Javadoc](http://twitter.github.io/heron/api/com/twitter/heron/api/tuple/Tuple.html) 页面了解相关 API。

### 通过下标获取基础数据类型值 (Accessing Primitive Types By Index)

Heron `Tuple` 支持多种 Java 主要数据类型，如 `String`、`Booleans`、`byte arrays`等等。可以通过 [`getString`](http://twitter.github.io/heron/api/com/twitter/heron/api/tuple/Tuple.html#getString-int-) 方法，传递下标获取对应的值。如果返回为 `null` 则说明该下标所对应的值不存在。可以参考 Javadoc 了解更多同类型函数。

### 通过字段获取基础数据类型值 (Accessing Primitive Types By Field)

除了下标方式，我们还可以通过 tuple 字段名来获取值信息。[`getStringByField`](http://twitter.github.io/heron/api/com/twitter/heron/api/tuple/Tuple.html#getStringByField-java.lang.String-) 方法，传递字段名获取对应的值。如果返回为 `null` 则说明该字段名所对应的值不存在。可以参考 Javadoc 了解更多同类型函数。

### 使用 Object 类型获取数据 (Using Non-primitive Types)

除了基础数据类型，您还可以通过 `Obejct` 来获取 `Tuple` 中的数据值。您同样也可以用下标和字段名这两种方式来获取相关值。如果返回为 `null` 则说明相关值不存在。

* [`getValue`](http://twitter.github.io/heron/api/com/twitter/heron/api/tuple/Tuple.html#getValue-int-)
* [`getValueByField`](http://twitter.github.io/heron/api/com/twitter/heron/api/tuple/Tuple.html#getValueByField-java.lang.String-)

也可以调用 [`getValues`](http://twitter.github.io/heron/api/com/twitter/heron/api/tuple/Tuple.html#getValues--) 方法，以 Java [List](https://docs.oracle.com/javase/8/docs/api/java/util/List.html) 形式返回 `Tuple` 中的所有数据

### 自定义类型 (User-defined Types)

当然，您也可以自定义 Tuple 中的数据类型。需要注意的是，请您确保自定义类型的[序列化方式(custom serializer)](http://twitter.github.io/heron/docs/developers/serialization/)，如下代码片段展示了一个自定义的 `Tweet` 类型：

```java
public void execute(Tuple input) {
    // The following return null if no value is present or throws a
    // ClassCastException if type casting fails:
    Tweet tweet = (Tweet) input.getValue(0);
    List<Tweet> allTweets = input.getValues();
}
```

### 使用字段 (Fields)

调用 `getFields` 方法可以返回 tuple 中的所有[`字段`](http://heronproject.github.io/topology-api/com/twitter/heron/api/tuple/Fields)。

### 其他方法

以上是我们在开发过程中可能会经常使用的方法，更多方法请参阅 Heron Tuple [Javadoc](http://twitter.github.io/heron/api/com/twitter/heron/api/tuple/Tuple.html)

## 字段 (Fields)

如前所述，可以通过 List 形式返回 Tuple 中的所有数据。反过来，您也可传递 List 来初始化 [`Fields`](http://twitter.github.io/heron/api/com/twitter/heron/api/tuple/Fields.html) 数据。代码片段如下：

```java
// Using varargs
Fields fruits = new Fields("apple", "orange", "banana");

// Using a list of strings
List<String> fruitNames = new LinkedList<String>();
fruitNames.add("apple");
// Add "orange" and "banana" as well
Fields fruits = new Fields(fruitNames);
```

你可以在 tuple 中直接使用该对象，如：

```java
public void execute(Tuple input) {
    List<Object> values = input.select(fruits);
    for (Object value : values) {
        System.out.println(value);
    }
}
```
