# deepexi_sse_utlis
与客户端建立长连接，解决需要需要轮询的困境。与webscoket不同的是，sse只是http协议，且只能有服务端发送通讯。

## how to install

```sh
npm install deepexi_sse_utils
```

## how to use

### server端
#### 单次发送消息（如状态需实时返回给前端）
```sh
const SSEUtils = require('deepexi_sse_utils');
# server为服务端，监听请求
server.on('request', writeEventsOnce('hello world'));
function writeEventsOnce(msg) {
  return (req, res) => {
    const stream = SSEUtils.send({
      setResHeader: resHeaders => {
        res.writeHead(200, resHeaders);
      },
      sendType: 'once',
      onceMsg: msg,
    });

    res.body = stream.pipe(res);
  };
}
```
#### 其他情况发送消息（非单次，或者其他任意情况）
```sh
const SSEUtils = require('deepexi_sse_utils');
# server为服务端，监听请求
server.on('request', writeEventsInterval('hello world', 1000));
function writeEventsInterval(msgs, times = 1000) {
  let interval = null;
  return (req, res) => {
    const stream = SSEUtils.send({
      setResHeader: resHeaders => {
        res.writeHead(200, resHeaders);
      },
      sendType: 'other',
      sender: send => {
        let index = 0;
        interval = setInterval(() => {
          const msg = index === msgs.length ? 'sseEnd' : msgs[index];
          send(msg);

          if (msg === 'sseEnd') {
            // 清除事件
            clearInterval(interval);
          }
          index++;
        }, times);
      },
    });

    res.body = stream.pipe(res);
  };
}
```

### 客户端
SSE 的客户端 API 部署在EventSource对象上
```sh 
const es = new EventSource(server.url);
es.onmessage = (m) => {
    #your code，m.data为通讯信息
};
#可以接收其他的事件，如接收sseEnd事件，代表通讯结束
es.addEventListener('sseEnd', (event) => {
    #your code 
  }, false);
#客户端可主动结束通讯
es.close()
```

