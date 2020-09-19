# 如何开发一个Babel插件

> 本文部分引自：
> - [babel 插件开发案例](https://juejin.im/post/6844903769805701128#heading-0)
> - [babel手册](https://github.com/jamiebuilds/babel-handbook/blob/master/translations/zh-Hans/user-handbook.md)
> - [babel插件手册](https://github.com/jamiebuilds/babel-handbook/blob/master/translations/zh-Hans/plugin-handbook.md)
> - [Babel插件开发入门指南](https://juejin.im/post/6844903616080281614#heading-0)

## 一、Babel是什么
根据`babel手册`介绍：
- Babel 是一个通用的多用途 `JavaScript` 编译器。Babel 把用最新标准编写的 JavaScript 代码向下编译成可以在今天随处可用的版本。 这一过程叫做“源码到源码”编译， 也被称为转换编译（transpiling，是一个自造合成词，即转换＋编译。以下也简称为转译）。
- Babel 的一切都是简单的插件，谁都可以创建自己的插件，利用 Babel 的全部威力去做任何事情。
- Babel 自身被分解成了数个核心模块，任何人都可以利用它们来创建下一代的 `JavaScript` 工具。
- 它还拥有众多模块可用于不同形式的静态分析。


再介绍一下几个babel核心知识点：
### 1.1 babel-cli
Babel 的 CLI 是一种在命令行下使用 Babel 编译文件的简单方法。
让我们先全局安装它来学习基础知识。
```
$ npm install --global babel-cli 
```
我们可以这样来编译我们的第一个文件：
```
$ babel my-file.js 
```
这将把编译后的结果直接输出至终端。使用 --out-file 或着 -o 可以将结果写入到指定的文件。
```
$ babel example.js --out-file compiled.js 
```
或
```
$ babel example.js -o compiled.js 
```
如果我们想要把一个目录整个编译成一个新的目录，可以使用 --out-dir 或者 -d。.
```
$ babel src --out-dir lib 
```
或
```
$ babel src -d lib 
```
注意：尽管你可以把 Babel CLI 全局安装在你的机器上，但是按项目逐个安装在本地会更好。
有两个主要的原因。
1. 在同一台机器上的不同项目或许会依赖不同版本的 Babel 并允许你有选择的更新。
2. 这意味着你对工作环境没有隐式依赖，这让你的项目有很好的可移植性并且易于安装。

### 1.2 babel-core

如果你需要以编程的方式来使用 `Babel`，可以使用 `babel-core`这个包。
首先安装 `babel-core`。
```
$ npm install babel-core 
```
```
var babel = require("babel-core") 
// 字符串形式的 JavaScript 代码可以直接使用 babel.transform 来编译。
babel.transform("code();", options);
// => { code, map, ast }
// 如果是文件的话，可以使用异步 api：
babel.transformFile("filename.js", options, function(err, result) {
  result; // => { code, map, ast }
});
// 或者是同步 api：
babel.transformFileSync("filename.js", options);
// => { code, map, ast }
// 要是已经有一个 Babel AST（抽象语法树）了就可以直接从 AST 进行转换。
babel.transformFromAst(ast, code, options);
// => { code, map, ast }
```

对于上述所有方法，options 指的都是 http://babeljs.io/docs/usage/options/

### 1.3 配置babel
目前为止通过运行 Babel 自己我们并没能“翻译”代码，而仅仅是把代码从一处拷贝到了另一处。

这是因为我们还没告诉 Babel 要做什么。

由于 Babel 是一个可以用各种花样去使用的通用编译器，因此默认情况下它反而什么都不做。你必须明确地告诉 Babel 应该要做什么。

我们一般项目中会对babel进行一些配置，主要是在 .babelrc 中对 presets 和 plugins 配置。

- <strong>presets</strong> （预设）就是一组插件，比如官方封装好的 `@babel/preset-react `，其中就包括以下插件`@babel/plugin-syntax-jsx`，`@babel/plugin-transform-react-jsx`，`@babel/plugin-transform-react-display-name`。
- <strong>plugins</strong> 相对来说功能比较单一，比如`transform-es2015-arrow-functions`，这个插件只负责转译es2015新增的箭头函数。

## 二、babel的编译流程
编译流程分为三个阶段：解析、转换、生成。
### 2.1 解析（Parse）

将源代码解析成AST抽象语法树。

解析步骤又分为两个阶段：词法分析（Lexical Analysis） 和 语法分析（Syntactic Analysis）。

#### 2.1.1 词法分析
词法分析阶段把字符串形式的代码转换为 令牌（tokens） 流。

你可以把令牌看作是一个扁平的语法片段数组：比如 `n * n;` 
```
[
  { type: { ... }, value: "n", start: 0, end: 1, loc: { ... } },
  { type: { ... }, value: "*", start: 2, end: 3, loc: { ... } },
  { type: { ... }, value: "n", start: 4, end: 5, loc: { ... } },
  ...
]
```
每一个 type 有一组属性来描述该令牌：
```
{
  type: {
    label: 'name',
    keyword: undefined,
    beforeExpr: false,
    startsExpr: true,
    rightAssociative: false,
    isLoop: false,
    isAssign: false,
    prefix: false,
    postfix: false,
    binop: null,
    updateContext: null
  },
  ...
}
```
和 AST 节点一样它们也有 start，end，loc 属性。

#### 2.1.2 语法分析

语法分析阶段会把一个令牌流转换成 AST 的形式。 这个阶段会使用令牌中的信息把它们转换成一个 AST 的表述结构，这样更易于后续的操作。

### 2.2 转换（Transform）

转换步骤接收 AST 并对其进行遍历，在此过程中对节点进行添加、更新及移除等操作。 这是 Babel 或是其他编译器中最复杂的过程 同时也是插件将要介入工作的部分。
将源代码版本的AST转换成目标版本的AST。

### 2.3 重新生成（Generate）
将目标版本的AST生成目标代码（将AST对象转换成字符串形式的代码，同时还会创建源码映射source  map）。至此就将源代码编译完成了。

其中，解析、生成阶段由`@babel/core`完成，转换阶段由babel插件完成，这也是本文的重点。

个人理解：
- babel的作用就是对源代码进行编译（翻译）：`Source Code`  --> `babel`  --> `new Source Code`

## 三、抽象语法树（AST）

>在计算机科学中，抽象语法树（Abstract Syntax Tree，AST），或简称语法树（Syntax tree），是源代码语法结构的一种抽象表示。它以树状的形式表现编程语言的语法结构，树上的每个节点都表示源代码中的一种结构。之所以说语法是“抽象”的，是因为这里的语法并不会表示出真实语法中出现的每个细节。

开发一个babel插件一定要对AST有所了解，上面是维基百科对抽象语法树的解释。
用这个在线查看代码ast结构的网站[astexplorer.net/](astexplorer.net/)，有助于更好的了解ast以及编写babel插件。

举个🌰：
```
var a = 1; 的ast如下：
{
  "type": "Program",
  "start": 0,
  "end": 10,
  "body": [
    {
      "type": "VariableDeclaration",
      "start": 0,
      "end": 10,
      "declarations": [
        {
          "type": "VariableDeclarator",
          "start": 4,
          "end": 9,
          "id": {
            "type": "Identifier",
            "start": 4,
            "end": 5,
            "name": "a"
          },
          "init": {
            "type": "Literal",
            "start": 8,
            "end": 9,
            "value": 1,
            "raw": "1"
          }
        }
      ],
      "kind": "var"
    }
  ],
  "sourceType": "module"
}
```

下面正式进入主题，自己开发一个 babel 插件：

## 四、开发插件
目标：
- 将 es6 语法的 let  改为 var
- 按需引入组件： 
import { Button } from 'antd'  ->   import Button form 'antd/lib/Button' 
### 4.1 安装脚手架和依赖
- @babel/cli、@bable/core 
```
npm i @babel/cli @babel/core -S
```
babel/cli 会调用 babel/core

### 4.2 新建文件编写插件逻辑
```
// src/index.js
const letToVar = function(babel) {
  const { types, template } = babel;
  return {
    visitor: {
      VariableDeclaration(path, state) {
        if (path.get('kind').node !== 'let') return;
        path.node.kind = 'var';
      },
      ImportDeclaration(path, state) { // 每个函数都会接收两个参数 path state
        const { opts: { libraryName, alias } } = state;
        // 判断如果不是我们的指定的包名就return 不用处理
        if (!types.isStringLiteral(path.node.source, { value: libraryName })) return;
        // 匹配到我们的包名，做自己的逻辑：替换节点
        // 生成节点
        const newImports = path.node.specifiers.map(item => {
          return types.importDeclaration([types.importDefaultSpecifier(item.local)], types.stringLiteral(`${alias}/${item.local.name}`))
        })
        // 替换节点
        path.replaceWithMultiple(newImports)
      }
    }
  }
}
module.exports = letToVar;
```
### 4.3 调试和使用
新建 `.babelrc.js ` 进行配置
```
{
    "plugins": [
    ["./src/index.js", {
        "libraryName": "lujun",
      "alias": "lujun/lib"
    }]
  ]
}
```
```
// src/test.js
import { A, B } from 'lujun';
let a = 1;
let b = 2;
```
```
// package.json
{
    "scripts": {
    "build": "babel src/test.js -d lib"
  }
}
```
执行 `npm run build `
```
// lib/test.js
import A from 'lujun/lib/A';
import B from 'lujun/lib/B';
var a = 1;
var b = 2;
```
最后发npm包即可在其他项目中使用了。

### 4.4 详细的知识点
下面介绍一些插件编写中更详细的知识点
- babelType：类似lodash那样的工具集，主要用来操作AST节点，比如创建、校验、转变等。举例：判断某个节点是不是标识符(identifier)。
- path：AST中有很多节点，每个节点可能有不同的属性，并且节点之间可能存在关联。path是个对象，它代表了两个节点之间的关联。你可以在path上访问到节点的属性，也可以通过path来访问到关联的节点（比如父节点、兄弟节点等）
- state：代表了插件的状态，你可以通过state来访问插件的配置项。
- visitor：Babel采取递归的方式访问AST的每个节点，之所以叫做visitor，只是因为有个类似的设计模式叫做访问者模式，不用在意背后的细节。
- Identifier、ASTNodeTypeHere：AST的每个节点，都有对应的节点类型，比如标识符（Identifier）、函数声明（FunctionDeclaration）等，可以在visitor上声明同名的属性，当Babel遍历到相应类型的节点，属性对应的方法就会被调用，传入的参数就是path、state。

<strong>访问者</strong> `visitor`

- 定义AST各种节点类型遍历时的操作方法，每个函数都接收两个参数 `path`  和 `state `
- AST节点类型有：
  - 变量 `VariableDeclaration`
  - 函数 `FunctionDeclaration`
  - 标识 `Identifier`
  - 二进制表达式 `BinaryExpression`
  - 文字/值 `Literal`
  - 更多可以自己查看[文档](https://github.com/babel/babel/blob/master/packages/babel-types/src/definitions/core.js)和AST实例

`path` 是遍历到当前的节点的路径。

`state.opts` 可以拿到配置 babel 时的 `options` 对象，用于可选逻辑

## 五、总结
Babel的插件入门比较简单，照葫芦画瓢即可。在编写插件过程中，可能会遇到的主要障碍，包括对ECMA规范不了解、对Babel的API不了解。
1. 对ECMA规范不了解：MemberExpression、FunctionDeclaration、Identifier等都是规范里的术语，如果对规范没有一定的了解，转换代码的时候就不知道如何入手。建议读者稍微了解下ECMA规范。
2. 对Babel的API不了解：Babel相关API的文档比较少，这会对插件编写造成不小的困难，目前比较好的解决办法，就是参考现有的插件进行修改。

总而言之，就是多看多写多查。

## 六、思考/TODO
- presets 有什么好处？
- plugins 和 presets的调用顺序？
- 如何开发一个presets？
- 分析插件源码
- `babel-plugin-import `
- `transform-es2015-arrow-functions`
- 小程序统一开发框架的是如何利用AST的？一次编写，多端使用的处理

## 七、参考
- [不容错过的 Babel7 知识](https://juejin.im/post/6844904008679686152)
- [手把手教你开发一个babel-plugin](https://segmentfault.com/a/1190000016459270)
