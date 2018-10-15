---
title: webpack Tree Shaking
date: 2018-10-15 20:58:12
tags: [webpack, 工程化]
categories: 基础杂谈
type:

---

> 纸上得来终觉浅，绝知此事要躬行

## 前言

很久前看过`webpack`，入职来一直做小程序快应用相关需求，对于框架和工程化的东西渐渐拉下了，忽然发现webpack都到4.0了，想想之前看还是2.0时代，真是是新月异啊，`webpack`入门的初始化配置就不再记述，基本看下编译之后的文件就能读明白（普通模块依赖、动态引用编译）主要记述下自己看`tree shaking`时的坑。

## Tree Shaking

###  它是何方神圣

首先介绍下什么是`tree shaking`，可以理解为通过工具"摇"我们的JS文件，将其中用不到的代码"摇"掉，是一个性能优化的范畴。具体来说，在 `webpack `项目中，有一个入口文件，相当于一棵树的主干，入口文件有很多依赖的模块，相当于树枝。实际情况中，虽然依赖了某个模块，但其实只使用其中的某些功能。通过` tree-shaking`，将没有使用的模块摇掉，这样来达到删除无用代码的目的。

<!--more-->

### how原理

Tree-shaking 较早由 Rich_Harris 的 rollup 实现，后来，`webpack2` 也增加了`tree-shaking` 的功能。

`Tree-shaking`的本质是消除无用的js代码，这个称之为`DCE（dead code elimination）`。

`Dead Code` 一般具有以下几个特征:

- 代码不会被执行，不可到达
- 代码执行的结果不会被用到
- 代码只会影响死变量（只写不读）

传统编译型的语言中，都是由编译器将`Dead Code`从AST（抽象语法树）中删除，那javascript中是由谁做DCE呢？

首先肯定不是浏览器做DCE，因为当我们的代码送到浏览器，那还谈什么消除无法执行的代码来优化呢，所以肯定是送到浏览器之前的步骤进行优化。

其实也不是通常使用的打包工具`rollup、webpack`做的，而是著名的代码压缩优化工具uglify，uglify完成了javascript的DCE（本人就是因为知道这一点纠结了很久，5555）。

## 看下编译代码
首先时入口和依赖文件，可以看到math.js导出了square和cube两个函数，index.js引入了其中一个，按照前面说的原理编译后不应该存在cube导出的函数的，结果却出乎预料。

````js
// index.js 入口文件
import {cube} from './math.js';
console.log(cube(5))
````

```js
// math.js
export function square(x) {
  return x * x;
}

export function cube(x) {
  return x * x * x;
}
```

编译后代码：

```js
/******/ (function(modules) { // webpackBootstrap
/******/ 	// The module cache
/******/ 	var installedModules = {};
/******/
/******/ 	// The require function
/******/ 	function __webpack_require__(moduleId) {
/******/
/******/ 		// Check if module is in cache
/******/ 		if(installedModules[moduleId]) {
/******/ 			return installedModules[moduleId].exports;
/******/ 		}
/******/ 		// Create a new module (and put it into the cache)
/******/ 		var module = installedModules[moduleId] = {
/******/ 			i: moduleId,
/******/ 			l: false,
/******/ 			exports: {}
/******/ 		};
/******/
/******/ 		// Execute the module function
/******/ 		modules[moduleId].call(module.exports, module, module.exports, __webpack_require__);
/******/
/******/ 		// Flag the module as loaded
/******/ 		module.l = true;
/******/
/******/ 		// Return the exports of the module
/******/ 		return module.exports;
/******/ 	}
/******/
/******/
/******/ 	// expose the modules object (__webpack_modules__)
/******/ 	__webpack_require__.m = modules;
/******/
/******/ 	// expose the module cache
/******/ 	__webpack_require__.c = installedModules;
/******/
/******/ 	// define getter function for harmony exports
/******/ 	__webpack_require__.d = function(exports, name, getter) {
/******/ 		if(!__webpack_require__.o(exports, name)) {
/******/ 			Object.defineProperty(exports, name, { enumerable: true, get: getter });
/******/ 		}
/******/ 	};
/******/
/******/ 	// define __esModule on exports
/******/ 	__webpack_require__.r = function(exports) {
/******/ 		if(typeof Symbol !== 'undefined' && Symbol.toStringTag) {
/******/ 			Object.defineProperty(exports, Symbol.toStringTag, { value: 'Module' });
/******/ 		}
/******/ 		Object.defineProperty(exports, '__esModule', { value: true });
/******/ 	};
/******/
/******/ 	// create a fake namespace object
/******/ 	// mode & 1: value is a module id, require it
/******/ 	// mode & 2: merge all properties of value into the ns
/******/ 	// mode & 4: return value when already ns object
/******/ 	// mode & 8|1: behave like require
/******/ 	__webpack_require__.t = function(value, mode) {
/******/ 		if(mode & 1) value = __webpack_require__(value);
/******/ 		if(mode & 8) return value;
/******/ 		if((mode & 4) && typeof value === 'object' && value && value.__esModule) return value;
/******/ 		var ns = Object.create(null);
/******/ 		__webpack_require__.r(ns);
/******/ 		Object.defineProperty(ns, 'default', { enumerable: true, value: value });
/******/ 		if(mode & 2 && typeof value != 'string') for(var key in value) __webpack_require__.d(ns, key, function(key) { return value[key]; }.bind(null, key));
/******/ 		return ns;
/******/ 	};
/******/
/******/ 	// getDefaultExport function for compatibility with non-harmony modules
/******/ 	__webpack_require__.n = function(module) {
/******/ 		var getter = module && module.__esModule ?
/******/ 			function getDefault() { return module['default']; } :
/******/ 			function getModuleExports() { return module; };
/******/ 		__webpack_require__.d(getter, 'a', getter);
/******/ 		return getter;
/******/ 	};
/******/
/******/ 	// Object.prototype.hasOwnProperty.call
/******/ 	__webpack_require__.o = function(object, property) { return Object.prototype.hasOwnProperty.call(object, property); };
/******/
/******/ 	// __webpack_public_path__
/******/ 	__webpack_require__.p = "/Users/zhouyh/codebase/webpack_test/dist";
/******/
/******/
/******/ 	// Load entry module and return exports
/******/ 	return __webpack_require__(__webpack_require__.s = "./src/index.js");
/******/ })
/************************************************************************/
/******/ ({

/***/ "./src/index.js":
/*!**********************!*\
  !*** ./src/index.js ***!
  \**********************/
/*! no exports provided */
/***/ (function(module, __webpack_exports__, __webpack_require__) {

"use strict";
eval("__webpack_require__.r(__webpack_exports__);\n/* harmony import */ var _math_js__WEBPACK_IMPORTED_MODULE_0__ = __webpack_require__(/*! ./math.js */ \"./src/math.js\");\n\nconsole.log(12)\nconsole.log(Object(_math_js__WEBPACK_IMPORTED_MODULE_0__[\"cube\"])(5))\n\n//# sourceURL=webpack:///./src/index.js?");

/***/ }),

/***/ "./src/math.js":
/*!*********************!*\
  !*** ./src/math.js ***!
  \*********************/
/*! exports provided: square, cube */
/***/ (function(module, __webpack_exports__, __webpack_require__) {

"use strict";
eval("__webpack_require__.r(__webpack_exports__);\n/* harmony export (binding) */ __webpack_require__.d(__webpack_exports__, \"square\", function() { return square; });\n/* harmony export (binding) */ __webpack_require__.d(__webpack_exports__, \"cube\", function() { return cube; });\nfunction square(x) {\n  return x * x;\n}\n\nfunction cube(x) {\n  return x * x * x;\n}\n\n//# sourceURL=webpack:///./src/math.js?");

/***/ })

/******/ });
```

坑爹的webpack最新版本编译后是eval函数解析了，可读性很差，大家感觉看着不易读可以去网上找找之前版本的，其实主要就是最后那个eval函数，可以看到cube和square函数都有定义，当时花了很多时间调试，比如和babel编译冲突，等等，最后居然发现是开发模式的原因，巨坑啊，webpack文档上写的就是开发模式的，哎，到底还是对原理了解不深，前面也说了tree shaking的原理是使用uglify实现的。

想想也能理解，开发模式是不开启开启压缩的，接下来看下压缩后的编译代码：

```js
! function(e) {
	var t = {};

	function r(n) {
		if (t[n]) return t[n].exports;
		var o = t[n] = {
			i: n,
			l: !1,
			exports: {}
		};
		return e[n].call(o.exports, o, o.exports, r), o.l = !0, o.exports
	}
	r.m = e, r.c = t, r.d = function(e, t, n) {
		r.o(e, t) || Object.defineProperty(e, t, {
			enumerable: !0,
			get: n
		})
	}, r.r = function(e) {
		"undefined" != typeof Symbol && Symbol.toStringTag && Object.defineProperty(e, Symbol.toStringTag, {
			value: "Module"
		}), Object.defineProperty(e, "__esModule", {
			value: !0
		})
	}, r.t = function(e, t) {
		if (1 & t && (e = r(e)), 8 & t) return e;
		if (4 & t && "object" == typeof e && e && e.__esModule) return e;
		var n = Object.create(null);
		if (r.r(n), Object.defineProperty(n, "default", {
				enumerable: !0,
				value: e
			}), 2 & t && "string" != typeof e)
			for (var o in e) r.d(n, o, function(t) {
				return e[t]
			}.bind(null, o));
		return n
	}, r.n = function(e) {
		var t = e && e.__esModule ? function() {
			return e.default
		} : function() {
			return e
		};
		return r.d(t, "a", t), t
	}, r.o = function(e, t) {
		return Object.prototype.hasOwnProperty.call(e, t)
	}, r.p = "/Users/zhouyh/codebase/webpack_test/dist", r(r.s = 0)
}([function(e, t, r) {
	"use strict";
	r.r(t), console.log(function(e) {
		return e * e * e
	}(5))
}]);
```

主要关注最后的立即执行函数传入的参数数组即可,可以看到现在只有用到的cube函数了。

```js
! function(e) {
}([function(e, t, r) {
	"use strict";
	r.r(t), console.log(function(e) {
		return e * e * e
	}(5))
}]);
```

