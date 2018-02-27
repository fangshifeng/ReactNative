# 《ReactNative实战讲义》框架篇---04.Redux基础应用
**|** 版权声明：本文为博主原创文章，未经博主允许不得转载。

Redux框架篇系列文章总目录：

* [《ReactNative实战讲义》Redux框架篇---（一）基础知识](http://blog.csdn.net/fsf_snail/article/details/79082351)
* [《ReactNative实战讲义》Redux框架篇---（二）基础应用](http://blog.csdn.net/fsf_snail/article/details/79156288)
* [《ReactNative实战讲义》Redux框架篇---（三）高级知识](http://blog.csdn.net/fsf_snail/article/details/79156393)
* [《ReactNative实战讲义》Redux框架篇---（四）高级应用](http://blog.csdn.net/fsf_snail/article/details/79162619)

### 一、简介
在上一篇文章中，我们学习了Redux框架的基础概念，对于Redux的核心理念，设计理念，工作流程，基本用法都有了一个基础的认识和理解。那么这篇文章通过一个简单的实例带大家更深入的理解体会一下Redux框架的使用方法和思想。

### 二、代码实现
实例简介：实现一个获取文章列表的简单功能

* 以下代码的实现顺序没有特殊要求

#### 1. 创建Action类型
文件路径及名称：项目根目录/components/redux/actions/types.js

```
export const ARTICLE = {
    ARTICLE_LIST_ALL: 'ARTICLE_LIST_ALL' // 获取文章列表
}
```

#### 2. 创建ActionCreator
文件路径及名称：项目根目录/components/redux/actions/article.js

```
import { ARTICLE } from './types';

let { ARTICLE_LIST_ALL } = ARTICLE;

export const getAllArticles = (response) => {
    return { type: ARTICLE_LIST_ALL, response }
}
```

#### 3. 创建Reducer
文件路径及名称：项目根目录/components/redux/reducers/article.js

```
import { ARTICLE } from '../action/types';

let { ARTICLE_LIST_ALL } = ARTICLE;

// 创建默认State
let initialArticle = {
    items: []
}

// 文章列表reducer
export const articles = (state = initialArticle, action) => {
    switch(action.type){
        case 'ARTICLE_LIST_ALL':
            return {
                ...state,
                items: action.response
            }
            
        return state;
    }
}
```

* 返回新状态的数据结构：原State + 新State;
* 从不直接修改state是Redux的核心理念之一，因此需要创建对象拷贝，而拷贝中会包含新创建或更新过的属性值。可使用Object.assign()函数，建议使用ES7中提案的对象展开运算符，该提案让你可以通过展开运算符（...），以更加简洁的形式将一个对象的可枚举属性拷贝至另一个对象。

#### 4. 合并Reducers
文件路径及名称：项目根目录/components/redux/reducers/index.js

```
import { combineReducers } from 'redux';
import { articles } from './article';

const AppReducer = combineReducers({
    articles: articles
});

export default AppReducer;
```

#### 5. 创建Store

```
import { createStore } from 'redux';
import AppReducer from '../reducers/index';
import * as ArticleActions from '../actions/article';

const store = createStore(AppReducer);

// 打印初始状态
console.log(store.getState())

// 每次 state 更新时，打印日志
// 注意 subscribe() 返回一个函数用来注销监听器
const unsubscribe = store.subscribe(() =>
  console.log(store.getState())
)

// 发起一系列 action
let articlesList = [
    {
        id: 0,
        name: 'Redux---基础知识'
    },
    {
        id: 1,
        name: 'Redux---基础应用'
    },
    {
        id: 2,
        name: 'Redux---高阶知识'
    },
    {
        id: 3,
        name: 'Redux---高阶应用'
    }
];

// 发起action
store.dispatch(ArticleActions.getAllArticles(articlesList))

// 停止监听 state 更新
unsubscribe();
```

* 实战中，articlesList中的数据大多会从服务器端获取，这点我们以后会讲到。

### 三、总结
以上几点就是Redux框架的基础应用，掌握了这几点，基本上算是把握住了Redux框架的核心思想和用法。其中Action和Reducer基本符合实战中的标注答案，而实战中的Store和dispatch方法的调用会有很大的不同。本文中所讲的只是基础概念，并不适合实战。至于实战中我们如何改造Store，在何时以何种方式调用dispatch我们后面文章中会陆续讲到。




