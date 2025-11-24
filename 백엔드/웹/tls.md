# TLS

## CA 인증서 발급
- 인증서 발급 주체는 다음 과정을 거쳐 인증서를 발급 받는다.

#### 1. 개인키 생성
- 명령어
    ```
    openssl genrsa -out server.key 2048
    ```
- 생성 파일
    - server.key
        - 개인키
- 추가로 알아두기
    - RSA는 비공개키(개인키)가 있으면 공개키를 추출할 수 있다.
    - 개인키에는 n,e,d가 공개키에는 n,e의 구성요소를 가짐
    - 즉, 개인키에 공개키의 구성요소가 포함됨
    - 추출만 하면 됨
    - 추출 명령어
    ```
    openssl rsa -in server.key -pubout -out public.pem
    ```
    - 이렇게 하면 public.pem이라는 공개키가 추출된다.

#### 2. CSR 생성
- 명령어
    ```
    openssl req -new -key server.key -out server.csr
    ```
    - 위에서 설명했든, rsa 개인키 server.key에는 공개키가 포함되어 있고
    - 이 명령어는 server.key로부터 개인키를 뽑아서 csr에 포함하는 것
- 추가 입력 사항
    ```
    Country Name (2 letter code) [AU]: KR
    State or Province Name (full name) []: Seoul
    Locality Name (eg, city) []: Gangnam-gu
    Organization Name (eg, company) []: MyCompany
    Organizational Unit Name (eg, section) []: DevTeam
    Common Name (e.g. server FQDN or YOUR name) []: www.example.com
    Email Address []: admin@example.com
    ```
    - CN - Common Name - 도메인 이름
    - O - Organization - 조직 이름
    - OU - Organizational Unit - 부서 이름
- 생성 파일
    - server.csr
        - 여기에는 server.key 개인키로부터 추출한 `공개키`도 포함됨
        - 이 공개키는 추후에 인증서에 포함되어 클라이언트가 사용한다.

#### 3. 인증서 발급
- 인증 기관에 CSR을 제출
- 인증 기관은 .crt 또는 .pem 파일로 인증서 발급
- 인증서에는 다음 값들이 포함
    - `tbsCertificate` - TBS (To Be Signed) 원본 데이터
        - 서버의 공개키
            - 서버가 CSR을 생성할 때 만들어진 공개키
            - 나중에 클라이언트가 세션키를 공유할 때 사용
        - 도메인 이름, 유효 기간, 발급자 정보 등
    - `signatureValue` - 서명 값
        - TBS를 signatureAlgorithm으로 해시화
            - digest = SHA256(TBS 데이터)
        - 그리고 이 값을 `CA의 private key`로 암호화
            - signature = Encrypt_with_CA_PrivateKey(digest)
        - signature 값을 인증서에 포함
    - `signatureAlgorithm` - 서명에 사용된 해시 알고리즘 정보
- 결과
    - X.509 인증서 발급!
- 인증서의 사용
    - 클라가 서버의 인증서를 인증하는 과정에서 CA의 공개키로 이 signature 복호화함
    - CA의 공개키는 브라우저의 루트 인증서에 포함돼 있음
- __주의__
    - CA의 개인-공개키는 인증서를 검증할 때 사용
        - 이를 위해 CA의 공개키를 클라이언트가 인증기관으로부터 받아 옴
        - 보통 미리 루트 인증서 저장소에 캐싱해놓음
    - 서버의 개인-공개키는 session 키를 공유할 때 사용
        - 이를 위해 서버의 공개키를 인증서에 포함함

#### (테스트용, 필수 아님) 자체 서명 인증서 만들기
    - 테스트용 TLS 인증서를 쓰고 싶다면 아래 명령어를 사용하여 self-signed certificate를 만들 수 있음
        ```
        openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt
        ```
---

## TLS 핸드쉐이크 과정
```
┌─────────────────────────────────────────────────────────────────────────┐
│                          TLS Handshake Overview                       │
└─────────────────────────────────────────────────────────────────────────┘

 Client (클라이언트)                                     Server (서버)
           |                                                      |
           |  1) ClientHello (Handshake Record)                   |
           |  ─────────────────────────────────────────────────▶  |
           |    - Content Type: 22 (Handshake)                    |
           |    - Version: TLS 1.2 (0x0303)                       |
           |    - Handshake Type: ClientHello (1)                 |
           |    - 세션 ID, 난수(Random), 가능 Ciphers, 등등...        |
           |                                                      |
           |                                                      |
           |                       2) ServerHello (Handshake)      |
           |                       + Certificate (Handshake)       |
           |                       + ServerKeyExchange (Handshake, 
           |                         필요 시)                      |
           |                       + ServerHelloDone (Handshake)   |
           |  ◀────────────────────────────────────────────────────|
           |    - Content Type: 22 (Handshake)                    |
           |    - Version: TLS 1.2 (0x0303)                       |
           |    - Handshake Type: ServerHello (2)                 |
           |    - 서버가 선택한 CipherSuite, 난수(Random), 등...      |
           |    - Certificate: 서버 인증서(공개키 포함)                |
           |    - ServerKeyExchange: (필요할 경우)                   |
           |    - ServerHelloDone: '서버 측 헬로' 끝 알림             |
           |                                                      |
           |  3) ClientKeyExchange (Handshake)                    |
           |    + ChangeCipherSpec (Record)                       |
           |    + Finished (Handshake)                            |
           |  ─────────────────────────────────────────────────▶  |
           |    - ClientKeyExchange: PreMasterSecret 전송          |
           |    - ChangeCipherSpec: 이제부터 암호화 시작               |
           |    - Finished: 첫 번째 암호화 검증 메시지                   |
           |                                                       |
           |                       4) ChangeCipherSpec (Record)    |
           |                       + Finished (Handshake)          |
           |  ◀────────────────────────────────────────────────────|
           |    - 서버 측도 암호화 변경 통보(ChangeCipherSpec)           |
           |    - Finished: 암호화 검증 메시지                         |
           |                                                       |
           └───────────────────────────────────────────────────────┘

            *** 이후 클라이언트와 서버는 대칭키로 암호화된 채널을 사용 ***
```

#### 단계별 간단 설명
1.	ClientHello
- 클라이언트가 “나는 TLS (버전 1.2 등)를 쓰고 싶고, 가능한 암호 스위트 목록은 이렇다” 등의 정보를 보냄.
- 난수(Random)와 세션 ID, 확장(Extensions) 정보 등이 들어있음.

2.	ServerHello + Certificate (+ ServerKeyExchange) + ServerHelloDone
- 서버가 선택한 TLS 버전과 CipherSuite를 클라이언트에게 알림
- 동시에 서버 인증서(Certificate)를 보냄. 여기에는 서버의 공개키가 포함되어 있음
- (필요하다면) ServerKeyExchange 메시지를 추가로 보냄
- 모든 서버 측 헬로 메시지가 끝났음을 알리는 ServerHelloDone으로 마무리

3.	ClientKeyExchange + ChangeCipherSpec + Finished
- 클라이언트가 `서버 공개키`에 기반해 PreMasterSecret(혹은 그와 동등한 자료)을 전송
    - 이 공개키는 인증서에 포함되어 있는 공개키,
    - 서버가 CA에 제출하기 위한 CSR을 생성할 때 만든 공개키임
- ChangeCipherSpec를 통해 “이제부터 암호화된 통신을 시작하자”고 알림
- Finished 메시지로 최초의 암호화 검증 데이터를 서버에 보냄

4.	ChangeCipherSpec + Finished
- 서버도 마찬가지로 ChangeCipherSpec를 보내, 암호화 모드 진입을 명시임.
- Finished로 첫 암호화 검증을 클라이언트에 보냄.

---

## 인증서 서명 검증
- 서명 생성 과정 (CA가 서명할 때)
    - CA가 인증서의 특정 데이터(주로 TBSCertificate, “To Be Signed Certificate”)를 해싱(Hashing)
        - 예: `SHA-256(인증서 데이터) = H(해시값)`
    - CA가 자신의 개인키(Private Key)로 해시값(H)을 암호화 (즉, 서명)
        - `서명 값 = Encrypt(H, CA의 Private Key)`
    - 서명 값(Signature)을 인증서의 일부로 포함하여 서버에 발급
- 서명 검증 과정 (클라이언트가 검증할 때)
    - 인증서에서 서명 값(Signature)과 CA의 공개키(Public Key)를 가져옴
        - 이 공개키는 CA가 서명할 때 사용한 비공개키와 쌍을 이루는 CA의 공개키
        - 서버가 CSR을 생성할 때 만든 비공개키와 쌍을 이루는 서버의 공개키와는 다름!!
    - 서명 값(Signature)을 CA의 공개키로 복호화
        - `Decrypted_H = Decrypt(서명 값, CA의 Public Key)`
        - 여기서 나온 값이 원래 CA가 서명할 때 사용했던 **해시값(H)**이어야 함.
    - 인증서에서 서명된 데이터를 직접 해시(Hash)하여 비교
        - `Calculated_H = SHA-256(인증서 데이터)`
        - `Decrypted_H`와 `Calculated_H`가 일치하면 서명이 유효함
- 서명 값은 CA의 개인키로 “해시값”을 암호화한 것이고,
    - 이를 검증할 때 CA의 공개키로 복호화하여 원본 해시값과 비교하는 과정이 수행
        - 브라우저는 미리 신뢰할 수 있는 CA(인증기관) 리스트를 가지고 있고 여기에 public key도 포함됨

- 클라는 CA가 제공하는 공개키와 인증서를 가지고, 서버의 인증서를 인증함

---

## 세션 키란?
- 세션 키는 대칭키 → 같은 키로 암호화 & 복호화
- 통신을 빠르고 효율적으로 암호화하려고 사용
- 이 키는 한 세션에만 유효하고, 통신 끝나면 버려짐

#### 세션 키 생성 방법 두 가지
- `방법 1.` 서버 공개키로 암호화 (TLS 1.2 이전의 기본 방식)
    1.	클라이언트가 랜덤한 세션 키를 생성함
    2.	이 키를 서버 인증서의 공개키로 암호화해서 전송
    3.	서버는 자신의 비공개키로 복호화
    4.	이제 서버와 클라는 같은 세션 키를 공유하게 됨
- 이 방법은 구현이 쉬운데, **Forward Secrecy(전방 비밀성)**이 없음
    - → 예전에 탈취된 비공개키로 과거 통신을 복호화할 수 있음
- `방법 2.` 디피-헬만 키 교환 (DHE or ECDHE - TLS 1.2/1.3에서 많이 씀)
	1.	클라이언트와 서버가 **임시 키 쌍(Ephemeral Keypair)**을 생성함
	2.	서로의 공개키를 교환
        - 이 때 CSR을 생성할 때 만든 공개키로 서명해서 안전하게 공개키를 보낸다.
        - 이 공개키는 CA의 공개키도, CSR을 생성할 때 만든 서버의 공개키도 아니다.
	3.	수학적으로 계산해서 같은 세션 키를 양쪽에서 계산
- 서버는 비공개키가 탈취되더라도 이전 세션의 세션 키는 알아낼 수 없음 (Forward Secrecy 보장)
    -  TLS 1.3에서는 이 방식만 사용함
