---
title: springboot整合websocket以stomp实现
date: 2020-04-11 00:00:00
tags:
- springboot
categories:
- springboot
---

## springboot整合websocket以stomp实现

- 记录springboot整合websocket的过程和遇到的一些问题,这里使用stomp来实现

<!--more-->

### 1. 引入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-websocket</artifactId>
</dependency>
```

<!-- more -->

### 2.配置websocket服务

```java
@Configuration
@EnableWebSocketMessageBroker // 开启stomp服务
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {

    @Override
    public void configureMessageBroker(MessageBrokerRegistry registry) {
        // 设置消息代理前缀,前缀为/topic的消息会转发给消息代理,再由消息代理广播给客户端
        // --- 接受客户端 订阅 的路径前缀
        registry.enableSimpleBroker("/topic","/queue");
        // 通过前缀过滤出需要被注解方法处理的消息
        // 前缀为/app的消息通过注解方法@MessageMapping处理
        // --- 接受客户端 消息 的路径前缀
        registry.setApplicationDestinationPrefixes("/app");

    }

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        // 定义前缀为/chat的endpoint,开启sockjs支持
        // 客户端通过该url建立连接
        // 设置允许跨域
        registry.addEndpoint("/websocket")
                .setAllowedOrigins("*")
                .withSockJS();
    }
}
```

### 3.消息控制层

```java
@Controller
public class MessageController {

    Logger logger = LoggerFactory.getLogger(getClass());

    @Autowired
    private SimpMessagingTemplate messagingTemplate;
    
    /**
     * 广播推送
     * @param inMessage 接受消息体
     */
    @MessageMapping("/chat")
    public void singleChat(InMessage inMessage) {
        String fromUser = inMessage.getFrom();
        String toUser = inMessage.getTo();
        // 根据用户id获取用户的在线情况
        String toPath = SocketSessionRegistry.getSessionId(toUser);
        if (toPath == null || toPath.isEmpty()) {
            logger.info("{},该用户不在线,存入数据库", toUser);
        } else {
            logger.info("{}===>{},一条在线消息", fromUser, toUser);
            // 消息推送给订阅了 /queue/chat/{toUser} 的用户
            messagingTemplate.convertAndSend(
                    "/queue/chat/" + toUser,
                    new OutMessage(inMessage.getFrom(), inMessage.getFrom() + "给你发送了:" + inMessage.getContent())
            );
        }
    }
}
```

### 4.监听websocket事件

```java
/**
 * 会话事件监听基类
 *
 * @author rxliuli
 */
public abstract class BaseSessionEventListener<Event extends AbstractSubProtocolEvent> implements ApplicationListener<Event> {

    protected final Logger log = LoggerFactory.getLogger(getClass());

    /**
     * 计算出 user id 和 session id 并传入到自定义的函数中
     *
     * @param event      事件
     * @param biConsumer 自定义的操作
     */
    protected void using(Event event, BiConsumer<String, String> biConsumer) {
        StompHeaderAccessor sha = StompHeaderAccessor.wrap(event.getMessage());
        //login get from browser
        List<String> shaNativeHeader = sha.getNativeHeader("token");
        String user;
        if (shaNativeHeader == null || shaNativeHeader.isEmpty()) {
            user = null;
        } else {
            user = shaNativeHeader.get(0);
        }
        String sessionId = sha.getSessionId();
        biConsumer.accept(user, sessionId);
    }
}
```

```java
/**
 * websocket建立连接事件
 * @date 2020/4/10
 */
@Component
public class SessionConnectEventListener extends BaseSessionEventListener<SessionConnectEvent> {

    @Override
    public void onApplicationEvent(SessionConnectEvent event) {
        using(event,(user,session) -> {
            //如果当前用户没有登录（没有认证信息），就添加到游客里面
            if (user == null || "".equals(user) || "undefined".equals(user) || "null".equals(user)) {
                log.info("user is null");
            }
            log.info("{}<===>{},connect",user,session);
            SocketSessionRegistry.registerSessionId(user,session);
        });
    }
}
```

```java
/**
 * websocket断开连接事件
 * @date 2020/4/10
 */
@Component
public class SessionDisconnectEventListener extends BaseSessionEventListener<SessionDisconnectEvent> {

    @Override
    public void onApplicationEvent(SessionDisconnectEvent event) {
        using(event,(user,session) -> {
            log.info("{}<===>{},disconnect",user,session);
            SocketSessionRegistry.removeSessionId(session);
        });
    }
}
```

### 5.客户端

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
  <script src="https://cdn.bootcss.com/jquery/3.4.1/jquery.min.js"></script>
  <script src="https://cdn.bootcss.com/sockjs-client/1.4.0/sockjs.min.js"></script>
  <script src="https://cdn.bootcss.com/stomp.js/2.3.3/stomp.min.js"></script>
  <script src="./chat2.js"></script>
</head>
<body>
  <div class="body-left">
    <div id="state">未连接</div>
    <button id="connect" onclick="connect()">连接</button>
    <button id="disConnect" onclick="disConnect()">断开</button>
    <div>
        <label for="from">
          发送方:<input id="from" type="text">
        </label>
        <label for="dest">
          接收方:<input id="dest" type="text"/>
        </label>
        <textarea id="content" rows="10"></textarea>
        <button id="send" onclick="send()">发送</button>
    </div>
    <div>
    </div>
</div>
<div id="chat" class="body-right">
    <p class="p-left">client message ...</p>
    <p class="p-right">server message ...</p>
</div>
</body>
</html>
```

```javascript
var username = 'niko'
var client = null
var headers = null
var token = 'asd0724'

function connect() {

  setHeaders()

  var ws = new SockJS('http://127.0.0.1:8090/websocket')
  client = Stomp.over(ws)
  // client = Stomp.client('ws://127.0.0.1:8090/websocket/websocket')
  // 建立连接
  client.connect(headers,
    function(frame) {
      console.log('success',frame);
      sub()
    },
    function(error) {
      console.log('error',error);
    })
}

// 断开连接
function disConnect() {
  client.disconnect(function() {
    alert('good bye!')
  },headers)
}

// 发送消息
// 参数为 地址, 请求头, 内容
function send() {
  var to = $('#dest').val()
  var from = $('#from').val()
  client.send('/app/chat',
  headers,
  JSON.stringify({'content': 'hello server', 'to':to, 'from':from}))
}

// 订阅
function sub() {
  var from = $('#from').val()
  client.subscribe('/queue/chat/'+from,function(message) {
    console.log(message);
  },headers)
}

function setHeaders() {
  var from = $('#from').val()
  headers = {
    'token': from
  }
}
```



### 参考

[https://blog.rxliuli.com/p/7eaebba3/](https://blog.rxliuli.com/p/7eaebba3/)