# ng 与异构体系

前面介绍了那么多有关于同构和 node 异构的内容，其实市面上大多数的场景是一个前端做好了打包和 cdn 分发之后，由 ng 指向的。

## 微前端解决的问题

这里用 qiankun 作为微前端试水的方案。

### 优势

1、支持多技术栈，多团队，独立开发
2、每次功能只是对一小部分进行改进,升级迭代更轻松，简单说就是粒度比较细
3、有统一的管理中心

### 困局

可能占用的 ng 资源比较多，在生态不好的 case 下反而容易混乱

可能更适用于比较巨大，子应用很多，需要数据共享的场景

### 实现

个人觉得对于微前端的一个良好模式应该是
一个 ng 的主域名，proxy_pass 一个 nodejs 服务端口，nodejs 服务端口打向不同的 path 会指向不同的打包产物，由一个统一的 nodejs 层做用户行为分析，日志，监控，如果需要可将用户行为数据存到数据库里(mysql, mongodb, redis)并进行独立分析

如果这个很困难。。。。。或者说有后端积极要做这件事

那么就专注于前端做这个事情
一个主前端应用管理所有前端应用，所有的数据可以统一保存在顶层(store 也好 model 也好)，这个时候就需要一个相对高端的数据流方案，统一的 sdk 捕捉错误

全局错误要 catch 住，不然会阻塞页面的运行，所以监测全局是否有 e 抛出来就很关键

但是感觉这一点在实际场景上带来的收益还需要认真思考。比如如果我们线上的代码需要迁移，直接配 ng 就好了，为什么要走微前端呢？

### platformBox 实操

但是出于对新技术的探索还是用 qiankun 搞了下

整体分为主应用基座和微应用们两部分

#### 主应用基座要注册

我们可以用 vue-cli 快速起一个应用

然后按照官网在入口文件里面注册微应用并启动就好了

```js
const apps = [
  /**
   * name: 微应用名称 - 具有唯一性
   * entry: 微应用入口 - 通过该地址加载微应用
   * container: 微应用挂载节点 - 微应用加载完成后将挂载在该节点上
   * activeRule: 微应用触发的路由规则 - 触发路由规则后将加载该微应用
   */
  {
    name: 'ReactMicroApp',
    entry: '//localhost:8000',
    container: '#frame',
    activeRule: '/react',
    props: {
      person: 'im baba',
    },
  },
];

export default apps;
```

```js
import { registerMicroApps, start, addGlobalUncaughtErrorHandler } from 'qiankun';

import apps from './app';

registerMicroApps(apps, {
  beforeLoad: (app) => {
    console.log('before load', app.name);
    return Promise.resolve();
  },
  beforeMount: [
    (app) => {
      console.log('before mount', app.name);
      return Promise.resolve();
    },
  ],
});

addGlobalUncaughtErrorHandler((event) => console.log('event', event));

export default start;
```

然后在主应用里启动一下就好了，这些都是参考 qiankun 的官方文档和示例，没啥好说的

```ts
// main.ts
import start from './micro';
start();
```

#### 接入的微应用

众所周知周知微应用是不区分技术栈的，但是因为 qiankun 和 umi 在一个生态里面，所以这里先说 umi 应用的接入

##### umi

对于 umi 微应用可以直接通过@umijs/plugin-qiankun 来实现微应用的接入能力。但是 umi 的文档不全，这个是个坑，要结合 qiankun 的文档一起看。

微应用需要配置三个事情

1、插件注册，按官网来就行。

```js
qiankun: {
    slave: {},
  },
```

2、配置运行时生命周期钩子，对于 umi 应用，可以将 src/app.ts 作为入口文件，按照文档导出就行。

```js
/* eslint-disable no-underscore-dangle */
if (window.__POWERED_BY_QIANKUN__) {
  // eslint-disable-next-line no-undef
  window.__webpack_public_path__ = window.__INJECTED_PUBLIC_PATH_BY_QIANKUN__;
}

export const qiankun = {
  // 应用加载之前
  async bootstrap(props: any) {
    console.log('app1 bootstrap', props);
  },
  // 应用 render 之前触发
  async mount(props: any) {
    console.log('app1 mount', props);
  },
  // 应用卸载之后触发
  async unmount(props: any) {
    console.log('app1 unmount', props);
  },
};
```

3、配置微应用的打包工具

```js
chainWebpack: (config) => {
  config.output
    .library(`${name}-[name]`)
    .libraryTarget('umd')
    .jsonpFunction(`webpackJsonp_${name}`)
    .globalObject('window');
  config.devServer.hot(false);
},
```

#### 非 umi 常规应用

也需要配置三个事情

1、导出相应的生命周期钩子

```js
function render(props: any) {
  const { container } = props;

  ReactDOM.render(
    <App />,
    container ? container.querySelector('#root') : document.querySelector('#root')
  );
}

/**
 * 渲染函数
 * 两种情况：主应用生命周期钩子中运行 / 微应用单独启动时运行
 */

// 独立运行时，直接挂载应用
if (!window.__POWERED_BY_QIANKUN__) {
  render({});
}

/**
 * bootstrap 只会在微应用初始化的时候调用一次，下次微应用重新进入时会直接调用 mount 钩子，不会再重复触发 bootstrap。
 * 通常我们可以在这里做一些全局变量的初始化，比如不会在 unmount 阶段被销毁的应用级别的缓存等。
 */
export async function bootstrap() {
  console.log('ReactMicroApp bootstraped');
}

/**
 * 应用每次进入都会调用 mount 方法，通常我们在这里触发应用的渲染方法
 */
export async function mount(props: any) {
  console.log('ReactMicroApp mount', props);
  render(props);
}

/**
 * 应用每次 切出/卸载 会调用的方法，通常在这里我们会卸载微应用的应用实例
 */
export async function unmount(props: any) {
  console.log('ReactMicroApp unmount');
  const { container } = props;
  ReactDOM.unmountComponentAtNode(
    container ? container.querySelector('#root') : document.querySelector('#root')
  );
}
```

2、配置微应用的打包工具

dev 模式下要设置 access-control 避免跨域，还有关掉一些可能导致报错的热更新。
生产模式下要定义一些全局变量

```js
webpack: config => {
    config.output.library = `${name}-[name]`;
    config.output.libraryTarget = 'umd';
    config.output.jsonpFunction = `webpackJsonp_${name}`;
    config.output.globalObject = 'window';

    return config;
  },

  devServer: _ => {
    const config = _;

    config.headers = {
      'Access-Control-Allow-Origin': '*',
    };
    config.historyApiFallback = true;

    config.hot = false;
    config.watchContentBase = false;
    config.liveReload = false;

    return config;
  },
```

**ps :** 其实好多内容文档的更新并不及时，建议不如直接看 example

### 参考

- [qiankun 官网](https://qiankun.umijs.org/zh)

- [umi 官网](https://umijs.org/zh-CN)

- [体验微前端](https://juejin.im/post/6844904182814605325)

- [基于 qiankun 的微前端最佳实践（万字长文） - 从 0 到 1 篇](https://juejin.im/post/6844904158085021704)

- [微前端(singleSpa + React )试玩](https://juejin.im/post/6844903999574016013)

- [一文带你看懂 UmiJS （3.x 版本）](https://juejin.im/post/6844904197331091464#heading-13)

- [基于 qiankun 落地部署微前端爬"坑"记](https://juejin.im/post/6854573214564089864)

- <https://github.com/umijs/qiankun/issues/937>)

- <https://github.com/umijs/qiankun/issues/868>
