# 《ReactNative系列讲义》基础篇---State and Lifecycle
**|** 版权声明：本文为博主原创文章，未经博主允许不得转载。
####一、简介
上篇文章中我们一起学习了Components and Props，知道了ReactNative中两个基本概念：组件和属性，并且知道了两者之间的关系和使用方法与特性。这篇文章我们一起来认识一下ReactNative中的第二种数据类型：State。
State数据类型应用范围在组件内部，并且可依据需求实时变化并改变UI。

####二、基础概念
* state是私有的并且完全被组件所控制的
* this.state中的数据只能用于render视觉呈现，并且只能在构造方法中定义
* 改变state状态时使用setState方法
* 和props一样，state的更新可能是异步的，因此不可用于计算下一个状态
* 如有多个state值，使用setState方法更新时需要单独明确指定更新

####三、应用
下面通过一个时钟的例子来体会一下state的应用场景和使用方法
#####1. 先定义好需要的类
```
// 导入需要使用的组件
import React, { Component } from 'react';
import { StyleSheet, Text, View } from 'react-native';

// 创建输出类
export default class ClockTick extends Component {

    // 呈现UI视图
    render() {
        return(
            <View>
                // TODO 加载UI布局
            </View>
        );
    }
}
```

#####2. 引入生命周期相关概念
* 构造方法 constructor [props参数和super(props)不可缺少]

```
// ClockTick类中添加构造方法
constructor(props) {
    super(props);
    
    // 定义时间state
    this.state = {
        date: new Date()
    }
}
```

* 创建时钟方法

```
// 时钟运转方法（这是ES6语法的箭头函数，后面会讲到），调用该方法会改变date的值
tick = () => {
    this.setState({
        date: new Date()
    });
}
```

* 组件装备完毕之后的生命周期方法（也就是UI组件渲染完毕之后） componentDidMount
* 延伸知识点，this.props是由React创建的，this.state有特殊的意义，应该只定义用于呈现UI的数据状态值（即在render()方法中使用），非UI呈现数据可使用this.propsName方式定义

```
// UI组件加载完毕后，开始调用计时器
componentDidMount() {
    // 创建计时器，setInterval方法返回计时器ID，存储在this.timeID中
    this.timeID = setInterval(
       () => this.tick(),
       1000
    );
}
```

* 组件将要卸载之前的生命周期方法 componentWillUnmount

```
componentWillUnmont() {
    // 解除定时器，释放资源
    clearInterval(this.timeID);
}
```

* render方法调用

```
render() {
    return(
        <View>
            {this.state.date.getSeconds()}
        </View>
    );
}
```

#####3. 完整代码

```
// 导入需要使用的组件
import React, { Component } from 'react';
import { StyleSheet, Text, View } from 'react-native';

// 创建输出类
export default class ClockTick extends Component {
    constructor(props) {
        super(props);

        this.state = {
            date: new Date()
        };
    }
    
    // 时钟运转
    tick() {
        this.setState({
            date: new Date()
        });
    }
    
    componentDidMount() {
        // 创建定时器
        this.timeID = setInterval(
            () => this.tick(),
            1000
        );
    }
    
    componentWillUnmount() {
        // 解除定时器
        clearInterval(this.timeID);
    }

    // 呈现UI视图
    render() {
        return(
            <View>
                // TODO 加载UI布局
                <Text>{this.state.date.getSeconds()}</Text>
            </View>
        );
    }
}

```

#####四、总结
到这里ReactNative两种控制UI的数据类型已经给大家介绍完了。props只可读不可修改，用于组件之间的数据通信；state只应用于单一组件内部，属于私有的，可根据业务需求随时修改UI。只有在构造方法中才能够给this.state赋值。关于state的修改这里再补充一点：多个state单一修改或者同时修改的方式

```
// 同时定义多个state
this.state = {
   state_one: 'one'
   ,state_two: 'two'
}

// 只需改一个state
this.setState({
    state_one: 'state_one'
});

// 只需改一个state
this.setState({
    state_two: 'state_two'
});

// 需要同时需改多个state
this.setState({
    state_one: 'state_one'
    ,state_two: 'state_two'
});
```

前面已经说明了，props和state的更新有可能是异步的，因此不要通过props和state直接计算下一个状态值。
代码示范：

```
// 错误示范！！！
this.setState({
    counter: this.state.counter + this.props.increment
})
```

```
// 正确示范！！！
this.setState((prevState, props) => {
    counter: preState.counter + props.increment
})
```
第二种写法是setState()方法接受的另一种参数形式，即参数不再是对象而是方法；其中第一个参数是上一次的状态值，第二个参数是当前的属性值


