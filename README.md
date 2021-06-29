## 一,模块化的理解

### 1,什么是模块化

- 将一个复杂的程序依据一定的规则(规范)封装成几个块(文件), 并进行组合在一起
- 块的内部数据和实现是私有的,只是向外部提供暴露的接口与其它模块通信

### 2.模块的进化历程

- 全局 function 模式: 将功能封装成全局函数

  > - 编码:将功能封装成全局函数
  > - 问题:污染全局命名空间,容易引起命名冲突,并且模块间的关系不明显

```
function m1(){
/...
}
function m2(){
/..
}
```

- namespace 模式:简单对象封装

  > - 作用:减少了全局变量,解决命名冲突
  > - 问题:数据不安全(外部可以直接修改内部数据)

```
let myModule = {
  data:"www.baidu.com"
  foo(){
    console.log(this.data)
  },
  bar(){
    console.log(this.data)
  }
}
myModule.data = 'other data' //可以直接修改
myModule.foo() //other data
```

这会暴露所有模块成员,内部状态可以被外部改写

- IIFE 模式:匿名函数调用(闭包)

  > - 作用:数据是私有的,外部只能通过暴露的方法操作
  > - 编码:将数据和行为封装到一个函数内部,通过给 window 添加属性来向外暴露接口
  > - 问题:如果当前模块需要依赖另一个模块呢

```
(function(window){
  let data = 'www.baidu.com'
  function foo(){
    console.log(data)
  }
  function bar(){
    console.log(data)
    oterFun()
  }
  function otherFun(){
    console.log("内部私有方法")
  }
  window.myModule = {foo,bar}
})(window)

myModule.foo() //www.baidu.com
myModule.bar() //www.baidu.com 内部私有方法
console.log(myModule.data) //undefined
myModule.data = 'www.163.com'
myModule.foo() //www.baidu.com
```

- IIFE 模式增强:引入依赖
  这就是先打模块实现的基石

```
<script type="text/javascript" src="jquery-1.10.1.js"></script>
(function(window,$){
  let data = 'www.baidu.com'
  function foo(){
    console.log(data)
    $('body').css('backGround','red')
  }
  function bar(){
    console.log(data)
    oterFun()
  }
  function otherFun(){
    console.log("内部私有方法")
  }
  window.myModule = {foo,bar}
})(window)

myModule.foo() //www.baidu.com 页面背景变红
```

上述的例子必须先引入 JQuery 库,把这个库当作参数传入.既保证了模块的独立性,也使模块间的关系变得明显

### 3.模块化的好处

- 避免命名冲突
- 更好的分离,按需加载
- 更高的复用性
- 高可维护性

### 4.引入多个`<script>`后出现的问题

- 请求过多
  首先我们要多个依赖,那样就会发送多个请求.
- 依赖模糊
  我们不知道他们具体的依赖关系是什么,不了解他们之间的依赖关系就很任意导致加载顺序的错误
- 难以维护
  以上两种原因就导致了很难维护,可能会出现牵一发动全身的严重问题,当一个页面引入多个 js 后就会出现这些问题,而这些问题可以用模块化规范来解决,下面介绍开发中最流行的 commonjs,AMD,ES6,CMD 规范

## 二,模块化规范

### 1.CommonJS

###### (1) 概述

Node应用由模块组成，采用CommonJS模块规范。每个文件就是一个模块，有自己的作用域。在一个文件里面定义的变量、函数、类，都是私有的，对其他文件不可见。在服务器端，模块的加载时运行时同步加载的；在浏览器端，模块需要提前打包处理。

###### （2）特点

- 所有代码都是运行在模块作用域，不会污染全局作用域。
- 模块可以多次加载，但是只会在第一次加载时运行一次，然后运行结果就会被缓存，以后在加载，就直接读取缓存结果。要想让模块再次运行，必须清楚缓存。
- 模块加载的顺序，按照其在代码中出现的顺序。

###### （3）基本语法

- 暴露语法：`module.exports = value`或`exports.xxx = value`
- 引入模块：`require(xxx)`,如果是第三方模块，xxx为模块名；如果是自定义模块，xxx为模块文件路径

此处我们有给疑问：CommonJS暴露的模块到底是什么？CommonJS规范规定，每个模块内部，module变量代表当前模块。这个变量是一个对象，它的exports属性是对外的接口，加载某个模块，其实是加载该模块的module.exports属性

```
//example.js
var x =5
var addX = function(value){
retuen value + x;
};
module.exports.x = x;
module.exports.addX = addX;
```

上面代码通过module.export输出变量x和函数addX。

```
var example = require('./example.js');
console.log(example.x); //5
console.log(example.addX(5)); //10
```

require命令用于加载模块文件。**require命令的基本功能是，读入并执行一个js文件，然后返回模块的exports对象。如果没有发现指定模块，会报错**

###### (4)模块的加载机制

**CommonJS模块的加载机制是，输入的是被输出的值的拷贝。就是说，一旦输出一个值，模块内部的变化就影响不到这个值。**这点与ES6模块化有重大差异，请看下面这个例子：

```
//lib.js
var counter = 3;
function incCounter(){
	counter++;
}
module.exports = {
	counter，
	incCounter
}
```

上面代码输出内部变量counter和改写这个变量的内部方法incCounter。

```
//main.js
var counter = require('./lib').counter;
var incCounter = require('./lib').incCounter;
console.log(counter); //3
incCounter();
console.log(counter); //3
```

上面代码说明，counter输出以后，lib。js模块内部的变化就影响不到counter了，这是因为counter是一个原始类型的值，会被缓存。除非写成一个函数，才能得到内部变动后的值。

###### （5）服务器端的实现

1. 下载安装node.js

2. 创建项目结构

   **注意：用npm init 自动生成package.json时，package name(包名)不能有中文和大写**

```
|-modules
  |-module1.js
  |-module2.js
  |-module3.js
|-app.js
|-package.json
  {
    "name": "commonJS-node",
    "version": "1.0.0"
  }
```

​	3.下载第三方模块

`npm install uniq --save // 用于数组去重`

 	4.定义模块代码

```
//module.js
module.exports = {
	msg:'module',
	foo(){
		console.log(this.msg)
	}
}
```

```
//module2.js
module.exports = function(){
	console.log('module2')
}
```

```
//module3.js
exports.foo = function(){
	console.log('foo() module3')
}
exports.arr = [1,2,3,3,2]
```

```
//app.js
let uniq = require('unip')
let module1 = require('./module/module1')
let module2 = require('./module/module2')
let module3 = require('./module/module3')

module1.foo() //module1
module2() //module2
module3.foo() //foo() module3
console.log(uniq(module3.arr)) //[1,2,3]
```

5.通过node运行app.js

命令行输入`node app.js`,运行app.js

###### （6）浏览器端实现（借助Browserify）

1.创建项目结构

```
|-js
  |-dist //打包生成文件的目录
  |-src //源码所在的目录
    |-module1.js
    |-module2.js
    |-module3.js
    |-app.js //应用主源文件
|-index.html //运行于浏览器上
|-package.json
  {
    "name": "browserify-test",
    "version": "1.0.0"
  }
```

2.下载browserify

- 全局：npm install browserify -g
- 局部：npm install browserify --save-dev

3.定义模块代码（同服务器）

注意：`index.html`文件要运行在浏览器上，需要借助browserify将`app.js`文件打包编译，如果直接在`index.html`引入`app.js`就会报错！

4.打包处理js

跟目录下运行`browserify js/src/app.js -o js/dist/bundle.js`

5.页面使用引入

在index.html文件中引入`<script type="text/javascript" src="js/dist/bundle.js"></script>`

### 2.AMD

CommonJS规范加载模块是同步的,就是说,只有加载完成,才能执行后面的操作。AMD规范则是非同步加载模块，允许指定回调函数。由于Node.js主要用于服务器编程，模块文件一般都已经存在与本地硬盘，所以加载起来比较快，不用考虑非同步加载的方式，所以CommonJS规范比较适用。但是，如果是浏览器环境，要从服务器端加载模块，这时就必须采用非同步模式，因此浏览器端一般采用AMD规范。此外AMD规范比CommonJS规范就在浏览器端实现要来得早。

###### （1）AMD规范基本语法

​	定义暴露模块

```
//没有依赖的模块
define(function(){
	return 模块
})
```

```
//有依赖的模块
define(['module1','module2'], function(m1,m2){
	return 模块
})
```

###### 引入使用模块

```
require(['module1','module2'],function(m1,m2){
	使用m1/m2
})
```

###### （2）未使用AMD规范与使用require.js

通过比较两者的实现方法，来说明AMD规范的好处

- 未使用AMD规范

```
//dataService.js
(function(window){
	let msg = 'www.baidu.com'
	function getMsg(){
		return msg.toUpperCase()
	}
	window.dataService = {getMsg}
})(window)
```

```
//alerter.js
(function(window,dataService){
	let name = 'tom'
	function showMsg(){
		alert(dataService.getMsg()+','+name)
	}
	window.alerter = {showMsg}
})(window,dataService)
```

```
//main.js
(function(alerter){
	alerter.showMsg()
})(alerter)
```

```
//index.js
<div><h1>Modular Demo 1: 未使用AMD(require.js)</h1></div>
<script type="text/javascript" src="js/modules/dataService.js"></script>
<script type="text/javascript" src="js/modules/alerter.js"></script>
<script type="text/javascript" src="js/main.js"></script>
```

这种方式缺点明显：**首先会发送多个请求，其次引入的js文件顺序不能错，否则就报错**

- 使用require.js

RequireJS是一个工具库，主要用于客户端的模块管理。它的模块管理遵守AMD规范，RequireJS的基本思想是，通过define方法，将代码定义为模块；通过require方法，实现代码的模块管理。

接下来是AMD规范在浏览器实现的步骤：

 1.下载require.js，并引入

- 官网：`http://www.requirejs.cn/`
- github:`https://github.com/requirejs/requirejs`

然后将require.js导入项目：js/libs/require.js

 2.创建项目结构

```
|-js
  |-libs
    |-require.js
  |-modules
    |-alerter.js
    |-dataService.js
  |-main.js
|-index.html
```

3.定义require.js的模块代码

```
//dataService.js
//没有依赖的模块
define(function(){
	let msg = 'www.baidu.com'
	function = getMsg(){
		return msg.toUppreCase()
	}
	return {getMsg}
}
```

```
//alerter.js
//有依赖的模块
define(['dataService'],function(dataService){
	let name = 'tom'
	function showMsg(){
		alert(dataService.getMsg()+','+name)
	}
	//暴露模块
	return {showMsg}
})
```

```
//main.js
(function(){
	require.config({
		baseUrl:'js/',
		paths:{
			//映射：模块标识名：路径
			alerter:'./modules/alerter',
			dataService:'./modules/dataService'
		}
	})
	require(['alerter'],function(alerter){
		alerter.showMsg()
	})
})()
```

```
//index.html
<!DOCTYPE html>
<html>
  <head>
    <title>Modular Demo</title>
  </head>
  <body>
    <!-- 引入require.js并指定js主文件的入口 -->
    <script data-main="js/main" src="js/libs/require.js"></script>
  </body>
</html>
```

4.页面引入require.js模块

在index.html引入`<script data-main="js/main" src="js/libs/require.js"></script>`

**此外在项目中如何引入第三方库？**只需在上面代码的基础稍作修改：

```
// alerter.js文件
define(['dataService', 'jquery'], function(dataService, $) {
  let name = 'Tom'
  function showMsg() {
    alert(dataService.getMsg() + ', ' + name)
  }
  $('body').css('background', 'green')
  // 暴露模块
  return { showMsg }
})
```

```
// main.js文件
(function() {
  require.config({
    baseUrl: 'js/', //基本路径 出发点在根目录下
    paths: {
      //自定义模块
      alerter: './modules/alerter', //此处不能写成alerter.js,会报错
      dataService: './modules/dataService',
      // 第三方库模块
      jquery: './libs/jquery-1.10.1' //注意：写成jQuery会报错
    }
  })
  require(['alerter'], function(alerter) {
    alerter.showMsg()
  })
})()
```

上例是在aleter.js文件中引入jQuery第三方库，main.js文件也哟啊有相应的路径配置。

小结：**AMD模块定义的方法非常清晰，不会污染全局环境，能够清楚地显示依赖关系**。AMD模式可以用于浏览器环境，并且允许非同步加载模块，也可以根据需要动态加载模块。

### 3.CMD

CMD规范专门用于浏览器端，模块的加载时异步的，模块使用时才会加载执行。CMD规范整合了CommonJS和AMD规范的特点。在Sea.js中，所有js模块都遵循CMD模块定义规范。

###### （1）CMD规范基本语法

**定义暴露模块 **

```
//没有依赖的模块
define(function(require,exports,module){
	exports.xxx = value
	module.exports = value
})
```

```
//有依赖的模块
define(function(require,exports,module){
	//引入依赖模块(同步)
	var module2 = require('./module2')
	//引入依赖模块(异步)
		require.async('./mnodule3',function(m3){})
		//暴露模块
		exports.xxx = value
})
```

**引入使用模块:**

```
define(function(require){
	var m1 = require('./module1')
	var m2 = require('./module2')
	m1.show()
	m2.show()
})
```

###### （2）.sea.js简单使用教程

1.下载sea.js,并引入

- 官网: http://seajs.org/
- github : https://github.com/seajs/seajs

然后将sea.js导入项目：js/libs/sea.js

2.创建项目结构

```
|-js
  |-libs
    |-sea.js
  |-modules
    |-module1.js
    |-module2.js
    |-module3.js
    |-module4.js
    |-main.js
|-index.html
```

3.定义sea.js的模块代码

```
//module1.js文件
define(function(require,exports,module){
	//内部变量数据
	var data = 'atguigu.com'
	//内部函数
	function show(){
		console.log('module show()'+data)
	}
	//暴露
	exports.show = show
})
```

```
//module2.js
define(function(require,exports,module){
	module.exports={
		msg:'I Will Back'
	}	
})
```

```
//module3.js
define(function(require,exports,module){
	const API_KEY = 'abc123'
	exports.API_KEY = API_KEY
})
```

```
//module4.js
define(require,exports,module){
	//同步引入模块2
	var module2 = require('./module2')
	function show(){
		console.log('module4 show()'+ module2.msg)
	}
	exports.show = show
	require.async('./module3',function(m3){
		console.log('异步引入依赖模块3' + m3.API_KEY)
	})
}
```

```
//main.js
define(function(require){
	var m1 = require('./module1')
	var m4 = require('./module4')
	m1.show()
	m4.show()
})
```

4.在index.html中引入

```
<script type="text/javascript" src="js/libs/sea.js"></script>
<script type="text/javascript">
  seajs.use('./js/modules/main')
</script>
```

### 4.ES6模块化

ES6模块化的设计思想是尽量的静态化，使得编译时就能确定模块的依赖关系，以及输入和输出的变量。CommonJS和AMD模块，都只能在运行时确定这些东西。比如，CommonJS模块就是对象，输入时必须查找对象属性。

###### （1）ES6模块化语法

​	export命令用于规定模块的对外接口，import命令用于输入其他模块提供的功能。

```
//模块 math.js
var basicNum = 0;
var add = funtion (a,b){
	return a+b;
}
export { basicNum,add};
//引用
import {basicNum,add} from './math'
function test(ele){
	ele.textContent = add(99+basicNum)
}
```

上例所示，使用import命令的时候，用户需要知道加载的变量名或函数名,否则无法加载。为了给用户提供方便，让他们不用阅读文档就能加载模块，就要用到export default命令，为模块指定默认输出

```
// export-default.js
export default function(){
	console.log('foo');
}
```

```
// import-default.js
import customName from './export-default'
custonName(); // foo
```

模块默认输出，其他模块加载模块时，import命令可以为该匿名函数指定任意名字。

###### (2)ES6模块与CommonJS模块的差异

它们有两个重大差异：

1.CommonJS模块输出的时一个值的拷贝，ES6模块输出的是值的引用。

2.CommonJS模块运行时加载，ES6模块时编译时输出接口。

第二个差异时因为CommonJS加载的时一个对象，该对象只有在脚本运行完才会生成。而ES6模块不是对象，它的对外接口只是一种静态定义，在代码静态解析阶段就会生成。

下面重点解释第一个差异，我们还是举上面那个CommonJS模块的加载机制例子：

```
//lib.js
export let counter = 3;
export function incCounter(){
	conunter++;
}
//main.js
import { counter,incCounter } from './lib';
console.log(counter); //3
incCounter();
console.log(counter); //4
```

ES6模块的运行机制与CommonJS不一样。ES6模块是动态引用，并且不会缓存值，模块里面的变量绑定其所在的模块。

###### （3）ES6-bable-Browserify使用教程

​	简单来说就一句话：**使用Babel将ES6编译为ES5代码，使用Browserify编译打包js。

​	1.定义package.json文件

```
{
	'name':'es6-babel-browserify',
	'version':'1.0.0'
}
```

​	2.安装babel-cli babel-preset-es2015和browserify

- npm install babel-cli browserify -g
- npm install babel-preset-es2015 --save-dev
- preset预设（将es6转换成es5的所有插件打包）

 3.定义.babelrc文件

```
{
	'presets':['es2015']
}
```

4.定义模块代码

```
//module1.js
export function foo(){
	console.log('foo module1')
}
export function bar(){
	console.log('bar module1')
}
```

```
//module2.js
function fun1(){
	console.log('fun1 module2')
}
function fun2(){
	console.log('fun2 module2')
}
```

```
//module3.js
export default()=>{
	console.log('默认暴露')
}
```

```
//app.js
import {foo,bar} from './module1'
import { fun1, fun2 } from './module2'
import module3 from './module3'
foo() // foo module1
bar() // bar module1
fun1() // fun1 module2
fun2() // fun2 module2
module3() //默认暴露
```

5.编译并在index.html中引入

- 使用Babel将ES6编译为ES5代码（但包含CommonJS语法) : `babel js/src -d js/lib`
- 使用Browserify编译js ：`browserify js/lib/app.js -o js/lib/bundle.js`

然后在index.html 文件中引入

```
<script type="text/javascript" src="js/lib/bundle.js"></script>
```



**此外第三方库如何引入呢？**

首先第三方库依赖npm i jquery@1

然后在app.js文件中引入

```
//app.js
import { foo, bar } from './module1'
import { fun1, fun2 } from './module2'
import module3 from './module3'
import $ from 'jquery'

foo() // foo module1
bar() // bar module1
fun1() // fun1 module2
fun2() // fun2 module2
module3() //默认暴露
$('body').css('background', 'black') //页面背景变黑
```

## 三，总结

1.当一个功能需要高复用性，高维护性就需要采用模块化的方式

2.模块化起初是不同的功能放入不同发全局函数，但这样不仅会污染全局命名空间，函数间的模块关系也不明显。之后发展到简单对象封装，但这样内部数据可以被随意修改。之后便是闭包，解决了以上的问题，也确保了模块的独立性，模块间的关系也变的明显

3.模块化规范

- CommonJS规范主要用于服务端编程， 加载是同步的，这并不适用与浏览器，因为同步会导致加载阻塞，浏览器需要异步加载。因此有了AMD CMD解决方案
  - 通过 require加载js文件中exports出来的对象，并只在加载时运行并会缓存，并且输入的值是被输出的值的拷贝，一旦输出，模块内部就无法对值进行改变 ，需要在node环境下运行
- AMD规范在浏览器环境中异步加载模块，而且可以并行加载多个模块。不过，AMD规范开发成本高，代码的阅读和书写比较困难，模块定义方式的语义不顺畅。
  - 使用define 定义模块，return 返回需要用到的变量或对象， 通过使用require.config 进行模块的使用配置，在index.html 引入require.js 并指定入口js文件
- CMD规范与AMD规范很相似，都用于浏览器编程，依赖就近，延迟执行，可以容易在Node.js中运行。不过，依赖SPM打包，模块的加载逻辑偏重
  - 使用define定义模块 使用require同步加载模块 require.async异步加载模块 exports 进行暴露 在index.html中引入sea.js 并 seajs.use() 指定入口js文件
- **ES6在语言标准的层面上，实现了模块功能，而且实现得相当简单，完全可以取代CommonJS和AMD规范，完成浏览器和服务器通用的模块解决方案**
  - 使用export 或export default 导出需要使用的变量或对象      	import 进行导入 	export default 可以暴露任意数据类项，暴露什么数据，接收到就是什么数据 并在import时可以任意命名    
  - 与CommonJS不同 ES6模块是动态引用，不会有缓存值，模块里的变量绑定其所在的模块 并且ES6的对外接口只是一种静态定义，带代码静态解析阶段就会生成

