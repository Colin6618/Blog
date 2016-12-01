---
title: 为什么不要直接管理state
tags: React reactjs redux state
---

Web前端的开发框架层出不穷，流行趋势变化得比时尚圈还快。以至于我们出去旅行一次回来发现社区又多了一个流行框架。
从另一方面来说，这也反映了Web前端作为一个历史不那么悠久的编程领域，其活力和发展速度不是其他社区可以比拟的。

现在流行用做一个Todo来熟悉某个前端框架，14年Angular大行其道，那么用Angular做一个TodoList的Demo学习一下吧。15年开始React迅速发酵，那就用React来做一个Todo吧。以至于国外有个[TodoMVC网站](http://todomvc.com/)专门聚合这些Demo和框架选型。 :laughing: :laughing:

======

现在仅仅使用一个React.js来做一个TodoList，最小化的Demo

### TodoList的React Component 关系

[图1 mindtu]()

我很迅(mo)速(ji)地做出了一个Demo

[图2 jietu ]()


#### 新增
Main.js作为最外层的组件会render整个List, 同时新增Todo操作也在Main中

``` js

//Main.js
class AppComponent extends React.Component {

  constructor(props) {
    super(props);
    this.state = {itemData: []};
    this.state.itemData.push({text: 'todo example 1', isEnded: false, isArchived: false});
    this.state.itemData.push({text: 'todo example 2', isEnded: false, isArchived: false});
  }

  render() {

    return (
      <div className="index">
        <img src={yeomanImage} alt="Yeoman Generator"/>
        <div className="notice">Please edit the input to add todos</div>
        <input className="input" type="text" placeholder="Add things" onKeyPress={(e)=>{
            if(e.key === 'Enter') {
              this.state.itemData.push({text: e.target.value, isEnded: false, isArchived: false})
              this.setState({itemData: this.state.itemData});
              e.target.value = '';
            }
          }} />
        <TodoItems className="itemsWrapper"  data={this.state.itemData} />
      </div>
    );
  }
}

```

新增后，React的调试工具可以看到State的状态变化

[State变化]()

[UI变化]()


现在看来没啥问题，毕竟只有一个组件在管理state

在单个TodoItems.jsx组件内，我把每个state作为pros传给组件TodoItem

```javascript
//TodoItems.jsx
class TodoItems extends React.Component {

  constructor(props) {
    super(props);
    this.state = this.props;
  }

  render() {
    return (
      <div className="TodoItems">
        <ol>
          {this.state.data.map((item, index) => {
            return (
              <TodoItem todo={item} key={index} />
            )
            }
          )}
        </ol>
      </div>
    )
  }

}

```


### 删除（标记为完成）一项操作

TodoItem组件代码，前边有个checkbox

```javascript

//TodoItem.jsx
export default class TodoItem extends React.Component {

  constructor(props) {
    super(props);
    this.state = this.props.todo;
  }

  // 勾选完成
  handleClick(e) {
    var _st = JSON.parse(JSON.stringify(this.state));
    _st.isEnded = e.target.checked;
    this.setState(_st);
  }

  render() {
    return (
      <li>
        //勾选完成
        <input type='checkbox' onClick={this.handleClick.bind(this)} />
        <span style={{
          textDecoration: this.state.isEnded ? 'line-through':'inherit'
        }}>{this.state.text}</span></li>
    )
  }
}

```
TodoItem里有个input checkbox, 勾选表示完成了这个Todo, 绑定onClick, onClick里更新相应state。

UI确实更新了。

![两个横线]()

一切看上去都很正常

![整个state截图]()


仔细看，划横线的`state`的`isEnded`仍然为`false`!

但是UI是真真切切更新了

再看子组件`TodoItem`的`state`

![TodoItem state截图]()

`isEnded`仍然为`true`!

当前的功能看上去没有问题，但如果继续开发新功能，在可以预见的未来，state和UI不同步的问题就会暴露出来。比如，我们要做离线的Todo。最外层的state需要保存，问题就会暴露。











我在了解和学习新的框架和工具的时候必须一个个单独用过来，这样用下来能更清楚地知道一个框架或者工具的特点、优点和缺陷。比如用React，单独用一个React做一个项目，做过之后才觉得管理State的异步更新和组件之间的状态同步有什么问题，然后开始配合使用Flux或Redux。

做到知其然，再到知其所以然。



### 另外
分享一张特别喜感的图片，熟悉NBA的朋友应该知道，2015-2016赛季的库里，可以有这个底气。

![库里为王](https://pic1.zhimg.com/712beb8da0a1a65b472255a794972538_b.jpg)


没错，出自周星驰的《破坏之王》  :laughing: 电影截图

![破坏之王](https://pic4.zhimg.com/b785f58d590e9a97bad0fe51e988af8f_b.jpg)
