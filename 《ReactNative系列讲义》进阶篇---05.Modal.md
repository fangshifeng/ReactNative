# 《ReactNative系列讲义》进阶篇---05.Modal
**|** 版权声明：本文为博主原创文章，未经博主允许不得转载。
#### 一、简介
项目开发中，总会遇到弹出窗，弹出窗的功能可能会是确认、单项选择、多选、提示等等，一般情况下我们想要的效果通常是在当前页面触发事件，弹出遮罩层，并不会完全覆盖掉当前页面，当前页面处于部分可见，而布局的焦点处于遮罩层上，一般可见布局位于中央部分，本篇博客带大家制作一个单选框的Modal

#### 二、思路整理
* 定义需要的属性，定义属性类型和默认值
* 绘制界面
* 调用组件

#### 三、具体实现
创建ListViewModal.js文件，定义ListViewModal类
##### 1. 添加组件
* name: prop-types
* homepage: https://www.npmjs.com/package/prop-types
* install:

```
npm install --save prop-types
```
* 组件说明: 该组件用来检测传递给组件的属性是否与定义的类型一致

##### 2. 定义属性
引入prop-types

```
import PropTypes from 'prop-types';
```

* 定义属性类型
* note: class外部定义（即不可包含在ListViewModal类内部）
* note: 需展示的数据以数组的形式传递

```
ListViewModal.propTypes = {
    title: PropTypes.string // 标题
    ,modalVisible: PropTypes.bool // 显示开关
    ,dataSource: PropTypes.array // 文本数据
    ,onConfirm: PropTypes.func.isRequired // 确认回调
    ,onCancel: PropTypes.func.isRequired // 取消回调
}
```

* 定义属性默认值

```
ListViewModal.defaultProps = {
    title: '请选择'
    ,dataSource: []
    ,modalVisible: false
}
```

##### 3. 绘制界面

```
<Modal
 animationType={'slide'}
 transparent={true}
 visible={this.props.modalVisible}
 onRequestClose={() => {this.onCancel()}}>
     <View style={{flex:1, justifyContent:'center',alignItems:'center', backgroundColor:'rgba(0, 0, 0, 0.5)'}}>
         <View style={{ backgroundColor:'#fff', borderRadius: 10 ,height:350}}>
             <View style={{justifyContent:'center', alignItems:'center', borderBottomWidth:1, borderBottomColor:'#d5d5d5', height:35}}>
                 <Text style={{ fontSize:15,color:'#009AD6'}}>{this.props.title}</Text>
             </View>
             <ScrollView style={{marginTop:10}}>
                 {
                     this.props.dataSource.map((rowData) => {
                         return(
                             <TouchableOpacity
                                 onPress={() => {this.optionSelected(rowData)}}>
                                 <View style={{ justifyContent: 'center', alignItems: 'center', height: 30, borderBottomColor: '#efefef', borderBottomWidth: 1 }}>
                                     <Text style={{ fontSize: 14 }}>{rowData.title}</Text>
                                 </View>
                             </TouchableOpacity>
                         )
                     })
                 }
             </ScrollView>
             <TouchableOpacity onPress={()=>{this.onCancel()}}>
                 <View style={{marginTop:15,marginRight:10,width:200,marginLeft:10,marginBottom:15,backgroundColor:'#009AD6',height:35,justifyContent:'center',borderRadius:1}}>
                     <Text style={{textAlign:'center',color:'white',fontSize:15}}>取消</Text>
                 </View>
             </TouchableOpacity>
         </View>
     </View>
</Modal>
```

* animationType: 动画类型
* transparent: 遮罩是否透明
* visible: 遮罩显示开关，由外部传递的参数控制
* onRequestClose: 取消请求方法，用户点击物理返回键是调用
* Modal标签中间部分为展示布局代码


##### 4. 两个回调方法的实现

```
onCancel() {
   this.props.onCancel()
}
```
* note: 外部调用时，实现onCancel()回调方法

```
optionSelected(option) {
   this.props.onConfirm(option)
   this.onCancel()
}
```
* note: 外部调用时，实现onConfirm()回调方法

##### 5. 外部调用
* 引入ListViewModal组件

```
import ListViewModal from '../../common/page/ListViewModal'
```

* 组件的使用

```
// 构造方法中：定义模拟数据
this.platFormData = [
            {
                "id": 0
                ,"title": "aaa"
            }
            ,{
                "id": 1
                ,"title": "bbb"
            }
            ,{
                "id": 2
                ,"title": "ccc"
            }
            ,{
                "id": 3
                ,"title": "ddd"
            },{
                "id": 4
                ,"title": "eee"
            }
            ,{
                "id": 5
                ,"title": "fff"
            }
        ]
        
// 构造方法中：组件初始状态
this.state = {
    modalVisible: false
}

// render方法中：布局调用组件
<ListViewModal
     title={'选择平台'}
     dataSource={this.platFormData}
     onCancel={() => { this.setState({ modalVisible: false })}}
     onConfirm={(option) => {
         console.log(JSON.stringify(option))
     }}
     modalVisible={this.state.modalVisible }
 />
 
 // render方法中：组件触发事件
 <TouchableOpacity
     onPress={() => { this.setState({ modalVisible: true })}}>
</TouchableOpacity>
```



