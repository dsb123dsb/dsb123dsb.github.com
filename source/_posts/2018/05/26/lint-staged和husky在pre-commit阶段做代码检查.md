---
title: lint-staged和husky在pre-commit阶段做代码检查
date: 2018-05-26 09:49:24
tags: [工程化]
categories: tools
type:
---

> 纸上得来终终觉浅，绝知此事要躬行

# 前言

前几天给项目代码加eslint，并且使用lint-staged和husky在pre-commit阶段做代码检查，也踩了个坑，这里对这两个npm包作简要介绍。

# 为什么使用

通常项目中我们通过`eslint`和`stylelint`这些lint工具来检查代码的规范与否，保证良好的代码规范，从而在多人协作中保障项目质量和可维护性。正常我们会在提交代码前手动执行语法检查，而`lint-staged`和`husky`而让这一过程自动化，在git的pre-commit阶段来检测你的代码，如果存在语法错误会中断commit。

<!--more-->

# husky

[husky](https://github.com/typicode/husky)可以让git hooks的使用变得更简单方便。运行`npm install husky@next --save-dev`安装最新版本，它会在我们项目根目录下面的`.git/hooks`文件夹下面创建`pre-commit`、`pre-push`等hooks。这些hooks可以让我们直接在`package.json`的`script`里运行我们想要在某个hook阶段执行的命令。

版本0.14之后配置有所改变，之前不知道npm安装使用@next会安装最新开发版本，一直安装稳定版本却使用心得配置，也算踩的坑吧，具体配置：

```json
// package.json， 最新版本
{
  "husky": {
    "hooks": {
      "pre-commit": "npm test",
      "pre-push": "npm test",
      "...": "..."
    }
  }
}
// package.json 版本0.14之前
{  "scripts": {    
	"precommit": "npm test",  
    "prepush": "npm test",  
    "...": "..."  
 }}
```

而仅使用husky在提交代码时会检查所有文件，我们肯定不希望这样，仅仅检查git add .的文件才是我们期望的。

# lint-staged

[lint-staged](https://github.com/okonet/lint-staged)可以在git staged阶段的文件上执行linters，简单点来说就是当我们运行`eslint`或`stylelint`的命令时，只会检查我们通过`git add`添加到暂存区的文件，可以避免我们每次检查都把整个项目的代码都检查一遍。

```json
// package.json
"lint-staged": {
    "*.js": ["eslint --fix", "git add"]
}
```

