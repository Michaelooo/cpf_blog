---
title: '深入浅出nodejs学习笔记--第九章 玩转进程管理'
date: 2017-04-20 10:05:41
categories:
  - nodejs
tags:
  - nodejs
  - 学习笔记
---

node的一个最大特性就是单线程，单线程带来的好处是不用像多线程编程那样去考虑状态的同步问题，也不用去担心出现死锁，也没有线程上下文所带来的性能的开销。但是同时也带来了一些问题，比如无法充分利用的多核CPU，线程会阻塞的问题。

但是node真的就不能更高效了吗，当然是不会的，如前几篇笔记所说，node对于“多进程”的处理有自己的一套解决方案，今天就来简单了解下。


## **服务模型的演变**

在了解node的解决方法之前吗，需要先了解一下Web服务器关于客户端请求的处理方案的一个演变过程，大概如下：

<span class="hljs-comment">同步</span> <span class="hljs-literal">-</span><span class="hljs-literal">-</span>&gt; <span class="hljs-comment">复制进程</span> <span class="hljs-literal">-</span><span class="hljs-literal">-</span>&gt; <span class="hljs-comment">多线程</span> <span class="hljs-literal">-</span><span class="hljs-literal">-</span>&gt; <span class="hljs-comment">事件驱动</span>

一开始，是属于同步的情况，同步的情况，一次只为一个请求服务。到后来，出现了改进，那就是复制进程，通过进程的复制可以服务更多的请求，但是这里的问题是每一个请求都需要一个进程来服务，性能上比较浪费。再后来是多线程，多核CPU的出现，创建多个线程来处理请求，但是这个方案的问题是，在切换现成的同时也需要切换线程的上下文，当线程的数量过多，时间就会被耗费到上下文的切换上。最后的一个就是事件驱动的方案，node和 nginx都是基于事件驱动的方式实现的，采用单线程避免了不必要的内存开销和上下文切换开销。


## **node的多进程架构**

node的多进程架构采用了child_process的方式，分为**主进程和工作进程**，主进程不负责具体的业务逻辑，只负责调度和管理工作进程。工作进程负责具体的业务逻辑。

如下演示如何创建一个子进程，以及一些操作

<pre class="prettyprint"><span class="hljs-keyword">var</span> child_process = <span class="hljs-built_in">require</span>(<span class="hljs-string">'child_process'</span>) 
<span class="hljs-comment">//启动一个子进程</span>
child_process.spawn(<span class="hljs-string">'node'</span>, [<span class="hljs-string">'test.js'</span>])
<span class="hljs-comment">//启动一个子进程，并执行命令</span>
child_process.exec(<span class="hljs-string">'node test.js'</span>, <span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-params">(err, stdout, stderr)</span> {</span>
    <span class="hljs-comment">//回调逻辑</span>
})
<span class="hljs-comment">//启动一个子进程，并执行可执行文件</span>
child_process.execFile(<span class="hljs-string">'test.js'</span>, <span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-params">(err, stdout, stderr)</span> {</span>
    <span class="hljs-comment">//回调的逻辑</span>
})
<span class="hljs-comment">//与spawn类似，启动一个子进程，不同的是它创建的进程只需要执行特定的文件模块即可，不参与其他的</span>
child_process.fork(<span class="hljs-string">'./test.js'</span>)</pre>


**进程之间的通信**

创建了子进程之后，主进程与子进程之间的通信也是个大问题，进程间通信的目的是为了让不同的进程能够互相访问资源并进行协调工作。node是这样的处理的。

node通过创建一个管道来解决。父进程在创建子进程之前，会创建管道（IPC通道）来监听，然后才会创建子进程，并且此时子进程可以通过环境变量得到这个管道的文件描述符。子进程在启动的过程中，根据文件描述符去连接这个已存在的IPC通道，从而完成父子进程之间的连接。当然在实际的通信的过程中还会采用句柄传递的方式，说的简单一点就是对一个资源的特殊标识，作用是可以实现多个子进程采用一个句柄来进行通信。


**充分利用多核CPU，node集群**

首先看一下集群的概念，第一眼看到这个名词的时候，有点蒙。

集群：
      在百度百科的解释里，集群（cluster）技术是一种较新的技术，通过集群技术，可以在付出较低成本的情况下获得在性能、可靠性、灵活性方面的相对较高的收益，其任务调度则是集群系统中的核心技术。集群的目的就是提高性能、降低成本、提高可扩展性、增强可靠性。

用我的理解就是，集群就是指将很多服务器集中起来，一起进行同一种服务,但是对客户端来说，在服务端感觉就是一个的存在。

关于集群，了解不深，只说两个概念，**负载均衡和状态共享**

*   **负载均衡**：服务器也有负载均衡，但对于node所讨论的，意思就是在多核CPU环境下，始终保证每个CPU都能被使用到从而保证最大效率。node采用的策略是轮叫调度，由主进程接手连接任务，然后依次分发给工作的子进程。
*   **状态共享**： 也是有两种情况，一种是要第三方存储，利用Redis来实现，一种是主动通知，其实也需要通过轮询来解决。


**一个杀器，cluster模块**

cluster是一个nodejs内置的模块，用于nodejs多核处理。有了这个东西，就基本不用上面的介绍的子进程child_process了。cluster模块可以帮助我们简化多进程并行化程序的开发难度，轻松构建一个用于负载均衡的集群。

下面看一下官方的示例：

<pre class="prettyprint"><span class="hljs-keyword">var</span> cluster = <span class="hljs-built_in">require</span>(<span class="hljs-string">'cluster'</span>)
<span class="hljs-keyword">var</span> http = <span class="hljs-built_in">require</span>(<span class="hljs-string">'http'</span>)
<span class="hljs-keyword">var</span> numCPUs = <span class="hljs-built_in">require</span>(<span class="hljs-string">'os'</span>).cpus().length  <span class="hljs-comment">//cpu的核心数</span>
<span class="hljs-keyword">if</span> (cluster.isMaster) {
<span class="hljs-comment">//创建多个子进程</span>
<span class="hljs-keyword">for</span> (<span class="hljs-keyword">var</span> i = <span class="hljs-number">0</span>; i &lt; numCPUs; i++) {
    cluster.fork();
}
cluster.on(<span class="hljs-string">'exit'</span>, <span class="hljs-function"><span class="hljs-keyword">function</span><span class="hljs-params">(worker, code, signal)</span> {</span>
    console.log(<span class="hljs-string">'worker'</span> + worker.process.id + <span class="hljs-string">'died'</span>)
})
} <span class="hljs-keyword">else</span> {
http.createServer(<span class="hljs-function"><span class="hljs-keyword">function</span><span class="hljs-params">(req, res)</span> {</span>
    res.writeHead(<span class="hljs-number">200</span>)
    res.end(<span class="hljs-string">'hello world'</span>)
}).listen(<span class="hljs-number">1234</span>)
}</pre>

顺便解释一下cluster的工作原理：

cluster模块实际上是**chlid_process和net模块**的组合应用。cluster启动时，会在内部启动一个TCP服务器，在cluster创建一个子进程（fork）时，将这个TCP服务器端socket的文件描述符发送给工作进程。如果进程是复制出来的，并且存在网络端口的调用，那么它就会拿到该文件描述符，并重用，从而实现多个子进程共享端口。

有关cluster详细的代码实践，可以参考这篇博客：[https://cnodejs.org/topic/56e84480833b7c8a0492e20c](https://cnodejs.org/topic/56e84480833b7c8a0492e20c)

* * *

<br/>前端新手，弱鸡一枚，如有错误，请指正，谢谢！
