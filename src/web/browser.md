# Web —— 从浏览器开始

浏览器始终是你在 Web 挑战中使用得最频繁的工具。

网页界面通常由 **HTML**（HyperText Markup Language）超文本标记语言实现，通过将其解析成树状的 **DOM**（Document Object Model），浏览器借此标记出每个标签所对应的层级。当然，这其中还涉及到 **CSS**（Cascading Style Sheets）样式为 HTML 标签添加格式及外观，**JS**（JavaScript）脚本为 HTML 标签添加动态触发器。

打开 https://example.com/ 或别的网站，并在网页中按下 F12，或单击右键并进入审查元素，你将有办法打开浏览器提供的开发者工具。

开发者工具里的 **Inspector** (Firefox) / **Element** (Chromium) 一栏展现了每个 HTML tag 对应网页里实际渲染出的元素。你可以通过鼠标在 DOM 树中切换，甚至对它进行修改，然后观察网页的变化。小练习：将大标题文字 Example Domain 改为 111111。

**Console** 一栏则是当前网页的 JS 控制台，它解析由你输入的 JavaScript 表达式，并在当前的上下文环境中执行；这个环境指的是，如果你在 Debugger 一栏中对某些操作下了断点，引擎执行到断点行代码时的变量及调用栈上下文环境，否则默认以 window（窗口）为执行环境。小练习：在控制台中执行 ```document.getElementsByTagName("p")[0].innerHTML = '2 <br> 3';``` 并观察效果。

**Network** 一栏则记录了所有浏览器与后端服务器的交互。如果为空，你可能需要按 F5 刷新界面以重新开始记录。在 CTF Web 题中，你往往会需要查看网络流量以推断某个功能到底是如何实现的（如排行榜，最高记录到底是保存在前端还是后端？如何把当前成绩存进去的？等等），并可以借助已有的数据交互格式，向后端服务器发送修改过后的新的数据。小练习：自行查看不同网站的网络流量，尝试理解 HTTP 状态码（200、404、500、400、302 等等）的含义、理解 HTTP 请求方式（GET/POST/HEAD）的含义、理解 HTTP URL 路径的基本写法及作用、理解不同 HTTP 请求/回复头的含义（如 Host、Cookie、Server、Content-Length、Content-Type、User-Agent 等等），并尝试修改某个特定请求的任一部分，观察最终返回结果的变化。

对于网络请求，除了浏览器自带的开发者工具，还有更专业用于分析的工具，如 Charles、Burpsuite 等，借助中间人 MiTM 的方式获取你的网络流量，可以自行安装尝试。

**Storage** 一栏则保存了网页存储在浏览器本地的数据，如 Local Storage、Cookie 等。你可以借助此功能查看或修改它们。

### 下一章节： [Web —— HTTP 协议](./http.md)