## 如何发布自己的 npm 包

[TOC]

作为一名前端开发人员安装使用 npm 包是日常工作的一部分，优秀的 npm 包为我们提供了很多便利，也为我们的开发节省了很多的时间。很多公司都会拥有自己的私有 npm 仓库用于管理私有的 npm 包，所以学习开发发布一个好用 **No Bug npm** 包是一项很重要的技能，下面我会结合自己的实战心得总结下发布一个 npm 包的流程。

#### 1.初始化项目

```bash
mkdir project-name
cd project-name
npm init
```

#### 2.package.json 配置

- **name:** 包的标识，如果你打算把它发布到全局 registry，请确保这个标识是唯一的
- **version:** 是语义版本号（semver），包可以被发布任意多次，但每次发布必须包含新的版本号
- **description:** 包的描述，用以让其他 npm 用户搜索并了解你的项目，这个字段非必须，但推荐填写
- **main:** 项目暴露的入口文件,默认值为 index.js
- **scripts:** shell 执行命令
- **keywords:** 关键字利于搜索
- **author:** 是包的创建者或维护者，遵循 "Your Name <you@example.com> (http://your-website.com)" 这样的格式。
- **license:** 是包发布的法律条款，以及什么是包代码的许可用法(ISC | MIT | Apache...)
- **files:** 会发布到到仓库的文件白名单,npm 会列出项目中的所有文件。
- **repository:** 可以帮助其他用户找到包的代码托管处，并为其做贡献，这同样是一个可选但推荐填写的字段
- **bin:** 是一个让 npm 在包安装时给包创建 cli 命令（二进制）的映射表
- **contributors:** 是包的贡献者列表，如果有别人参与你的项目，你可以在这里指明
- **bugs:** 是帮助用户了解包现有问题的 URL 链接
- **homepage:** 是项目的主页，包含包的简介、文档和其他附加资源链接

#### 3.项目命名和版本管理规范

##### 项目命名

注册 npm 用户帐户或创建组织时，系统会授予与您的用户或组织名称匹配的范围。您可以将此作用域用作相关程序包的命名空间。

##### 版本管理

SemVer（Semantic Versioning，语义化版本控制）是 Github 起草的一个语义化版本号管理模块，它实现了版本号的解析和比较，规范版本号的格式，它解决了依赖地狱的问题。

**_基本版本格式_**

> 主版本号（Major）.次版本号（Minor）.修订号（Patch）
> 每个部分都为整数（>=0），按照递增的规则改变。

**_版本号递增规则_**

- 主版本号（Major）：当你做了不兼容的 API 修改
- 次版本号（Minor）：当你做了向下兼容的功能性新增
- 修订号（Patch）：当你做了向下兼容的问题修正

> 先行版本号及版本编译信息可以加到基本版本格式的后面，作为延伸
> 先行版本号由首位的连接号”-“、标识符号（由 ASCII 码的英文数字和连接号标识符[0-9A-Za-z-]组成）、句点”.“组成。如 1.0.0-alpha、1.0.0-alpha.1、1.0.0-0.3.7、1.0.0-x.7.z.92。先行版的优先级低于相关联的标准版本
> 版本编译信息由首位的一个加号和一连串以句点分隔的标识符号（由 ASCII 码的英文数字和连接号标识符[0-9A-Za-z-]组成）组成。如 1.0.0-alpha+001、1.0.0+20130313144700、1.0.0-beta+exp.sha.5114f85。判断版本优先层级时，版本编译信息可以被忽略

#### 4.开发功能以及打包编译

**前端库：** 需要考虑到使用场景和引用的模块类型，通常需要 gulp 或者 webpack 来进行打包构建成 umd 模块规范
**纯 js 库：** 需要通过编译成 es5 语法，通常需要 babel 或者 ts 进行编译

#### 5.README 文档编写

为了帮助其他人在 npm 上找到你发布的软件包，你需要提供一个文档包括安装，配置和使用程序包中的代码的说明。README 文档将会自动作为 npm 包的主页显示。

#### 6.发布到 npm 仓库

- 搜索项目名查看是否有重名项目

```bash
npm search package-name
```

- 注册登录

```bash
npm login
```

- 发布

```bash
cd project-dir
npm publish --access public
```

- 安装测试

```bash
npm install package-name
```

**常用 npm 命令表**
| Action | Commond |
| :----------------: | :------------------------------:|
| 创建软连接 | npm link [package-name] |
| 删除链接 | npm unlink |
| 登录 | npm login |
| 查找 npm 包 | npm search [package-name] |
| 发布 | npm publish [package-name] |
| 撤销发布 | npm unpublish [package-name] |
| 查看包信息 | npm info [package-name] |
| 查看包的当前版本 | npm view [package-name] version |
| 查看包的所有版本 | npm view [package-name] versions |
| 查看包安装依赖|npm ls |
