# 《ReactNative系列讲义》进阶篇---7.自定义机场选择列表 AirportListView
**|** 版权声明：本文为博主原创文章，未经博主允许不得转载。
#### 一、内容简介
对于拥有国外市场或者通讯功能的APP来说，通过列表展示选择自己需要的数据是很常见的功能模块，例如：通讯录，选择国家等等。这类组件的实现原理基本一致，本篇文章带大家实现一个机场的列表选择模块。
目前只实现了最基础的首字母定位查找功能，首字母分组功能，后期会加上模糊查询，热门城市推荐。
##### 技术点总结
* ReactNative 原生ListView组件
* 查询并存储所有选项的首字母，并且去重
* 将城市名称按照首字母分组
* 记录每个分组起始的高度
* 异步读取本地数据

#### 二、代码实现
##### 1、数据源数据结构

数据片段

```
{
  "allAirportList":[
    {
        "airport": "伊尔施",
        "city": "阿尔山",
        "enAirport": "",
        "match": "阿尔山 伊尔施|aershanyiershi|YIE",
        "pinyin": "aershanyiershi",
        "tcode": "YIE",
        "sortLetters": "a"
    },
    {
        "airport": "阿克苏",
        "city": "阿克苏",
        "enAirport": "",
        "match": "阿克苏 阿克苏|akesu|AKU",
        "pinyin": "akesu",
        "tcode": "AKU",
        "sortLetters": "a"
    },
    {
        "airport": "阿勒泰",
        "city": "阿勒泰",
        "enAirport": "",
        "match": "阿勒泰 阿勒泰|aletai|AAT",
        "pinyin": "aletai",
        "tcode": "AAT",
        "sortLetters": "a"
    },
    {
        "airport": "昆莎",
        "city": "阿里",
        "enAirport": "",
        "match": "阿里 昆莎|alikunsha|NGQ",
        "pinyin": "alikunsha",
        "tcode": "NGQ",
        "sortLetters": "a"
    },
  ]
}
```

##### 2、实现ListView基础功能
* 创建AirportListView类
* 初始化ListView，这里用到了ListView的Section功能，详情可查看官网Demo
* 定义状态机

```
import React, {Component,PropTypes}from 'react';
import  {
  StyleSheet,
  View,
  Text,
  Platform,
  TouchableOpacity,
  ListView,
  Dimensions,
} from 'react-native';
export default class AirportListView extends Component {
    constructor(props) {
       super(props);
       this.totalHeight = []; // 存放每个分组的起始高度
        
       var getSectionData = (dataBlob, sectionID) => {
         return sectionID;
       };
       var getRowData = (dataBlob, sectionID, rowID) => {
         return dataBlob[sectionID][rowID];
       };
        
       let ds = new ListView.DataSource({
         getRowData: getRowData,
         getSectionHeaderData: getSectionData,
         rowHasChanged: (row1, row2) => row1 !== row2,
         sectionHeaderHasChanged: (s1, s2) => s1 !== s2,
       });
        
       this.state = {
         citiesData: ds.cloneWithRowsAndSections([])
         ,letters: []
       };
    }
}
```

##### 3、数据处理
* 获取源数据
* 分拣出涉及到的首字母
* 将城市数据按首字母索引分组
* 计算每个分组的高度并存储
* 异步处理数据

###### (1) 获取数据源
数据源由调用该组件的页面传递

```
let data = this.props.dataSource;
```

###### (2) 获取字母索引数组

```
let letterList = this._getSortLetters(data);
```

* for循环的大意：拿到数据列表中每个对象的sortLetter字段，首先小写转化成大写，其次遍历字母索引数组，遍历过程中拿前一步获取到字段进行比较，如果有相等的项，说明此字母已出现过，不必做存储，中断循环，跳入下一次的遍历，如果不存在相等项，则将该字母存入索引数组，最后返回字母索引数组

```
_getSortLetters(dataList) {
    let list = []; // 存放数据数组
    
    for (let i = 0; i < dataList.length; i++) {
      let sortLetters = dataList[i].sortLetters.toUpperCase();

      let exist = false;
      for (let j = 0; j < list.length; j++) {
        if (list[j] === sortLetters) {
            exist = true;
        }
        if (exist) {
            break;
        }
      }
      if (!exist) {
          list.push(sortLetters);
      }
    }

    return list;
  }
```

######（3）将城市数据按首字母索引分组
* dataBlob中存放的是按照首字母分好组的城市数据

```
 let dataBlob = {};
 data.map(cityJson => {
   let key = cityJson.sortLetters.toUpperCase();

   if (dataBlob[key]) {
     let subList = dataBlob[key];
     subList.push(cityJson);
   } else {
     let subList = [];
     subList.push(cityJson);
     dataBlob[key] = subList;
   }
 });
```

###### (4) 计算每个分组的高度并存储
* 通过Object.keys(dataBlog)获取到所有的key值，也就是所有分组的header
* 提炼出每个组所包含的子元素总数
* 计算出每个section的总高度：header高度 + row高度 * row每个组的总个数

```
 let sectionIDs = Object.keys(dataBlob);
 let rowIDs = sectionIDs.map(sectionID => {
   let thisRow = [];
   let count = dataBlob[sectionID].length;
   for (let i = 0; i < count; i++) {
     thisRow.push(i);
   }

   let eachheight = SECTION_HEIGHT + ROW_HEIGHT * thisRow.length;
   this.totalHeight.push(eachheight);

   return thisRow;
 });
``` 

###### (5) 添加异步功能

```
groupingData() {
    return new Promise((resolve, reject) => {
     let data = this.props.dataSource;
     let letterList = this._getSortLetters(data);
     let dataBlob = {};
    
     // 将城市按字母索引分组
     data.map(cityJson => {
       let key = cityJson.sortLetters.toUpperCase();
    
       if (dataBlob[key]) {
         let subList = dataBlob[key];
         subList.push(cityJson);
       } else {
         let subList = [];
         subList.push(cityJson);
         dataBlob[key] = subList;
       }
     });
    
     let sectionIDs = Object.keys(dataBlob);
     let rowIDs = sectionIDs.map(sectionID => {
       let thisRow = [];
       let count = dataBlob[sectionID].length;
       for (let i = 0; i < count; i++) {
         thisRow.push(i);
       }
    
       let eachheight = SECTION_HEIGHT + ROW_HEIGHT * thisRow.length;
       this.totalHeight.push(eachheight);
    
       return thisRow;
     });
    
     let result = {
       dataBlob: dataBlob
       ,sectionIDs: sectionIDs
       ,rowIDs: rowIDs
     }
    
     resolve(result);
    });
  }
```

##### 4、布局显示
* 主布局
* 字母索引布局
* ListView定位方法
* 点击响应方法

主布局

```
render() {
    return (
      <View style={{flexDirection: 'column',backgroundColor: '#F4F4F4'}}>
        <View style={{height: Dimensions.get('window').height,marginBottom: 10}}>
          <ListView
            ref={listView => this._listView = listView}
            contentContainerStyle={{flexDirection: 'row',width: width,backgroundColor: 'white',justifyContent: 'space-around',flexWrap: 'wrap',}}
            dataSource={this.state.citiesData}
            renderRow={this._renderListRow}
            renderSectionHeader={this._renderListSectionHeader}
            enableEmptySections={true}
            initialListSize={1000}
          />
          <View style={styles.letters}>
            {this.renderLetters()}
          </View>
        </View>
      </View>
    )
  }
```

sectionheader布局

```
_renderListSectionHeader(sectionData, sectionID) {
    return (
      <View style={{paddingTop: 5,paddingBottom: 5,height: 30,paddingLeft: 10,width: width,backgroundColor: '#F4F4F4',}}>
        <Text style={{color: '#333333',fontWeight: 'bold'}}>
          {sectionData}
        </Text>
      </View>
    );
  }
```

row布局

* 构造方法中获取this

```
constructor(props) {
    super(props);
    that = this;
}

```

```
_renderListRow(cityJson, rowId) {
    return (
      <TouchableOpacity
        key={'list_item_' + cityJson.tcode}
        style={{height: ROW_HEIGHT,paddingLeft: 10,paddingRight: 10,borderBottomColor: '#F4F4F4',borderBottomWidth: 1,}}
        onPress={()=> {
          that._cityNameClick(cityJson)
        }}>
        <View style={{ paddingTop: 10, paddingBottom: 2 }}>
          <Text style={{color: '#333333',width: width}}>{cityJson.city} ({cityJson.tcode})</Text>
        </View>
      </TouchableOpacity>
    )
  }
```
    
字母索引布局

* 这里用到了第三方数据操作框架underscorejs

```
renderLetters() {
    if(!_.isEmpty(this.state.letters)) {
      return(
        <View>
          {this.state.letters.map((letter, index) => this._renderRightLetters(letter, index))}
        </View>
      )
    }
  }
```

索引定位

```
  // 根据字母索引定位城市列表
  _scrollTo(index, letter) {
    let position = 0;
    for (let i = 0; i < index; i++) {
      position += this.totalHeight[i]
    }
    this._listView.scrollTo({
      y: position
    });
  }
```

点击响应方法callback

* 类中定义回调函数类型


```
static propTypes = {
    onConfirm: PropTypes.func.isRequired,
  };
  
// 选择城市后的callback
_cityNameClick(cityJson) {
    this.props.onConfirm(cityJson);
}
```

样式

```
const styles = StyleSheet.create({
    letters: {
        position: 'absolute',
        height: height,
        top: 0,
        bottom: 0,
        right: 10,
        backgroundColor: 'transparent',
        justifyContent: 'flex-start',
        alignItems: 'flex-start',
    },
    letter: {
        height: height * 3.3 / 100,
        width: width * 3 / 50+10,
        justifyContent: 'center',
        alignItems: 'center',
    }
});
```

* 注：核心思路来源于简书上的一篇文章，在此基础之上进行封装改造
* 文章链接：http://www.jianshu.com/p/cec3bf7ed7b1


