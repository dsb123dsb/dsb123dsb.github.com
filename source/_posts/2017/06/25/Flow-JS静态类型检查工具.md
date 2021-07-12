---
title: Flow-JS静态类型检查工具
date: 2017-06-25 20:08:37
tags: [js,flow]
categories: tools
type:
---
>纸上得来终觉浅，绝知此事要躬行

# 前言 #
最近学习vue源码看到下面代码

![](http://i.imgur.com/iNBVV3k.png)

心里不由WTF,查了下原来是flow代码,就立马去[官网把flow文档](https://flow.org/en/docs/)撸了一遍，网上又找了几遍文章，做下总结
<!--more-->

先看下简介

![](https://segmentfault.com/img/bVH6mL?w=1200&h=675)

Flow本质上也只是个检查工具，它并不会自动修正代码中的错误，也不会强制说你没按照它的警告消息修正，就不会让你运行程序。当然，并没有要求什么时候一定要用这类的工具，只是这种作法可以让你的代码更具强健性与提高阅读性，也可以直接避去很多不必要的数据类型使用上的问题，这种开发方式目前在许多框架与函数库项目，或是以JavaScript应用为主的开发团队中都已经都是必用工具。

- 由于Flow还是个年轻的项目，问题仍然很多，功能也没你想像中完整，用起来有时候会卡顿是正常的，效能仍须改善。以后用户愈来愈多就会愈作愈好。
- Windows平台的支持也是不久前(2016.8)时的事，Flow只支持64位元的作业系统，32位元就不能用了。
- 如果你是要学或用React或Vue.js等等，Flow是必学的。不管你要用不用，库源码里面都用了。

# Flow介绍 #
[Flow](https://flowtype.org/)是个JavaScript的静态类型检查工具，由Facebook出品的开源码项目，问世只有一年多，是个相当年轻的项目。简单来说，它是对比TypeScript语言的解决方式。

会有这类解决方案，起因是JavaScript是一种弱(动态)数据类型的语言，弱(动态)数据类型代表在代码中，变量或常量会自动依照赋值变更数据类型，而且类型种类也很少，这是直译式脚本语言的常见特性，但有可能是优点也是很大的缺点。优点是容易学习与使用，缺点是像开发者经常会因为赋值或传值的类型错误，造成不如预期的结果。有些时候在使用框架或函数库时，如果没有仔细看文件，亦或是文件写得不清不楚，也容易造成误用的情况。

这个缺点在应用规模化时，会显得更加严重。我们在开发团队的协同时，一般都是用详尽的文字说明，来降低这个问题的发生，但JS语言本身无法有效阻止这些问题。而且说明文件也需要花时间额外编写，其他的开发者阅读也需要花时间。在现今预先编译器流行的年代，像TypeScript这样的强(静态)类的JavaScript超集语言就开始流行，用严格的角度，以JavaScript语言为基底，来重新打造另一套具有强(静态)类型特性的语言，就如同Java或C#这些语言一样，这也是为什么TypeScript称自己是企业级的开发JavaScript解决方案。

>注: 强(静态)类型语言，意思是可以让变量或常量在声明(定义)时，就限制好只能使用哪种类型，之后在使用时如果发生类型不相符时，就会发出错误警告而不能编译。但不只这些，语言本身也会拓展了更多的类型与语法。

TypeScript自然有它的市场，但它有一些明显的问题，首先是JavaScript开发者需要再进一步学习，内容不少，也有一定陡峭的学习曲线，不过这还算小事情。重大的事情是需要把已经在使用的应用代码，都要整个改用TypeScript代码语法，才能发挥完整的功用。这对很多已经有内部代码库的大型应用开发团队而言，将会是个重大的决定，因为如果不往全面重构的路走，将无法发挥强(静态)类型语言的最大效用。

所以许多现行的开源码函数库或框架，并不会直接使用TypeScript作为代码的语言，另一方面当然因为是TypeScript并非普及到一定程度的语言，社群上有热爱的粉丝也有不是那么支持的反对者。当然，TypeScript也有它的优势，自从TypeScript提出了DefinitelyTyped的解决方式之后，让现有的函数库能额外再定义出里面使用的类型，这也是另一个可以与现有框架与库相整合的方案，这让许多函数库与框架都提交定义档案，提供了另一种选择。另一个优势是，TypeScript也是个活跃的开源码项目，发展到现在也有一段时间，算是逐渐成熟的项目。它的背后有微软公司的支持，在最近发布的知名的、全新打造过的Angular2框架中(由Google主导)，也采用了TypeScript作为基础的开发语言。

现在，Flow提供了另一个新的选项，它是一种强(静态)类型的辅助检查工具。Flow的功能是让现有的JavaScript语法可以事先作类型的声明(定义)，在开发过程中进行自动检查，当然在最后编译时，一样可以用babel工具来移除这些标记。

相较于TypeScript是另外重新制定一套语言，最后再经过编译为JavaScript代码来运行。Flow走的则是非强制与非侵入性的路线。Flow的优点是易学易用，它的学习曲线没有TypeScript来得高，虽然内容也很多，但大概一天之内学个大概，就可以渐进式地开始使用。而且因为Flow从头到尾只是个检查工具，并不是新的程序语言或超集语言，所以它可以与各种现有的JavaScript代码兼容，如果你哪天不想用了，就去除掉标记就是回到原来的代码，没什么负担。当然，Flow的功用可能无法像TypeScript这么全面性，也不可能改变要作某些事情的语法结构。

总结来说，这两种方式的目的是有些相似的，各自有优点也有不足之处，青菜萝卜各有所爱，要选择哪一种方式就看你的选择。

# 从一个小例子演示 #
这种类型不符的情况在代码中非常容易发生，例如以下的例子:
```js
	function foo(x) {
	  return x + 10
	}
	
	foo('Hello!')
```
x这个传参，我们在函数声明时希望它是个数字类型，但最后使用调用函数时则用了字符串类型。最后的结果会是什么吗？ "Hello!10"，这是因为加号(+)在JavaScript语言中，除了作为数字的加运算外，也可以当作字符串的连接运算。想当然这并不是我们想要的结果。

聪明如你应该会想要用类型来当传参的识别名，容易一眼看出传参要的是什么类型，像下面这样:
```js
	function foo(number) {
	  return number + 10
	}
```
但如果在复合类型的情况，例如这个传参的类型可以是数字类型也可以是布尔类型，你又要如何写得清楚？更不用说如果是个复杂的对象类型时，结构又该如何先确定好？另外还有函数的返回类型又该如何来写？

利用Flow类型的定义方式，来解决这个小案例的问题，可以改写为像下面的代码:
```js
	// @flow
	
	function foo(x: number): number {
	  return x + 10
	}
	
	foo('hi')
```
你有看到在函数的传参，以及函数的圆括号(())后面的两个地方，加了: number标记，这代表这个传参会限定为数字类型，而返回值也只允许是数字类型。

当使用非数字类型的值作为传入值时，就会出现由Flow工具发出的警告消息，像下面这样:

>message: '[flow] string (This type is incompatible with number See also: function call)'

这消息是说，你这函数的传参是string(字符串)类型，与你声明的number(数字)不相符合。

如果是要允许多种类型也是很容易可以加标记的，假使这个函数可以使用布尔与数字类型，但返回可以是数字或字符串，就像下面这样修改过:
```js
	// @flow
	
	function foo(x: number | boolean): number | string {
	  if (typeof x === 'number') {
	    return x + 10
	  }
	  return 'x is boolean'
	}
	
	foo(1)
	foo(true)
	foo(null)  // 这一行有类型错误消息
```
由上面这个小例子你可以想见，如果在多人协同开发某个有规模的JavaScript应用时，这种类型的输出输入问题就会很常遇见。如果利用Flow工具的检查，可以避免掉许多不必要的类型问题。

# 真实案例 #
可能你会认为Flow工具只能运用在小型代码中，但实际上Facebook会创造出Flow工具，有很大的原因是为了React与React Native。

举一个我最近正在研究的的函数库代码中[NavigationExperimental](https://github.com/facebook/react-native/blob/9ee815f6b52e0c2417c04e5a05e1e31df26daed2/Libraries/NavigationExperimental/NavigationTypeDefinition.js)(这网址位置有可能会变，因为是直接连到源码里)，这里面就预先声明了所有的对象结构，像下面这样的代码:
```js
	export type NavigationGestureDirection = 'horizontal' | 'vertical';
	
	export type NavigationRoute = {
	  key: string,
	  title?: string
	};
	
	export type NavigationState = {
	  index: number,
	  routes: Array<NavigationRoute>,
	};
	
	// ...
```
Flow具备有像TypeScript语言中，预先定义对象类型的作用。上面代码的都是这个组件中预先定义的类型，这些类型可以再套用到不同的代码文档之中。

	export type NavigationGestureDirection = 'horizontal' | 'vertical';

上面这行类似于列举(enum)的类型，意思是说要不就是'horizontal'(水平的)，要不然就'vertical'(垂直的)，就这两种字符串值可使用。
```js
	export type NavigationRoute = {
	  key: string,
	  title?: string
	};
```
这行里面用了一个问号(?)定义在title属性的后面，这代表这属性是可选的(Optional)，不过你可能会有点搞混，因为问号(?)可以放在两个位置，见下面的例子:
```js
	export type Test = {
	  titleOne?: string,
	  titleTwo: ?string
	}
```
titleOne代表的是属性为可自定义的(可有可无)，但一定是字符串类型。titleTwo代表的是类型可自定义，也就是值的部份除了定义的类型，也可以是null或undefined，不过这属性是需要的，而且你一定要给它一个值。好的，这有些太细部了，如果有用到再查手册文档就可以。
```
	export type NavigationState = {
	  index: number,
	  routes: Array<NavigationRoute>,
	};
```
上面的代码可以看到，只要是声明过的类型(type)，同样可以拿来拿在其他类型中套用，像这里的`Array<NavigationRoute>`，就是使用了上面已声明的NavigationRoute类型。它是一个数组，里面放的成员是NavigationRoute类型，是个对象的结构。

刚已经有说过Flow工具有很大的原因是为了React与React Native所设计，因为Flow本身就内建对PropTypes的检查功能，也可以正确检查JSX语法，在这篇官方文档中有说明，而这在之后介绍React的文档的例子中就可以看到。

# 安装与使用 #
Flow目前可以支持macOS、Linux(64位元)、Windows(64位元)，你可以从以下的四种安装方式选择其中一种:

直接从Flow的发布页面下载可运行档案，加到计算机中的PATH(路径)，让flow指令可以在命令列窗口访问即可。
透过npm安装即可，可以安装在全局(global)或是各别项目中。下面为安装在项目中的指令:

    npm install --save-dev flow-bin

macOS中可以使用homebrew安装:

    brew update
    brew install flow

透过OCaml OPAM套装管理程序打包与安装，请见[Flow的Github页面](https://github.com/facebook/flow)。
# Flow简单使用三步骤 #
## 第1步: 初始化项目 ##
在你的项目根目录的用命令列工具输入下面的指令，这将会创建一个`.flowconfig`文档，如果这文档已经存在就不需要再进行初始化，这个设置档一样是可以加入自定义的设置值，请参考[Advanced Configuration](https://flowtype.org/docs/advanced-configuration.html)这里的说明，目前有很多项目里面都已经内附这个设置档，例如一些React的项目:

	flow init

## 第2步: 在代码文档中加入要作类型检查的注释 ##
一般都在代码档案的最上面一行加入，没加Flow工具是不会进行检查的，有两种格式都可以:

    // @flow

或

	/* @flow */

## 第3步: 进行检查 ##
目前支持Flow工具插件的代码编辑工具很多，常见的Atom, Visual Studio Code(VSC), Sublime与WebStorm都有，当有安装搭配代码编辑工具的插件时，编辑工具会辅助显示检查的讯息。不过有时候会有点卡顿的要等一下，因为检查速度还不是那么快。

或是直接用下面的命令列指令来进行检查:

	flow check

在Visual Studio Code中因为它内建TypeScript与JavaScript的检查功能，如果要使用Flow工具来作类型检查，需要在用户设置中，加上下面这行设置值以免冲突:

	"javascript.validate.enable": false

# 转换(编译)有Flow标记的代码 #
>注: 有些脚手架就已经装好与设置好这个babel拓展插件，你不用再多安装了。

在开发的最后阶段要将原本有使用Flow标记，或是有类型注释的代码，进行清除或转换。转换的工作要使用babel编译器，这也是目前较推荐的方式。

使用babel编译器如果以命令列工具为主，可以使用下面的指令来安装在全局中:

	npm install -g babel-cli

再来加装额外移除Flow标记的npm套件`babel-plugin-transform-flow-strip-types`在你的项目中:

	npm install --save-dev babel-plugin-transform-flow-strip-types

然后创建一个`.babelrc`设置档案，档案内容如下:
```js
	{
	  "plugins": [
	    "transform-flow-strip-types"
	  ]
	}
```
完成设置后，之后babel在编译时就会一并转换Flow标记。

下面的指令则是直接把src目录的档案编译到dist目录中:

	babel src -d dist

当然，babel的使用方式不是只有上面说的这种命令列指令，你可以视项目的使用情况来进行设置。

# Flow支持的数据类型 #
Flow用起来是的确是简单，但里面的内容很多，主要原因是是要看实际不同的使用情况作搭配。JavaScript里面的原始数据类型都有支持，而在函数、对象与一些新的ES6中的类，在搭配使用时就会比较复杂，详细的情况就请到官网文档中观看，以下只能提供一些简单的介绍说明。

## 原始数据类型 ##
Flow支持原始数据类型，如下面的列表:

- boolean
- number
- string
- null
- void

其中的void类型，它就是JS中的`undefined`类型。

这里可能要注意的是，在JS中`undefined`与`null`的值会相等但类型不同，意思是作值相等比较时，像(`undefined == null`)时会为`true`，有时候在一些运行期间的检查时，可能会用值相等比较而不是严格的相等比较，来检查这两个类型的值。

所有的类型都可以使用垂直线符号(|)作为联合使用(也就是 OR 的意思)，例如`string | number`指的是两种类型其中一种都可使用，这是一种联合的类型，称为"联合(Union)类型"。

最特别的是可选的(Optional)类型的设计，可选类型代表这个变量或常量的值有可能不存在，也就是允许它除了是某个类型的值外，也可以是`null`或`undefined`值。要使用可选类型，就是在类型名称定义前加上问号(?)，例如?string这样，下面是一个简单的例子:

	let bar: ?string = null
## 字面文字(literal)类型 ##
字面文字类型指的是以真实值作为数据类型，可用的值有三种，即数字、字符串或布尔值。字面文字类型搭配联合的类型可以作为列举(enums)来使用，例如以下的一个扑克牌的类型例子:
```js
	type Suit =
	  | "Diamonds"
	  | "Clubs"
	  | "Hearts"
	  | "Spades";
	
	type Rank =
	  | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10
	  | "Jack"
	  | "Queen"
	  | "King"
	  | "Ace";
	
	type Card = {
	  suit: Suit,
	  rank: Rank,
	}
```
>注: type是Flow中定义类型别名(Type Alias)的关键字，是一种预先声明的类型，这些声明的标记一样只会在开发阶段中使用，最后编译去除。

## 类型别名 ##
类型别名(Type Alias)提供了可以预先定义与集中代码中所需要的类型，一个简单的例子如下:
```js
	type T = Array<string>
	var x: T = []
	x["Hi"] = 2 //有Flow警告
```
类型别名(Type Alias)也可以用于复杂的应用情况，详见Flow官网提供的[Type Aliases](https://flowtype.org/docs/type-aliases.html)内容。

## 任何的数据类型 ##
在某一些情况可能不需要定义的太过于严格，或是还在开发中正在调试时，有一种作为渐进的改善代码的类型。

Flow提供了两种特殊的类型可以作为松散的数据类型定义:

- any: 相当于不检查。既是所有类型的超集(supertype)，也是所有类型的子集(subtype)
- mixed: 类似于any是所有类型的超集(supertype)，但不同于any的是，它不是所有类型的子集(subtype)

`mixed`是一个特别的类型，中文是混合的意思，`mixed`算是any的"啰嗦"进化类型。mixed用在函数的输入(传参)与输出(返回)时，会有不一样的状态，例如以下的例子会出现警告:
```js
	function foo(x: mixed): string {
	  return x + '10'
	}
	
	foo('Hello!')
	foo(1)
```
>会出现警告消息如下:
[flow] mixed (Cannot be added to string)

这原因是虽然输入时可以用`mixed`，但Flow会认为函数中x的值不见得可以与string类型作相加，所以会请求你要在函数中的代码，要加入检查对传入类型在运行期间的类型检查代码，例如像下面修改过才能过关:
```js
	function foo(x: mixed): string {
	  if (typeof x === 'number' || typeof x === 'string') {
	    return x + '10'
	  }
	  throw new Error('Invalid x type')
	}
	
	foo('Hello!')
	foo(1)
```
`mixed`虽然"啰嗦"，但它是用来渐进替换any使用的，有时候往往开发者健忘或偷懒没作传入值在运行期间的类型检查，结果后面要花更多的时间才能找出错误点，这个类型的设计大概是为了提早预防这样的情况。

>注: 从上面的例子可以看到Flow除了对类型会作检查外，它也会请求对某些类型需要有动态的检查。在官方的文件可以参考[Dynamic Type Tests](https://flowtype.org/docs/dynamic-type-tests.html#_)这个章节。

## 复合式的数据类型 ##
数组(Array)

数组类型使用的是`Array<T>`，例如`Array<number>`，会限定数组中的值只能使用数字的数据类型。当然你也可以加入埀直线(|)来定义允许多种类型，例如`Array<number|string>`。

对象(Object)

对象类型会比较麻烦，主要原因是在JavaScript中所有的数据类型大概都可以算是对象，就算是基础数据类型也有对应的包装对象，再加上有个异常的null类型的typeof返回值也是对象。

对象类型在Flow中的使用，基本上要分作两大部份来说明。

第一种是单指Object这个类型，Flow会判断所有的基础数据类**不是**属于这个类型的，以下的例子全部都会有警告:
```js
	// 以下都有Flow警告
	
	(0: Object);
	("": Object);
	(true: Object);
	(null: Object);
	(undefined: Object);
```
其他的复合式数据类型，除了数组之外，都会认为是对象类型。如下面的例子:
```js
	({foo: "foo"}: Object);
	(function() {}: Object);
	(class {}: Object);
	([]: Object); // Flow不认为数组是属于对象
```
>注意: 上面有两个特例，`typeof null`typeof与`typeof []`都是返回'object'。也就是说在JS的标准定义中，`null`与`数组`用``检测都会返回对象类型。所以，Flow工具的检查会与JS预设并不相同，这一点要注意。

>注: typeof在Flow中有一些另外的用途，详见Typeof的说明。

第二种方式是要定义出完整的对象的字面文字结构，像`{ x1: T1; x2: T2; x3: T3;}`的语法，用这个结构来检查，以下为例子:
```js
	let object: {foo: string, bar: number} = {foo: "foo", bar: 0};
	
	object.foo = 111; //Flow警告
	object.bar = '111'; //Flow警告
```
函数(Function)

上面已经有看到，函数也属于对象(Object)类型，当然也有自己的`Function`类型，函数的类型也可以从两大部份来看。

第一是单指`Function`这个类型，可以用来定义变量或常量的类型。如下面的代码例子:

	var anyFunction: Function = () => {};

第二指的是函数中的用法，上面已经有看到函数的输出(返回值)与输入(传参)的用法例子。例如以下的例子:
```js
	function foo(x: number): number {
	  return x + 10;
	}
```
因为函数有很多种不同的使用情况，实际上可能会复杂很多，Flow工具可以支持目前最新的arrow functions、async functions与generator functions，详见[官方的这篇Functions](https://flowtype.org/docs/functions.html)的说明。

类(Class)

类是ES6(ES2015)中新式的特性，类目前仍然只是原型的语法糖，类本身也属于一种对象(Object)类型。类的使用情况也可能会复杂，尤其是涉及多型与实例的情况，详见Flow网站提供的[Classes](https://flowtype.org/docs/classes.html)内容。

# Flow的现在与未来的发展 #
Flow在[最近的博客](https://flowtype.org/blog/2016/10/13/Flow-Typed.html)中说明引入了`flow-typed`的函数库定义档("libdefs")，在这个Github存储库中将统一存放所有来自社群提供的函数库定义档案。这是一种可以让现有的函数库与框架，预先写出里面使用的类型定义。让项目里面有使用Flow工具与这些函数库，就可以直接使用这些定义档，以此结合现有的函数库与框架来使用。这个作法是参考TypeScript的DefinitelyTyped方式。因为这还是很新的消息(2016.10)，目前加入的函数库还没有太多，不过React周边的一些函数库或组件都已经开始加入，其他常用的像underscore、backbone或lodash也已经有人在提交或维护。

Flow另一个发展会是在开发工具的自动完成功能的改进，因为如果已经能在撰写代码时，就知道变量或常量的类型(静态类型)，那么在自动完成功能中就可以更准确地给出可用的属性或方法。这一个功能在Facebook自家的Nuclide开发工具的Flow说明页中就有看到。Nuclide是基于Atom开发工具之上的工具，计算机硬件如果不够力是跑不动的，而且它稳定性与运行速度都还需要再努力。这大概是未来可见到的一些新趋向。

# 结论 #
本文简单的说明了Flow工具的功能介绍，以及其中的一些简要的内容等等。相信看过后你已经对这个Flow工具有一些认识，相较于T**ypeScript的学习曲线****，Flow大概是等于不用学**。Flow虽然是一个很新的工具，但相当的有用，建议每个JavaScript开发者都可以试试，一开始不用学太多，大概这篇文档看完就可以开始用了。复杂的地方就再查找官方的文件即可。

对于每个正在使用JS开发稍具规模化的应用，或是开发开源码的函数库或框架的团队来说，让JS具有静态类型特性，是一个很重要而且必要的决定。以我的观察，在网络上一直有很多的超集语言(例如TypeScript)的爱好者，会提出要全面改用TypeScript(或其他超集语言)的声音，例如Vue.js在很早之前就有讨论是不是要全面采用TypeScript的声音。后来Vue.js只有提交TypeScript的[DefinitelyTyped](https://github.com/DefinitelyTyped/DefinitelyTyped/tree/master/vue)文档，但在2.0中则采行了Flow工具。在这篇Vue作者于知乎上发表的: [Vue 2.0 为什么选用 Flow 进行静态代码检查而不是直接使用 TypeScript？](https://www.zhihu.com/question/46397274)的内容中，你可以看到为何选择Flow的理由，这可能也是整个开发团队所认同的最后结果。作者回答的文中可以总结下面这句话:

>全部换 TS(TypeScript) 成本过高，短期内并不现实。 相比之下 Flow 对于已有的 ES2015 代码的迁入/迁出成本都非常低 … 万一哪天不想用 Flow 了，转一下，就得到符合规范的 ES。
总之，Flow提供了另一个选择，要用什么工具就看聪明的你如何选择了。

[参考文章地址 https://segmentfault.com/a/1190000008088489?utm_source=tuicool&utm_medium=referral](https://segmentfault.com/a/1190000008088489?utm_source=tuicool&utm_medium=referral)