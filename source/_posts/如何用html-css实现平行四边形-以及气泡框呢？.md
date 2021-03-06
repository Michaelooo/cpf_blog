---
title: '[原]如何用 html+css 实现平行四边形，以及气泡框呢？'
date: 2016-08-02 10:55:39
categories:
  - 前端
tags:
  - css
---

**【效果图】** 

首先说一下平行四边形，想要做成的效果大致是下面这个样子的 

![这里写图片描述](http://img.blog.csdn.net/20160801103810284)

**【思路】** 

如果考虑用昨天的方法，利用 border 边界值，就可以分解成 一个右三角+矩形+上三角（这里右，上的意思指的是需要加上颜色显示的边界颜色），但这样就挺复杂了，所以要换一种方法来做

**【做法】** 

其实利用上次说的 css3 的属性 transform 属性来设置

<pre class="prettyprint"><span class="hljs-rules">{
    <span class="hljs-rule"><span class="hljs-attribute">display</span>:<span class="hljs-value"> inline-block</span></span>;
    <span class="hljs-rule"><span class="hljs-attribute">padding</span>:<span class="hljs-value"> <span class="hljs-number">5</span>px <span class="hljs-number">20</span>px</span></span>;
    <span class="hljs-rule"><span class="hljs-attribute">border</span>:<span class="hljs-value"> <span class="hljs-number">1</span>px solid <span class="hljs-hexcolor">#44a5fc</span></span></span>;
    <span class="hljs-rule"><span class="hljs-attribute">color</span>:<span class="hljs-value"> <span class="hljs-hexcolor">#333</span></span></span>;
    <span class="hljs-rule"><span class="hljs-attribute">transform</span>:<span class="hljs-value"> <span class="hljs-function">skew(-<span class="hljs-number">20</span>deg)</span></span></span>; <span class="hljs-comment">/* 定义沿着 X 和 Y 轴的 2D 倾斜转换*/</span>
<span class="hljs-rule">}</span></span></pre>

**【注意】** 

但是这么做，有个注意的地方就是，如果当前盒子里包裹其他内容，这个其他内容也会跟着旋转，所以为了避免这种效果，需要在内部重新加一个盒子，并对这个盒子进行一个逆向的 transform，这样就实现了平行四边形了，也就是一个属性的事情  

**【思考】** 

要是个梯形，该怎么做呢？


接下来再说一下怎么制作一个类似 tooltip 的气泡提示框呢？ 

**【效果图】** 

先看一下大致的效果 

![这里写图片描述](http://img.blog.csdn.net/20160801104015487) 

**【思路】** 

其实做起来也相当的简单，就是用一个盒子加一个三角形就行，然后控制好定位的问题就可以做到

**【做法】** 

在有一个叫做 rectangle 的盒子，然后在这个盒子里面有一个 trangle 的盒子

<pre class="prettyprint"><span class="hljs-class">.rectangle</span><span class="hljs-rules">{
    <span class="hljs-rule"><span class="hljs-attribute">position</span>:<span class="hljs-value">relative</span></span>;
    <span class="hljs-rule"><span class="hljs-attribute">width</span>:<span class="hljs-value"><span class="hljs-number">150</span>px</span></span>;
    <span class="hljs-rule"><span class="hljs-attribute">height</span>:<span class="hljs-value"><span class="hljs-number">35</span>px</span></span>;
    <span class="hljs-rule"><span class="hljs-attribute">background</span>:<span class="hljs-value">green</span></span>;
    <span class="hljs-rule"><span class="hljs-attribute">border-radius</span>:<span class="hljs-value"><span class="hljs-number">5</span>px</span></span>;
    <span class="hljs-rule"><span class="hljs-attribute">margin</span>:<span class="hljs-value"><span class="hljs-number">30</span>px auto <span class="hljs-number">0</span></span></span>;
    <span class="hljs-rule"><span class="hljs-attribute">padding</span>:<span class="hljs-value"> <span class="hljs-number">10</span>px</span></span>;
    <span class="hljs-rule"><span class="hljs-attribute">color</span>:<span class="hljs-value">white</span></span>; 
<span class="hljs-rule">}</span></span>
<span class="hljs-class">.rectangle</span> <span class="hljs-class">.trangle</span><span class="hljs-rules">{
    <span class="hljs-rule"><span class="hljs-attribute">position</span>:<span class="hljs-value">absolute</span></span>;
    <span class="hljs-rule"><span class="hljs-attribute">top</span>:<span class="hljs-value"><span class="hljs-number">12</span>px</span></span>;
    <span class="hljs-rule"><span class="hljs-attribute">right</span>:<span class="hljs-value">-<span class="hljs-number">16</span>px</span></span>;
    <span class="hljs-rule"><span class="hljs-attribute">width</span>:<span class="hljs-value"><span class="hljs-number">0</span></span></span>;
    <span class="hljs-rule"><span class="hljs-attribute">height</span>:<span class="hljs-value"><span class="hljs-number">0</span></span></span>;
    <span class="hljs-rule"><span class="hljs-attribute">font-size</span>:<span class="hljs-value"><span class="hljs-number">0</span></span></span>;
    <span class="hljs-rule"><span class="hljs-attribute">border</span>:<span class="hljs-value">solid <span class="hljs-number">8</span>px</span></span>;
    <span class="hljs-rule"><span class="hljs-attribute">border-color</span>:<span class="hljs-value">transparent transparent transparent green</span></span>;
<span class="hljs-rule">}</span></span>


然后差不多就可以实现了 

**【注意】** 

有一个注意的地方就是 position 的使用，在外部盒子的是一定要使用相对定位的 

**【思考】** 

需要做一个不规则的小三角呢？（其实应该是类似的，就是用不同类型的三角形层层的遮罩，或者利用 css3 的2D变换属性来实现）