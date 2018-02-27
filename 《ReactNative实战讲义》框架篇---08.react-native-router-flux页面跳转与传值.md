# 《ReactNative实战讲义》框架篇---08.react-native-router-flux页面跳转与传值
#### 一、内容简介
实现以下功能：
1. 正向跳转
2. 正向跳转并传值
3. 反向跳转
4. 反向跳转并传值
5. 指定页面跳转
6. 指定页面跳转并传值

#### 二、代码实现
##### 1、正向跳转
假设情景：从Home页跳转到Profile页面，Profile场景的key值为profile

* 不带参数 `Actions.profile`
* 带参数`Actions.profile({'key':value})`

接收参数：
`this.props.KEY_NAME`

eg：
通过TouchableOpacity的onPress方法实现页面的跳转

`onPress={Actions.proflis} // 不带参数的最简写法`
`onPress={() => {Actions.proflie({'key':value})}} // 带参数的最简写法，传递的参数必须是Object类型，每个参数建议使用键值对方式传递`
`this.props.key // 接收参数`

##### 2、反向跳转
假设情景：从Profile页返回Home页面

* 返回上一页面，不带参数`Actions.pop()`
* 返回上一页面，带参数`Actions.pop({refresh:({’key‘:value})})`
* 指定回退页面数`Actions.pop({popNum:2})`
* 指定回退页面数，带参数`Actions.pop({popNum:2, refresh:({'key':value})})`
* 返回指定页面`Actions.popTo('home')`

注释：

* refresh是框架自带函数，可用于刷新属性（props）
* `Actions.pop({refresh:({'key':value})}) // 用于刷新回退到的页面的属性`
* `Actions.refresh(’params‘) // 用于刷新当前页面的属性对应回退页面刷新属性，即接受传递的参数`

接收参数：
 
```
// 1. 必须在componentWillReceiveProps(nextProps)生命周期中接受传递的参数
// 2. 该生命周期方法中的参数必须叫做nextProps
// 3. 所有传递过来的参数都包含在nextProps参数中
// 4. 以nextProps.PARAM_NAME的方式获取指定的参数
componentWillReceiveProps(nextProps) {
    // 假设前一个页面传递过来一个名字叫做isRefresh的布尔型参数
    if(nextProps.isRefresh) {
        // TODO 根据需求执行相关操作
        ......
    }
}
```









