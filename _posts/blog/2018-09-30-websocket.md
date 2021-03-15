---
title: "如何通过 rails5 的 action cable 实现实时通信"
categories: blog
scomments: true
---

1. 什么是 WebSocket？

WebSocket 是一种建立在 TCP 协议上的一种双全工通信协议。WebSocket 的出现主要是为了试图解决基于现有 http 基础架构下加的 http 双向通信问题。WebSocket 使得客户端和服务器之间的数据交换变得更加简单，允许服务端主动向客户端推送数据。在 WebSocket API 中，浏览器和服务器只需要完成一次握手，两者之间就直接可以创建持久性的连接，并进行双向数据传输。WebSocket 是一个功能完善的独立协议，并不仅限于在浏览器中使用，但是它主要还是应用 web 应用程序。


