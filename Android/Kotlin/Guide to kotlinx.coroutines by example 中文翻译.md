# 通过示例指导kotlinx.coroutines

这是一个关于kotlinx.coroutines核心功能的指南，附带一系列例子。

## 介绍和设置

作为一种语言，Kotlin在其标准库中只提供了最低限度的低级API，以使其他各种库能够使用协程。 与许多其他具有类似功能的语言不同，异步和等待不是Kotlin中的关键字，甚至不是其标准库中的一部分。

kotlinx.coroutines就是这样一个丰富的库。 它包含许多本指南涵盖的高级协程启用的原始类型，包括启动，异步等。 您需要添加对kotlinx-coroutines-core模块的依赖关系，如此处所述，以便在项目中使用本指南中的原始类型。

## 目录

[TOC]

## 协程基础

本节涵盖基本的协程概念。

### 你的第一个协程

运行以下代码：

```kotlin
fun main(args: Array<String>) {
    launch { // launch new coroutine in background and continue
        delay(1000L) // non-blocking delay for 1 second (default time unit is ms)
        println("World!") // print after delay
    }
    println("Hello,") // main thread continues while coroutine is delayed
    Thread.sleep(2000L) // block main thread for 2 seconds to keep JVM alive
}
```

运行这个代码：

```text
Hello,
World!
```

基本上，协程是轻量级的线程。 他们与发射协程生成器一起发射。 你可以使用Thread.sleep（...）来实现相同的结果，用线程{...}和延迟（...）替换launch {...}。 尝试一下。

如果以thread替换launch，编译器会产生以下错误：

```
Error: Kotlin: Suspend functions are only allowed to be called from a coroutine or another suspend function
```

这是因为延迟是一个特殊的暂停功能，它不会阻塞线程，而是暂停协程，只能从协程中使用。

### 桥接阻塞和非阻塞的世界

第一个例子在相同的代码中混合了非阻塞 delay(...)和阻塞 Thread.sleep(...)。很容易迷失哪一个是阻塞的，哪一个不是。让我们来明确一下使用runBlocking协同生成器进行阻塞：

```kotlin
fun  main（args ： Array < String >）{
    启动{ //在后台启动新协程并继续 
        延迟（1000L）
        println（“世界！”）
    }
    println（“你好，”）//主线程立即继续 
    runBlocking {      //但是这个表达式阻塞了主线程的 
        延迟（2000L）   // ...当我们延迟2秒来让JVM保持活动
    } 
}
```

结果是一样的，但是这个代码只使用非阻塞延迟。在主线程，调用runBlocking，块，直到协程内runBlocking完成。

这个例子也可以用更习惯的方式重写，使用runBlocking来封装主函数的执行：

```kotlin
fun main(args: Array<String>) = runBlocking<Unit> { // start main coroutine
    launch { // launch new coroutine in background and continue
        delay(1000L)
        println("World!")
    }
    println("Hello,") // main coroutine continues here immediately
    delay(2000L)      // delaying for 2 seconds to keep JVM alive
}
```

这里runBlocking <Unit> {...}作为一个适配器，用来启动顶级主协程。 我们明确指定了它的单位返回类型，因为Kotlin中一个格式正确的主函数必须返回Unit。

这也是为暂停函数编写单元测试的一种方法：

```kotlin
class MyTest {
    @测试
    fun  testMySuspendingFunction（）= runBlocking < Unit > {
         //这里我们可以使用任何我们喜欢的断言风格
    }
}
```

### 等待工作

当另一个协程正在工作时延迟一段时间不是一个好方法。让我们明确地等待（以非阻塞的方式），直到我们启动的后台Job完成：

```kotlin
fun main(args: Array<String>) = runBlocking<Unit> {
    val job = launch { // launch new coroutine and keep a reference to its Job
        delay(1000L)
        println("World!")
    }
    println("Hello,")
    job.join() // wait until child coroutine completes
}
```

现在结果仍然是一样的，但是主协程的代码并不以任何方式与后台作业的持续时间相关联。好多了。

### 提取函数重构

我们将启动{...}中的代码块提取到一个单独的函数中。 当你对这个代码执行“提取函数”重构时，你会得到一个带有暂停修饰符的新函数。 这是你的第一个暂停功能。 暂停功能可以像常规功能一样在协同程序中使用，但是它们的附加功能是可以使用其他暂停功能（例如本例中的延迟）来暂停协程的执行。

```kotlin
fun main(args: Array<String>) = runBlocking<Unit> {
    val job = launch { doWorld() }
    println("Hello,")
    job.join()
}

// this is your first suspending function
suspend fun doWorld() {
    delay(1000L)
    println("World!")
}
```

### 协程是轻量级的

运行以下代码：

```kotlin
fun main(args: Array<String>) = runBlocking<Unit> {
    val jobs = List(100_000) { // launch a lot of coroutines and list their jobs
        launch {
            delay(1000L)
            print(".")
        }
    }
    jobs.forEach { it.join() } // wait for all jobs to complete
}
```

它启动100K协程，一秒钟后，每个协程打印一个点。现在，尝试用线程。会发生什么？（很可能你的代码会产生某种内存不足的错误）

### 协程就像守护线程

以下代码将启动一个长时间运行的协程，每秒打印两次“I'm sleeping”，然后在一段延迟之后从主函数返回：

```kotlin
fun main(args: Array<String>) = runBlocking<Unit> {
    launch {
        repeat(1000) { i ->
            println("I'm sleeping $i ...")
            delay(500L)
        }
    }
    delay(1300L) // just quit after delay
}
```

你可以运行并看到它打印三行并终止：

```text
I'm sleeping 0 ...
I'm sleeping 1 ...
I'm sleeping 2 ...
```

主动协程不会使进程保持活跃状态​​。它们就像守护线程一样。

## 取消和超时

本节介绍了协程的取消和超时。

### 取消协程执行

在小应用程序中，从“main”方法返回可能听起来像是一个好主意，让所有的协程都隐式地终止。在一个更大的，长期运行的应用程序中，您需要更细致的控制。在launch 函数返回一个Job，可用于取消运行协程：

```kotlin
fun main(args: Array<String>) = runBlocking<Unit> {
    val job = launch {
        repeat(1000) { i ->
            println("I'm sleeping $i ...")
            delay(500L)
        }
    }
    delay(1300L) // delay a bit
    println("main: I'm tired of waiting!")
    job.cancel() // cancels the job
    job.join() // waits for job's completion 
    println("main: Now I can quit.")
}
```

它产生以下输出：

```text
I'm sleeping 0 ...
I'm sleeping 1 ...
I'm sleeping 2 ...
main: I'm tired of waiting!
main: Now I can quit.
```

一旦主调用job.cancel，我们看不到其他协同程序的任何输出，因为它被取消了。还有一个作业扩展功能cancelAndJoin ，它结合了cancel和join的调用。

### 取消是合作的

协程取消是合作的。 协程代码必须配合才能被取消。 kotlinx.coroutines中的所有挂起函数都是可以取消的。 他们检查coroutine的取消，并取消时抛出CancellationException。 但是，如果一个协程在计算中工作，并且不检查取消，那么它不能被取消，如下面的例子所示：

```kotlin
fun main(args: Array<String>) = runBlocking<Unit> {
    val startTime = System.currentTimeMillis()
    val job = launch {
        var nextPrintTime = startTime
        var i = 0
        while (i < 5) { // computation loop, just wastes CPU
            // print a message twice a second
            if (System.currentTimeMillis() >= nextPrintTime) {
                println("I'm sleeping ${i++} ...")
                nextPrintTime += 500L
            }
        }
    }
    delay(1300L) // delay a bit
    println("main: I'm tired of waiting!")
    job.cancelAndJoin() // cancels the job and waits for its completion
    println("main: Now I can quit.")
}
```

运行它以查看即使在取消之前它仍继续打印“我正在睡觉”，直到五次迭代完成后才自行完成。

### 使计算代码可以取消

有两种方法可以取消计算代码。第一个是定期调用一个检查取消的暂停功能。有一个yield函数是一个很好的选择。另一个是明确检查取消状态。让我们尝试后面的方法。

在while（isActive）前面的例子中（i <5）替换并重新运行。

```kotlin
fun main(args: Array<String>) = runBlocking<Unit> {
    val startTime = System.currentTimeMillis()
    val job = launch {
        var nextPrintTime = startTime
        var i = 0
        while (isActive) { // cancellable computation loop
            // print a message twice a second
            if (System.currentTimeMillis() >= nextPrintTime) {
                println("I'm sleeping ${i++} ...")
                nextPrintTime += 500L
            }
        }
    }
    delay(1300L) // delay a bit
    println("main: I'm tired of waiting!")
    job.cancelAndJoin() // cancels the job and waits for its completion
    println("main: Now I can quit.")
}
```

正如你所看到的，现在这个循环被取消了。isActive是通过CoroutineScope对象在协程代码中可用的属性。

### 使用finally关闭资源

可取消的挂起函数在取消时抛出CancellationException，这可以用通常的方式处理。 例如，尝试{...}最后{...}表达式和Kotlin使用函数在协程被取消时通常执行它们的终结操作：

```kotlin
fun main(args: Array<String>) = runBlocking<Unit> {
    val job = launch {
        try {
            repeat(1000) { i ->
                println("I'm sleeping $i ...")
                delay(500L)
            }
        } finally {
            println("I'm running finally")
        }
    }
    delay(1300L) // delay a bit
    println("main: I'm tired of waiting!")
    job.cancelAndJoin() // cancels the job and waits for its completion
    println("main: Now I can quit.")
}
```

join和cancelAnd都等待所有完成动作完成，所以上面的例子产生下面的输出：

```text
I'm sleeping 0 ...
I'm sleeping 1 ...
I'm sleeping 2 ...
main: I'm tired of waiting!
I'm running finally
main: Now I can quit.
```

### 运行不可取消的块

任何尝试finally在前面的示例的块中使用暂停功能都将导致 CancellationException，因为运行此代码的协程被取消。通常，这不是问题，因为所有关闭操作（关闭文件，取消作业或关闭任何类型的通信通道）通常都是非阻塞的，不涉及任何挂起功能。但是，在极少数情况下，如果需要在取消的协程中暂停，则可以run(NonCancellable) {...}使用运行函数和NonCancellable上下文来包装相应的代码， 如下例所示：

```kotlin
fun main(args: Array<String>) = runBlocking<Unit> {
    val job = launch {
        try {
            repeat(1000) { i ->
                println("I'm sleeping $i ...")
                delay(500L)
            }
        } finally {
            run(NonCancellable) {
                println("I'm running finally")
                delay(1000L)
                println("And I've just delayed for 1 sec because I'm non-cancellable")
            }
        }
    }
    delay(1300L) // delay a bit
    println("main: I'm tired of waiting!")
    job.cancelAndJoin() // cancels the job and waits for its completion
    println("main: Now I can quit.")
}
```

###超时

在实践中取消协同执行的最明显的原因是因为它的执行时间已经超过了一些超时。虽然您可以手动跟踪对相应作业的引用，并启动一个单独的协程以在延迟之后取消所跟踪的协程，但是可以使用Timeout功能来执行此操作。看下面的例子：

```kotlin
fun main(args: Array<String>) = runBlocking<Unit> {
    withTimeout(1300L) {
        repeat(1000) { i ->
            println("I'm sleeping $i ...")
            delay(500L)
        }
    }
}
```

它产生以下输出：

```text
I'm sleeping 0 ...
I'm sleeping 1 ...
I'm sleeping 2 ...
Exception in thread "main" kotlinx.coroutines.experimental.TimeoutCancellationException: Timed out waiting for 1300 MILLISECONDS
```

withTimeout引发的TimeoutCancellationException是CancellationException的子类。 我们之前没有看到它在控制台上打印的堆栈轨迹。 这是因为在取消的协程中，CancellationException被认为是协程完成的正常原因。 但是，在这个例子中，我们已经在主函数内部使用了Timeout。

由于取消只是一个例外，所有的资源将以通常的方式关闭。 你可以在try {...} catch（e：TimeoutCancellationException）{...}中使用timeout来封装代码，如果你需要在任何类型的超时时间内做一些额外的操作，或者使用与withTimeout类似的TimeoutOrNull函数， 但在超时时返回null，而不是抛出异常：

```kotlin
fun main(args: Array<String>) = runBlocking<Unit> {
    val result = withTimeoutOrNull(1300L) {
        repeat(1000) { i ->
            println("I'm sleeping $i ...")
            delay(500L)
        }
        "Done" // will get cancelled before it produces this result
    }
    println("Result is $result")
}
```

运行此代码时不再有例外：

```text
I'm sleeping 0 ...
I'm sleeping 1 ...
I'm sleeping 2 ...
Result is null
```

## 撰写挂起函数

本节介绍构成挂起函数的各种方法。

### 默认顺序

假设我们在别处定义了两个暂停函数，它们执行某种类似远程服务调用或计算的有用操作。我们只是假装他们是有用的，但实际上每个人只是为了这个例子的目的而延迟一秒钟：

```kotlin
suspend fun doSomethingUsefulOne(): Int {
    delay(1000L) // pretend we are doing something useful here
    return 13
}

suspend fun doSomethingUsefulTwo(): Int {
    delay(1000L) // pretend we are doing something useful here, too
    return 29
}
```

如果需要按顺序调用它们，我们该怎么做- 首先doSomethingUsefulOne ，然后 doSomethingUsefulTwo计算结果的总和？在实践中，如果我们使用第一个函数的结果来决定是否需要调用第二个函数或决定如何调用它，那么我们会这样做。

我们只是使用正常的顺序调用，因为协程中的代码与常规代码一样，默认情况下是连续的。以下示例通过测量执行两个挂起功能所需的总时间来演示它：

```kotlin
fun main(args: Array<String>) = runBlocking<Unit> {
    val time = measureTimeMillis {
        val one = doSomethingUsefulOne()
        val two = doSomethingUsefulTwo()
        println("The answer is ${one + two}")
    }
    println("Completed in $time ms")
}
```

它产生这样的东西：

```text
The answer is 42
Completed in 2017 ms
```

### 使用async来处理并发

如果doSomethingUsefulOne和doSomethingUsefulTwo的调用之间没有依赖关系，并且我们希望通过同时执行这两个方法来更快地得到答案？ 这是异步来帮助。

从概念上讲，异步就像启动。 它启动一个单独的协程，它是一个与所有其他协程同时工作的轻量级线程。 不同之处在于启动会返回一个Job，而不会带来任何结果值，而异步返回Deferred - 一个轻量级的非阻塞未来，表示稍后提供结果的承诺。 你可以使用延迟值的.await（）来得到最终的结果，但是Deferred也是一个Job，所以你可以根据需要取消它。

```kotlin
fun main(args: Array<String>) = runBlocking<Unit> {
    val time = measureTimeMillis {
        val one = async { doSomethingUsefulOne() }
        val two = async { doSomethingUsefulTwo() }
        println("The answer is ${one.await() + two.await()}")
    }
    println("Completed in $time ms")
}
```

它产生这样的东西：

```text
The answer is 42
Completed in 1017 ms
```

这是两倍的速度，因为我们同时执行两个协程。请注意，并发的协程总是显式的。

### 懒加载的async

有一个懒惰的选项，使用可选的开始参数与CoroutineStart.LAZY值异步异步。 它只有在需要某个结果的时候才启动协程，或者启动一个启动函数。 仅通过此选项运行以下与上一个不同的示例：

```kotlin
fun main(args: Array<String>) = runBlocking<Unit> {
    val time = measureTimeMillis {
        val one = async(start = CoroutineStart.LAZY) { doSomethingUsefulOne() }
        val two = async(start = CoroutineStart.LAZY) { doSomethingUsefulTwo() }
        println("The answer is ${one.await() + two.await()}")
    }
    println("Completed in $time ms")
}
```

它产生这样的东西：

```text
The answer is 42
Completed in 2017 ms
```

所以，我们又回到顺序执行，因为我们先开始等待one，然后开始等待two。这不是懒惰的预期用例。它被设计为lazy在计算值涉及暂停功能的情况下替代标准功能。

### 异步风格的函数

我们可以定义使用异步协程生成器异步调用doSomethingUsefulOne和doSomethingUsefulTwo的异步风格函数。 使用“Async”后缀的“async”前缀来命名这些函数是一种很好的风格，以突出显示这样的事实，即它们只启动异步计算，并且需要使用得到的延迟值来获得结果。

```kotlin
// asyncSomethingUsefulOne的结果类型是Deferred <Int> 
fun  asyncSomethingUsefulOne（） = async {
    doSomethingUsefulOne（）
}

// asyncSomethingUsefulTwo的结果类型是Deferred <Int> 
fun  asyncSomethingUsefulTwo（） = async {
    doSomethingUsefulTwo（）
}
```

请注意，这些asyncXXX函数不是 挂起函数。他们可以从任何地方使用。然而，它们的使用总是意味着它们的行为与调用代码的异步（这里意味着并发）执行。

以下示例显示了它们在协程之外的用法：

```kotlin
// note, that we don't have `runBlocking` to the right of `main` in this example
fun main(args: Array<String>) {
    val time = measureTimeMillis {
        // we can initiate async actions outside of a coroutine
        val one = asyncSomethingUsefulOne()
        val two = asyncSomethingUsefulTwo()
        // but waiting for a result must involve either suspending or blocking.
        // here we use `runBlocking { ... }` to block the main thread while waiting for the result
        runBlocking {
            println("The answer is ${one.await() + two.await()}")
        }
    }
    println("Completed in $time ms")
}
```

## 协程上下文和调度

协程总是在由 Kotlin标准库中定义的CoroutineContext类型的值表示的上下文中执行 。

协程的上下文是一组不同的元素。主要内容是我们前面看到的协程的Job，以及本节讨论的调度程序。

### 调度员和线程

协程上下文包括协程调度程序（参见CoroutineDispatcher），该协程确定相应协程执行的线程或线程。协程调度程序可以将协程执行限制在一个特定的线程中，调度它到一个线程池，或者让它运行非限制。

所有协程构建器（如启动和异步）都接受可选的 CoroutineContext 参数，该参数可用于为新协程和其他上下文元素显式指定调度程序。

试试下面的例子：

```kotlin
fun main(args: Array<String>) = runBlocking<Unit> {
    val jobs = arrayListOf<Job>()
    jobs += launch(Unconfined) { // not confined -- will work with main thread
        println("      'Unconfined': I'm working in thread ${Thread.currentThread().name}")
    }
    jobs += launch(coroutineContext) { // context of the parent, runBlocking coroutine
        println("'coroutineContext': I'm working in thread ${Thread.currentThread().name}")
    }
    jobs += launch(CommonPool) { // will get dispatched to ForkJoinPool.commonPool (or equivalent)
        println("      'CommonPool': I'm working in thread ${Thread.currentThread().name}")
    }
    jobs += launch(newSingleThreadContext("MyOwnThread")) { // will get its own new thread
        println("          'newSTC': I'm working in thread ${Thread.currentThread().name}")
    }
    jobs.forEach { it.join() }
}
```

它会产生以下输出（可能按不同的顺序）：

```text
      'Unconfined': I'm working in thread main
      'CommonPool': I'm working in thread ForkJoinPool.commonPool-worker-1
          'newSTC': I'm working in thread MyOwnThread
'coroutineContext': I'm working in thread main
```

我们在前面部分中使用的默认调度程序由DefaultDispatcher表示，这与当前实现中的CommonPool相同。 因此，launch {...}与launch（DefaultDispather）{...}相同，与launch（CommonPool）{...}相同。

父coroutineContext和Unconfined上下文之间的区别将在稍后显示。

请注意，newSingleThreadContext创建一个新的线程，这是一个非常昂贵的资源。 在真正的应用程序中，它必须被释放，不再需要时，使用close函数，或者存储在顶层变量中，并在整个应用程序中重用。

### 不受限和受限的调度器

Unconfined协程调度程序在调用者线程中启动协程，但仅在第一个暂停点之前。 暂停后，它将在被调用的暂停功能完全确定的线程中恢复。 协程不消耗CPU时间，也不更新限于特定线程的任何共享数据（如UI）时，无限制的分派器是合适的。

另一方面，通过CoroutineScope接口在任何协程的块内可用的coroutineContext属性是对此特定协程的上下文的引用。 这样，可以继承父上下文。 runBlocking协同程序的默认调度程序特别限于调用程序线程，因此继承它的作用是通过可预测的FIFO调度将执行限制在该线程中。

```kotlin
fun main(args: Array<String>) = runBlocking<Unit> {
    val jobs = arrayListOf<Job>()
    jobs += launch(Unconfined) { // not confined -- will work with main thread
        println("      'Unconfined': I'm working in thread ${Thread.currentThread().name}")
        delay(500)
        println("      'Unconfined': After delay in thread ${Thread.currentThread().name}")
    }
    jobs += launch(coroutineContext) { // context of the parent, runBlocking coroutine
        println("'coroutineContext': I'm working in thread ${Thread.currentThread().name}")
        delay(1000)
        println("'coroutineContext': After delay in thread ${Thread.currentThread().name}")
    }
    jobs.forEach { it.join() }
}
```

产生输出：

```text
      'Unconfined': I'm working in thread main
'coroutineContext': I'm working in thread main
      'Unconfined': After delay in thread kotlinx.coroutines.DefaultExecutor
'coroutineContext': After delay in thread main
```

因此，继承了runBlocking {...}的coroutineContext的协程在主线程中继续执行，而非限制的协程在延迟函数使用的默认执行程序线程中恢复。

### 调试协程和线程

协程可以在一个线程上挂起，并在具有Unconfined dispatcher的另一个线程上或使用默认的多线程调度程序恢复。即使使用单线程调度程序，也很难弄清楚协程在什么地方做什么，在什么地方，什么时候做什么。使用线程调试应用程序的常用方法是在每个日志语句的日志文件中打印线程名称。日志框架通常支持此功能。当使用协程时，单独的线程名称不会提供很多上下文，因此 kotlinx.coroutines包含调试工具以使其更容易。

使用-Dkotlinx.coroutines.debugJVM选项运行以下代码：

```kotlin
fun log(msg: String) = println("[${Thread.currentThread().name}] $msg")

fun main(args: Array<String>) = runBlocking<Unit> {
    val a = async(coroutineContext) {
        log("I'm computing a piece of the answer")
        6
    }
    val b = async(coroutineContext) {
        log("I'm computing another piece of the answer")
        7
    }
    log("The answer is ${a.await() * b.await()}")
}
```

有三个协程。主协程（＃1） - runBlocking一个和两个协程计算延迟值a（＃2）和b（＃3）。它们都是runBlocking在主线程的背景下执行的，并被限制在主线程中。这个代码的输出是：

```text
[main @coroutine#2] I'm computing a piece of the answer
[main @coroutine#3] I'm computing another piece of the answer
[main @coroutine#1] The answer is 42
```

日志函数在方括号中打印线程的名称，您可以看到它是主线程，但是当前正在执行的协程的标识符被附加到它。 打开调试模式时，此标识符将连续分配给所有创建的协程。

你可以在newCoroutineContext函数的文档中阅读更多关于调试工具的信息。

### 在线程之间跳跃

使用-Dkotlinx.coroutines.debugJVM选项运行以下代码：

```kotlin
fun log(msg: String) = println("[${Thread.currentThread().name}] $msg")

fun main(args: Array<String>) {
    newSingleThreadContext("Ctx1").use { ctx1 ->
        newSingleThreadContext("Ctx2").use { ctx2 ->
            runBlocking(ctx1) {
                log("Started in ctx1")
                run(ctx2) {
                    log("Working in ctx2")
                }
                log("Back to ctx1")
            }
        }
    }
}
```

它演示了几种新技术。 一个是使用带有明确指定的上下文的runBlocking，另一个是使用run函数来更改协程的上下文，同时仍然保留在同一个协程中，如下面的输出中所示：

```text
[Ctx1 @coroutine#1] Started in ctx1
[Ctx2 @coroutine#1] Working in ctx2
[Ctx1 @coroutine#1] Back to ctx1
```

请注意，该示例也使用Kotlin标准库中的use函数来释放在不再需要时使用newSingleThreadContext创建的线程。

### 上下文中的Job

协程的Job是其上下文的一部分。协程可以使用coroutineContext[Job]表达式从它自己的上下文中检索它：

```kotlin
fun main(args: Array<String>) = runBlocking<Unit> {
    println("My job is ${coroutineContext[Job]}")
}
```

在调试模式下运行时会产生类似的结果：

```text
My job is "coroutine#1":BlockingCoroutine{Active}@6d311334
```

因此，在CoroutineScope中的活动只是一个方便的快捷方式，用于coroutineContext [Job] !! isActive。

### 协程的Children

当协程的coroutineContext被用来启动另一个协程时，新协程的Job就成为了父协程的工作。 当父协程被取消时，它的所有孩子也被递归地取消。

```kotlin
fun main(args: Array<String>) = runBlocking<Unit> {
    // launch a coroutine to process some kind of incoming request
    val request = launch {
        // it spawns two other jobs, one with its separate context
        val job1 = launch {
            println("job1: I have my own context and execute independently!")
            delay(1000)
            println("job1: I am not affected by cancellation of the request")
        }
        // and the other inherits the parent context
        val job2 = launch(coroutineContext) {
            delay(100)
            println("job2: I am a child of the request coroutine")
            delay(1000)
            println("job2: I will not execute this line if my parent request is cancelled")
        }
        // request completes when both its sub-jobs complete:
        job1.join()
        job2.join()
    }
    delay(500)
    request.cancel() // cancel processing of the request
    delay(1000) // delay a second to see what happens
    println("main: Who has survived request cancellation?")
}
```

这个代码的输出是：

```text
job1: I have my own context and execute independently!
job2: I am a child of the request coroutine
job1: I am not affected by cancellation of the request
main: Who has survived request cancellation?
```

### 组合上下文

协程的上下文可以使用+运算符来组合。右侧的上下文替换左侧上下文的相关条目。例如，父协程的Job可以被继承，而调度器被替换：

```kotlin
fun main(args: Array<String>) = runBlocking<Unit> {
    // start a coroutine to process some kind of incoming request
    val request = launch(coroutineContext) { // use the context of `runBlocking`
        // spawns CPU-intensive child job in CommonPool !!! 
        val job = launch(coroutineContext + CommonPool) {
            println("job: I am a child of the request coroutine, but with a different dispatcher")
            delay(1000)
            println("job: I will not execute this line if my parent request is cancelled")
        }
        job.join() // request completes when its sub-job completes
    }
    delay(500)
    request.cancel() // cancel processing of the request
    delay(1000) // delay a second to see what happens
    println("main: Who has survived request cancellation?")
}
```

这个代码的预期结果是：

```text
job: I am a child of the request coroutine, but with a different dispatcher
main: Who has survived request cancellation?
```

### 家长的责任

父协程总是等待所有的孩子完成。Parent不必显式地跟踪它启动的所有子项，也不必使用Job.join在最后等待它们：

```kotlin
fun main(args: Array<String>) = runBlocking<Unit> {
    // launch a coroutine to process some kind of incoming request
    val request = launch {
        repeat(3) { i -> // launch a few children jobs
            launch(coroutineContext)  {
                delay((i + 1) * 200L) // variable delay 200ms, 400ms, 600ms
                println("Coroutine $i is done")
            }
        }
        println("request: I'm done and I don't explicitly join my children that are still active")
    }
    request.join() // wait for completion of the request, including all its children
    println("Now processing of the request is complete")
}
```

结果将是：

```text
request: I'm done and I don't explicitly join my children that are still active
Coroutine 0 is done
Coroutine 1 is done
Coroutine 2 is done
Now processing of the request is complete
```

### 命名协程进行调试

当通常情况下协程记录日志时自动分配ID是很好的，你只需要关联来自同一个协程的日志记录。但是，当协程与特定请求的处理或执行一些特定的后台任务相关联时，为了调试目的，最好明确地命名它。 CoroutineName上下文元素提供与线程名称相同的功能。在打开调试模式时，它将显示在执行此协程的线程名称中。

以下示例演示了这个概念：

```kotlin
fun log(msg: String) = println("[${Thread.currentThread().name}] $msg")

fun main(args: Array<String>) = runBlocking(CoroutineName("main")) {
    log("Started main coroutine")
    // run two background value computations
    val v1 = async(CoroutineName("v1coroutine")) {
        delay(500)
        log("Computing v1")
        252
    }
    val v2 = async(CoroutineName("v2coroutine")) {
        delay(1000)
        log("Computing v2")
        6
    }
    log("The answer for v1 / v2 = ${v1.await() / v2.await()}")
}
```

它使用-Dkotlinx.coroutines.debugJVM选项生成的输出类似于：

```text
[main @main#1] Started main coroutine
[ForkJoinPool.commonPool-worker-1 @v1coroutine#2] Computing v1
[ForkJoinPool.commonPool-worker-2 @v2coroutine#3] Computing v2
[main @main#1] The answer for v1 / v2 = 42
```

### 取消通过明确的工作

让我们把关于情境，孩子和工作的知识放在一起。假设我们的应用程序有一个生命周期对象，但是这个对象不是一个协程。例如，我们正在编写一个Android应用程序，并在Android活动的上下文中启动各种协同程序，以执行异步操作来获取和更新数据，执行动画等。所有这些协程必须在活动被销毁时被取消，以避免内存泄漏。

我们可以通过创建与我们活动的生命周期相关的Job实例来管理协同程序的生命周期。作业实例是使用Job（）工厂函数创建的，如以下示例所示。为了方便起见，我们可以编写启动（coroutineContext，parent = job），而不是使用启动（coroutineContext + job）表达式来明确使用父作业的事实。

现在，一个Job.cancel的调用取消了我们启动的所有子项。而且，Job.join等待所有的完成，所以我们也可以在这个例子中使用cancelAndJoin：

```kotlin
fun main(args: Array<String>) = runBlocking<Unit> {
    val job = Job() // create a job object to manage our lifecycle
    // now launch ten coroutines for a demo, each working for a different time
    val coroutines = List(10) { i ->
        // they are all children of our job object
        launch(coroutineContext, parent = job) { // we use the context of main runBlocking thread, but with our parent job
            delay((i + 1) * 200L) // variable delay 200ms, 400ms, ... etc
            println("Coroutine $i is done")
        }
    }
    println("Launched ${coroutines.size} coroutines")
    delay(500L) // delay for half a second
    println("Cancelling the job!")
    job.cancelAndJoin() // cancel all our coroutines and wait for all of them to complete
}
```

这个例子的输出是：

```text
Launched 10 coroutines
Coroutine 0 is done
Coroutine 1 is done
Cancelling the job!
```

正如你所看到的，只有前三个协程已经打印了一条消息，而另外一条协程被一次调用取消了job.cancelAndJoin()。所以我们在我们假设的Android应用程序中需要做的是在创建活动时创建父作业对象，将其用于子协程，并在活动被销毁时将其取消。我们不能join在Android生命周期中使用它们，因为它是同步的，但是在构建后端服务以确保有限的资源使用时，这种连接能力非常有用。

## 通道

Deferred values提供了在协程之间传递单个值的简便方法。Channel提供了一种方式来传递数据流。

### 通道基础

Channel在概念上与BlockingQueue非常相似。 一个关键的区别是，它有一个挂起的send方法而不是阻塞的put操作，有一个挂起的receive方法而不是一个阻塞的take操作。

```kotlin
fun main(args: Array<String>) = runBlocking<Unit> {
    val channel = Channel<Int>()
    launch {
        // this might be heavy CPU-consuming computation or async logic, we'll just send five squares
        for (x in 1..5) channel.send(x * x)
    }
    // here we print five received integers:
    repeat(5) { println(channel.receive()) }
    println("Done!")
}
```

这个代码的输出是：

```text
1
4
9
16
25
Done!
```

### 关闭通道和通过通道迭代

不像一个队列，一个通道可以关闭，表示没有更多的元素来了。 在接收端，使用常规的for循环来接收来自通道的元素是很方便的。

从概念上讲，结束就像发送一个特殊的密码令牌给该频道。 一旦接收到这个关闭标记，迭代就会停止，所以可以保证收到关闭之前的所有先前发送的元素：

```kotlin
fun main(args: Array<String>) = runBlocking<Unit> {
    val channel = Channel<Int>()
    launch {
        for (x in 1..5) channel.send(x * x)
        channel.close() // we're done sending
    }
    // here we print received values using `for` loop (until the channel is closed)
    for (y in channel) println(y)
    println("Done!")
}
```

### 建立通道生产者

协程正在产生一系列元素的模式是相当普遍的。 这是通常在并发代码中发现的生产者 - 消费者模式的一部分。 你可以把这样一个生产者抽象成一个以通道为参数的函数，但是这与常识是相反的，即结果必须从函数中返回。

有一个命名为produce的便利协程生成器，可以很容易地在生产者端做到这一点，并且扩展函数消耗了每一个，这就取代了消费者端的for循环：

```kotlin
fun produceSquares() = produce<Int> {
    for (x in 1..5) send(x * x)
}

fun main(args: Array<String>) = runBlocking<Unit> {
    val squares = produceSquares()
    squares.consumeEach { println(it) }
    println("Done!")
}
```

### 管道

管道是一个协程生产数据的模式，可能是无限的数据流：

```kotlin
fun produceNumbers() = produce<Int> {
    var x = 1
    while (true) send(x++) // infinite stream of integers starting from 1
}
```

另一个协程或协程正在消耗这个流，做一些处理，并产生一些其他的结果。在下面的例子中，数字只是平方：

```kotlin
fun square(numbers: ReceiveChannel<Int>) = produce<Int> {
    for (x in numbers) send(x * x)
}
```

主代码启动并连接整个管道：

```kotlin
fun main(args: Array<String>) = runBlocking<Unit> {
    val numbers = produceNumbers() // produces integers from 1 and on
    val squares = square(numbers) // squares integers
    for (i in 1..5) println(squares.receive()) // print first five
    println("Done!") // we are done
    squares.cancel() // need to cancel these coroutines in a larger app
    numbers.cancel()
}
```

我们不必在这个示例应用程序中取消这些协同程序，因为 协程就像守护程序线程，但是在一个更大的应用程序中，如果我们不再需要它，我们需要停止管道。或者，我们可以将管道协程作为 主协程的子程序运行，如以下示例所示。

### 素数与管道

让我们通过一个使用一系列协程来生成素数的例子，将管道推向极致。我们从无限的数字序列开始。这次我们引入一个明确的context参数并传递给生成器，以便调用者可以控制我们的协程运行的地方：

```kotlin
fun numbersFrom(context: CoroutineContext, start: Int) = produce<Int>(context) {
    var x = start
    while (true) send(x++) // infinite stream of integers from start
}
```

下面的管道阶段过滤一个输入的数字流，删除所有可以被给定质数整除的数字：

```kotlin
fun filter(context: CoroutineContext, numbers: ReceiveChannel<Int>, prime: Int) = produce<Int>(context) {
    for (x in numbers) if (x % prime != 0) send(x)
}
```

现在我们通过从2开始的数字流来构建我们的管道，从当前通道取一个素数，并为每个找到的素数启动新的流水线阶段：

```kotlin
numbersFrom(2) -> filter(2) -> filter(3) -> filter(5) -> filter(7) ... 
```

以下示例打印前十个素数，在主线程的上下文中运行整个管道。 由于所有的协程都是在coroutineContext中作为主要runBlocking协程的子程序启动的，因此我们不必保留我们已经启动的所有协程的明确列表。 我们使用cancelChildren扩展函数来取消所有的儿童协同程序。

```kotlin
fun main(args: Array<String>) = runBlocking<Unit> {
    var cur = numbersFrom(coroutineContext, 2)
    for (i in 1..10) {
        val prime = cur.receive()
        println(prime)
        cur = filter(coroutineContext, cur, prime)
    }
    coroutineContext.cancelChildren() // cancel all children to let main finish
}
```

这个代码的输出是：

```text
2
3
5
7
11
13
17
19
23
29
```

请注意，您可以使用标准库中的buildIterator协同构建器来构建相同的管道。 用buildIterator替换产品，用yield发送，接收下一个，使用Iterator的ReceiveChannel，摆脱上下文。 你也不需要runBlocking。 但是，如上所示使用通道的管道的好处是，如果在CommonPool上下文中运行，它实际上可以使用多个CPU内核。

无论如何，这是一个非常不切实际的方式来找到素数。 实际上，管道确实涉及到一些其他的暂停调用（比如异步调用远程服务），而且这些管道不能用buildSeqeunce / buildIterator构建，因为它们不允许任意挂起，而不同于完全异步的产品。

### 扇出

多个协程可以从同一个通道接收，在他们之间分配工作。 让我们从定期生成整数（每秒十个数字）的生产者协程开始：

```kotlin
fun produceNumbers() = produce<Int> {
    var x = 1 // start from 1
    while (true) {
        send(x++) // produce next
        delay(100) // wait 0.1s
    }
}
```

然后我们可以有几个处理器协程。在这个例子中，他们只是打印他们的ID和收到号码：

```kotlin
fun launchProcessor(id: Int, channel: ReceiveChannel<Int>) = launch {
    channel.consumeEach {
        println("Processor #$id received $it")
    }
}
```

现在让我们启动五个处理器，让他们工作几乎一秒钟。看看发生什么：

```kotlin
fun main(args: Array<String>) = runBlocking<Unit> {
    val producer = produceNumbers()
    repeat(5) { launchProcessor(it, producer) }
    delay(950)
    producer.cancel() // cancel producer coroutine and thus kill them all
}
```

>你可以在[这里]得到完整的代码

输出将类似于下一个，尽管接收每个特定整数的处理器ID可能不同：

```text
Processor #2 received 1
Processor #4 received 2
Processor #0 received 3
Processor #1 received 4
Processor #3 received 5
Processor #2 received 6
Processor #4 received 7
Processor #0 received 8
Processor #1 received 9
Processor #3 received 10
```

请注意，取消生产者协同程序会关闭其通道，从而最终终止对处理器协同进程的通道的迭代。

### 扇入

多个协程可以发送到同一个通道。例如，让我们有一个字符串的通道，以及一个暂停功能，该功能以指定的延迟重复地向该通道发送指定的字符串：

```kotlin
suspend fun sendString(channel: SendChannel<String>, s: String, time: Long) {
    while (true) {
        delay(time)
        channel.send(s)
    }
}
```

现在，让我们看看如果我们启动一些发送字符串的协程，会发生什么情况（在本例中，我们将它们作为主协程的子进程在主线程的上下文中启动）：

```kotlin
fun main(args: Array<String>) = runBlocking<Unit> {
    val channel = Channel<String>()
    launch(coroutineContext) { sendString(channel, "foo", 200L) }
    launch(coroutineContext) { sendString(channel, "BAR!", 500L) }
    repeat(6) { // receive first six
        println(channel.receive())
    }
    coroutineContext.cancelChildren() // cancel all children to let main finish
}
```

输出是：

```text
foo
foo
BAR!
foo
foo
BAR!
```

### 缓冲通道

目前显示的通道没有缓冲区。 当发送者和接收者彼此相遇（又名会合）时，无缓冲的信道传送元素。 如果发送先被调用，那么它被挂起直到接收被调用，如果接收被首先调用，它被挂起直到发送被调用。

Channel（）工厂函数和产品生成器都采用可选的容量参数来指定缓冲区大小。 缓冲区允许发送者在挂起之前发送多个元素，类似于具有指定容量的BlockingQueue，当缓冲区已满时该BlockingQueue被阻塞。

看看下面的代码的行为：

```kotlin
fun main(args: Array<String>) = runBlocking<Unit> {
    val channel = Channel<Int>(4) // create buffered channel
    val sender = launch(coroutineContext) { // launch sender coroutine
        repeat(10) {
            println("Sending $it") // print before sending each element
            channel.send(it) // will suspend when buffer is full
        }
    }
    // don't receive anything... just wait....
    delay(1000)
    sender.cancel() // cancel sender coroutine
}
```

它使用四个容量的缓冲通道打印“发送” 五次：

```text
Sending 0
Sending 1
Sending 2
Sending 3
Sending 4
```

前四个元素被添加到缓冲区，发送者在尝试发送第五个元素时挂起。

### 通道是公平的

对通道的发送和接收操作对于从多个协同程序调用的顺序是公平的。它们以先进先出的顺序被服务，例如要调用的第一个协程receive 获取元素。在以下示例中，两个协程“ping”和“pong”正从共享的“表”通道接收“球”对象。

```
data class Ball(var hits: Int)

fun main(args: Array<String>) = runBlocking<Unit> {
    val table = Channel<Ball>() // a shared table
    launch(coroutineContext) { player("ping", table) }
    launch(coroutineContext) { player("pong", table) }
    table.send(Ball(0)) // serve the ball
    delay(1000) // delay 1 second
    coroutineContext.cancelChildren() // game over, cancel them
}

suspend fun player(name: String, table: Channel<Ball>) {
    for (ball in table) { // receive the ball in a loop
        ball.hits++
        println("$name $ball")
        delay(300) // wait a bit
        table.send(ball) // send the ball back
    }
}
```

“ping”协程首先启动，所以是第一个接收球。即使“ping”协同程序在发回球后立即开始接收球，但球被“乒乓”协程接收，因为它已经在等待：

```text
ping Ball(hits=1)
pong Ball(hits=2)
ping Ball(hits=3)
pong Ball(hits=4)
```

请注意，由于正在使用的执行者的性质，有时渠道可能会产生看起来不公平的执行。详情请参阅此问题。

## 共享可变状态和并发

可以使用多线程调度程序（如默认的CommonPool）同时执行协程。它介绍了所有常见的并发问题。主要的问题是同步访问共享可变状态。协程领域的这个问题的一些解决方案与多线程世界中的解决方案类似，但是其他解决方案是独特的。

### 问题

让我们启动所有千次执行相同动作的协程（共执行一百万次）。我们还会测量完成时间，以便进一步比较：

```kotlin
suspend fun massiveRun(context: CoroutineContext, action: suspend () -> Unit) {
    val n = 1000 // number of coroutines to launch
    val k = 1000 // times an action is repeated by each coroutine
    val time = measureTimeMillis {
        val jobs = List(n) {
            launch(context) {
                repeat(k) { action() }
            }
        }
        jobs.forEach { it.join() }
    }
    println("Completed ${n * k} actions in $time ms")    
}
```

我们从一个非常简单的动作开始，使用多线程的CommonPool上下文来增加一个共享的可变变量。

```kotlin
var counter = 0

fun main(args: Array<String>) = runBlocking<Unit> {
    massiveRun(CommonPool) {
        counter++
    }
    println("Counter = $counter")
}
```

最后打印什么？打印“Counter = 1000000”是不太可能的，因为千位协程counter从多个线程同时递增而没有任何同步。

注意：如果你有一个拥有2个或更少的CPU的旧系统，那么你会一直看到1000000，因为 CommonPool在这种情况下，只有一个线程正在运行。要重现此问题，您需要进行以下更改：

```kotlin
val mtContext = newFixedThreadPoolContext(2, "mtPool") // explicitly define context with two threads
var counter = 0

fun main(args: Array<String>) = runBlocking<Unit> {
    massiveRun(mtContext) { // use it instead of CommonPool in this sample and below 
        counter++
    }
    println("Counter = $counter")
}
```

### Volatiles没有帮助

有一个常见的错误认为，使变量volatile解决并发问题。让我们试试看：

```kotlin
@Volatile // in Kotlin `volatile` is an annotation 
var counter = 0

fun main(args: Array<String>) = runBlocking<Unit> {
    massiveRun(CommonPool) {
        counter++
    }
    println("Counter = $counter")
}
```

这个代码工作得很慢，但是最后我们还是没有得到“Counter = 1000000”，因为volatile变量保证可线性化（这是“原子”的技术术语）读写相应的变量，但是不提供原子性更大的行动（在我们的情况下增加）。

### 线程安全的数据结构

对线程和协程都适用的通用解决方案是使用线程安全（也称为同步，线性或原子）数据结构，为需要在共享状态上执行的相应操作提供所有必要的同步。 在简单计数器的情况下，我们可以使用AtomicInteger类，它具有原子incrementAndGet操作：

```kotlin
var counter = AtomicInteger()

fun main(args: Array<String>) = runBlocking<Unit> {
    massiveRun(CommonPool) {
        counter.incrementAndGet()
    }
    println("Counter = ${counter.get()}")
}
```

这是解决这个问题的最快解决方案。它适用于简单的计数器，集合，队列和其他标准数据结构以及它们的基本操作。但是，它不容易扩展到复杂的状态或者复杂的操作，没有准备使用的线程安全的实现。

### 线程限制细粒度

线程约束是一种解决共享可变状态问题的方法，其中对特定共享状态的所有访问都局限于单个线程。它通常用于UI应用程序，其中所有UI状态都局限于单个事件派发/应用程序线程。通过使用
单线程的上下文，使用协程很容易：

```kotlin
val counterContext = newSingleThreadContext("CounterContext")
var counter = 0

fun main(args: Array<String>) = runBlocking<Unit> {
    massiveRun(CommonPool) { // run each coroutine in CommonPool
        run(counterContext) { // but confine each increment to the single-threaded context
            counter++
        }
    }
    println("Counter = $counter")
}
```

此代码工作非常缓慢，因为它执行细粒度的线程约束。每个单独的增量CommonPool使用运行块从多线程上下文切换到单线程上下文。

### 线程约束粗粒度

在实践中，线程限制是以大块执行的，例如大块状态更新业务逻辑被限制在单个线程中。下面的例子就是这样做的，在单线程上下文中运行每个协程来开始。

```kotlin
val counterContext = newSingleThreadContext("CounterContext")
var counter = 0

fun main(args: Array<String>) = runBlocking<Unit> {
    massiveRun(counterContext) { // run each coroutine in the single-threaded context
        counter++
    }
    println("Counter = $counter")
}
```

这现在工作得更快，并产生正确的结果。

### 相互排斥

相互排斥解决问题的办法是用一个永远不会同时执行的关键部分来保护共享状态的所有修改。 在阻塞的世界中，你通常使用synchronized或ReentrantLock。 协程的替代方法称为互斥。 它具有锁定和解锁功能来划定关键部分。 关键的区别是，Mutex.lock是一个暂停功能。 它不会阻塞一个线程。

还有一个方便表示mutex.lock（）的锁定扩展函数。 尝试{...}终于{mutex.unlock（）}模式：

```kotlin
val mutex = Mutex()
var counter = 0

fun main(args: Array<String>) = runBlocking<Unit> {
    massiveRun(CommonPool) {
        mutex.withLock {
            counter++        
        }
    }
    println("Counter = $counter")
}
```

在这个例子中的锁定是细粒度的，所以它付出了代价。但是，对于一些绝对必须定期修改某些共享状态的情况，这是一个不错的选择，但是这种状态是不存在的。

### Actors

Actors是一个协程组合，即约束和被封装到本协程的状态下，并与其他协同程序进行通信的信道的组合。一个简单的演员可以写成一个函数，但是具有复杂状态的演员更适合于一个类。

还有，它可以方便结合演员的邮箱通道进入其范围从接收消息并结合发送通道进入生成的作业对象，这样一个单一的参考演员可以围绕它的把手，可以提一个演员协程建设者。

使用actor的第一步是定义一个actor将要处理的消息类。 Kotlin的密封课很适合这个目的。我们使用IncCounter消息定义CounterMsg密封类来增加一个计数器和GetCounter消息来获得它的值。后者需要发送回复。这里使用一个CompletableDeferred通信原语来代表未来将被知道（通信）的单个值。

```kotlin
// Message types for counterActor
sealed class CounterMsg
object IncCounter : CounterMsg() // one-way message to increment counter
class GetCounter(val response: CompletableDeferred<Int>) : CounterMsg() // a request with reply
```

然后我们定义一个使用actor协程构建器启动actor的函数：

```kotlin
// This function launches a new counter actor
fun counterActor() = actor<CounterMsg> {
    var counter = 0 // actor state
    for (msg in channel) { // iterate over incoming messages
        when (msg) {
            is IncCounter -> counter++
            is GetCounter -> msg.response.complete(counter)
        }
    }
}
```

主要代码很简单：

```kotlin
fun main(args: Array<String>) = runBlocking<Unit> {
    val counter = counterActor() // create the actor
    massiveRun(CommonPool) {
        counter.send(IncCounter)
    }
    // send a message to get a counter value from an actor
    val response = CompletableDeferred<Int>()
    counter.send(GetCounter(response))
    println("Counter = ${response.await()}")
    counter.close() // shutdown the actor
}
```

（正确性）执行者自身的执行环境并不重要。执行者是一个协程，协程是按顺序执行的，因此将该状态限制到特定的协程可以解决共享可变状态的问题。

Actor在加载时比锁定效率更高，因为在这种情况下，它总是有工作要做，而且根本不需要切换到不同的上下文。

请注意，演员协同程序生成器是生成协程生成器的双重对象。一个参与者与它接收消息的通道相关联，而一个参与者与它发送元素的通道相关联。

## 选择表达式

选择表达式可以同时等待多个暂停功能，并选择 第一个可用的暂停功能。

### 从频道中选择

让我们有两个字符串的生产者：fizz和buzz。该fizz生产“菲斯”串每300毫秒：

```kotlin
fun fizz(context: CoroutineContext) = produce<String>(context) {
    while (true) { // sends "Fizz" every 300 ms
        delay(300)
        send("Fizz")
    }
}
```

而buzz每500毫秒生产一个"Buzz!"字符串：

```kotlin
fun buzz(context: CoroutineContext) = produce<String>(context) {
    while (true) { // sends "Buzz!" every 500 ms
        delay(500)
        send("Buzz!")
    }
}
```

使用接收暂停功能，我们可以接收任一从一个通道或其他。但是，选择的表达使我们能够从接收两个同时使用它 的onReceive条款：

```kotlin
suspend fun selectFizzBuzz(fizz: ReceiveChannel<String>, buzz: ReceiveChannel<String>) {
    select<Unit> { // <Unit> means that this select expression does not produce any result 
        fizz.onReceive { value ->  // this is the first select clause
            println("fizz -> '$value'")
        }
        buzz.onReceive { value ->  // this is the second select clause
            println("buzz -> '$value'")
        }
    }
}
```

让我们运行它七次：

```kotlin
fun main(args: Array<String>) = runBlocking<Unit> {
    val fizz = fizz(coroutineContext)
    val buzz = buzz(coroutineContext)
    repeat(7) {
        selectFizzBuzz(fizz, buzz)
    }
    coroutineContext.cancelChildren() // cancel fizz & buzz coroutines    
}
```

这段代码的结果是：

```text
fizz -> 'Fizz'
buzz -> 'Buzz!'
fizz -> 'Fizz'
fizz -> 'Fizz'
buzz -> 'Buzz!'
fizz -> 'Fizz'
buzz -> 'Buzz!'
```

### 选择关闭

所述的onReceive条款select当信道是封闭的和对应的失败 select抛出异常。当频道关闭时，我们可以使用onReceiveOrNull子句来执行特定的操作。以下示例还显示了select一个返回其所选子句的结果的表达式：

```kotlin
suspend fun selectAorB(a: ReceiveChannel<String>, b: ReceiveChannel<String>): String =
    select<String> {
        a.onReceiveOrNull { value -> 
            if (value == null) 
                "Channel 'a' is closed" 
            else 
                "a -> '$value'"
        }
        b.onReceiveOrNull { value -> 
            if (value == null) 
                "Channel 'b' is closed"
            else    
                "b -> '$value'"
        }
}
```

和4次生成“Hello”的通道a，4次生成“World”的通道b一起使用吧：

```kotlin
fun main(args: Array<String>) = runBlocking<Unit> {
    // we are using the context of the main thread in this example for predictability ... 
    val a = produce<String>(coroutineContext) {
        repeat(4) { send("Hello $it") }
    }
    val b = produce<String>(coroutineContext) {
        repeat(4) { send("World $it") }
    }
    repeat(8) { // print first eight results
        println(selectAorB(a, b))
    }
    coroutineContext.cancelChildren()    
}
```

这段代码的结果是非常有趣的，所以我们会详细分析一下：

```text
a -> 'Hello 0'
a -> 'Hello 1'
b -> 'World 0'
a -> 'Hello 2'
a -> 'Hello 3'
b -> 'World 1'
Channel 'a' is closed
Channel 'a' is closed
```

有几个意见可以做出来。

首先select是偏向于第一个条款。当几个子句同时选择时，其中的第一个被选中。在这里，两个频道都在不断地产生字符串，所以a频道，作为选择的第一个句子，获胜。但是，由于我们使用的是无缓冲的频道，因此a在发送调用时会暂时中止，并且也有机会b发送。

第二个观察是，当通道已经关闭时，onReceiveOrNull被立即选中。

### 选择发送

选择表达式有onSend子句，可以用于一个伟大的好处与偏见的选择性质的组合。

让我们来写一个整数生成器的例子，side当主通道上的消费者跟不上它时，它将它的值发送到一个通道：

```kotlin
fun produceNumbers(context: CoroutineContext, side: SendChannel<Int>) = produce<Int>(context) {
    for (num in 1..10) { // produce 10 numbers from 1 to 10
        delay(100) // every 100 ms
        select<Unit> {
            onSend(num) {} // Send to the primary channel
            side.onSend(num) {} // or to the side channel     
        }
    }
}
```

消费者会很慢，需要250毫秒来处理每个数字：

```kotlin
fun main(args: Array<String>) = runBlocking<Unit> {
    val side = Channel<Int>() // allocate side channel
    launch(coroutineContext) { // this is a very fast consumer for the side channel
        side.consumeEach { println("Side channel has $it") }
    }
    produceNumbers(coroutineContext, side).consumeEach { 
        println("Consuming $it")
        delay(250) // let us digest the consumed number properly, do not hurry
    }
    println("Done consuming")
    coroutineContext.cancelChildren()    
}
```

那么让我们看看会发生什么：

```text
Consuming 1
Side channel has 2
Side channel has 3
Consuming 4
Side channel has 5
Side channel has 6
Consuming 7
Side channel has 8
Side channel has 9
Consuming 10
Done consuming
```

### 选择延期值

可以使用onAwait子句选择延迟值。让我们从一个异步函数开始，它在一个随机延迟之后返回一个延迟的字符串值：

```kotlin
fun asyncString(time: Int) = async {
    delay(time.toLong())
    "Waited for $time ms"
}
```

让我们随机推迟十几个。

```kotlin
fun asyncStringsList(): List<Deferred<String>> {
    val random = Random(3)
    return List(12) { asyncString(random.nextInt(1000)) }
}
```

现在，主函数正在等待第一个完成，并计算仍然活动的延期值的数量。请注意，我们在这里使用了select表达式是Kotlin DSL 的事实，所以我们可以使用任意代码为它提供子句。在这种情况下，我们遍历延迟值的列表，onAwait为每个延期值提供子句。

```kotlin
fun main(args: Array<String>) = runBlocking<Unit> {
    val list = asyncStringsList()
    val result = select<String> {
        list.withIndex().forEach { (index, deferred) ->
            deferred.onAwait { answer ->
                "Deferred $index produced answer '$answer'"
            }
        }
    }
    println(result)
    val countActive = list.count { it.isActive }
    println("$countActive coroutines are still active")
}
```

输出是：

```text
Deferred 4 produced answer 'Waited for 128 ms'
11 coroutines are still active
```

### 切换延迟值的通道

让我们编写一个通道生成器函数，它消耗一个延迟字符串值的通道，等待每个接收到的延迟值，但只有等到下一个延迟值结束或通道关闭。这个例子把onReceiveOrNull和onAwait子句放在一起 select：

```kotlin
fun switchMapDeferreds(input: ReceiveChannel<Deferred<String>>) = produce<String> {
    var current = input.receive() // start with first received deferred value
    while (isActive) { // loop while not cancelled/closed
        val next = select<Deferred<String>?> { // return next deferred value from this select or null
            input.onReceiveOrNull { update ->
                update // replaces next value to wait
            }
            current.onAwait { value ->  
                send(value) // send value that current deferred has produced
                input.receiveOrNull() // and use the next deferred from the input channel
            }
        }
        if (next == null) {
            println("Channel was closed")
            break // out of loop
        } else {
            current = next
        }
    }
}
```

为了测试它，我们将使用一个简单的异步函数，在指定的时间之后解析为指定的字符串：

```kotlin
fun asyncString(str: String, time: Long) = async {
    delay(time)
    str
}
```

主函数只是启动一个协程来打印结果switchMapDeferreds并发送一些测试数据给它：

```kotlin
fun main(args: Array<String>) = runBlocking<Unit> {
    val chan = Channel<Deferred<String>>() // the channel for test
    launch(coroutineContext) { // launch printing coroutine
        for (s in switchMapDeferreds(chan)) 
            println(s) // print each received string
    }
    chan.send(asyncString("BEGIN", 100))
    delay(200) // enough time for "BEGIN" to be produced
    chan.send(asyncString("Slow", 500))
    delay(100) // not enough time to produce slow
    chan.send(asyncString("Replace", 100))
    delay(500) // give it time before the last one
    chan.send(asyncString("END", 500))
    delay(1000) // give it time to process
    chan.close() // close the channel ... 
    delay(500) // and wait some time to let it finish
}
```

这个代码的结果是：

```text
BEGIN
Replace
END
Channel was closed
```

### 进一步阅读

* [Guide to UI programming with coroutines](ui/coroutines-guide-ui.md)
* [Guide to reactive streams with coroutines](reactive/coroutines-guide-reactive.md)
* [Coroutines design document (KEEP)](https://github.com/Kotlin/kotlin-coroutines/blob/master/kotlin-coroutines-informal.md)
* [Full kotlinx.coroutines API reference](http://kotlin.github.io/kotlinx.coroutines)