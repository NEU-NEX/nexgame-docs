# Web —— HTTP 协议

HTTP 协议是浏览器与后端服务器交互的最基本协议，在 99.99% 的情况下。

先前通过 Network 一栏，你已经了解了一些基本的 HTTP 格式。

你应该已经理解后端服务器会凭借 HTTP 请求中的哪一部分，辨别请求的资源是什么、如何请求、返回格式等信息。

对于基础的 Web 挑战，理解到这里已经足够了。近年来随着 CDN 的普遍运用，一种新型攻击方法 HTTP Request Smuggling 开始逐渐浮出水面，其借助不同软件对 HTTP RFC 的解析差（如 Content-Length / Transfer-Encoding）绕过一些安全限制，甚至可以伪造经过该中间件的请求-回复包。

**curl** 是一个在 Web 测试中必不可少的工具。其可以很方便地显示 HTTP 请求的详细信息，并指定使用特定的 URL、HTTP 版本、POST 数据或 HTTP 头。尝试在终端里执行 ```curl https://1.1.1.1/cdn-cgi/trace -vv --http1.1 --path-as-is -k -d "aaa=b"``` 并解释每个参数以及输出的含义。

当然，有时候还可能会遇到构造非法 HTTP 请求的情况（在本次比赛中不会出现），这时候只能使用系统提供的 socket 套接字手动发送 TCP 包。

**nc**（netcat） 是在 TCP/UDP 发收包测试中必不可少的工具。尝试使用 nc 连接 example.com 80 端口，并发送一个基础的 HTTP 请求，然后解释服务器的返回数据。

### 下一章节： [Web —— 后端应用](./backend.md)