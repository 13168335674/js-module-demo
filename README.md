<!--
 * @Author: ADI
 * @Date: 2020-07-16 14:54:12
 * @LastEditors: ADI
 * @LastEditTime: 2020-07-17 17:56:40
-->

# 聊聊 JavaScript Module 与 Vite

# Catalog

- 什么是 JavaScript Module?
- 关于 Vite
  - 它带来的新特性
  - Snowpack(v1)
    - 功能
    - 原理
- 闲谈
  - 还记得[Parcel](https://parceljs.org/)不?

# 什么是 JavaScript Module ?

它是原生的模块化解决方案。

> 在这之前，我们通过 CommonJS/ESModule 导出与引用模块，最后由 Webpack 一类的构建工具将代码合并打包成可供浏览器识别运行的 js。

JS Module 依赖于 import 和 export 完成模块导出和引入。

## JS Module Demo

```index.html
// index.html
<script type="module" src="./js/index.js"></script>

```

```js/index.js
// js/index.js
import sayName from "./modules/sayname.js";
sayName("ADI");
```

```js/modules/sayname.js
// js/modules/sayname.js
const sayName = (name = "") => {
  console.log(`my name is : ${name}`);
};
export default sayName;
```

## 那么它的兼容性如何呢？

> [can i use JavaScript Module ?](https://caniuse.com/#search=JavaScript%20Module)

[![caniuse.png](https://s1.ax1x.com/2020/07/17/Uyao8g.png)](https://imgchr.com/i/Uyao8g)

## 在了解 Vite 之前，我想问问大家，在我们进行项目开发时，我们需要 Webpack 的哪些功能？

- 模块化(Webpack 中一切资源皆可当做模块引入)
- 资源打包与压缩

- 关于模块化：我们现在可以用原生 js module 进行模块化开发；对于 js 以外的资源我们可以通过 Vite 一类的构建工具来处理。
- 关于资源打包与压缩： 由于 HTTP/1.1 的原因，我们需要打包资源、合并请求来优化性能；HTTP/2 普及后，合并文件、内联资源、雪碧图、域名分片对于 HTTP/2 来说是不必要的；对于资源压缩，我们可以在项目发布时配合 Rollup 进行压缩资源。

# Vite

Vite 是一个由原生 ESM 驱动的 Web 开发构建工具。在开发环境下基于浏览器原生 ES imports 开发，在生产环境下基于 Rollup 打包。

它主要具有以下特点：

- 快速的冷启动
- 即时的模块热更新
- 真正的按需编译

Vite 在设计时参考了 Snowpack(v1)的模式，两者具有相似的特征，Snowpack(v1) 的代码很轻量，我们可以通过 Snowpack(v1)来学习一下他们的工作原理。

在开发环境下，Snowpack 不同于 Webpack 的静态分析后打包编译返回资源文件。它使用了 js module 特性，通过拦截浏览器发出的 ES imports 请求，在服务器端处理资源文件后按需返回。

那么两者有具体什么差别呢？
我们可以看看这个例子：

```
// a.js
const a = () => { ... }
export { a }
// b.js
const b = () => { ... }
export { b }

// c.js
import { a } from './a'
import { b } from './b'
const c = () => {
  return a() + b()
}
```

我们同样以 c 为入口文件。
用 Webpack 时，它通过 c.js 进行递归地构建一个依赖关系图，然后将这些模块打包成 bundle。

```
// bundle.js
const a = () => { ... }
const b = () => { ... }
const c = () => {
  return a() + b()
}
```

当某个依赖改变了，由于我们只打了一个 bundle.js,所有会重新打包一个新的 bundle.js。当我们的依赖关系复杂时，就算只修改一个文件，热更新的速度也会越来越慢。

用 Snowpack 时，只编译不用打包，拦截到 ES imports 请求时，只需要按需编译资源返回，如上例，编译后返回三个文件 a.js、b.js、c.js, 只有编译耗时,不用打包也可以加载到需要的代码，省去了打包的时间。
当某个依赖改变了，只需要重新编译改变的依赖，未发生的依赖无需重新编译。当我们的依赖关系复杂时，也不会增加热更新的速度。

> Snowpack 的一大特点是快 —— 全量构建快，增量构建也快。因为不需要打包，所以它不需要像 Webpack 那样构筑一个巨大的依赖图谱，并根据依赖关系进行各种合并、拆分计算。snowpack 的增量构建基本就是改动一个文件就处理这个文件即可，模块之间算是“松散”的耦合。

![Snowpack.png](https://s1.ax1x.com/2020/07/17/UyNlq0.png)

## 那么 Snowpack 是怎么编译的呢？

Snowpack 会开启一个 web 服务器，拦截浏览器发起的资源请求。

```
// Origin:
import sayName from "./modules/sayname.js";
import vue from "vue";
sayName("ADI");
console.log("vue", vue);

// Build Output:
import sayName from "./modules/sayname.js";
import vue2 from "/web_modules/vue.js";
sayName("ADI");
console.log("vue", vue2);
```

web_modules 是 snowpack 对 node_modules 构建的结果。

## 接下来我们来尝尝 Vite

```
$ yarn create vite-app <project-name>
$ cd <project-name>
$ yarn
$ yarn dev
```

```index.html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <link rel="icon" href="/favicon.ico" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Vite App</title>
</head>
<body>
  <div id="app"></div>
  <script type="module" src="/src/main.js"></script>
</body>
</html>
```

```main.js
// Origin:
import { createApp } from 'vue'
import App from './App.vue'
import './index.css'

createApp(App).mount('#app')


// Build Output:
import { createApp } from '/@modules/vue.js'
import App from '/src/App.vue'
import '/src/index.css?import'

createApp(App).mount('#app')
```

可以看到 @modules 是 Vite 对 node_modules 构建的结果。
仔细看一下发现 css 引入路径后面多了点东西(?import)，这是 Vite 除 js 以外的资源文件进行了转译并做了热模块替换，对于这类资源有兴趣可以去翻翻 Vite 看看，基本和 Webpack 的采用处理方式类似。

# 闲谈

## 还记得[Parcel](https://parceljs.org/)不?

还记得 Parcel 出世时，主打“0 配置”，在社区引起了热潮。接着 Webpack4 的到来，加之 Parcel 在生态和社区影响度方面还是跟 Webpack 比不了，随后慢慢淡出了社区话题里。

这次 Vite 一些的构建工具借助浏览器模块的支持，将会对传统全量构建的打包工具进行一次降维打击，相信也是下一代打包工具的发展趋势。

### Webpack: "什么？有新玩具？Webpack 6 将支持,回到我的怀抱吧!"

![run_serve.png](https://s1.ax1x.com/2020/07/17/UsuHhV.png)

[![zhihu.png](https://s1.ax1x.com/2020/07/17/UstNsx.png)](https://imgchr.com/i/UstNsx)

# Reference link

[Vite website](https://github.com/vitejs/vite)
[Snowpack website](https://github.com/pikapkg/Snowpack)
[精读《snowpack》](https://zhuanlan.zhihu.com/p/144993158)
[5 分钟快速理解 HTTP2](https://www.zhihu.com/question/34074946/answer/833938962)
