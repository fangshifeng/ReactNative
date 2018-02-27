# 《ReactNative系列讲义》高级篇---05.JavaScript与Native之间的通信（二）

**|** 版权声明：本文为博主原创文章，未经博主允许不得转载。

#### 一、简介
上篇文章介绍到功能组件的封装，JS端和Native端之间的通讯相对来讲还是很简单的，今天我们介绍封装UI组件以及JS端和Native之间的通信方式。

#####  1. 创建CustomImageView类

```
// 创建自定义View
public static class CustomImageView extends View {
   public CustomImageView(Context context) {
       super(context);
       // TODO 可做一些初始化工作
   }

   public CustomImageView(Context context, @Nullable AttributeSet attrs) {
       super(context, attrs);
   }

   public CustomImageView(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
       super(context, attrs, defStyleAttr);
   }
}
```

##### 2. 创建ViewManager的子类

* 继承ReactNative封装的SimpleViewManager<T>类，或者继承ViewManager<T>类；

```
public class UIComponentViewManager extends SimpleViewManager<UIComponentViewManager.CustomImageView> {
    private static final String REACT_CLASS = "RCTImageView";

    @Override
    public String getName() {
        return REACT_CLASS;
    }
}
```

##### 3. 实现createViewInstance方法

```
public class UIComponentViewManager extends SimpleViewManager<UIComponentViewManager.CustomImageView> {
    private static final String REACT_CLASS = "RCTImageView";
    private CustomImageView mCustomImageView;

    @Override
    public String getName() {
        return REACT_CLASS;
    }
    
    @Override
    protected CustomImageView createViewInstance(ThemedReactContext reactContext) {
        // 初始化自定义视图类
        mCustomImageView = new CustomImageView(reactContext);
        return mCustomImageView;
    }
}
```

##### 4. 暴露出设置视图属性的Setter方法，使用@ReactProp或者@ReactPropGroup声明

```
public class UIComponentViewManager extends SimpleViewManager<UIComponentViewManager.CustomImageView> {
    private static final String REACT_CLASS = "RCTImageView";
    private CustomImageView mCustomImageView;

    @Override
    public String getName() {
        return REACT_CLASS;
    }
    
    @Override
    protected CustomImageView createViewInstance(ThemedReactContext reactContext) {
        mCustomImageView = new CustomImageView(reactContext);
        return mCustomImageView;
    }
    
    // 第一个参数是当前需要更新的View；
    // 第二个参数是要修改的属性参数
    // 支持的参数类型：boolean int float double String Boolean Integer ReadableArray ReadableMap
    // 使用@ReactProp声明，有一个强制性的字符串属性：name; 还可设置默认属性;
    // 圆角属性，默认值是 浮点类型 0
    @ReactProp(name = "borderRadius", defaultFloat = 0f)
    public void setBorderRadius(ReactImageView view, float borderRadius) {
        view.setBorderRadius(borderRadius);
    }

    @ReactProp(name = ViewProps.RESIZE_MODE)
    public void setResizeMode(ReactImageView view, @Nullable String resizeMode) {
        // 图片展示模式
        view.setScaleType(ImageResizeMode.toScaleType(resizeMode));
    }
}
```

##### 5. 在Applications package管理类中的createViewManagers方法里注册视图管理者
* 创建NativeReactPackage.java类，实现ReactPackage接口

```
@Override
public List<ViewManager> createViewManagers(ReactApplicationContext reactContext) {
    return Arrays.<ViewManager>asList(
        new ReactImageManager()
    );
}
```

##### 6. MainApplication.java文件中注册NativeReactPackage类

```
@Override
    protected List<ReactPackage> getPackages() {
      return Arrays.<ReactPackage>asList(
          new MainReactPackage(),
          new NativeReactPackage()
      );
    }
```

##### 7. 在Native层定义事件，并处理事件

* 重写getCommandsMap方法（提供JS与Native通信的命令组），每组命令需要包括名称（js端调用的方法名）和命令ID
* 重写receiveCommand方法，处理接受来自JS端的对应的命令

```
private static final int DO_ACTION_ID = 0;
private static final String DO_ACTION_NAME = "doAction";
    
/**
* 接受多组命令，接收各式：k1,v1,k2,v2,k3,v3...
*
* @return
*/
@Override
public Map<String, Integer> getCommandsMap() {
   return MapBuilder.of(
           DO_ACTION_NAME, DO_ACTION_ID
   );
}

/**
* 处理相应的命令
*
* @param root
* @param commandId
* @param args
*/
@Override
public void receiveCommand(CustomImageView root, int commandId, @javax.annotation.Nullable ReadableArray args) {
   super.receiveCommand(root, commandId, args);
   switch (commandId){
       case DO_ACTION_ID:
           // TODO Do something for your native
           break;
       default:
           break;
   }
}
```

##### 8. JS端API封装（如何发送命令给Native）
* 创建UIComponentView类，封装供JS端调用的API
* 调用NativeModules.UIManager.dispatchViewManagerCommand()方法，主要是实现该方法的三个参数

```
import PropTypes from 'prop-types';
import React, { Component } from 'react';
import { requireNativeComponent, View, findNodeHandle, NativeModules} from 'react-native';

class UIComponentView extends Component {
    constructor(props) {
        super(props);
    }
    
    // 主要方法，向Native发送命令，提供给JS端调用的API方法
    _runCommand = (name, args) => {
        NativeModules.UIManager.dispatchViewManagerCommand(
            this._getHandle() // 获取组件的实例对象
            ,this._uiManagerCommand(name) // 给Native发送的命令名称，该名称与Native定义的DO_ACTION_NAME常量一样
            ,args // 命令携带的参数数据
        )
    };
    
    _getHandle = () => {
        return findNodeHandle(this.imageView);
    };
    
    _uiManagerCommand(name) {
        return NativeModules.UIManager['RCTImageView'].Commands[name];
    }

    render() {
        return <RCTImageView {...this.props} onChange={this._onChange} ref={(ref) => {this.imageView = ref}}/>
    }
}

UIComponentView.name = "ImageView";
UIComponentView.propTypes = {
    borderRadius: PropTypes.number
    ,resizeMode: PropTypes.oneOf(['cover', 'contain', 'stretch'])
    ,...View.propTypes // include the default view properties
};

// 第一个参数是原生视图的名称；第二个参数是封装组件的类；
var RCTImageView = requireNativeComponent('RCTImageView', UIComponentView);

// 注意这里导出的是哪个对象!!!不是RCTImageView!!!
module.exports = UIComponentView;
```

##### 9. Native接收到JavaScript发送的命令做出处理并反馈给Native，（这里做成Native向JavaScript发送事件消息样式用以说明API的调用方法）
* 在自定义的视图类中操作

```
public static class CustomImageView extends View {
    
  public CustomImageView(Context context) {
      super(context);
  }
    
  public CustomImageView(Context context, @Nullable AttributeSet attrs) {
      super(context, attrs);
  }
    
  public CustomImageView(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
      super(context, attrs, defStyleAttr);
  }
    
  /**
    * 接受JS端发来的命令后，Native做出处理，并通过该方法返回结果给JS
   *  该方法会作为属性定义在JS端API封装文件中
  */
    public void onReceiveNationEvent() {
      WritableMap event = Arguments.createMap();
      event.putString("message", ""); // key值用于js中的nativeEvent
      ReactContext reactContext = (ReactContext) getContext();
      reactContext.getJSModule(RCTEventEmitter.class).receiveEvent(
              getId() // native层和js层两个视图会根据getId()返回的值关联在一起
              ,"topChange" // 名称固定
              ,event
      );
    }
}
```

##### 10. Native命令发送
* 来到CustomUINativeModule类中，在接收到JS发送的命令方法中，发送Native给JS端的命令

```
 /**
    * 处理相应的命令
    *
    * @param root
    * @param commandId
    * @param args
 */
@Override
public void receiveCommand(CustomImageView root, int commandId, @javax.annotation.Nullable ReadableArray args) {
   super.receiveCommand(root, commandId, args);
   switch (commandId){
       case DO_ACTION_ID:
           // TODO Do something for your native
           Log.d("msg", "JavaScript发来命令！！！");
           // 发射！！！
           mCustomImageView.onReceiveNationEvent();
           break;
       default:
           break;
   }
}
```

##### 11. JS端API封装的修改
* CustomUINativeModule.propTypes中添加一个属性
* JS端API中实现该类的调用

```
/**
 * Created by fangshifeng on 2017/11/7.
 */

import PropTypes from 'prop-types';
import React, { Component } from 'react';
import { requireNativeComponent, View, findNodeHandle, NativeModules} from 'react-native';

class CustomUINativeModule extends Component {
    constructor(props) {
        super(props);
    }

    // 主要方法，向Native发送命令
    _runCommand = (name, args) => {
        NativeModules.UIManager.dispatchViewManagerCommand(
            this._getHandle() // 获取组件的实例对象
            ,this._uiManagerCommand(name) // 发送的命令名称，该名称与Native定义的DO_ACTION_NAME常量一样
            ,args // 命令携带的参数数据
        )
    };

    _getHandle = () => {
        return findNodeHandle(this.imageView);
    };

    _uiManagerCommand(name) {
        // RCTCustomImageView 名称一定要和Native定义的名称常量REACT_CLASS一致
        return NativeModules.UIManager['RCTCustomImageView'].Commands[name];
    }

    _onChange = (event) => {
        if(!this.props.onChangeMessage) {
            return;
        }
        this.props.onChangeMessage(event.nativeEvent.message); // message是Native层onReceiveNationEvent()里面event的key值
    };

    render() {
        // <RCTCustomImageView/> 名称一定要和Native定义的名称常量REACT_CLASS一致
        return <RCTCustomImageView {...this.props} ref={(ref) => {this.imageView = ref}} onChange={this._onChange}/>
    }
}

CustomUINativeModule.name = "RCTCustomImageView";
CustomUINativeModule.propTypes = {
    onChangeMessage: PropTypes.func
    ,borderRadius: PropTypes.number
    ,resizeMode: PropTypes.oneOf(['cover', 'contain', 'stretch'])
    ,...View.propTypes
};

let RCTCustomImageView = requireNativeComponent(
    'RCTCustomImageView',
    CustomUINativeModule,
    {nativeOnly: {onChange: true}}
);
module.exports = CustomUINativeModule;
```
* nativeOnlyd的相关介绍：我个人理解是，在封装UI组件时，Native层给JS端发送事件消息，在封装功能性组件的时候，我们已经学过三种方法，而UI组建封装的时候，前三种方式可能也同样适用（未验证），但是如果不想将Native封装的API暴露给JS端，又想实现通信，那么就可以使用nativeOnly参数。onChange作为统一的事件出口，event参数作为统一的参数出口，NativeAPI方法的名称可随意定义。

#### 12. JS端视图层调用API

```
import React, { Component } from 'react';
import {
    View
    ,StyleSheet
    ,Text
    ,TouchableOpacity
} from 'react-native';
import ImageView from '../../../js/CustomUINativeModule';
import { Toast } from 'antd-mobile';

/**
 * 封装NativeUIComponent
 */
export default class CustomUIComponentScreen extends Component{
    constructor(props) {
        super(props);

        this.state = {
            message: '没有消息'
        }
    }

    _receiveEvent = (message) => {
        this.setState({
            message: message
        });
    };

    _sendEvent = () => {
        if(this.imageView != null) {
            this.imageView._runCommand('doAction', null); // name参数值要与native中Command Name保持一致
            // console.log('JS端已将命令发送至Native层~');
            // Toast.info('命令已发送！');
        } else {
            Toast.info('imageView = null', 2);
        }
    };

    render() {
        return(
            <View style={styles.container}>
                <ImageView
                    src={'http://pic.qjimage.com/pm0050/high/pm0050-0471it.jpg'}
                    style={{ height:100, width:100 }}
                    ref={(ref) => {this.imageView = ref}}
                    onChangeMessage={ this._receiveEvent }
                />
                <Text>接收Native事件：{this.state.message}</Text>
                <TouchableOpacity onPress={this._sendEvent}>
                    <Text>向Native发送命令</Text>
                </TouchableOpacity>
            </View>
        );
    }
}

const styles = StyleSheet.create({
    container: {
        flex: 1,
        justifyContent: 'center',
        alignItems: 'center',
        backgroundColor: '#F5FCFF',
    },
    welcome: {
        fontSize: 20,
        textAlign: 'center',
        margin: 10,
    },
    instructions: {
        textAlign: 'center',
        color: '#333333',
        marginBottom: 5,
    },
});
```

#### 三、 
到这里关于JS端与Native层之间的通信，在两种情况下的方式方法都已经讲完了，后续我们会深入两者之间通信桥的原理进行讲解。




