# 《ReactNative实战讲义》Redux框架篇---05.高级知识
**|** 版权声明：本文为博主原创文章，未经博主允许不得转载。

Redux框架篇系列文章总目录：
[《ReactNative实战讲义》Redux框架篇---（一）基础知识](http://blog.csdn.net/fsf_snail/article/details/79082351)
[《ReactNative实战讲义》Redux框架篇---（二）基础应用](http://blog.csdn.net/fsf_snail/article/details/79156288)
[《ReactNative实战讲义》Redux框架篇---（三）高级知识](http://blog.csdn.net/fsf_snail/article/details/79156393)
[《ReactNative实战讲义》Redux框架篇---（四）高级应用](http://blog.csdn.net/fsf_snail/article/details/79162619)

### 一、简介
Redux本身是同步的Action，但是实战项目中同步是不能满足我们的需求的，反而我们大部分的Action都是异步的，例如：API请求。在介绍Redux的异步用法前我们先了解一下异步和同步的区别。

* Action发出后，Reducer立刻算出新的State，这叫做同步；
* Action发出后，过一段时间再执行Reducer，这叫做异步；

接下来我们以异步调用API为例子，整体讲解一下异步Action。

### 二、详情讲解
#### 1.异步Action请求使用场景分析

异步调用API时，有两个非常关键的时刻：发起请求的时刻，接受到响应的时刻。
一般情况下，这种异步调用API请求都需要dispatch至少三种action：

 * 通知reducer请求开始的action
 * 通知reducer请求成功的action
 * 通知reducer请求失败的action
 
#### 2.如何实现异步功能
这里需要引入一个新的概念：中间件
目前围绕着Redux框架可使用的中间件相当多，这里介绍一下能够帮助我们实现异步Action的中间件：redux-thunk

##### （1）改造原理：

* 将之前定义的同步action创建函数和网络请求结合起来；
* 使用redux-thunk中间件，action创建函数除了返回action对象外还可以返回函数，此时，这个action创建函数就成为了thunk；
* 默认情况下，createStore()所创建的Redux store没有使用middleware，所以只支持同步数据流，通过使用applyMiddleware()来增强createStore()，可以帮助你用简便的方式来描述异步的action；

##### （2）react-thunk工作方式
* redux-thunk支持异步action，内部包装了store的dispatch()方法，以此来让你dispatch一些除了action以外的其他内容，例如：函数。
* 你所使用的任何中间件都可以以自己的方式解析你dispatch的任何内容，并继续传递actions给下一个中间件；
* 当中间件链中的最后一个中间件开始dispatch action时，这个action必须是一个普通对象，这是同步式Redux数据流开始的地方。

##### （3）代码示例
* 改造createStore

```
import { createStore, applyMiddleware } from 'redux';
import thunk from 'redux-thunk';
import AppReducer from '../reducers/index';

const store = createStore(
    AppReducer,
    // 增加了applyMiddleware()和react-thunk
    applyMiddleware(
        thunk, // 允许我们 dispatch() 函数
    )
);

export const configureStore = () => {
    // console.log(store.getState());
    // store.subscribe(() =>
    //     console.log(store.getState())
    // );

    return store;
};
```

* 改造API请求方法

```
export const getArticlesList = () => (dispatch) => {
    // TODO dispatch(); 准备请求服务器数据Action
    return fetch(url, {
            method: 'GET',
            headers: {}
        }).then((response)=>{
            if (response.ok) {
                return response.json();
            } 
        }).then((responseJson)=>{
            // TODO dispatch(ActionCreator); 获取网络数据成功Action
        }).catch((error)=>{
            // TODO dispatch(ActionCreator); 网络请求失败Action
        });
};
```

#### 3. 小结
本篇文章介绍了Redux高级用法中最核心的一点，异步Action；引入了Redux高级用法的新概念，中间件。但是本文讲解的并不细致与全面，本文旨在让大家接受一些关于Redux高级用法的概念、方法、形式，后面我们会通过完整的Demo为大家讲解Redux高级应用的具体实现的样式。



  


