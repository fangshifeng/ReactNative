# 《ReactNative系列讲义》进阶篇---8.ListView基础功能
**|** 版权声明：本文为博主原创文章，未经博主允许不得转载。

#### 一、内容简述
##### 1、功能描述
ListView系列文章将实现应用中常用的几种功能：加载列表的基础功能，上拉加载更多功能，下拉刷新功能
##### 2、过程描述
（1）实现ListView的基础功能
（2）添加上拉加载更多功能
（3）添加下拉刷新功能
#### 二、代码实现
##### 1、引入RN原生组件 ListView
```
import { ListView, View, Text } from 'react-native';
```
##### 2、声明并初始化
###### 2.1 创建全局变量（类外声明）
```
let ds = new ListView.DataSource({rowHasChanged: (r1,r2) => r1 !== r2});
```
##### 2.2 构造方法中初始化状态机

```
this.state = { demoList: ds.cloneWithRows([])};
```

##### 3、创建视图
###### 3.1 render()方法中创建ListView视图

```
<ListView dataSource={this.state.demoList} renderRow={(rowData) => this._renderRow(rowData)} />
```

注：dataSource属性用于获取数据；renderRow属性用于获取item布局视图并传递参数
###### 3.2 创建item布局

```
_renderRow(rowData) { return(<View style={{ flex:1, height:30 }}><Text>{rowData.title}</Text></View>); }
```

##### 4、创建获取网络数据方法
###### 4.1 接口数据结构样式
```
{
    "status": 1,
    "data": [
        {
            "id": "1",
            "title": "“城堡之乡圣诞节”活动",
            "des": "灯火辉煌！为了庆祝“城堡之乡圣诞节”(« Noël au pays des châteaux »)活动，朗热城堡在整个12月都将点亮灯饰。"
        },
        {
            "id": "2",
            "title": "世界最佳餐厅就在这里",
            "des": "LA LISTE榜单由法国外交部和旅游发展署发起，于2015年8月在巴黎推出，旨在筛选出全球1000家最杰出餐厅，堪称世界美食“排名甄选”。"
        }
    ],
    "msg": "success"
};
```
###### 4.2 获取网络数据

```
fetchData() { 
    let url = ; 
    fetch(url, { method: 'GET', headers: {}, }) 
    .then((response)=>{ 
        if(response.ok){ 
            return response.json(); 
        } 
    }) 
    .then((responseJson)=>{ 
        if(responseJson.status == 1)
    { 
    this.setState({ demoList: ds.cloneWithRows(responseJson.data) }); } }) .catch((error)=>{
    }); 
}
```
###### 4.3 调用获取网络数据方法
```
componentDidMount() { this.fetchData(); }
```


-------
ListView组件的基础功能已全部实现







