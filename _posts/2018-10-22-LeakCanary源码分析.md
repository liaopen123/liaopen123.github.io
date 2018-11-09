---
layout:     post
title:   LeakCanary源码分析
subtitle:  开源项目源码分析
date:       2018-11-09
author:     Pony
header-img: img/post-bg-cook.jpg
catalog: true
tags:
    - java
    - 开源项目源码分析
---
## 作用
leakcanary是用于检测app内存泄漏的工具
### 什么是内存泄漏？
当一个对象已经不需要再使用，本该被回收时，而有另一个正在使用的对象持有它的引用从而就导致对象不能被回收。这种导致了本该被回收的对象不能被回收而停留在堆内存中，就产生了内存泄漏。简而言之，内存不在GC掌控之内了。

### LeakCanary执行步骤：
#### 监听
当一个activity走完onDestory后，就说明该页面已经被销毁，应该被系统的GC回收。通过调用Application.registerActivityLifeCycleCallbacks()方法注册监听。当每一个Activity被销毁的时候，在**onDestory生命周期的回调里获取到这个Activity**，并检测这个页面是否真真正正被回收。
#### 检测
 通过WeakReference+ReferenceQueue来判断对象是否被回收。弱引用创建时，传入的是ReferenceQueue对象。当被WeakReference引用的生命周期结束，一旦被GC检查到，GC会把该对象添加到ReferenceQueue中，当 GC 过后对象一直不被加入 ReferenceQueue，就说明有可能存在内存泄漏，然后手动出发GC,进行二次确认。
 #### 分析
 开启了一个IntentService，在工作线程中去分析Heap文件，然后存储日志记录，发送Notification通知，避免ANR。
 分析使用了Square的开源库haha，利用它获取当前内存中的heap堆的快照t，然后通过待分析对象去snapshot里面去查找最短路径，并通过系统通知告知用户。
 
 #### 其他准备工作
##### LeakCanary.enableDisplayLeakActivity(隐藏用于展示内存泄漏的Activity)
 ```java
 public static void enableDisplayLeakActivity(Context context) {
    setEnabled(context, DisplayLeakActivity.class, true);
  }

  public static void setEnabled(Context context, final Class<?> componentClass,
      final boolean enabled) {
    final Context appContext = context.getApplicationContext();
    executeOnFileIoThread(new Runnable() {
      @Override public void run() {
        setEnabledBlocking(appContext, componentClass, enabled);
      }
    });
  }

  public static void setEnabledBlocking(Context appContext, Class<?> componentClass,
      boolean enabled) {
    ComponentName component = new ComponentName(appContext, componentClass);
    PackageManager packageManager = appContext.getPackageManager();
    int newState = enabled ? COMPONENT_ENABLED_STATE_ENABLED : COMPONENT_ENABLED_STATE_DISABLED;
    // Blocks on IPC.
    packageManager.setComponentEnabledSetting(component, newState, DONT_KILL_APP);
  }
```
```xml
 <activity
        android:theme="@style/leak_canary_LeakCanary.Base"
        android:name=".internal.DisplayLeakActivity"
        android:enabled="false"
        android:label="@string/leak_canary_display_activity_label"
        android:icon="@drawable/leak_canary_icon"
        android:taskAffinity="com.squareup.leakcanary.${applicationId}"
        >
      <intent-filter>
        <action android:name="android.intent.action.MAIN"/>
        <category android:name="android.intent.category.LAUNCHER"/>
      </intent-filter>
    </activity>
```
##### ActivityRefWatcher.install(**监听生命周期**)
```java
public final class ActivityRefWatcher {
  public static void install(Application application, RefWatcher refWatcher) {
    new ActivityRefWatcher(application, refWatcher).watchActivities();
  }

  private final Application.ActivityLifecycleCallbacks lifecycleCallbacks =
      new Application.ActivityLifecycleCallbacks() {
        @Override public void onActivityCreated(Activity activity, Bundle savedInstanceState) {
        }

        @Override public void onActivityStarted(Activity activity) {
        }

        @Override public void onActivityResumed(Activity activity) {
        }

        @Override public void onActivityPaused(Activity activity) {
        }

        @Override public void onActivityStopped(Activity activity) {
        }

        @Override public void onActivitySaveInstanceState(Activity activity, Bundle outState) {
        }

        @Override public void onActivityDestroyed(Activity activity) {
        //在ActivityLifeCycler的onDestory中注册  监听内存泄漏
          ActivityRefWatcher.this.onActivityDestroyed(activity);
        }
      };

  private final Application application;
  private final RefWatcher refWatcher;

  /**
   * Constructs an {@link ActivityRefWatcher} that will make sure the activities are not leaking
   * after they have been destroyed.
   */
  public ActivityRefWatcher(Application application, RefWatcher refWatcher) {
    this.application = checkNotNull(application, "application");
    this.refWatcher = checkNotNull(refWatcher, "refWatcher");
  }

  void onActivityDestroyed(Activity activity) {
    refWatcher.watch(activity);
  }

  public void watchActivities() {
    // Make sure you don't get installed twice.
    stopWatchingActivities();//确保没有二次注册
  application.registerActivityLifecycleCallbacks(lifecycleCallbacks);//重新注册
  }

  public void stopWatchingActivities() {
    application.unregisterActivityLifecycleCallbacks(lifecycleCallbacks);
  }
}
```
#### 方法详解
监听生命周期中的onDestroy调用了onActivityDestroyed。实质是调用了 **refWatcher.watch(activity);**
```java
ActivityRefWatcher.this.onActivityDestroyed(activity);

RefWatcher：
     void onActivityDestroyed(Activity activity) {
    refWatcher.watch(activity);
  }
    
```

`watch(Activity)`:
```java
 public void watch(Object watchedReference) {
    watch(watchedReference, "");
  }
  public void watch(Object watchedReference, String referenceName) {
    if (this == DISABLED) {
      return;
    }
    // 检测watchedReference是否为null
    checkNotNull(watchedReference, "watchedReference");
    // 检测referenceName是否为null
    checkNotNull(referenceName, "referenceName");
    final long watchStartNanoTime = System.nanoTime();
    // 生成对应watchReference的唯一标识，并添加到retainedKeys集合中
    String key = UUID.randomUUID().toString();
    retainedKeys.add(key);
    // 构造WeakReference，这里的WeakReference是集合queue一起使用的
    final KeyedWeakReference reference =
        new KeyedWeakReference(watchedReference, key, referenceName, queue);
    // 检测对象是否内存泄漏
    ensureGoneAsync(watchStartNanoTime, reference);
  }



private void ensureGoneAsync(final long watchStartNanoTime, final KeyedWeakReference reference) {
    watchExecutor.execute(new Retryable() {
      @Override public Retryable.Result run() {
        return ensureGone(reference, watchStartNanoTime);
      }
    });
  }

  Retryable.Result ensureGone(final KeyedWeakReference reference, final long watchStartNanoTime) {
    long gcStartNanoTime = System.nanoTime();
    long watchDurationMs = NANOSECONDS.toMillis(gcStartNanoTime - watchStartNanoTime);
    // 移除掉已经回收的对象的key
    removeWeaklyReachableReferences();

    // 如果retainedKeys已经不包含这个reference，说明对象已经回收了
    if (gone(reference)) {
      return DONE;
    }

    // 手动触发一次系统GC
    gcTrigger.runGc();
    // 再次移除GC后已经回收的对象的key
    removeWeaklyReachableReferences();
    if (!gone(reference)) {
      // 如果reference没被回收，记录最短路径，标识内存泄漏
      long startDumpHeap = System.nanoTime();
      long gcDurationMs = NANOSECONDS.toMillis(startDumpHeap - gcStartNanoTime);

      File heapDumpFile = heapDumper.dumpHeap();
      if (heapDumpFile == RETRY_LATER) {
        // Could not dump the heap.
        return RETRY;
      }
      long heapDumpDurationMs = NANOSECONDS.toMillis(System.nanoTime() - startDumpHeap);
      heapdumpListener.analyze(
          new HeapDump(heapDumpFile, reference.key, reference.name, excludedRefs, watchDurationMs,
              gcDurationMs, heapDumpDurationMs));
    }
    return DONE;
  }

//LeakCanary是如何触发GC的
GcTrigger DEFAULT = new GcTrigger() {
    @Override public void runGc() {
      // Code taken from AOSP FinalizationTest:
      // https://android.googlesource.com/platform/libcore/+/master/support/src/test/java/libcore/
      // java/lang/ref/FinalizationTester.java
      // System.gc() does not garbage collect every time. Runtime.gc() is
      // more likely to perfom a gc.
      Runtime.getRuntime().gc();
      enqueueReferences();
      System.runFinalization();
    }

    private void enqueueReferences() {
      // Hack. We don't have a programmatic way to wait for the reference queue daemon to move
      // references to the appropriate queues.
      try {
        Thread.sleep(100);
      } catch (InterruptedException e) {
        throw new AssertionError();
      }
    }
  };

```
 