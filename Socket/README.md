# Socket 통신 스터디 정리

> [!NOTE]
> Socket 통신을 학습하며 스터디 중 논의한 내용을 정리한 문서입니다.

### 📌 Socket 통신이란?

**소켓(Socket)** 은 네트워크를 경유하는 프로세스 간 통신의 종착점이다.

OSI 7계층 중 응용 계층에 속하는 프로세스들은 데이터 송수신을 위해 반드시 소켓을 거쳐 전송 계층으로 데이터를 전달해야한다.

즉, 소켓은 전송 계층과 응용 프로그램 사이의 인터페이스 역할을 하며 떨어져 있는 두 호스트를 연결해준다.

**소켓 통신**이란 네트워크 상에서 서로 다른 컴퓨터나 프로세스 간에 데이터를 주고받기 위한 통신 방식이다.

소켓(Socket)은 네트워크 통신을 위한 종단점(endpoint) 역할을 하며 주로 TCP/IP 또는 UDP 프로토콜을 기반으로 한다.

<br>

### 📌 Socket의 필요성

프로세스는 직접 네트워크 장치에 접근할 수 없다. 오직 커널만이 장치와 통신할 수 있음!

따라서, 사용자 프로세스는 네트워크를 통해 다른 컴퓨터와 통신하려면 커널에게 요청할 수 있는 API가 필요. 이때 **커널과 소통하는 창구**가 소켓이다.

즉, 소켓은 “**사용자 프로세스와 커널 네트워크 스택 사이의 계약된 인터페이스**”

![1](https://github.com/user-attachments/assets/45b3c8d9-17ca-4250-ba9d-fc2c39211a05)

<br>

### 📌 Socket의 구성

| 개념 | 설명 |
| --- | --- |
| **IP 주소** | 통신할 컴퓨터의 주소 |
| **포트 번호** | 하나의 컴퓨터 내 여러 서비스 구분용 번호 |
| **소켓** | IP + 포트 번호로 구성된 통신 종단점 |
| **프로토콜** | 데이터 통신 규약 (TCP, UDP 등) |

<br>

### 📌 Socket 통신의 흐름 (TCP)

<img width="517" alt="2" src="https://github.com/user-attachments/assets/0d3399e2-b2e4-429e-87a3-b54e77be173c" />

- 서버 (Server)
    - Client 소켓의 연결 요청을 대기하고, 연결 요청이 오면 Client 소켓을 생성하여 통신이 가능하게 한다.
    1. `socket()` 함수를 이용하여 소켓 생성
    2. `bind()` 함수로 소켓에 IP와 Port 번호 설정
    3. `listen()` 함수로 Client 접속 요청을 대기
    4. `accept()` 함수를 사용하여 실제 연결 요청을 수락하고 새로운 소켓 반환

- 클라이언트 (Client)
    - 실제 데이터 송수신이 일어나는 곳
    1. `socket()` 함수로 소켓 생성
    2. `connect()` 함수를 이용하여 통신할 서버의 설정된 IP와 port 번호에 통신을 시도
    3. 통신 시도 시, 서버가 accpet() 로 요청을 받아들이면, 클라이언트와 연결된 소켓이 생성됨
    4. 이를 통해 클라이언트와 서버가 서로 `read()`, `write()` 또는 `recv()`, `send()` 하며 통신

<br>

### 📌 Socket 통신의 흐름 (UDP)

UDP 소켓 통신 과정은 조금 더 간결하다.

서버측에서 `socket()`을 통해 소켓을 생성하고 `bind()`로 IP와 포트 번호를 지정한 후 바로 `recvfrom()`과 `sendto()`를 통해 특정한 연결 없이 데이터 송수신이 가능하다.

클라이언트 측에서도 `socket()`을 통해 소켓을 생성한 뒤 `sendto()`와 `recvfrom()`을 통해 데이터 송수신을 진행한다.

<br>

### 📌 HTTP 통신과 소켓 통신

HTTP는 소켓 통신을 기반으로 한 애플리케이션 계층 프로토콜이다.

- 소켓 통신: 데이터를 보내고 받는 파이프 (저수준, TCP/UDP 사용)
- HTTP 통신: 그 파이프를 통해 무엇을 어떻게 주고받을지 정한 규칙 (고수준, 웹 요청/응답)

```swift
HTTP (웹 요청/응답 규칙)
  ↓
TCP (신뢰성 있는 연결)
  ↓
IP (라우팅, 주소 지정)
  ↓
소켓 (종단점: IP + Port 조합)
```

HTTP 통신은 단발적으로 요청이 수신 되었을 때만 연결을 허가하고 응답까지 마무리된 후에는 연결을 해제하는 방식인 만큼, 소규모의 전달이 다수 발생할 경우 연결과 해제 과정에 많은 낭비를 하게 된다.

또한, 클라이언트의 요청 없이 서버 쪽에서 먼저 클라이언트의 문을 두드리는 것이 불가능하다.

이러한 HTTP의 단점을 보완하기 위해 소켓 통신이 사용된다.

![3](https://github.com/user-attachments/assets/743513b0-d004-4dca-8b91-7b6ef2dd4512)

소켓 통신은 클라이언트와 서버, 두 컴퓨터가 특정한 Port를 통해 실시간 양방향 통신이 가능하며 이는 클라이언트만이 통신을 시작할 수 있었던 HTTP 통신과는 큰 차이점을 보여준다.

<br>

---

<br>

### 📌 멀티미디어 관련 소켓 통신 기술

**1️⃣ WebSocket (채팅, 실시간 반응 등)**

- HTTP 업그레이드 → TCP 기반 소켓으로 연결 유지
- 예: 유튜브 스트리밍 실시간 채팅

**2️⃣ WebRTC (라이브 화상 스트리밍)**

- 실시간 송수신용 UDP 소켓 사용
- UDP + DTLS + SRTP (암호화된 오디오/비디오)

**3️⃣ RTMP (기존 라이브 방송 기술)**

- TCP 기반 RTMP 소켓
- Facebook Live, Twitch 등에서 사용

<br>

### 📌 WebSocket vs WebRTC vs RTMP 비교 (From.GPT)

| 항목 | **WebSocket** | **WebRTC** | **RTMP** |
| --- | --- | --- | --- |
| 📌 용도 | 양방향 텍스트/데이터 통신 | 브라우저 간 실시간 미디어(P2P) 스트리밍 | 라이브 영상/오디오 스트리밍 (서버 중심) |
| 🧱 기반 프로토콜 | TCP (HTTP 업그레이드) | UDP + DTLS + SRTP + ICE | TCP |
| 🔁 통신 방향 | 클라이언트 ↔ 서버 (양방향) | 브라우저 ↔ 브라우저 (P2P), 서버 릴레이도 가능 | 클라이언트 → 서버 (거의 단방향) |
| 🔊 오디오/비디오 지원 | ❌ 없음 (텍스트/바이너리 전용) | ✅ 있음 (고품질 미디어 스트리밍) | ✅ 있음 (RTMP stream으로 전송) |
| 🔐 보안 | TLS(HTTPS 기반) | DTLS + SRTP (기본 암호화) | 지원은 가능하나 일반적으로 암호화 없음 |
| 🕒 지연 시간 | 중간 (수 ms ~ 수백 ms) | 매우 낮음 (10~100ms 수준, 실시간 통화에 적합) | 중간 ~ 낮음 (약 1초 내외, TCP 기반이라 약간 느림) |
| 📦 주 사용 사례 | 실시간 채팅, 알림, 대시보드 등 | 화상 회의, 음성 통화, 실시간 방송 | YouTube Live, Twitch, OBS 방송 송출 등 |
| 🌐 브라우저 지원 | ✅ 브라우저 완벽 지원 | ✅ 브라우저 기본 내장 지원 | ❌ 브라우저 미지원 (Flash 기반, 전용 플레이어 필요) |
| 🔧 구성 복잡도 | 낮음 (간단한 코드로 구현 가능) | 높음 (시그널링, NAT, 미디어 처리 등 추가 필요) | 중간 (FFmpeg/OBS 등과 연동 필요) |

WebRTC가 RTMP보다 훨씬 최신 기술이지만 RTMP는 오래된 만큼 장비 호환성, 서버 구성의 단순성 및 안정성 등의 장점이 있다.

지연 시간이나 속도 자체는 WebRTC가 더 빠름(UDP 기반)

<br>

### 📌 WebSocket

WebSocket은 HTTP를 통해 연결을 시작한 후, 양방향 소켓 통신으로 전환되는 고수준 프로토콜이다.

최초에는 HTTP로 서버와 핸드셰이크를 하고, 이후에는 지속적인 TCP 소켓 연결을 유지하며 클라이언트와 서버가 실시간으로 데이터를 주고받을 수 있게 한다.

기존 HTTP 방식으로는 실시간 통신이 비효율적이었기 때문에, WebSocket은 **TCP 위에 경량화된 통신 프레임 구조를 얹어** 더 효율적으로 통신할 수 있게 한다.

<br>

### 📌 WebSocket의 등장 배경 및 통신 흐름

기존 HTTP는 **요청-응답 구조**만을 가지고 있었다.

따라서 웹 브라우저가 서버로부터 실시간으로 데이터를 받고 싶다면, 지속적으로 새 요청을 보내거나 **long poling**이라는 우회 방식을 사용해야 했다.

이는 **네트워크 오버헤드**가 발생하고 실시간 통신에 적합한 구조는 아니었다.

하지만 웹 브라우저는 보안을 위해 임의의 TCP 포트로 직접 소켓을 연결할 수 없습니다.

따라서 브라우저에서 사용할 수 있는 안전한 TCP 소켓이 필요했고, WebSocket이 등장하게 되었다.

<img width="731" alt="4" src="https://github.com/user-attachments/assets/440191e3-fee4-4007-aa01-1f4616a7cf0b" />

WebSocket 연결이 수립되면 이후 모든 데이터는 **프레임 단위(Frame)**로 전송된다.

프레임은 WebSocket 프로토콜이 정의한 최소 단위로, 각각의 메시지를 작은 조각으로 나누어 전송하고 다시 합쳐 하나의 메시지로 만든다.

프레임은 작은 헤더 + payload 로 구성되어 있다.

![5](https://github.com/user-attachments/assets/7ea0812d-3df9-41e7-b846-a2566bf0f20a)

<br>

### 📌 HTTP 업그레이드

HTTP 1.1에는 `Upgrade`라는 헤더 필드가 존재한다.

처음엔 HTTP 요청처럼 시작되지만, 중간에 프로토콜을 바꾼다.

업그레이드가 되면 기존 HTTP 기능은 더 이상 사용되지 않는다.

```swift
GET /chat HTTP/1.1
Host: example.com
...
**Upgrade: websocket
Connection: Upgrade**
```

<br>

### 📌 Swift에서의 WebSocket 통신

`URLSessionWebSocketTask`를 사용하면 서버와의 WebSocket 통신을 직접 구현할 수 있다.

```swift
let url = URL(string: "wss://echo.websocket.org")!  // wss = WebSocket over TLS
let task = URLSession(configuration: .default).webSocketTask(with: url)
task.resume()

// 1. 메시지 보내기
let message = URLSessionWebSocketTask.Message.string("Hello, WebSocket!")
task.send(message) { error in
    if let error = error {
        print("Send error: \(error)")
    }
}

// 2. 메시지 받기
func receiveMessage() {
    task.receive { result in
        switch result {
        case .success(let message):
            switch message {
            case .string(let text):
                print("Received string: \(text)")
            case .data(let data):
                print("Received data: \(data)")
            @unknown default:
                break
            }
            // 다시 기다리기 (지속적으로)
            receiveMessage()
            
        case .failure(let error):
            print("Receive error: \(error)")
        }
    }
}

receiveMessage()
```

`receive()`는 메시지를 한 번만 수신하기 때문에 반복 호출해야 한다.
