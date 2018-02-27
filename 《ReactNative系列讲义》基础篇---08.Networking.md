# 《ReactNative系列讲义》基础篇---08.Networking
**|** 版权声明：本文为博主原创文章，未经博主允许不得转载。
#### 一、简介
几乎所有的应用都需要和服务器端进行交互，以此从服务器端获取各种数据。RN中网络请求简单明了并且支持多种请求方式。下面让我们一起看看官方标准的网络请求方法（fetch）如何使用。
#### 二、基础知识
* fetch方法支持GET和POST请求方式
* 第一个参数输入URL，第二个参数用来自定义HTTP请求，添加自定义的请求头或者POST请求传递的参数

#### 三、应用
* 请求方式

```
// GET请求
fetch(url, { method: 'GET' })

// POST请求 formData为表单格式传递的参数
fetch(url, {
  method: 'POST',
  body: formData
})
```

* 结果处理
 
```
// 发送请求！
function getMoviesFromApiAsync() {
    let url = 'https://facebook.github.io/react-native/movies.json';
    
    //传递参数使用FormData对象
    let formData = new FormData();
    formData.append("username", 'admin');
    loginForm.append("password", '123456'); 

    return fetch(url, {
        method: 'POST',
        headers: {},
        body: formData    
    })
    .then((response) => response.json())
    .then((responseJson) => {
        return responseJson.movies;
    })
    .catch((error) => {
        console.error(error);
    });
}

// 调用发送请求并处理服务器返回数据的方式
getMoviesFromApiAsync()
.then((responseJson) => {
    // TODO handle data
})
.catch((error) => {
    // TODO handle error
});
```

* GET请求处理方式同上，只需将method属性改成GET并去掉参数即可


