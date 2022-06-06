<script id="MathJax-script" async src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js"></script>

[TOC]
## 安装
[sublineText下载](https://www.sublimetext.com/3)
## 更改插件存储目录
- Preferences->Browse Packages 找到`包存储目录`
- 关闭sublime 在sublime安装目录下新建Data
- 删除`默认包存储目录`(整个用户目录下的sublime都删掉)

## 安装插件
### Package Control管理插件
1. [安装方法](https://packagecontrol.io/installation)
2. 如果在Perferences->package settings中看到package control这一项，则安装成功。
3. Ctrl+Shift+P调出命令面板(Comment Palette)输入install 调出 Install Package 选项并回车，然后在列表中选中要安装的插件。
4. update更新插件，remove移除插件

### 插件介绍
|       插件      |                 功能                |
|-----------------|-------------------------------------|
| MardownEditing  | 提高Sublime中Markdown编辑特性的插件 |
| MarkdownPreview | 将md文件导出html，浏览器中预览    |
| tableEditor     | 表格编辑                            |

### markdown配置
1. 使用`MarkdownPreview`在浏览器预览md文件:在`Comment Palette`中打开`Markdown Preview in Browser`选择github API解析会生成HTML文件在浏览器中预览。
2. 自定义预览快捷键`alt+m`生成html并用浏览器打开
```json5
// Preferences->KeyBindings打开的文件的右侧栏的中括号中添加一行代码：
// "keys": ["自定义"], "parser"："markdown/github"
{ "keys": ["alt+m"], "command": "markdown_preview", "args":{"target": "browser", "parser":"markdown"}  }
```
3. `ctrl+B`用MD生成对应HTML文件

### table Editor插件
- 用于表格编辑的插件，使用默认配置即可。
- 在每次使用该插件时`ctrl + shift + p`选择`table Editor: Enable for current syntax`与`able Editor: Enable for current view`
- 输入首行按`tab`键就可生成下一行并跳转到下一单元格  
- [table Editor进阶使用方法](https://segmentfault.com/a/1190000007935021)  


## markdown使用
### 基础
[sublimeText3指南](https://www.cnblogs.com/gaosheng-221/p/6108033.html)
![插入图片.png](http://note.youdao.com/favicon.icox)
> 这是强调引用
*这是斜体*
**这是粗体**
__这是粗体__
***
---
- - -


### 使用TeX语法书写数学公式
- [mathjax github地址](https://github.com/mathjax/MathJax-src)
- [mathjax使用方法](https://blog.csdn.net/qq_30717203/article/details/81139708#t29)
- [tex教程](https://blog.csdn.net/mrgeroge/article/details/52549093)
- 只要在要渲染的网页中加入下面js代码，就可以解析latex语法 $$X\stackrel{F}{\longrightarrow}Y$$
```html
<script id="MathJax-script" async src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js"></script>
```
file:///D:/project/myNotes/%5Broot%5D/IMG/git%E6%96%87%E4%BB%B6%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F.png