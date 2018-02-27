# 《ReactNative系列讲义》进阶篇---09.ListView + 上拉加载更多
**|** 版权声明：本文为博主原创文章，未经博主允许不得转载。
#### 一、内容简述
ListView列表添加上拉加载更多功能

#### 二、代码实现

##### 1、添加第三方组件 immutability-helper
注：关于immutability-helper库的作用与使用后面的文章会详细说明

`import update from 'immutability-helper';`

##### 2、添加RN原生相关组件
`import { ListView, View, Text, ActivityIndicator } from 'react-native';`
注：ActivityIndicator 动画组件

##### 3、修改构造方法
创建全局变量
`this.demoList = []; // 数据内存存储`
`this.demoListPageIndex = [1]; // 页码内存存储 默认存储第一页`
`this.cachedDemoList = []; // 增量数据内存存储，用于判断是否还有更多数据`

增加状态机：

```
isLoadingTail: false // 默认false
isNoMoreData：false // 用来控制下拉刷新时footer布局
```

构造方法代码：

```
constructor(props) { 
    super(props);

    this.demoList = [];
    
    this.demoListPageInde = [1];
    
    this.cachedDemoList = [];
    
    this.state = {
      demoList:  ds.cloneWithRows([]),
      isLoadingTail: false,
      isNoMoreData：false
    };
}
```

##### 4、修改render()方法

```
render() {
    return (
      <ListView
        enableEmptySections={true}
        dataSource={this.state.demoList}
        renderRow={(rowData) => this._renderRow(rowData)}
        renderFooter={() => this._renderFooter()}
        onEndReached={() => this._endReached()}
        onEndReachedThreshold={20}
      />
    );
  }
```
属性说明：

renderFooter：加载footer布局
onEndReached：到达规定的底部距离时，调用该方法
onEndReachedThreshold：规定到达底部的距离

##### 5、创建加载相关布局方法
###### 5.1 底部footer布局方法
```
_renderFooter = () => {
    // 返回没有更多数据视图 缓存的增量数据为0并且页数不是初始页
    if(this.cachedDemoList.length == 0 && this.state.isNoMoreData) {
      return (<View style={{marginVertical:20}}>
          <Text style={{fontSize:14,color:'#000000',textAlign:'center'}}>没有更多数据啦...</Text>
      </View>);
    }

    // 显示过度布局
    if(!this.state.isLoadingTail) {
      return <View style={{marginVertical:20}}/>
    }

    // 加载动画
    return (<ActivityIndicator style={{ marginVertical:20 }}/>);
  }
```

###### 5.2 自动发起请求获取更多数据方法
```
_endReached = () => {
    // 防止重复申请
    if(this.state.isLoadingTail) {
      return
    }

    // 获取数据
    this.fetchData(true)
  }
```

##### 6、修改fetch方法

```
fetchData() { 
    // 修改加载状态 
    // TODO 此处可优化，可优化成第一次请求服务器时，不修改此状态
    this.setState({ isLoadingTail: true });
    // 取出默认页码
    let page = this.demoListPageInde[0];
    
    let url = '';
    fetch(url, {
      method: 'GET',
      headers: {},
    })
    .then((response)=>{
      if(response.ok){
        return response.json();
      }
    })
    .then((responseJson)=>{
      if(responseJson.status == 1) {
        let responseData = responseJson.data;
    
        // 清空增量数据缓存数组
        this.cachedDemoList = [];
        // 存储新的增量数据
        this.cachedDemoList = this.cachedDemoList.concat(responseData);
    
        // 将新数据追加到旧数据中
        this.demoList = this.demoList.concat(responseData);
        // 页数+1
        this.demoListPageInde[0] += 1;
    
        // 利用 immutability-helper 更新状态机
        this.setState((previousState) => {
          var newState = update(previousState, {demoList:{$set: ds.cloneWithRows(this.demoList)}});
          return newState;
        });
    
        // 关闭加载状态
        this.setState({
          isLoadingTail: false
        });
        
        // 默认每十条为一页，不足十条，则说明没有更多数据
       if(responseData.length < 10) {
         this.setState({
           isNoMoreData: true
         });
       }
      }
    })
    .catch((error)=>{
        // TODO 待完善
        // 关闭加载状态
        this.setState({
          isLoadingTail: false
        });
    });
}
```

