# [Tuple 序列化](http://twitter.github.io/heron/docs/developers/serialization/)

Tuple 是 Heron 的核心数据类型，它支持很多[基本数据类型](../Topology-Writers/Heron-Data-Model.md#使用-tuple-using-tuples)，如 strings、integer、boolean 等。如果您需要，您也可以自定义数据类型，但请注意此时的序列化方式。

## Kryo

目前，Heron 使用 [Kryo](https://github.com/EsotericSoftware/kryo) 对 tuple 进行序列化和反序列化操作。在构建 tuple 后，您可以通过继承 Kryo 的抽象类 [`Serializer`](http://code.google.com/p/kryo/source/browse/trunk/src/com/esotericsoftware/kryo/Serializer.java) 自定义序列化器，帮助您完成序列化操作。可以参阅 [Kryo 文档](https://github.com/EsotericSoftware/kryo#serializers)了解更多信息。

## 注册 Serializer (Registering a Serializer)

在拥有一个自定义的 Kryo serializer 之后：

1. 请确保 Heron 在 classpath 中可访问该类。
2. 通过 `topology.kryo.register` 参数注册该类，例如：

```yaml
topology.kryo.register:
  - biz.acme.heron.datatypes.CustomType1 # This type will use the default FieldSerializer
  - biz.acme.heron.datatypes.CustomType2: com.example.heron.serialization.CustomSerializer
```

---

***笔者后记***

Heron 全面使用 Kryo 进行 tuple 序列化，在编写拓扑时，我们也可以通过 Config.registerSerialization() 方法注册序列化类。如，在 Kryo 注册 `LinkedHashMap`

```java
org.apache.storm.Config config = new Config();
config.registerSerialization(LinkedHashMap.class, MapSerializer.class);
```
