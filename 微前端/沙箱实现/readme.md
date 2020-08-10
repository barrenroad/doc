## JS沙箱机制-应用隔离
- 防止全局变量污染
- 应用切换之后需要还原
### 快照沙箱-适用于单应用切换
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
### Proxy 代理沙箱
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
