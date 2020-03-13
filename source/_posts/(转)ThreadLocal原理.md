---
title: (转)ThreadLocal原理
date: 2018-9-30 18:53:12
updated: 2018-9-30 18:53:12
tags:
  - 多线程
categories: Java
---
# 【多线程】ThreadLocal原理

## 使用

```java
ThreadLocal<String> threadLocalA= new ThreadLocal<String>(); 

threadLocalA.set(new String("A")); 

String str = threadLocalA.get(); 
```
<!-- more -->
## 原理

在每个线程的内部有个数据结构为`Map`的`threadLocals`变量，以`<ThreadLocal,Value>`的形式保存着线程变量和其对应的值。

当使用`set()`方法时：

1. 获取到当前线程的`threadLocals`，类型为`Map`
2. 将这值放到这个`Map`结构的变量中，key为ThreadLocal对象，value为所有存放的值

当使用`get()`方法时：

1. 获取到当前线程的`threadLocals`，类型为`Map`。
2. 以`ThreadLocal`对象为Map的key获取到它的`value`值。

因为ThreadLocal对象作为Map的key，所以一个ThreadLocal对象只能存放一个值，当存放多个时，会将新值覆盖旧值。

## 源码

数据结构：

```java
public void set(T value) { 

Thread t = Thread.currentThread();

ThreadLocalMap map = getMap(t);//当前线程为入参，获取当前线程的threadLocals变量 

if (map != null) 

//入参为this，也就是说key为ThreadLocal对象

map.set(this, value);

else

createMap(t, value);

}



public T get() { 

Thread t = Thread.currentThread();

ThreadLocalMap map = getMap(t);//当前线程为入参，获取当前线程的threadLocals 

if (map != null) { 

//入参为this，也就是说key为ThreadLocal

ThreadLocalMap.Entry e = map.getEntry(this); 

if (e != null) { 

@SuppressWarnings("unchecked")

T result = (T)e.value;

return result; 

}

}

return setInitialValue(); 

}

ThreadLocalMap getMap(Thread t) { 

return t.threadLocals;//threadLocals为线程的变量 

}

private Entry getEntry(ThreadLocal<?> key) { 

int i = key.threadLocalHashCode & (table.length - 1); 

Entry e = table[i];

if (e != null && e.get() == key) 

return e; 

else

return getEntryAfterMiss(key, i, e);//避免内存泄漏，下文有提。 

}
```

## 内存泄漏

ThreadLocalMap结构如下：

~~~java
static class ThreadLocalMap { 

/**

```
* The entries in this hash map extend WeakReference, using
* its main ref field as the key (which is always a
* ThreadLocal object). Note that null keys (i.e. entry.get()
* == null) mean that the key is no longer referenced, so the
* entry can be expunged from table. Such entries are referred to
* as "stale entries" in the code that follows.
*/
```

static class Entry extends WeakReference<ThreadLocal<?>> { 

Object value; 



Entry(ThreadLocal<?> k, Object v) { 

super(k); //key，这个是一个弱引用，如果没有强引用来引用key(也就是ThreadLocal)，则key会被回收，形成了key为null的Entry。 

value = v;

}

}

private Entry[] table;

}
~~~

threadLocals变量是在线程内部的，故没有多个线程去访问它，所以不存在线程不安全的说法，同时只要线程被回收了就不会存在内存泄漏。

ThreadLocal对象被回收时(key为null)，没有办法获取到value，而线程又不会被回收时则value一直占用空间导致内存泄漏。线程不会被回收的常见场景是线程池。

JDK在此做了一个优化，在调用get(),set(),remove()方法会做额外处理来清理ThreadLocalMap中key为null的value，以减少内存泄漏的影响。但是如果key未使用弱引用，即使ThreadLocal被回收了，key也不为null，也就是说是没法判断哪个value需要回收的，最终造成内存泄漏。所以此处的弱引用key是内存泄漏的一个优化处理方式。

## 使用场景

在实际项目中，可以用来减少同一个线程内多个函数或者组件之间一些公共变量的传递的复杂度，因为servlet是单例多线程的，每个请求执行的操作都是同一个线程中。比如：可以用ThreadLocal来存每一次请求用户的信息，定义了一个类

```java
public class UserContextHolder{ 

private static ThreadLocal<User> userContextHolder = new ThreadLocal<>(); 

public static setUser(User user){ 

userContextHolder.set(user);

}

public static User getUser(){ 

return userContextHolder.get(); 

}

public static void remove(){ 

return userContextHolder.remove(); 

}

}
```

当用户每次请求进来时，在拦截器中获取用户信息调用`UserContextHolder.setUser()`将其放到`userContext`中，无论在哪我们只要调用`UserContextHolder.getUser()`可以很轻松的获取到用户的信息，而不用在函数调用时一层一层的传递。同时在拦截器结束时调用`UserContextHolder.remove()`移除掉即可。

> 转载自简书: [eejron](https://www.jianshu.com/u/a0e9b7c36fdd)