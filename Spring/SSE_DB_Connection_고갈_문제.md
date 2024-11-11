## 오류
SSE 기능을 만들면서 겪었던 DB Connection 고갈 문제에 대해 다룰려고 한다. <br>
사장님들의 실시간 주문접수 기능을 만들기 위해 SSE를 사용하였다. <br>
기능을 만들고 테스트를 진행하던 와중 DB Connection 고갈 문제가 발생하였다. <br>

## 문제 원인
SSE는 클라이언트와 서버 간의 연결을 유지하며, 서버에서 이벤트를 클라이언트로 전송한다. <br>
EventListener를 통해 이벤트를 수신하고, 이벤트를 전송하는 방식으로 구현하였다. <br>
이때, DB Connection을 사용하여 데이터를 조회하고 전송하였다. <br>
이로 인해 DB Connection이 고갈되어 서버가 더 이상 요청을 처리하지 못하는 문제가 발생하였다. <br>

## 왜 DB Connection이 고갈되었나?
DB Connection 을 반납해 주지 못하고 지속적으로 점유하는 것에 문제가 있었다.
```java
public void sendEvent() {
    try {
        List<Order> orders = orderService.getOrders();
        for (Order order : orders) {
            // DB Connection을 사용하여 데이터를 조회
            eventEmitter.send(order);
        }
    } catch (Exception e) {
        log.error("Failed to send event", e);
    }
}
```
위 코드에서 `orderService.getOrders()` 메서드를 통해 DB Connection을 사용하고 있다. <br>
이 메서드는 DB Connection을 사용한 후 반환하지 않고 지속적으로 점유하고 있었다. <br>
해당 메서드가 끝이나면 반납했어야 했지만 그렇지 않고 있는 상황이었다.

## 해결 방법
DB Connection을 사용한 후에는 반드시 반환해야 한다. <br>
보기에는 간단해 보이지만 이는 실수하기 쉬운 부분이다. <br>
해당 Service 메서드의 호출이 끝나면 반납하도록 OSIV를 비활성화 설정하였다. <br>
이를 비활성화 하게되면 Service 메서드의 호출이 끝나면 DB Connection을 반환하게 된다. <br>
```yaml
spring:
  jpa:
    open-in-view: false
```
이를 통해 DB Connection 고갈 문제를 해결할 수 있었다.