# 【译】WebSocket认证

## 前言

最近有一个在线客服系统的重构，正好实际使用下 `WebSocket` ，对这个技术也是经常听到但还没机会实际上手试试。

之前一些候选人的简历提到 `WebSocket`，都会问一下在这个场景下，怎么做 **用户认证**，从回答结果来看，思路很多同学都说不清楚。有的回答没有做过，有的回答浏览器自动完成的，有的回答带上了 `token`。

之前也大概了解过，浏览器里 `WebSocket` **不支持发送自定义header**，今天正好看看，通常可以有哪些方式来实现websocket里的用户认证。

**说明**：下面内容是来自于末尾的英文翻译，用到了Kimi这个工具。

下面是正文部分。

--------------

WebSocket协议被设计用于创建需要在浏览器中运行的客户端与服务器之间进行双向通信的Web应用程序。

在大多数实际用例中，WebSocket服务器需要对客户端进行身份验证，以便适当和安全地路由通信。

**RFC 6455**在认证方面保持了模糊：

> 该协议没有规定服务器可以在WebSocket握手期间以任何特定方式对客户端进行身份验证。WebSocket服务器可以使用任何通用HTTP服务器可用的客户端身份验证机制，例如cookies、HTTP认证或TLS认证。

## 系统设计
考虑一个WebSocket服务器与HTTP服务器分开的设置。

大多数使用websocket构建的服务器采用此设计来补充Web应用程序，因为websocket并不旨在支持HTTP。

下面的图表说明了认证流程。

![websocket 认证流程](./authentication.svg)

假设当前用户已经通过HTTP服务器进行了身份验证(1)，应用程序需要从HTTP服务器获取凭据(2)以便将它们发送到WebSocket服务器(3)，后者可以将其与用户账户数据库(4)进行核对。

用户名和密码不是这里凭据的好选择，即使只是因为数据库中没有以明文形式提供的密码。

与用户账户相关联的令牌是更好的选择。这些令牌必须使攻击者无法伪造。为了增加安全性，它们可以是短暂的，甚至是一次性的。


## 发送凭据

假设Web应用程序已经从HTTP服务器获得了身份验证凭据，很可能是一个令牌。有四种方法可以将它们传递给WebSocket服务器。

1. **作为WebSocket连接的第一个消息发送凭据。**
   这是完全可靠的，并且是本文讨论中最安全的机制。它有两个小缺点：
   - 认证是在应用层执行的。理想情况下，它应该在协议层进行管理。
   - 认证是在WebSocket握手之后执行的，这使得无法使用HTTP响应代码监控认证失败。

2. **将凭据添加到WebSocket URI的查询参数中。**
   这也是完全可靠的，但安全性较低。它有一个主要缺点：
   - URIs最终会出现在日志中，这会泄露凭据。即使可以通过一次性令牌降低这种风险，通常也被认为是不可接受的。
   认证仍然是在应用层执行的，但它可以在WebSocket握手之前发生，这改善了关注点的分离，并允许使用HTTP 401响应认证失败。

3. **设置WebSocket URI域的cookie。**
   Cookies是将凭据从Web应用程序发送到服务器的最常见的机制。在HTTP应用程序中，凭据将是一个会话标识符或序列化、签名的会话。
   不幸的是，当WebSocket服务器与Web应用程序运行在不同的域时，这个想法就会遇到同源策略。出于安全原因，无法在不同的源上设置cookie。
   适当的解决方法包括：
   - 创建一个从WebSocket服务器域提供服务的隐藏iframe。
   - 使用`postMessage`将令牌发送到iframe。
   - 在iframe中设置cookie。

   在打开WebSocket连接之前进行这些操作。

   共享HTTP服务器（例如www.example.com）和WebSocket服务器（例如ws.example.com）之间的父域（例如example.com），并在该父域上设置cookie也是一个解决方案。
   
   然而，cookie将与父域的所有子域共享。对于包含凭据的cookie，这是不可接受的。

   
   
   **译者注**：这种方案和当年PC端做的跨域登录挺像的，通过 `iframe` 来完成跨域操作，只不过那会儿还没有 `window.postMessage` 这个API，只能使用 *两层iframe嵌套*。
   
   
   
4. **将凭据添加到用户信息中的WebSocket URI。**
   理论上，让浏览器执行HTTP基本认证是一个好主意。
   实际上，由于浏览器支持不佳，它不起作用。截至2021年5月：
   - Chrome 90按预期工作。
   - Firefox 88过于积极地缓存凭据。当使用新的凭据再次连接到同一个服务器时，它会重用旧的凭据，这可能导致HTTP 401错误。然后下一个连接成功。也许错误会清除缓存。
   - Safari 14根本不理会凭据。

另外两个选项是不可行的：

1. **设置自定义HTTP头**
   这将是最优雅的机制，解决了上述所有选项的问题。
   不幸的是，由于WebSocket API不支持设置自定义头，这不起作用。

2. **使用TLS证书进行认证**
   虽然RFC建议这样做，但对于浏览器用户来说，安装TLS证书的体验太不主流了。这在高安全环境中可能有意义。我希望从事此类项目的开发人员不要从随机开源项目的文档中获取安全建议。


## 译者注

下面的实验太长了，从上述4个可能的方案中，我认为选择使用 message 在应用层面做用户认证，整体安全性不错，虽然实现复杂度会高一点。使用 `cookie` 的方案也可以，只是在*跨域*场景会比较麻烦。


## 做个实验

```shell
$ python experiments/authentication/app.py
Running on http://localhost:8000/
```

当您浏览到http://localhost:8000/上的HTTP服务器并提交用户名时，服务器会创建一个令牌并返回一个测试网页。

此页面尝试使用四种不同的方式在四个不同的源上打开四个WebSocket服务器的WebSocket连接进行认证。

### 应用层发message认证
```javascript
const websocket = new WebSocket("ws://...");
websocket.onopen = function() {
    websocket.send(token);
};
```
连接一打开，客户端就发送一个包含令牌的消息。

在连接处理程序的开始处，服务器接收此消息并对用户进行身份验证。如果认证失败，服务器将关闭连接：
```python
async def first_message_handler(websocket):
    token = await websocket.recv()
    user = get_user(token)
    if user is None:
        await websocket.close(CloseCode.INTERNAL_ERROR, "authentication failed")
        return

    ...
```

### 查询参数(query string)
客户端在打开连接之前将令牌添加到WebSocket URI的查询参数中：
```javascript
const uri = `ws://.../?token=${token}`;
const websocket = new WebSocket(uri);
```
服务器截取HTTP请求，提取令牌并对用户进行身份验证。如果认证失败，它返回HTTP 401：
```python
class QueryParamProtocol(websockets.WebSocketServerProtocol):
    async def process_request(self, path, headers):
        token = get_query_parameter(path, "token")
        if token is None:
            return http.HTTPStatus.UNAUTHORIZED, [], b"Missing token\n"

        user = get_user(token)
        if user is None:
            return http.HTTPStatus.UNAUTHORIZED, [], b"Invalid token\n"

        self.user = user
```

### Cookie
客户端在打开连接之前设置一个包含令牌的cookie。

cookie必须由从与WebSocket服务器相同来源加载的iframe设置（**主要是针对websocket的域和当前H5页面不同的情况下**）。这需要将令牌传递给此iframe。

```javascript
// 在主窗口
iframe.contentWindow.postMessage(token, "http://...");

// 在iframe中
document.cookie = `token=${data}; SameSite=Strict`;

// 在主窗口
const websocket = new WebSocket("ws://.../");
```

这一系列操作必须在主窗口和iframe之间同步。

服务器截取HTTP请求，提取令牌并对用户进行身份验证。如果认证失败，它返回HTTP 401：
```python
class CookieProtocol(websockets.WebSocketServerProtocol):
    async def process_request(self, path, headers):
        # 在非WebSocket请求上提供iframe
        ...

        token = get_cookie(headers.get("Cookie", ""), "token")
        if token is None:
            return http.HTTPStatus.UNAUTHORIZED, [], b"Missing token\n"

        user = get_user(token)
        if user is None:
            return http.HTTPStatus.UNAUTHORIZED, [], b"Invalid token\n"

        self.user = user
```

### 用户信息
客户端在打开连接之前将令牌添加到用户信息中的WebSocket URI：
```javascript
const uri = `ws://token:${token}@.../`;
const websocket = new WebSocket(uri);
```
由于HTTP基本认证设计用于接受用户名和密码，而不是令牌，我们发送`token`作为用户名并将令牌作为密码。

服务器截取HTTP请求，提取令牌并对用户进行身份验证。如果认证失败，它返回HTTP 401：
```python
class UserInfoProtocol(websockets.BasicAuthWebSocketServerProtocol):
    async def check_credentials(self, username, password):
        if username != "token":
            return False

        user = get_user(password)
        if user is None:
            return False

        self.user = user
        return True

async def user_info_handler(websocket):
    user = websocket.user

    ...
```

### 服务器之间websocket认证
当WebSocket客户端是一个独立的程序而不是在浏览器中运行的脚本时，限制要少得多。HTTP认证是这种情况下的最佳解决方案。

使用HTTP基本认证（**RFC 7617**）对websockets客户端进行身份验证，将凭据包含在URI中：
```python
async with websockets.connect(
    f"wss://{username}:{password}@example.com",
) as websocket:
    ...
```

（如果它们包含不安全字符，则必须对`username`和`password`进行`quote()`。）

使用HTTP承载者认证（**RFC 6750**）对websockets客户端进行身份验证，添加一个合适的`Authorization`头：
```python
async with websockets.connect(
    "wss://example.com",
    extra_headers={"Authorization": f"Bearer {token}"}
) as websocket:
    ...
```



>原文：https://websockets.readthedocs.io/en/stable/topics/authentication.html