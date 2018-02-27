# 《ReactNative系列讲义》进阶篇---01.Refs

**|** 版权声明：本文为博主原创文章，未经博主允许不得转载。

#### 一、简介
有时我们需要组件的非典型数据或者方法，并不是以属性的方式出现，而是通过组件的引用调用。一般常见的使用场景有：管理焦点、文本选择、媒体播放、触发必要的动画、合并第三方库等等。下面让我们通过几个简单的demo来认识一下refs。
#### 二、基础知识
* ref是组件的一个特殊属性，任何一个组件都带有该属性。
* ref属性通过callback获取组件引用。
* callback调用时机：在组件装备完成或者卸载完成之后执行。
* 获取引用的情况：1. 同类别中 2. 子组件暴露引用给父组件

#### 三、应用
* 同类别中获取组件引用

```
import React, { Component } from 'react';
import { TextInput, TouchableOpacity, View, Text } from 'react-native';

export default class RefsBasic extends Component {
    _handleClick = () => {
        if(this.textInput) {
            // 使输入框获取光标
            this.textInput.focus();
        }
    };

    render() {
        return(
            <View style={{ padding:20 }}>
                <TextInput
                    ref={ input => this.textInput = input }
                    style={{ height: 40 }}
                    placeholder="请输入数据"
                />
                <TouchableOpacity onPress={ this._handleClick }>
                    <Text>Click</Text>
                </TouchableOpacity>
            </View>
        );
    }
}
```

* 子组件暴露引用给父组件，在父组件中操作子组件相关属性和方法

```
import React, { Component } from 'react';
import { TextInput, TouchableOpacity, View, Text } from 'react-native';

export default class RefsBasic extends Component {
    // 处理点击事件
    _handleClickCustom = () => {
        if(this.customTextInput) {
            // 使输入框获取光标
            this.customTextInput.focus();
        }
    };

    render() {
        return(
            <View style={{ padding:20 }}>
                // 引用子组件
                <CustomTextInput
                    // 获取子组件的引用
                    inputRef={ ref => this.customTextInput = ref }
                />
                <TouchableOpacity onPress={ this._handleClickCustom }>
                    <Text>Click Custom</Text>
                </TouchableOpacity>
            </View>
        );
    }

}

// 自定义子组件
class CustomTextInput extends Component{
    render() {
        return(
            <TextInput
                // 子组件通过props形式暴露自己的引用
                ref={ this.props.inputRef }
                style={{ height: 40 }}
                placeholder="请输入数据"
            />
        );
    }
}
```


