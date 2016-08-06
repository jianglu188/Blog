---
title: 用RxJava实现事件总线
date: 2016-07-19 23:29:06
categories: java
tags: Rxjava
---
事件总线可以使Android中各组件之间的通信变得简单，最重要的是可以**解耦**！
目前大多数开发者使用EventBus或者Otto作为事件总线通信库，对于RxJava使用者来说，RxJava也可以轻松实现事件总线，因为它们都依据于观察者模式。

不多说，上代码

{% codeblock lang:java %}
public class RxBus {
    private static volatile RxBus sInstance;

    private final Subject<Object, Object> subject;

    private RxBus() {
        subject = new SerializedSubject<>(PublishSubject.create());
    }

    private static RxBus getInstance() {
        RxBus rxBus = sInstance;
        if (sInstance == null) {
            synchronized (RxBus.class) {
                rxBus = sInstance;
                if (sInstance == null) {
                    rxBus = new RxBus();
                    sInstance = rxBus;
                }
            }
        }
        return rxBus;
    }

    //发送一个新的事件
    public void post(Object o) {
        subject.onNext(o);
    }

    //返回指定类型的Observable
    public <T> Observable<T> toObservable(Class<T> eventType) {
        return subject.ofType(eventType);
    }
}
{% endcodeblock %}


注：
1、[Subject](http://reactivex.io/documentation/subject.html)同时充当了Observer和Observable的角色，Subject是非线程安全的，要避免该问题，需要将Subject转换为一个 SerializedSubject ，上述RxBus类中把线程非安全的PublishSubject包装成线程安全的Subject。

2、PublishSubject只会把在订阅发生的时间点之后来自原始Observable的数据发射给观察者。

3、ofType操作符只发射指定类型的数据