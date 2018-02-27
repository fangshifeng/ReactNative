# 《ReactNative系列讲义》高级篇---02.制作npm插件并发布

**|** 版权声明：本文为博主原创文章，未经博主允许不得转载。

#### 一、简介
npm组件制作完成，下一步需要提交到npm服务器。这样，无论是自己还是其他开发者，都可以通过npm命令进行组件的安装。
#### 二、流程
##### 1. 创建新文件夹
创建新文件夹（独立的）react-native-vxgplayer
##### 2. 创建新目录
react-native-vxgplayer文件下创建新路径android，将RNVXGPlayer项目android工程的vxgplayer module下所有的文件拷贝到上面的android目录下；将js文件夹及其子文件全部拷贝到react-native-vxgplayer的根目录下。
##### 3. 创建入口文件
在react-native-vxgplayer根目录创建index.js文件

```
import _VXGPlayer from './js/VXGPlayer';

export default VXGPlayer = _VXGPlayer;
```

##### 4. 创建README文件
README文件创建完成之后，插件的目录结构基本完成，如下图：
![屏幕快照 2017-08-22 下午2.13.19](media/15033109632144/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-08-22%20%E4%B8%8B%E5%8D%882.13.19.png)

##### 5. 创建npm账号
##### 6. npm login
在命令窗口中cd到react-native-vxgplayer根目录下，输入命令行 `npm login` 登录npm服务器，需要输入username，password，email
##### 7. npm init
react-native-vxgplayer根目录下输入命令行 `npm init` 输入一些配置信息，完成之后会生成package.json文件

```
{
  "name": "react-native-vxgplayer",
  "version": "0.0.1",
  "description": "react-native android rtsp",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "repository": {
    "type": "git",
    "url": "git+https://github.com/fangshifeng/react-native-vxgplayer.git"
  },
  "keywords": [
    "react-native",
    "android",
    "rtsp"
  ],
  "author": "fsf",
  "license": "ISC",
  "bugs": {
    "url": "https://github.com/fangshifeng/react-native-vxgplayer/issues"
  },
  "homepage": "https://github.com/fangshifeng/react-native-vxgplayer#readme"
}

```

##### 8. npm publish
还是根目录下，输入命令行 `npm publish` 将插件发布到npm服务器上，供其他开发者使用

##### 9. Install 安装

npm i react-native-vxgplayer --save

##### 10. Import 导入
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

##### 11. Usage 使用方法

`import VXGPlayer from 'react-native-vxgplayer'`


