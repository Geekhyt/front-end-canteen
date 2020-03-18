# 老舅教你VS Code Disco


这是最好的时代，也是最坏的时代。

今年听到过最浪漫的一句话：`我们在键盘上留下的余温，也将随时代传递到更远的将来。`


感觉让理性的技术人多了份柔光滤镜。

也许你收藏了千篇万篇VS Code快捷键，很可惜却没能记住他们，是因为你没有实际操作过，英文不好没关系，你真正需要的是让你双手指尖的肌肉增加一些记忆。

为了让你们能跟着我一起操练起来，为了让你们节约宝贵的时间，提高工作效率、得到leader夸奖、同事羡慕你疯狂操作的同时还可以有时间快乐摸🐟。

我只能请出老舅了。

(`老舅来了赶紧点个赞`)

![](/images/vscode/vs2.png)


## 左边跟我一起画个龙🐲
### 左手键盘操作

左手不够长，那就右手来凑。**记住，[触控板锁上](https://jingyan.baidu.com/article/54b6b9c0b38bc02d583b47c6.html)，再把鼠标扔一边。**

#### 面板控制
`Command + Shift + P / F1` 命令面板

`Command + Shift + E` 文件资源管理器

`Command + Shift + F`跨文件搜索

`Command + Shift + D` 启动和调试

`Command + Shift + X`管理扩展

`Command + Shift + M`查看错误和警告

`Command + J` 打开关闭面板

`Command + N ` 新建文件

`Command + Shift + N ` 打开新的编辑器窗口

`Command + W ` 关闭当前编辑器内窗口

`Command + Shift + W ` 关闭当前的编辑器

`Command + / — ` 缩放

`Command + /` 添加注释

**Ctrl + `** 打开/关闭终端

## 晃动你的胯胯轴 
#### 移动你的代码块

`Command + Shift + Enter` 将光标移动到当前行的上面一行，开启新的一行代码

`Command + Enter` 将光标移动到当前行的下面一行，开启新的一行代码

`Option + 上下方向键` 将当前行，或者当前选中的几行代码，在编辑器里上下移动

`Shift + option + 上下方向键` 向上或向下复制一行

**这些操作好好练习一下，你的Cmd + C和Cmd + V键寿命能长点。**

### 格式化代码

`Option + Shift + F` 格式化代码

`Command + Shift + P `打开命令面板输入 tra 选择大小写实现切换

`Command + J` 合并代码行

选中代码块按`Tab`增加缩进，按`Shift + Tab`减少缩进

依次按下`Command + k Command + 0` 全部折叠代码

依次按下`Command + K Command + J` 全部展开代码


## 指向闪耀的灯球儿
### 操作光标
`Option + 左右方向键` 以单词为单位移动光标

`Command + 左右方向键` 以行首行尾为单位移动光标

`Command + 上下方向键` 以文档第一行和最后一行为单位移动光标

`Command + Shift + \ ` 以花括号为单位移动光标

`Option + 左右方向键 + Shift`  以单词为单位选中开头/结尾到光标之间的字符

`Command + Shift + 上下方向键` 以当前光标为单位选中前面/后面所有内容

`Option + Delete`  删除当前单词光标前的内容

`fn + Option + Delete` 删除当前单词光标后的内容

`fn + Command + Delete` 删除当前行光标右侧所有内容

`Command + Delete` 删除当前行光标左侧所有内容

`Command + Shift + K` 删除当前行

`Command + X ` 剪切当前行

`Command + U`  撤销光标的移动和选择

`Command + Shift + V ` 粘贴纯文本

#### 多光标组合技

`Command + Option + 下方向键` 在当前光标下创建新的光标

`Command + 右方向键` 将光标全部整理移动到每一行的行尾


##### Command + D
将光标处于需要创建多光标的单词处，按Command + D、Command + D、Command + D……即可实现在同一单词处添加光标

##### Option + Shift + I
选中内容的每一行行尾添加光标

### 跳转操作
`Command + P`搜索文件，选中即打开，如果想要保留原文件，在新窗口打开选中文件后按`Command + Enter`

`Ctrl + Tab`同时按下，先松开`Tab`，在列表中通过`Tab`切换选择你需要打开的文件，选中即松开`Ctrl`实现跳转。

`Ctrl + G:行号`可实现行跳转

`Command + F12`跳转到函数定义的位置

`Shift + F12`跳转到被引用的引用

## 在你右边画一道彩虹🌈
### 右手鼠标操作

虽然说快捷键是解放鼠标，但是VS Code对鼠标的支持也整挺好的。

- 单击鼠标左键：移动光标

- 双击：选中当前光标下的单词

- 三连击：选中当前行

- 四连击：选中整个文档

- 单击行号并移动鼠标即可选中多行代码

- 鼠标选中行直接拖放可以移动被选中的代码块

- 鼠标左键拖拽过程中按Option键 复制粘贴代码块
#### 多光标操作
按住`Option` 鼠标在需要创建光标处点击


## 如何查看已有快捷键/自定义快捷键？
在命令面板输入`“打开键盘快捷方式(Open Keyboard Shortcuts)”`并执行。

搜索框里输入对应字符`“cmd+c”`或者点击右侧小键盘图标，进行录制按键。

即可找到对应按键组合进行自定义修改。

## 正经插件推荐
正则大全 [any-rule](https://github.com/any86/any-rule)
[作者：铁皮饭盒](https://juejin.im/user/58913fdf8d6d810058206a75)

汉化 `chinese`

在浏览器中打开 `Open-In-Browser`

自动闭合HTML/XMl标签 `Auto Close Tag`

自动对应修改HTML/XMl标签 `Auto Rename Tag`

HTML片段/模板 `HTML Snippets`/`HTML Boilerplate`

高亮注释 `TODO Highlight`

代码风格 `stylelint`/`TSLint`

Vue开发必备 `Vetur`

React开发必备 `ES7 React/Redux/GraphQL/React-Native snippets`

Go开发必备 `Go`

ES6代码片段 `JavaScript (ES6) code snippets`

映射VSCode上的断点到 `Chrome Debugger for Chrome`

路径自动提示补全 `Path Intellisense`

弥补VSCode原生git不足 `GitLens`

渲染颜色到代码下 `vscode-pigments`

代码缩进提供颜色上的提示 `Indent Rainbow` 

npm的包最终导致项目的增加量 `Import Cost`

花括号单独配色 `Rainbow Brackets`

项目管理器，多项目开发者福音 `Project Manager`

同步VS Code配置 `Settings Sync`

代码格式化的神器 保证更容易写出风格一致的代码 `Prettier`

icons图标 `vscode-icons-mac`

更多插件请[自行探索](https://marketplace.visualstudio.com/search?target=VSCode&category=All%20categories&sortBy=Relevance)


**选择适合自己项目需求的插件安装下载**

![爱老舅，爱尤大](/images/vscode/vs3.png)











