---
title: '深入浅出nodejs学习笔记-前三章 简介、模块机制、异步I/O'
date: 2017-04-16 11:47:48
categories:
  - nodejs
tags:
  - nodejs
  - 学习笔记
---


## 第一章 node简介

在学习这章之前，先提三个问题，node是什么，为什么用node，node可以用来干什么？

**首先是回答第一个问题，node是什么？**

node习惯称为nodejs，听起来类似于js库，但是其实node并不是一个库，node其实是一个可以在后端运行JavaScript的环境。

js为什么可以在后端运行呢，不是只可以在浏览器运行吗？其实是js之所以可以再浏览器运行是因为在浏览器中集成了js解析引擎，类似Firefox的SpiderMonkey和IE的Chakra，其中最有名的当属于chrome的v8引擎，而node便是利用到了v8引擎，所以才可以在服务器端跑js，当如正如你所想的，因为node并没有渲染引擎，所以node是不处理UI的。

**那为什么要用node？**

作为一个后端js的运行平台，node要好用才会用啊，喏，如下：

1.  **异步IO**：其实就是异步执行的IO操作，不用多说
2.  **单线程**：node是单线程运行的，所以不用考虑多线程地带来的同步啊、死锁之类的问题，但同时会带来一些问题，比如无法充分的利用多核CPU、一旦报错，程序就GG、CPU过载就会出异常等，对于这些问题，node也有一个简单的解决办法就是child_process，这个暂时先不谈
3.  **跨平台**：作为一个后台运行平台，这是JavaScript浩浩荡荡进军后台之势，当然也要像后台爸爸Java一样跨平台才好用哈
4.  **事件与回调函数**：node是基于事件驱动的，基于事件编程有**轻量级、轻耦合、只关注事务点**等优势，其实就是更关注业务逻辑，毕竟是前端

**最后的问题，node可以用来干什么**

如上的node优势所说，一方面，node基于异步IO的特点，node擅长于处理IO密集型业务，另一方面，V8是十分强大的，所以node也很适合CPU密集型业务，而且效率不比java差哈，同时，node也比较适合于做一些游戏开发领域的事情、做分布式应用和一些工具类应用的开发


## 第二章 node模块机制

**node模块机制**

关于node的模块机制，首先提一下CommonJS，CommonJS就是为JS的表现来制定规范，因为js没有模块的功能所以CommonJS应运而生，它希望js可以在任何地方运行，不只是浏览器中。CommonJS的模块规范如下

<pre class="prettyprint"><span class="hljs-comment">//a.js</span>
exports.yeah = <span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-params">()</span> {</span>
    console.log(<span class="hljs-string">'yeah'</span>)
}

<span class="hljs-comment">//b.js</span>
<span class="hljs-keyword">var</span> yeah = <span class="hljs-built_in">require</span>(<span class="hljs-string">'yeah'</span>)</pre>

在文件a中通过exports暴露出去一个yeah方法，然后在b.js文件中通过require引用，十分方便,但是CommonJS的缺陷还是很明显的，**没有模块系统，标准库很少，没有标准的接口，缺乏包管理系统**等。

作为站在巨人肩膀上的node，node在实现中并非完全按照CommonJS的模块规范，而是对模块规范进行了一定的取舍，同事也增加了不少自身需要的特性。

node的模块分为两类，一种是Node提供的模块，叫做**核心模块**;一种是用户自己编写的模块，叫做**文件模块** 

核心模块被编译成二进制执行文件，在node启动时就被直接加载到内存中，所以执行速度最快，文件模块则是运行时加载，按照路径分析 –&gt; 文件定位 –&gt; 编译执行 的步骤加载执行

node有自己的包规范NPM，npm是什么，npm是一个包管理的神器，这个真的不能多说，每天都用

**前后端共用模块**

前后端共用模块主要有两种，AMD和CMD

AMD 即Asynchronous Module Definition，中文名是异步模块定义的意思。它是一个在浏览器端模块化开发的规范，由于不是JavaScript原生支持，使用AMD规范进行页面开发需要用到对应的库函数，也就是大名鼎鼎RequireJS，实际上AMD 是 RequireJS 在推广过程中对模块定义的规范化的产出

requireJS主要解决两个问题

1.  多个js文件可能有依赖关系，被依赖的文件需要早于依赖它的文件加载到浏览器
2.  js加载的时候浏览器会停止页面渲染，加载文件越多，页面失去响应时间越长

    看一个使用requireJS的例子

<pre class="prettyprint"><span class="hljs-comment">// 定义模块 myModule.js</span>
define([<span class="hljs-string">'dependency'</span>], <span class="hljs-function"><span class="hljs-keyword">function</span><span class="hljs-params">()</span>{</span>
    <span class="hljs-keyword">var</span> name = <span class="hljs-string">'Byron'</span>;
    <span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">printName</span><span class="hljs-params">()</span>{</span>
        console.log(name);
    }

    <span class="hljs-keyword">return</span> {
        printName: printName
    };
});

<span class="hljs-comment">// 加载模块</span>
<span class="hljs-built_in">require</span>([<span class="hljs-string">'myModule'</span>], <span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-params">(my)</span>{</span>
　 my.printName();
});</pre>

CMD由国内的玉伯提出，CMD模块更接近于Node对CommonJS的实现，CMD和AMD有什么区别，看下[玉伯](https://www.zhihu.com/people/lifesinger/answers)是怎么回答的吧[https://www.zhihu.com/question/20351507/answer/14859415](https://www.zhihu.com/question/20351507/answer/14859415)


## 第三章 异步IO

异步IO真心没啥可提的，需要知道的是异步IO实现的核心是事件循环(过去是轮询技术)，他与浏览器中的执行模型基本保持了一致。

* * *

<br/>**前端新手，弱鸡一枚，如有错误，请指正，谢谢！**
