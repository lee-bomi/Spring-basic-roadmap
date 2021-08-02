# 1. http(HyperText Transfer Protocol)

모든 것을 HTTP 메시지에 전송할 수 있다.

- HTML, TEXT, 이미지, 영상, JSON, XML
- 서버 간에 데이터 통신도 대부분 HTTP 사용

- **HTTP/1.1** 1997년: 가장 많이 사용, 우리에게 가장 중요한 버전
- HTTP/1.1, HTTP/2 : TCP위에서 동작
- HTTP/3 : UDP위에서 동작

### HTTP 특징

- 클라이언트 서버 구조
- 무상태 프로토콜(stateless), 비연결성(Connectionless)
- HTTP 메시지
-  단순함, 확장 가능

# 2. 클라이언트 서버 구조

클라이언트는 요청하고, 서버는 응답한다.

역할을 나누면, 독립적으로 진행할 수 있다.

# 3. Stateful, Stateless

### 무상태 프로토콜(Stateless)

- 서버가 클라이언트의 상태를 보존 X
- 장점 : scale-out(서버 확장) 
- 단점 : 클라이언트가 추가 데이터 전송

### Stateful, Stateless 차이

stateful : 항상 같은 서버가 유지되어야 한다.

stateless : 서버가 바뀌어도 된다. -> 클라이언트의 요청(트래픽)이 증가하면 서버를 대거 투입할 수 있다.

<img src="https://user-images.githubusercontent.com/38436013/127138575-1830038f-a95a-41e9-8f96-2602f5638d8f.png" alt="image" style="zoom:80%;" />
<img src="https://user-images.githubusercontent.com/38436013/127138598-dc49d4eb-981b-48ac-9d95-5a60a88cb4be.png" alt="image" style="zoom:80%;" />
<img src="https://user-images.githubusercontent.com/38436013/127138625-8fc4e4c1-bb37-4fec-b8ce-dfae94744119.png" alt="image" style="zoom:80%;" />
<img src="https://user-images.githubusercontent.com/38436013/127138656-803a0722-ab10-4ca3-8caa-297563fad15e.png" alt="image" style="zoom:80%;" />
<img src="https://user-images.githubusercontent.com/38436013/127138717-c4e724e8-e2db-40af-91fe-c7496a8ea8a5.png" alt="image" style="zoom:80%;" />

> 상태 유지의 경우,  서버가 상태 정보를 보관하기 때문에 서버에 장애가 생기면 다시 요청받아야 한다.
>
> 무상태의 경우, 클라는 필요한 요청을 다 담아서 요청을 보내므로, 서버 변경 가능하다. 스케일 아웃 가능하다.

### STATELESS의 한계

모든 것을 무상태로 설계 할 수는 없다.

예 : 로그인, 브라우저 쿠키, 서버 세션  등

# 4. 비 연결성(connectionless)

<img src="https://user-images.githubusercontent.com/38436013/127144261-b3dad783-162f-4050-a8a2-8485a5ab82cc.png" alt="image" style="zoom:80%;" />

<img src="https://user-images.githubusercontent.com/38436013/127144426-039ea5da-0ad0-48a8-bfc7-9bc48646481f.png" alt="image" style="zoom:80%;" />

- HTTP는 기본이 연결을 유지하지 않는 모델 -> 최소한의 자원 유지 
- 일반적으로 초 단위의 이하의 빠른 속도로 응답 
- 1시간 동안 수천명이 서비스를 사용해도 실제 서버에서 동시에 처리하는 요청은 수십개 이 하로 매우 작음 
- 서버 자원을 매우 효율적으로 사용할 수 있음

### 비 연결성 한계와 극복

- TCP/IP 연결을 새로 맺어야 함 - 3 way handshake 시간 추가
- 웹 브라우저로 사이트를 요청하면 HTML 뿐만 아니라 자바스크립트, css, 추가 이미지 등 등 수 많은 자원이 함께 다운로드 -> 매번 반복
- 지금은 HTTP 지속 연결(Persistent Connections)로 문제 해결

<img src="https://user-images.githubusercontent.com/38436013/127144921-dfd3e6c1-d759-479f-b198-b58ab0413e1e.png" alt="image" style="zoom:80%;" />

<img src="https://user-images.githubusercontent.com/38436013/127144898-bca4d438-e294-488a-84ec-59f82d397af9.png" alt="image" style="zoom:80%;" />

> 햇갈리짐 말기 ! stateful은 서버의 상태 유지를 말하고, Persistent connectionless는 요청-응답을 일정시간동안 유지해준다.

### 서버 개발자의 고충

같은 시간에 발생하는 대용량 트래픽 -> 최대한 stateless 연결을 하여 대응

