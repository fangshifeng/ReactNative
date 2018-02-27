# 《ReactNative实战讲义》框架篇---02.Flux框架应用
**|** 版权声明：本文为博主原创文章，未经博主允许不得转载。

#### 三、登录功能封装

##### 1. 创建Api.js文件
封装POST请求方法

```
function _doPost(url, formData) {
  return new Promise((resolve, reject) => {
    fetch(url, {
      method: 'POST',
      headers: {

      },
      body: formData
    })
    .then((response)=>{
      if(response.ok){
        return response.json();
      }
    }).then((responseJson)=>{
      resolve(responseJson);
    }).catch((error)=>{
      reject(error);
    });
  });
}
```
方法说明：
_doPost方法返回Promise对象

封装login方法

```
export function doUserLogin(loginData) {
  let loginForm = new FormData();
  loginForm.append("user", loginData.user);
  loginForm.append("pw", loginData.pw);

  dispatchAsync(_doPost('url', loginForm), {
    request: ActionTypes.USER_LOGIN,
    success: ActionTypes.USER_LOGIN_SUCCESS,
    failure: ActionTypes.USER_LOGIN_ERROR
  }, loginData);
}
```

##### 2. AppDispatcher.js文件中增加新方法
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
方法说明：
（1）dispatch(request, action); 分发网络请求类型的action
（2）promise.then(); 分发网络请求返回结果的action

##### 3、创建Login.js文件

componentDidMount()生命周期中接收dispatch分发的action

```
register(action => {
      switch(action.type) {
        case 'USER_LOGIN':
          break;
        case 'USER_LOGIN_SUCCESS':
            if(action.response.status == 1) {
              // {type: ActionConst.RESET} 解决Home时点击返回键，退回到登录页面的问题
              Actions.tabbar({type: ActionConst.RESET});
            } else {
              // TODO 提示失败原因
            }
          break;
        case 'USER_LOGIN_ERROR':
          // TODO 提示失败原因
          break;
      }
    });
```


