# 《ReactNative系列讲义》高级篇---04.JavaScript与Native之间的通信（一）

**|** 版权声明：本文为博主原创文章，未经博主允许不得转载。

#### 一、简介

随着项目功能的多样化，复杂程度逐渐的增大，就会出现ReactNative官方API中有些组件无法满足产品需求，那么我们就需要自己来定义我们产品特殊需求的组件。
组件又包含功能性组件和视图组件；功能性组件中大部分都是用户通过点击之类的触发JS端调用Native组件功能，触发类型与操作方式相对简单，因此JS与Native之间的通信方式也相对简单，方式也多样化，可满足不同的时间和空间上的需求；而自定义UI组件中，JS端和Native之间的通信相对来讲就复杂的多，比如用户会将图片放大缩小，甚至会绘画，使用场景相对于功能组件的使用场景复杂的多。

下面先让我们看看功能模块的封装。

#### 二、封装Native功能组件
##### 1. 创建CustomNativeModule子类，继承ReactContextBaseJavaModule父类

```
public class CustomNativeModule extends ReactContextBaseJavaModule {

    private static final String REACT_CLASS = "CustomNativeModule";
    private ReactApplicationContext context;

    public CustomNativeModule(ReactApplicationContext reactContext) {
        super(reactContext);
        context = reactContext;
    }

    @Override
    public String getName() {
        return REACT_CLASS;
    }
}
```

##### 2. 定义Native API，供JS端直接调用

```
public class CustomNativeModule extends ReactContextBaseJavaModule {

    private static final String REACT_CLASS = "CustomNativeModule";
    private ReactApplicationContext context;

    public CustomNativeModule(ReactApplicationContext reactContext) {
        super(reactContext);
        context = reactContext;
    }

    @Override
    public String getName() {
        return REACT_CLASS;
    }

    // @ReactMethod 设定该方法是供JS端调用的API
    // 设置一个字符串类型参数；JS端调用sendMessageToJS方法，Native层可接收到JS端传递过来的参数arg
    @ReactMethod
    public void sendMessageToJSByCallback(String arg) {
        Log.d("msg", arg);
    }

}
```

##### 3. 在Applications package管理类中的createViewManagers方法里注册视图管理者
* 创建NativeReactPackage.java类，实现ReactPackage接口

```
public class NativeReactPackage implements ReactPackage {
    @Override
    public List<NativeModule> createNativeModules(ReactApplicationContext reactContext) {
        List<NativeModule> modules = new ArrayList<>();
        modules.add(new CustomNativeModule(reactContext));

        return modules;
    }

    @Override
    public List<Class<? extends JavaScriptModule>> createJSModules() {
        return Collections.emptyList();
    }

    @Override
    public List<ViewManager> createViewManagers(ReactApplicationContext reactContext) {
        return Collections.emptyList();
    }
}
```

##### 4. MainApplication.java文件中注册NativeReactPackage类

```
@Override
protected List<ReactPackage> getPackages() {
 return Arrays.<ReactPackage>asList(
     new MainReactPackage(),
     new NativeReactPackage()
 );
}
```

##### 5. Native通过三种方式向JS端传输数据
* Callback 方式

```
/**
 * 通信方式一：Callback call result to JS
 * 自定义组件中才可以自定义方法！！！
 *
 * @param callback
*/
@ReactMethod
public void sendMessageToJSByCallback(Callback callback) {
   callback.invoke("hello js, i am callback");
}
```

* Callback 方式 JS端接收方式

创建CustomNativeModule.js文件

```
import { NativeModules } from 'react-native';

module.exports = NativeModules.CustomNativeModule;
```

* Callback 方式 JS视图层调用

```
import CustomNativeModule from '../../../js/CustomNativeModule';

CustomNativeModule.sendMessageToJSByCallback((msg) => {
    console.log(msg);
});
```

* Promise 方式

```
/**
 * 通信方式二：Promise
 *
 * @param promise
*/
@ReactMethod
public void sendMessageToJSByPromise(Promise promise) {
   promise.resolve("hello js, i am promise");
}
```

* Promise 方式 JS端调用方式

```
CustomNativeModule.sendMessageToJSByPromise()
  .then((msg) => {
      console.log(msg);
  });
```

* SendEvent 方式

```
/**
 * 通信方式三：Sending Events to JavaScript
 *
 * @param eventName
 * @param params
*/
private void sendEvent(String eventName, WritableMap params) {
   context.getJSModule(DeviceEventManagerModule.RCTDeviceEventEmitter.class)
           .emit(eventName, params);
}

/**
 * 使用sendEvent方式 Native 发送消息到 JS
 * 该方法会作为方法属性在JS端API封装中暴露出去
*/
@ReactMethod
public void receiveNativeEvent() {
   WritableMap params = Arguments.createMap();
   params.putString("eventMessage", "hello js, i am sendEvent");
   sendEvent("testSendEvent", params);
}
```

* SendEvent 方式 JS端调用方式

```
componentWillMount() {        
    // 接收Native发送来的事件信息
    DeviceEventEmitter.addListener('testSendEvent', (event) => {
        console.log(event.eventMessage);
}
    
// 发送事件
doSendEvent() {
    CustomNativeModule.receiveNativeEvent();
}
```

##### 6. 可传递数据类型详解与技巧（未完待续...）
Boolean -> Bool、Integer -> Number、Double -> Number、Float -> Number、String -> String、Callback -> function
ReadableMap -> Object、ReadableArray -> Array 


#### 三、总结
**功能性组件中，JS端和Native层之间的通信方式共三种，JS给Native发送信息都是通过参数的方式，Native给JS端回馈信息有三种方式，分别适应不同的情形。**

**JS一次请求Native方法，Native只支持调用它的Callback参数一次，但是可以存储Callback并且延迟调用；**

**JavaScript和Native之间的桥接是异步请求，因此，Callback并不会在Native方法执行完成之后立即调用。**

