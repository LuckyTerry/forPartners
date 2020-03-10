# 编写自己的Serializers

创建Serializer最简单直接的方法是直接在类上编写注解@Serializable：

```kotlin
@Serializable
class MyData(val s: String)
```

在这种情况下，编译器插件将为您生成：

- .serializer()在伴随对象上获取序列化器的方法。如果您的类是泛型类，则此方法将具有参数KSerializer<T1>, KSerializer<T2>...，其中T1, T2- 您的泛型类型参数。

- 您的类中的特殊嵌套对象，它实现 KSerializer<MyData>

- 接口KSerialSaver和KSerialLoader的方法save，load以及update

- KSerializer的描述属性serialClassDesc

## 定制

如果要自定义类的表示形式，在大多数情况下，您需要编写自己的方法save和load方法。update方法有默认的实现throw UpdateNotSupportedException(serialClassDesc.name)。串行描述符属性通常用于这些方法的生成版本，因此您可能不需要它。

您可以直接在伴随对象上编写方法，使用它进行注释@Serializer(forClass = ...)，序列化插件将其视为默认Serializer。（注意你仍然需要应用@Serializable注释，因为我们需要定义serialClassDesc即使我们不使用它）

```kotlin
@Serializable
class MyData(val s: String) {

    @Serializer(forClass = MyData::class)
    companion object /*: KSerializer<MyData> can be omitted or not, if you like*/{
        override fun save(output: KOutput, obj: MyData) {
            output.writeStringValue(HexConverter.printHexBinary(obj.s.toByteArray()))
        }

        override fun load(input: KInput): MyData {
            return MyData(String(HexConverter.parseHexBinary(input.readStringValue())))
        }
    }
}
```

注意：这种方法对于泛型类还没有用。

## 库类的外部Serializer

如果您无法修改类的源代码，则上述方法将无法工作 - 例如，它是Kotlin/Java库类。如果它是Kotlin类，你可以让插件知道你想要从object创建序列化器：

```kotlin
// imagine that MyData is a library class
// if you have generic class, you should use `class` instead of `object`
@Serializer(forClass = MyData::class)
object DataSerializer {}
```

这称为外部序列化并强加某些限制 - 类应该只有主构造函数的vals / vars和类体var属性（你可以在docs中学到更多）

与第一个示例一样，您可以通过覆盖save和load方法自定义过程。

如果它是Java类，事情会变得更复杂：因为Java没有主构造函数的概念，插件不知道它可以采用哪些属性。对于Java类，您始终应该覆盖save/ load方法。您仍然可以使用@Serializer(forClass = ...)生成空的串行描述符。例如，让我们编写序列化器java.util.Date：

```kotlin
@Serializer(forClass = Date::class)
object DateSerializer: KSerializer<Date> {
    private val df: DateFormat = SimpleDateFormat("dd/MM/yyyy HH:mm:ss.SSS")

    override fun save(output: KOutput, obj: Date) {
        output.writeStringValue(df.format(obj))
    }

    override fun load(input: KInput): Date {
        return df.parse(input.readStringValue())
    }
}
```

如果您的类具有泛型类型参数，则它不应该是对象。它必须是具有可见主构造函数的类，其参数为KSerializer<T0>, KSerializer<T1>, etc..- 类的每个类型参数一个。

## JS注意

由于存在问题，现在不可能@Serializer(forClass=XXX::class)在JavaScript中编写一种类型的注释。不幸的是，你必须自己实现所有方法：

```kotlin
/**
* MyData serializer for JavaScript
*/
object MyDataSerializer: KSerializer<MyData> {
    override fun save(output: KOutput, obj: MyData) {
        output.writeStringValue(HexConverter.printHexBinary(obj.s.toUtf8Bytes()))
    }

    override fun load(input: KInput): MyData {
        return MyData(stringFromUtf8Bytes(HexConverter.parseHexBinary(input.readStringValue())))
    }

    override fun update(input: KInput, old: MyData): MyData {
        throw UpdateNotSupportedException(serialClassDesc.name)
    }
    
    override val serialClassDesc: KSerialClassDesc = SerialClassDescImpl("com.mypackage.MyData")
}
```

## 使用自定义序列化器

使用自定义Serializer的推荐方法是，为插件提供相关的线索来指定某个属性该使用哪个Serializer，在属性定义时使用注解 @Serializable(with = SomeKSerializer::class)：

```kotlin
@Serializable
data class MyWrapper(
    val id: Int,
    @Serializable(with=MyExternalSerializer::class) val data: MyData
)
```

这将仅影响为此专用类生成save/load方法，并允许插件在编译时解析序列化程序以减少运行时开销。

## 注册和上下文

默认情况下，编译可序列化类时，所有序列化程序都通过静态插件解析。这为我们提供了类型安全性，性能并将反射使用量降至最低。但是，如果没有 @Serializable类的注释而没有@Serializable(with=...)属性，通常，在编译时不可能知道使用哪个序列化程序 - 用户可以定义多个外部序列化程序，或者在其他模块中定义它们，或者甚至是来自库的类，对序列化一无所知。

为了支持这种情况，SerialContext引入了一个概念。粗略地说，它是一个映射，其中框架的运行时部分正在寻找序列化器，如果它们在编译时未被插件解析。从某种意义上说，它与杰克逊模块类似。

如果要使用外部序列化程序，则必须在某些（可能是新的）上下文中注册它们。然后，您必须将上下文传递给框架。上下文是灵活的，可以继承; 此外，您可以将不同的上下文传递给输出格式的不同实例。我们在示例中看到它：

```kotlin
// Imagine we have classes A and B with external serializers
fun foo(b: B): Pair<String, ByteArray> {
    val aContext = SerialContext().apply { registerSerializer(A::class, ASerializer) }
    
    val strContext = SerialContext(aContext) 
    strContext.registerSerializer(B::class, BStringSerializer)
    val json = JSON(context = strContext) // use string serializer for B in JSON
    
    val binContext = SerialContext(aContext)
    binContext.registerSerializer(B::class, BBinarySerializer)
    val cbor = CBOR(context = binContext) // user binarySerializer for B in CBOR
    
    return json.stringify(b) to cbor.dump(b)
} 
```