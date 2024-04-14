---
title: "[네트워크] TCP, 3-way handshake & 4-way handshake"
date: 2023-11-01 12:00:00 +0900
categories: [CS & Tech-Interview, 네트워크]
tags: [network, tcp, 3-way handshake, 4-way handshake]
---

> 3-Way Handshake는 TCP의 연결,4-Way Handshake는 TCP의 연결 해제 과정이다.

## 3 way handshake - 연결

TCP는 연결 지향 프로토콜이다. 클라이언트와 서버가 1:1로 선로를 설정해, 순차적으로 데이터 통신을 한다. <br>순차적으로 통신함으로써 TCP는 정확하고 안전한 전송을 보장할 수 있다. <br>따라서 통신하기에 앞서, 선로 설정을 위해 3 way handshake 과정을 진행한다.

### 포트(PORT) 상태 정보

- **CLOSED**: 포트가 닫힌 상태
- **LISTEN**: 포트가 열린 상태로 연결 요청 대기 중
- **SYN_RCV**: SYNC 요청을 받고 상대방의 응답을 기다리는 중
- **ESTABLISHED**: 포트 연결 상태

### 플래그 정보

- TCP Header에는 CONTROL BIT(플래그 비트, 6bit)가 존재하고, 각각의 bit는 "URG-ACK-PSH-RST-SYN-FIN"를 뜻한다.
  <br>→ 즉, **해당 위치의 bit가 1이면 해당 패킷이 어떠한 내용을 담고 있는 패킷**인지를 나타낸다. <br>예로 SYN/ACK의 경우 010010
- **SYN**(Synchronize) / 000010
  - 연결 설정. Sequence Number를 랜덤으로 설정하여 세션을 연결하는 데 사용하며, 초기에 Sequence Number를 전송한다.
- **ACK**(Acknowledgement) / 010000
  - 응답 확인. 패킷을 받았다는 것을 의미한다.
  - Acknowledgement Number 필드가 유효한지를 나타낸다.
  - 양단 프로세스가 쉬지 않고 데이터를 전송한다고 가정하면 최초 연결 설정 과정에서 전송되는 첫 번째 세그먼트를 제외한 모든 세그먼트의 ACK 비트는 1로 지정된다. SYN 세그먼트 전송 이후(TCP 연결 시작 후) 모든 세그먼트에는 항상 이 비트가 1로 지정된다.
- **FIN**(Finish) / 000001
  - 연결 해제. 세션 연결을 종료시킬 때 사용되며, 더 이상 전송할 데이터가 없음을 의미한다.

### 작동방식

- Client는 Server와 연결하기 위해 3-way handshake를 통해 연결 요청을 한다.
- **SYN(Synchronization)** : 연결요청, 세션을 설정하는데 사용되며 초기에 시퀀스 번호를 보낸다.
- **ACK(Acknowledgement)** : 보낸 시퀀스 번호에 TCP 계층에서의 길이 또는 양을 더한 것과 같은 값을 ACK에 포함하여 전송한다.
  - 동기화 요청에 대한 답변 : `Client의 Sequence Number+1`을 하여 ACK로 돌려줍니다.

![3way-handshake.jpeg](/assets/img/3-way-handshake/3way-handshake.jpeg){: width="500" height="400"}

- **Step 1 (SYN)**
  client**는** server**와 연결하기 위해 SYN을 보낸다. (seq : x)**
  - 송신자가 최초로 데이터를 전송할 때 Sequence Number를 임의의 랜덤 숫자로 지정하고, SYN 플래그 비트를 1로 설정한 세그먼트를 전송한다.
  - PORT 상태
    - Client : ` CLOSED``SYN_SENT ` 로 변함
    - Server : `LISTEN`
- **Step 2 (SYN + ACK)**
  **server가 SYN(x)을 받고, client로 받았다는 신호인 ACK와 SYN 패킷을 보냄 (seq : y, ACK : x + 1)**
  - 접속요청을 받은 Q가 요청을 수락했으며, 접속 요청 프로세스인 P도 포트를 열어달라는 메세지를 전송 (SYN-ACK signal bits set)
  - ACK Number필드를 Sequence Number + 1 로 지정하고 SYN과 ACK 플래그 비트를 1로 설정한 새그먼트 전송 (`Seq=y, Ack=x+1, SYN, ACK`)
  - PORT 상태
    - Client : `CLOSED`
    - Server : `SYN_RCV`
- **Step 3 (ACK)**
  **client는 server의 응답은 ACK(x+1)와 SYN(y) 패킷을 받고, ACK(y+1)를 server로 보냄**
  - 마지막으로 접속 요청 프로세스 P가 수락 확인을 보내 연결을 맺음 (ACK)
  - 이때, 전송할 데이터가 있으면 이 단계에서 데이터를 전송할 수 있다.
  - PORT 상태
    - Client : `ESTABLISED`
    - Server : `SYN_RCV` ⇒ ACK ⇒ `ESTABLISED`

### full-duplex 통신의 구성

- Step 1, 2에서는 P→Q 방향에 대한 연결 파라미터(시퀀스 번호)를 설정하고 이를 승인한다.
- Step 2, 3에서는 Q→P 방향에 대한 연결 파라미터(시퀀스 번호)를 설정하고 이를 승인한다.

⇒ 이를 통해 full-duplex 통신이 구축된다.

## 4 way handshake - 연결 해제

- 4-Way Handshake은 연결을 **해제** (Connecntion **Termination**)하는 과정이다. 여기서는 FIN 플래그를 이용한다.
- `FIN` (finish) : 세션을 종료시키는데 사용되며, 더 이상 보낸 데이터가 없음을 나타낸다.

### Termination의 종류

TCP는 대부분의 connection-oriented 프로토콜과 같은 두 가지 연결 해제 방식이 있습니다.

1. **Graceful connection release(정상적인 연결 해제**)

   정상적인 연결 해제에서는 양쪽이 커넥션이 서로 모두 커넥션을 닫을 때까지 연결되어 있다.

2. **Abrupt connection release(갑작스런 연결 해제**)
   1. 갑자기 한 TCP 엔티티가 연결을 강제로 닫는 경우
   2. 한 사용자가 두 데이터 전송 방향을 모두 닫는 경우

### 작동방식 (Abrupt)

`RST(TCP reset)` 세그먼트가 전송되면 갑작스러운 연결 해제가 수행되는데, RST 세그먼트는 다음과 같은 경우에 전송된다.

1. 존재하지 않는 TCP 연결에 대해 비SYN 세그먼트가 수신된 경우
2. 열린 커넥션에서 일부 TCP 구현은 잘못된 헤더가 있는 세그먼트가 수신된 경우
   - RST 세그먼트를 보내, 해당 커넥션을 닫아 공격을 방지한다.
3. 일부 구현에서 기존 TCP 연결을 종료해야 하는 경우

- 연결을 지원하는 리소스 부족할때
- 원격 호스트에 연결할 수 없고 응답이 중지되었을때

### 작동방식 (Graceful)

연결 종료 요청 역시, 일반적으로 생각하는 우리가 일반적으로 생각하는 Client와 Server는 모두 서로 연결 요청을 먼저 할 수 있기 때문에, 연결 요청을 먼저 시도한 요청자를 Client로, 연결 요청을 받은 수신자를 Server쪽으로 생각하면 된다.

> ### Half-Close 기법
>
> 아래 그림에서 처음 보내는 종료 요청인 **(1) FIN** 패킷에 실질적으로 ACK가 포함되어 있는 것을 알 수 있는데, 이는**Half-Close 기법** 을 사용하기 때문이다.
>
> - 즉, 연결을 종료하려고 할 때 완전히 종료하지 않고 `반만 종료`합니다.
> - Half-Close 기법을 사용하면 종료 요청자가 처음 보내는 FIN 패킷에 승인 번호를 함께 담아서 보내게 되는데, 이때 승인 번호의 의미는 **"일단 연결은 종료할건데 귀는 열어둘게. 이 승인 번호까지 처리했으니까 더 보낼 거 있으면 보내"** 가 됩니다.
> - 이후 수신자가 남은 데이터를 모두 보내고 나면 다시 요청자에게 **FIN 패킷**을 보냄으로써 모든 데이터가 처리되었다는 신호 **(3) FIN**를 보냅니다. 그럼 요청자는 그때 나머지 반을 닫으면서 좀 더 안전하게 연결을 종료할 수 있게 됩니다.

![4way-handshake.jpeg](/assets/img/3-way-handshake/4way-handshake.jpeg){: width="500" height="400"}

- **STEP1 (Client → Server : FIN(+ACK)**
  - server와 client가 연결된 상태에서 client가 close()를 호출하여 접속을 끊을 것이라고 선포한다.
  - 이때, client는 server에게 연결을 종료한다는 `FIN` 플래그를 보낸다.
    - 이때 FIN 패킷에는 실질적으로 ACK도 포함되어있다.
- **STEP2 (Server → Client : ACK)**
  - Server는 FIN을 받고, 확인했다는 `ACK`를 Client에게 보내고 자신의 통신이 끝날때까지 기다린다. (이상태가 CLOSE_WAIT 상태)
    - Server(수신자)는 ACK Number 필드를 (Sequence Number + 1)로 지정하고, ACK 플래그 비트를 1로 설정한 세그먼트를 전송한다.
  - 서버는 클라이언트에게 응답을 보내고 **`CLOSE_WAIT` 상태**에 들어갑니다. 그리고**아직 남은 데이터가 있다면 마저 전송을 마친 후에 close( )를 호출**
  - 클라이언트에서는 서버에서 ACK를 받은 후에 서버가 남은 데이터 처리를 끝내고 FIN 패킷을 보낼 때까지 기다리게 됩니다. (**`FIN_WAIT_2`)**
- **STEP3 (Server → Client : FIN)**
  - 데이터를 모두 보냈다면, 서버는 연결이 종료에 합의 한다는 의미로 `FIN` 패킷을 클라이언트에게 보낸 후에, 승인 번호를 보내줄 때까지 기다리는 `LAST_ACK` 상태로 들어간다.
- **STEP4 (Client → Server : ACK)**
  - 클라이언트는 FIN을 받고, 확인했다는 `ACK`를 서버에게 보낸다.
  - 아직 서버로부터 받지 못한 데이터가 있을 수 있으므로 `TIME_WAIT` 을 통해 기다린다. (실질적인 종료과정 `CLOSED`에 들어가게 된다.)
    - 이때 `TIME_WAIT` 상태는 **의도치 않은 에러로 인해 연결이 데드락으로 빠지는 것을 방지**
    - 만약 에러로 인해 종료가 지연 되다가 타임이 초과되면 `CLOSED`로 들어갑니다.
- 서버는 ACK를 받은 이후 소켓을 닫는다 (Closed)
- TIME_WAIT 시간이 끝나면 클라이언트도 닫는다 (Closed)

이러한 3-way handshake와 4-way handshake는 TCP/IP 프로토콜 스택에서 안정적인 연결 설정 및 해제를 보장하며, 데이터 전송 중의 신뢰성을 제공합니다.
