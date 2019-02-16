---
layout:     post
title:  ReactNative混合开发
subtitle: Android为主RN为辅助 
date:     2019-02-15
author:     Pony
header-img: img/post-bg-mayday-bubble.jpg
catalog: true
tags:
    - ReactNative
---
目录层级
![](https://ws2.sinaimg.cn/large/006tKfTcly1g079oty9toj30si0ae76h.jpg)


1. 先创建总的项目文件夹`reactNativeProject`
2. 再在根目录下创建Android项目的根文件夹：`android`
3. 并创建项目：
![](https://ws3.sinaimg.cn/large/006tKfTcly1g079ue6bdsj31b00kqdhp.jpg)
保持与`步骤2`的目录统一

1. 在`reactNativeProject`总的项目目录下创建一个名为`package.json`的空文本。
 
 >touch package.json
 >open package.json
 
 并且填入一下内容：
 ```json
 {
  "name": "MyReactNativeApp",
  "version": "0.0.1",
  "private": true,
  "scripts": {
    "start": "node node_modules/react-native/local-cli/cli.js start"
  },
  "dependencies": {
    "react": "16.0.0-alpha.6",
    "react-native": "0.44.3"
  }
}
 ```
 其中各个属性的定义分别为：
 //TO-DO

5. 接下来使用`npm(node package manager)`来安装React和React Native模块：
    打开终端 切换到项目`reactNativeProject`目录，执行：
    npm install 
    如果报错：`ERR! code EINTEGRITY`
    有可能是权限的问题，执行`sudo npm install`。
    
    就会多出一个`node_modules`的目录。
    
6. 接下来对Android Project配置maven:
 在android/build.gradle 文件中添加 React Native 依赖:
 
 ```json
 allprojects {
    repositories {
        jcenter()
        maven {
            // All of React Native (JS, Android binaries) is installed from npm
            url "$rootDir/../node_modules/react-native/android"
        }
    }
}
 ```
 
 再在app的build.gradle下配置依赖库：
 
 ```json
 dependencies {
   .......
    compile "com.facebook.react:react-native:+" // From node_modules.
}
 ```
 
7. 清单文件中配置权限 如果不配，页面会报错`404`。

```xml
<uses-permission android:name="android.permission.INTERNET" />
```

如果需要访问rn开发者菜单(DevSettingActivity),还需要添加activity:

```xml
<activity android:name="com.facebook.react.devsupport.DevSettingsActivity" />
```

8. 代码基础，这一步生成的文件主要是用来展示rn页面内容的文件:
    >touch index.android.js
    >open index.android.js
    
    并复制如下代码：
    
    ```javascript
    'use strict';
import React from 'react';
import {
  AppRegistry,
  StyleSheet,
  Text,
  View
} from 'react-native';
class HelloWorld extends React.Component {
  render() {
    return (
      <View style={styles.container}>
        <Text style={styles.hello}>Hello, World</Text>
      </View>
    )
  }
}
var styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
  },
  hello: {
    fontSize: 20,
    textAlign: 'center',
    margin: 10,
  },
});
AppRegistry.registerComponent('MyReactNativeApp', () => HelloWorld);
    ```
在Android创建用于展示的rn内容的activity:

```java
public class ReactActivity extends AppCompatActivity implements DefaultHardwareBackBtnHandler {
    public static final int OVERLAY_PERMISSION_REQ_CODE = 1235;
    private ReactRootView mReactRootView;
    private ReactInstanceManager mReactInstanceManager;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_react);

        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
            if (!Settings.canDrawOverlays(this)) {
                Intent intent = new Intent(Settings.ACTION_MANAGE_OVERLAY_PERMISSION,
                        Uri.parse("package:" + getPackageName()));
                startActivityForResult(intent, OVERLAY_PERMISSION_REQ_CODE);
            }
        }

        mReactRootView = new ReactRootView(this);
        mReactInstanceManager = ReactInstanceManager.builder().setApplication(getApplication())
                .setBundleAssetName("index.android.bundle")
                .setJSMainModuleName("index.android")
                .addPackage(new MainReactPackage())
                .setUseDeveloperSupport(BuildConfig.DEBUG)
                .setInitialLifecycleState(LifecycleState.RESUMED)
                .build();
        mReactRootView.startReactApplication(mReactInstanceManager, "MyReactNativeApp", null);
        setContentView(mReactRootView);
    }

    @Override
    public void invokeDefaultOnBackPressed() {
        super.onBackPressed();
    }

    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
//        super.onActivityResult(requestCode, resultCode, data);

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



这样混合开发就完成了
最后如何在真机上运行：
命令行切换到`reactNativeProject`,执行`npm start`，或者执行`react-native run-android`。即可执行运行Android程序。
