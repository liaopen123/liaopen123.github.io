---
layout:     post
title:  React Native调用Android原生方法和传值
subtitle: RN Call Android 和  传值
date:     2019-02-18
author:     Pony
header-img: img/post-bg-mayday-bubble.jpg
catalog: true
tags:
    - ReactNative
    - Android
---
## 创建react native 项目：
 > react-native init callAndroidProject
 >
 >cd callAndroidProject
 >
 >react-native run-android 
 
 这样就初步创建出了一个React-Natvie项目
 
## 完成Android端的操作
 
### 第一步： 创建一个ReactModule类 继承 `ReactContextBaseJavaModule`。
这个类是一个桥梁类，react-native调用Android就在这里进行操作。
复写其中的1个方法：`getName()`。**返回的名称用于在rn代码中调用时使用**。
如果要添加调用方法，需要在新增的方法中添加注解`@ReactMethod`.
具体代码实现：
```java
public class MyCallModule extends ReactContextBaseJavaModule {
    private final ReactApplicationContext mContext;

    public MyCallModule(ReactApplicationContext reactContext) {
        super(reactContext);
        this.mContext = reactContext;
    }

    @Override
    public String getName() {
        return "MyCallModule";
    }


    @ReactMethod
    public void showToast(String str,Callback callback){
        Toast.makeText(mContext, str, Toast.LENGTH_SHORT).show();
        callback.invoke("success");
    }
}
```
### 第二步：创建一个类 用于实现 `ReactPackage`。
主要用于注册module。当有多个module类的时候，用ReactPackage类，注册多个module。
代码展示：
```java
public class TotalPackage implements ReactPackage {
    @Override
    public List<NativeModule> createNativeModules(ReactApplicationContext reactContext) {
        List<NativeModule> list = new ArrayList<>();
        list.add(new MyCallModule(reactContext));//在这添加 交互的module
        //list.add(...)
        return list;
    }

    @Override
    public List<ViewManager> createViewManagers(ReactApplicationContext reactContext) {
        return Collections.emptyList();
    }
}
```

### 第三步：在Application类中`ReactNativeHost#getPackages`添加步骤二中的package。
```java
public class MainApplication extends Application implements ReactApplication {

    private final ReactNativeHost mReactNativeHost = new ReactNativeHost(this) {
        @Override
        public boolean getUseDeveloperSupport() {
            return BuildConfig.DEBUG;
        }

        @Override
        protected List<ReactPackage> getPackages() {
            return Arrays.<ReactPackage>asList(
                    new MainReactPackage(),
                    new TotalPackage()////新添加需要注册的模块
            );
        }

        @Override
        protected String getJSMainModuleName() {
            return "index";
        }
    };
    }
```
 这样 Android 原生端的代码就初步完成了。
 
## 完成React Native端操作
需要注意的地方有：
1. 添加的有Android端的交互，所以导包的时候需要import {NativeModules}。
2. `NativeModules.MyCallModule.showToast`中`MyCallModule`为Android端`MyCallModule#getName()`返回值。
3. `AppRegistry.registerComponent('callAndroidProject', () => HelloWorldApp);`中引号括起来的'callAndroidProject'必须和你init创建的项目名一致。


代码如下所示：
```javascript
import React, { Component } from 'react';
import { AppRegistry, Text,NativeModules} from 'react-native';//添加的有NativeModules交互，使用就需要导入NativeModules到项目中。

class HelloWorldApp extends Component {
  _toast(){
    NativeModules.MyCallModule.showToast('toastlph111',(success)=>{alert(success)})
}
  render() {
    return (
      <Text  onPress={this._toast} >Hello 22222222221!</Text>
    );
  }
}

// 注意，这里用引号括起来的'asproject'必须和你init创建的项目名一致
AppRegistry.registerComponent('callAndroidProject', () => HelloWorldApp);
```


这样就初步完成了React Native 与 Android端的交互流程。

源码地址：[react-native-call-android](https://github.com/liaopen123/react-native-call-android)

## Android页面内嵌套rn布局
### 方法一：通过container#addView()添加进去
具体代码：
```java
 @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        mReactRootView = new ReactRootView(this);
        mReactInstanceManager = ReactInstanceManager.builder()
                .setApplication(getApplication())
                .setBundleAssetName("index.android.bundle")
                .setJSMainModuleName("index.android")
                .addPackage(new MainReactPackage())
                .setUseDeveloperSupport(BuildConfig.DEBUG)
                .setInitialLifecycleState(LifecycleState.RESUMED)
                //.setUseOldBridge(true) // uncomment this line if your app crashes
                .build();
        mReactRootView.startReactApplication(mReactInstanceManager, "HelloWorld", null);

        setContentView(R.layout.myreactactivity);
        initView();
    }

   
    private void initView() {
        ll = (LinearLayout) findViewById(R.id.ll);
        ll.addView(mReactRootView);
    }
}
```

### 方法二：通过xml布局中添加ReactRootView，进行展示：
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:orientation="vertical"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".OriginalContainRNActivity">
<TextView
    android:text="我是原生"
    android:textColor="#ffffff"
    android:textSize="24dp"
    android:gravity="center"
    android:background="#000000"
    android:layout_width="match_parent"
    android:layout_height="50dp" />
<com.facebook.react.ReactRootView
    android:id="@+id/reactRootView"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"></com.facebook.react.ReactRootView>
    <TextView
        android:text="我是原生"
        android:textColor="#ffffff"
        android:textSize="24dp"
        android:gravity="center"
        android:background="#000000"
        android:layout_width="match_parent"
        android:layout_height="50dp" />
</LinearLayout>
```
```java
public class OriginalContainRNActivity  extends AppCompatActivity implements DefaultHardwareBackBtnHandler {
    public static final int OVERLAY_PERMISSION_REQ_CODE = 1235;
    private ReactRootView mReactRootView;
    private ReactInstanceManager mReactInstanceManager;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_original_contain_rn);
//        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
//            if (!Settings.canDrawOverlays(this)) {
//                Intent intent = new Intent(Settings.ACTION_MANAGE_OVERLAY_PERMISSION,
//                        Uri.parse("package:" + getPackageName()));
//                startActivityForResult(intent, OVERLAY_PERMISSION_REQ_CODE);
//            }
//        }

        mReactRootView = findViewById(R.id.reactRootView);
        mReactInstanceManager = ReactInstanceManager.builder().setApplication(getApplication())
                .setBundleAssetName("index.android.bundle")
//                .setJSMainModuleName("index.android")
                .addPackage(new MainReactPackage())
                .setUseDeveloperSupport(BuildConfig.DEBUG)
                .setInitialLifecycleState(LifecycleState.RESUMED)
                .build();
        mReactRootView.startReactApplication(mReactInstanceManager, "asproject", null);

    }

    @Override
    public void invokeDefaultOnBackPressed() {
        super.onBackPressed();
    }

    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
// super.onActivityResult(requestCode, resultCode, data);

        if (requestCode == OVERLAY_PERMISSION_REQ_CODE) {
            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
                if (!Settings.canDrawOverlays(this)) {
                    // SYSTEM_ALERT_WINDOW permission not granted...
                }
            }
        }
    }
    @Override
    protected void onPause() {
        super.onPause();
        if (mReactInstanceManager != null) {
            mReactInstanceManager.onHostPause(this);
        }
    }

    @Override
    protected void onResume() {
        super.onResume();
        if (mReactInstanceManager != null) {
            mReactInstanceManager.onHostResume(this, this);
        }
    }
    @Override
    protected void onDestroy() {
        super.onDestroy();
        if (mReactInstanceManager != null) {
            mReactInstanceManager.onHostDestroy(this);
        }
    }
    @Override
    public void onBackPressed() {
        if (mReactInstanceManager != null) {
            mReactInstanceManager.onBackPressed();
        } else {
            super.onBackPressed();
        }
    }
    @Override
    public boolean onKeyDown(int keyCode, KeyEvent event) {
        if (keyCode == KeyEvent.KEYCODE_MENU && mReactInstanceManager != null) {
            mReactInstanceManager.showDevOptionsDialog();
            return true;
        }
        return super.onKeyDown(keyCode, event);
    }
}
```

### rn与Android数据交互的三种方式：
1. Callback
2. Promise
3. RN端DeviceEventEmitter注册监听，App端发送消息。类似于广播。

#### Callback
RN端调用方法，Android端执行对应方法，并通过Promise进行回调。
Android端代码演示：
```java
 @ReactMethod
    public void testAndroidCallbackMethod(String msg, Callback callback){
        Toast.makeText(getReactApplicationContext(),msg,Toast.LENGTH_LONG).show();
        callback.invoke("abc");
    }
```
RN端代码演示：
```javascript
   _onPressButton2(){
        NativeModules.CallModule.testAndroidCallbackMethod("HelloJack",(result)=>{
           this.setState({text:result});
       });
    }
```
#### Promise
RN端调用方法，Android端执行对应方法，并通过Promise进行回调。
Android端代码演示：
```java
  @ReactMethod
    public void textAndroidPromiseMethod(String msg, Promise promise){
        Toast.makeText(getReactApplicationContext(),msg,Toast.LENGTH_SHORT).show();
        String result="lph23333";
        promise.resolve(result);
    }
```
RN端代码演示：
```javascript
  _onPressButton3(){
       NativeModules.CallModule.textAndroidPromiseMethod("abcx").then((result)=>{
                 this.setState({text3:result});
             }).catch((error)=>{
                 this.setState({text:'error'});
             })
    }
```

#### DeviceEventEmitter
执行流程：
1. RN端注册监听
2. Android 发送时间
3. RN端执行监听中的回调事件

RN端代码：
```javascript
//componentWillMount为生命周期方法， 组件将要装载时候执行，在render之前调用。
 componentWillMount() {
        DeviceEventEmitter.addListener('EventName', function  (msg) {
            console.log(msg);
            let rest=NativeModules.CallModule.MESSAGE;
            ToastAndroid.show("DeviceEventEmitter收到消息:" + "\n" + rest, ToastAndroid.SHORT)
        });
```
Android端：
```java
   public void onScanningResult(){
        WritableMap params = Arguments.createMap();
        params.putString("key", "myData");
        getReactApplicationContext().getJSModule(DeviceEventManagerModule.RCTDeviceEventEmitter.class).emit("EventName", params);
    }
```
参考文章:[React Native与Android 原生通信](https://juejin.im/entry/5ae3edcaf265da0b9b071346)
[RN之Android:原生界面与React界面的相互调用及数据传递](https://github.com/ipk2015/RN-Resource-ipk/blob/master/react-native-docs/RN%E4%B9%8BAndroid:%E5%8E%9F%E7%94%9F%E7%95%8C%E9%9D%A2%E4%B8%8EReact%E7%95%8C%E9%9D%A2%E7%9A%84%E7%9B%B8%E4%BA%92%E8%B0%83%E7%94%A8%E5%8F%8A%E6%95%B0%E6%8D%AE%E4%BC%A0%E9%80%92.md)
