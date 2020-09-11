## React 16.0 ～ 16.12 新特性

#### React hook

useState + useEffect

好处：

- 避免在 didmount，didupdate，willupdate 生命周期做一些复杂处理，useEffect 的 return 函数，可以返回一个卸载操作
- 避免了 render props 等模式无法公用逻辑代码的问题，同时也避免了组件树嵌套这种场景

注意:

- useEffect 的第二个参数为观察点，观察点不更新则不进行重新更新，若为空，就只动一次再也不动了
- 这两个特性的使用注意要保证每次的执行顺序是固定的，不要在他们的执行过程中引用逻辑判断，或者说逻辑判断应该是在里面，

#### React context

层级过深，组件比较多且复杂的情况下，context && contextType 的使用可以更好地完成数据共享，但是并不利于组件复用

#### getDerivedStateFromProps

用来替代 componentwillreceiveprops, 和原方法不同的是，只要组件重新渲染了，这个方法就会执行（原方法是只要是父组件变化影响了子组件便会重新渲染）,他可以避免在这个生命周期内进行 setState 操作，有一种说法是 React 即将推出的 async render 推出后在虚拟 dom 构建层面可能会中断渲染，setState 值得不到返回可能会出现一个问题. 但是同样也有很多人吐槽 getDerivedStateFromProps 会在每次组件渲染的时候都调用，造成性能浪费，所以最后的结论可能是这俩都不好。。。。这俩都别用。。。。
getDerivedStateFromError 就比较简单了，在 get props 的过程里捕捉并返回错误变量，便于在特殊情况下做 UI 降级处理。

[参考 1](https://www.jianshu.com/p/cafe8162b4a8)
[参考 2](https://hackernoon.com/replacing-componentwillreceiveprops-with-getderivedstatefromprops-c3956f7ce607)
[官网说明](https://zh-hans.reactjs.org/blog/2018/03/27/update-on-async-rendering.html)
[不是功能问题是性能问题](https://www.jianshu.com/p/50fe3fb9f7c3)

#### React.ForwardRef

为函数式组件做 ref 传递

#### Shallow Renderer 浅层渲染

常用于单元测试，官方说法：
你可以把 shallowRenderer 看作用来渲染测试中组件的“容器”，且可以从容器中取到该组件的输出内容。

shallowRenderer.render() 和 ReactDOM.render() 很像，但是它不依赖 DOM 且只渲染一层。这意味着你可以对组件和其子组件的实现进行隔离测试。

#### React.memo() && React.lazy() && suspense

第一个是纯函数，函数式组件，后两个一般用于加载前等待

#### React profiler 性能分析

这个特性主要体现在 React developer tools，一般不会直接在代码里引
