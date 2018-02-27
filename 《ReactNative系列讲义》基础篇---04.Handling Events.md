# 《ReactNative系列讲义》基础篇---04.Handling Events
**|** 版权声明：本文为博主原创文章，未经博主允许不得转载。
####一、简介
学完了数据，接下来我们学学方法的创建与调用，以及触发方法的载体。
####二、基础概念
#####1. 触发Event事件的载体
ReactNative组件：Button, Touchables（包括：TouchableHighlight, TouchableNativeFeedback, TouchableOpacity, TouchableWithoutFeedback）
#####2. 处理Event事件的方式
处理Event事件时，一定要注意绑定该事件。这不是React特有的功能，而是JavaScript基础语法，在JS中，事件不会自动绑定到当前类，因此需要手动绑定。
####三、应用
#####1. 通过Button标签的使用，学习第一种Event事件的处理方式。Button标签触发事件的属性名称叫做onClick

```
import React, { Component } from 'react-native';
import { Button, Alert } from 'react-native';

export default class ButtonAndEvent extends Component {
    constructor(props) {
        super(props);
        this.handleEvent = this.handleEvent.bind(this);
    }
    
    // 处理Button Event
    handleEvent() {
        Alert.alert('You tapped the button!')
    }
    
    render() {
        return(
            <View>
                <Button 
                    title="Press Me"
                    onPress={this.handleEvent} 
                    color="#841584"
                />
            </View>
        );
    }
}
```

#####2. 通过TouchableOpacity标签的使用学习第二种绑定事件的方式
```
import React, { Component } from 'react-native';
import { Alert, TouchableOpacity, Text } from 'react-native';

export default class ButtonAndEvent extends Component {
    constructor(props) {
        super(props);
    }
    
    // 处理Button Event
    handleEvent = () => {
        Alert.alert('You tapped the button!')
    }
    
    render() {
        return(
            <View>
                <TouchableOpacity onPress={this.handleEvent}>
                    <Text>Press Me</Text>
                </TouchableOpacity>
            </View>
        );
    }
}
```
* 前两种方式都是不带参数的情况下处理事件，如果处理事件的同时需要带参数，那么需要用到下面这种方式

#####3. 处理带参数的事件（onPress属性中使用箭头函数或者直接绑定）
```
import React, { Component } from 'react-native';
import { Alert, TouchableOpacity, Text } from 'react-native';

export default class ButtonAndEvent extends Component {
    
    // 处理Button Event
    handleEvent(msg) {
        Alert.alert(msg)
    }
    
    render() {
        return(
            <View>
                // 第一种方式：使用箭头函数
                <TouchableOpacity onPress={() => { this.handleEvent('hello handleEvent') }}>
                    <Text>Press Me</Text>
                </TouchableOpacity>
                // 第二种方式：bind绑定的方式
                <TouchableOpacity onPress={this.handleEvent.bind(this, 'hello handleEvent')}>
                    <Text>Press Me</Text>
                </TouchableOpacity>
            </View>
        );
    }
}
```
* 注：事件处理中存在着人造事件e；上面代码中第一种方式如果需要传递e参数时，需要将参数e放在‘hello handleEvent’参数后面；第二种方式会自动传递参数e，在接收的时候也是放在'hello handleEvent'参数后面。

代码示例

```
// 参数e添加的方式
<TouchableOpacity onPress={() => { this.handleEvent('hello handleEvent', e) }}>
    <Text>Press Me</Text>
</TouchableOpacity>

// 接受参数
handleEvent(msg, e) {
    Alert.alert(msg)
}
```


#####4. 几种异常情况分析
######（1）正常情况总结
* onPress属性方法中只要未使用箭头函数调用方法时，方法名后面都不可以带小括号。
* 使用前两种方法处理事件时，不可传递参数；如果想传递参数就必须使用第三种方式
* 在不需要传递参数时，尽量使用前两种方式

######（2）异常情况总结
* 前两种方式如果在调用方法时添加了小括号，事件处理会在加载完页面之后自动执行。
* 在最新版本中Button或者TouchableOpacity标签中，onPress属性在调用事件处理方法时使用第一种方式，但是未在构造方法中绑定。使用this.handleEvent方式也可调用事件处理方法，如果后面也加上小括号，同样会出现页面加载完毕，自动加载事件处理方法的状况。
* 第三种方式存在一定问题，每当Button或者TouchableOpacity绘制的时候，都会创建一个不同的callback。大多数情况下，这种方法是没有问题的。但是当callback被当做属性传递到下级组件的时候，这些下级组件会执行多余的重新绘制动作。

####四、总结
综上所述，在无需传递参数的时候尽量使用前两种方式调用事件处理方法，需要传递参数时，使用第三种方式调用。


