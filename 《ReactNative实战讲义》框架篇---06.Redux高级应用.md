# 《ReactNative实战讲义》Redux框架篇---06.高级应用
**|** 版权声明：本文为博主原创文章，未经博主允许不得转载。

Redux框架篇系列文章总目录：

* [《ReactNative实战讲义》Redux框架篇---（一）基础知识](http://blog.csdn.net/fsf_snail/article/details/79082351)
* [《ReactNative实战讲义》Redux框架篇---（二）基础应用](http://blog.csdn.net/fsf_snail/article/details/79156288)
* [《ReactNative实战讲义》Redux框架篇---（三）高级知识](http://blog.csdn.net/fsf_snail/article/details/79156393)
* [《ReactNative实战讲义》Redux框架篇---（四）高级应用](http://blog.csdn.net/fsf_snail/article/details/79162619)

### 一、简介
上篇文章中我们对于redux的高级知识进行了讲解，对部分代码的改造进行了展示。这篇文章我们通过实战中一个完整的功能模块来讲解一下redux在实际项目中的应用标准和具体的方式。下面就让我们通过实战中最常见的获取文章列表功能来一起看看，在这个简单而又实际的功能模块中redux是如何一步一步的实现这个简单的功能的。
### 二、思路
#### 1. 定义ActionType
文件路径及名称：项目根目录/components/redux/actions/types.js

首先我们需要分解用户动作节点，分析出一个完整的流程中可以分解出几个关键动作节点。API请求过程可以分解为：通知UI准备请求服务器；服务器数据请求成功；服务器请求失败或超时。

```
// 网络请求
export const FetchPosts = {
    FETCH_POSTS_REQUEST : 'FETCH_POSTS_REQUEST', // 开始网络请求
    FETCH_POSTS_FAILURE : 'FETCH_POSTS_FAILURE'  // 网络请求失败
};

// 文章
export const ARTICLE = {
    RECEIVE_POSTS : 'RECEIVE_POSTS', // 获取服务器数据
};
```

#### 2. 定义ActionCreator
* 公共ActionCreator（可复用，和业务逻辑无关）

文件路径及名称：项目根目录/components/redux/actions/common.js

```
import { FetchPosts } from './types';

let { FETCH_POSTS_REQUEST, FETCH_POSTS_FAILURE } = FetchPosts;

/**
 * 发起网络请求
 *
 * @returns {{type}}
 */
export const fetchPostsRequest = () => {
    return { type: FETCH_POSTS_REQUEST }
};

/**
 * 网络请求失败
 *
 * @returns {{type}}
 */
export const fetchPostsFailure = (errorMessage) => {
    return { type: FETCH_POSTS_FAILURE, errorMessage }
};
```

* 业务ActionCreator

文件路径及名称：项目根目录/components/redux/actions/article.js

```
import { ARTICLE } from './types';

let { RECEIVE_POSTS } = ARTICLE;

/**
 * 获取服务器数据
 *
 * @param response  服务器反馈
 * @returns {{type, response: *}}
 */
export const receivePosts = (response) => {
    return { type: RECEIVE_POSTS, response }
};
```

#### 3. 定义Reducer
* 公共Reducer

文件路径及名称：项目根目录/components/redux/reducers/common.js

```
import { FetchPosts } from '../actions/types';

let { FETCH_POSTS_REQUEST, FETCH_POSTS_FAILURE } = FetchPosts;

// 初始化状态
let initialState = {
    isFetching: false
};

export const fetchPosts = (state = initialState, action) => {
    switch (action.type) {
        case 'FETCH_POSTS_REQUEST':
            return {
                ...state,
                isFetching: true
            };

        case 'FETCH_POSTS_FAILURE':
            return {
                ...state,
                isFetching: false
            };

        default:
            return state;
    }
};
```

* 业务Reducer

文件路径及名称：项目根目录/components/redux/reducers/article.js

```
import { ARTICLE } from '../actions/types';

let { RECEIVE_POSTS } = ARTICLE;

// 初始化状态
let initialArticles = {
    isFetching: false,
    total_count: 0,
    incomplete_results: false,
    items: []
};

/**
 * 文章状态相关处理
 *
 * @param state
 * @param action
 * @returns {*}
 */
export const articles = (state = initialArticles, action) => {
    switch (action.type) {
        case RECEIVE_POSTS:
            return {
                ...state,
                isFetching: false,
                // action后面的数据结构根据你的业务数据结构自定义
                total_count: action.response.total_count,
                incomplete_results: action.response.incomplete_results,
                items: action.response.items
            };

        default:
            return state;
    }
};
```

#### 4. 合并Reducer
文件路径及名称：项目根目录/components/redux/reducers/index.js

```
import { combineReducers } from 'redux';
import { articles } from './article';
import { fetchPosts } from './common';

const AppReducer = combineReducers({
    articles: articles,
    fetchPosts: fetchPosts
});

export default AppReducer;
```

#### 5. 定义API请求方法
文件路径及名称：项目根目录/components/api/SnailApi.js

```
import * as CommonActions from '../redux/actions/common';
import * as ArticleActions from '../redux/actions/article';

export const getArticlesList = () => (dispatch) => {
    // 通知UI准备请求服务器
    dispatch(CommonActions.fetchPostsRequest());
    
    return fetch(url, {
            method: 'GET',
            headers: {}
        }).then((response)=>{
            if (response.ok) {
                return response.json();
            } 
        }).then((responseJson)=>{
            // 服务器请求成功
            dispatch(ArticleActions.receivePosts(responseJson))
        }).catch((error)=>{
            // 服务器请求失败
            dispatch(CommonActions.fetchPostsFailure(error));
        });
};
```

#### 6. 改造CreateStore
文件路径及名称：项目根目录/components/redux/store/configureStore.js

```
import { createStore, applyMiddleware } from 'redux';
import thunk from 'redux-thunk';
import AppReducer from '../reducers/index';
import * as SnailApi from '../../api/SnailApi';

const store = createStore(
    AppReducer,
    applyMiddleware(
        thunk, // 允许我们 dispatch() 函数
    )
);

// 打印初始化状态
console.log(store.getState());
// 打印状态变化
const unsubscribe =  store.subscribe(() => console.log(store.getState()));

// 一系列的Action
store.dispatch(SnailApi.getArticlesList());

// 注销state变化监听
unsubscribe();
```

### 三、总结
以上就是redux在实战中就简单而又最常见的应用了，这套标准模式可以随意迁移，换个名称就可以改成其他的功能。不过这还不是最完善的demo，毕竟我们没有加入React UI部分。如果加入UI部分，在获取数据，展示数据，数据变动修改数据几个点上还有一些知识需要学习，我们后面的文章会陆续介绍。

