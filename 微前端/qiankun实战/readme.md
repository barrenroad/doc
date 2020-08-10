## qiankun实战
- JS沙箱机制：sandbox
- 基于Single-Spa
- import-html-entry
- 就是将子应用的HTML一起加载到主应用中来，将子应用的script引用js资源注释掉，用fetch把资源请求过来加载
- 预加载
### 主应用编写

### 注册子应用
```
const apps = [{...}]
registerMicroApp(apps);
start();
```
### 子Vue应用
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
### 子React应用
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
### 流程总结
