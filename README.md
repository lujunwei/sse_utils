# deepexi_sse_utlis
与客户端建立长连接，解决需要需要轮询的困境。与webscoket不同的是，sse只是http协议，且只能有服务端发送通讯。

注意：node http连接默认2分钟会超时，如服务端连接发送事件的间隔需超过2分钟，需服务端自行处理（设置timeout时间 or 在2分钟的间隔内发送无效事件）

## how to install

```sh
npm install deepexi_sse_utils
```

## how to use

### 单次发送消息（如状态需实时返回给前端）
```js
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
### 其他情况发送消息（非单次，或者其他任意情况）
```js
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

## API
### Methods
|   方法名   | 说明 | 参数 |
| :--: | :--: | ---- |
| send | 与客户端建立长连接,返回值是 stream对象 | optsions选项<br />options.setResHeader 设置请求头function，required<br />options.sendType once单次 repeat重复 other其他，默认发送一次<br />options.sender 消息发送者，处理什么时候发送消息和结束发送消息，参数send func，结束需要发送'sseEnd'消息，非一次使用<br />options.onceMsg 单次发送消息主体，默认是''<br />options.retry 长连接发送错误时，重试频率，毫秒, 默认10s |
### Events
|   事件名称   | 说明 | 参数 |
| :--: | :--: | ---- |
| sseEnd | 客户端接收到sseEnd事件，说明通讯已结束 |  |




