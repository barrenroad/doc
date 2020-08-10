## 1. SingleSpa实战
[代码仓库](https://github.com/barrenroad/single-spa-demo)

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
