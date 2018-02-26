# 《ReactNative系列讲义》基础篇---Handling TextInput
**|** 版权声明：本文为博主原创文章，未经博主允许不得转载。
####一、简介
学完了数据控制，学完了设置样式，也学完了事件的处理，今天我们来学习第一个交互性组件：TextInput。该组件的应用过程中会使用到上面所有学过的知识，因此该组件也算是一个综合应用知识的案例。
####二、基础知识
* 基础属性：onChangeText 获取用户输入的内容，方法类型的属性（callback），用户输入的数据会以参数的形式返回给onChangeText属性
* 基础属性：placeholder 用户输入信息前显示，多用于提示
* 基础属性：multiline 设置多行输入
* 基础属性：value 设置默认值

####三、应用

```
import React, { Component } from 'react';
import { Text, TextInput, View } from 'react-native';

export default class PizzaTranslator extends Component {
  constructor(props) {
    super(props);
    // 因为用户输入的内容是实时改变的，所以我们将内容保存在状态中
    this.state = {text: ''};
  }

  render() {
    return (
      <View style={{padding: 10}}>
        <TextInput
          style={{height: 40}}
          // 输入提示内容
          placeholder="请输入数据"
          // text参数就是用户输入的数据
          onChangeText={(text) => this.setState({text})}
        />
        <Text style={{padding: 10, fontSize: 42}}>
           // 实时显示用户输入的数据
          {this.state.text}
        </Text>
      </View>
    );
  }
}
```

####四、总结
通过onChangeText和placeholder两个属性就可以制作一个简单的交互组件。在开发中，android平台TextInput组件自带下划线样式，下划线的样式会根据手机系统版本来定义，不可修改。如果想去掉，可使用underlineColorAndroid属性；如果想换上自己的下划线，可在TextInput组件外层包裹一层View。
本文只描述了一下常用到的，也是最基本的属性，还有很多能够实现个性化的属性，比如：设置弹出键盘的类型、自动转换大写、自动获取光标、是否可编辑、输入内容的最大长度等等，还有一些回调方法，处理各种事件。


