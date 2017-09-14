# https证书链校验失败


好容易线上运行稳定了连个月了。

今天上午突然说 `https` 证书快到期了，已经申请了新的证书，必须要今天更换。

中午OP直接在线上更换了，然后，Android APP接口全线挂掉，iOS却没问题……

原来Android上，在APP内置了证书进行校验，现在更换了服务器证书，导致Android校验失败；果然是做的越多，
越容易出事。

除了native接口挂掉(native请求的是 `api.we.com` 域名)，`webview`请求的 `m.we.com` 页面也挂掉了，显示白屏，然而问了Android开发同学，并没有对
`webview` 的 `https` 证书校验进行处理。


## 复现问题

在PC浏览器里访问某个移动端页面(m.we.com)，chrome和Safari正常显示，未提示异常；Firefox 提示证书错误
`SEC_ERROR_UNKNOWN_ISSUER`，具体信息如下：

>由于发布者未知，所以证书不可信。
服务器可能未发送合适的中间证书。
或许需要额外的根证书。

在本地Android 6模拟器里，直接用`webview`加载上述的URL，复现了APP里的问题，白屏，并且有如下报错信息:

```
Failed to validate the certificate chain, error: java.security.cert.CertPathValidatorException: Trust anchor for certification path not found
```

看来 `m.we.com` 的证书确实有问题！


## 定位问题

通过Google上述的错误关键字，找到了这两篇文章，很有帮助:
[1)来自stackoverflow](https://stackoverflow.com/a/16302527/2583885);
[2)中文版本](https://gxnotes.com/article/49960.html)

按照 `stackoverflow` 里说的，通过 `openssl` 进行诊断，在命令行执行如下：

```shell
openssl s_client -debug -connect m.we.com:443
```

显示异常信息：

```
---
Certificate chain
 0 s:/businessCategory=Private Organization/serialNumber=911101080573384678/1.3.6.1.4.1.311.60.2.1.3=CN/1.3.6.1.4.1.311.60.2.1.2=Beijing/C=CN/ST=\xE5\x8C\x97\xE4\xBA\xAC/L=\xE5\x8C\x97\xE4\xBA\xAC/OU=\xE8\xBF\x90\xE7\xBB\xB4\xE9\x83\xA8/O=\xE5\x8C\x97\xE4\xBA\xAC\xE5\xBE\xAE\xE8\xB4\xA2\xE7\xA7\x91\xE6\x8A\x80\xE6\x9C\x89\xE9\x99\x90\xE5\x85\xAC\xE5\x8F\xB8/CN=m.we.com
   i:/C=BE/O=GlobalSign nv-sa/CN=GlobalSign Extended Validation CA - SHA256 - G3
---
```

应该是 `https证书链` 有问题，并没有一直指向某个 `根证书`


## 修复问题

和OP同学确认了，确实`https证书`有问题，重新更换了证书之后，再次执行：

```shell
openssl s_client -debug -connect m.we.com:443
```

这次输出的 `https证书链` 信息如下：

```
---
Certificate chain
 0 s:/businessCategory=Private Organization/serialNumber=911101080573384678/1.3.6.1.4.1.311.60.2.1.3=CN/1.3.6.1.4.1.311.60.2.1.2=Beijing/C=CN/ST=\xE5\x8C\x97\xE4\xBA\xAC/L=\xE5\x8C\x97\xE4\xBA\xAC/OU=\xE8\xBF\x90\xE7\xBB\xB4\xE9\x83\xA8/O=\xE5\x8C\x97\xE4\xBA\xAC\xE5\xBE\xAE\xE8\xB4\xA2\xE7\xA7\x91\xE6\x8A\x80\xE6\x9C\x89\xE9\x99\x90\xE5\x85\xAC\xE5\x8F\xB8/CN=api.we.com
   i:/C=BE/O=GlobalSign nv-sa/CN=GlobalSign Extended Validation CA - SHA256 - G3
 1 s:/C=BE/O=GlobalSign nv-sa/CN=GlobalSign Extended Validation CA - SHA256 - G3
   i:/OU=GlobalSign Root CA - R3/O=GlobalSign/CN=GlobalSign
---
```

可以看出，这次证书链最终指向到了 `GlobalSign` 这个根CA。

刷新Android webview，页面正常显示，OK了。


## 相关链接

* [Firefox官方关于证书错误文档](https://support.mozilla.org/zh-CN/kb/%E9%94%99%E8%AF%AF%E7%A0%81sec_error_unknown_issuer%E7%9A%84%E6%95%85%E9%9A%9C%E6%8E%92%E9%99%A4?as=u&utm_source=inproduct)
* [https原理讲解](https://juejin.im/entry/5681e5cd00b01b9f2bd6d9e8)
* [stackoverflow 使用openssl定位证书问题](https://stackoverflow.com/a/16302527/2583885)



时20170914 18：47 竣工于帝都五道口清华科技园
