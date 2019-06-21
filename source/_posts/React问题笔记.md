---
title: React问题笔记
date: 2018-10-15 17:46:16
tags: 笔记
categories:
- React
---
## 1.this 作用域处理方案

问题背景：手动触发事件，更改state中数据

1. 箭头表达式
```
  mergeFilter(_this) {
    let store = [];
    if (!_this.state.merge) {
      store = _this.mergetIndexName(this.state.indices, /[^a-z]+$/);
    } else {
      store = _this.reBuildOriginData(this.state.indices);
    }
    _this.setState({
      store: store,
      merge: !_this.state.merge,
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
        onChange={()=>this.mergeFilter(this)}
/>  
```
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
        <label>
          Pick your favorite flavor:
          <select value={this.state.value} onChange={this.handleChange}>
            <option value="grapefruit">Grapefruit</option>
            <option value="lime">Lime</option>
            <option value="coconut">Coconut</option>
            <option value="mango">Mango</option>
          </select>
        </label>
        <input type="submit" value="Submit" />
      </form>
    );
  }
```






## 参考
1.[he-select-tag](https://reactjs.org/docs/forms.html#the-select-tag)

