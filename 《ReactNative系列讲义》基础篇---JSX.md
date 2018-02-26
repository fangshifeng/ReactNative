# 《ReactNative系列讲义》基础篇---JSX
**|** 版权声明：本文为博主原创文章，未经博主允许不得转载。 
#### 一、简介
JSX属于React的基础概念，是JavaScript的延伸部分，正是JSX创造了React的元素。通俗的来讲，JSX类似于Web端的DIV等标签，用来描述UI该是什么样子。但是和HTML还是有一些差别的，因此我们需要重新来学一下这门新的标签语言，如果有熟悉HTML标签语言的朋友可以在学习JSX时类比一下HTML，看看到底有什么不一样。
#### 二、基础概念
JSX有四大特性：
##### 1. 通过大括号{}，JSX中可嵌入任何的JS表达式

示例：

```
const user = {
    firstName: 'lucky'
    ,lastName: 'fang'
}

formatName = (user) => {
    return user.firstName + user.lastName
}

// JSX中嵌入JS表达式
const element = (
    <View>
        <Text>Hello, {this.formatName(user)}</Text>
    </View>
);

```
 
#### 2. JSX编译后可变成JS的对象（也就是说JSX也是表达式，可以嵌入到JS表达式中）
几种情况：可以在if状态判断和for循环中、分配给变量、以参数的形式接收、通过函数返回出去。

示例：

```
// 在状态判断中使用，并将结果返回出去
visitor = (user) => {
    if(user) {
        return (
            <View>
                <Text>Hello, {this.formatName(user)}!</Text>
            </View>     
        );
    }
    
    return (
        <View>
            <Text>Hello, Stranger.</Text>
        </View>
    );
}
```

#### 3. JSX标签中可嵌入特殊属性
例如：
`<View>`标签中可加入style属性，用来定义视图的样式

```
<View style={{ width:100, height:100 }}></View>
```

`<Image/>`标签中可加入src属性，用来引入呈现的图片地址，也可加入style属性，用来定义图片样式

```
<Image source={require('./img/favicon.png')}/>
```

内容可包裹在单引号中，也可放在对象中

```
<Image source={{uri: 'https://facebook.github.io/react/img/logo_og.png'}}/>
```

```
<Image source={{uri: user.avator}}/>
```

#### 4. JSX标签之间可进行嵌套
例如文章上面的Demo中，`<View>`标签中嵌套了`<Text>`标签，一般情况下`<View>`标签用来控制视图的样式，`<Text>`标签用来控制文字的样式

```
<View>
    <Text>Hello, ReactNative</Text>
    <Image source={{uri: 'https://facebook.github.io/react/img/logo_og.png'}}/>
</View>
```

#### 5. JSX防止注入攻击
默认情况下，React在渲染UI时不会嵌入任何的数据。任何数据再被渲染之前都会被转换成字符串。（注：我理解的也很模糊）

#### 三、总结
以上只是JSX语言的基本特性，在今后的学习中，我们还会遇到更多类型的标签，也会讲到JSX标签布局的FlexBox思想。下节课我们一起来学习一下React中的数据类型。

