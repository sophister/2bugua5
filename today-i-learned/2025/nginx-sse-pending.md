# Nginx HTTPS 下 SSE 接口响应延迟问题的排查与解决

**PS：这篇文章内容，是我让ChatGPT根据我的问题描述和排查过程，自己写出来的，略有删减**

在实际项目中，我们常常使用 Nginx 作为反向代理服务器，将请求转发到后端 Java 服务，其中一些服务会采用 Server-Sent Events（SSE）技术实现实时数据推送。最近，我们遇到一个有趣的问题：在使用 80 端口（HTTP）访问 SSE 接口时，浏览器能立即接收到响应数据，而使用 443 端口（HTTPS）时，同样的请求在浏览器端却会显示长达 12 秒的 pending 状态，之后才开始收到数据。

## 问题现象

#### 同一个接口，不同访问方式对比

* 通过nginx的80端口访问，很快
* 不走nginx，直接访问后端服务，很快
* 通过nginx443端口访问，基本都会阻塞12 - 18秒

#### 在nginx日志中，检查上游服务的响应时间

即使我们在浏览器里，通过nginx的443端口访问接口，在浏览器里阻塞十几秒的情况下，nginx日志里从上游接收到的响应，仍然非常快。

- **upstream_connect_time**：0.000 秒
- **upstream_header_time**：0.004 秒
- **upstream_response_time**：0.004 秒

也就是说，nginx 从后端 Java 服务接收到数据的时间极短。但在浏览器中观察时，HTTPS 下的请求却有明显延迟，浏览器显示请求 pending 约 12 秒后才开始呈现数据。

## 排查过程

针对这一问题，我们进行了以下排查工作：

1. **确认上游响应时间**
   首先检查 nginx 日志，发现上游响应时间几乎没有延迟，说明后端服务处理速度正常，不存在性能瓶颈。
2. **比较 HTTP 与 HTTPS 行为**
   我们分别使用 80（HTTP）和 443（HTTPS）两种方式访问 SSE 接口，结果显示 HTTP 下响应即时，而 HTTPS 下延迟明显。由于两端口的 nginx 配置几乎一致，因此怀疑问题出在 HTTPS 下的某些细节上。
3. **检查协议与配置**
   初步排查时，我们确认浏览器显示的连接均为 HTTP/1.1，并非 HTTP/2，因此问题与协议版本无关。接着我们重点关注 nginx 的缓冲机制，因为 nginx 默认启用响应缓冲（proxy_buffering on），可能会对 SSE 这种实时推送的数据产生影响。
4. **定位到代理缓冲设置**
   各种配置都相同，问题都指向nginx，只能怀疑SSL下nginx的某些缓冲设置有问题。通过GPT指示，尝试明确关闭缓冲区配置，问题得到解决。

## 最终解决方案

经过确认后，我们在 443 端口对应的 location 配置中加入了以下配置项：

```nginx
location /api/ {
    proxy_pass http://backend/api/;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_http_version 1.1;
    proxy_set_header Connection 'keep-alive';
    proxy_set_header Upgrade $http_upgrade;
    proxy_read_timeout 3600s;
    proxy_send_timeout 3600s;
    proxy_cache off;
    proxy_buffering off;  # 关键配置，关闭缓冲
}
```

加入 `proxy_buffering off;` 后，nginx 在 HTTPS 下的 SSE 响应数据能够实时传递，浏览器不再出现长时间的 pending 状态，响应速度与 80 端口一致。

## 技术解析

为什么会出现这种现象？主要原因在于 nginx 的代理缓冲机制。默认情况下，nginx 会对从上游返回的数据进行缓冲，直到缓冲区中积累到一定量的数据后再发送给客户端。而 SSE 接口通常是用来实时推送数据的，其特点是数据可能很少甚至只有一小段，缓冲机制就会延迟数据的下发。

仍然有个疑问，为什么同样的配置，80端口下也没有明确关闭缓冲区，也不会导致请求阻塞住，几乎瞬间就返回给浏览器端了。

但在 HTTPS 模式下，只有明确关闭缓冲机制（proxy_buffering off）才能确保 SSE 数据一产生就被立即转发给客户端。

## 总结

通过本次问题的排查，我们可以得出以下几点经验：

- 对于实时推送的 SSE 接口，务必检查 nginx 的缓冲配置，确保数据能即时下发。
- HTTPS 环境下可能会因为 SSL/TLS 的额外处理而引入延迟，配置上应特别留意相关参数。
- 排查问题时，应综合使用 nginx 日志和浏览器开发者工具（如 Network 面板），对比各个阶段的耗时，从而准确定位问题原因。

------- 时2025年2月28日19：44竣工于成都万安镇