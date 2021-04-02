---
title: 前端工程师需要知道的性能知识
date: 2021-02-02
tags:
  - 端监控
author: Y的三次方
location: Shanghai  
---

我们都知道网站性能很重要。那在说到网站性能、网站速度很快的时候，我们具体指的是什么呢？

首先我们要知道，网站打开速度是一个相对的概念。同一个网站，在高端机上或者网络优良的情况下，可以是很快的，在低端机，网络差的情况却很慢。

下图是一个网站的打开时间分布图，X 轴是耗时区间，Y 轴是用户量。首先从这个图中，我们可以知道，网站打开速度并不是一个固定的数值，而且一个区分分布。虽然大部分用户打开时间在 2s 内，但其实还是存在少量用户，他们的打开时间大于 10 秒。

![FMP区间分布图](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/59a10cc38f05434a814cb45466fa8b18~tplv-k3u1fbpfcp-watermark.image)

还有一些其他的概念：

- 两个网站加载时间相同，渐进式渲染的网站比加载完渲染的网站，感觉上会更快。
- 网站加载速度很快，交互响应很慢，我们也会觉得网站性能差。

## 定义指标

那么我们该如果衡量网站性能呢，站在用户的角度上想，我们需要回答一下几个问题？

|                |                                            |
| -------------- | ------------------------------------------ |
| 是否发生？     | 导航发生了吗？服务器响应了吗？             |
| 是否有用？     | 页面是否渲染出足够可用内容                 |
| 是否可用？     | 用户可以与该页面进行交互，还是仍在加载中？ |
| 是否令人愉快？ | 交互是否顺畅自然，没有滞后和卡顿？         |

## 如何度量

先用一个直观的图片来展示，页面显示的各个阶段。

![Page Painting](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d9a99363efbe4e96aa0d889e51ae0754~tplv-k3u1fbpfcp-watermark.image)

#### 是否发生 FP 、 FCP

FP (First Paint) 首次绘制，代表浏览器第一次像屏幕传输像素的时间，也就是页面在屏幕上首次发生视觉变化的时间。

FCP (First Contentful Paint) 首次内容绘制，代表浏览器第一次绘制内容的时间。

**如何计算**

```js
function showPaintTimings() {
  if (window.performance) {
    let performance = window.performance;
    let performanceEntries = performance.getEntriesByType('paint');
    performanceEntries.forEach((performanceEntry, i, entries) => {
      console.log(
        'The time to ' +
          performanceEntry.name +
          ' was ' +
          performanceEntry.startTime +
          ' milliseconds.'
      );
    });
  } else {
    console.log("Performance timing isn't supported.");
  }
}

showPaintTimings();
// The time to first-paint was 949.5900000038091 milliseconds.
// The time to first-contentful-paint was 949.5900000038091 milliseconds.
```

> FP 不包含默认背景绘制，但是包含非默认背景绘制。

> 只有绘制文本、图片(包含背景图)、非白色的 svg 或者 canvas 是才被算作 FCP


### 是否有用 FMP

FMP (First Meaningful Paint) 首次有效绘制，是页面主要内容绘制的时间点。

#### 如何计算 FMP 指标

在打开一个网页的时候，随着网页的加载与解析，浏览器会将布局对象（Layout Object）逐步添加到布局树（Layout Tree）上进行布局。

![example1](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/db31f67d204c424381560eeb7ab67d15~tplv-k3u1fbpfcp-watermark.image)

![example2](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1956d63124b34a19b9da3b0992bba1a4~tplv-k3u1fbpfcp-watermark.image)

- 图 1 展示了当加载谷歌搜索结果页面时，被逐步加载到布局树中的布局对象的数量。
- 图 2 展示了加载「谷歌搜索结果页」在加载和渲染过程中的可视化过程

**将两张图结合起来解读**

1. 1.577s，页面头部渲染，”布局对象“总数是 60 个。
2. 1.76s，页面头部渲染完成，“布局对象”总数是 103 个。
3. 1.907s, 搜索结果数据返回并渲染，“布局对象”总数是 261。而此时页面主体内容已经绘制完成，从用户体验的角度看，此时的时间点就是是 FMP。
4. 2.425s, 其他搜索结果和页面底部的布局对象继续被添加到布局树中并进行绘制，页面完成最终加载和渲染。

从以上对于「谷歌搜索结果页」加载过程的例子中可以发现，布局对象的数量与页面完成度高度相关。我们得出以下结论：

```
FMP = 页面在加载和渲染过程中最大布局变动之后的那个绘制时间
```

#### 代码实现

基于刚刚得出的结论：FMP 的时间点为 DOM 结构变化最剧烈的时间点。DOM 结构变化的时间点可以通过[ MutationObserver API ](https://developer.mozilla.org/zh-CN/docs/Web/API/MutationObserver)来获得。

```js
// 用于存放每次 dom 变话时，时间和 dom 数量
const list = [];
// 起止时间
const startTime = Date.now();
// 创建监听
const observer = new MutationObserver(callback);
// 监听 dom 变更
observer.observe(document, {
  childList: true,
  subtree: true,
});

// dom 变更回调
function callback() {
  const duration = Date.now() - startTime;
  const body = document.querySelector('body');
  list.push({
      number: body ? count(body) : 0,
      duration,
    });
}

// 计算 dom 数量
function count(element) {
  let number = 0;
  const childrenLength = element.children ? element.children.length : 0;
  if (childrenLength > 0) {
    const children = element.children;
    for (let length = childrenLength - 1; length >= 0; length--) {
      number += count(children[length]);
    }
  }
  number += 1;
  return number;
}

// 找出变化做大的时间点, 计做 FMP
function getFmp() {
  let result;
  for (let i = 1; i < list.length; i++) {
    const diff = list[i].number - list[i - 1].number;
    if (!result.diff || diff > result.diff) {
      result = {
        duration: list[i].duration,
        diff,
      };
    }
  }
  return result?.duration || 0;
}
```

#### 优化实现

当前方法可以在大部分情况下得出 FMP 值，但是在一些其他场景下，还是存在偏差。

![example3](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b8dd1d96c503409e827160bb6b0795c7~tplv-k3u1fbpfcp-watermark.image)

![example4](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c18bab1322df4bfb844b2e751a63c431~tplv-k3u1fbpfcp-watermark.image)

图 3 是微博页面在加载和渲染的可视化过程

图 4 展示了当加载微博页面时，被逐步加载到布局树中的布局对象的数量。

通过图 3 图 4 可以看出，主要元素加载时间在 6.047 秒，但是布局对象发生变化最大的时候，是 24.25 秒。在 24.25 秒的时候，页面底部到可见区域外，大量元素被添加到布局树上。

为了优化上述现象，我们引入**布局意义概念**。

```
布局意义 = 添加的布局对象数量 / max(1, 页面高度 / 屏幕高度)
```

#### 代码优化

```js
// 计算页面比率
function getRatio() {
  const clinetH = document.body.clientHeight;
  const screenH = window.screen.availHeight;
  return Math.max(clinetH / screenH, 1);
}

// 更新 callback 方法
function callback() {
  const duration = Date.now() - startTime;
  const body = document.querySelector('body');
  list.push({
      number: body ? count(body) / getRatio(): 0,
      duration,
    });
}
```

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/77c4e7b4cef843a69144d2aee6d2973b~tplv-k3u1fbpfcp-watermark.image)

图 5 展示了当加载微博页面时，布局意义变化情况。布局意义最大变化发生在 5.89 秒，在 FMP(6.047)秒之前，符合预期。



### 是否可用 TTI

TTI (Time to Interactive) 可交互时间，代表网页第一次达到可交互的时间点。首页是页面的 ui 是可交互的状态，并且无场长任务运行，即页面是流畅的。

下图演示了如何查找 TTI。

![TTI](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4fda6a29ace348d0abc1f503834bed61~tplv-k3u1fbpfcp-watermark.image)

- 从 FCP 开始，向前搜索一个 5s 以上的静默窗口。静默窗口：无场任务，且网络请求不超过 2 个。
- 从静默窗口往后搜索，最后一个长任务的结束时间，如果搜索不到，则在 FCP 处停止。
- TTI 是静默窗口之前的最后一个长任务的结束时间（如果找不到长任务，则使用 FCP 的值作为 TTI）。

**如何计算**

谷歌团队开发了一个[polyfill](https://github.com/GoogleChromeLabs/tti-polyfill)，用于检测 TTI，适用于所有支持 Long Tasks API 的浏览器。

```js
import ttiPolyfill from './path/to/tti-polyfill.js';

ttiPolyfill.getFirstConsistentlyInteractive(opts).then((tti) => {
  // Use `tti` value in some way.
});
```

> [长任务(Long Tasks API)](https://developer.mozilla.org/zh-CN/docs/Web/API/Long_Tasks_API)：任何连续不间断的且主 UI 线程繁忙 50 毫秒及以上的时间区间。

### 是否令人愉快 长任务监听

页面”是否令人愉快“，主要有几个角度，动画是否流畅，用户交互是否可以快速影响。而卡顿、交互响应慢的情况通常由长任务导致的，了解长任务的发生频率，可以帮助我们判断页面是否流畅。

**如何计算**

```js
var observer = new PerformanceObserver(function(list) {
  var perfEntries = list.getEntries();
  for (var i = 0; i < perfEntries.length; i++) {
    // Process long task notifications:
    // report back for analytics and monitoring
    // ...
  }
});
// register observer for long task notifications
observer.observe({ entryTypes: ['longtask'] });
// Long script execution after this will result in queueing
// and receiving "longtask" entries in the observer.
```

## 其他性能指标

下图一、图二是 Navigation Timing API 第一版和第二版，该 API 提供了可用于衡量一个网站性能的数据。记录了页面重定向、DNS 查询、TCP、SSL 连接、内容请求、DOM 解析等的时间点。

![Navigation Timing Level 1](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1747b30c1735477ea02c1ce64ff34215~tplv-k3u1fbpfcp-watermark.image)

![Navigation Timing Level 2](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5d70a0ec1098411baaae6029abc77117~tplv-k3u1fbpfcp-watermark.image)

### 指标解读

| 指标                       | 说明                                                                                                          |
| -------------------------- | ------------------------------------------------------------------------------------------------------------- |
| startTime                  | 0                                                                                                             |
| unloadEventStart           | 上一个页面 unload 事件的触发时间，只有同源跳转可以记录，非同源返回 0                                          |
| unloadEventEnd             | 上一个页面 unload 事件的结束时间                                                                              |
| redirectStart              | 第一个 http 重定向请求发起时间，只有同源跳转可以记录，非同源返回 0                                            |
| redirectEnd                | 最后一个 http 重定向请求发起时间                                                                              |
| fetchStart                 | 请求开始时间，发生在检查本地缓存之前                                                                          |
| domainLookupStart          | 域名解析(DNS)开始时间，如果存在本地缓存，则该值等同于 fetchStart                                              |
| domainLookupEnd            | 域名解析(DNS)结束时间，如果存在本地缓存，则该值等同于 fetchStart                                              |
| connectStart               | 请求连接被发送到网络的时间                                                                                    |
| secureConnectionStart      | 安全连接握手开始的时间                                                                                        |
| connectEnd                 | 网络链接建立的时间                                                                                            |
| requestStart               | 浏览器发送文档请求的开始时间                                                                                  |
| responseStart              | 浏览器接收到响应的第一个字节的时间                                                                            |
| responseEnd                | 浏览器接收到响应的最后一个字节的时间                                                                          |
| domInteractive             | 主文档的解析器结束工作，Document.readyState 改变为 interactive 的时间，相当于 readystatechange 时间触发的时间 |
| domContentLoadedEventStart | 所有的需要被运行的脚本已经被解析，即 DOMContentLoaded 事件触发时间                                            |
| domContentLoadedEventEnd   | 所有的需要被运行的脚本已经执行完毕                                                                            |
| domComplete                | 主文档的解析器结束工作，Document.readyState 变为 complete                                                     |
| loadEventStart             | load 事件触发时间                                                                                             |
| loadEventEnd               | load 时间完成时间                                                                                             |

### 阶段指标

| 阶段                 | 计算方式                                  |
| -------------------- | ----------------------------------------- |
| DNS 查询             | domainLookupEnd - domainLookupStart       |
| TCP 连接             | connectEnd - connectStart                 |
| SSL 建连             | connectEnd - secureConnectionStart        |
| 首字节网络请求(TTFB) | responseStart - requestStart              |
| 内容传输             | responseEnd - responseStart               |
| DOM 解析             | domInteractive - responseEnd              |
| 白屏时间             | domInteractive - t.fetchStart             |
| 资源加载             | loadEventStart - domContentLoadedEventEnd |
| DOM Ready            | domContentLoadedEventEnd - fetchStart     |

**相关文章**

>[User-centric performance metrics](https://web.dev/user-centric-performance-metrics/)

> [First Contentful Paint](https://web.dev/first-contentful-paint/)  

>[Time to First Meaningful Paint](https://docs.google.com/document/d/1BR94tJdZLsin5poeet0XoTW60M0SjvOJQttKT-JK8HI/view?hl=zh-cn#heading=h.tdqghbi9ia5d)

> [Time to Interactive (TTI)](https://web.dev/tti/)

>[渲染页面：浏览器的工作原理](https://developer.mozilla.org/zh-CN/docs/Web/Performance/%E6%B5%8F%E8%A7%88%E5%99%A8%E6%B8%B2%E6%9F%93%E9%A1%B5%E9%9D%A2%E7%9A%84%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86)


