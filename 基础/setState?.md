# 个人对setState的思考

本篇主要是对《深入React技术栈》那部分对于setState的拓展吧。。

先就原代码再进一步拓展,刚学React时写的，博客迁移博客迁移

```js
import React from 'react';
import ReactDOM from 'react-dom';

class Example extends React.Component {
  constructor() {
    super()
    this.state = {
      value: 0
    }
  }

  componentDidMount() {
    this.setState({ value: this.state.value + 1 })
    console.log(this.state.value) // 第一次输出
    this.setState({ value: this.state.value + 1 })
    console.log(this.state.value) // 第二次输出
    setTimeout(() => {
      this.setState({ value: this.state.value + 1 })
      console.log(this.state.value) // 第三次输出
      this.setState({ value: this.state.value + 1 })
      console.log(this.state.value) // 第四次输出
    }, 0);
    this.refs.button.addEventListener('click', this.click)
  }

  click = () => {
    this.setState({ value: this.state.value + 1 })
    this.setState({ value: this.state.value + 1 })
    console.log(this.state.value);
  }

  render() {
    return (
      <div><span>value: {this.state.value}index: </span>
        <button ref="button" onClick={this.click}>点击</button>
      </div>
    )
  }
}
ReactDOM.render(
  <Example />,
  document.getElementById('root')
)
//输出是：0 0 2 3 点击之后输出5 5 最后显示在页面上的value为6
```

这是一个比较复杂的情况我觉得，分为onclick、setTimeout事件外，和事件内两种情况。

>第一类：直接在componentDidMount里执行的两次setState

两次操作在同一次调用栈中执行，React组件渲染到DOM的过程就在一个大的事务(wrapper)里,所以在这个生命周期里调用setState时batchingStrategy的isBatchingUpdates已经被设为true了，两次操作因此没有立即生效，而是被放进了dirtyComponents里，新的state还没被应用到组件中，等回头在队列里执行后，这个过程是异步过程。

>第二类：setTimeout事件内的两次setState

这一次不在React的“大事务”之内，没有前置的batchedUpdate调用，所以batchingStrategy的isBatchingUpdates为false，因此这样的话setState会立即生效，即类似同步过程，两行setState的this.state.value是一样的相当于是重复操作。因此如果我们把setTimeout里面的第三次输出和第四次输出之间的+1变成+2就会输出0 0 2 4；

>第三类：两个附加click事件的setState

这个输出非常之谜，0 0 2 3 点击按钮之后的输出是5 5 ，react官方对此的看法是setState本身都不一定是固定的同步异步这个和实际场景有关，也就是说，如果我们在框架的设计理念内去使用它，得到的就是可控的结果，如果非要游走在框架之外，答案就可能扑朔迷离。。。

原文大佬的解释：

关于setState ,我们首先要有个正确的认识，官网中给出的解释是setState不是保证同步的

这说明它有时候是同步的有时候的异步的，那什么时候是同步，什么时候是异步？

答案是在React库控制之内时，它就会以异步的方式来执行，否则以同步的方式执行。

但大部份的使用情况下，我们都是使用了React库中的组件，例如View、Text、Image等等，

它们都是React库中人造的组件与事件，是处于React库的控制之下，在这个情况下，

setState就会以异步的方式执行。一般理解为this.state的值会在DOM中渲染，其他的情况比如取token,作为接口的参数的你setState是同步的

ps:如果第三类情况是可以讲的通的，请路过大佬批评教育orz