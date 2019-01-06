# nodejs stream (1) : 从 `req` 说起

`stream`作为nodejs里非常重要的一个概念，其实很早就应该好好学习下了，偶尔也会看到一些文章有介绍的，但是大部分文章，都是看过之后，也就忘了。正因为这样的原因(主要还是懒吧)，一直没有好好的学习下stream，正好，趁着这次遇到的一个问题，是时候学习一下了。


## node服务响应超时

有天OP突然找到我，说线上`nginx`的error日志里，大部分的错误，都来自一个微信的回调接口响应超时。根据OP提供的日志，找到node服务上的接口，发现是微信异步通知我们的回调接口。大概接口实现如下：

```javascript
//对外提供的：微信服务回调入口
async wx_callback(req, res){
    const method = req.method;

    //微信服务器签名验证，确认请求来自微信
    if(method === 'GET'){
        // 省略
    }else if(method === 'POST'){
        let buf = '';
        const parseString = xmllib.parse;
        req.setEncoding('utf8');
        req.on('data', function (chunk) {
            buf += chunk;
        });

        req.on('end', function(){
            parseString(buf, function (error, json) {
                if(error){
                    res.json({
                        status: 1,
                        message: "fail to parse xml body",
                    });
                }else{
                    // 其他处理
                }
            });
        });
    }
}
```

看这段代码，逻辑其实挺清晰的，微信回调都是 `POST` 请求，因此在 `POST` 请求里，主动接受请求的数据，接收完之后，调用第三方的 `xml` 解析库，将请求的 `xml` 解析成 `JSON` 格式(BTW，实在不明白微信回调，为什么要用xml，不用JSON呢)。

根据之前node代码里发现的问题，node接口响应超时，很大概率是 **接口执行完了，但是没有给client返回数据，也就没有结束掉本次http请求**，通常有2种情况下，会发生这种问题：

1. 代码逻辑有问题，进入到了一些异常分支，没有输出响应。比如很多时候，在 `try-catch`里，只打了error日志，没有输出响应
2. 一些 `async` 的调用，没有合理的 `catch` 异常分支，导致执行到异常分支里，没有任何处理，在 `process` 上触发全局error事件

根据这两点，大概来对照上面的代码，没能发现什么问题。监听 `req` 的流事件，读取结束后，整体进行解析，并且也有响应的错误处理。鬼了还？只能在各个关键节点，打日志进行查找，最后发现，`req`的 `end` 事件，根本就没有触发！what？`req`不是一个stream么，为什么end事件没有触发呢？突然想起，在请求进入我们这个 `wx_callback`之前，我们默认添加了其他的middleware，比如涉及到请求body解析的：

```javascript
app.use(bodyParser.json({ })); // for parsing application/json
app.use(bodyParser.urlencoded({ })); // for parsing application/x-www-form-urlencoded
```

再看线上相关公众号的操作，功能都是正常的，如果代码写的有bug，那测试应该都不能过(当然也不一定……)，更别说，目前的情况下，线上功能都是OK的，说明是代码在某些异常条件下，会触发问题。再回到我们的代码，发现 `req`的 `end` 事件没触发，会不会在执行到我们的 `wx_callback`之前，`req`的 `end`事件已经触发过了呢？再看`bodyParser`这里，会解析两种请求的 `content-type`，那么，如果请求的 `content-type`是 `applicatin/json` 或者 `application/x-www-form-urlencoded`呢？经过本地测试，发现这两种请求下，确实有问题，我们`wx_callback`里的 `req end`事件没有触发！而如果用 `text/xml` 的请求类型，接口就能正常处理了！看来就是和这两个全局的middleware有关系了。果然，在 `bodyParser` 的源码里，也有监听 `req`的各种流事件：

```javascript
// attach listeners
  stream.on('aborted', onAborted)
  stream.on('close', cleanup)
  stream.on('data', onData)
  stream.on('end', onEnd)
  stream.on('error', onEnd)
```

如果请求的 `content-type` 命中 `applicatin/json` 或者 `application/x-www-form-urlencoded` 或者，就会先被 `bodyParser` 处理，在这里，`req`流已经触发了`end`事件，导致在 `wx_callback`里，没有再次触发。这里在 `wx_callback` 里监听 `end` 事件，没有再次触发，可能有点疑惑。其实，`stream`都是 `EventEmitter`的实例，各种事件绑定的方法，都是来自`EventEmitter`，根据nodejs源代码里 [EventEmitter.prototype.on](https://github.com/nodejs/node/blob/master/lib/events.js#L265) 这里的实现，可以看到，只是把回调函数放到了内部该事件的回调数组里。这样就能理解，`end`事件触发后，第二次监听不再触发的原因了。



         ----- 时2019年1月6日12:13竣工于帝都望京东
