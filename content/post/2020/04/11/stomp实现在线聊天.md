---
title: stomp实现聊天室
date: 2020-04-11 00:00:00
tags:
- springboot
categories:
- springboot
---

## stomp实现聊天室

记录springboot+stomp+fastdep实现的在线聊天.

主要需要实现的功能包括

- 通过fastdep实现的shiro登录以及token验证
- 通过stomp实现websocket的服务端,实现广播推送,在线聊天,离线消息处理

参照`springboot整合websocket`与`springboot整合fastdep`搭建基本环境

<!--more-->

### 实现websocket连接token携带

因为项目基于token做验证,因此我希望websocket建立连接的时候也带上token

1. sockjs与stomp实现建立websocket连接
2. 通过connect方法建立连接,第一个参数带上自定义头部信息

```java
var ws = new SockJS('http://127.0.0.1:8090/websocket')
client = Stomp.over(ws)
// client = Stomp.client('ws://127.0.0.1:8090/websocket/websocket')
// 建立连接
client.connect(headers,
	function(frame) {
		console.log('success',frame);
        // 订阅方法
		sub()
	},
	function(error) {
		console.log('error',error);
})
        
function setHeaders() {
	var from = $('#from').val()
	headers = {
		'token': from
	}   
}
```

### 实现服务端监听websocket的连接,订阅等事件

通过该类从`event`中获取`NativeHeader`,其中包含了我们从客户端携带的自定义头部信息

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

### 实现判断用户websocket连接情况,用于处理在线消息和离线消息

通过监听服务端的`SessionConnectEvent`事件来记录用户上线了,监听`SessionDisconnectEvent`事件来记录用户下线了.

因为前端的stompjs调用disconnect事件无法携带自定义头部发送到服务端.

因此只能通过websocket连接的simpSessionId来记录用户与连接的映射关系.



首先建立一个工具类用于记录用户与socket连接的映射关系

```java
/**
 * 用户session记录类
 * @date 2020/4/10
 */
public class SocketSessionRegistry {

    private static final Logger logger = LoggerFactory.getLogger(SocketSessionRegistry.class);

    /**
     * 集合存储用户名和simpSessionId
     */
    private static final ConcurrentHashMap<String,String> userSession = new ConcurrentHashMap<>();

    /**
     * 根据用户获取simpSessionId
     * @param user 用户id
     * @return 用户的simpSession
     */
    public static String getSessionId(String user) {
        return SocketSessionRegistry.userSession.get(user);
    }

    /**
     * 用户id记录sessionId
     * @param user 用户id
     * @param sessionId simpSessionId
     */
    public static void registerSessionId(String user,String sessionId) {
        if(user != null && !user.isEmpty() && sessionId != null && !sessionId.isEmpty()) {
            SocketSessionRegistry.userSession.put(user,sessionId);
        } else {
            SocketSessionRegistry.logger.warn("register session fail ===> user or sessionId is null");
        }
    }

    /**
     * 根据session删除用户记录
     * @param sessionId
     */
    public static void removeSessionId(String sessionId) {
        if(sessionId != null && !sessionId.isEmpty()) {
            SocketSessionRegistry.userSession.entrySet().stream()
                    .filter(entry -> entry.getValue().equals(sessionId))
                    .forEach(entry -> {
                        SocketSessionRegistry.userSession.remove(entry.getKey());
                    });
        } else {
            SocketSessionRegistry.logger.warn("remove session fail ===> sessionId is null");
        }
    }

    /**
     * 获取记录
     * @return
     */
    public static ConcurrentHashMap<String,String> getUserSession() {
        return SocketSessionRegistry.userSession;
    }

}
```

在websocket连接事件触发后,将头部中的token信息与socket连接id做映射关系存入map中,代表用户上线了.

```java
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
```

在websocket断开连接事件触发后,将头部中的token信息与socket连接id做映射关系从map中删除,代表用户下线了.

```java
@Override
    public void onApplicationEvent(SessionDisconnectEvent event) {
        using(event,(user,session) -> {
            log.info("{}<===>{},disconnect",user,session);
            SocketSessionRegistry.removeSessionId(session);
        });
    }
```

### 处理在线消息和离线消息

- 首先根据消息的目的用户id去判断该用户是否在线
- 在线消息直接通过正常的websocket方式推送出去即可
- 离线消息首先将消息存入数据库中,当每次有用户上线时,且触发了订阅事件后,从数据库中查询该用户的离线消息,推送出去

```java
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
```

触发订阅事件,将离线消息推送给用户

```java
@Override
    public void onApplicationEvent(SessionSubscribeEvent event) {
        using(event,(user,session) -> {
            log.info("{}<===>{},subscribe",user,session);
            messagingTemplate.convertAndSend(
                    "/queue/chat/"+user,
                    "你tm有离线消息啊啊啊啊啊啊"
            );
        });
    }
```

