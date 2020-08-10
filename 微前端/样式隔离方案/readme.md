## CSS隔离方案

### 子应用间的样式隔离
- Dynamic Stylesheet 动态样式表：加载子应用时加载相应的样式表，卸载子应用时同时移除相应的子应用样式，添加新应用的样式
### 主应用与子应用间的样式隔离
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