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

![图1 mindtu](http://ohic55ix9.bkt.clouddn.com/image/blog/%E7%BB%84%E4%BB%B6%E7%BB%93%E6%9E%84.png)

于是迅(mo)速(ji)地做出了一个Demo

![图2 jietu ](http://ohic55ix9.bkt.clouddn.com/image/blog/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202016-12-01%2021.41.03.png)


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

![State变化](http://ohic55ix9.bkt.clouddn.com/image/blog/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202016-12-01%2021.46.17.png)

![UI变化](http://ohic55ix9.bkt.clouddn.com/image/blog/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202016-12-01%2021.46.07.png)


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

![两个横线](http://ohic55ix9.bkt.clouddn.com/image/blog/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202016-12-01%2023.29.18.png)

一切看上去都很正常

![整个state截图](http://ohic55ix9.bkt.clouddn.com/image/blog/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202016-12-01%2023.29.40.png)


发现问题了么？划横线的`state`的`isEnded`仍然为`false`!

但是UI是真真切切更新了

再看子组件`TodoItem`的`state`

![TodoItem state截图](http://ohic55ix9.bkt.clouddn.com/image/blog/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202016-12-01%2023.30.00.png)

子组件的`isEnded`是预期的`true`


尽管如此，当前的功能能通过测试，用起来完全没有问题。但如果继续开发新功能，在可以预见的未来state和UI不同步的问题就会暴露出来。比如，我们要做离线的Todo。最外层的state需要保存，你原本预计一天能做完的feature可能就delay了。


React.js难道没有解决办法么？当然有的，我们让最外层的组件来管理整个state，也就是只维护一个state副本，那么子组件只通过外层组件传入的onChange方法来触发setState。在需求比较简单的情况下，这么做也是行得通的。

也就是按照以下步骤做就行了
1. 对所有子组件数据的操作，最外层都需要相应的onChange方法，方法内是遍历state结构，参数是state内数据的key或者index。
2. 将change方法作为props一层一层传递给子组件。

这行的通的
但如果

1. 有一些新需求，导致state的层次结构变了呢，是不是要改写所有的onChange。如果项目规模大，多人协作呢？
2. 组件之间的嵌套关系变了...

这下难办了，尤其是。这个项目是你接手的，这个嵌套，修改的逻辑岂不是做的要喊救命。

这个问题根本上就是组件的数据流需要管理。就像平时你需要有一个助手帮你处理一些事务，只知道告诉助手，我想要取哪个账户里的钱，再买哪个基金股票，注销某张卡，如果你亲自去管理这些，很可能就出错了。

所以，和流行的方案一样，在稍复杂的项目里，建议引入额外的库来管理数据流，引入redux或者半年后又变成其他，这也都是成本，就看你如何衡量了。





### 最后

我在了解和学习一些新出现的框架和工具的时候喜欢把坑刻意踩一遍，这样用下来能更清楚地知道一个框架或者工具的特点、优点和缺陷。只用React做Demo，做过之后才觉得管理State的管理和异步更新和组件之间的状态同步有什么问题，然后开始配合使用Flux或Redux再尝试一下。

个人认为，踩坑踩得越早期越纯粹。如果在做项目时候踩坑，干扰因素很多，成本就高了。

做到知其然，再到知其所以然。



### 最后的最后
分享一张特别喜感的图片，NBA库里 :laughing:

![库里为王](https://pic1.zhimg.com/712beb8da0a1a65b472255a794972538_b.jpg)


2015到2016赛季的库里的超一流表现，确实像图中一样屌


原图其实出自周星驰的电影《破坏之王》  :laughing:  雅俗共赏吧

![破坏之王](https://pic4.zhimg.com/b785f58d590e9a97bad0fe51e988af8f_b.jpg)
