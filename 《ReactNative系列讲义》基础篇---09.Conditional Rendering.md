# 《ReactNative系列讲义》基础篇---09.Conditional Rendering
**|** 版权声明：本文为博主原创文章，未经博主允许不得转载。
#### 一、简介
前面几篇文章介绍了React的数据类型和组件的概念，因此我们知道在开发时，我们可以将一些功能拆分出来，哪怕很小的功能，都是可以拆分出来的。当我们使用的时候，会根据状态值来判断我们需要加载哪一部分组件，不加载哪一部分组件。今天，我们就一起来看看根据条件渲染UI组件。
React中根据条件渲染组件的工作原理和JavaScript中根据条件执行相关逻辑的工作原理是一样的。
下面，我们通过判断当前用户是登录用户还是游客来显示不同的UI提示这么一个Demo来讲解一下根据条件加载UI的思想到底是什么。
#### 二、代码讲解
##### 1、创建主框架

```
import React, { Component } from 'react';
import { View, TouchableOpacity, Text } from 'react-native';

export default class ConditionalRendering extends Component {
    constructor(props) {
        super(props);

        this.state = {
            isLoginIn: false
        }
    }

    render() {
        const isLoginIn = this.state.isLoginIn;
        // 使用三元运算简化代码 condition ? true : false
        // UI展示逻辑：已登录，显示logout字样；未登录，显示login字样
        // 在这里分离出两个子组件<LogoutButton/><LoginButton/>
        // 两个子组件只负责UI的展示，事件的处理放在父组件中，作为属性由父组件传递给子组件，这样子组件便可以在任何的地方重复使用了。
        return(
            <View>
                {isLoginIn ? <LogoutButton onClick={() => {
                    this.setState({ isLoginIn: false });
                }}/> : <LoginButton onClick={() => {
                    this.setState({ isLoginIn: true });
                }}/>}
            </View>
        );
    }
}
```

##### 2、创建子组件

```
class LoginButton extends Component {
    constructor(props) {
        super(props);
    }

    render() {
        return(
            <TouchableOpacity onPress={this.props.onClick}>
                <Text>Login</Text>
            </TouchableOpacity>
        );
    }
}

class LogoutButton extends Component {
    constructor(props) {
        super(props);
    }

    render() {
        return(
            <TouchableOpacity onPress={this.props.onClick}>
                <Text>Logout</Text>
            </TouchableOpacity>
        );
    }
}
```

#### 三、总结
组件的分离思想遍布在整个项目的开发周期中，刚开始可能会有些不得要领，对于思想上的领悟略显生涩，运用上不是很得心应手，大家不要着急，只要你在平时的开发中时刻记着这个开发思想，慢慢的你会发现将一个复杂的组件拆分成若干个小组件是一件很容易的事情。分离出来的组件为的是能够在任何地方都能够重复使用，那这个组件必定只决定公共部分的功能（也可以说是永不变的功能），需要根据业务场景随时改变的功能通过属性功能暴露给父组件，由父组件定义完成后传递给子组件（即子组件将这部分功能的定义权交给父组件）。


