---
layout: post
title: sublime 快捷键集合
excerpt: sublime
comments: true
categories: blog
share: true
image:
  feature: sublime_text.jpg
---

<!-- <figure>
    <img src="/images/sublime1.jpeg">
    <figcaption>sublime text 被称作最性感的编辑器!</figcaption>
</figure> -->

#### 查找功能

* `Command + Shift + F`: 从当前项目中搜索目标;
* `Command + F`: 从当前文件中搜索目标;
* `Command + P`: 可寻找匹配的文件，方法，跳转行号等功能;
* `#`: 跳转到指定的symbol;
* `:`: 跳转到指定的行号;
* `@`: 跳转到指定的方法或函数;


#### 编辑功能

##### 单行或多行编辑

* `Command + L`: 选中当前行, 按住Command，持续点击L可以依次选取多行
* `Command + Shift + L`: 同时编辑多行，此时光标位于每行的行末，点击右键可以将光标移至行首
* `Command + Enter`: 在当前行后插入新行
* `Command + Shift + Enter`: 在当前行前插入新行
* `Control + Shift + K`: 删除当前行
* `Command + Shift + D`: 复制当前行或多行
* `Command + J`: 合并单行或多行J
* `Control + M`: 跳转至对应的括号
* `cmd + KU`:    小写转大写
* `cmd + KL`:    大写转小写

##### 注释

* `Command + /`: 注释当前行或所选的行
* `Command + Alt + /`: 块注释

##### 选取字符

* `Command + D`: 首先用光标选取字符，按住Command，持续点击D可以选取当前文件相同的字符,如果需要跳过当前选中的字符，`Command+K, Command +D`,如过要消除当前选中的字符，`Command + U`
* `Command + Control + G`: 一次性选取所有相同的词
* `Control + Shift + M`: 选取括号中的内容


##### 拆分窗口

* `Command + Alt + 1`: 单个窗口;
* `Command + Alt + 2`: 拆分为两个窗口;
* `Command + Alt + 5`: 拆分为两行两列（共四格）窗口;
* `Control + (1,2,3,4)`: 跳转至相应的窗口;
* `Control + shift + (1,2,3,4)`: 将当前的文件移动到对应的窗口;

##### 显示

* `Ctrl+Tab`: 按文件浏览过的顺序，切换当前窗口的标签页

##### 其他设置

###### 打开vim模式

在`Preferences->Setting:user`中添加以下代码：

```ruby
"ignored_packages": [
  "Vintage"
]
```

###### 显示全路径

```ruby
"show_full_path": true
```

###### 高亮编辑中的那一行

```ruby
"highlight_line": true
```

###### 保存的时候把无用的空格去掉

```
"trim_trailing_white_space_on_save": true
```

###### 宽度指导线

```ruby
"rulers": [80]
```

这个数字是字符的宽度，当开启这个设置的时候，会出现一条垂直的虚线。但你的代码宽度超出这条线的时候，说明你要重新组织一下了

###### 加粗文件夹名称

```ruby
"bold_folder_labels": true
```
