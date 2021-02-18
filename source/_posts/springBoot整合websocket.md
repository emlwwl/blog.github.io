---
title: SpringBoot整合WebSocket
date: 2021-02-18 11:05:55
tags: Java
categories: 
- Java
--- 

> Websocket测试工具：http://www.jsons.cn/websocket/

### 一、基础概念以及实现需求

#### 1、概念

Websocket它的最大特点就是，服务器可以主动向客户端推送信息，客户端也可以主动向服务器发送信息，是真正的双向平等对话，属于服务器推送技术的一种。 

特点：

- 建立在 TCP 协议之上，服务器端的实现比较容易。 
- 数据格式比较轻量，性能开销小，通信高效。 
- 协议标识符是`ws`（如果加密，则为`wss`），服务器网址就是 URL。 
- ` ws://example.com:80/path`

<!--more-->

#### 2、需求

这里实现了一个小程序搜索设备的功能。由客户端向服务器建立连接，在url上拼接参数token，服务端检验token的有效性，检验通过则成功建立连接。连接后，客户端发送一条json数据，服务端检验参数后，开始推送搜索到的设备数据。

### 二、整合webSocket  

####  1、引入依赖

```
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-websocket</artifactId>
</dependency>
```

#### 2、新建消息处理器：webSocketServer

用处：接收来自客户端的请求。

```
package com.fangzhizun.websocket;

import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.JSONException;
import com.alibaba.fastjson.JSONObject;
import com.alibaba.fastjson.serializer.SerializerFeature;
import com.fangzhizun.common.utils.JWTUtils;
import com.fangzhizun.framework.utils.JsonUtils;
import com.fangzhizun.framework.utils.RequestUtils;
import com.fangzhizun.model.parm.YunHaiDeviceCallbackPARM;
import com.fangzhizun.service.IYunHaiCallbackService;
import lombok.SneakyThrows;
import lombok.extern.slf4j.Slf4j;
import org.apache.commons.text.StringEscapeUtils;
import org.springframework.context.annotation.Lazy;
import org.springframework.stereotype.Component;
import org.springframework.web.socket.CloseStatus;
import org.springframework.web.socket.PongMessage;
import org.springframework.web.socket.TextMessage;
import org.springframework.web.socket.WebSocketSession;
import org.springframework.web.socket.handler.TextWebSocketHandler;

import javax.annotation.Resource;
import java.io.IOException;
import java.util.Map;
import java.util.Objects;
import java.util.concurrent.CopyOnWriteArraySet;

/**
 * @Author: willivie
 * @Description: 消息处理器，用处接收来自客户端的请求
 * 需要继承AbstractWebSocketHandler这个抽象类来实现自己的自定义消息处理器
 * TextWebSocketHandler是用于处理文本消息处理器，也是AbstractWebSocketHandler的派生类
 * 将WebSocketServer注册成spring的一个Bean
 * @Date: 2020/10/21 11:37
 */
@Component
@Slf4j
@Lazy
public class WebSocketServer extends TextWebSocketHandler {
    /**
     * concurrent包的线程安全Set，用来存放每个客户端对应的MyWebSocket对象。
     **/
    private static CopyOnWriteArraySet<WebSocketSession> webSocketSet = new CopyOnWriteArraySet<>();

    @Resource
    private IYunHaiCallbackService yunHaiCallbackService;

    /**
     * 群发自定义消息
     **/
    @SneakyThrows
    public void sendInfo(String message, String co, Integer uid) {
        log.info("推送消息到窗口" + co + "，推送内容:" + message);
        //这里处理根据具体业务场景推送不同数据
        log.info("当前webSocketSet容量:"+webSocketSet.size());
        for (WebSocketSession item : webSocketSet) {
            String tokenTo = getWsSesionToken(item);
            String coTo = JWTUtils.getCo(tokenTo);
            Integer uidTo = JWTUtils.getUid(tokenTo);
            //这里可以设定只推送给这个token的，为null则全部推送
            if (Objects.isNull(co) || Objects.equals(co,coTo) && Objects.isNull(uid)) {
                log.info("群体推送");
                item.sendMessage(new TextMessage(message));
            } else if (Objects.equals(co,coTo) && Objects.equals(uid,uidTo)){
                log.info("单人推送");
                item.sendMessage(new TextMessage(message));
            }
        }
    }

    /**
     * 来自客户端的消息在此处理
     **/
    @SneakyThrows
    @Override
    public void handleTextMessage(WebSocketSession session, TextMessage message) {
        log.info("客户端消息: {}", message.getPayload());
        try {
            JSONObject object = JSON.parseObject(message.getPayload());
            String action = object.getString("action");
            switch (action) {
                case "searchDevice":
                    String devSn = object.getString("sn");
                    Integer finish = object.getInteger("finish");
                    String co = JWTUtils.getCo(getWsSesionToken(session));
                    if (Objects.isNull(devSn)) {
                        throw new IllegalStateException("搜索设备时，网关SN不能为空!");
                    }
                    YunHaiDeviceCallbackPARM parm = new YunHaiDeviceCallbackPARM();
                    parm.setCo(co);
                    parm.setSn(devSn);
                    parm.setFinish(finish);
                    String resp = yunHaiCallbackService.searchDevice(parm);
                    sendMessage(session, JSON.parseObject(resp));
                    break;
                case "templatePush":
                    //TODO 微信消息推送
                    sendInfo(object.toString(),object.getString("co"),object.getInteger("sid"));
                    break;
                case "deviceWarning":
                    //TODO 设备预警推送
                    sendInfo(object.toString(), object.getString("co"), null);
                    break;
                case "heartbeat":
                    //心跳包，维持连接
                    break;
                default:
                    throw new IllegalStateException("action不符合规则!");
            }
        } catch (JSONException e) {
            JSONObject data = JsonUtils.set(-1, "json格式异常!", e.getClass().getName() + "：" + e.getMessage());
            sendMessage(session, data);
        } catch (IllegalStateException e) {
            JSONObject data = JsonUtils.set(-1, e.getMessage(), null);
            sendMessage(session, data);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    /**
     * 消息推送
     */
    private void sendMessage(WebSocketSession session, JSONObject data) throws IOException {
        String messageBack = JSON.toJSONString(data, SerializerFeature.WriteMapNullValue);
        session.sendMessage(new TextMessage(messageBack));
        log.info("服务端推送数据 Data：{}", data);
    }

    /**
     * 在连接断后后会调用该方法进行回收处理
     **/
    @Override
    public void afterConnectionClosed(WebSocketSession session, CloseStatus status) {
        String token = getWsSesionToken(session);
        String co = JWTUtils.getCo(token);
        // 移除session
        webSocketSet.remove(session);
        log.info("断开客户端连接:{}", co);
    }

    /**
     * 在建立连接后调用此方法
     */
    @Override
    public void afterConnectionEstablished(WebSocketSession session) throws Exception {
        String token = getWsSesionToken(session);
        webSocketSet.add(session);
        log.info("有新窗口开始监听: {},当前在线人数: {}", JWTUtils.getCo(token) + JWTUtils.getUid(token), webSocketSet.size());
        //JSONObject set = JsonUtils.set(1, "connection succeeded!", null);
        //String message = JSON.toJSONString(set, SerializerFeature.WriteMapNullValue);
        //session.sendMessage(new TextMessage(message));
        super.afterConnectionEstablished(session);
    }

    /**
     * 获取当前连接对象的参数值
     *
     * @param session 当前会话
     * @return 参数值
     */
    private String getWsSesionToken(WebSocketSession session) {
        String parm = StringEscapeUtils.unescapeHtml4(Objects.requireNonNull(session.getUri()).getQuery());
        Map<String, String> map = RequestUtils.mapQueryString(parm);
        return map.get(CustomHandshakeInterceptor.TOKEN);
    }

    /**
     * 当链接发生异常后触发的方法，关闭出错会话的连接，和删除在Map集合中的记录
     */
    @Override
    public void handleTransportError(WebSocketSession session, Throwable exception) throws Exception {
        //判断当前的链接是否在继续，关闭连接
        if (session.isOpen()) {
            session.close();
        }
        webSocketSet.remove(session);
        log.info("连接出错,移除会话");
    }


    @Override
    protected void handlePongMessage(WebSocketSession session, PongMessage message) throws Exception {
        super.handlePongMessage(session, message);
    }

    @Override
    public int hashCode() {
        return super.hashCode();
    }

    @Override
    public boolean equals(Object obj) {
        return super.equals(obj);
    }
}
```

#### 3、新建握手处理器：CustomHandshakeInterceptor

用处：握手前的调用，可在这里进行请求的校验工作（如权限的校验）

```
package com.fangzhizun.websocket;

import com.fangzhizun.common.utils.JWTUtils;
import com.fangzhizun.framework.utils.RequestUtils;
import lombok.SneakyThrows;
import lombok.extern.slf4j.Slf4j;
import org.apache.commons.text.StringEscapeUtils;
import org.springframework.http.server.ServerHttpRequest;
import org.springframework.http.server.ServerHttpResponse;
import org.springframework.stereotype.Component;
import org.springframework.web.socket.WebSocketHandler;
import org.springframework.web.socket.server.HandshakeInterceptor;

import java.util.Map;

/**
 * @Author 吴威良
 * @Description: 握手处理器，用于客户端的握手请求
 * 需要实现HandshakeInterceptor接口并注册层spring的一个Bean
 * @Date: 2020/11/4 10:25
 */
@Slf4j
@Component
public class CustomHandshakeInterceptor implements HandshakeInterceptor {

    public final static String TOKEN = "token";

    /**
     * 握手前的调用，可在这里进行请求的校验工作（如权限的校验）
     **/
    @SneakyThrows
    @Override
    public boolean beforeHandshake(ServerHttpRequest serverHttpRequest, ServerHttpResponse serverHttpResponse,
                                   WebSocketHandler webSocketHandler, Map<String, Object> attributes) {
        String queryString = StringEscapeUtils.unescapeHtml4(serverHttpRequest.getURI().getQuery());
        Map<String, String> queryMap = RequestUtils.mapQueryString(queryString);
        queryMap.forEach((k, y) -> log.info("ws连接参数：" + k + " : " + y));
        if (queryMap.containsKey(TOKEN)) {
            String token = queryMap.get("token");
            return !JWTUtils.isExpired(token);
        }
        return false;
    }


    @Override
    public void afterHandshake(ServerHttpRequest serverHttpRequest, ServerHttpResponse serverHttpResponse, WebSocketHandler webSocketHandler, Exception e) {
        // 握手之后调用
    }
}
```

#### 4、新建配置类：WebSocketConfiguration

用处：注入消息处理器、拦截器，设置url路由

```
package com.fangzhizun.websocket;

import com.sun.istack.Nullable;
import lombok.extern.slf4j.Slf4j;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.scheduling.TaskScheduler;
import org.springframework.scheduling.concurrent.ThreadPoolTaskScheduler;
import org.springframework.web.socket.config.annotation.EnableWebSocket;
import org.springframework.web.socket.config.annotation.WebSocketConfigurer;
import org.springframework.web.socket.config.annotation.WebSocketHandlerRegistry;
import org.springframework.web.socket.server.standard.ServerEndpointExporter;

import javax.annotation.Resource;

/**
 * @Author: 吴威良
 * @Description:
 * @Date: 2020/10/21 11:55
 */
 //注意注解EnableWebSocket
@Configuration
@EnableWebSocket
@Slf4j
public class WebSocketConfiguration implements WebSocketConfigurer {

    /**
     * 注入握手拦截器
     **/
    @Resource
    private CustomHandshakeInterceptor customHandshakeInterceptor;
    /**
     * 注入消息处理器
     **/
    @Resource
    private WebSocketServer webSocketServer;

    /**
     * 注册webSocket处理器
     */
    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
        // 注册消息处理器，并使用 "/channel"作为处理器的标识，客户端连接路径使用"/channel"就把请求发送给指定的处理器
        registry.addHandler(webSocketServer, "/channel")
                // 允许跨域
                .setAllowedOrigins("*")
                // 注册拦截器
                .addInterceptors(customHandshakeInterceptor);
    }

    @Bean
    public ServerEndpointExporter serverEndpointExporter() {
        return new ServerEndpointExporter();
    }

    /**
     * 解决springboot + quartz + websocket (spring自带的websocket 模块)
     * 引起的冲突问题
     *
     * @return TaskScheduler
     */
    @Bean
    @Nullable
    public TaskScheduler taskScheduler() {
        ThreadPoolTaskScheduler threadPoolScheduler = new ThreadPoolTaskScheduler();
        threadPoolScheduler.setThreadNamePrefix("SockJS-");
        threadPoolScheduler.setPoolSize(Runtime.getRuntime().availableProcessors());
        threadPoolScheduler.setRemoveOnCancelPolicy(true);
        return threadPoolScheduler;
    }
}
```

#### 5、nginx配置反向代理

```
location wss/ {
            proxy_pass http://socket.example.com/;
            proxy_set_header Host $host:$server_port;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
}
```

注意配置关键部分在于HTTP的请求中多了如下： 

```
proxy_set_header Upgrade $http_upgrade;
proxy_set_header Connection $connection_upgrade;
proxy_http_version 1.1;
```

http1.1协议支持长连接，Upgrade和Connection这两个字段表示请求服务器升级协议为WebSocket。 