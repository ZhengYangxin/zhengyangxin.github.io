---
title: RxJava的使用及源码分析
copyright: true
date: 2018-03-18 15:25:44
tags: [RxJava]
category:
---

### 基于Rxjava-2.1.10版本源码分析
从开源文档范例开始分析
#### 范例一.Flowable.just输出HelloWord
```
Flowable.just("Hello world").subscribe(new Consumer<String>() {
    @Override
    public void accept(String s) throws Exception {

    }
});
```
1.从just方法开始,首先可以看到三个注解方法
```
@CheckReturnValue  //检查返回值
@BackpressureSupport(BackpressureKind.FULL)  //支持的背压方式
@SchedulerSupport(SchedulerSupport.NONE) //调度方式,即处理事件的方式IO数据流，新开一个线程等。
public static <T> Flowable<T> just(T item) { //方法参数是个泛型，
    ObjectHelper.requireNonNull(item, "item is null");
    //返回值是个Flowable对象
    return RxJavaPlugins.onAssembly(new FlowableJust<T>(item)); 
}
```
<!-- more -->
2.方法第一行是判空操作在很多地方都使用到了，查看方法不为空则返回原对象，否则，抛出异常NullPointerException
```
public static <T> T requireNonNull(T object, String message) {
        if (object == null) {
            throw new NullPointerException(message);
        }
        return object;
    }
```
3.方法第三行中的看new FlowableJust<T>(item)创建的实例对象
```
public final class FlowableJust<T> extends Flowable<T> implements ScalarCallable<T> {
    private final T value; //final变量，赋值后不能改变
    public FlowableJust(final T value) {
        this.value = value;
    }
    //重写了Flowable的subscribeActual方法，传入了观察者与发射内容构造一个订阅对象
    //由观察者去订阅这个对象
    @Override
    protected void subscribeActual(Subscriber<? super T> s) {
        s.onSubscribe(new ScalarSubscription<T>(s, value));
    }

    @Override
    public T call() {
        return value;
    }
}
```
4.然后看RxJavaPlugins.onAssembly()方法
```
@SuppressWarnings({ "rawtypes", "unchecked" })
    @NonNull
    public static <T> Flowable<T> onAssembly(@NonNull Flowable<T> source) {
        //在这里钩子函数为null，即直接返回source
        Function<? super Flowable, ? extends Flowable> f = onFlowableAssembly;
        if (f != null) {
            return apply(f, source);
        }
        return source;
    }
```
5.所以Flowable.just("Hello world")只是生成了一个Flowable对象。接着看subscribe()方法。实现了Consumer接口
```
public interface Consumer<T> {
    /**
     * 回调函数accept去消费这个传入值，
     * Consume the given value.
     * @param t the value
     * @throws Exception on error
     */
    void accept(T t) throws Exception;
}
```
6.接着看subscribe()方法
```
@CheckReturnValue
@BackpressureSupport(BackpressureKind.UNBOUNDED_IN)
@SchedulerSupport(SchedulerSupport.NONE)
public final Disposable subscribe(Consumer<? super T> onNext) {
    //onNext即要去消费的接口对象
    //其他传入参数为默认的onError，onComplete接口回调，和最大数量的订阅对象
    return subscribe(onNext, Functions.ON_ERROR_MISSING,
            Functions.EMPTY_ACTION, FlowableInternalHelper.RequestMax.INSTANCE);
}
```
7.然后去看第二个subscribe()内部方法
```
@CheckReturnValue
@BackpressureSupport(BackpressureKind.SPECIAL)
@SchedulerSupport(SchedulerSupport.NONE)
public final Disposable subscribe(Consumer<? super T> onNext, Consumer<? super Throwable> onError,
        Action onComplete, Consumer<? super Subscription> onSubscribe) {
    ObjectHelper.requireNonNull(onNext, "onNext is null");
    ObjectHelper.requireNonNull(onError, "onError is null");
    ObjectHelper.requireNonNull(onComplete, "onComplete is null");
    ObjectHelper.requireNonNull(onSubscribe, "onSubscribe is null");

    //上面一系列的判空操作之后，将传入参数组合成一个LambdaSubscriber
    LambdaSubscriber<T> ls = new LambdaSubscriber<T>(onNext, onError, onComplete, onSubscribe);

    //调用了第三个subscribe方法
    subscribe(ls);

    return ls;
}
```
8、先来看一下LambdaSubscriber对象，除了对传入参数赋值之外，内部重写了熟悉的onSubscribe、onNext()、onError()、onComplete()三个方法。从6.可以看出在这四个方法中除了onNext()是我们传入的，其他都是框架默认的。源码过长，不copy了。
9.接着继续看第三个subscribe(ls)方法
```
@BackpressureSupport(BackpressureKind.SPECIAL)
@SchedulerSupport(SchedulerSupport.NONE)
@Beta
public final void subscribe(FlowableSubscriber<? super T> s) {
    ObjectHelper.requireNonNull(s, "s is null");
    try {
        // 传入当前要观察的对象和处理方法对象构建了一个观察者对象
        Subscriber<? super T> z = RxJavaPlugins.onSubscribe(this, s);

        ObjectHelper.requireNonNull(z, "Plugin returned null Subscriber");

        //在这传入一个观察者对象
        subscribeActual(z);
    } catch (NullPointerException e) { // NOPMD
        throw e;
    } catch (Throwable e) {
        Exceptions.throwIfFatal(e);
        // can't call onError because no way to know if a Subscription has been set or not
        // can't call onSubscribe because the call might have set a Subscription already
        RxJavaPlugins.onError(e);

        NullPointerException npe = new NullPointerException("Actually not, but can't throw other exceptions due to RS");
        npe.initCause(e);
        throw npe;
    }
}
```
10.接着看subscribeActual(),似曾相识在3.中FlowableJust重写了父类的subscribeActual()方法，所以最后有回来了
```
@Override
protected void subscribeActual(Subscriber<? super T> s) {
    //1.传入观察者对象和要发射的值创建了一个标量的订阅对象
    //2.观察者对象订阅了这个标量的订阅对象
    s.onSubscribe(new ScalarSubscription<T>(s, value));
}
```
11.先看ScalarSubscription方法，可以看到当调用了request(n)的方法时onNext方法就会被调用,
```
public ScalarSubscription(Subscriber<? super T> subscriber, T value) {
    this.subscriber = subscriber;
    this.value = value;
}

@Override
public void request(long n) {
    if (!SubscriptionHelper.validate(n)) {
        return;
    }
    if (compareAndSet(NO_REQUEST, REQUESTED)) {
        Subscriber<? super T> s = subscriber;
        //这里就是LambdaSubscriber中的onNext，也是我们传入的onNext会去回调的地方
        s.onNext(value);
        if (get() != CANCELLED) {
            s.onComplete();
        }
    }

}
```
12.所以我们要看request()在哪里调用，接着看继续看s.onSubscribe()方法，它就是LambdaSubscriber中的onSubscribe()方法
```
@Override
public void onSubscribe(Subscription s) {
    if (SubscriptionHelper.setOnce(this, s)) {
        try {
            //onSubscribe对象是在3.中传入的FlowableInternalHelper.RequestMax.INSTANCE，并调用了accept方法
            onSubscribe.accept(this);
        } catch (Throwable ex) {
            Exceptions.throwIfFatal(ex);
            s.cancel();
            onError(ex);
        }
    }
}
```
13.接着继续看FlowableInternalHelper.RequestMax.INSTANCE。
```
public enum RequestMax implements Consumer<Subscription> {
    //枚举法创建的单例
    INSTANCE;
    @Override
    public void accept(Subscription t) throws Exception {
        //可以看到在这里调用了request方法
        t.request(Long.MAX_VALUE);
    }
}
```
14.接着可以回到11.ScalarSubscription类中，在request方法中就调用了LambdaSubscriber的onNext*()方法并传入了value值
15.接着再去看LambdaSubscriber类中的onNext方法
```
@Override
    public void onNext(T t) {
        if (!isDisposed()) {
            try {
                //在这里回调了我们当初我们传入的实现对象onNext的accept方法，
                onNext.accept(t);
            } catch (Throwable e) {
                Exceptions.throwIfFatal(e);
                get().cancel();
                onError(e);
            }
        }
    }
```
16.走完onNext方法后，继续看11.它会继续走s.onComplete()方法，这个方法也是默认的Functions.EMPTY_ACTION
```
static final class EmptyAction implements Action {
        @Override
        public void run() { 
            //空方法
        }

        @Override
        public String toString() {
            return "EmptyAction";
        }
    }
```
17.至于onError方法会14.在onNext()发生异常时去调用，且在这里也是默认传入的Functions.ON_ERROR_MISSING
```
static final class OnErrorMissingConsumer implements Consumer<Throwable> {
    @Override
    public void accept(Throwable error) {
        RxJavaPlugins.onError(new OnErrorNotImplementedException(error));
    }
}
```
#### 最后范例一just的整个流程就是这样，从流程看出just操作符是真的很简单的，只是实现了一个对象的传递，内部也只是对我们要实现的onNext()进行了回调处理，因此其实对其他onError和onComplete我们也可以自定义处理方式