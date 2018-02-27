# 《ReactNative系列讲义》高级篇---01.react-native-vxgplayer

**|** 版权声明：本文为博主原创文章，未经博主允许不得转载。

#### 一、简介
ReactNative Android版的发布已有两年多了，RN官网的迭代速度也是相当之快，给广大开发者提供的API也相当丰富了，但是官方给出的API也只能是共有功能或者是常用功能。但实际中，每款APP都会根据自己的产品需求开发一些特有功能。这篇文章就通过一个支持RTSP协议的摄像头功能组件，给大家介绍一下如何制作个性化的npm组件。

#### 二、流程
##### 1. 初始化ReactNative项目
```
react-native init your-project-name
```
Demo工程叫做RNVXGPlayer
##### 2. android目录下添加lib工程
使用Android Studio打开RN项目的android工程，按照原生项目的开发方式，创建Android Library Module工程，名称叫做vxgplayer
步骤如图：![屏幕快照 2017-08-21 下午9.56.09](media/15033069663671/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-08-21%20%E4%B8%8B%E5%8D%889.56.09.png)
![屏幕快照 2017-08-21 下午9.56.34](media/15033069663671/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-08-21%20%E4%B8%8B%E5%8D%889.56.34.png)


##### 3. 导入第三方SDK
（1）将RTSPPlayer-master开源工程中libs目录下的mediaplayersdk.jar、org.apache.http.legacy.jar、android-support-v4.jar文件移入RNVXGPlayer项目vxgplayer工程中的libs目录下。
（2）将RTSPPlayer-master开源项目中libs目录下armeabi、armeabi-v7a、x86三个文件夹移入RNVXGPlayer项目vxgplayer工程jniLibs目录下（如果没有jniLibs目录请自行创建）。
（3）vxgplayer工程下build.gradle文件添加依赖

```
// 添加RN依赖
compile 'com.facebook.react:react-native:+'
// 添加第三方jar包依赖
compile files('libs/mediaplayersdk.jar')
compile files('libs/org.apache.http.legacy.jar')
```

（4）调整app主工程和lib工程SDK版本，根据自己项目的情况而定
（5）添加AndroidManifest文件相关配置

```
android:hardwareAccelerated="true"
android:largeHeap="true"
<uses-permission android:name="android.permission.CHANGE_NETWORK_STATE" />
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.CHANGE_WIFI_MULTICAST_STATE" />
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
<uses-permission android:name="android.permission.DISABLE_KEYGUARD" />
<uses-permission android:name="android.permission.WAKE_LOCK" />
<uses-permission android:name="android.permission.GET_TASKS" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
<uses-permission android:name="android.permission.READ_LOGS"/>
```

##### 4. 编写原生代码
vxgplayer工程下创建包路径：com.vxgplayer.reactnative和com.vxgplayer.utils
######（1）com.vxgplayer.utils路径下：
* 移植SharedSettings.java文件

######（2）com.vxgplayer.reactnative路径下：
* 创建VXGPlayerViewManager.java文件，在该文件中创建RN Native UI

```
public class VXGPlayerViewManager extends ViewGroupManager<MediaPlayer>{
    private static final String REACT_CLASS = "RCTVXGPlayer";
    private ThemedReactContext mReactContext;
    private MediaPlayer mPlayer;
    public boolean mPanelIsVisible = true;
    public boolean isFileUrl = false;
    private PlayerCallBacks currentPlayerCallBacks = null;

    public VXGPlayerViewManager(ReactApplicationContext reactContext) {
        Looper.prepare();
        mPlayer = new MediaPlayer(reactContext);
    }

    @Override
    public String getName() {
        return REACT_CLASS;
    }

    @Override
    protected MediaPlayer createViewInstance(ThemedReactContext reactContext) {
        if(mPlayer != null) {
            mPlayer.Close();
        }

        mReactContext = reactContext;

        return mPlayer;
    }

    @ReactProp(name = "src")
    public void setSrc(MediaPlayer player, String src) {
        if(!TextUtils.isEmpty(src)) {
            startRending(src);
        }
    }

    /**
     * 开始播放
     *
     * @param src
     */
    private void startRending(String src) {
        SharedSettings.getInstance(mReactContext).loadPrefSettings();
        SharedSettings sett = SharedSettings.getInstance();
        int connectionProtocol = sett.connectionProtocol;
        int connectionDetectionTime = sett.connectionDetectionTime;
        int connectionBufferingTime = sett.connectionBufferingTime;

        int decoderType = sett.decoderType;
        int rendererType = sett.rendererType;
        int rendererEnableColorVideo = sett.rendererEnableColorVideo;
        int rendererAspectRatioMode = mPanelIsVisible ? 0 : sett.rendererAspectRatioMode;
        int synchroEnable = sett.synchroEnable;
        int synchroNeedDropVideoFrames = sett.synchroNeedDropVideoFrames;

        isFileUrl = isUrlFile(src);

        currentPlayerCallBacks = new PlayerCallBacks();

        mPlayer.Open(src,
                connectionProtocol,
                connectionDetectionTime,
                connectionBufferingTime,
                decoderType,
                rendererType,
                synchroEnable,
                synchroNeedDropVideoFrames,
                rendererEnableColorVideo,
                rendererAspectRatioMode,
                isFileUrl ? 1 : mPlayer.getConfig().getDataReceiveTimeout(),
                sett.decoderNumberOfCpuCores,
                currentPlayerCallBacks);
    }
    
    private boolean isUrlFile(String url) {
        return (url != null && !url.isEmpty() &&
                (!url.contains("://") || url.contains("file://")));
    }

    private class PlayerCallBacks implements MediaPlayer.MediaPlayerCallback{
        public PlayerCallBacks(){}

        @Override
        public int Status(int i) {
            return 0;
        }

        @Override
        public int OnReceiveData(ByteBuffer byteBuffer, int i, long l) {
            return 0;
        }
    }
}
```

* note: REACT_CALSS很重要，需要和后面VXGPlayer.js文件中的名称保持一致

* 创建VXGPlayerPackage.java文件

```
public class VXGPlayerPackage implements ReactPackage {
    public VXGPlayerPackage() {}
    
    @Override
    public List<NativeModule> createNativeModules(ReactApplicationContext reactContext) {
        return Collections.emptyList();
    }

    @Override
    public List<ViewManager> createViewManagers(ReactApplicationContext reactContext) {
        return Arrays.<ViewManager>asList(
                new VXGPlayerViewManager(reactContext)
        );
    }
}
```

##### 5. 封装Native UI Component
* vxgplayer工程根目录下创建js目录，该目录下创建VXGPlayer.js文件
* 安装插件prop-types

```
import PropTypes from 'prop-types';
import { requireNativeComponent, View } from 'react-native';

var iface = {
  name: 'VXGPlayer',
  propTypes: {
      src: PropTypes.string,
      ...View.propTypes // include the default view properties
  },
};

module.exports = requireNativeComponent('RCTVXGPlayer', iface);
```

##### 6. 测试插件封装是否成功，app主工程引入组件（settting.gradle, build.gradle）
* app目录下build.gradle文件中添加依赖

```
compile project(':vxgplayer')
```

* RN工程index.android.js文件中引入该UI插件

```
import VXGPlayer from './android/vxgplayer/js/VXGPlayer'

this.state = {
    player_url: ''
}

render() {
    return (
        <View style={styles.container}>
            <VXGPlayer src={this.state.player_url} style={{ width:100, height:100}}/>
        </View>
    );
}
```

* note: <VXGPlayer/>标签中的style属性很重要，一定要设置width和height两个属性，不然不会显示视图！！！



