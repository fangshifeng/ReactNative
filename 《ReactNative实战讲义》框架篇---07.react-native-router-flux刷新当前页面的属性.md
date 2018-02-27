# 《ReactNative实战讲义》框架篇---07.react-native-router-flux刷新当前页面的属性
## 一、内容简介
给NavigationBar左上角或右上角添加点击事件
## 二、代码实现
1. 使用到的框架属性

* leftTitle：左上角按钮的名字
* onLeft：左上角按钮的点击事件
* rightTitle：右上角按钮的名字
* onRight：右上角按钮的点击事件
* 其他属性也可进行刷新

eg：

```
componentDidMount() {
    Actions.refresh({
        leftTitle: '添加',
        onLeft: () => {
            // TODO 相关操作
        },
        rightTitle: '减少',
        onRight: () => {
            // TODO 相关操作
        }
    });
}
```

