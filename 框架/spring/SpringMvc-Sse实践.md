[toc]

---

# SSE介绍

SSE（Server-SentEvents，即服务器发送事件）是围绕只读Comet交互推出的API或者模式。SSE API用于创建到服务器的单向连接，服务器通过这个连接可以发送任意数量的数据。服务器响应的MIME类型必须是text/event-stream，而且是浏览器中的JavaScript API能解析格式输出。SSE支持短轮询、长轮询和HTTP流，而且能在断开连接时自动确定何时重新连接。

- SSE特点：实现简单、 单向通信、自动重连
- 业务场景：客户端与服务端建立连接后，只需要服务端给客户端发送数据，客户端无需要给服务端发送数据

# 前端DEMO

## 基于AXIOS

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <script src="https://cdn.jsdelivr.net/npm/axios/dist/axios.min.js"></script>
  <script src="https://ajax.aspnetcdn.com/ajax/jquery/jquery-3.5.1.min.js"></script>
  <title>Document</title>
</head>

<body>
  <div class="form-group">
    <label for="clientId">clientId:</label>
    <input type="text" class="form-control" id="clientId">
  </div>
  <div class="form-group">
    <label for="content">content</label>
    <input type="text" class="form-control" id="content">
  </div>
  <button id="subscribeBtn" class="btn btn-primary">订阅</button>
  <button id="sendBtn" class="btn btn-primary">发送</button>
  <button id="closeBtn" class="btn btn-primary">关闭</button>

  <p id="responseText">

  </p>
</body>

</html>
<script>
  window.onload = function () {
    let start = 0;
    $("#subscribeBtn").click(() => {
      const clientId = $("#clientId").val();
      start = 0;
      axios({
        method: 'get',
        url: 'http://localhost:8080/smilex/open/sse/subscribe?clientId=' + clientId,
        responseType: 'stream', // 设置为流
        headers: {
          Accept: "text/event-stream" // 接受类型
        },
        onDownloadProgress: res => {
            // 每次推送会在这里打印，但不一定每次都是一次传输完的。
          console.log(res.event.currentTarget.response.substring(start, start + res.bytes))
          start += res.bytes;
        },
      }).then(function (response) {
        console.log("resposne", response)
        const stream = response.data
      });
    })
    $("#sendBtn").click(() => {
      const clientId = $("#clientId").val();
      const content = $("#content").val();
      axios({
        method: 'get',
        url: 'http://localhost:8080/smilex/open/sse/send?clientId=' + clientId + "&content=" + content,
      }).then(function (response) {
        // console.log("发送", response)
      });
    })
    $("#closeBtn").click(() => {
      const clientId = $("#clientId").val();
      axios({
        method: 'get',
        url: 'http://localhost:8080/smilex/open/sse/close?clientId=' + clientId,
      }).then(function (response) {
        // console.log("关闭", response)
      });
    })

  }
</script>
```



## 基于 EventSource

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <script src="https://cdn.jsdelivr.net/npm/axios/dist/axios.min.js"></script>
  <script src="https://ajax.aspnetcdn.com/ajax/jquery/jquery-3.5.1.min.js"></script>
  <title>Document</title>
</head>

<body>
  <div class="form-group">
    <label for="clientId">clientId:</label>
    <input type="text" class="form-control" id="clientId">
  </div>
  <div class="form-group">
    <label for="content">content</label>
    <input type="text" class="form-control" id="content">
  </div>
  <button id="subscribeBtn" class="btn btn-primary">订阅</button>
  <button id="sendBtn" class="btn btn-primary">发送</button>
  <button id="closeBtn" class="btn btn-primary">关闭</button>

  <p id="responseText">

  </p>
</body>

</html>
<script>
  // EventSource
 window.onload = function () {
     let source = null
 
     $("#subscribeBtn").click(() => {
       const clientId = $("#clientId").val();
       source = new EventSource("http://localhost:8080/smilex/open/sse/subscribe?clientId=" + clientId);
       source.addEventListener('message', function (e) {
         // console.log("message", e);
         //do something
         $("#responseText").html($("#responseText").html() + "\n" + e.data);
       });
 
       source.addEventListener('open', function (e) {
         //do something
         console.log("open", e)
       }, false);
 
       source.addEventListener('error', function (e) {
         console.log("error", e, source.readyState)
         if (source.readyState == EventSource.CLOSED) {
           //do something
           source.close();
         } else {
           //do something
         }
       }, false);
     })
     $("#sendBtn").click(() => {
       const clientId = $("#clientId").val();
       const content = $("#content").val();
       axios({
         method: 'get',
         url: 'http://localhost:8080/smilex/open/sse/send?clientId=' + clientId + "&content=" + content,
       }).then(function (response) {
         console.log("发送", response)
       });
     })
     $("#closeBtn").click(() => {
       const clientId = $("#clientId").val();
         // 需要主动关闭，如果直接调用接口关闭，会导致自动重连。
       source.close();
       axios({
         method: 'get',
         url: 'http://localhost:8080/smilex/open/sse/close?clientId=' + clientId,
       }).then(function (response) {
         console.log("关闭", response)
         $("#responseText")[0].innerHTML = ""
       });
     })
   }

</script>
```

# SseEmitter

1. 前端发起连接 创建并返回`SseEmitter`对象
2. 调用`SseEmitter`对象的`send`方法
3. 发送结束后，调用 `complete`方法

## service

```java
/**
 * Sse服务
 */
public interface SseService {

    /**
     * 连接
     *
     * @param clientId
     * @return
     */
    SseEmitter connect(String clientId);

    /**
     * 发送
     *
     * @param clientId
     * @param content
     * @return
     */
    boolean send(String clientId, String content);

    /**
     * 关闭
     *
     * @param clientId
     * @return
     */
    boolean close(String clientId);

}
```

## serviceImpl

```java
@Slf4j
@Service
public class SseServiceImpl implements SseService {
    @Override
    public SseEmitter connect(String clientId) {
        if (SseSession.exists(clientId)) {
            SseSession.remove(clientId);
        }
        SseEmitter sseEmitter = new SseEmitter(0L);
        sseEmitter.onError((err) -> {
            log.error("type: SseSession Error, msg: {} session Id : {}", err.getMessage(), clientId);
            SseSession.onError(clientId, err);
        });

        sseEmitter.onTimeout(() -> {
            log.info("type: SseSession Timeout, session Id : {}", clientId);
            SseSession.remove(clientId);
        });

        sseEmitter.onCompletion(() -> {
            log.info("type: SseSession Completion, session Id : {}", clientId);
            SseSession.remove(clientId);
        });
        SseSession.add(clientId, sseEmitter);
        return sseEmitter;
    }

    @Override
    public boolean send(String clientId, String content) {
        if (SseSession.exists(clientId)) {
            try {
                SseSession.send(clientId, content);
                return true;
            } catch (IOException exception) {
                log.error("type: SseSession send Erorr:IOException, msg: {} session Id : {}", exception.getMessage(), clientId);
            }
        } else {
            throw new SXException("User Id " + clientId + " not Found");
        }
        return false;
    }

    @Override
    public boolean close(String clientId) {
        log.info("type: SseSession Close, session Id : {}", clientId);
        return SseSession.remove(clientId);
    }
}
```

## SseSession

> 利用Map 管理全局的 SseEmitter

```java
public class SseSession {


    private static Map<String, SseEmitter> sessionMap = new ConcurrentHashMap<>();

    public static void add(String sessionKey, SseEmitter sseEmitter) {
        if (sessionMap.get(sessionKey) != null) {
            throw new SXException("client exists!");
        }
        sessionMap.put(sessionKey, sseEmitter);
    }

    public static boolean exists(String sessionKey) {
        return sessionMap.get(sessionKey) != null;
    }

    public static boolean remove(String sessionKey) {
        SseEmitter sseEmitter = sessionMap.get(sessionKey);
        if (sseEmitter != null) {
            sseEmitter.complete();
            sessionMap.remove(sessionKey);
            return true;
        }
        return false;
    }

    public static void onError(String sessionKey, Throwable throwable) {
        SseEmitter sseEmitter = sessionMap.get(sessionKey);
        if (sseEmitter != null) {
            sseEmitter.completeWithError(throwable);
        }
    }

    public static void send(String sessionKey, String content) throws IOException {
        sessionMap.get(sessionKey).send(content);
    }

}

```

## Controller

```java
/**
 * SSE测试
 */
@RestController
@RequestMapping("/demo/sse")
public class DemoSseController {

    @Resource
    private SseService sseService;

    @RequestMapping(value = "/subscribe", produces = {MediaType.TEXT_EVENT_STREAM_VALUE})
    public SseEmitter subscribe(String clientId) {
        return sseService.connect(clientId);
    }


    @RequestMapping(value = "/send")
    public R send(String clientId, String content) {
        if (sseService.send(clientId, content)) {
            return R.success();
        }
        return R.fail();
    }


    @RequestMapping(value = "/close")
    public R close(String clientId) {
        sseService.close(clientId);
        return R.success();
    }


}

```

# 注意事项

1. 前端使用`EventSource`时，如果后端服务先关闭了连接，那么 `EventSource`会抛出异常并自动发起重连。所以如果不想重连，需要前端关闭后，再关闭后端连接。