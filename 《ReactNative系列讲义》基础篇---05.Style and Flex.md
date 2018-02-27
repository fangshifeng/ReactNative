# 《ReactNative系列讲义》基础篇---05.Style and Flex

**|** 版权声明：本文为博主原创文章，未经博主允许不得转载。

#### 一、简介
每一个核心组件都自带一个style属性，用来添加组件的样式，比如字体大小、颜色、排列方式，组件的布局方式、位置等等。下面让我们一起来学习一下style样式的基本用法。
#### 二、基础知识
* style属性名称和CSS一样，只不过命名规则使用驼峰法。
* style属性最简单的方式是JavaScript对象类型，也可写成数组类型；在数组类型中，优先加载最后的元素，因此可应用于继承中。
* style的声明方式：StyleSheet.create({})
* 组件尺寸：一种通过给予Width和Height具体的数值确定组件的尺寸，一种是通过flex属性使组件在可获得的空间里面任意的伸缩。
* flex：当flex:1时组件会充满所有可用的空间，如果一个父元素内有几个子元素，可使用flex属性按比例分割父元素的空间，flex的值越大，子元素所占的比例越大。如果父元素未定义Width或Height或Flex值，即父元素尺寸等于零，那么子元素将不会显示。

#### 三、应用
* style的具体应用

```
import React, { Component } from 'react';
import { StyleSheet, Text, View } from 'react-native';

class LotsOfStyles extends Component {
  render() {
    return (
      <View>
        // 单独定义方式
        <Text style={{ color: 'blue', fontSize: '12' }}></Text>
        // 使用统一定义方式
        <Text style={styles.red}>just red</Text>
        <Text style={styles.bigblue}>just bigblue</Text>
        // 数组方式
        <Text style={[styles.bigblue, styles.red]}>bigblue, then red</Text>
        <Text style={[styles.red, styles.bigblue]}>red, then bigblue</Text>
      </View>
    );
  }
}

// 统一定义style属性
const styles = StyleSheet.create({
  bigblue: {
    color: 'blue',
    fontWeight: 'bold',
    fontSize: 30,
  },
  red: {
    color: 'red',
  },
});
```

* FixedDimensions的具体应用

```
import React, { Component } from 'react';
import { AppRegistry, View } from 'react-native';

export default class FixedDimensionsBasics extends Component {
  render() {
    return (
      <View>
        // 通过给予width和height具体数值的方式确定组件的尺寸
        <View style={{width: 50, height: 50, backgroundColor: 'powderblue'}} />
        <View style={{width: 100, height: 100, backgroundColor: 'skyblue'}} />
        <View style={{width: 150, height: 150, backgroundColor: 'steelblue'}} />
      </View>
    );
  }
}
```

```
import React, { Component } from 'react';
import { AppRegistry, View } from 'react-native';

export default class FlexDimensionsBasics extends Component {
  render() {
    return (
      // 通过flex方式确定组件的尺寸
      <View style={{flex: 1}}>
        <View style={{flex: 1, backgroundColor: 'powderblue'}} />
        <View style={{flex: 2, backgroundColor: 'skyblue'}} />
        <View style={{flex: 3, backgroundColor: 'steelblue'}} />
      </View>
    );
  }
}
```

* 如果去掉父组件View的flex属性，子组件也不会显示出来；
* 如果父组件View换成height属性，那么子组件会在规定高的布局内按比例分配
* 无论组件子元素的布局方式是纵向还是横向，父组件都要设置高或者flex:1


