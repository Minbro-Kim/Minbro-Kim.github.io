---
title: "[Spring Boot] Websocket(STOMP) 연결 종료"
excerpt: "서버에서 STOMP 연결 종료하기"

categories:
  - Spring Boot
tags:
  - [springboot, websocket, stomp]

permalink: /springboot/post-5/

toc: true
toc_sticky: true

date: 2026-07-08
last_modified_at: 2026-07-08
---

> 웹소켓 연결에서 클라이언트에 종속하지 않고 스프링부트 서버에서 임의로 연결을 종료하는 방법

---


## WebsocketSession

- 실제 네트워크 자원(TCP/IP 소켓 파이프라인 및 서블릿 컨테이너의 세션)을 래핑하고 있는 <mark>물리적 객체</mark>
- 연결 종료를 위해서는 `WebsocketSession`에 대한 `close()` 메서드 호출 필요

### WebsocketSession 저장 위치

- `SubProtocolWebsocketHandler` 클래스에서 `private WebsocketHolder`객체로 저장
- `WebsocketHolder` Map에 대한 `getter`가 없기 때문에 세션 아이디로도 외부에서 접근 불가

```java
@Override
public void afterConnectionEstablished(WebSocketSession session) throws Exception {
    // WebSocketHandlerDecorator could close the session
    if (!session.isOpen()) {
        return;
    }

    checkSessions();

    this.stats.incrementSessionCount(session);
    session = decorateSession(session);
    this.sessions.put(session.getId(), new WebSocketSessionHolder(session));
    findProtocolHandler(session).afterSessionStarted(session, this.clientInboundChannel);
}
```

---

## 해결 방안

### 1) WebsocketSession 별도 저장

- <mark>웹소켓 연결 시점에 WebsocketSession을 별도 메모리(attribute/map)에 저장</mark>
- HandShakeInterceptor 또는 WebsocketHandlerDecorator 활용
- 장점
  - 명시적 `close()` 메서드 호출 가능
  - 원하는 Key형태로 저장 가능
- 단점
  - 별도의 생명주기 관리 필요(생성/삭제)
  - 다중 서버 환경에서 구현 복잡도 상승

```java
@Component
public class WebsocketSessionManager {

  // 스프링이 세션 생명주기 흐름에 따라 add/remove를 호출
  private final Map<String, WebSocketSession> sessionMap = new ConcurrentHashMap<>();

  public void registerSession(WebSocketSession session) {
    sessionMap.put(session.getId(), session);
  }

  public void removeSession(String sessionId) {
    sessionMap.remove(sessionId);
  }

  // 물리 세션을 NORMAL(1000)으로 닫기 가능
  public void closeSession(String sessionId) {
    WebSocketSession session = sessionMap.get(sessionId);
    if (session != null && session.isOpen()) {
      try {
        session.close(CloseStatus.NORMAL); // 정상 종료 코드 발송
      } catch (IOException e) {
        // 예외 처리
      }
    }
  }
}
```

```java
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {
  // 그외 설정
  @Override
  public void configureWebSocketTransport(WebSocketTransportRegistration registration) {
    // 스프링 웹소켓 핸들러가 동작할 때 데코레이터를 입혀서 세션을 가로챔
    registration.addDecoratorFactory(new WebSocketHandlerDecoratorFactory() {
      @Override
      public WebSocketHandler decorate(WebSocketHandler handler) {
        return new WebSocketHandlerDecorator(handler) {

          @Override // 1. 웹소켓이 연결되어 세션 객체가 활성화되는 순간 추출
          public void afterConnectionEstablished(WebSocketSession session) throws Exception {
            sessionManager.registerSession(session);
            super.afterConnectionEstablished(session);
          }

          @Override // 2. 브라우저가 닫히거나 연결이 끊어지는 순간 자동 제거
          public void afterConnectionClosed(WebSocketSession session, CloseStatus closeStatus)
                  throws Exception {
            sessionManager.removeSession(session.getId());
            super.afterConnectionClosed(session, closeStatus);
          }
        };
      }
    });
  }
}
```

### 2) STOMP ERROR FRAME 활용

- `SubProtocolHandler`의 `SendToClient()`메서드에서 명령어가 `ERROR`인 경우 자동으로 세션 종료
- 종료가 필요한 시점에 `StompHeaderAccessor` + `clientOutboundChannel`을 통해 `StompCommand.ERROR` 메세지 전송
- 장점
  - 리소스 절약
- 단점
  - 웹소켓 종료 코드가 `1002 (PROTOCOL_ERROR)`로 고정
  - 생명주기 직접 관리 불가

```java
private void sendToClient(WebSocketSession session, StompHeaderAccessor stompAccessor, byte[] payload) {
    StompCommand command = stompAccessor.getCommand();
    try {
        byte[] bytes = this.stompEncoder.encode(stompAccessor.getMessageHeaders(), payload);
        boolean useBinary = (payload.length > 0 && !(session instanceof SockJsSession) &&
                MimeTypeUtils.APPLICATION_OCTET_STREAM.isCompatibleWith(stompAccessor.getContentType()));
        if (useBinary) {
            session.sendMessage(new BinaryMessage(bytes));
        }
        else {
            session.sendMessage(new TextMessage(bytes));
        }
    }
    catch (SessionLimitExceededException ex) {
        // Bad session, just get out
        throw ex;
    }
    catch (Throwable ex) {
        // Could be part of normal workflow (for example, browser tab closed)
        if (logger.isDebugEnabled()) {
            logger.debug("Failed to send WebSocket message to client in session " + session.getId(), ex);
        }
        command = StompCommand.ERROR;
    }
    finally {
        if (StompCommand.ERROR.equals(command)) {
            try {
                session.close(CloseStatus.PROTOCOL_ERROR);
            }
            catch (IOException ex) {
                // Ignore
            }
        }
    }
}

```
  
##### 예시

```java
@Component
@RequiredArgsConstructor
@Slf4j
public class WebsocketControlService {

  private final MessageChannel clientOutboundChannel;
  private final SimpUserRegistry userRegistry;
  private static final String FORCE_DISCONNECT_MESSAGE = "FORCE_CLOSE_BY_SERVER";

  public void sendForceLogoutSignal(String email) {

    SimpUser simpUser = userRegistry.getUser(email);

    if (simpUser == null) {
      return;
    }
    String message = "Disconnect By Logout";
    simpUser.getSessions().forEach(session -> sendErrorFrame(session.getId(), message));
  }

  private void sendErrorFrame(String sessionId, String payloadMessage) {
    try {
      StompHeaderAccessor accessor = StompHeaderAccessor.create(StompCommand.ERROR);

      accessor.setSessionId(sessionId);
      accessor.setMessage(FORCE_DISCONNECT_MESSAGE); // 에러 사유 메시지

      byte[] payload = payloadMessage == null ? new byte[0] : payloadMessage.getBytes(
          StandardCharsets.UTF_8);

      Message<byte[]> message = MessageBuilder.createMessage(
          payload,
          accessor.getMessageHeaders()
      );

      clientOutboundChannel.send(message);
    } catch (Exception e) {
      log.warn("Failed to send disconnect message. sessionId={}", sessionId, e);
    }
  }
}
```

![image-1](/assets/images/posts_img/springboot/spring-5-1.png)

```text
 o.s.w.s.s.t.s.WebSocketServerSockJsSession [ |  | ] - Closing SockJS session 4k4uuwa0 with CloseStatus[code=1002, reason=null]
 o.s.w.s.a.NativeWebSocketSession     [ |  | ] - Closing StandardWebSocketSession[id=8e3afcd9-7cc8-47c5-8ad4-ca6f281e4986, uri=ws://localhost:8080/ws/820/4k4uuwa0/websocket]
 o.s.w.s.h.LoggingWebSocketHandlerDecorator [ |  | ] - WebSocketServerSockJsSession[id=4k4uuwa0] closed with CloseStatus[code=1002, reason=null]
 o.s.w.s.m.SubProtocolWebSocketHandler [ |  | ] - Clearing session 4k4uuwa0
 c.t.m.w.l.WatchingSessionEventListener [ |  | ] - [EVENT TRACE] SessionDisconnectEvent 수신 완료: sessionId=4k4uuwa0
 c.t.m.w.l.WatchingSessionEventListener [ |  | ] - [EVENT TRACE] SessionDisconnectEvent 처리 완료: sessionId=4k4uuwa0
 o.s.m.s.b.SimpleBrokerMessageHandler [ |  | ] - Processing DISCONNECT session=4k4uuwa0
 o.s.w.s.m.SubProtocolWebSocketHandler [ |  | ] - No session for GenericMessage [payload=byte[0], headers={simpMessageType=DISCONNECT_ACK, simpDisconnectMessage=GenericMessage [payload=byte[0], headers={simpMessageType=DISCONNECT, stompCommand=DISCONNECT, simpSessionAttributes={org.springframework.messaging.simp.SimpAttributes.COMPLETED=true, userId=7a20eb19-7d52-4033-b66c-3e1003b701e6}, simpUser=UsernamePasswordAuthenticationToken [Principal=com.team6.moduply.auth.userdetails.ModuPlyUserDetails@4be2edaa, Credentials=[PROTECTED], Authenticated=true, Details=null, Granted Authorities=[ROLE_USER, ROLE_ADMIN]], simpSessionId=4k4uuwa0}], simpUser=UsernamePasswordAuthenticationToken [Principal=com.team6.moduply.auth.userdetails.ModuPlyUserDetails@4be2edaa, Credentials=[PROTECTED], Authenticated=true, Details=null, Granted Authorities=[ROLE_USER, ROLE_ADMIN]], simpSessionId=4k4uuwa0}]
 o.s.w.s.s.t.h.DefaultSockJsService   [ |  | ] - Closed 1 sessions: [4k4uuwa0]
```