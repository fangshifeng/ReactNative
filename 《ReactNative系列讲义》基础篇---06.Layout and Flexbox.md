# 《ReactNative系列讲义》基础篇---06.Layout and Flexbox
**|** 版权声明：本文为博主原创文章，未经博主允许不得转载。
#### 一、简介
上一篇中介绍了组件尺寸的定义方式，今天聊聊组件位置的定义方式。首先是在父组件中确定子组件的布局方式，然后选择子组件在主轴上的排列方式和次轴的排列方式。Flexbox的设计初衷是为了使布局适应不同尺寸的屏幕。除个别例外，其他的方面都和CSS中的思想保持一致。
#### 二、基础知识
* flexDirection：决定主轴的布局方式；两种布局方式：横向(row)和纵向(column)；默认是纵向，这也是和CSS中不同的方面之一。
* justifyContent：决定子元素在主轴上的分布样式；五种样式：flex-start, center, flex-end, space-around, space-between
* alignItems：决定子元素在次轴上的分布方式；四种样式：flex-start, center, flex-end, stretch（注：使用stretch属性时，子组件在次轴上不能有固定的尺寸）
* flex：只支持单个号码；这也是和CSS的不同之处。

#### 三、应用

```
import React, { Component } from 'react';
import { View } from 'react-native';

export default class JustifyContentBasics extends Component {
  render() {
    return (
      // 主轴方向是纵向
      // 在纵向方向子元素均分主轴屏幕的可见长度
      <View style={{
        flex: 1,
        flexDirection: 'column',
        justifyContent: 'space-between',
      }}>
        <View style={{width: 50, height: 50, backgroundColor: 'powderblue'}} />
        <View style={{width: 50, height: 50, backgroundColor: 'skyblue'}} />
        <View style={{width: 50, height: 50, backgroundColor: 'steelblue'}} />
      </View>
    );
  }
};
```

* justifyContent: flex-start 主轴的起点
* justifyContent: center 主轴的中心
* justifyContent: flex-end 主轴的末尾
* justifyContent: space-around 延主轴方向子元素左右等距分布
* justifyContent: space-between 延主轴方向均分主轴

```
import React, { Component } from 'react';
import { View } from 'react-native';

export default class AlignItemsBasics extends Component {
  render() {
    return (
      // 主轴方向是纵向
      // 主轴方向居中
      // 次轴方向居中
      // 最终效果：所有的子元素聚集在屏幕中央
      <View style={{
        flex: 1,
        flexDirection: 'column',
        justifyContent: 'center',
        alignItems: 'center',
      }}>
        <View style={{width: 50, height: 50, backgroundColor: 'powderblue'}} />
        <View style={{width: 50, height: 50, backgroundColor: 'skyblue'}} />
        <View style={{width: 50, height: 50, backgroundColor: 'steelblue'}} />
      </View>
    );
  }
};
```

* alignItems: flex-start 次轴的起点
* alignItems: center 次轴的中心
* alignItems: flex-end 次轴的末尾
* alignItems: stretch 延伸，即充满可充满的空间，有些类似flex，不过stretch更加灵活；该属性在使用的时候不可在次轴方向设置该子元素的宽度


