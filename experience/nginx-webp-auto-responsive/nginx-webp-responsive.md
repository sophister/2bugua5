# nginx webp 响应式


说是“响应式”，其实就是针对同一个图片URL，根据客户端是否支持 `webp` 图片，返回`webp`版本或者原图，
实在不知道怎么起个名字(Naming is the most difficult thing in programming)，就跟潮流用个所谓的
“响应式”吧……


## 现状分析

目前H5上使用了非常多的图片，因为很多页面都是运营、产品直接通过内部的 `page builder`系统生成，
大量的2x图片，导致页面渲染很慢。

虽然我们静态资源都会缓存在CDN上，但是手机里看H5，还是发现很慢，貌似并没有缓存住，每次都发了请求。


## 解决方案

最终效果：希望能做到 **同一个图片URL，针对不同终端返回不同的图片**，即具体业务开发时，对图片URL
不需要做任何修改，在支持 `webp` 的终端，返回 `webp` 版本；其他情况返回原图。

如果不修改图片URL，那怎么解决被CDN缓存的问题呢？在下面腾讯云的这篇文章里，看到了 `Vary` 这个
`http header`，可以用来实现，同一个URL在CDN上根据不同的 `http request header` 来缓存不同
的内容，具体可以看文末的 `MDN` 文档。

解决了CDN缓存问题，现在面临 `webp` 图片怎么生成的问题了，主要这两个办法：

* 在CMS上传图片时，当时就生成对应的 `webp` 版本
* 在 `nginx` 上，收到图片请求时，第一次动态生成 `webp`，以后就读取生成的图片

第1个方案，实现上要简单些，并且操作是离线的，不会影响线上业务；但也有缺点，线上已经存在的大量图片，
需要想办法来产出`webp`版本。

第2个方案，表面看是在线实时生成，其实也可以认为是离线的。因为我们在页面里使用图片，都是先上传CMS，
然后会在各个环境里访问，这时候，`nginx`上已经生成了`webp`，等到线上普通用户访问的时候，其实是不会
动态生成的。当然，对于那些已经存在的图片，还是会在第一次访问的时候，动态去生成`webp`版本。

看了网上一些文章，大多是在 `nginx` 上直接用 `lua`脚本来生成的，看来这个方案在生产环境还是比较稳定了。
就采用方案2了。

采用方案2，其实还有个问题，我们知道，现在前端的图片资源，一般都是 **强缓存(设置很大的 max-age)** 的，
保证图片会在浏览器、CDN上都缓存很长时间，但有的时候，我们在源站修改了图片，需要强制终端刷新，如果
不能修改图片的文件名，那一般就是在图片的 `URL` 上加个 `query` 参数，来强制终端和CDN刷新。但是
上这个 `webp` 的响应式方案后，如果回源到服务器的图片请求，**含有query参数** ，怎么办呢？我们
当然 **不能** 每次都重新生成 `webp`版本，毕竟这个还是要消耗服务器资源的。大概方法还是有2个：

* 简单做，针对包含了 `query` 参数的图片请求，直接返回原图，不尝试`webp`版本
* 完美点，不同 `query`的图片对应不同的 `webp`版本，这就需要在生成 `webp`时，把 `query` 也作为
文件名的一部分来存储。

昨天本来想简单搞的，现在看来，还是先尝试完美一点的方案吧~


## 撸码过程

### `nginx lua支持`

嗯，第一步就卡住了，发现我们目前的 `nginx` 没有支持 `lua`脚本，还得找OP同学帮忙重新编译 `nginx`了。

### `CentOS`上安装 `libwebp`

`CentOS`上安装依赖包：

```shell
yum install libjpeg libjpeg-devel
yum install libpng libpng-devel
yum install giflib giflib-devel
yum install libtiff libtiff-devel
# yum install freetype freetype-devel # 官网上没有安装这个
```

完成上面的依赖安装，再下载 `libwebp-0.6.0.tar.gz` ，从源码安装：

```shell
tar xvzf libwebp-0.6.0.tar.gz
cd libwebp-0.6.0
./configure
make
sudo make install
```

根据官方文档，安装完了之后，提到一个环境变量 `LD_LIBRARY_PATH`，说是需要把 `/usr/local/lib/`
加入到这个环境变量里：

> The library will usually be installed under the /usr/local/lib/ directory.
To avoid run-time errors, make sure that your LD_LIBRARY_PATH environment variable includes this location

这个在正式环境安装的时候，需要注意下。

### `lua`入门

`lua`官方网站，貌似没有详细的学习文档！！


* [lua官方文档](https://www.lua.org/start.html#installing)
* [LUA简明教程](http://coolshell.cn/articles/10739.html)
* [lua菜鸟入门教程](http://www.runoob.com/lua/lua-tutorial.html)

## 相关链接

* [腾讯云:基于 CDN 的 sharpP 自适应图片技术实践](https://www.qcloud.com/community/article/164816001481011868)
* [webp前端图片适配流量优化](https://github.com/ShowJoy-com/showjoy-blog/issues/10)
* [webp-automagically](https://www.maxcdn.com/blog/how-to-reduce-image-size-with-webp-automagically/)
* [webp 官方文档](https://developers.google.com/speed/webp/)
* [http Vary header](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Vary)
* [Tengine + Lua + GraphicsMagick 实现图片自动裁剪/缩放](https://my.oschina.net/eduosi/blog/169606?utm_source=tuicool&utm_medium=referral)
