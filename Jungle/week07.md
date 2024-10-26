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