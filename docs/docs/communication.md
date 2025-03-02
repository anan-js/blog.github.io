---
title: 前端实现即时通讯，常用的技术包括短轮询、长轮询、WebSocket和SSE
createTime: 2025/03/02 15:11:00
permalink: /article/aomd5h4p/
---
## **短轮询**

`短轮询（Short Polling）`是一种通过定期向服务器发送请求来获取最新数据的实时通讯方法。与`长轮询（Long Polling）`不同，短轮询不会保持持久连接，而是在每次请求时向服务器询问是否有新数据。

以下是`短轮询`的基本工作流程：

1. **客户端发送请求**：前端通过定时发送 HTTP 请求到服务器，以获取最新数据。这个请求通常是一个简单的 AJAX 请求。
2. **服务器响应**：服务器接收到请求后，会立即返回当前的数据状态给客户端。如果有新数据，服务器会将其包含在响应中返回；否则，服务器会返回一个空响应或者一个指示没有新数据的状态码。
3. **客户端处理响应**：客户端收到服务器的响应后，解析数据并进行相应的处理。如果服务器返回了新数据，客户端可能会更新界面显示或者执行其他逻辑。
4. **重复请求**：客户端在处理完服务器响应后，会等待一段时间（通常是几秒钟），然后再次发送新的请求，以获取最新的数据状态。
5. **循环进行**：这个过程会不断循环执行，客户端定期发送请求，服务器定期响应，从而实现实时通讯的效果。

在短轮询的实现中，客户端可以使用`setInterval()`函数周期性地发送请求，然后在收到响应后更新界面。以下是一个简单的示例：

```javascript
function pollServerForUpdates() {
  setInterval(function() {
    fetch('http://example.com/update')
      .then(response => response.json())
      .then(data => {
        // 处理从服务器返回的数据
        console.log('收到新数据：', data);
      })
      .catch(error => {
        console.error('请求错误：', error);
      });
  }, 5000); // 每5秒发送一次请求
}

pollServerForUpdates();
```

在实际应用中，需要根据具体情况调整轮询的频率，并且考虑到服务器和网络的性能，以避免对系统造成过大的负担。

### 优缺点

优点：

- `简单易实现`：短轮询的实现相对简单，不需要特殊的服务器支持，使用简单的HTTP请求即可。
- `兼容性好`：与长轮询相比，短轮询在各种浏览器和网络环境下都能够正常工作。

缺点：

- `效率较低`：因为客户端需要频繁地发送请求，可能会造成网络流量浪费，同时也会增加服务器压力。
- `实时性差`：由于客户端需要等待固定的时间间隔才能获取更新，因此实时性不如长轮询或WebSocket。

总的来说，短轮询`适用于一些对实时性要求不高，且对服务器压力要求相对较低`的场景。虽然实时性和效率不如长轮询或WebSocket，但在一些简单的应用中，短轮询可以是一种简单有效的实现方式。 在前端实现即时通讯，常用的技术包括短轮询，WebSocket和SSE。

## 长轮询

`长轮询（Long Polling）`是一种改进的轮询技术，它在某些方面优于传统的短轮询。

`长轮询`通过客户端向服务器发送一个`持续连接`的请求，在有`新数据可用时`立即返回响应，否则保持连接直到超时或者有新数据到达。

以下是长轮询的`基本工作流程`：

1. `**客户端发送请求**`：客户端向服务器发送一个长轮询请求，通常是一个普通的 HTTP 请求，但在服务器端会保持连接打开。
2. `**服务器处理请求**`：服务器接收到长轮询请求后，会检查是否有新的数据可用。如果有新数据，服务器会立即将其包含在响应中返回给客户端。
3. `**响应返回**`：如果服务器有新数据，它会立即返回响应给客户端，并关闭连接。如果没有新数据可用，服务器会保持连接打开，等待一段时间直到超时或者有新数据到达。
4. `**客户端处理响应**`：客户端收到服务器的响应后，解析数据并进行相应的处理。然后，客户端会立即发送一个新的长轮询请求，以便在服务器端保持持续连接。
5. `**循环进行**`：这个过程会不断循环执行，客户端和服务器之间保持持续连接，从而实现实时通讯的效果。

长轮询的优点是可以实现实时通讯的效果，并且相对于短轮询来说减少了不必要的 HTTP 请求次数，减少了网络流量和服务器负载。但是长轮询仍然存在一定的延迟，因为客户端需要等待服务器响应或者超时才能发送下一个请求。同时，长轮询需要服务器支持保持长连接，因此对服务器的资源消耗较大。

以下是使用 Node.js 实现的简单长轮询服务器和客户端的代码示例：

首先是服务器端的代码 (`server.js`)：

```javascript
const express = require('express');

const app = express();
const port = 3000;

let data = "Initial Data";

app.get('/data', (req, res) => {
  const timeout = 10; // 设置超时时间，单位为秒
  const startTime = new Date().getTime();

  const checkData = () => {
    // 检查数据是否发生变化或特定事件是否发生，这里假设直接使用一个全局变量 data 来模拟数据的变化
    if (data !== "Initial Data") {
      res.json({ data: data });
    } else if ((new Date().getTime() - startTime) > timeout * 1000) {
      res.json({ data: null }); // 如果超时，则返回空响应
    } else {
      setTimeout(checkData, 1000); // 等待一段时间后再次检查
    }
  };

  checkData();
});

app.post('/update', express.json(), (req, res) => {
  data = req.body.data;
  res.json({ message: "Data updated successfully" });
});

app.listen(port, () => {
  console.log(`Server is listening at http://localhost:${port}`);
});
```

接下来是客户端的代码 (`client.js`)：

```javascript
const fetch = require('node-fetch');

function longPolling() {
  setInterval(() => {
    fetch('http://localhost:3000/data')
      .then(response => response.json())
      .then(data => {
        if (data.data !== null) {
          console.log("Received updated data:", data.data);
        }
      })
      .catch(error => {
        console.error('Error:', error);
      });
  }, 1000); // 每秒重新发起请求
}

longPolling();
```

在这个示例中，服务器端使用 Express 框架创建了一个简单的 API，其中 `/data` 路由实现了长轮询的逻辑。客户端通过不断发送 GET 请求到 `/data` 路由来获取最新的数据。当数据发生变化时，服务器会立即返回响应。如果超过了设置的超时时间而没有新的数据可用，服务器会返回一个空响应，客户端会在收到空响应后立即重新发起请求。

### 优缺点

**优点**：

1. `**即时性（Real-Time）**`**：** 长轮询允许服务器在有新数据时立即向客户端推送信息，从而实现近乎实时的通信效果。
2. `**简单实现**`：与WebSocket相比，长轮询更容易实现，因为它利用了HTTP协议，无需特殊的服务器支持。
3. `**兼容性**`：由于长轮询基于HTTP协议，因此在大多数现代浏览器和服务器上都能够正常工作，而不需要特殊的配置或兼容性处理。

**缺点**：

1. `**资源浪费**`**：** 长轮询会造成服务器维持大量的长期连接，这可能会导致服务器资源的浪费，尤其是在大规模部署的情况下。
2. `**延迟较大**`**：** 即使在有新数据时，客户端也需要等待一段时间才能收到响应，因此长轮询的延迟相对较大，不如 WebSocket 或 Server-Sent Events 实时。
3. `**连接维持开销**`**：** 由于长轮询需要维持较长时间的连接，因此会增加网络和服务器的连接维持开销，尤其是在高并发情况下。
4. `**不支持全双工通信**`**：** 长轮询是一种单向通信方式，即客户端发送请求等待服务器响应，而服务器不能直接向客户端发送消息。这意味着在某些场景下，长轮询无法满足双向通信的需求。

综上所述，长轮询适用于需要实时通信但对`延迟要求不是很高`的场景，但在高并发和大规模部署的情况下，可能会`带来一定的性能和资源开销`。在选择长轮询还是其他实时通信技术时，需要根据具体的业务需求和技术环境进行权衡。

## SSE

`SSE（Server-Sent Events）` 服务器推送事件，简称 SSE，是一种服务端实时`主动`向浏览器推送消息的技术。

`SSE` 是 `HTML5` 中一个与通信相关的 API，主要由两部分组成：服务端与浏览器端的通信协议（`HTTP` 协议）及浏览器端可供 JavaScript 使用的 `EventSource` 对象。

SSE是一种用于`实现服务器向客户端推送数据`的技术，它允许服务器端`实时`地向客户端发送事件流。相比于长轮询，SSE 更加轻量级且易于实现，但也有一些限制。

SSE 协议非常简单，本质是浏览器发起 http 请求，服务器在收到请求后，返回状态与数据，并附带以下 headers：

```http
Content-Type: text/event-stream
Cache-Control: no-cache
Connection: keep-alive
```

- SSE API规定推送事件流的 MIME 类型为 `text/event-stream`。
- 必须指定浏览器`不缓存`服务端发送的数据，以确保浏览器可以实时显示服务端发送的数据。
- SSE 是一个一直保持开启的 TCP 连接，所以 Connection 为 `keep-alive`。

下面是一个简单的使用 Node.js 实现 SSE 的示例代码：

```javascript
const express = require('express');

const app = express();
const port = 3000;

let clients = [];

app.get('/events', (req, res) => {
  res.setHeader('Content-Type', 'text/event-stream');
  res.setHeader('Cache-Control', 'no-cache');
  res.setHeader('Connection', 'keep-alive');

  const clientId = Date.now();
  clients.push({ id: clientId, res });

  // 发送一个初始化消息
  res.write(`data: Connected\n\n`);

  // 客户端断开连接时移除对应的客户端
  req.on('close', () => {
    clients = clients.filter(client => client.id !== clientId);
  });
});

// 路由用于发送数据到所有连接的客户端
app.post('/update', express.json(), (req, res) => {
  const newData = req.body.data;

  clients.forEach(client => {
    client.res.write(`data: ${JSON.stringify({ data: newData })}\n\n`);
  });

  res.json({ message: "Data sent successfully" });
});

app.listen(port, () => {
  console.log(`Server is listening at http://localhost:${port}`);
});
```

在这个示例中，客户端通过向 `/events` 路由发送 GET 请求来订阅服务器端的事件流。服务器端会维护一个客户端列表，当有新数据需要推送时，会遍历客户端列表将数据发送给所有客户端。

客户端可以使用 JavaScript 来监听事件流：

```javascript
const eventSource = new EventSource('http://localhost:3000/events');

eventSource.onmessage = function(event) {
  const data = JSON.parse(event.data);
  console.log("Received updated data:", data.data);
};

eventSource.onerror = function(error) {
  console.error('Error:', error);
};
```

### 优缺点

优点：

1. **实时性**：SSE 提供了更接近实时的数据推送，因为它是基于单个持久连接，服务器端可以实时向客户端发送数据。
2. **轻量级**：SSE 使用标准的 HTTP 协议，不需要额外的库或协议，因此实现更加轻量级。
3. **简单易用**：相比于 WebSockets，SSE 的实现和使用更加简单，无需处理复杂的连接管理和协议。

但是，SSE 也有一些限制：

1. **单向通信**：SSE 只支持服务器向客户端的单向通信，客户端无法直接向服务器发送数据。
2. **兼容性**：虽然大多数现代浏览器都支持 SSE，但并不是所有浏览器都支持。特别是一些旧版本的浏览器可能不支持 SSE。
3. **连接断开重连**：在某些情况下，连接可能会断开，需要客户端重新连接。这就需要客户端实现重连逻辑来保持连接的持久性。

总的来说，SSE 适合那些需要实时推送数据且对双向通信要求不高的应用场景，例如实时通知、实时数据更新等。在选择实时通信技术时，开发人员应根据具体的需求和场景来选择合适的技术。

### 应用场景

1. **即时通知和提醒：** SSE 可用于向用户发送即时通知和提醒，例如社交媒体应用中的新消息通知、电子邮件应用中的新邮件提醒等。
2. **实时数据更新：** SSE 可以用于在网页应用程序中实时更新数据，例如股票市场应用程序中的股票价格、即时聊天应用程序中的新消息等。
3. **在线游戏：** SSE 可以用于实现在线游戏中的实时通信和事件更新，例如多人在线游戏中的玩家位置更新、游戏状态变化等。
4. **监控和警报系统：** SSE 可以用于实时监控系统和警报系统，例如监控服务器状态、网络流量等，并在发生异常时向管理员发送警报。
5. **实时数据可视化：** SSE 可以用于在网页应用程序中实时更新数据可视化图表，例如实时股票走势图、实时天气预报等。
6. **在线交易系统：** SSE 可以用于在线交易系统中实时更新交易价格和订单状态，以及向用户发送交易确认和通知。
7. **事件流处理：** SSE 可以用于处理事件流数据，例如日志记录、数据分析等，实时将处理结果推送给客户端。
8. **客服支持：** SSE 可以用于实现客服支持系统，客户可以在网页上与 ChatGPT 进行实时交流，向其提出问题并获得即时的回复。

### 兼容性

![img](./communication/1730187793543-db373c8a-8300-41de-b069-18ab37810ce1.webp)

## Websocket

`Websocket` 是一种在客户端和服务器之间实现`双向通信`的技术，它通过一个`持久连接`在`客户端和服务器之间`建立`实时、高效`的通信机制。

相比传统的 HTTP 请求，它能够提供`更低的延迟`和`更高的效率`，使得实时通讯成为可能。

下面是使用 JavaScript 的简单示例：

**服务端代码**：

```javascript
const WebSocket = require('ws');

const wss = new WebSocket.Server({ port: 8080 });

wss.on('connection', function connection(ws) {
  console.log('A client connected');

  ws.on('message', function incoming(message) {
    console.log('Received message:', message);

    // 发送数据给所有连接的客户端
    wss.clients.forEach(function each(client) {
      if (client !== ws && client.readyState === WebSocket.OPEN) {
        client.send(message);
      }
    });
  });

  ws.on('close', function close() {
    console.log('Client disconnected');
  });
});
```

**客户端代码**：

```javascript
const ws = new WebSocket('ws://localhost:8080');

ws.onopen = function() {
  console.log('Connected to the server');
};

ws.onmessage = function(event) {
  console.log('Received message:', event.data);
};

ws.onclose = function() {
  console.log('Disconnected from the server');
};

// 发送数据给服务器
ws.send('Hello, server!');
```

### 特点

- 支持`双向通信，实时性更强 `
- 可以发送文本，也可以发送`二进制数据`
- 建立在`TCP协议`之上，服务端的实现比较容易
- 数据格式比较`轻量`，`性能开销小`，通信高效
- `没有同源限制`，客户端可以与任意服务器通信
- 协议标识符是ws（如果加密，则为wss），服务器网址就是 URL
- 与 HTTP 协议有着良好的`兼容性`。默认端口也是80和443，并且握手阶段采用 HTTP 协议，因此握手时不容易屏蔽，能通过各种 HTTP 代理服务器。

 

### 优缺点

优点包括：

1. `**实时性**`：Websocket 提供了双向通信的能力，可以实现实时的数据传输，适用于需要及时更新的应用场景，如实时聊天、在线游戏等。
2. `**高性能**`：Websocket 使用单个 TCP 连接，减少了握手和请求头的开销，降低了延迟，提高了性能。
3. `**跨平台支持**`：Websocket 是一种标准化的协议，并且受到现代浏览器的广泛支持，适用于多种平台和设备。
4. `**持久连接**`：Websocket 使用长期的连接，可以保持连接状态，服务器端可以主动向客户端推送数据，而不需要客户端主动发起请求。

然而，Websocket 也有一些缺点：

1. `**跨域限制**`**：** 与其他跨域通信技术一样，WebSocket 也受到浏览器的同源策略限制，需要通过 CORS 或代理等方式解决跨域访问的问题。
2. `**安全性考虑**`**：** WebSocket 的持久性连接可能会增加一些安全风险，例如长时间的连接可能会增加 DoS 攻击的风险，因此需要采取相应的安全措施，如限制连接数、实施身份验证等。
3. `**网络代理限制**`**：** 一些网络代理可能会阻止 WebSocket 连接，导致部分用户无法使用 WebSocket 进行通信，需要考虑兼容性和容错性。
4. `**状态管理复杂**`**：** WebSocket 的持久性连接需要服务器端维护连接状态，可能会增加服务器端的状态管理复杂性，需要考虑连接的管理和维护。
5. `**协议版本兼容性**`**：** 不同的浏览器和服务器可能支持不同版本的 WebSocket 协议，需要确保客户端和服务器端之间的协议版本兼容性，以确保通信的稳定性和可靠性。

总的来说，Websocket 是一种功能强大、实时性高的通信技术，适用于需要实时双向通信的应用场景。在选择实时通信技术时，开发人员可以根据需求和情况权衡各种通信技术的优缺点来选择最适合的技术。

### 应用场景

WebSocket 适用于需要实时双向通信的各种应用场景，包括但不限于：

1. **在线聊天应用：** 实时性是在线聊天应用的关键需求之一，WebSocket 可以实现客户端和服务器之间的即时通信，使得用户能够实时收发消息，提高用户体验。
2. **实时协作编辑：** 对于需要多人协作编辑的应用，如 Google Docs、在线白板等，WebSocket 可以实现实时的文档同步，保持所有参与者的编辑内容同步更新。
3. **实时游戏：** 在线多人游戏需要快速、实时的数据传输，WebSocket 可以实现游戏客户端和服务器之间的实时通信，支持游戏中的角色移动、游戏事件等实时更新。
4. **实时监控和通知：** 对于需要实时监控数据或发送实时通知的应用，如监控系统、实时报警系统等，WebSocket 可以实现服务器向客户端实时推送监控数据或通知信息。
5. **实时数据可视化：** 在需要实时展示数据的应用中，如股票行情、实时交通情况等，WebSocket 可以实现数据的实时更新和展示，使用户能够及时获取最新的信息。

总的来说，Websocket 在需要实时双向通信的各种应用场景中都有广泛的应用前景，特别是对于那些需要及时更新数据、实时交互的场景来说，是一种非常值得选择的通信技术。

### 兼容性

![img](./communication/1730187793572-eb1d155d-a600-4788-97cd-344a9104ba5b.webp)

### 与SSE对比

好的，下面是一个比较 WebSocket 和 Server-Sent Events (SSE) 的表格：

| 特性     | WebSocket                              | Server-Sent Events (SSE)                                   |
| -------- | -------------------------------------- | ---------------------------------------------------------- |
| 通信方向 | 双向通信                               | 单向通信                                                   |
| 协议     | 基于 TCP                               | 基于 HTTP                                                  |
| 使用难度 | 相对复杂                               | 轻量级，使用简单                                           |
| 实时性   | 实时通信，适用于需要即时更新的应用场景 | 实时推送，适用于只需要服务器向客户端推送数据的简单应用场景 |
| 延迟     | 通常较低，由于建立了持久性连接         | 可能较高，受限于 HTTP 请求-响应模式                        |
| 数据格式 | 任意类型的数据                         | 只能传输文本数据                                           |
| 自动重连 | 不支持，需要客户端手动重连             | 支持，客户端断开连接时会自动尝试重新连接                   |
| 连接个数 | 可同时支持大量连接                     | 连接数 HTTP/1.1 6 个，HTTP/2 可协商（默认 100）            |
| 应用场景 | 实时双向通信、即时更新的应用           | 只需要服务器向客户端推送数据的简单应用                     |

## 