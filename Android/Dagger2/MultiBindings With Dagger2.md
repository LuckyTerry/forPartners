# MultiBindings With Dagger2

Dagger allows you to bind several objects into a collection even when the objects are bound in different modules using multibindings. Dagger assembles the collection so that application code can inject it without depending directly on the individual bindings.

*Dagger允许您将多个对象绑定到一个集合中，即使对象使用多重绑定绑定到不同模块中时也是如此。Dagger组合集合，以便应用程序代码可以注入它而不依赖于单个绑定。*

You could use multibindings to implement a plugin architecture, for example, where several modules can contribute individual plugin interface implementations so that a central class can use the entire set of plugins. Or you could have several modules contribute individual service providers to a map, keyed by name.

*您可以使用多重绑定实现插件体系结构，例如，其中几个模块可以贡献单个插件接口实现，以便中心类可以使用整套插件。或者你可以有几个模块将单个服务提供者提供给map，并以名称为key。*

## Set multibindings（*设置多重绑定*）

In order to contribute one element to an injectable multibound set, add an @IntoSet annotation to your module method:

*为了将一个元素添加到可注入的多重绑定集中，请为您的模块方法添加一个@IntoSet：*

```java
@Module
class MyModuleA {
  @Provides @IntoSet
  static String provideOneString(DepA depA, DepB depB) {
    return "ABC";
  }
}
```

You can also contribute several elements at one time by adding a module method that returns a subset and is annotated with @ElementsIntoSet:

*您还可以一次提供多个元素，通过为模块添加返回子集并使用@ElementsIntoSet注释的方法：*

```java
@Module
class MyModuleB {
  @Provides @ElementsIntoSet
  static Set<String> provideSomeStrings(DepA depA, DepB depB) {
    return new HashSet<String>(Arrays.asList("DEF", "GHI"));
  }
}
```

Now a binding in that component can depend on the set:

*现在，该组件中的绑定可以取决于该组：*

```java
class Bar {
  @Inject Bar(Set<String> strings) {
    assert strings.contains("ABC");
    assert strings.contains("DEF");
    assert strings.contains("GHI");
  }
}
```

Or the component can provide the set:

*或者组件可以提供这个集合：*

```java
@Component(modules = {MyModuleA.class, MyModuleB.class})
interface MyComponent {
  Set<String> strings();
}

@Test void testMyComponent() {
  MyComponent myComponent = DaggerMyComponent.create();
  assertThat(myComponent.strings()).containsExactly("ABC", "DEF", "GHI");
}
```

As with any other binding, in addition to depending on a multibound Set<Foo>, you can also depend on Provider<Set<Foo>> or Lazy<Set<Foo>>. You cannot, however, depend on Set<Provider<Foo>>.

*与其他任何绑定一样，除了依赖于多重绑定Set<Foo>之外，还可以依赖于Provider<Set<Foo>>或Lazy<Set<Foo>>。但是，你不能依靠Set<Provider<Foo>>。*

To contribute to a qualified multibound set, annotate each @Provides method with the qualifier:

*要实现一个约束的多重绑定集合，请使用限定符对每个@Provides方法进行注释：*

```java
@Module
class MyModuleC {
  @Provides @IntoSet
  @MyQualifier
  static Foo provideOneFoo(DepA depA, DepB depB) {
    return new Foo(depA, depB);
  }
}

@Module
class MyModuleD {
  @Provides
  static FooSetUser provideFooSetUser(@MyQualifier Set<Foo> foos) { ... }
}
```

## Map multibindings（*映射多重绑定*）

Dagger lets you use multibindings to contribute entries to an injectable map as long as the map keys are known at compile time.

*只要map key在编译时已知，Dagger就可以使用多重绑定将条目分配给可注入的map。*

To contribute an entry to a multibound map, add a method to a module that returns the value and is annotated with @IntoMap and with another custom annotation that specifies the map key for that entry. To contribute an entry to a qualified multibound map, annotate each @IntoMap method with the qualifier.

*要为多重绑定map添加一个entry，添加一个特定的方法，该方法返回value、标记有@IntoMap注解、标记有指定entry的key的自定义注解。要为一个受约束的多重绑定map提供entry，请使用限定符注解对每个@IntoMap方法进行标记。*

Then you can inject either the map itself (Map<K, V>) or a map containing value providers (Map<K, Provider<V>>). The latter is useful when you don’t want all of the values to be instantiated because you’re going to extract one value at a time, or because you want to get a potentially new instance of each value each time you query the map.

*然后你可以注入Map本身（Map<K, V>）或包含值Provider（Map<K, Provider<V>>）的Map。当你不希望所有的值都被立即实例化，因为你一次要提取一个值，或者每次查询Map时想要获取每个值的潜在新实例时，后者很有用。*

### Simple map keys（*简单的Map keys*）

For maps with keys that are strings, Class<?>, or boxed primitives, use one of the standard annotations in dagger.multibindings:

*对于key是字符串、Class<?>或基础类型的装箱时，请使用dagger.multibindings中标准注释之一：*

```java
@Module
class MyModule {
  @Provides @IntoMap
  @StringKey("foo")
  static Long provideFooValue() {
    return 100L;
  }

  @Provides @IntoMap
  @ClassKey(Thing.class)
  static String provideThingValue() {
    return "value for Thing";
  }
}

@Component(modules = MyModule.class)
interface MyComponent {
  Map<String, Long> longsByString();
  Map<Class<?>, String> stringsByClass();
}

@Test void testMyComponent() {
  MyComponent myComponent = DaggerMyComponent.create();
  assertThat(myComponent.longsByString().get("foo")).isEqualTo(100L);
  assertThat(myComponent.stringsByClass().get(Thing.class))
      .isEqualTo("value for Thing");
}
```

For maps with keys that are enums or a more specifically parameterized class, write an annotation type with one member whose type is the map key type, and annotate it with @MapKey:

*对于以枚举或更具体的参数化类作为key的map，使用一个类型为映射键类型的成员编写注释类型，并使用以下注释@MapKey：*

```java
enum MyEnum {
  ABC, DEF;
}

@MapKey
@interface MyEnumKey {
  MyEnum value();
}

@MapKey
@interface MyNumberClassKey {
  Class<? extends Number> value();
}

@Module
class MyModule {
  @Provides @IntoMap
  @MyEnumKey(MyEnum.ABC)
  static String provideABCValue() {
    return "value for ABC";
  }

  @Provides @IntoMap
  @MyNumberClassKey(BigDecimal.class)
  static String provideBigDecimalValue() {
    return "value for BigDecimal";
  }
}

@Component(modules = MyModule.class)
interface MyComponent {
  Map<MyEnum, String> myEnumStringMap();
  Map<Class<? extends Number>, String> stringsByNumberClass();
}

@Test void testMyComponent() {
  MyComponent myComponent = DaggerMyComponent.create();
  assertThat(myComponent.myEnumStringMap().get(MyEnum.ABC)).isEqualTo("value for ABC");
  assertThat(myComponent.stringsByNumberClass.get(BigDecimal.class))
      .isEqualTo("value for BigDecimal");
}
```

Your annotation’s single member can be any valid annotation member type except for arrays, and can have any name.

*您的注释的单个成员可以是除数组之外的任何有效的注释成员类型，并且可以具有任何名称。*

#### Complex map keys（*复杂的 map keys*）

If your map’s key is more than can be expressed by a single annotation member, you can use the entire annotation as the map key by setting @MapKey’s unwrapValue to false. In that case, the annotation can have array members as well.

*如果您的map的key超过了可以由单个注释成员表示的值，则可以将整个注记用作map key，方法是将@MapKeys的 unwrapValue设置为false。在这种情况下，注释也可以具有数组成员。*

```java
@MapKey(unwrapValue = false)
@interface MyKey {
  String name();
  Class<?> implementingClass();
  int[] thresholds();
}

@Module
class MyModule {
  @Provides @IntoMap
  @MyKey(name = "abc", implementingClass = Abc.class, thresholds = {1, 5, 10})
  static String provideAbc1510Value() {
    return "foo";
  }
}

@Component(modules = MyModule.class)
interface MyComponent {
  Map<MyKey, String> myKeyStringMap();
}
```

#### Using @AutoAnnotation to create annotation instances（*使用@AutoAnnotation创建注解实例*）

If your map uses complex keys, then you may need to create an instance of your @MapKey annotation at run-time to pass to the map’s get(Object) method. The easiest way to do that is to use the @AutoAnnotation annotation to create a static method that instantiates your annotation. See @AutoAnnotation’s documentation for more details.

*如果你的map使用复杂的key，那么你可能需要在运行时创建一个@MapKey注解的实例以传递给map的get(Object)方法。最简单的方法是使用@AutoAnnotation注解来创建一个实例化注释的静态方法。见 @AutoAnnotation的文档，了解更多详情。*

```java
class MyComponentTest {
  @Test void testMyComponent() {
    MyComponent myComponent = DaggerMyComponent.create();
    assertThat(myComponent.myKeyStringMap()
        .get(createMyKey("abc", Abc.class, new int[] {1, 5, 10}))
        .isEqualTo("foo");
  }

  @AutoAnnotation
  static MyKey createMyKey(String name, Class<?> implementingClass, int[] thresholds) {
    return new AutoAnnotation_MyComponentTest_createMyKey(name, implementingClass, thresholds);
  }
}
```

#### Maps whose keys are not known at compile time（*编译时Keys未知的Maps*）

Map multibindings work only if your map’s keys are known at compile time and can be expressed in an annotation. If your map’s keys don’t fit in those constraints, then you cannot create a multibound map, but you can work around that by using set multibindings to bind a set of objects that you can then transform into a non-multibound map.

*只有在编译时Map的Keys已知并且可以在注解中表示时多重绑定才起作用。如果Map的keys不符合这些约束条件，则无法创建多重绑定的Map，但可以使用set multibindings绑定一组对象，然后转换为非多重绑定Map，从而解决该问题。*

```java
@Module
class MyModule {
  @Provides @IntoSet
  static Map.Entry<Foo, Bar> entryOne(...) {
    Foo key = ...;
    Bar value = ...;
    return new SimpleImmutableEntry(key, value);
  }

  @Provides @IntoSet
  static Map.Entry<Foo, Bar> entryTwo(...) {
    Foo key = ...;
    Bar value = ...;
    return new SimpleImmutableEntry(key, value);
  }
}

@Module
class MyMapModule {
  @Provides
  static Map<Foo, Bar> fooBarMap(Set<Map.Entry<Foo, Bar>> entries) {
    Map<Foo, Bar> fooBarMap = new LinkedHashMap<>(entries.size());
    for (Map.Entry<Foo, Bar> entry : entries) {
      fooBarMap.put(entry.getKey(), entry.getValue());
    }
    return fooBarMap;
  }
}
```

Note that this technique does not give you the automatic binding of Map<Foo, Provider<Bar>> as well. If you want a map of providers, the Map.Entry objects in your multibound set should include the providers. Then your non-multibound map can have Provider values.

*请注意，这种技术不会给你自动绑定 Map<Foo, Provider<Bar>>。如果您需要Provider的Map，则Map.Entry多个绑定集中的对象应包含Provider。那么你的非多重绑定Map可以有Provider值。*

```java
@Module
class MyModule {
  @Provides @IntoSet
  static Map.Entry<Foo, Provider<Bar>> entry(
      Provider<BarSubclass> barSubclassProvider) {
    Foo key = ...;
    return new SimpleImmutableEntry(key, barSubclassProvider);
  }
}

@Module
class MyProviderMapModule {
  @Provides
  static Map<Foo, Provider<Bar>> fooBarProviderMap(
      Set<Map.Entry<Foo, Provider<Bar>>> entries) {
    return ...;
  }
}
```

## Declaring multibindings（*声明多重绑定*）

You can declare that a multibound set or map is bound by adding an abstract @Multibinds-annotated method to a module that returns the set or map you want to declare.

*您可以向module添加一个标记有@Multibinds注解的abstract方法，该方法返回你期望的Map或Set，以此来声明多重绑定的Map或Set*

You do not have to use @Multibinds for sets or maps that have at least one @IntoSet, @ElementsIntoSet, or @IntoMap binding, but you do have to declare them if they may be empty.

*对于至少有一个 @IntoSet，@ElementsIntoSet或@IntoMap的情况，您不必使用@Multibinds标记Set或Map，但你必须声明它们是否可能是空的。*

```java
@Module
abstract class MyModule {
  @Multibinds abstract Set<Foo> aSet();
  @Multibinds @MyQualifier abstract Set<Foo> aQualifiedSet();
  @Multibinds abstract Map<String, Foo> aMap();
  @Multibinds @MyQualifier abstract Map<String, Foo> aQualifiedMap();
}
```

A given set or map multibinding can be declared any number of times without error. Dagger never implements or calls any @Multibinds methods.

*给定的Map或Set多重绑定可以被声明任意次数而不会出错。Dagger从不实现或调用任何@Multibinds注解标记的方法。*

### Alternative: @ElementsIntoSet returning an empty set（*选择：@ElementsIntoSet返回一个空集*）

For empty sets only, as an alternative, you can add a @ElementsIntoSet method that returns an empty set:

*仅对于空集，作为替代方案，您可以添加一个@ElementsIntoSet 返回空集的方法：*

```java
@Module
class MyEmptySetModule {
  @Provides @ElementsIntoSet
  static Set<Foo> primeEmptyFooSet() {
    return Collections.emptySet();
  }
}
```

### Inherited subcomponent multibindings（*继承的子组件多重绑定*）

A binding in a subcomponent can depend on a multibound set or map from its parent, just as it can depend on any other binding from its parent. But a subcomponent can add elements to multibound sets or maps that are bound in its parent as well, by simply including the appropriate @Provides methods in its modules.

*子组件中的绑定可以依赖于来自其父组件的多重绑定Map或Set，就像它可以依赖于其父组件的任何其他绑定一样。但是，子组件可以将元素添加到绑定在其父项中的多重绑定Map或Set中，只需在其模块中包含适当的@Provides方法即可。*

When that happens, the set or map is different depending on where it is injected. When it is injected into a binding defined on the subcomponent, then it has the values or entries defined by the subcomponent’s multibindings as well as those defined by the parent component’s multibindings. When it is injected into a binding defined on the parent component, it has only the values or entries defined there.

*当发生这种情况时，根据注入位置的不同，Set或Map会有所不同。当它被注入到子组件定义的绑定中时，它具有由子组件的多重绑定定义的值或条目以及由父组件的多重绑定定义的值或条目。当它被注入到在父组件上定义的绑定中时，它只有在那里定义的值或条目。*

```java
@Component(modules = ParentModule.class)
interface ParentComponent {
  Set<String> strings();
  Map<String, String> stringMap();
  ChildComponent childComponent();
}

@Module
class ParentModule {
  @Provides @IntoSet
  static String string1() {
    "parent string 1";
  }

  @Provides @IntoSet
  static String string2() {
    "parent string 2";
  }

  @Provides @IntoMap
  @StringKey("a")
  static String stringA() {
    "parent string A";
  }

  @Provides @IntoMap
  @StringKey("b")
  static String stringB() {
    "parent string B";
  }
}

@Subcomponent(modules = ChildModule.class)
interface ChildComponent {
  Set<String> strings();
  Map<String, String> stringMap();
}

@Module
class ChildModule {
  @Provides @IntoSet
  static String string3() {
    "child string 3";
  }

  @Provides @IntoSet
  static String string4() {
    "child string 4";
  }

  @Provides @IntoMap
  @StringKey("c")
  static String stringC() {
    "child string C";
  }

  @Provides @IntoMap
  @StringKey("d")
  static String stringD() {
    "child string D";
  }
}

@Test void testMultibindings() {
  ParentComponent parentComponent = DaggerParentComponent.create();
  assertThat(parentComponent.strings()).containsExactly(
      "parent string 1", "parent string 2");
  assertThat(parentComponent.stringMap().keySet()).containsExactly("a", "b");

  ChildComponent childComponent = parentComponent.childComponent();
  assertThat(childComponent.strings()).containsExactly(
      "parent string 1", "parent string 2", "child string 3", "child string 4");
  assertThat(childComponent.stringMap().keySet()).containsExactly(
      "a", "b", "c", "d");
}
```