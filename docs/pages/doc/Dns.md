## 关于 DNS 解析的 TTL 参数：

我们在配置 DNS 解析的时候，有一个参数常常容易忽略，就是 DNS 解析的 TTL 参数，Time To Live。TTL 这个参数告诉本地 DNS 服务器，域名缓存的最长时间。用阿里云解析来举例，阿里云解析默认的 TTL 是 10 分钟，10 分钟的含义是，本地 DNS 服务器对于域名的缓存时间是 10 分钟，10 分钟之后，本地 DNS 服务器就会删除这条记录，删除之后，如果有用户访问这个域名，就要重复一遍上述复杂的流程。

其实，如果网站已经进入稳定发展的状态，不会轻易更换 IP 地址，我们完全可以将 TTL 设置到协议最大值，即 24 小时。带来的好处是，让域名解析记录能够更长时间的存放在本地 DNS 服务器中，以加快所有用户的访问。设置成 24 小时，其实，还解决了 Googlebot 在全球部署的服务器抓取网站可能带来的问题，这个问题麦新杰专门有一篇博文，请参考：http://www.maixj.net/wlyx/googlebot-8890

阿里云之所以只将 TTL 设置成 10 分钟，是为了让域名解析更快生效而已。因为之前的解析会在最长 10 分钟之后失效（本地 DNS 服务器将对应的解析条目删除），然后新的解析生效。如果是 24 小时，这个生效的时间最长就是 24 小时，甚至更长（本地 DNS 服务器要有用户请求，才会发起查询）。

https://www.cnblogs.com/crazylqy/p/7110357.html

https://developer.mozilla.org/zh-CN/docs/Web/Performance/%E6%B5%8F%E8%A7%88%E5%99%A8%E6%B8%B2%E6%9F%93%E9%A1%B5%E9%9D%A2%E7%9A%84%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86