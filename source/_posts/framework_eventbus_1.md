---
title: android开源框架——eventbus（1）
tags: [android, 框架]
---
![img](/images/framework_eventbus_1/EventBus-Publish-Subscribe.png)
<!--more-->
在android原生应用开发过程中，我们常常会用大量的handler, broadcast等等进行线程、组件、activity间通讯，代码复杂度、耦合度、维护成本越来越高。我们怎样将这些繁杂的数据交互进行统一呢，eventbus就应运而生，使用简单，开销小，代码偶合度低，是典型的观察者模式，从图上可以看出来eventbus就负责对event进行调度和路由。

源码地址：https://github.com/greenrobot/EventBus

## 简单上手

### 1、先来一发POJO
``` java
public class TestEvent
{
    String msg = "";
    public TestEvent(String message)
    {
         msg = message;
    }

    public String getMsg()
    {
        return this.msg;
    }
}
```
   这个POJO我们就可以理解为是传递的事件，这里为什么要使用POJO，因为他只负责进行一个暂时性的数据存放，并不具有业务处理，也会与其它类有继承和被继承关系，更不会被框架侵入，它很纯洁。

### 2、注册接收
``` java
   // 注册接收
   EventBus.getDefault().register(this);
```

### 3、事件发送
   在一个子线程中发送一条消息给UI主纯种
``` java
    public class TestThread implements Runnable
    {
        @Override
        public void run()
        {
            EventBus.getDefault().post(new TestEvent("sub thread send to ui thread msg: zoom bong bong bong!"));
        }
    }
```
### 4、事件接收
   在注册接收的组件里添加一个接收事件的方法，老版本中是通过不同的方法名来进行接收处理类型的，但新版本中通过注解的方式来实现不同的接收方法，可以在SubscriberMethodFinder类中findUsingReflectionInSingleClass方法看到是通过解析注解来区分不同的处理方式的，与方法名无关，所以可以自己定义不同的接收方法。这样做的好处是，老版本中对于不同的接收处理逻辑需要写多个不同的事件来实现接收方法的重写，否则不同的接收逻辑将混淆。新版本要以直接用方法名来区分不同的处理逻辑，代码可以更清晰。
   老版本：
 ``` java
   public void onEvent(Event event)
   {
       tv_return.setText(event.getMsg());
   }

   public void onEvent(Event1 event)
   {
       tv_return.setText(event.getMsg());
   }

   public void onEventMainThread(Event event)
   {
       ...
   }

   public void onEventBackground(Event event)
   {
       ...
   }

   public void onEventAsync(Event event)
   {
       ...
   }
```

   新版本：
``` java
   @Subscribe(threadMode = ThreadMode.MainThread) //在ui线程执行
   public void onEventFromThread(TestEvent event)
   {
        tv_return.setText(event.getMsg());
   }

   @Subscribe(threadMode = ThreadMode.MainThread) //在ui线程执行
   public void onEventFromThread1(TestEvent1 event)
   {
        tv_return.setText(event.getMsg());
   }

   @Subscribe(threadMode = ThreadMode.BackgroundThread) //如果发送事件在主线程中，则创建新子线程来执行；如果发送事件在子线程中，则直接在该线程中执行
   public void onEvent(TestEvent event)
   {
        ...
   }

   @Subscribe(threadMode = ThreadMode.Async) //创建新的子线程来执行
   public void onEvent(TestEvent event)
   {
        ...
   }

   @Subscribe(threadMode = ThreadMode.PostThread) //与发送事件线程在同一线程执行
   public void onEvent(TestEvent event)
   {
        ...
   }
```

### 5、注销接收
``` java
   EventBus.getDefault().unregister(this);
```



## 完整代码：
### 主activity
``` java
package com.visomic.eventbustest;

import android.app.Activity;
import android.os.Bundle;
import android.view.View;
import android.widget.Button;
import android.widget.TextView;

import org.greenrobot.eventbus.EventBus;
import org.greenrobot.eventbus.Subscribe;
import org.greenrobot.eventbus.ThreadMode;


public class MainActivity extends Activity
{
    private Button btn_go;
    private TextView tv_return;
    private TextView tv_return1;
    @Override
    protected void onCreate(Bundle savedInstanceState)
    {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }

    @Override
    protected void onResume()
    {
        super.onResume();
        // 注册接收
        EventBus.getDefault().register(this);


        btn_go = (Button) findViewById(R.id.btn_go);
        tv_return = (TextView) findViewById(R.id.tv_return);
        tv_return1 = (TextView) findViewById(R.id.tv_return1);
        btn_go.setOnClickListener(new View.OnClickListener()
        {
            @Override
            public void onClick(View v)
            {
                new Thread(new TestThread()).start();
                new Thread(new TestThread1()).start();
            }
        });
    }

    @Override
    protected void onDestroy()
    {
        super.onDestroy();
        // 注销接收
        EventBus.getDefault().unregister(this);
    }

    @Subscribe(threadMode = ThreadMode.MAIN) //在ui线程执行
    public void onEventFromThread(TestEvent event)
    {
        tv_return.setText(event.getMsg());
    }

    @Subscribe(threadMode = ThreadMode.MAIN) //在ui线程执行
    public void onEventFromThread1(TestEvent1 event)
    {
        tv_return1.setText(event.getMsg());
    }

    public class TestThread implements Runnable
    {
        @Override
        public void run()
        {
            EventBus.getDefault().post(new TestEvent("sub thread send to ui thread msg: zoom bong bong bong!"));
        }
    }

    public class TestThread1 implements Runnable
    {
        @Override
        public void run()
        {
            EventBus.getDefault().post(new TestEvent1("sub thread1 send to ui thread msg: zoom bong bong bong!"));
        }
    }
}

```

### 主布局文件
``` xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">
    <Button
        android:id="@+id/btn_go"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="go"/>
    <TextView
        android:id="@+id/tv_return"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="haha"/>
    <TextView
        android:id="@+id/tv_return1"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="haha"/>
</LinearLayout>


```

### 事件POJO
``` java
public class TestEvent
{
    String msg = "";
    public TestEvent(String message)
    {
        msg = message;
    }

    public String getMsg()
    {
        return this.msg;
    }
}


public class TestEvent1
{
    String msg = "";
    public TestEvent1(String message)
    {
        msg = message;
    }

    public String getMsg()
    {
        return this.msg;
    }
}
```
