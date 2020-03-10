# Kotlin跨平台/多格式无反射序列化

Kotlin序列化支持由三部分组成：编译器插件，它为对象生成访问者/序列化器代码，IntelliJ插件和运行时库。

- 支持标记为@Serializable标准集合的Kotlin类。

- 支持JSON，CBOR和Protobuf格式的开箱即用。

- 相同的代码适用于Kotlin / JVM和Kotlin / JS。Kotlin / Native支持有限，请参阅下面的安装部分。

## 运行时概述

该项目包含运行时库。运行时库提供：

- 由编译器生成的代码（KInput，KOutput）调用的接口。

- 这些接口的基本框架实现，如果要实现自定义数据格式（，）ElementValueInput/Output，应该覆盖某些方法NamedValueInput/OutputElementValueTransformer

- 一些内部类，如内置函数和集合序列化程序。

- 即用型序列化格式。

- 受益于序列化框架的其他有用类（例如，对象到映射变换器）

您可以打开JVM或JS的示例项目以开始使用它。

## 目录

- 快速举例
- Kotlin 1.3即将发生的变化
- 图书馆安装
- 使用IntelliJ IDEA
- 兼容性说明
- 用法
- 支持的Kotlin类的更多示例
- 编写自定义序列化程序
- 附加格式
- 从源代码构建库和编译器插件