# 微信小程序和web-view登录态共享

目前，很多(中国)公司的C端(甚至B端)业务，为了更好的用户体验(薅微信流量)，都会开发微信小程序版本，甚至有的公司只有微信小程序，没有自己的APP了。

另外一方面，为了用户留存等原因，公司也会开通响应的服务号，一部分业务逻辑采用H5方式开发。

这就存在一个问题，有的业务模块已经用H5开发完成了，不希望单独再开发小程序版本，所以会采用小程序的 `web-view` 组件，将对应H5页面嵌套在小程序里。







## 相关资料



* [微信网页开发-网页授权](https://developers.weixin.qq.com/doc/offiaccount/OA_Web_Apps/Wechat_webpage_authorization.html) 
* 