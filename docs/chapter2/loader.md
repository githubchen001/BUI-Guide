# 模块化

## 前言

?> 随着应用的功能逐渐丰富,逻辑的复杂度不断的增加,多人协作等问题, BUI有了自己的模块化方案, 类似`requirejs`的AMD. 熟悉`requirejs`,`seajs`都可以很好的适应过来. 

## 模块化解决什么问题?

- 命名冲突
- 文件依赖
- 多人协作
- 可维护性
- 跨页面共享


## 定义模块

!> `window.loader` 默认注册给了 `bui.loader`. 关于loader的用法,可以查看 <a href="http://www.easybui.com/demo/api/classes/bui.loader.html" target="_blank">bui.loader API</a>. 

### loader.define

*loader.define 定义一个匿名模块. *

```js
loader.define(function(require,exports,module){
    
    // 以下几个参数非必须,如果前面加载了依赖,则这三个参数后移;
    // require : 相当于 loader.require, 获取依赖的模块
    // exports : 如果没有return 可以采用这种方式输出模块
    // module : 拿到当前模块信息
    
    // 模块如果需要给其它模块加载,通过 return 的方式抛出来,或者module.exports的方式
    return {};
})
```
*定义模块需要遵循什么?*
1. 一个js 文件里面只能有一个 `loader.define` 的匿名模块;
2. 业务逻辑需要在 `loader.define` 里面,防止加载其它模块的时候冲突;
3. 避免循环依赖 A ->依赖 B 模块, 而 B模块 -> A模块, 这就造成循环依赖,一般需要避免这种设计,如果一定要用, 不使用依赖前置的方式;
4. 避免循环嵌套自身, 在loader.define 里面 又 require 加载当前模块, 这个时候还没实例化,就会造成死循环;

## 加载模块
### bui.require 

>假设我们定义了一个匿名模块, 是在pages/page2/目录下, 目录下有 page2.html ,page2.js 两个文件. 则默认匿名模块的 模块名是 pages/page2/page2 会根据.html 文件提取前面路径作为模块名.

page2.js
```
loader.define(function(require,exports,module){
  
    return {
      pageName: "page2"
    }
})
```
>现在我们想在刚刚的main.js里面加载这个模块,调用pages/page2/page2 的名称.

main.js
```
loader.define(function(require,exports,module){
    
    // 加载pages/page2/page2模块
    require("pages/page2/page2",function(page2){

        // 访问page2模块的名称
        console.log( page2.pageName )
    })

    return {
      pageName: "main"
    }
})
```
这样打开首页的时候,就会加载main.js, main.js 会去加载pages/page2/page2模块,成功以后输出名称.


模块的定义及加载更多用法，请大家自行查阅 <a href="http://www.easybui.com/demo/api/classes/bui.loader.html" target="_blank">bui.loader API</a> 


## 加载文件资源
### loader.import 

?> 没有经过define的第三方资源,又不想全局引用,可以使用`loader.import`动态引入进来. 例如,图表控件.


```js

例子1: 动态加载单个样式
loader.import("main.css",function(){
  // 创建成功以后执行回调
});

例子2: 动态加载单个脚本
loader.import("main.js",function(){
  // 创建成功以后执行回调
});

例子3: 动态加载多个脚本
loader.import(["js/plugins/baiduTemplate.js","js/plugins/map.js"],function(){
  // 创建成功以后执行回调
});

```
!> 样式的引入没有局部作用域,所以加载样式文件可能会造成影响全局,最好样式还是统一`sass模块化`管理.

## 获取及配置模块
### loader.map 
?> 可以用于设置或者获取已经加载的模块的相关信息

```js
例子1: 获取所有模块的配置信息
var map = loader.map();

例子2: 声明单个模块, router路由默认声明了main模块,页面打开会自动加载该模板下的资源,也可以通过map去修改
修改首页,必须在 window.router=bui.router(); 之后;

loader.map({
    moduleName: "main",
    template: "pages/main/main.html",
    script: "pages/main/main.js"
})

例子3: 定义多个模块,并修改路径
loader.map({
  baseUrl: "",
  modules: {
    "main": {
      moduleName: "main",
      template: "pages/main/main.html",
      script: "pages/main/main.js"
    }
    "home": {
      moduleName: "home",
      template: "pages/main/home.html",
      script: "pages/main/home.js"
    }
  }
})
```

*注意:*

- 定义了模块名以后,单页跳转则不能使用路径跳转,而需要传模块跳转, 例如, `router.load({url:"home"})`
- 如果`loader.define`的第一个参数有自定义名称, 则还需要通过`loader.map`配置下模块的路径及模板指向. 

## 疑难解答

?> 1. 如何抛出当前模块的方法共享 

>1. 推荐 使用return 的方式 ;
>2. 使用module.exports 的方式; 
>3. exports 的方式;
>使用任意一种就可以.


?> 2. 微信调试的缓存问题怎么解决?

*在 index.js 配置`bui.loader`的cache参数*
```js
window.loader = bui.loader({
    cache: false
})
```

?> 3. 为什么不直接采用requirejs或者seajs呢?

>这两种方式都有在项目中使用,这样模块的复用及开发方式就无法统一,A项目开发完的部分模块,可能B项目也能用,但两者各自用的模块化方式不同, 这就需要熟悉的人去做一定的修改. 采用我们自己的模块化方式,可以跟bui.router路由更好的配合, 后面模块化的公共插件也会越来越多, 这是我们以后希望看到的. 

?> 4. 如何定义模块的依赖呢?

main.js
```js
// 依赖前置, 这种会优先加载完 page2,page3模块以后再执行main的回调.
loader.define(["pages/page2/page2","pages/page3/page3"],function(page2,page3){
  // 如果需要用到当前模块信息的话, page3后面依次还有 require,exports,module 
  
})
```

?> 5. 如何定义一个自定义名字的模块呢?

* 第1步: 映射脚本路径

*index.js*

```js
// 映射脚本路径
loader.map({
  moduleName: "page2",
  script: "pages/page2/page2.js"
})

// 把路由实例化给 window.router 
window.router = bui.router();

bui.ready(function(){

    // 加载页面到div容器里面, 更多参数请查阅API
    router.init({
        id: "#bui-router"
    })
})
```
* 第2步: 声明自定义模块, 名称需要跟映射的模块名一致

*pages/page2/page2.js*
```js
loader.define("page2",function(require,exports,module){
  // 这里是page2的业务逻辑 
})

```

模块的定义及加载更多用法，请大家自行查阅  <a href="http://www.easybui.com/docs/index.html?id=api" target="_blank">bui.loader API</a> 

