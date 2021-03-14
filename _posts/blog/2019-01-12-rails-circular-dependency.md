---
title: 解决 rails 代码里的 circular dependency
categories:
  - rails
tags:
  - ruby
  - rails
---

## rails 中的循环依赖

做过 python 开发的同学对于循环依赖这个问题一定不会陌生，如果代码结构组织不合理，不同模块互相 import
就会导致循环依赖。而 ruby 却很少听过循环依赖这个名词，而最近的 rails 开发中，我却碰到了 ruby 版本的
循环依赖。

错误信息如下：

```
RuntimeError
Circular dependency detected while autoloading constant Utils::DateTool
```