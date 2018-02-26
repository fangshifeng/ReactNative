#《ReactNative系列讲义》基础篇---Components and Props
**|** 版权声明：本文为博主原创文章，未经博主允许不得转载。
####一、简介
Components能够让你将UI独立分开，碎片化并重复使用。今天我们通过学习ReactNative中的数据类型来看看如何创建独立的UI Components。RN中控制组件的数据类型有两种，一种作用于组件间，叫做Props；一种作用于组件内部，叫做State。这篇文章我们先来学习一下组件间数据类型Props，也正是因为有了Props，UI组件之间才可以互相传递数据。

####二、基础概念
* Props属性可以帮助你制作单独的组件，可以使得该组件在App的任何地方使用。
* Props属性由父组件设置（即调用该独立组件的组件），并且稳定的存在于这个组件的生命周期里面（不可改变，即只读性）
* Props属性的更新可能是异步的，因此不可用于计算下一个状态

####三、应用
* 本文及其以后文章所写语法均为ES6语法

#####1. 引入你需要的组件或者类
```
// 引入react包的React对象及其Component组件
import React, { Component } from 'react';
// 引入react-native包中组件
import { StyleSheet, Text, View } from 'react-native';
```
#####2. 创建类组件，ES6语法，通过构建类创建独立UI组件
```
// 类的基本结构
class Welcome extends Component{
    render() {
        return(
            <Text>Hello,{this.props.name}</Text>
        );
    }
}
```
#####3. 创建唯一输出类（最顶级的类）并调用类组件

```
export default class ClockTick extends Component {
    render() {
        return (
            <View>
                // 独立组件可重复调用
                <Welcome name="蜗牛慢慢跑1"/>
                <Welcome name="蜗牛慢慢跑2"/>
                <Welcome name="蜗牛慢慢跑3"/>
            </View>
        );
    }
}
```
调用过程解读：ClockTick类调用Welcome组件，其中name属性赋值“蜗牛慢慢跑”，这个值传递到了Welcome组件中的this.props.name中，加上前面的字符，最后组合成“Hello，蜗牛慢慢跑”。return方法又将这组字符串传递出去，因此调用Welcome的ClockTick类会输出“Hello，蜗牛慢慢跑”字样。

注：

* 一个js文件中只能有一个默认输出的类（最顶级的类），关键字：export default
* 类组件的命名规则：首字母必须大写
* 可多次重复调用类组件
* 只读性或者纯净性解读：只要调用该独立组件的父组件传递的props值相同，该独立组件返回的值也应该相同，即在该独立组件内部，props属性值不可改变！
* Components其实是在提倡一种思想，不要害怕组件分离的太小，如果某部分UI会被多次使用，还是推荐封装成UI组件，例如：Button、Avatar

####四、总结
在这一节里，我们讲述了第一种控制数据的方式以及创建一个类的基本要素。通过这一节学到的知识再加上上节课中的JSX知识，你完全可以写出一个属于你自己的第一个ReactNative页面了，是不是有些小激动呢。在下节课中，我们将继续学习控制数据的第二种方式。

