title: spring websocket使用总结
author: 禾田
tags:
  - websocket
categories:
  - websocket
date: 2018-05-29 16:02:00
---
> 最近使用了下websocket，总结下~

## websocket原理概念

WebSocket是HTML5提供的一种全双工通信的协议，通常是浏览器（或其他客户端）与Web服务器之间的通信。这使得它适合于高度交互的Web应用程序，如及时通讯聊天等。  
相比较于Http的短连接，Websocket的作用是可以保持长连接，也就是服务器也能够向客户端发送信息。从而实现了全双工通讯。

具体原理可以参考：[阮一峰的websocket原理讲解](http://www.ruanyifeng.com/blog/2017/05/websocket.html)



## 实现websocket客户端的两种方式


### js实现

```javascript
<script type="text/javascript">
    var websocket = null;
    //判断当前浏览器是否支持WebSocket
    if ('WebSocket' in window) {
        //websocket协议格式用ws / wss(加密)开始，可以类比http协议
        websocket = new WebSocket("ws://localhost:8088/ws/api/audiostream?appkey=zzzz&lan=zh&trans=0");

        //连接成功建立的回调方法
        websocket.onopen = function () {
            //给ws服务端发送消息
            websocket.send("客户端链接成功");
            websocket.send("Hello");
        }

        //接收到消息的回调方法
        websocket.onmessage = function (event) {
            //处理接收到的消息
            doSomething(event.data);
        }
        
        //连接发生错误的回调方法
        websocket.onerror = function () {
            alert("WebSocket连接发生错误");
        };

       //连接关闭的回调方法
        websocket.onclose = function () {
        	alert("WebSocket连接关闭");
        }

        //监听窗口关闭事件，当窗口关闭时，主动去关闭websocket连接，防止连接还没断开就关闭窗口，server端会抛异常。
        window.onbeforeunload = function () {
            closeWebSocket();
        }
        
    }
    else {
        alert('当前浏览器 Not support websocket')
    }


    //处理消息
    function doSomething(innerHTML) {
        //......
    }
    //关闭WebSocket连接
    function closeWebSocket() {
        websocket.close();
    }
</script>
```

### java实现

```java

import org.java_websocket.client.WebSocketClient;
import org.java_websocket.drafts.Draft_17;
import org.java_websocket.WebSocket.READYSTATE;
import org.java_websocket.handshake.ServerHandshake;
public class WebSocketClientTest {
    public static WebSocketClient client;
    
    public static void main(String[] args) throws URISyntaxException, NotYetConnectedException, UnsupportedEncodingException, InterruptedException {
        // new Draft_17()一定要加上
        client = new WebSocketClient(new URI("ws://localhost:8088/ws/api/audiostream?appkey=zzzz&lan=zh&trans=0"), new Draft_17()) {

            @Override
            public void onOpen(ServerHandshake serverHandshake) {
                System.out.println("打开链接");
            }

            @Override
            public void onMessage(String arg0) {
                System.out.println("收到消息:" + arg0);
            }

            @Override
            public void onError(Exception arg0) {
                arg0.printStackTrace();
                System.out.println("发生错误已关闭");
            }

            @Override
            public void onClose(int arg0, String arg1, boolean arg2) {
                System.out.println("链接已关闭");
            }

            @Override
            public void onMessage(ByteBuffer bytes) {
                try {
                    System.out.println(new String(bytes.array(), "utf-8"));
                } catch (UnsupportedEncodingException e) {
                    e.printStackTrace();
                }
            }
        };

        client.connect();

        while (!client.getReadyState().equals(READYSTATE.OPEN)) {
            System.out.println("还没有打开");
        }
        
        System.out.println("打开了");
        
        //发送json文本信息
        sendTextMessage(JSON.toJSONString("hello"));
        //发送二进制信息
        sendByte("".getBytes());
    }

    public static void sendByte(byte[] bytes) {
        client.send(bytes);
    }

    public static void sendTextMessage(String jsonString) {
        client.send(jsonString);
    }
}
```

## 用spring websocket处理websocket客户端发送过来的信息

### 导入 maven jar包

```
<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-websocket</artifactId>
	<version>${spring.version}</version>
</dependency>
```

### 添加配置类

```
@Configuration
@EnableWebSocket
public class WebSocketConfig implements WebSocketConfigurer {

    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
        // 注册拦截器
        // 注册webSocket url和处理器映射关系
        registry.addHandler(audioStreamWebSocketHandler(), "/ws/audiostream")
                .addHandler(audioStreamOpenApiWebSocketHandler(), "/ws/api/audiostream")
                .addInterceptors(myAudioStreamInterceptor());
    }

    @Bean
    public AudioStreamWebSocketHandler audioStreamWebSocketHandler() {
        return new AudioStreamWebSocketHandler();
    }

    @Bean
    public AudioStreamOpenApiWebSocketHandler audioStreamOpenApiWebSocketHandler() {
        return new AudioStreamOpenApiWebSocketHandler();
    }

    @Bean
    public AudioStreamWebSocketHandshakeInterceptor myAudioStreamInterceptor() {
        return new AudioStreamWebSocketHandshakeInterceptor();
    }
}
```
### 配置拦截器和处理器

```
// 拦截器
public class AudioStreamWebSocketHandshakeInterceptor implements HandshakeInterceptor {

    private static final Logger LOGGER = Logger.getLogger(AudioStreamWebSocketHandshakeInterceptor.class);

    //握手前处理请求
    @Override
    public boolean beforeHandshake(ServerHttpRequest request, ServerHttpResponse response, WebSocketHandler wsHandler,
            Map<String, Object> attributes) throws Exception {
        if (request instanceof ServletServerHttpRequest) {
            ServletServerHttpRequest serverHttpRequest = (ServletServerHttpRequest) request;
            ServletServerHttpResponse serverHttpResponse = (ServletServerHttpResponse) response;
            // 转化为servlet request和response
            HttpServletRequest servletRequest = serverHttpRequest.getServletRequest();
            HttpServletResponse servletResponse = serverHttpResponse.getServletResponse();
            // 获取http session
            HttpSession httpSession = servletRequest.getSession();
            String uri = servletRequest.getRequestURI();
            if ("/ws/api/audiostream".equals(uri)) {
                String appkey = servletRequest.getParameter("appkey");
                String lan = servletRequest.getParameter("lan");
                String trans = servletRequest.getParameter("trans");
                if (StringUtils.isBlank(appkey) || StringUtils.isBlank(lan) || StringUtils.isBlank(trans)) {
                    LOGGER.error("直播ws开放接口传参错误！");
                    return false;
                }
                attributes.put("appkey", appkey);
                attributes.put("lan", lan);
                attributes.put("trans", trans);
            } else if ("/ws/audiostream".equals(uri)) {
                String liveId = servletRequest.getParameter("liveId");
                if (StringUtils.isBlank(liveId)) {
                    LOGGER.error("直播id不能为空！");
                    return false;
                }
                attributes.put("liveId", liveId);
            } else {
                return false;
            }
        }
        return true;
    }

    @Override
    public void afterHandshake(ServerHttpRequest request, ServerHttpResponse response, WebSocketHandler wsHandler,
            Exception exception) {
    }
}
```

```java
public class AudioStreamWebSocketHandler extends TextWebSocketHandler {

    // 监听连接状态
    @Override
    public void afterConnectionEstablished(WebSocketSession session) throws Exception {
        System.out.println("websocket建立连接成功")
    }

    // 处理接收到的文本信息
    @Override
    public void handleTextMessage(WebSocketSession session, TextMessage message) {
        if (!session.isOpen())
            return;
        System.out.println("收到的文本信息为: " + message.getPayload());
    }

    // 判断心跳，检查服务器状态
    @Override
    protected void handlePongMessage(WebSocketSession session, PongMessage message) throws Exception {
        if (!session.isOpen())
            return;
        super.handlePongMessage(session, message);
    }
    
    // 处理接收到的二进制信息
    @Override
    protected void handleBinaryMessage(WebSocketSession session, BinaryMessage message) {
        if (!session.isOpen())
            return;
        ByteBuffer byteBuffer = message.getPayload();
        byte[] data = byteBuffer.array();
        System.out.println("收到的二进制消息为: " + data)
    }

    // 监听连接异常信息
    @Override
    public void handleTransportError(WebSocketSession session, Throwable exception) throws Exception {
        if (session.isOpen()) {
            session.close();
        }
        System.out.println("websocket连接出错！");
    }

    // 监听连接关闭信息
    @Override
    public void afterConnectionClosed(WebSocketSession session, CloseStatus status) throws Exception {
        if (status.getCode() == MyCloseStatus.LIVE_REPEATABLE_ERROR.getCode()
                || status.getCode() == MyCloseStatus.LIVE_END_NORMALLY.getCode()
                || status.getCode() == MyCloseStatus.LIVE_STATUS_ERROR.getCode()) {
            return;
        }
        // 处理一些释放资源操作
        ..............
    }


    // 发送消息返回给客户端
    public void sendMessage(WebSocketSession session, String message) {
        if (session.isOpen()) {
            try {
                session.sendMessage(new TextMessage(message));
            } catch (IOException e) {
                LOGGER.error("发送消息失败", e);
            }
        }
    }
}
```