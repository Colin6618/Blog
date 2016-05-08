---
title: 用ECMAScript6的方式写Javascript模块
date:  2014-12-04 16:58:43
tags:
---

## 用ECMAScript6的方式写Javascript模块


### 简述

&emsp;&emsp; JavaScript的发展非常迅速，但不可否认Javascript有不少瑕疵，其中最显著的便是缺少语言自身的模块系系统(Module System)：将应用分割为一系列粒度更小的文件并且相互依赖执行的方式。
&emsp;&emsp; Ruby有require,Python有import，你熟悉的其他语言都能举出例子，甚至连CSS都有@import！Javascript没有类似的关键字，开发者在开发小型web应用时还能任性一下，但是当面对一个大型应用的时候就得借助各种工具了。当然这不是意味着ES6的模块系统只能适用于小规模应用。庆幸的是，没有模块系统的Javascript很快就会成为过去时。下个版本的Javascript，也就是ECMAScript6，会带来一个功能全面的模块和依赖管理方案。虽然浏览器还不能在短时间内支持这个特性，但毕竟有关模块系统的规范标准已经定稿，支持已经是时间问题。现在已经有很多模块加载框架已经服役良久且在浏览器上运行的很好。本篇文章会对有关JS模块语法以及如何在使用做一些介绍。总之，一定比你想象的简单。

### 关于ES6

&emsp;&emsp; ECMAScript是[Ecma International](http://www.ecma-international.org/)公司制定的脚本语言标准。Javascript是该标准的实现，ES6是下个版本的ECMAScript，因此也就是下个版本的Javascript。ES6规范会在2014年底完全确定下来，并目标在2015年6月发布（都定稿了还隔半年发布是几个意思？）。规范是定下来了，等现在流行的浏览器完全支持却遥遥无期，最新的Chrome和Firefox已经对ES6的一些特性提供支持。等标准公布，再等浏览器厂商都提供支持，恐怕要等的心塞，所以，现在就想用的话就必须借助一些工具和框架。

### ES6模块规范

&emsp;&emsp; ES6模块规范是在今年7月完全定稿，所以不用担心本文介绍的语法会变化。本文除了介绍语法还会顺带介绍一些语言新加的API和用法。JS新的模块系统分为两部分。第一部分是声明模块和依赖的语法，第二部分是手动加载模块的API。我们肯定会用到的是第一部分，这也是本文要着重介绍的。
<br/>
#### 模块语法


&emsp;&emsp; 理解模块语法的关键也分为两部分。首先模块间有依赖，也就是你在编写的模块或方法需要正常运行所依赖的。举个例子，你在用jQuery写一个图片轮播的模块，这时jQuery就是这个模块所依赖的模块。你需要import这些依赖到你的模块中。第二个，模块有exports，也就是对外声明的接口。还是举jQuery的例子，$就可以被认为是jQuery的exoprts，你的模块依能使用jQuery提供的$是因为jQuery提供了。
&emsp;&emsp; 另外需要记住的是Javascript的每一个模块就是一个Javascript文件。

<br/>
#### Named exports

模块可以输出各种对象，可以是普通的对象，也可以是方法，使用`export`关键字来声明


```javascript
export function double(x) {
  return x + x;
};
```

也可以赋值之后再输出。如果这么做，你需要用一对大括号把变量包起来

```
var double = function(x) {
  return x + x;
}

export { double };
```

可以像这样引入`double` 方法

```
import { double } from 'mymodule';
double(2); // 4
```
又见大括号，引入模块是要用到大括号的。注意`from ‘mymodule’`会在当前目录下寻找一个文件名为`mymodule.js`的文件，不需要写文件后缀。


大括号可以包含多个变量。

```
var double = function(x) {
  return x + x;
}

var square = function(x) {
  return x * x;
}

export { double, square }
```

一般情况下，`exprots {}`写在文件末尾。从可读性上考虑，写在文件末尾的话可以扫一眼就知道这个模块输出什么。

和你预想的一样，引入多个方法是也是这么写的。

```
import { double, square } from 'mymodule';
double(2); // 4
square(3); // 9
```

有没有发现你不能很容易的把整个模块和所有的方法引进来。
> 这是故意设计成这样的——鼓励使用者仅仅引入必须使用的方法。


#### Default exports
除了命名的输出，ES6同样允许模块有其默认export。我们总有要引入整个库的场景，尤其是在使用像jQuery,Underscore,Backbone或其他规模较大的库的时候默认输出就很有用。一个模块只能定义一个默认输出，就像这样：
```
export default function(x) {
  return x + x;
}
```

引入：
```
import double from 'mymodule';
double(2); // 4
```

这里没有使用大括号包裹引入模块的名字。默认exports是没有命名的，所以你能import他们并以任何你想要的名字命名。
```
import christmas from 'mymodule';
christmas(2); // 4
```

一个模块可以既有默认的export，又有命名的export，只要你愿意，但这种用法并不常见。
> ES6模块规范的设计目标之一是希望是使用者更多地使用默认模块输出。

译者注：这里有[非常细节的讨论](https://esdiscuss.org/topic/moduleimport#content-0)。
这个讨论说了这么多其实就是在说：如果你更偏向于对exports命名，没有问题，不需要改变习惯去迎合ES6的细则。

### Programmatic API
除了以上新语法，还有新的API进入语言特性，所以你能以编程的方式引入模块。虽然很少用到，但仍然有必要举一个例子：根据具体条件的判断才加载模块的情形。这个例子的意思是：如果你的浏览器不支持某个特性，你可以这样做。

```
if(someFeatureNotSupported) {
  System.import('my-polyfill').then(function(myPolyFill) {
    // use the module from here
  });
}
```

`System.import`提供勾子，允许使用者在一个模块生命周期的某个时间点上注册并运行回调。有关勾子的语法还没有最终确定，但是一旦定稿一定会起巨大的作用。新API开启了许多可能性，试想一下，你可以利用勾子写一些代码在引入每个模块前运行一些像JSHint的东西。你不必在命令行上起一个watch任务来监控代码了，这不失为一种保证代码质量的新方法。

### 在浏览器还不支持的情况下怎么使用呢？

这些新东西有非常好的使用前景，但遗憾的是现在还没有一个浏览器完全支持，可能过不了多久情况会发生改变，而现在可以使用以下工具支持。

#### ES6 module transpiler

现有的一个解决方案是使用[ES6 module transpiler](https://github.com/esnext/es6-module-transpiler)， 一个可以让你使用ES6模块语法的写Javascript的编译器，编译器将源代码编译成CommonJS风格的代码，或者AMD风格的代码。[编译器也有插件提供](https://github.com/esnext/es6-module-transpiler#build-tools)，支持各种构建工具如Grunt，Gulp。
使用编译器的优势在于，如果你已经使用了像RequireJS或Browserify的工具，你能立即切换到编译器开始使用ES6的原生语法并且不必担心新代码在浏览其中不运行。如果你本来就没有使用模块加载器，编译器也不会使代码产生变化。所以，ES6模块编译器做的只是把ES6模块代码转变成CommonJS或AMD风格的代码，除此之外不做其他转换，如果在你的工作流程中还没有这一环，使用它绝对是个很棒的选择。

#### SystemJS


另一个方案是使用[SystemJS](https://github.com/systemjs/systemjs)。我认为这是最好的解决方案。SystemJS是一个通用的模块加载器：它能载入ES6模块，AMD模块，CommonJS模块，以及作为全局变量加入的模块。
SystemJS能做到这些，有赖于另外两个库，[ES6 module loader polyfill](https://github.com/ModuleLoader/es6-module-loader/tree/v0.9.4) 和 [Traceur](https://github.com/google/traceur-compiler/)。可以选择通过bower安装[bower-traceur](https://github.com/jmcriffey/bower-traceur)，实在没必要花时间寻找Traceur的可下载版本。ES6 module loader polyfill实现了`System.import`。Traceur是一个ES6-to-ES5的模块加载器，Traceur把ES6代码转换为ES5代码，ES5标准是已经被大部分浏览器所支持的。利用Traceur的优势是即使在浏览器还是各种不支持的环境下你也能使用ES6的各种新特性。缺点是每次修改代码你都必须编译一遍。当然，SystemJS会自动帮你做这些事情。
你要做的仅仅是在web应用内使用`<script>`标签来引入SystemJS。它会自动载入ES6 module loader 和Traceur。在HTML内使用`System.import`来载入模块。

```
<script>
  System.import('./app');
</script>

```

当你载入页面，app.js会被异步载入。在app.js内可以使用ES6模块，SystemJS会检测到那是一个ES6文件，然后自动载入Traceur来编译。使用SystemJS模块化编程的时候，一个比较好的做法是有建立一个主模块（上例中的app.js）作为应用的入口，app.js文件负责载入应用的所有模块，通过只载入一个”初始化"文件来管理你的应用，剩下的就交给这个文件来做了。
SystemJS也提供[工具](https://github.com/systemjs/systemjs#es6-systemregister-compilation)把应用合并到一个文件。

### 总结

ES6模块距离我们至少还有六个月至一年，但那不意味着现在我们用不了。虽然使用起来还有掣肘——需要使用SystemJS或其他方案，这并不意味着不值得我们这么做。在浏览器使用任何其他的模块系统：RequireJS、Browserify还是其他方案也都要引入额外的工具和库来支持。你以为我在说SystemJS和其他工具比起来没啥优势？too naive, 如果哪天ES6模块语法终于被浏览器支持了，你只需要移除SystemJS然后一切都像以前一样工作，所以其实我就是在说无缝升级。
所以如果你打算开始一个新的工程，我非常推荐使用ES6模块。因为这个已经定稿的语法已经是跑不掉的了，迟早会被浏览器支持，现在就使用新语法比以后再换过来代价可小得多。


### 英文原文地址
花了两个上午终于翻译完成和理通词句，也谢谢你读到这里
[http://24ways.org/2014/javascript-modules-the-es6-way/](http://24ways.org/2014/javascript-modules-the-es6-way/)
