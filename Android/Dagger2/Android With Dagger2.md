# Android With Dagger2

Updated at 05/10/2018

One of the primary advantages of Dagger 2 over most other dependency injection frameworks is that its strictly generated implementation (no reflection) means that it can be used in Android applications. However, there are still some considerations to be made when using Dagger within Android applications.

*Dagger 2比其他大多数依赖注入框架的主要优点之一是其严格生成的实现（无反射）意味着它可以在Android应用程序中使用。然而，在Android应用程序中使用Dagger进行时仍然要注意一些事项。*

## Philosophy

While code written for Android is Java source, it is often quite different in terms of style. Typically, such differences exist to accomodate the unique performance considerations of a mobile platform.

*虽然为Android编写的代码是Java源代码，但它在风格方面通常很不相同。通常情况下，这些差异是为了适应移动平台的独特 性能考虑。*

But many of the patterns commonly applied to code intended for Android are contrary to those applied to other Java code. Even much of the advice in Effective Java is considered inappropriate for Android.

*但是，通常应用于Android代码的许多模式与应用于其他Java代码的模式相反。甚至于Effective Java一书中的许多有效建议被认为不适合Android。*

In order to achieve the goals of both idiomatic and portable code, Dagger relies on ProGuard to post-process the compiled bytecode. This allows Dagger to emit source that looks and feels natural on both the server and Android, while using the different toolchains to produce bytecode that executes efficiently in both environements. Moreover, Dagger has an explicit goal to ensure that the Java source that it generates is consistently compatible with ProGuard optimizations.

*为了达到惯用和可移植代码的目标，Dagger依靠ProGuard来后处理编译的字节码。这使得Dagger能够在服务器和Android上发出看起来和感觉自然的源代码，同时使用不同的工具链来生成在两个环境中都能高效执行的字节码。此外，Dagger明确的目标是确保它生成的Java源与ProGuard优化一致。*

Of course, not all issues can be addressed in that manner, but it is the primary mechanism by which Android-specific compatbility will be provided.

*当然，并不是所有问题都可以用这种方式来解决，但它是提供Android特定兼容性的主要机制。*

Dagger assumes that users on Android will use ProGuard.

*Dagger假定Android上的用户将使用ProGuard。*

## Recommended ProGuard Settings

Watch this space for ProGuard settings that are relevant to applications using Dagger.

*观看此空间以查看与使用Dagger的应用程序相关的ProGuard设置。*

### dagger.android

One of the central difficulties of writing an Android application using Dagger is that many Android framework classes are instantiated by the OS itself, like Activity and Fragment, but Dagger works best if it can create all the injected objects. Instead, you have to perform members injection in a lifecycle method. This means many classes end up looking like:

*使用Dagger设计一个Android应用程序的主要困难是，许多Android框架类（如Activity和Fragment）是由操作系统本身实例化，但是当Dagger可以创建所有的注入对象时才是它最有效的工作模式。所以相反，您必须在生命周期方法中执行成员注入。这意味着许多类最终看起来像：*

```java
public class FrombulationActivity extends Activity {
  @Inject Frombulator frombulator;

  @Override
  public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    // DO THIS FIRST. Otherwise frombulator might be null!
    ((SomeApplicationBaseType) getContext().getApplicationContext())
        .getApplicationComponent()
        .newActivityComponentBuilder()
        .activity(this)
        .build()
        .inject(this);
    // ... now you can write the exciting code
  }
}
```

This has a few problems:（*这有几个问题：*）

- Copy-pasting code makes it hard to refactor later on. As more and more developers copy-paste that block, fewer will know what it actually does.（*复制粘贴代码使得以后很难重构。随着越来越多的开发者复制粘贴该块，越来越少的人会知道它实际上做了什么。*）

- More fundamentally, it requires the type requesting injection (FrombulationActivity) to know about its injector. Even if this is done through interfaces instead of concrete types, it breaks a core principle of dependency injection: a class shouldn’t know anything about how it is injected.（*更根本的是，它需要请求注入（FrombulationActivity）的类型来了解其注入器。即使这是通过接口而不是具体类型完成的，它打破了依赖注入的核心原则：类不应该知道注入的方式。*）

The classes in dagger.android offer one approach to simplify this pattern.

*dagger.android中的这些类提供了一种简化这种模式的方法。*

### Injecting Activity objects

1. Install AndroidInjectionModule in your application component to ensure that all bindings necessary for these base types are available.(*在应用程序组件中安装AndroidInjectionModule以确保这些基本类型所需的所有绑定都可用。*)

2. Start off by writing a @Subcomponent that implements AndroidInjector<YourActivity>, with a @Subcomponent.Builder that extends AndroidInjector.Builder<YourActivity>:(*首先写一个实现了AndroidInjector<YourActivity>接口的@Subcomponent，然后在其内部，定义一个继承AndroidInjector.Builder<YourActivity>类的@Subcomponent.Builder：*)

```java
@Subcomponent(modules = ...)
public interface YourActivitySubcomponent extends AndroidInjector<YourActivity> {
  @Subcomponent.Builder
  public abstract class Builder extends AndroidInjector.Builder<YourActivity> {}
}
```

3. After defining the subcomponent, add it to your component hierarchy by defining a module that binds the subcomponent builder and adding it to the component that injects your Application:（*定义subcomponent后，通过定义一个绑定到subcomponent builder的module，并把它添加到将要注入application的component中，以此来将subcomponent添加到组件层次结构：*）

```java
@Module(subcomponents = YourActivitySubcomponent.class)
abstract class YourActivityModule {
  @Binds
  @IntoMap
  @ActivityKey(YourActivity.class)
  abstract AndroidInjector.Factory<? extends Activity>
      bindYourActivityInjectorFactory(YourActivitySubcomponent.Builder builder);
}

@Component(modules = {..., YourActivityModule.class})
interface YourApplicationComponent {}
```

Pro-tip: If your subcomponent and its builder have no other methods or supertypes than the ones mentioned in step #2, you can use @ContributesAndroidInjector to generate them for you. Instead of steps 2 and 3, add an abstract module method that returns your activity, annotate it with @ContributesAndroidInjector, and specify the modules you want to install into the subcomponent. If the subcomponent needs scopes, apply the scope annotations to the method as well.（*提示：如果您的子组件及其构建器没有其他方法或超类型，则可以使用 @ContributesAndroidInjector为您生成它们。除了第2步和第3步之外，还要添加一个abstract模块方法，该方法返回您的activity，对其进行注释@ContributesAndroidInjector，并指定要安装到子组件中的模块。如果子组件需要范围，则也可以将范围注释应用于该方法。*）

```java
@ActivityScope
@ContributesAndroidInjector(modules = { /* modules to install into the subcomponent */ })
abstract YourActivity contributeYourActivityInjector();
```

4. Next, make your Application implement HasActivityInjector and @Inject a DispatchingAndroidInjector<Activity> to return from the activityInjector() method:（*接下来，让你的Application实现HasActivityInjector接口并@Inject一个从activityInjector()方法返回的DispatchingAndroidInjector<Activity>：*）

```java
public class YourApplication extends Application implements HasActivityInjector {
  @Inject DispatchingAndroidInjector<Activity> dispatchingActivityInjector;

  @Override
  public void onCreate() {
    super.onCreate();
    DaggerYourApplicationComponent.create()
        .inject(this);
  }

  @Override
  public AndroidInjector<Activity> activityInjector() {
    return dispatchingActivityInjector;
  }
}
```

5. Finally, in your Activity.onCreate() method, call AndroidInjection.inject(this) before calling super.onCreate():（*最后，在你的Activity.onCreate()方法中，在调用super.onCreate()之前调用：AndroidInjection.inject(this);*）

```java
public class YourActivity extends Activity {
  public void onCreate(Bundle savedInstanceState) {
    AndroidInjection.inject(this);
    super.onCreate(savedInstanceState);
  }
}
```

6. Congratulations!（*恭喜！*）

#### How did that work?（*这是如何工作的？*）

AndroidInjection.inject() gets a DispatchingAndroidInjector<Activity> from the Application and passes your activity to inject(Activity). The DispatchingAndroidInjector looks up the AndroidInjector.Factory for your activity’s class (which is YourActivitySubcomponent.Builder), creates the AndroidInjector (which is YourActivitySubcomponent), and passes your activity to inject(YourActivity).

*AndroidInjection.inject()从Application得到了一个DispatchingAndroidInjector<Activity>，然后将您的activityt传递给inject(Activity)方法作为参数。在 DispatchingAndroidInjector为您的activity类查找AndroidInjector.Factory（这是YourActivitySubcomponent.Builder），创建了AndroidInjector（这是YourActivitySubcomponent），并通过你的activity传递给inject(YourActivity)方法。*

### Injecting Fragment objects(*注入Fragment对象*)

Injecting a Fragment is just as simple as injecting an Activity. Define your subcomponent in the same way, replacing Activity type parameters with Fragment, @ActivityKey with @FragmentKey, and HasActivityInjector with HasFragmentInjector.

*注入一个Fragment与注入Activity一样简单。以同样的方式定义你的subcomponent，使用Fragment取代Activity类型参数，使用@FragmentKey取代@ActivityKey，使用HasFragmentInjector取代HasActivityInjector。*

Instead of injecting in onCreate() as is done for Activity types, inject Fragments to in onAttach().

*与activity在onCreate()方法注入不同，Fragment需要在其onAttach()方法中进行注入。*

Unlike the modules defined for Activitys, you have a choice of where to install modules for Fragments. You can make your Fragment component a subcomponent of another Fragment component, an Activity component, or the Application component — it all depends on which other bindings your Fragment requires. After deciding on the component location, make the corresponding type implement HasFragmentInjector. For example, if your Fragment needs bindings from YourActivitySubcomponent, your code will look something like this:

*与为Activitys定义的模块不同，您可以选择在哪里安装Fragments模块。您可以将Fragment组件作为另一个Fragment、Activity或Application组件的子组件 - 这完全取决于您Fragment所需要的绑定。确定组件位置后，完成HasFragmentInjector接口相应的类型实现。例如，如果您Fragment需要绑定YourActivitySubcomponent，您的代码将如下所示：*

```java
public class YourActivity extends Activity
    implements HasFragmentInjector {
  @Inject DispatchingAndroidInjector<Fragment> fragmentInjector;

  @Override
  public void onCreate(Bundle savedInstanceState) {
    AndroidInjection.inject(this);
    super.onCreate(savedInstanceState);
    // ...
  }

  @Override
  public AndroidInjector<Fragment> fragmentInjector() {
    return fragmentInjector;
  }
}

public class YourFragment extends Fragment {
  @Inject SomeDependency someDep;

  @Override
  public void onAttach(Activity activity) {
    AndroidInjection.inject(this);
    super.onAttach(activity);
    // ...
  }
}

@Subcomponent(modules = ...)
public interface YourFragmentSubcomponent extends AndroidInjector<YourFragment> {
  @Subcomponent.Builder
  public abstract class Builder extends AndroidInjector.Builder<YourFragment> {}
}

@Module(subcomponents = YourFragmentSubcomponent.class)
abstract class YourFragmentModule {
  @Binds
  @IntoMap
  @FragmentKey(YourFragment.class)
  abstract AndroidInjector.Factory<? extends Fragment>
      bindYourFragmentInjectorFactory(YourFragmentSubcomponent.Builder builder);
}

@Subcomponent(modules = { YourFragmentModule.class, ... }
public interface YourActivityOrYourApplicationComponent { ... }
```

### Base Framework Types

Because DispatchingAndroidInjector looks up the appropriate AndroidInjector.Factory by the class at runtime, a base class can implement HasActivityInjector/HasFragmentInjector/etc as well as call AndroidInjection.inject(). All each subclass needs to do is bind a corresponding @Subcomponent. Dagger provides a few base types that do this, such as DaggerActivity and DaggerFragment, if you don’t have a complicated class hierarchy. Dagger also provides a DaggerApplication for the same purpose — all you need to do is to extend it and override the applicationInjector() method to return the component that should inject the Application.

*由于DispatchingAndroidInjector在运行时查找相应的AndroidInjector.Factory的类，那么一个基类可以实现 HasActivityInjector、HasFragmentInjector等接口并调用AndroidInjection.inject()方法。每个子类需要做的就是绑定一个对应的@Subcomponent。如果你没有复杂的类层次结构，Dagger提供了一些基本类型，例如DaggerActivity和DaggerFragment供您使用。处于相同的目的，Dagger也提供了一个DaggerApplication - 你需要做的就是扩展它并覆盖applicationInjector()方法并返回应该注入到Application的组件。*

The following types are also included:（*以下类型也包括在内：*）

- DaggerService and DaggerIntentService
- DaggerBroadcastReceiver
- DaggerContentProvider

Note: DaggerBroadcastReceiver should only be used when the BroadcastReceiver is registered in the AndroidManifest.xml. When the BroadcastReceiver is created in your own code, prefer constructor injection instead.

*注意： DaggerBroadcastReceiver只能在使用AndroidManifest.xml方式注册BroadcastReceiver时使用。当你在自己的代码中创建BroadcastReceiver时，更推荐构造器注入。*

### Support libraries（*支持库*）

For users of the Android support library, parallel types exist in the dagger.android.support package. Note that while support Fragment users have to bind
AndroidInjector.Factory<? extends android.support.v4.app.Fragment>, AppCompat users should continue to implement AndroidInjector.Factory<? extends Activity> and not <? extends AppCompatActivity> (or FragmentActivity).

*对于Android支持库的用户，dagger.android.support包中存在并行类型 。请注意，虽然support Fragment用户必须绑定
AndroidInjector.Factory<? extends android.support.v4.app.Fragment>，但AppCompat用户应该继续执行AndroidInjector.Factory<? extends Activity>而不是<? extends AppCompatActivity>（或FragmentActivity）。*

### How do I get it?（*我如何得到它？*）

Add the following to your build.gradle:

*将以下内容添加到您的build.gradle中：*

```java
dependencies {
  compile 'com.google.dagger:dagger-android:2.x'
  compile 'com.google.dagger:dagger-android-support:2.x' // if you use the support libraries
  annotationProcessor 'com.google.dagger:dagger-android-processor:2.x'
}
```

## When to inject（*何时注射*）

Constructor injection is preferred whenever possible because javac will ensure that no field is referenced before it has been set, which helps avoid NullPointerExceptions. When members injection is required (as discussed above), prefer to inject as early as possible. For this reason, DaggerActivity calls AndroidInjection.inject() immediately in onCreate(), before calling super.onCreate(), and DaggerFragment does the same in onAttach(), which also prevents inconsistencies if the Fragment is reattached.

*只要有可能，构造其注入是首选，因为javac确保在该字段设置之前没有字段引用，这有助于避免NullPointerExceptions。当需要成员注射（如上所述）时，倾向于尽早注射。为此，DaggerActivity呼吁在onCreate()方法中调用 super.onCreate()之前就立刻执行AndroidInjection.inject()，DaggerFragment在onAttach()方法中同理，这也防止了如果Fragment重新创建导致的不一致。*

It is crucial to call AndroidInjection.inject() before super.onCreate() in an Activity, since the call to super attaches Fragments from the previous activity instance during configuration change, which in turn injects the Fragments. In order for the Fragment injection to succeed, the Activity must already be injected. For users of ErrorProne, it is a compiler error to call AndroidInjection.inject() after super.onCreate().

*在Activity的super.onCreate()之前调用AndroidInjection.inject()是非常重要的，因为super方法的调用会attach先前的fragment（先前的配置更改期间的activity的fragment）到现在的activity，而这又会注入 Fragments。为了使Fragment注射成功，Activity 必须已经注射。对于ErrorProne的用户来说，super.onCreate()之后调用AndroidInjection.inject()会出现编译器错误。*

## FAQ（*常问问题*）

### Scoping AndroidInjector.Factory（*作用域 AndroidInjector.Factory*）

AndroidInjector.Factory is intended to be a stateless interface so that implementors don’t have to worry about managing state related to the object which will be injected. When DispatchingAndroidInjector requests a AndroidInjector.Factory, it does so through a Provider so that it doesn’t explicitly retain any instances of the factory. Because the AndroidInjector.Builder implementation that is generated by Dagger does retain an instance of the Activity/Fragment/etc that is being injected, it is a compile-time error to apply a scope to the methods which provide them. If you are positive that your AndroidInjector.Factory does not retain an instance to the injected object, you may suppress this error by applying
@SuppressWarnings("dagger.android.ScopedInjectoryFactory") to your module method.

*AndroidInjector.Factory旨在成为一个无状态的接口，以便实现者不必担心管理与将被注入的对象相关的状态。当DispatchingAndroidInjector请求时 AndroidInjector.Factory，它通过一个Provider这样做，以便它不明确地保留工厂的任何实例。由于AndroidInjector.BuilderDagger生成的 实现确实 保留了正在被注入的Activity/ Fragment/ etc 的实例，因此将一个作用域应用于提供它们的方法是一个编译时错误。如果您肯定您AndroidInjector.Factory没有将实例保留在注入对象中，则可以通过应用来抑制此错误
@SuppressWarnings("dagger.android.ScopedInjectoryFactory") 到你的模块方法。*