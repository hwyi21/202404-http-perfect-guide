# 커넥션 관리
# Overview

HTTP 애플리케이션을 개발하고 있다면 HTTP 커넥션과 그것이 어떻게 사용되는지에 대해 잘 이해하고 있어야 한다.

# TCP 커넥션

모든 HTTP 통신은, 지구상의 컴퓨터와 네트워크 장비에서 널리 쓰이고 있는 패킷 교환 네트워크 프로토콜들의 계층화된 집합인 TCP/IP를 통해 이루어진다.

일단 커넥션이 맺어지면 클라이언트와 서버 컴퓨터 간에 주고받는 메시지들은 손실 혹은 손상되거나 순서가 바뀌지 않고 안전하게 전달된다.

## 신뢰할 수 있는 데이터 전송 통로인 TCP

TCP는 HTTP에게 신뢰할 만한 통신 방식을 제공한다. TCP 커넥션의 한쪽에 있는 바이트들은 반대쪽으로 순서에 맞게 정확히 전달된다.

## TCP 스트림은 세그먼트로 나뉘어 IP 패킷을 통해 전송된다

TCP는 IP패킷 이라고 불리는 작은 조각을 통해 데이터를 전송한다. HTTP가 메시지를 전송하고자 할 경우, 현재 연결되어 있는  TCP 커넥션을 통해서 메시지 데이터의 내용을 순서대로 보낸다.

TCP는 세그먼트라는 단위로 데이터 스트림을 잘게 나누고, 세그먼트를 IP 패킷이라 불리는 봉투에 담아 인터넷을 통해 데이터를 전달한다.

## TCP 커넥션 유지하기

컴퓨터는 항상 TCP 커넥션을 여러 개 가지고 있다. TCP는 포트 번호를 통해서 이런 여러 개의 커넥션을 유지한다.

TCP 커넥션은 네 가지 값으로 식별한다. `<발신지 IP주소, 발신지 포트, 수신지 IP주소, 수신지 포트>`

이 네 가지 값으로 유일한 커넥션을 생성하며, 일부 구성요소가 같을 순 있지만 네 가지 모두가 같을 순 없다.

## TCP 소켓 프로그래밍

운영체제는 TCP 커넥션의 생성과 관련된 여러 기능을 제공한다. TCP API는, 기본적인 네트워크 프로토콜의 핸드셰이킹, 그리고 TCP 데이터 스트림과 IP 패킷 간의 분할 및 재조립에 대한 모든 세부사항을 외부로부터 숨긴다.

# TCP의 성능에 대한 고려

HTTP는 TCP 바로 위에 있는 계층이기 때문에 HTTP 트랜잭션의 성능은 그 아래 계층인 TCP 성능에 영향을 받는다.

## HTTP 트랜잭션의 지연

트랜잭션을 처리하는 시간은 TCP 커넥션을 설정하고, 요청을 전송하고, 응답 메시지를 보내는 것에 비하면 상당히 짧다. 클라이언트나 서버가 너무 많은 데이터를 내려받거나 복잡하고 동적인 자원들을 실행하지 않는 한, 대부분의 HTTP 지연은 TCP 네트워크 지연 때문에 발생한다.

1. DNS 이름 분석에 걸리는시간
2. TCP 커넥션 요청을 서버에 보내고 서버가 커넥션 허가를 하는 시간
3. HTTP 요청을 보내고 해당 요청을 서버가 처리하는데 까지 걸리는 시간
4. 웹 서버가 응답을 보내는 시간

## TCP 커넥션 핸드셰이크 지연

어떤 데이터를 전송하든 새로운 TCP 커넥션을 열 때면, TCP 소프트웨어는 커넥션을 맺기 위한 조건을 맞추기 위해 연속으로 IP 패킷을 교환한다. 이런 패킷 교환은 HTTP 성능을 크게 저하시킬 수 있다.

![image](https://github.com/Zero-ToHero/202404-http-perfect-guide/assets/71249347/2aa9253a-46b0-4ea6-8f96-60f8e302e736)

이러한 TCP 핸드셰이크는 크기가 작은 HTTP 트랜잭션의 경우 50% 이상의 시간을 TCP를 구성하는 데 쓴다.

## 확인응답 지연

인터넷 자체가 패킷 전송을 완벽히 보장하지 않기 때문에, TCP는 성공적인 데이터 전송을 보장하기 위해 자체적인 확인 체계를 가진다.

각 TCP 세그먼트는 순번과 데이터 무결성 체크섬을 가지고, 각 세그먼트의 수신자는 세그먼트를 온전히 받으면 확인응답 패킷을 송신자에게 반환한다.

확인응답은 크기가 작기 때문에, TCP는 같은 방향으로 송출되는 데이터 패킷에 확인응답을 편승시킨다. 즉, TCP는 송출 데이터 패킷과 확인응답을 하나로 묶음으로써 네트워크를 좀 더 효율적으로 사용한다.

확인응답이 같은 방향으로 가는 데이터 패킷에 편승되는 경우를 늘리기 위해서, 많은 TCP 스택은 `확인응답 지연` 알고리즘을 구현한다.

확인응답 지연은 송출할 확인응답을 특정 시간 동안 버퍼에 저장해 두고, 확인응답을 편승시키기 위한 송출 데이터 패킷을 찾는다. 만약 일정 시간 안에 송출 데이터 패킷을 찾지 못하면 확인응답은 별도 패킷을 만들어 전송된다.

HTTP 동작 방식은 요청과 응답 두 가지 형식으로만 이루어지기 때문에, 확인 응답이 송출 데이터 패킷에 편승할 기회를 감소시킨다. 편승할 패킷을 찾으려고 하면 해당 방향으로 송출될 패킷이 많지 않기 때문에, 확인응답 지연 알고리즘으로 인한 지연이 자주 발생한다.

## TCP 느린 시작

TCP의 데이터 전송 속도는 TCP 커넥션이 만들어진 지 얼마나 지났는지에 따라 달라질 수 있다. TCP 커넥션은 시간이 지나면서 자체적으로 튜닝되어서, 처음에는 커넥션의 최대 속도를 제한하고 데이터가 성공적으로 전송됨에 따라서 속도 제한을 높여나간다.

이러한 조율 방식을 TCP 느린 시작이라고 하며, 이는 급작스러운 부하와 혼잡을 방지하는 데 쓰인다.

TCP 느린 시작은 TCP가 한 번에 전송할 수 있는 패킷의 수를 제한한다.

이 기능 때문에, 새로운 커넥션은 이미 어느 정도 데이터를 주고받은 튜닝된 커넥션보다 느리다. 튜닝된 커넥션이 더 빠르기 때문에, HTTP에는 이미 존재하는 커넥션을 재사용하는 기능이 있다. -> **지속 커넥션**

## 네이글(Nagle)알고리즘과 TCP_NODELAY

TCP가 작은 크기의 데이터를 포함한 많은 수의 패킷을 전송할 경우 네트워크 성능이 크게 떨어진다.

네이글 알고리즘은 네트워크 효율을 위해, 패킷을 전송하기 전에 많은 양의 TCP 데이터를 한 개의 덩어리로 합치는 것이다. 기본적으로 세그먼트가 최대 크기가 되지 않으면 전송을 하지 않는다.

다만 이런 네이글 알고리즘은 문제가 있다.

- 크기가 작은 HTTP 메시지는 패킷을 채우지 못하기 때문에, 앞으로 생길지 모르는 추가적인 데이터를 기다리며 지연된다.
- 네이글 알고리즘이 확인응답 지연과 함께 쓰일 경우 부정적인 방향으로 시너지 효과가 나서 형편없이 동작한다. 네이글 알고리즘은 확인응답이 도착할 때까지 데이터 전송을 멈추는데, 확인응답 지연 알고리즘은 확인응답을 100~200밀리초 지연시킨다.

## TIME_WAIT의 누적과 포트 고갈

TCP 커넥션의 종단에서 TCP 커넥션을 끊으면, 종단에서는 커넥션의 IP 주소와 포트 번호는 메모리의 제어영역에 기록해 놓는다. 이 정보는 같은 주소와 포트 번호를 사용하는 새로운 TCP 커넥션이 일정 시간 동안에는 생성되지 않게 하기 위한 것으로, 보통 세그먼트의 최대 생명주기의 2배 정도의 시간동안 유지된다. 이는 이전 커넥션과 관련된 패킷이 그 커넥션과 같은 주소와 같은 포트번호를 가지는 새로운 커넥션에 삽입되는 문제를 방지한다.

TIME_WAIT 알고리즘은 특정 커넥션이 생성되고 닫힌 다음, 그와 같은 IP 주소와 포트 번호를 가지는 커넥션이 2분 이내에 또 생성되는 것을 막아준다.

# HTTP 커넥션 관리

## 순차적인 트랜잭션 처리에 의한 지연

순차적인 트랜잭션의 처리 방식은 물리적인 지연 뿐만 아니라, 하나의 이미지를 내려받고 있는 중에 웹 페이지의 나머지 공간에 아무런 변화가 없어서 느껴지는 심리적 지연도 있다.

또한 특정 브라우저의 경우 객체를 화면에 배치하려면 객체의 크기를 알아야 하기 때문에, 모든 객체를 내려받기 전까지는 텅 빈 화면을 보여준다는 것이다.

# 병렬 커넥션

HTTP는 클라이언트가 여러 개의 커넥션을 맺음으로써 여러 개의 HTTP 트랜잭션을 병렬로 처리할 수 있게 한다.

## 병렬 커넥션은 페이즈를 더 빠르게 내려받는다

단일 커넥션의 대역폭 제한과 커넥션이 동작하지 않고 있는 시간을 활용하면, 객체가 여러 개 있는 웹페이지를 더 빠르게 내려받을 수 있을 것이다.

하나의 커넥션으로 객체들을  로드할 때의 대역폭 제한과 대기 시간을 줄일 수 있다면 더 빠르게 로드할 수 있을 것이다.

## 병렬 커넥션이 항상 더 빠르지는 않다

병렬 커넥션이 일반적으로 더 빠르기는 하지만, 항상 그렇지는 않다. 클라이언트의 네트워크 대역폭이 좁을 때 각 객체를 전송받는 것은 느리기 때문에 성능상의 장점은 거의 없다.

또한 다수의 커넥션은 `메모리`를 많이 소모하고 서버는 다른 여러 사용자의 요청도 함께 처리해야 하기 떄문에 수백 개의 커넥션을 허용하는 경우는 드물다.

백명의 가상 사용자가 각각 100개의 커넥션을 맺고 있다면, 서버는 총 10,000개의 커넥션을 떠안게 되는 것이다.

## 병렬 커넥션은 더 빠르게 느껴질 수 있다.

병렬 커넥션이 페이지를 항상 더 빠르게 로드하지는 안는다. 다만 화면에 여러 개의 객체가 동시에 보이면서 내려받고 있는 상황을 볼 수 있기 때문에 사용자는 더 빠르게 내려받고 있는 것처럼 느낄 수 있다.

여러 작업이 일어나는 것을 눈으로 확인할 수 있으면 그것을 더 빠르다고 여긴다.

# 지속 커넥션

HTTP1.1을 지원하는 기기는 처리가 완료된 후에도 TCP 커넥션을 유지하여 앞으로 있을 HTTP 요청에 재사용할 수 있다. 처리가 완료된 후에도 계속 연결된 상태로 있는 TCP 커넥션을 `지속 커넥션` 이라고 부른다.

## 지속 커넥션 vs 병렬 커넥션

지속 커넥션은 병렬 커넥션에 비해 몇 가지 장점이 있다. 커넥션을 맺기 위한 사전 작업과 지연을 줄여주고, 튜닝된 커넥션을 유지하며, 커넥션의 수를 줄여준다. 하지만 지속 커넥션을 잘못 관리할 경우, 계속 연결된 상태로 있는 수많은 커넥션이 쌓이게 될 것이다. 이는 로컬의 리소스 그리고 원격의 클라이언트와 서버의 리소스에 불필요한 소모를 발생 시킨다.

## HTTP/1.0+의 Keep-Alive 커넥션

Keep-alive 커넥션의 성능상의 장점은 커넥션을 맺고 끊는 데 필요한 작업이 없어서 시간이 단축될 수 있다는 점이다.

## Keep-Alive의 동작

HTTP/1.0 keep-alive 커넥션을 구현한 클라이언트는 커넥션을 유지하기 위해서 요청에 `Connection:Keep-Alive` 헤더를 포함한다. 이 요청을 받은 서버는 그다음 요청도 이 커넥션을 통해 받고자 한다면, 응답 메시지에 같은 헤더를 포함시켜 응답한다. 만일 해당 헤더가 없다면, 클라이언트는 서버가 keep-alive를 지원하지 않으며, 응답 메시지가 전송되고 나면 서버 커넥션을 끊을 것이라 추정한다.

<aside>
⚠️ keep-alive는 사용하지 않기로 결정되어 HTTP/1.1 명세에서 빠졌다.

</aside>

## Keep-Alive 옵션

클라이언트나 서버가 keep-alive 요청을 받았다고 해서 무조건 그것을 따를 필요는 없다. 언제든 끊을 수 있으며, 트랜잭션의 수를 제한할 수도 있다.

Keep-Alive 헤더 사용은 선택 사항이지만, `Connection:Keep-Alive` 헤더가 있을 때만 사용할 수 있다.

```java
Connection: Keep-Alive
Keep-Alive: max=5, timeout=120
```

## Keep-Alive와 멍청한 프록시

### Connection 헤더의 무조건 전달

특히 문제는 프록시에서 시작되는데, 프록시는 Connection 헤더를 이해하지 못해서 해당 헤더들을 삭제하지 않고 요청 그대로를 다음 프록시에 전달한다. 오래되고 단순한 수많은 프록시들이 Connection 헤더에 대한 처리 없이 요청을 그대로 전달 한다.

결론 적으로 브라우저는 자신이나 서버가 타임아웃이 나서 커넥션이 끊길 때까지 기다리는 상황이 생긴다.

### 프록시와 홉별 헤더

이런 종류의 잘못된 통신을 피하려면, 프록시는 Connection 헤더와 Connection 헤더에 명시된 헤더들은 절대 전달하면 안 된다. 따라서 프록시가 Connection:Keep-Alive 헤더를 받으면 Connection 헤더뿐만 아니라 Keep-Alive란 이름의 헤더도 전달 하면 안 된다.

## Proxy-Connection 살펴보기

위와 같은 문제의 차선책으로 클라이언트의 요청이 중개서버를 통해 이어지는 경우 모든 헤더를 무조건 전달하는 문제를 해결하기 위해 `Proxy-Connection` 이라는 헤더를 사용하는 방법이 있다.(이는 웹 서버에서 무시될 것이다.)

다만 이러한 방법도 양옆에 이를 이해할 수 있는 영리한 프록시가 있을 경우 결국 문제가 발생한다.

### HTTP/1.1의 지속 커넥션

HTTP/1.1에서는 설계가 더 개선된 `지속 커넥션` 을 지원한다.

이는 기본적으로 활성화 되어 있으며, 별도 설정을 하지 않는 한, 모든 커넥션을 지속 커넥션으로 취급한다. HTTP/1.1 클라이언트는 `Connection:close` 헤더가 없을 경우 계속 유지하는 것으로 추정한다.

하지만 이는 클라이언트와 서버가 언제든지 끊을수 있는 것으로 영원히 유지하겠다는 것을 뜻하지는 않는다.

## 지속 커넥션의 제한과 규칙

- 클라이언트가 요청에 `Connection: close 헤더`를 보냈으면 클라이언트는 그 커넥션으로 추가적인 요청을 보낼 수 없다.
- 클라이언트가 해당 커넥션에 추가적인 요청을 보내지 않으면 마지막 요청에 `Connection: close 헤더`를 보내야한다.
- 모든 메시지가 `자신의 길이 정보를 정확히 갖고있을때만` 커넥션을 지속할 수 있다.
    - 정확한 Content-Length 값을 가지거나 청크 전송 인코등으로 인코드 되어있어야 한다.
- HTTP/1.1 프락시는 `클라이언트와 서버 각각`에 대해 별도로 `지속 커넥션`을 맺고 있어야한다.
- HTTP/1.1 프락시는 커넥션에 관해서 `클라이언트의 지원 범위`를 알고 있지 않는한 지속 커넥션을 맺으면 안된다.(현실적으로 쉽지않음, 많은 벤더가 이 규칙을 지키지 않음)
    - 오래된 프락시가 Connection 헤더를 전달하는 문제가 발생할 수 있음
- HTTP/1.1은 Connection 헤더와 상관없이 언제든 커넥션을 끊을 수 있다.
- HTTP/1.1 애플리케이션은 중간에 끊어지는 커넥션을 복구할 수 있어야한다.
    - 커넥션이 끊기는 경우 클라이언트는 요청을 다시 보낼 준비가 되어 있어야 한다.
- `서버 과부하 방지를 위해` 하나의 사용자 클라이언트는 넉넉잡아 `2개의 지속 커넥션만을 유지해야한다.`

# 파이프라인 커넥션

지속 커넥션을 통해 요청을 파이프라이닝 할 수 있으며, 이는 keep-alive 커넥션의 성능을 높여준다.

![image](https://github.com/Zero-ToHero/202404-http-perfect-guide/assets/71249347/083a10ee-f3eb-4ae7-aadd-809055e856ea)

- HTTP 클라이언트는 커넥션이 지속 커넥션인지 확인하기 전까지는 파이프라인을 이어서는 안된다.
- HTTP 응답은 요청 순서와 같게 와야한다.
    - HTTP 메시지는 순번이 없기 때문에 응답이 순서없이 오면 정렬시킬 방법이없다.
- HTTP 클라이언트는 커넥션이 끊기거나, 완료되지 않은 요청이 파이프라인에 있으면 다시 요청을 보낼 준비가 되어있어야한다.
    - 클라이언트가 10개의 요청을 보내더라도 서버는 5개의 요청만 처리하고 커넥션을 끊을 수도 있다.
- 비멱등 요청을 반복해서 파이프라인에 보내면 안된다.
    - 비멱등 요청을 여러번 보내는 경우, 에러 발생시 파이프라인을 통한 요청중에서 어떤 것들이 서버에 처리되었는지 알 방법이 없다.

# 커넥션 끊기에 대한 미스터리

## 마음대로 커넥션 끊기

보통 커넥션은 메시지를 다 보낸 다음 끊지만, 에러가 있는 상황에서는 헤더의 중간이나 다른 엉뚱한 곳에서 끊길 수 있다.

이 상황은 파이프라인 지속 커넥션에서와 같으며 애플리케이션은 언제든지 지속 커넥션을 임의로 끊을 수 있다.

서버가 유휴상태에 있는 커넥션을 끊는 시점에, 서버는 클라이언트가 데이터를 전송하지 않을 것이라고 확신하지 못한다. 만약 이 일이 벌어지면 클라이언트는 그요청 메시지를 보내는 도중 문제가 생긴다.

### Content-Length와 Trucation

각 HTTP 응답은 본문의 정확한 크기 값을 가지는 `Content-Length` 헤더를 가지고 있어야 한다. 만일 클라이언트나 프록시가 커넥션이 끊어졌다는 HTTP 응답을 받은 후, 실제 전달된 Entity의 길이와 Content-Length의 값이 일치하지 않거나 자체가 존재하지 않으면 수신자는 데이터의 정확한 길으를 서버에게 물어봐야 한다.

## 커넥션 끊기의 허용, 재시도, 멱등성

HTTP 애플리케이션은 예상치 못하게 커넥션이 끊어졌을 떄에 적절히 대응할 수 있는 준비가 되어 있어야 한다. 클라이언트가 트랜잭션을 수행 중에 전송 커넥션이 끊기게 되면, 클라이언트는 그 트랜잭션을 재시도 하더라도 문제가 없다면 커넥션을 다시 맺고 한 번 더 전송을 시도해야 한다

다만 이런경우 어떤 요청 데이터가 전송되었지만, 응답이 오기 전에 끊긴 경우 문제가 생긴다. 정적인 HTML 페이지를 GET 하는 요청은 문제가 없지만, 온라인 서점에 주문을 POST 하는 부류의 요청은 문제가 생긴다.

비멱등인 메서드나 순서에 대해 에이전트가 요청을 다시 보낼 수 있도록 기능을 제공한다 하더라도, 자동으로 재시도하면 안 된다. 예를 들어 대부분 브라우저는 캐시된 POST 요청 페이지를 다시 로드하려 하면, 다시 보내기를 원하는지 묻는 대화상자를 보여준다.

<aside>
⚠️ 멱등성
한 번 혹은 여러 번 실행됐는지에 상관없이 같은 결과를 반환 하는 것
(GET, HEAD, PUT, DELETE, TRACE 그리고 OPTION 등)

</aside>

## 우아한 커넥션 끊기

TCP 커넥션은 `양방향`으로 TCP 커넥션의 양쪽에는 데이터를 읽거나 쓰기 위한 입력 큐와 출력 큐가 있다. 한쪽 출력 큐에 있는 데이터는 다른 쪽의 입력 큐에 보내질 것이다.

### 전체 끊기와 절반 끊기

애플리케이션은 TCP 입력 채널과 출력 채널 중 한 개만 끊거나 둘 다 끊을 수 있다.

close() 호출하는 경우를 `전체 끊기` 라고 하며, shutdown() 을통해 `절반 끊기` 가가능하다.

### TCP 끊기와 리셋 에러

단순한 HTTP 애플리케이션은 전체 끊기만을 사용할 수 있다. 하지만 기기들에 예상치 못한 쓰기 에러를 발생하는 것을 예방하기 위한 `절반 끊기` 를 사용해야 한다.

보통은 커넥션의  출력 채널을 끊는 것이 안전하며, 클라이언트에서 더는 데이터를 보내지 않을 것임을 확신할 수 없는 이상, 커넥션의 `입력 채널`을 끊는 것은 위험하다.

### 우아하게 커넥션 끊기

이반적으로  애플리케이션이 우아한 커넥션 끊기를 구현하는 것은 애플리케이션 자신의 출력 채널을 먼저 끊고 다른 쪽에 있는 기기의 출력 채널이 끊기는 것을 기다리는 것이다.

안타깝게도 상대방이 절반 끊기를 구현햇다는 보장도 없고 절반 끊기를 했는지 검사해준다는 보장도 없다. 따라서 커넥션을 우아하게 끊고자 하는 애플리케이션은 출력 채널에 절반 끊기를 하고 난 후에도 데이터나 스트림의 끝을 식별하기 위해 입력 채널에 대해 상태 검사를 주기적으로 해야한다.
