---
title: React学习总结
tags:
  - 笔记
originContent: ''
categories:
  - React
toc: false
date: 2019-04-07 21:12:40
---


## 资源
看过很多文档，经过精心整理对比，觉得以下几个对我来说最有帮助，在此记录一下，分享给更多的人。

1. [React 基础知识和介绍](https://alibaba.github.io/ice/docs/basis/intro-react)
2. [JavaScript 标准库](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects)
3. [React Getting Started](https://reactjs.org/docs/getting-started.html)

## 常识

props 用来传递数据，state 用来存储组件内部的状态和数据。props 是只读的，state 当前组件 state 的值可以作为 props 传递给下层组件

1. React 组件的变化是基于状态的

     通过setState更改状态的内容
```
this.setState({
      switchStatus: !this.state.switchStatus
    });
```
2. React 组件的生命周期
![](https://img.alicdn.com/tps/TB1Ng7_MpXXXXamXFXXXXXXXXXX-2850-2945.jpg)
更多详见[地址](https://alibaba.github.io/ice/docs/basis/intro-react#React%20%E7%BB%84%E4%BB%B6%E7%9A%84%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F)

## 问题解决

### 1.this 作用域处理方案

问题背景：手动触发事件，更改state中数据

1. 箭头表达式
```
  mergeFilter = ()=> {
    this.setState({
      store: store,
      merge: !this.state.merge,
    });
  } 
  
```
   render组件中调用  
```
<EuiSwitch
        label="Merge"
        key="MergeEuiSwitch"
        checked={
          this.state.merge
        }
        onChange={()=>this.mergeFilter()}
/>  
```
handlePaginationChange也是可以调用的
2. bind
```
  constructor(props) {
    super(props);
    this.state = {value: 'coconut'};

    this.handleChange = this.handleChange.bind(this);
  }

  handleChange(event) {
    this.setState({value: event.target.value});
  }
  
 render() {
    return (
      <form onSubmit={this.handleSubmit}>
         <select value={this.state.value} onChange={this.handleChange}>
            <option value="grapefruit">Grapefruit</option>
          </select>
        <input type="submit" value="Submit" />
      </form>
    );
  }
```

### 2.传值
这里的传值是组件化传值，属性在组件中是可以自定义的。
在父组件中添加子组件后，可以添加任意属性，如下的dialogObj：
```
<DetailDialog dialogObj={dialogObj} } />
```
对于在子组件中可以在构造方法中获取该属性
```
constructor(props) {
      super(props);
      this.state = {
        dialogObj: this.props.dialogObj,
      };
    }
```
对于父组件该属性发生变化，如何更新呢，这里利用生命周期中的componentWillReceiveProps,这样即可获悉到属性的变化。
```
    // 接受父类props改变，修改子类中的属性
    componentWillReceiveProps(nextProps) {
      this.setState({
        dialogObj: nextProps.dialogObj,
      });
    }
```
### 3.父调用子函数
在开发的场景中，经常存在需要在父组件中触发子组件方法

通过属性传值能解决业务问题的，优先建议使用属性传值解决。如果不能解决，可以考虑使用ref解决调用问题

例如：
```
<SimpleFormDialog ref="getDialog">

父组件方法
parentMethod = () => {
            //调用组件进行通信
            this.refs.getDialog.childMethod();
}
```
### 4.子调用父函数
在开发的场景中，经常存在需要在子组件中触发父组件方法

在添加子组件的时候使用如下方式
```
<DetailDialog hideDetailDialog={this.hideDetailDialog} />
```
类似于属性传递，方法也可以传递的，这样在子组件中方法调用一下即可：
```
hideDetailDialog = () => {
      this.props.hideDetailDialog();
    }
```

## 高阶学习
### react-route
略，详见[简明React Router v4教程](https://juejin.im/post/5a7e9ee7f265da4e7832949c)

### react-redux
略,详见[redux](https://www.redux.org.cn/)
## 参考

1. [he-select-tag](https://reactjs.org/docs/forms.html#the-select-tag)
2. [React 基础知识和介绍](https://alibaba.github.io/ice/docs/basis/intro-react)
3. [React Refs](http://www.runoob.com/react/react-refs.html)