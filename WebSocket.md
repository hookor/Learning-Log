참고자료: 
1. https://velog.io/@guswns3371/WebSocket-and-Socket.IO
1. https://mingule.tistory.com/60
2. https://recipes4dev.tistory.com/153
# WebSocket
1. **"소켓(Socket)"** 은 사전적으로 *"구멍", "연결", "콘센트"* 등의 의미

2. 프로그램이 네트워크에서 데이터를 송수신할 수 있도록, *"네트워크 환경에 연결할 수 있게 만들어진 연결부"*가 바로 **"네트워크 소켓(Socket)"**

3. 전기 소켓이 전기를 공급받기 위해 **정해진 규격(110V, 220V 등)** 에 맞게 만들어져야 하듯, 네트워크에 연결하기 위한 소켓 또한 **정해진 규약**, 즉, 통신을 위한 **프로토콜(Protocol)** 에 맞게 만들어져야함.
OSI 7 Layer(Open System Interconnection 7 Layer)의 네 번째 계층인 **TCP(Transport Control Protocol)** 상에서 동작하는 소켓을 주로 사용하는데, 이를 "TCP 소켓" 또는 "TCP/IP 소켓"(참고: https://shlee0882.tistory.com/110)

4. 소켓(Socket)으로 네트워크 통신 기능을 구현하기 위해서 다음 과정을 필요로 함.
	1. 소켓을 **만드는 것** 
	1. 만들어진 소켓을 통해 **데이터를 주고 받는 절차**에 대한 이해 
	1. 운영체제 및 프로그래밍 언어에 종속적으로 제공되는 **소켓 API 사용법***을 숙지

5. *두 개의 시스템(또는 프로세스)* 이 소켓을 통해 **데이터 통신을 위한 연결(Connection)** 을 만들기 위해서는, 연결 요청을 보내는지 또는 요청을 받아들이는지에 따라 소켓의 역할이 나뉘게 되는데, 전자에 사용되는 소켓을 **클라이언트 소켓(Client Socket)**, 후자에 사용되는 소켓을 **서버 소켓(Server Socket)**
두 소켓(Socket)은 **!! 동일 !!**, 소켓의 역할과 구현 절차 구분을 위해 다르게 부르는 것일 뿐, 전혀 다른 형태의 소켓이 아님. 단지 **역할에 따라 처리되는 흐름**, 즉, 호출되는 *API 함수의 종류와 순서들*이 다를 뿐.
소켓 연결이 완료된 다음 클라이언트 소켓과 서버 소켓이 직접 데이터를 주고 받는다고 생각X. <u>서버 소켓은 클라이언트 소켓의 연결 요청을 받아들이는 역할만 수행할 뿐, 직접적인 데이터 송수신은 서버 소켓의 연결 요청 수락의 결과로 만들어지는 새로운 소켓을 통해 처리.</u>

6. 클라이언트 소켓(Client Socket)은 처음 소켓(Socket)을 **[1]생성(create)** **=>** 서버 측에 **[2]연결(connect)** 을 요청 **=>** 그리고 서버 소켓에서 연결이 받아들여지면 데이터를 **[3]송수신(send/recv)** **=>** 모든 처리가 완료되면 소켓(Socket)을 **[4]닫습니다(close).**

7. 일단 클라이언트와 마찬가지로, 첫 번째 단계는 소켓(Socket)을 **[1]생성(create)** 하는 것 **=>** 그리고 서버 소켓이 해야 할 두 번째 작업은, <u> 서버가 사용할 IP 주소와 포트 번호</u>를 생성한 소켓에 **[2]결합(bind)** **=>**  클라이언트로부터 연결 요청이 수신되는지 **[3]주시(listen)** **=>** 요청이 수신되면 요청을 **[4]받아들여(accept)** 데이터 통신을 위한 소켓을 생성 **=>** 일단 새로운 소켓을 통해 *연결이 수립(ESTABLISHED)* 되면, 클라이언트와 마찬가지로 데이터를 **[5]송수신(send/recv)** 가능. **=>** 마지막으로 데이터 송수신이 완료되면, 소켓(Socket)을 **[6]닫습니다(close)**.
![enter image description here](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https://t1.daumcdn.net/cfile/tistory/995C23465C7DD7E30B)

## "만들고, 연결하고, 주고받고, 닫는다." - Client
### 1 클라이언트 소켓 생성. (socket())

1. 최초 소켓이 만들어지는 시점에는 어떠한 "연결 대상"에 대한 정보도 들어 있지 않음 **(껍데기 뿐인 소켓)**   **=>** 
2. `연결 대상(IP:PORT)` 을 지정하고 연결 요청을 전달하기 위해서는, 여기서 **생성한 소켓**을 사용하여 `connect()` API를 호출.

### 2 연결 요청. (connect())

1. `connect()`  API는 `"IP주소"와 "포트 번호"로 식별되는 대상(Target)`으로 연결 요청.

1. `connect()`  API는 **블럭(Block) 방식으로 동작** (<u>연결 요청에 대한 결과(성공, 거절, 시간 초과 등)가 결정되기 전에는  connect()의 실행이 끝나지 않음</u>) 
그러므로  `connect()`  API가 실행되자마자  **=>** 실행 결과와 관계없이 무조건 바로 리턴될 것이라 가정 **X**.

1. `connect()`  **API 호출이 성공**하면  **=>**  `send()`  /  `recv`  API를 통해 데이터를 송수신 가능.

### 3 데이터 송수신. (send()/recv())

1. 연결된 소켓을 통해 데이터를 보낼 때는  `send()`, 데이터를 받을 때는  `recv()`  API를 사용. 
1. `send()`와  `recv()`  API가 모두 **블럭(Block) 방식으로 동작** (<u>두 API 모두 실행 결과(성공, 실패, 종료)가 결정되기 전까지는 API가 리턴되지 않음</u>). 
1. 특히  `recv()`는 데이터가 수신되거나, 에러가 발생하기 전에는 실행이 종료되지 않음.  **=>** 데이터 수신 작업을 생각만큼 단순하게 처리하기 쉽지 않음.

1. `send()`의 경우 데이터를 보내는 주체가 자기 자신이기 때문에, 얼마만큼의 데이터를 보낼 것인지, 언제 보낼 것인지를 알 수 있지만, 데이터를 수신하는 경우, **통신 대상이 언제, 어떤 데이터를 보낼 것인지를 특정할 수 없기 때문에**  `recv()`  API가 한번 실행되면 언제 끝날지 모르는 상태가 됨.

 1. 따라서 데이터 수신을 위한  `recv()`  API는 별도의 스레드에서 실행.  **=>** 소켓의 **생성과 연결이 완료**된 후, `새로운 스레드`를 하나 만든 다음 그곳에서  `recv()`를 실행하고 데이터 수신대기.  

1. `send()`  /  `recv()`  API를 통해 **데이터 송수신 과정이 완료**되면,  `close()`  API를 사용하여 소켓을 닫음.

### 4 소켓 닫기. (close())

1. 더 이상 데이터 송수신이 필요없게되면, 소켓을 닫기 위해  `close()`  API를 호출. **=>**  `close()`에 의해 닫힌 소켓은 더 이상 유효한 소켓 **X**, 해당 소켓을 사용하여 데이터를 송수신 불가.

2. 만약 소켓 연결이 종료된 후 또 다시 데이터를 주고 받고자 한다면, 또 한번의 `소켓 생성(socket())과 연결(connect()) 과정`을 통해, 소켓이 데이터를 송수신할 수 있는 상태가 되어야 함.

## "+ bind, listen, accept" - Server
### 1  서버 소켓 생성. (socket())
클라이언트 소켓과 마찬가지로, 서버 소켓을 사용하려면 최초에 소켓을 생성

### 2 서버 소켓 바인딩. (bind())
bind() API에 사용되는 인자는 `소켓(Socket)과 포트 번호(또는 IP 주소+포트 번호)` **=>** *소켓(Socket)과 포트 번호를 결합(bind)*

1. 보통 시스템에는 많은 수의 프로세스가 동작. 그 중 네트워크 관련 기능을 수행하는 프로세스도 다수 포함, 만약 프로세스가 TCP 또는 UDP 프로토콜을 사용한다면, TCP 또는 UDP 표준에 따라, 각 소켓은 시스템이 관리하는 포트(0~65535) 중 하나의 포트 번호를 사용. 
2. 그런데 만약 소켓이 사용하는 포트 번호가 다른 소켓의 포트 번호와 중복된다면? 모든 소켓이 10000 이라는 동일한 포트 번호를 사용하게 된다면, 네트워크를 통해 10000 포트로 데이터가 수신될 때 **어떤 소켓이 처리해야 하는지** 결정할 수 없는 문제가 발생할 것입니다.
3. 운영체제에서는 소켓들이 <u>중복된 포트 번호를 사용하지 않도록</u>, 내부적으로 **포트 번호와 소켓 연결 정보**를 관리. **=>** 
4. `bind()`  API는 해당 소켓이 지정된 포트 번호를 사용할 것이라는 것을 운영체제에 요청하는 API. 만약 <u>지정된 포트 번호를 다른 소켓이 사용하고 있다면</u>,  `bind()`  API는 **에러**를 리턴.

서버 소켓은 *고정된 포트 번호*를 사용 **=>** 그리고 그 포트 번호로 클라이언트의 연결 요청을 수신 **=>** 그래서 운영체제가 <u>**특정 포트 번호를 서버 소켓이 사용하도록 만들기 위해 소켓과 포트 번호를 결합(bind)**</u>해야 하는데, 이 때 사용하는 API가 바로 *bind*.

### 3 클라이언트 연결 요청 대기. (listen())

`listen()`  API는 서버 소켓(Server Socket)에 `바인딩된 포트 번호`로 <u>**클라이언트의 연결 요청이 있는지 확인하며 대기 상태**</u>에 머무름. *클라이언트에서 호출된 `connect()` API*에 의해 연결 요청이 수신되는지 귀 기울이고 있다가, 요청이 수신되면, 그 때 대기 상태를 종료하고 리턴.

`listen()`  API가 대기 상태에서 빠져나오는 경우는 크게 두 가지. 
1. 클라이언트 요청이 **수신**되는 경우 
2. **에러**가 발생(소켓 `close()` 포함)하는 경우 

그런데  `listen()`  API가 성공한 경우라도, **리턴 값에 클라이언트의 요청에 대한 정보는 들어 있지 않음.**  `listen()`의 리턴 값으로 판단할 수 있는 것은 *클라이언트 연결 요청이 수신되었는지(SUCCESS), 그렇지 않고 에러가 발생했는지(FAIL)* 뿐.

대신 클라이언트 연결 요청에 대한 정보는 시스템 내부적으로 관리되는 큐(Queue)에서 쌓이게 되는데, <u>**이 시점에서 클라이언트와의 연결은 아직 완전히 연결되지 않은(not ESTABLISHED state) 대기 상태**</u>.

  

대기 중인 연결 요청을 큐(Queue)로부터 꺼내와서, **연결을 완료하기 위해서는  `accept()`  API**를 호출.

### 4 클라이언트 연결 수립. (accept())

 `listen()`  API가 클라이언트의 연결 요청을 확인하고 문제없이 리턴한다고 해서, 클라이언트와의 연결 과정이 모두 완료되는 것은 아님. 아직 **실질적인 소켓 연결(Connection)을 수립하는 절차**가 남아 있음. <u>**최종적으로 연결 요청을 받아들이는 역할을 수행하는 것은  `accept()`  API**</u> .

`accept()`  API는 그 사전적 의미만큼 직관적인 역할을 수행 **=>** 연결 요청을 받아들여(accept) 소켓 간 연결을 수립. 
그런데 주의할 점은 <u>**최종적으로 데이터 통신을 위해 연결되는 소켓이, 앞서  `bind()`  또는  `listen()`  API에서 사용한 `서버 소켓(Server Socket)`이 아니라는 것**</u>.

최종적으로 <u>**클라이언트 소켓(Client Socket)과 연결(Connection)이 만들어지는 소켓(Socket)은 앞서 사용한 `서버 소켓(Server Socket)`이 아니라,  `accept()`  API 내부에서 `새로 만들어지는 소켓(Socket)`**</u>.  

1. 서버 소켓(Server Socket)의 핵심 역할은 **클라이언트의 연결 요청을 수신하는 것**. 
2. 이를 위해  `bind()`  및  `listen()`을 통해 소켓에 포트 번호를 바인딩하고 요청 대기 큐를 생성하여 **클라이언트의 요청을 대기**.
3.  그리고  `accept()`  API에서, 데이터 송수신을 위한 <u>**새로운 소켓(Socket)**</u>을 만들고 <u>서버 소켓의 대기 큐에 쌓여있는 첫 번째 연결 요청을 매핑</u>. 여기까지, 하나의 연결 요청을 처리하기 위한 **서버 소켓의 역할은 끝**. 서버 소켓의 입장에서 남은 일은, 또 다른 연결 요청을 처리하기 위해 *다시 대기(listen)하거나, 서버 소켓(Socket)을 닫는(close) 것 뿐*.

*실질적인 데이터 송수신*은  `accept()`  API에서 생성된, **연결(Connection)이 수립(Established)된 소켓(Socket)을 통해 처리.**

### 5. 데이터 송수신. (send()/recv())

데이터를 송수신하는 과정은 클라이언트 소켓 처리 과정에서 설명했던 내용과 동일

### 6 소켓 연결 종료. (close())

클라이언트 소켓 처리 과정과 마찬가지로 소켓을 닫기 위해서는  `close()`  API를 호출.

그런데 **서버 소켓에서는  `close()`의 대상이 하나만 있는 것이 아니라는 것에 주의**.  최초  <u>`socket()`  API를 통해 생성한 **서버 소켓**</u>에 더해,  <u>`accpet()`  API 호출에 의해 **생성된 소켓**도 관리</u>해야한다는 의미.
