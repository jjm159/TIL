# 7주차
- 소켓
- 웹서버, CGI, HTTP
- 네트워크
- 프로세스와 스레드
- 동시성

---

# 학습 키워드 정리

### 네트워크 계층 (OSI7 Layer, TCP/IP Layer)

#### 왜 계층을 만들었을까?
- 모든 네트워크 참여 개발자가 동일한 규칙을 기반으로 작업하기 위해

#### TCP/IP Layer는 왜 따로 구분해서 다룰까?
- OSI 7계층 모델을 통신에서 중요한 기능을 중심으로 단순화함
- [참고](https://westahn.com/osi-7-%EA%B3%84%EC%B8%B5%EC%9D%B4%EB%9E%80/)

### 클라이언트-서버 모델
- 클라이언트 : 서버에 요청을 보내는 친구 (서비스 사용자)
- 서버 : 클라이언트의 요청을 받아서 처리, 응답하는 친구 (서비스 제공자)

### 소켓(socket, bind, listen, accept, connect, close)

#### BSD 소켓이란?
- 버클리에서 만든 소켓 라이브러리
- 표준화 됨
- https://velog.io/@ceusun0815/%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC-BSD-%EC%86%8C%EC%BC%93

#### 함수 알아보기 
- socket()
    - 소켓 생성
- bind
    - 서버에서 클라이언트 소켓과 바인드
- listen
    - 서버에서 클라이언트의 연결 요청를 기다림
- accept
    - 서버에서 클라이언트의 연결 요청를 수락
- connect
    - 서버와 클라이언트 소켓을 연결
- close
    - 소켓 통신 종료

### 파일 디스크립터
- 파일 식별자 (int)
- 프로세스마다 커널에서 파일 식별자 테이블 가지고 있음
- fd 1: stdin, fd 2: stdout, fd 3: stderr

#### "모든 것은 파일이다"
- 디스크 데이터(pdf, txt, exe...)
![image](https://hackmd.io/_uploads/ryYo7sOgJx.png)
- I/O bus에 연결된 친구들은 싹다 file로 추상화
- 프로세스의 모든 I/O 목적지는 파일로 추상화
    - 외부 프로세스도 파일
    - 디스크도 파일
    - 네트워크 장치도 파일
    - 표준 입출력 장치도 파일

### Datagram Socket vs Stream Socket
![image](https://hackmd.io/_uploads/Hy204sOxye.png)

#### 연속적?
- "TCP 계층에서 순서나 데이터 손실을 체크 하느냐"

### CGI / WebServer / MIME Type

#### 역사
- 웹은 처음에 html만 주고 받을 수 있었음 -> static
- 사진, 비디오 그 외 데이터 타입도 주고 받아야 하는 수요가 생김 -> dynamic
- CGI라는 인터페이스를 만들어서 웹서버에서 동적인 데이터 처리를 할 수 있도록 함
- 근데 CGI에는 한계가 있었고 이걸 더 발전시켜서 WAS가 나옴

#### WebServer
- 정적 웹 서비스를 위한 서버
- 아파치, nginx

#### CGI(Common Gateway Interface)
- client의 요청을 처리해 줄 응용 프로그램과 web server간 interface
- https://velog.io/@reasonoflife39/CGICommon-Gateway-Interface-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0-%EC%9B%B9-%ED%8E%98%EC%9D%B4%EC%A7%80%EB%A5%BC-%EB%8F%99%EC%A0%81%EC%9C%BC%EB%A1%9C-%EB%A7%8C%EB%93%9C%EB%8A%94-%EA%B8%B0%EC%88%A0

#### WAS(Web Application Server)
- 웹페이지 처리 뿐 아니라 범용적인 데이터 처리를 해주는 서버
- https://velog.io/@bky373/Web-%EC%9B%B9-%EC%84%9C%EB%B2%84%EC%99%80-WAS

#### MIME(Multipurpose Internet Mail Extensions) Type
- HTTP 표준에서 Html 뿐 아니라 다양한 타입에 대한 요청을 처리할 수 있도록 타입 정의
- RFC 1341 - MIME 표준의 도입 (1991년)

### HTTP (요청/응답, 헤더, 메소드, 상태코드, HEAD 메소드)
- 웹 프로토콜 (데이터 규격?)

#### RESTful API
- html 웹 서버를 위해 만들어진 http가 더 범용적으로 사용되면서, 규칙 부재로 혼란
- html 뿐 아니라 인터넷 상에 존재하는 리소스를 "더 잘" 식별하고 처리할 수 있게 만든 규칙이 RESTful API.

### Proxy
- https://namu.wiki/w/%ED%94%84%EB%A1%9D%EC%8B%9C%20%EC%84%9C%EB%B2%84
- 캐시 서버 + 보안 + 우회 + 검열 + ...

---

# 퀴즈

### HTTP GET 요청과 POST 요청의 가장 큰 차이점, 이것이 요청 헤더나 데이터 전송에 어떤 영향을 미치는지?

GET 요청은 데이터를 URL의 일부(query string)로 전송하지만 POST 요청은 데이터를 요청 본문(body)에 포함한다. 이 차이로 인해 

1) GET 요청의 URL 길이에는 URL 길이에는 브라우저나 서버에 따라 제한이 있다. 이로 인해 전송할 수 있는 데이터의 양이 제한된다.
2) POST 요청은 'Content-Length'와 'Content-Type' 같은 추가적인 헤더 정보를 필요로 한다.

### HTTP 응답 코드 404의 의미는 무엇? 그리고 서버가 요청을 처리할 수 없을 때 반환하는 HTTP 상태 코드는 무엇?

HTTP 응답 코드 404는 "Not-Found"를 의미한다. 이 코드는  서버가 요청된 리소스를 찾을 수 없을 때 반환된다.
서버가 요청을 처리할 수 없을 때 반환하는 HTTP 상태코드는 500이다. 이 코드는 "Internal Server Error"를 나타낸다.

### 파일 디스크립터(File Descriptor)란? UNIX/Linux 시스템에서 표준 입출력/에러의 파일 디스크립터의 번호를 쓰시오

파일 디스크립터는 운영 체제에서 파일이나 다른 입출력 리소스에 대한 접근을 추상화 하는데 사용되는 정수다. 파일 디스크립터를 통해 운영 체제는 파일, 파이프, 소켓 등 다양한 입출력 리소스를 `일관된 방식`으로 관리할 수 있다. 예를 들어, 파일을 열면 운영체제는 해당 파일을 가리키는 파일 디스크립터를 프로그램에 제공한다.

UNIX/Linux 시스템에서 표준 입력의 파일 디스크립터 번호는 0, 표준 출력은 1, 그리고 표준 에러는 2다.

### TCP에서의 '3-way handshake'의 절차 설명

1) 클라이언트가 서버에 SYN(Synchronize) 패킷을 보내 연결 요청
2) 서버는 SYN-ACK(Synchronize-Acknowledge) 패킷으로 응답하여 연결 요청을 받았음을 알리고, 자신도 연결 준비가 되었음을 나타냄
3) 클라이언트는 ACK(Acknowledge) 패킷을 서버에 보내 연결 확정

### TCP와 UDP에서 패킷 손실 시 대처 방법에는 어떤 차이가 있나요?

TCP는 패킷 손실이 발생하면 자동으로 재전송을 시도한다. 수신자는 받은 패킷에 대해 확인 응답(ACK)을 보내고, 송신자는 ACK를 받지 못한 패킷을 재전송한다.
반면, UDP는 패킷 손실에 대해 자체적으로 대처하지 않는다. UDP는 확인 응답이나 재전송 기능이 없어, 패킷 손실이 발생하면 이를 어플리케이션 레벨에서 처리해야 한다.

### 허프만 코딩은 데이터를 압축하는 알고리즘 중 하나로, 빈도가 높은 문자에는 짧은 비트를 할당하고, 빈도가 낮은 문자에는 긴 비트를 할당하여 데이터를 효율적으로 압축하는 방법이다. 다음과 같은 문자와 빈도수로 이루어진 메시지에 허프만 코딩을 적용할 때, 질문에 답

- 문자(빈도수): A(2), B(3), C(7), D(11), E(15)

#### 1) 허프만 트리 그리기 (상대적으로 작은 값을 가진 노드로의 간선에 작은 값 배정)
#### 2) 1)의 트리에서 각 문자에 대한 허프만 코드를 생성
#### 3) 2)의 허프만 코드를 활용하여 문자열 "ABCADEAE"를 압축한 결과의 비트수를 계산
#### 4) 2)의 허프만 코드에 의해 압축된 비트열 "011001101010110010100100"을 해독하여 원래의 문자열을 찾아 

- ![solution](https://private-user-images.githubusercontent.com/44443949/382652288-e0b45309-1a58-46ac-a531-809cef405b7b.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3MzA2OTkxMDIsIm5iZiI6MTczMDY5ODgwMiwicGF0aCI6Ii80NDQ0Mzk0OS8zODI2NTIyODgtZTBiNDUzMDktMWE1OC00NmFjLWE1MzEtODA5Y2VmNDA1YjdiLnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNDExMDQlMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjQxMTA0VDA1NDAwMlomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPWNjYzk4ODU0NjI1NzZmYjg3M2JjYTQzZWE2MjY2YTgyZmJhMTJiYTBmYjFjZTI5ZjBkN2YwNWZkMWU1MTlmZDQmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0In0.6wl7-NsjW1Dfw0JcevhxMA6D23Fajj53uE-JNSbNvW0)