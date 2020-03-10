# 常见需求

## 空间维度

## 时间维度

- 点击事件防抖

1. debounce 

时间间隔小于指定阈值的最大子序列的最后一个元素

```kotlin
RxView.clicks(btn)
    .debounce(500, TimeUnit.MILLISECONDS)
    .observerOn(AndroidSchedulers.mainThread())
    .subscribe(o -> {
        // handle clicks
    })
```

2. throttleFirst

时间间隔小于指定阈值的最大子序列的第一个元素

```kotlin
RxView.clicks(btn)
    .throttleFirst(500, TimeUnit.MILLISECONDS)
    .observerOn(AndroidSchedulers.mainThread())
    .subscribe(o -> {
        // handle clicks
    })
```

3. throttleLast

时间间隔小于指定阈值的最大子序列的最后一个元素，同debounce

```kotlin
RxView.clicks(btn)
    .throttleLast(500, TimeUnit.MILLISECONDS)
    .observerOn(AndroidSchedulers.mainThread())
    .subscribe(o -> {
        // handle clicks
    })
```

- 搜索提示 （事件防抖 && 取消上一个无效请求）

```kotlin
RxTextView.textChanges(input)
    .debounce(300, TimeUnit.MILLISECONDS)
    .switchMap(text -> api.queryKeyword(text.toString()))
    .observeOn(AndroidSchedulers.mainThread())
    .subscribe(results -> {
        // handle results
    });
```

- 双击事件

```kotlin
Observable<Long> clicks = RxView.clicks(btn)
    .map(o -> System.currentTimeMillis())
    .share()
    
clicks.zipWith(clicks.skip(1), (t1, t2) -> t2 - t1)
    .filter(interval -> interval < 500)
    .subscribe(o -> {
        // handle double click
    });
```

- 双击事件防抖

```kotlin
Observable<Object> clicks = RxView.clicks(btn).share()

clicks.buffer(clicks.debounce(500, TimeUnit.MILLISECONDS))
    .filter(events -> events.size >= 2)
    .observeOn(AndroidSchedulers.mainThread())
    .subscribe(o -> {
        // handle double click
    });
```

- 点赞事件 （事件防抖 && 的确是相反事件才发送请求）

```kotlin
Observable<Boolean> debounced = RxView.clicks(btn_like)
    .debounce(1000, TimeUnit.MILLISECONDS);
debounced.zipWith(
    debounced.startWith(like),
    (last, current) -> last == current ? new Pair<>(false, false) : new Pair<>(true, current)
)
    .flatMap(pair -> pair.first ? Observable.just(pair.second) : Observable.empty())
    .subscribe(like -> {
        if (like) {
            sendCancelLikeRequest(postId);
        } else {
            sendLikeRequest(postId);
        }
    });
```

- 输入框清除按钮控制 （有焦点且有文本才显示清除按钮）

```kotlin
Observables.combineLatest(et_user_name.focusChanges(), et_user_name.afterTextChangeEvents()) { 
    focused, event -> focused && event.editable.length > 0
}.subscribe(iv_clear_user_name.visibility());
```

- 输入框清除按钮控制 && 多输入框内容合法判断

```kotlin
val userNameEvent = et_user_name.afterTextChangeEvents().share()
val userPasswordEvent = et_user_password.afterTextChangeEvents().share()

Observables.combineLatest(et_user_name.focusChanges(), userNameEvent) { 
    focused, event -> focused && event.editable.length > 0
}.subscribe(iv_clear_user_name.visibility())

Observables.combineLatest(et_user_password.focusChanges(), userPasswordEvent) { 
    focused, event -> focused && event.editable.length > 0
}.subscribe(iv_clear_user_password.visibility())

Observables.combineLatests(userNameEvent, userPasswordEvent) {
    nameEvent, passwordEvent -> nameEvent.editable.length > 0 && passwordEvent.editable.length > 0
}.subscribe(btn_login::setEnable)
```

- 多种类型的RecyclerView

```kotlin
val typesObservable = networkApi.types()
val itemObservableA = networkApi.a().startWith(mutableListOf())
val itemObservableB = networkApi.b().startWith(mutableListOf())
val itemObservableC = networkApi.c().startWith(mutableListOf())

typesObservable
    .flatMap(types -> Observable.fromIterable(types)
        .map(type -> {
            when (type) {
                "a" -> return itemObservableA
                "b" -> return itemObservableB
                "c" -> return itemObservableC
                else -> throw new IllegalArgumentException()
            }
        })
        .collectInto(mutableListOf(), List::add)
        .toObservable()
    )
    .flatMap(requestObservables -> Observable.combineLates(requestObservables, objects -> objects))
    .flatMap(objects -> Observable.fromArray(objects)
        .collectInto(mutableListOf()) { (items, o) -> items.addAll(o as MutableList<Item>)}
        .toObservable()
    )
    .subscribeOn(Schedulers.io())
    .observeOn(AndroidSchedulers.mainThread())
    .subscribe(data -> {
        adapter.setData(data)
        adapter.notifyDataSetChanged()
    });
```

## 复杂维度