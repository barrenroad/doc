# 微前端专栏

## 什么是微前端
- 适用于大型中后台管理系统：因为随着公司的发展，内部的各种中后台系统会越来越多。
- 如果每个系统都用SPA的形式来开发，那么每个系统都需要单独的域名来承载，多了就记不住，而且个应用间如果有相互依赖跳转的话，将频繁的浏览器重载刷新跳转，体验不好
- 如果维护在同一个代码仓库中，那么仓库会越来越臃肿，维护困难、打包困难、测试困难（改了其中某个小的功能点，需要全量回归测试）、无法独立发布。
- 基于以上的痛点问题，微前端解决方案应运而生。借鉴了微服务的思想，将一个大的应用按功能模块拆分成若干个微小的子应用，子应用单独开发、打包、测试、发布，各子应用间互不影响。最后再将子应用统一组合到一个主应用中。达到统一入口，且各应用独立维护的目的，并且应用切换时可以达到单页应用中的局部刷新体验。
应用场景
工作台的场景：基于产品体验维度
- 多个管理后台聚合成统一的工作台
大型单体应用：侧重于从技术角度优化
- 单体应用过于巨大，不利于开发维护
微前端方案的好处
- 巨型应用拆成小应用后可以交由不同的团队维护不同的功能模块，子应用独立开发、维护、构建、测试、部署
- 统一入口，提升统一体验，一体化配置
- 还可以整合不同技术栈的项目，如Vue、React、Angela等或者是历史老项目
核心思想
- 微前端的核心在于拆，拆完之后再合。
- 将一个大应用划分成一个个子应用，将子应用打包成一个一个的lib资源，在路由切换的时候去匹配各个子应用的名字，匹配上就加载对应的子应用资源
如何落地微前端
各个公司基于不同的内部环境有不同的微前端实现方案，这里主要介绍两种：SingleSpa、qiankun。
- Single-Spa微前端解决方案诞生于2018年，实现了路由劫持和应用加载（本身没有实现样式隔离和JS沙箱）
- qiankun微前端解决方案诞生于2019年，2020年发布了更加完善的2.x版本，它是基于Single-Spa，在Single-Spa的基础上，实现了一套开箱即用的API，做到了：技术栈无关，且接入简单
- 协议接入：子应用必须导出 bootstrap、mount、unmount方法，放主应用去调用

## 1. SingleSpa实战
### 1.1 构建子应用
- 子应用需要导出 bootstrap、mount、unmount三个生命周期放到window上供主应用调用---协议接入

``` 
// main.js
import singleSpaVue from 'single-spa-vue'
const appOtions = {
    el: '#vue', // 将要挂载到主应用中的dom节点
  router,
  render: h => h(App)
}
const vueLifeCycle = singleSpaVue({
    Vue,
  appOptions,
})
export const bootstrap = vueLifeCycle.bootstrap;
export const mount = vueLifeCycle.mount;
export const unmount = vueLifeCycle.unmount;
```

### 1.2 配置库打包

- 增加vue.config.js 配置打包
```
module.exports = {
    configuWebpack: {
    output: {
        library: 'singleVue', // 包名
      libraryTarget: 'umd' // umd 格式可以将singleVue作为全局变量挂载到window上
    },
    devServer: {
        port: 10000
    }
  }
}
``` 
- 打包后的umd资源

### 1.3 主应用搭建
- 主应用配置
```
// main.js
import {registerApplication, start} from 'single-spa';
registerApplication(
  'myVueApp', 
  async () => { // 需要返回 bootstrap、mount、unmount 三个方法
    console.log('加载资源');
    // 真正加载资源 
    async loadScript('http://localhost:10000/js/chunk-vender.js');
    async loadScript('http://localhost:10000/js/app.js');
    return window.singleVue; // singleVue为子应用打包成umd格式之后挂载到window上
  },
  loacation => location.pathName.startsWith('/vue'),
  {}, // customProps
)
```
- 如何加载资源
- 动态创建script标签
- system.js
- webpack配置：external cdn
```
function loadScript(src) {
    return new Promise((resolve, reject) => {
    let scriptDom = document.createElemnt('script');
    scriptDom.src = src;
    scriptDom.onload = resolve;
    scriptDom.onerror = reject;
    document.body.appendChild(scriptDom);
  })
}
```
### 1.4 动态设置子应用publicPath
- 在主应用中切换子应用路由的话默认是相对主应用加载路由资源的，所以我们需要在子应用中设置切换路由资源加载绝对路径资源，可以通过window.singleSpaNavigate属性是否存在来判断应用是不是被以子应用的形式加载
if (window.singleSpaNavigate) {
    __webpack_public_path__ = 'http://localhost:10000/'; // 设置路由资源的绝对路径前缀
}
- 同样也可以通过这种方式判断，如果应用不是以子应用的形式加载的话，说明就是在本地开发，需要本地启动调试
```
// 子应用 main.js
if (!window.singleSpaNavigate) {
    delete appOptions.el;
  new Vue(appOptions).$mount('#app');
}
```

### 1.5 Single-Spa的缺陷
- 没有实现样式隔离
- 没有JS沙箱机制
下面我们看乾坤是如何实现的：

## 2. qiankun实战
- JS沙箱机制：sandbox
- 基于Single-Spa
- import-html-entry
- 就是将子应用的HTML一起加载到主应用中来，将子应用的script引用js资源注释掉，用fetch把资源请求过来加载
- 预加载
### 2.1 主应用编写
### 2.2 注册子应用
```
const apps = [{...}]
registerMicroApp(apps);
start();
```
2.3 子Vue应用
- 同样需要配置webpack打包：vue.config.js
- 导出三个生命周期函数：bootstrap、mount、unmount，必须是promise
- 在mount中调用render方法
- 在unmount中销毁Vue实例
- 通过判断window上是否存在 __POWER_BY_QIANKUN__ 来判断应用是不是被当成子应用加载，如果不是子应用加载方式，那么直接调用render方法，就可以在本地跑起来
- 如果是被子应用方式加载，那么加上publicPath
```
if(window.__POWER_BY_QIANKUN__) {
    __webpack_public_path__ = window.__INJECTED_PUBLIC_PATH_BY_QIANKUN__;
}
```
- 遇到问题：
- 子应用路由使用了懒加载的话，子应用懒加载的路由模块主应用加载不到，不使用懒加载则正常
- 
### 2.4 子React应用
- 安装包 react-app-rewired
- 配置变线打包配置：config-overrides.js ：打包成lib、子应用必须解决跨域问题
- 配置.env 环境变量 修改端口号 PORT、WDS_SOCKET_PORT
- 导出三个生命周期函数：bootstrap、mount、unmount
- 通过window上的__POWER_BY_QIANKUN__判断是否是被以子应用的形式加载
- 如果是子应用 设置 __webpack_public_path__ 属性
- 如果不是子应用，调用render方法，渲染页面
```
if (window.__POWER_BY_QIANKUN__) {
  __webpack_public_path__ = window.__INJECTED_PUBLIC_PATH_BY_QIANKUN__;
}
if (!window.__POWER_BY_QIANKUN__) {
  render();
}
```
### 2.5 流程总结

## 3. CSS隔离方案

### 3.1 子应用间的样式隔离
- Dynamic Stylesheet 动态样式表：加载子应用时加载相应的样式表，卸载子应用时同时移除相应的子应用样式，添加新应用的样式
3.2 主应用与子应用间的样式隔离
- CSS modules
- namespace 、BEM(Block Element Modifier) 约定项目前缀
- css-in-js
- shadow DOM，真正意义上的隔离
- 兼容性有问题
- React中的事件代理有坑
- antd Modal 会把元素挂载到body上，导致不生效
```
<body>
  <div>
    <p>hello world</p>
    <div id="shadow"></div>
  </div>
  <script>
    const shadowDom = shadow.attachShadow({ mode: 'closed' }) // open/closed
    const pEl = document.createElement('p');
    pEl.innerHTML = 'hi i am shadow dom';
    const styleEl = document.createElement('style');
    styleEl.textContnet = 'p{color: red}';
    shadowDom.appendChild(styleEl);
    shadowDOm.appendChild(pEl);
  </script>
</body>
```
效果如下：
## 4. JS沙箱机制-应用隔离
- 防止全局变量污染
- 应用切换之后需要还原
### 4.1 快照沙箱-适用于单应用切换
- 创建一个SnapShotSandBox类
- SnapShotSandBox中有个modifyPropsMap 对象来记录全局对象的变化
- SnapShotSandBox有一个激活的方法，默认激活，
- 激活时会对全局对象拍照，记录全局对象的初始的属性，
- 并将全局对象变化添加到全局对象中
- SnapShotSandBox有一个失效的方法，失效时，
- 记录全局对象的属性变化，
- 并将其还原为拍照前
- 快照沙箱只支持单个应用
```
class SnapShotSandBox {
    constructor(){
    this.proxy = window;
    this.modifyPropsMap = {};
    this.active();
  }
    active(){
        this.windowSnapShot = {}; // 1.拍照
    for(const prop in window) {
        if (window.hasOwnProperty(prop)) {
        this.windowSnapShot[prop] = window[prop];
      }
    }
    Object.keys(this.modifyPropsMap).forEach(p => {
        window[p] = this.modifyPropsMap[p]; // 2.还原变化
    })
  }
  inactive(){
    for(const prop in window) {
        if(window.hasOwnProperty(prop)) {
        if (window[prop] !== this.windowSnapShot[prop]) { 
            this.modifyPropsMap[prop] = window[prop]; // 1.记录变化
          window[prop] = this.windowSnapShot[prop]; // 2.还原属性
        }
      }
    }
  }
}
// test
const sandBox = new SnapShotSandBox();
((window) => {
    window.a = 1;
  window.b = 2;
  console.log(window.a, window.b);
  sandBox.inactive();
  console.log(window.a, window.b);
  sandBox.active();
  console.log(window.a, window.b);
})(sandBox.proxy)
```
### 4.2 Proxy 代理沙箱
- 由于Proxy的代理对象是不同的，可以解决多应用沙箱问题
- 定义一个原始全局 rawWindow,就是window
- 定义一个虚构全局 fakeWindow, 就是一个空对象
- 通过 Proxy去代理 fakeWindow
- 在设置属性值的时候，直接给 fakeWindow设置（因为全局对象我们只会增加自定义属性，而不会去修改/重写window上的属性）
- 取值的时候，先从rawWindow上取，如果没有取到，说明是自定义的属性，所以从fakeWindow中取
```
class ProxySandBox {
    constructor() {
        const rawWindow = window;
    const fakeWindow = {};
    const proxy = new Proxy(fakeWindow, {
        set(target, prop, value){
        target[prop] = value;
        return true;
      },
      get(target, prop){
        return rawWindow[prop] || target[prop];
      }
    });
    this.proxy = proxy;
  }
}
let sandBox1 = new ProxySandBox();
let sandBox2 = new ProxySandBox();
((window) => {
  window.a = 1;
  console.log(window.a)
})(sandBox1.proxy);
((window) => {
  window.a = '2';
  console.log(window.a)
})(sandBox2.proxy);
```
## 5. 实际运用中遇到的问题
前端现存问题
- 需要解决子应用简单配置即可加载，本地开发测试、还有预加载资源
- 需要解决APP级别隔离与无缝切换
- 子应用原有路由跳转不受干扰
后端现存问题
- 如何解决主子应用接口跨域的问题
- 如何解决各应用间登录互通的问题
- 如何解决子应用post请求拦截问题
- 如何支持子应用请求接口Proxy 转发
## 6. 问题
1. 与iframe的区别？iframe的不足？--> 不熟
- 当子应用用iframe加载，用户切换路由时刷新页面的话，状态会丢失
- 父子应用通信问题，使用成本过高
2. 应用间的通信问题
- 如果采用URL参数，那么消息传递能力弱，只能以字符串的形式显示的放在URL上，也不优雅
- 基于 CustomEvent 实现通信，浏览器自定义事件
- 基于 Props 主子应用通信
- 使用全局变量、redux 通信
3. 公共依赖
- webpack 5.x Federated Modules（联合模块）
- cdn - externals
4. 不同应用依赖同一包的不同版本如何处理？
- 比如React，可以使用DLL
5. 前端进行微前端改造后，可能会涉及不同应用的域名合并，导致跨域
- 服务端也需要做出相应的调整
- 跨域线上一般配置Nginx代理解决 --> 待确定
6. 希望听听毛宝分享底层编译方面的开发
- 目前还是处于实现业务需求的层面，需要开阔一下视野
- 希望能有实际的案例分享，让我们切身的感受一下