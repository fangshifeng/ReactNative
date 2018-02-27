# 《ReactNative系列讲义》进阶篇---4.FlatList（三）
**|** 版权声明：本文为博主原创文章，未经博主允许不得转载。
####一、简介
截止到上篇文章，关于FlatList无论是简单的还是高级的属性用法都已经介绍完毕，今天我们一起来看看FlatList更高级的玩法，相关方法的调用。
####二、基础知识
* 获取FlatList的引用
* scrollToEnd：直接跳转到内容的底部，建议设置getItemLayout属性，不然会出现卡顿
* scrollToIndex：跳转到指定索引的行。属性：viewPosition: 指定选定行显示的位置，0代表top，0.5代表middle，1代表bottom；index: 输入的索引值；如果不设置getItemLayout属性，无法到达不可见区域
* 以上两个方法在使用时，需设置getItemLayout属性

####三、应用
* 新增功能

```
// 修改构造方法
constructor(props) {
   super(props);

   this.state = {
       sourceData : []
       ,selected: (new Map(): Map<String, boolean>)
       ,refreshing: false
       // 添加存储用户输入索引的状态
       ,indexText: ''
   }
}
    
// 跳转到指定位置
_doActionToItem = () => {
   this.flatList.scrollToIndex({ viewPosition: 0, index: this.state.indexText });
};

// 跳转到内容最底端
_doActionToBottom = () => {
   this.flatList.scrollToEnd();
};

// Header布局
_renderHeader = () => (
   <View style={{ flexDirection:'row' }}>
       <TextInput
           style={{ height:50, flex:1 }}
           placeholder='请输入行号'
           onChangeText={ (text)=> {this.setState({ indexText: text })} }
       />
       <TouchableOpacity
           onPress={ this._doActionToItem }
           style={{ height:50, width:90, backgroundColor:'green', justifyContent:'center', alignItems:'center' }}
       >
           <Text style={{ color:'#fff' }}>跳转到指定行</Text>
       </TouchableOpacity>
       <TouchableOpacity
           onPress={ this._doActionToBottom }
           style={{ height:50, width:90, backgroundColor:'red', justifyContent:'center', alignItems:'center' }}
       >
           <Text style={{ color:'#fff' }}>跳转到底部</Text>
       </TouchableOpacity>
   </View>
);

// 是一个可选的优化，用于避免动态测量内容；+50是加上Header的高度
getItemLayout={(data, index) => ( { length: 40, offset: (40 + 1) * index + 50, index } )}
```

* 完整代码

```
import React, { PureComponent } from 'react';
import { FlatList, TouchableOpacity, Text, View, TextInput } from 'react-native';
import CustomToastAndroid from '../../../js/ToastAndroid';

export default class FlatListBasic extends PureComponent {
    dataContainer = [];

    constructor(props) {
        super(props);

        this.state = {
            sourceData : []
            ,selected: (new Map(): Map<String, boolean>)
            ,refreshing: false
            ,indexText: ''
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
     * 若不指定此函数，则默认抽取item.key作为key值。若item.key也不存在，则使用数组下标
     *
     * @param item
     * @param index
     * @private
     */
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

        CustomToastAndroid.show(JSON.stringify(id), CustomToastAndroid.SHORT);
    };

    // 跳转到指定位置
    _doActionToItem = () => {
        // viewPosition: 指定选定行显示的位置，0代表top，0.5代表middle，1代表bottom
        this.flatList.scrollToIndex({ viewPosition: 0, index: this.state.indexText });
    };

    // 跳转到内容最底端
    _doActionToBottom = () => {
        this.flatList.scrollToEnd();
    };

    // Header布局
    _renderHeader = () => (
        <View style={{ flexDirection:'row' }}>
            <TextInput
                style={{ height:50, flex:1 }}
                placeholder='请输入行号'
                onChangeText={ (text)=> {this.setState({ indexText: text })} }
            />
            <TouchableOpacity
                onPress={ this._doActionToItem }
                style={{ height:50, width:90, backgroundColor:'green', justifyContent:'center', alignItems:'center' }}
            >
                <Text style={{ color:'#fff' }}>跳转到指定行</Text>
            </TouchableOpacity>
            <TouchableOpacity
                onPress={ this._doActionToBottom }
                style={{ height:50, width:90, backgroundColor:'red', justifyContent:'center', alignItems:'center' }}
            >
                <Text style={{ color:'#fff' }}>跳转到底部</Text>
            </TouchableOpacity>
        </View>
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
        this.setState({refreshing: true})//开始刷新
        //这里模拟请求网络，拿到数据，3s后停止刷新
        setTimeout(() => {
            CustomToastAndroid.show('没有可刷新的内容！', CustomToastAndroid.SHORT);
            this.setState({refreshing: false});
        }, 3000);
    };

    // 上拉加载更多
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
                ref={ ref => this.flatList = ref }
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
                // 是一个可选的优化，用于避免动态测量内容；+50是加上Header的高度
                getItemLayout={(data, index) => ( { length: 40, offset: (40 + 1) * index + 50, index } )}
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


