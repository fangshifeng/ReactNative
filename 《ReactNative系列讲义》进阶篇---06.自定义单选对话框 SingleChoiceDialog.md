# 《ReactNative系列讲义》进阶篇---6.自定义单选对话框 SingleChoiceDialog
**|** 版权声明：本文为博主原创文章，未经博主允许不得转载。
#### 一、内容简介
在我们日常的开发中经常会遇到以Dialog对话框的形式，进行单项选择的操作，例如：选择性别，选择语言等等。在这篇文章中，我们以翻译工具的选择语言功能为例，编写单项选择插件。
大体思路如下：
以ReactNative官方提供的Modal组件为核心，通过定义插件所需要的属性(props)，封装出SingleChoiceDialog组件类
#### 二、代码实现
##### 1、数据结构展示
* 创建LanguageData.json文件
* 数据结构可根据项目需求变更，代码也将跟着改变

```
{
  "allLanguageData":[
    {
      "language":"简体中文",
      "languageCode":"zh"
    },
    {
      "language":"英语",
      "languageCode":"en"
    },
    {
      "language":"法语",
      "languageCode":"fra"
    },
    {
      "language":"意大利语",
      "languageCode":"it"
    },
    {
      "language":"日语",
      "languageCode":"jp"
    },
    {
      "language":"德语",
      "languageCode":"de"
    },
    {
      "language":"韩语",
      "languageCode":"kor"
    },
    {
      "language":"西班牙语",
      "languageCode":"spa"
    },
    {
      "language":"泰语",
      "languageCode":"th"
    },
    {
      "language":"俄语",
      "languageCode":"ru"
    },
    {
      "language":"葡萄牙语",
      "languageCode":"pt"
    },
    {
      "language":"希腊语",
      "languageCode":"el"
    }
  ]
}

```

##### 2、创建SingleChoiceDialog类

```
import React, {Component,PropTypes} from 'react';
import  {
  View,
  Text,
  TouchableOpacity,
  Modal,
  ScrollView
} from 'react-native';
export default class SingleChoiceDialog extends Component {
    constructor(props){
        super(props);
    }
    
    render(){
        return(<View></View>);
    }
}
```

##### 3、定义插件需要的属性
* 组件标题
* 默认选中项
* 是否显示组件
* 数据源
* 回调函数，选择结果的返回
* 回调函数，关闭Dialog

```
static propTypes = {
    title: PropTypes.string // Dialog标题
    ,isSelected: PropTypes.string // 默认选中项
    ,isVisible: PropTypes.bool // 是否显示Dialog
    ,dataSource: PropTypes.array // 数据源
    ,onConfirm: PropTypes.func.isRequired // 确认回调
    ,onCancel: PropTypes.func.isRequired // 取消回调
  };
```

##### 4、设置组件属性的默认值

```
static defaultProps = {
    title: '请选择' // 默认题目
    ,isSelected: 'en' // 默认语言为中文
    ,isVisible: false // 默认关闭组件
    ,dataSource:[] // 默认数据源为空数组
  }
```

##### 5、创建组件布局
* 关于Modal可参照官网的API编写
* 如果需要Dialog弹出窗覆盖在当前页面上并且被覆盖的页面部分可见，需要设置Modal的transparent属性为true，即透明
* demo代码中，第一层View是用来设置遮罩层颜色，flex和background属性不可缺少
* demo代码中，第二层View开始，全部用来绘制Dialog内容部分
* onRequestClose属性在android平台需要引入，该属性是方法类型，实例中的写法可实现点击back键时，取消Dialog对话框

```
render() {
    return(
      <Modal
        animationType={'slide'}
        transparent={true}
        visible={this.props.isVisible}
        onRequestClose={() => {this.onCancel()}}>

        <View style={{flex:1, justifyContent:'center',alignItems:'center', backgroundColor:'rgba(0, 0, 0, 0.5)'}}>
          <View style={{ backgroundColor:'#fff', borderRadius: 10 ,height:350}}>
            <View style={{justifyContent:'center', alignItems:'center', borderBottomWidth:1, borderBottomColor:'#d5d5d5', height:35}}>
              <Text style={{ fontSize:15,color:'#009AD6'}}>{this.props.title}</Text>
            </View>
            <ScrollView style={{marginTop:10}}>
              {this.renderOptionsList()}
            </ScrollView>

            <TouchableOpacity onPress={()=>{this.onCancel()}}>
              <View style={{marginTop:15,marginRight:10,width:200,marginLeft:10,marginBottom:15,backgroundColor:'#009AD6',height:35,justifyContent:'center',borderRadius:1}}>
                <Text style={{textAlign:'center',color:'white',fontSize:15}}>取消</Text>
              </View>
            </TouchableOpacity>
          </View>
        </View>

      </Modal>
    );
  }
```

##### 6、展示options列表 renderOptionsList()
* 可用LisView组件，也可用数组的map循环
* this.props.dataSource.map循环数据源
* this.props.isSelected.languageCode === option.languageCode可实现当前选项的高亮

```
renderOptionsList(){
    return this.props.dataSource.map(option => {
      return (
        <TouchableOpacity onPress={() => {this.optionSelected(option)}}>
          <Text style={{width:200,textAlign:'center',fontSize:18,marginTop:5,alignSelf:'center',
            color: this.props.isSelected.languageCode === option.languageCode ? '#009ad6': null}}>
            {option.language}
          </Text>
        </TouchableOpacity>
      )
    })
  }
```

##### 7、关闭对话框 onCancel()
* 作用：关闭Dialog对话框
* 点击取消按钮时会调用
* 选择完成后调用

```
onCancel() {
    this.props.onCancel();
}
```

##### 8、返回选择的option数据

```
optionSelected(option) {
    // 通过回调传递数据
    this.props.onConfirm(option);
    // 关闭Dialog对话框
    this.onCancel();
}
```

##### 9、组件的调用
* 引入数据源，
* 数据源文件和场景文件在同一文件夹下（可随意更改）

```
import DATA_LANGUAGE from '../LanguageData.json';
```

```
this.languageArr = DATA_LANGUAGE.allLanguageData;
this.setState({
    isVisible: false
    ,isSelected: this.dataSource[0] // 默认选中第一个，可随意更改
});
```

```
<SingleChoiceDialog
    title='请选择语言'
    isVisible={this.state.isVisible}
    isSelected={this.state.isSelected}
    dataSource={this.languageArr}
    onConfirm={(option) => {this.doSelected(option)}}
    onCancel={() => {this.setState({isVisible: false})}}
/>
```

* 触发Dialog对话框

```
onPress() {
    this.setState({isVisible:true});
}
```

* 处理回调数据

```
doSelected(option) {
    // TODO 处理数据
}
```

