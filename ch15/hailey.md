# 엔터티와 인코딩

## HTTP/2 에서의 인코딩
- HTTP/2 에서 전송 방식이 효율적으로 발전하였기 때문에 더이상 전송 인코딩을 사용하지 않음
### 이유
1. 바이너리 프로토콜을 사용
   - 데이터를 이진데이터 (0과1)의 형태로 표현하고 전송하는 프로토콜
2. 헤더 압축 (HPACK)
   - 중복된 헤더 정보를 줄이고, 네트워크 대역폭 사용을 최적화
3. 멀티플렉싱
 - 여러 스트림을 통해 다중 요청과 응답을 동시에 전송
4. 스트림 우선순위
    - 스트림에 우선 순위 지정을 통해 중요한 리소스를 먼저 처리 할 수 있음

### 전송 방식
- binary frame으로 인코딩되어 전송
- header 와 body 를 하나로 함쳐서 보냈던 1.1 버전에 비해 2.0 버전에서는 가장 작은 단위인 frame 단위로 나누어서 헤더와 데이터를 전송
<img width="788" alt="image" src="https://github.com/hwyi21/202404-http-perfect-guide/assets/58624211/8c7ce9df-c117-4e7c-a1bb-5e33bf2445fd">

프레임 단위의 요청/응답 메시지는 하나의 스트림을 통해 이뤄지고 이러한 스트림이 하나의 커넥션에서 병렬로 처리되기 때문에 속도가 빠름

<img width="736" alt="image" src="https://github.com/hwyi21/202404-http-perfect-guide/assets/58624211/4e61c5f1-d36c-4436-a9de-af1cb8556979">