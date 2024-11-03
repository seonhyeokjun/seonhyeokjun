
# Server-Sent Events (SSE)

## 개요
Server-Sent Events(SSE)는 클라이언트가 서버로부터 실시간 이벤트를 받을 수 있도록 하는 HTML5 표준이다. 서버가 클라이언트로 데이터를 단방향으로 푸시하는 방식을 지원하며, 주로 서버에서 클라이언트로 지속적으로 업데이트되는 데이터에 적합하다.

## 학습 목적
회사에서 실시간 데이터 전송이 필요한 프로젝트를 진행하면서 SSE를 적용해보고자 한다. 이를 위해 다음과 같은 목표를 설정하였다.

- SSE의 기본 개념을 이해하고, 실시간 데이터 전송 방식의 원리를 학습한다.
- WebSocket 등 다른 실시간 데이터 전송 기술과의 차이점을 파악하고 적절한 상황에서의 활용 방법을 익힌다.
- Java Spring을 이용한 SSE 구현을 실습하여, 실제 애플리케이션에서 SSE를 적용할 수 있는 기반을 마련한다.

## SSE의 특징
- **단방향 통신**: 서버가 클라이언트로 데이터 이벤트를 전송할 수 있지만, 클라이언트가 서버로 이벤트를 전송하지는 않는다.
- **재연결 지원**: 연결이 끊어졌을 때 자동으로 재연결을 시도하여 클라이언트가 데이터를 계속 수신할 수 있다.
- **간단한 구현**: HTTP 기반이므로 WebSocket보다 설정이 간단하고, 브라우저에서 기본적으로 지원된다.

## 구현 방법 (Java Spring 예시)
Spring에서는 `SseEmitter` 클래스를 사용하여 SSE를 쉽게 구현할 수 있다. 예제는 다음과 같다.

```java
import org.springframework.web.servlet.mvc.method.annotation.SseEmitter;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class SseController {

    @GetMapping("/sse")
    public SseEmitter stream() {
        SseEmitter emitter = new SseEmitter();
        
        new Thread(() -> {
            try {
                for (int i = 0; i < 10; i++) {
                    emitter.send("Event " + i);
                    Thread.sleep(1000);
                }
                emitter.complete();
            } catch (Exception e) {
                emitter.completeWithError(e);
            }
        }).start();

        return emitter;
    }
}
```

### 설명
- `SseEmitter`를 사용해 서버가 클라이언트로 이벤트를 보낼 수 있다.
- 클라이언트는 JavaScript `EventSource` 객체로 SSE 스트림을 수신할 수 있다.

### 클라이언트 예제 (JavaScript)
```javascript
const eventSource = new EventSource('/sse');

eventSource.onmessage = function(event) {
    console.log("Received event:", event.data);
};

eventSource.onerror = function() {
    console.log("Connection error");
};
```

## SSE vs WebSocket
| 특성          | SSE                          | WebSocket                        |
|---------------|------------------------------|----------------------------------|
| 통신 방식     | 단방향 (서버 → 클라이언트)     | 양방향                            |
| 연결 유지 방식 | HTTP/1.1 표준 지원           | TCP 소켓 기반                    |
| 용도          | 실시간 알림, 주가, 날씨 등     | 채팅, 게임, 금융 거래 등           |
| 브라우저 지원 | 모든 주요 브라우저 지원        | 모든 주요 브라우저 지원           |

## 장단점
### 장점
- HTTP 기반이므로 방화벽과 프록시를 쉽게 통과한다.
- WebSocket보다 구현이 단순하며 유지보수가 용이하다.
- 자동 재연결 기능을 제공한다.

### 단점
- 단방향 통신만 지원하므로 양방향 통신이 필요한 경우 적합하지 않는다.
- HTTP/2를 지원하지 않아, HTTP/1.1의 연결 제한에 영향을 받을 수 있다.

## 결론
SSE는 서버가 실시간 데이터를 클라이언트에 푸시해야 하는 경우에 유용한 기술이다. 단방향 통신만 필요한 알림 시스템이나, 지속적인 데이터 업데이트가 필요한 애플리케이션에 적합하다.
다음 문서는 SSE를 프로젝트에 적용해 보면서 발생한 문제와 해결 방법을 다룰 것이다.
