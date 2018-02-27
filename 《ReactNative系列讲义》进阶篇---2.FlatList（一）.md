# 《ReactNative系列讲义》进阶篇---2.FlatList（一）
**|** 版权声明：本文为博主原创文章，未经博主允许不得转载。
#### 一、简介
FlatList，平面列表。应用场景一般是有大量的数据一条一条的罗列出来。这些数据的属性相同，但是内容不一样。那现实中究竟是怎样的一种情景呢，下面我们一起来看看FlatList的基本用法。
#### 二、基础知识
##### 1. FlatList有哪些特性？
* 支持跨平台（同时支持Android和IOS等平台）（一）
* 支持水平布局（一）
* 行组件显示或隐藏时可配置回调事件（三）
* 支持单独的头部文件（二）
* 支持单独的尾部文件（二）
* 支持自定义行间分割线（二）
* 支持下拉刷新（二）
* 支持上拉加载（二）
* 支持跳转到指定行（ScrollToIndex）（三）

##### 2. 支持两种用法？
* 简单用法：继承Component类
* 复杂用法：继承PureComponent类

#### 三、应用
##### 1. 简单用法
* 按照常规调用组件的方式调用

```
import React, { Component } from 'react';
import { FlatList } from 'react-native';

export default class SampleFlatList extends Component {
    render() {
        return(
            <FlatList
                // 页面需要渲染的数据，数组类型
                data={[{name: 'a'}, {name: 'b'}]}
                // 每条数据展示的UI布局
                renderItem={({item}) => <Text>{item.key}</Text>}
                // 横向滚动
                horizontal={true}
            />
        );
    }
}

```
##### 2. 复杂用法
* 类继承PureComponent，进一步优化性能和减少bug产生的可能

```
import React, { PureComponent } from 'react';
import { FlatList, TouchableOpacity, Text, View } from 'react-native';
export default class FlatListBasic extends PureComponent{
    // 数据容器，用来存储数据
    dataContainer = [];
    
    constructor(props) {
        super(props);

        this.state = {
            // 存储数据的状态
            sourceData : []
            ,selected: (new Map(): Map<String, boolean>)
        }
    }
    
    // 当视图全部渲染完毕之后执行该生命周期方法
    componentDidMount() {
        // 创造模拟数据
        for (let i = 0; i < 10; i ++) {
            let obj = {
                id: i
                ,title: i + '只柯基'
            };

            //  将模拟数据存入数据容器中
            this.dataContainer.push(obj);
        }
        
        // 将存储的数据赋予状态并更新页面
        this.setState({
            sourceData: this.dataContainer
        });
    }
    
    /**
     * 此函数用于为给定的item生成一个不重复的Key。
     * Key的作用是使React能够区分同类元素的不同个体，以便在刷新时能够确定其变化的位置，减少重新渲染的开销。
     * 若不指定此函数，则默认抽取item.key作为key值。若item.key也不存在，则使用数组下标
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
    
    // 加载item布局
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
    
    // 空布局
    _renderEmptyView = () => (
        <View><Text>EmptyView</Text></View>
    );
    
    render() {
        return(
            <FlatList
                data={ this.state.sourceData }
                // 实现PureComponent时使用
                extraData={ this.state.selected }
                keyExtractor={ this._keyExtractor }
                renderItem={ this._renderItem }
                ItemSeparatorComponent={({highlighted}) => (<View style={{ height:1, backgroundColor:'#000' }}></View>)}
                ListEmptyComponent={ this._renderEmptyView }
            />
        );
    }
}

// 封装Item组件
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

#### 四、总结
本篇文章实现了FlatList的基本功能---数据展示，下篇文章我们一起来学习一下如何添加上拉加载更多和下拉刷新功能。


