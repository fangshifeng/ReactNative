# 《ReactNative实战讲义》框架篇---01.Flux框架基础
**|** 版权声明：本文为博主原创文章，未经博主允许不得转载。
#### 一、基础概念介绍
##### 1.register(function callback):string
注册监听器，用于监听任何的dispatched payload
##### 2.unregister(string id)
取消某个dispatch的监听
##### 3. waitFor(array<string> ids):void
可以使当前执行的dispatch，等到某个指定的dispatch执行完之后再执行
##### 4.dispatch(object payload):void
发送一个dispatch给所有的监听器
##### 5.isDispatching():boolean
判断当前的dispatch是哪个
#### 二、基础功能封装
##### 1.创建AppDispatcher.js文件
作用：封装并暴露各种方法
###### (1) 引入Flux框架
`import { Dispatcher } from 'flux';`
###### (2) 实例化Dispatcher类
`const flux = new Dispatcher();`
###### (3) 暴露各种方法
注册监听方法

```
export function register(callback) {
    return flux.register(callback);
}
```

等待方法

```
export function waitFor(ids) {
  return flux.waitFor(ids);
}
```

分发方法：分发单个动作

```
export function dispatch(type, action = {}) {
  if (!type) {
    throw new Error('You forgot to specify type.');
  }
  if (process.env.NODE_ENV !== 'production') {
    if (action.error) {
      console.error(type, action);
    } else {
      console.log(type, action);
    }
  }

  flux.dispatch({ type, ...action });
}
```

分发方法：分发promise类型的三个动作

```
export function dispatchAsync(promise, types, action = {}) {
  const { request, success, failure } = types;

  dispatch(request, action);
  promise.then(
    response => dispatch(success, { ...action, response }),
    error => dispatch(failure, { ...action, error })
  );
}
```
##### 2、创建ActionTypes.js文件
###### (1) 引入keymirror框架
`import keyMirror from 'keymirror';`

框架介绍
通过该框架创建的对象，对象中的value值等于key值，通俗的讲，当你创建{key:value}时，key===value

###### (2) 代码实现
```
export default keyMirror({
  USER_LOGIN: null,
  USER_LOGIN_SUCCESS: null,
  USER_LOGIN_ERROR: null,
  USER_INFO_UPDATE_NAME: null,
});
```
##### 3、实际应用
######（1） register()方法的实现
```
import { register } from 'AppDispatcher';
import ActionTypes from 'ActionTypes';

register(action => {
      switch(action.type) {
        case 'USER_INFO_UPDATE_NAME':
          this.setState({
            nickName: action.response
          });
         break;
      }
    });
```

使用说明：
register()方法的参数是function类型，demo中使用了箭头方法的方式，接收到的内容都在箭头方法的action参数中，根据action的类型进行相关操作，这里是修改用户昵称
######（2） dispatch()方法的实现
```
import { dispatch } from 'AppDispatcher';
import ActionTypes from 'ActionTypes';

dispatch(ActionTypes.USER_INFO_UPDATE_NAME, {response: this.state.nick});
```

使用说明:
参数一：action的类型
参数二：具体的action，Object类型



