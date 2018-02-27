# 《ReactNative系列讲义》高级篇---03.react-native-vxgplayer ReadMe

**|** 版权声明：本文为博主原创文章，未经博主允许不得转载。

#### Install 安装
npm i react-native-vxgplayer --save

#### Import 导入
###### Android Studio
* setting.gradle 

```
include ':react-native-vxgplayer'
project(':react-native-vxgplayer').projectDir = new File(settingsDir, '../node_modules/react-native-vxgplayer/android') 
```

* build.gradle

`compile project(':react-native-vxgplayer')`

* MainApplication

`new VXGPlayerPackage()`

#### Usage 使用方法

`import VXGPlayer from 'react-native-vxgplayer'`

