# 《ReactNative实战讲义》Redux框架篇---03.基础知识
**|** 版权声明：本文为博主原创文章，未经博主允许不得转载。

Redux框架篇系列文章总目录：
[《ReactNative实战讲义》Redux框架篇---（一）基础知识](http://blog.csdn.net/fsf_snail/article/details/79082351)
[《ReactNative实战讲义》Redux框架篇---（二）基础应用](http://blog.csdn.net/fsf_snail/article/details/79156288)
[《ReactNative实战讲义》Redux框架篇---（三）高级知识](http://blog.csdn.net/fsf_snail/article/details/79156393)
[《ReactNative实战讲义》Redux框架篇---（四）高级应用](http://blog.csdn.net/fsf_snail/article/details/79162619)

### 一、简介
今天给大家介绍一个新的框架---Redux，这个框架在项目中的位置很重要，基本可以算作项目基础框架的核心。在以后的项目中我们会围绕Redux框架打造一个框架群，因此Redux框架战略地位很重要，当然理解起来还是有相当程度的难度，本篇文章为大家介绍一些关于Redux的基础概念，我会通过一系列的文章带大家认识Redux，理解Redux，使用Redux，首先我们先来看看Redux是个什么东东。

### 二、Redux被创造出来的目的是什么？
无论是做过React开发，还是ReactNative开发的小伙伴应该都清楚在开发的过程中每个页面中或多或少都会出现状态（State），这是React的基础元素。随着项目不断的迭代，功能不断地增加，复杂程度不断地提高，有越来越多的State值需要我们去管理，复杂的会遇到多页面共用一个状态值，反过来说，一个状态值会影响多个页面的显示。这时，对于状态的管理要求就更高了。而Redux的出现就是为了解决这一问题。

讲到这里我们知道了Redux的出现和项目的State有关，那么它能做到什么呢？

将所有的State变化进行统一流程的处理，会使我们的程序状态变化清晰可变。Redux的最终目的就是让状态变化变得可预测。

### 三、核心理念
* Redux将应用的state状态集中管理；
* 唯一能够修改状态的方法是通过发起action动作的方式修改；
* 通过reducer将state和action连接起来，根据action的类型创建相应新的state；
* React根据新的State修改UI；

以上几点串联起来就是Redux框架工作的流程，这只是笼统的概括，细节的地方我们后面会讲到。

### 四、基础元素

Redux框架使用时的三大元素

#### 1. Action
* 类型：普通JS对象
* 参数：type字段，表示要执行的动作，多数情况下会被定义成字符串常量
* 代码示例：
* 深度解释：state只读，唯一修改state的方式是发起action动作，action动作实质上是个普通对象，正因为他包含了动作类型（type），需要修改的数据内容（比如，id，data...）。记住，action只是描述有事情发生了并且携带了相关数据元素。他并不会描述应用如何更新state。
* 代码示例：

```
// 定义区分动作类型的常量
const LOAD_LIST = 'LOAD_LIST';
// 不成文的规定第一个参数我们设置成type，后面表示发起Action的类型
// 比如：LOAD_LIST 加载列表内容，response后面所带内容为需要加载的列表数据
// 原则上这个普通的JS对象里面的参数都是可以自定义的，根据项目实际需求来定。
{ type: 'LOAD_LIST', response: { ... } }

// 实战中创建ActionCreater的标准样式
export const getList = (listData) => {
    return { type: 'LOAD_LIST', response: listData } 
}
```

#### 2. Reducer
* 类型：纯函数
* 作用：将action和state结合在一起。接受Action，根据Action中所承载的object数据，创建新的state并返回。
* 任务：reducer要完成的任务仅仅是接受action和旧的state，返回新的state
* 禁止：修改传入的参数，执行副作用的操作（API请求和路由跳转）,调用非纯函数（如：Data.now()和Math.random()）
* 返回值：返回值相当于旧的state在末尾加上新建的state。而这个新的state又是基于action中携带的数据创建的。
* 代码示例：

```
// 初始化的State值
let initialList = { list: [] }

// 前面讲到了Action就是一个普通的JS对象，里面包含了自定义的参数。
export const listData = (state = initialList, action) => {
    switch(action.type) {
        case: 'LOAD_LIST':
            return {
                ...state,
                list: action.response
            }
            
        return state;
    }   
}
```

#### 3. Store
* 作用：将action和reducers联系在一起
* 功能：维持存储应用的state
* API：
* getState()方法获取state
* dispatch(action)方法更新state
* subscribe(listener)注册监听器
* subscribe(listener)返回的函数注销监听器
* 特点：全局只有一个单一的store
* 代码示例

```
import { createStore } from 'redux';
import { listData } from './reducers';
import { getList } from './actions';
let store = createStore(listData);

// 打印初始状态
console.log(store.getState());
// 每次state更新时，打印日志
// 注意subscribe()返回一个函数用来注销监听器
const unsubscribe = store.subscribe(
    () => console.log(store.getState())
);
// 监听一系列action
store.dispatch(getList(
[
    {id: 1, name: 'bob'}, 
    {id: 2, name: 'fsf'}
]));
// 停止监听state更新
unsubscribe();
```

注：关于Store做了些什么工作，我们后续讲解。

### 4. 框架特性（设计核心）
**严格的单向数据流**

-------

下面我们整体串联一下Redux框架每个使用步骤，从中体会一下数据流单向流动的特点。

#### 1. 用户触发按钮或者其他动作，调用store.dispatch(action)；
* Action就是一个描述了“发生了什么”的普通对象，同时也是数据的载体。

示例代码：

```
{ type: 'LIKE_ARTICLE', articleId:42 }
{ type: 'FETCH_USER_SUCCESS', response: { id: 3, name: 'bob' } }
{ type: 'ADD_TODO', text: 'Read the Redux docs' }
```

#### 2. Redux store 调用传入的reducer函数
* Store会把两个参数传入reducer：当前的state树和action。

内部代码示例

```
// 当前应用的 state（todos 列表和选中的过滤器）
 let previousState = {
   visibleTodoFilter: 'SHOW_ALL',
   todos: [
     {
       text: 'Read the docs.',
       complete: false
     }
   ]
 }

 // 将要执行的 action（添加一个 todo）
 let action = {
   type: 'ADD_TODO',
   text: 'Understand the flow.'
 }

 // reducer 返回处理后的应用状态
 let nextState = todoApp(previousState, action);
```
* reducer是纯函数，它仅仅用于计算下一个state，即多次传入相同的输入必须产生相同的输出。它不应该有副作用的操作，如API调用或路由跳转，这些应该在dispatch action前发生。

#### 3. 根reducer应该把多个子reducer输出合并成一个单一的state树。
* 根reducer的结构完全由你决定，Redux原生提供combineReducers()辅助函数，来把根reducer拆分成多个函数，用于分别处理state树的一个分支。

```
// 以下代码演示了combineReducers()函数的使用方法
function todos(state = [], action) {
   // 省略处理逻辑...
   return nextState;
 }

 function visibleTodoFilter(state = 'SHOW_ALL', action) {
   // 省略处理逻辑...
   return nextState;
 }

 let todoApp = combineReducers({
   todos,
   visibleTodoFilter
 })
```

* 当你触发action时，combineReducers()返回的todoApp会负责调用两个reducer

```
let nextTodos = todos(state.todos, action);
let nextVisibleTodoFilter = visibleTodoFilter(state.visibleTodoFilter, action);
```

* 然后会把两个结果集合并成一个state树：

```
 return {
   todos: nextTodos,
   visibleTodoFilter: nextVisibleTodoFilter
 };
```

#### 4. Redux store 保存了根reducer返回的完整state树
* 这个新的树就是应用的下一个state！所有订阅store.subscribe(listener)的监听器都将被调用；监听器里可以调用store.getState()获取当前state，最后根据新的state更改UI。

* 小结：以上就是Redux框架的核心工作流程，其中也蕴含着Redux的设计理念。

