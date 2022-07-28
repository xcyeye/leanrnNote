---
date: 2021/10/14 21:19
keyword: 'git,commit规范,git提交规范,git生成日志,Commit message,Commitizen,validate-commit-msg,conventional-changelog'
description: 规范的使用git进行commit提交，并且搭配Commitizen完成提交检验，使用conventional-changelog工具生成提交日志
---



# 关于commit message 和change log的编写规范

::: tip

这篇文章是关于如何规范化的编写commit message，以及生成Change log文件的过程，此篇文章是我看<a href="http://www.ruanyifeng.com/blog/2016/01/commit_message_change_log.html" target="_blank">阮一峰的网络日志</a>之后总结的学习笔记

:::



这几天都在更改主题的bug，其中提交到git是必须的，就查找了git提交的规范，看完我才知道，原来我以前的`commit message`啥都不是，以前就是随便写commit message，对于一个开发者来说，无论是写代码，还是做其他有关开发的事，一定要遵循规范，这个规范可以是你公司自己的，也可以是其他大家都需要遵循的，如果你写Java，那么你就要遵守阿里巴巴开发规范，其能够让我们养成一种习惯，下面是我学习的笔记，如果有不对的地方，欢迎大家留言指出，一起学习。



## Commit message 的格式



每一次提交，commit message都必须是

```
<type>(<scope>): <subject> // header
// 空一行
<body>
// 空一行
<footer>
```



::: warning

其中，header是必需的，body和footer可以省略

:::



## Header

### type

`type`用于说明 commit 的类别

目前，我使用`Commitizen`的`git cz`，可以看到，这个值有11个

| type  | description   |
| ----- | ------------- |
| feat | A new feature(`新功能`) |
| fix   | A bug fix(`bug修复`) |
| style | Changes that do not affect the meaning of the code (white-space, formatting, missing semi-colons, etc)(`格式，不影响代码运行的变动`) |
| refactor |A code change that neither fixes a bug nor adds a feature(`代码重构，不是新增功能，也不是修改bug的代码变动`)|
| perf | A code change that improves performance(`提高代码性能的改变`) |
| test | Adding missing tests or correcting existing tests(`增加测试`) |
| build | Changes that affect the build system or external dependencies (example scopes: gulp, broccoli, npm)(`影响构建系统或外部依赖项的更改`) |
| ci | Changes to our CI configuration files and scripts (example scopes: Travis, Circle, BrowserStack, SauceLabs)(`例如脚本的改变`) |
| chore | Other changes that don't modify src or test files(`其他修改, 比如构建流程, 依赖管理.`) |
| revert | Reverts a previous commit(`恢复之前的提交`) |
| docs | Documentation only changes(`仅仅是文档的更改`) |



::: tip

如果`type`为`feat`和`fix`，则该 commit 将肯定出现在 Change log 之中。其他情况（`docs`、`chore`、`style`、`refactor`、`test`）由你决定，要不要放入 Change log，建议是不要。

:::



### scope

`scope`用于说明 commit 影响的范围，比如数据层、控制层、视图层等等，视项目不同而不同。

### subject

`subject`是 commit 目的的简短描述，不超过50个字符。

关于subject需要遵循以下原则

- 以动词开头，使用第一人称现在时，比如`change`，而不是`changed`或`changes`
- 第一个字母小写
- 结尾不加句号（`.`）



如果不使用``git cz`，手动写commit message，那么header需要像下面一样(例如一个新功能的增加,影响到view层)

```sh
feat(view):add a new feature

#没有scope
feat:add a new feature
```



## Body

Body 部分是对本次 commit 的详细描述，可以分成多行，例如

```sh
feat(view):add a new feature #header

Further paragraphs come after blank lines. #body
#请注意，如果你想要在change log中以li展示body，这里需要空一行
- Bullet points are okay, too
- Use a hanging indent

Further paragraphs come after blank lines. 

- Bullet points are okay, too
- Use a hanging indent
```



::: warning

有两个注意点。

（1）使用第一人称现在时，比如使用`change`而不是`changed`或`changes`。

（2）应该说明代码变动的动机，以及与以前行为的对比。

:::



上面这个在`change log`文件中，会展示成下图效果

::: details 点击查看



![](https://picture.xcye.xyz/image-20211014220109861.png?x-oss-process=style/pictureProcess1)

:::





##  Footer

Footer 部分只用于两种情况

- **不兼容变动**(`BREAKING CHANGE`)
- **关闭 Issue**(`eg Closes #234`)



### **不兼容变动**

如果当前代码与上一个版本不兼容，则 Footer 部分以`BREAKING CHANGE`开头，后面是对变动的描述、以及变动理由和迁移方法

```sh
BREAKING CHANGE: isolate scope bindings definition has changed.

    To migrate the code follow the example below:

    Before:

    scope: {
      myAttr: 'attribute',
    }

    After:

    scope: {
      myAttr: '@',
    }

    The removed `inject` wasn't generaly useful for directives so there should be no code using it.
```





### **关闭 Issue**

如果当前 commit 针对某个issue，那么可以在 Footer 部分关闭这个 issue 。

```sh
Closes #234
```



也支持同时关闭多个issue

```sh
Closes #123, #245, #992
```





## Revert

还有一种特殊情况，如果当前 commit 用于撤销以前的 commit，则必须以`revert:`开头，后面跟着被撤销 Commit 的 Header。

```sh
revert: feat(pencil): add 'graphiteWidth' option

This reverts commit 667ecc1654a317a13331b17617d973392f415f02.
```

Body部分的格式是固定的，必须写成`This reverts commit <hash>.`，其中的`hash`是被撤销 commit 的 SHA 标识符。

如果当前 commit 与被撤销的 commit，在同一个发布（release）里面，那么它们都不会出现在 Change log 里面。如果两者在不同的发布，那么当前 commit，会出现在 Change log 的`Reverts`小标题下面。





上面使用`git cz`的操作打印结果为

```sh
feat(view): add a new featur #header

add user login #body

#footer
BREAKING CHANGE: The removed `inject` wasn't generaly useful for directives so there should be no 
code using it
#footer ---> 关闭#2
#2
```





## Commitizen

[Commitizen](https://github.com/commitizen/cz-cli)是一个撰写合格 Commit message 的工具。

其操作界面如下

![](https://picture.xcye.xyz/image-20211014222107313.png?x-oss-process=style/pictureProcess1)



### 安装

```sh
npm install -g commitizen
```

然后，在项目目录里，运行下面的命令，使其支持 Angular 的 Commit message 格式。

```bash
commitizen init cz-conventional-changelog --save --save-exact
```

以后，凡是用到`git commit`命令，一律改为使用`git cz`。这时，就会出现选项，用来生成符合格式的 Commit message。



::: warning

如果你使用`Git Bash`，那么你不能使用键盘上下键进行选择，所以不能使用`Git Bash`，windows的话，可以使用cmd

:::



## validate-commit-msg

[validate-commit-msg](https://github.com/kentcdodds/validate-commit-msg) 用于检查 Node 项目的 Commit message 是否符合格式。

它的安装是手动的。首先，拷贝下面这个[JS文件](https://github.com/kentcdodds/validate-commit-msg/blob/master/index.js)，放入你的代码库。文件名可以取为`validate-commit-msg.js`。

接着，把这个脚本加入 Git 的 hook。下面是在`package.json`里面使用 [ghooks](http://npm.im/ghooks)，把这个脚本加为`commit-msg`时运行。

```json
"config": {
    "ghooks": {
      "commit-msg": "./validate-commit-msg.js"
    }
  }
```

然后，每次`git commit`的时候，这个脚本就会自动检查 Commit message 是否合格。如果不合格，就会报错。

```
INVALID COMMIT MSG: does not match "<type>(<scope>): <subject>" ! was: edit markdown
```



## 生成 Change log

如果你的所有 Commit 都符合 Angular 格式，那么发布新版本时， Change log 就可以用脚本自动生成

生成的文档包括以下三个部分

- New features
- Bug fixes
- Breaking changes.



::: details view

![](https://picture.xcye.xyz/image-20211014222556607.png?x-oss-process=style/pictureProcess1)

:::



### 安装

```bash
npm install -g conventional-changelog
```

如果运行上面命令完成之后，运行

```bash
conventional-changelog -p angular -i CHANGELOG.md -w
```

报下面错误

::: danget

'conventional-changelog' 不是内部或外部命令，也不是可运行的程序 或批处理文件。

:::



那么你可以使用下面命令

 ```sh
npm install -g conventional-changelog-cli
 ```



为了方便操作，你可以将以下脚本写在`package.json`中

```json
"changelog": "conventional-changelog -p angular -i CHANGELOG.md -s -r 0 && git add CHANGELOG.md",
```

然后以后，直接运行`npm run changelog`便可以了





`package.json`的所有内容如下

```json
{
  "name": "commit",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "version": "conventional-changelog -p angular -i CHANGELOG.md -s -r 0 && git add CHANGELOG.md",
	"changelog": "conventional-changelog -p angular -i CHANGELOG.md -w -r 0"
  },
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "conventional-changelog": "^3.1.24",
    "cz-conventional-changelog": "^3.3.0",
    "validate-commit-msg": "^2.14.0"
  },
  "config": {
    "commitizen": {
      "path": "./node_modules/cz-conventional-changelog",
      "ghooks": {
        "commit-msg": "./validate-commit-msg.js"
      }
    }
  }
}
```

