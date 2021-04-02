## 性能监控指标介绍

下图一、图二是 Navigation Timing API 第一版和第二版，该 API 提供了可用于衡量一个网站性能的数据。记录了页面重定向、DNS 查询、TCP、SSL 连接、内容请求、DOM 解析等的时间点。

![Navigation Timing Level 1](./v1.png)

![Navigation Timing Level 2](./v2.svg)

## 指标解读

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

## 阶段耗时

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
