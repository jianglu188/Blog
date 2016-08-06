---
title: EventBus入门
date: 2016-08-06 14:32:18
tags:
---

EventBus
========
EventBus is a publish/subscribe event bus optimized for Android.<br/>
<img src="EventBus-Publish-Subscribe.png" width="500" height="187"/>

EventBus...

 * simplifies the communication between components
    * decouples event senders and receivers
    * performs well with Activities, Fragments, and background threads
    * avoids complex and error-prone dependencies and life cycle issues
 * makes your code simpler
 * is fast
 * is tiny (~50k jar)
 * is proven in practice by apps with 100,000,000+ installs
 * has advanced features like delivery threads, subscriber priorities, etc.

 EventBus是一个基于观察者模式的事件发布/订阅框架，开发者可以通过极少的代码去实现多个模块之间的通信，而不需要以层层传递接口或者回调的形式去单独构建通信桥梁。从而降低因多重回调导致的模块间强耦合，同时避免产生大量内部类。它拥有使用方便，性能高，接入成本低和支持多线程的优点，实乃模块解耦、代码重构必备良药

添加EventBus支持
--------------

Gradle:
```gradle
compile 'org.greenrobot:eventbus:3.0.0'
```
[download EventBus from Maven Central](http://search.maven.org/#search%7Cga%7C1%7Cg%3A%22de.greenrobot%22%20AND%20a%3A%22eventbus%22)

EventBus 使用示例
---------------
1. 定义事件:

    ```java  
public class LoadEvent {
}
```

2. onCreate中注册事件
   ```java
   EventBus.getDefault().register(this);
```

3. onDestroy中取消注册

   ```java
   EventBus.getDefault().unregister(this);
```

4. 订阅事件
    ```java
    @Subscribe(threadMode = ThreadMode.MAIN, priority = 0, sticky = false)
    public void onLoadEvent(LoadEvent event) {
        Log.d(TAG, "onLoadEvent");
    }
```
 * threadMode
    * POSTING 在调用post所在的线程执行回调，直接运行。
    * MAIN 在主线程回调，如果post所在线程为主线程则直接执行，否则则通过mainThreadPoster来调度。
    * BACKGROUND 如果post所在线程为非UI线程则直接执行，否则则通过backgroundPoster来调度，这里只适合执行时间比较短的任务。
    * ASYNC（交给线程池来管理）：直接通过asyncPoster调度。
  * sticky 是否监听黏性事件      
    >If true, delivers the most recent sticky event (posted with
     * {@link EventBus#postSticky(Object)}) to this subscriber (if event available).


5. 发布事件
   ```java
   EventBus.getDefault().post(new LoadEvent());
```

6. 完整示例
```java
public class MainActivity extends AppCompatActivity {
    public static final String TAG = "MainActivity";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        EventBus.getDefault().register(this);

        Button start = (Button) findViewById(R.id.start_load);
        start.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                EventBus.getDefault().post(new LoadEvent());
            }
        });
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        EventBus.getDefault().unregister(this);
    }

    @Subscribe(threadMode = ThreadMode.MAIN, priority = 0, sticky = false)
    public void onLoadEvent(LoadEvent event) {
        Log.d(TAG, "onLoadEvent");
    }

    public static class LoadEvent {

    }
}
```

EventBus的索引加速
-------



用RxJava实现事件总线
-----------------
