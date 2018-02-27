# 《ReactNative系列讲义》进阶篇---03.FlatList（二）
**|** 版权声明：本文为博主原创文章，未经博主允许不得转载。
#### 一、简介
上篇文章中我们了解到了FlatList组件的基本用法，其实FlatList还有很多丰富的功能，可实现更多更灵活的业务需求，接下来让我们一起来看看吧~
#### 二、基础知识
* 支持单独的头部文件

```
// Header布局
_renderHeader = () => (
   <View><Text>Header</Text></View>
);
    
// FlatList属性
ListHeaderComponent={ this._renderHeader }
```

* 支持单独的尾部文件

```
// Footer布局
_renderFooter = () => (
   <View><Text>Footer</Text></View>
);

// FlatList属性
ListFooterComponent={ this._renderFooter }
```

* 支持自定义行间分割线

```
// 自定义分割线
_renderItemSeparatorComponent = ({highlighted}) => (
   <View style={{ height:1, backgroundColor:'#000' }}></View>
);

// FlatList属性
ItemSeparatorComponent={ this._renderItemSeparatorComponent }
```

* 支持下拉刷新

```
// 上拉加载
_onEndReached = () => {
   let newData = [];

   for (let i = 20; i < 30; i ++) {
       let obj = {
           id: i
           ,title: i + '生了只小柯基'
       };

       newData.push(obj);
   }

   this.dataContainer = this.dataContainer.concat(newData);
   this.setState({
       sourceData: this.dataContainer
   });
};

// 决定当距离内容最底部还有多远时触发onEndReached回调；数值范围0~1，例如：0.5表示可见布局的最底端距离content最底端等于可见布局一半高度的时候调用该回调
onEndReachedThreshold={0.1}
// 当列表被滚动到距离内容最底部不足onEndReacchedThreshold设置的距离时调用
onEndReached={ this._onEndReached }
```

* 支持上拉加载

```
this.state = {
    // 初始化状态值
    refreshing: false
}
        
// 下拉刷新
_renderRefresh = () => {
   this.setState({refreshing: true})//开始刷新
   //这里模拟请求网络，拿到数据，3s后停止刷新
   setTimeout(() => {
       CustomToastAndroid.show('没有可刷新的内容！', CustomToastAndroid.SHORT);
       this.setState({refreshing: false});
   }, 3000);
};

// FlatList属性
refreshing={ this.state.refreshing }
onRefresh={ this._renderRefresh }
```


#### 三、应用

```
import React, { PureComponent } from 'react';
import { FlatList, TouchableOpacity, Text, View } from 'react-native';

export default class FlatListBasic extends PureComponent {
    // 数据容器，用来存储数据
    dataContainer = [];

    constructor(props) {
        super(props);

        this.state = {
            sourceData : []
            ,selected: (new Map(): Map<String, boolean>)
            ,refreshing: false
        }
    }

    componentDidMount() {
        // 初始化数据
        for (let i = 0; i < 20; i ++) {
            let obj = {
                id: i
                ,title: i + '只柯基'
            };

            this.dataContainer.push(obj);
        }

        this.setState({
            sourceData: this.dataContainer
        });
    }

    /**
     * 此函数用于为给定的item生成一个不重复的Key。
     * Key的作用是使React能够区分同类元素的不同个体，以便在刷新时能够确定其变化的位置，减少重新渲染的开销。
     * 若不指定此函数，则默认抽取item.key作为key值。
     * 若item.key也不存在，则使用数组下标
     *
     * @param item
     * @param index
     * @private
     */
     // 这里指定使用数组下标作为唯一索引
    _keyExtractor = (item, index) => index;

    /**
     * 使用箭头函数防止不必要的re-render；
     * 如果使用bind方式来绑定onPressItem，每次都会生成一个新的函数，导致props在===比较时返回false，
     * 从而触发自身的一次不必要的重新render，也就是FlatListItem组件每次都会重新渲染。
     *
     * @param id
     * @private
     */
    _onPressItem = (id: string) => {
        this.setState((state) => {
            const selected = new Map(state.selected);
            selected.set(id, !selected.get(id));
            return {selected}
        });
    };

    // Header布局
    _renderHeader = () => (
        <View><Text>Header</Text></View>
    );

    // Footer布局
    _renderFooter = () => (
        <View><Text>Footer</Text></View>
    );

    // 自定义分割线
    _renderItemSeparatorComponent = ({highlighted}) => (
        <View style={{ height:1, backgroundColor:'#000' }}></View>
    );

    // 空布局
    _renderEmptyView = () => (
        <View><Text>EmptyView</Text></View>
    );

    // 下拉刷新
    _renderRefresh = () => {
        this.setState({refreshing: true}) // 开始刷新
        // 这里模拟请求网络，拿到数据，3s后停止刷新
        setTimeout(() => {
            // TODO 提示没有可刷新的内容！
            this.setState({refreshing: false});
        }, 3000);
    };

    // 上拉加载更多
    _onEndReached = () => {
        // 以下是制造新数据
        let newData = [];

        for (let i = 20; i < 30; i ++) {
            let obj = {
                id: i
                ,title: i + '生了只小柯基'
            };

            newData.push(obj);
        }

        // 将新数据添加到数据容器中
        this.dataContainer = this.dataContainer.concat(newData);
        // 将新数据集合赋予数据状态并更新页面
        this.setState({
            sourceData: this.dataContainer
        });
    };

    _renderItem = ({item}) =>{
        return(
            <FlatListItem
                id={item.id}
                onPressItem={ this._onPressItem }
                selected={ !!this.state.selected.get(item.id) }
                title={ item.title }
            />
        );
    };

    render() {
        return(
            <FlatList
                data={ this.state.sourceData }
                extraData={ this.state.selected }
                keyExtractor={ this._keyExtractor }
                renderItem={ this._renderItem }
                // 决定当距离内容最底部还有多远时触发onEndReached回调；数值范围0~1，例如：0.5表示可见布局的最底端距离content最底端等于可见布局一半高度的时候调用该回调
                onEndReachedThreshold={0.1}
                // 当列表被滚动到距离内容最底部不足onEndReacchedThreshold设置的距离时调用
                onEndReached={ this._onEndReached }
                ListHeaderComponent={ this._renderHeader }
                ListFooterComponent={ this._renderFooter }
                ItemSeparatorComponent={ this._renderItemSeparatorComponent }
                ListEmptyComponent={ this._renderEmptyView }
                refreshing={ this.state.refreshing }
                onRefresh={ this._renderRefresh }
                // 是一个可选的优化，用于避免动态测量内容，+1是加上分割线的高度
                getItemLayout={(data, index) => ( { length: 40, offset: (40 + 1) * index, index } )}
            />
        );
    }
}

class FlatListItem extends React.PureComponent {
    _onPress = () => {
        this.props.onPressItem(this.props.id);
    };

    render() {
        return(
            <TouchableOpacity
                {...this.props}
                onPress={this._onPress}
                style={{ height: 40, justifyContent: 'center', alignItems: 'center' }}
            >
                <Text>{this.props.title}</Text>
            </TouchableOpacity>
        );
    }
}
```


