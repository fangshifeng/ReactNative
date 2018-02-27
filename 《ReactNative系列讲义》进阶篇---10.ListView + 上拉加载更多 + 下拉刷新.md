# 《ReactNative系列讲义》进阶篇---10.ListView + 上拉加载更多 + 下拉刷新
**|** 版权声明：本文为博主原创文章，未经博主允许不得转载。
#### 一、内容简介
ListView列表在添加了上拉加载更多功能之后再添加下拉刷新
#### 二、代码实现
##### 1、引入原生组件 RefreshControl
```
import { ListView, View, Text, ActivityIndicator, RefreshControl } from 'react-native';
```
注：RefreshControl 组件是RN原生的下拉刷新组件，可应用与ListView和ScrollView
##### 2、修改构造方法
增加状态机：
isRefreshing：用来控制下拉刷新

```
this.state = {
      demoList:  ds.cloneWithRows([]),
      isLoadingTail: false,
      isRefreshing: false,
      isNoMoreData: false
    };
```

##### 3、修改render方法
 增加下拉刷新组件

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
        refreshControl={
          <RefreshControl
            refreshing={this.state.isRefreshing}
            onRefresh={() => this._onRefresh()}/>
        }
      />
    );
  }
```

##### 4、修改fetchData方法
重点：添加isLoadMore、isFirst标记

isFirst：标记是否是第一次进入该页面，即初始化加载
isLoadMore：标记区分下拉刷新和下拉加载更多

```
fetchData(isFirst, isLoadMore) {
    let page;

    if(isLoadMore) { // 上拉加载更多
      // 取出页码
      page = this.demoListPageInde[0];
      // 修改加载状态
      this.setState({ isLoadingTail: true });
    } else { // 下拉刷新
      // 刷新时页码始终是1
      page = 1;
      // 第一次加载数据时不打开刷新机制
      if(!isFirst) {
        this.setState({
          isRefreshing: true
        });
      }
    }

    let url = 'http://travel.9797168.com/user/tips/getDynamicTips?type=45&p=' + page + '&num=10';
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

        if(responseData.length != 0) {
          if(isLoadMore) { // 上拉加载更多
            // 清空增量数据缓存数组
            this.cachedDemoList = [];
            // 存储新的增量数据
            this.cachedDemoList = this.cachedDemoList.concat(responseData);

            // 将新数据追加到旧数据中
            this.demoList = this.demoList.concat(responseData);
            // 页数+1
            this.demoListPageInde[0] += 1;
            
            // 默认每十条为一页，不足十条，则说明没有更多数据
            if(responseData.length < 10) {
              this.setState({
                isNoMoreData: true
              });
            }
          } else { // 下拉刷新
            if(!isFirst) {
              // 清空数据内存存储数组
              this.demoList = [];

              // 重置页数存储数组
              this.demoListPageInde[0] = 1;
            }

            // 存储数据
            this.demoList = this.demoList.concat(responseData)

            // 自增
            this.demoListPageInde[0] += 1;
          }
          
          // 利用 immutability-helper 更新状态机
          this.setState((previousState) => {
            var newState = update(previousState, {demoList:{
              $set: ds.cloneWithRows(this.demoList)
            }});
            return newState;
          });
        } else {
          if(isLoadMore) {
            // 清空增量数据缓存数组
            this.cachedDemoList = [];

            // TODO 提示没有更多数据
          } else {
            // TODO 第一次加载或者下拉刷新
          }
        }

        // 修改加载状态
        if(isLoadMore) {
          // 关闭加载状态
          this.setState({
            isLoadingTail: false
          });
        } else {
          // TODO 可区分是否是第一次加载
          this.setState({
            isRefreshing: false
          });
        }
      }
    })
    .catch((error)=>{
      // 修改加载状态
      if(isLoadMore) {
        // 关闭加载状态
        this.setState({
          isLoadingTail: false
        });
      } else {
        // TODO 可区分是否是第一次加载
        this.setState({
          isRefreshing: false
        });
      }
    });
  }
```
##### 5、修改_endReached方法

```
_endReached = () => {
    // 防止重复申请
    if(this.state.isLoadingTail) {
      return
    }

    // 获取数据
    this.fetchData(false, true);
  }
```

##### 6、创建_onRefresh方法

```
_onRefresh = () => {
    // 当加载到最后一页数据，再次下拉刷新时，需关闭isNoMoreData状态机
    this.setState({
      isNoMoreData: false
    });

    this.fetchData(false, false);
  }
```
##### 7、修改componentDidMount生命周期中的方法

```
this.fetchData(true, false);
```


