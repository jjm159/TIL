# TLS

## 과정
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

## 단계별 간단 설명
1.	ClientHello
- 클라이언트가 “나는 TLS (버전 1.2 등)를 쓰고 싶고, 가능한 암호 스위트 목록은 이렇다” 등의 정보를 보냄.
- 난수(Random)와 세션 ID, 확장(Extensions) 정보 등이 들어있음.

2.	ServerHello + Certificate (+ ServerKeyExchange) + ServerHelloDone
- 서버가 선택한 TLS 버전과 CipherSuite를 클라이언트에게 알림.
- 동시에 서버 인증서(Certificate)를 보내어 공개키를 증명임.
- (필요하다면) ServerKeyExchange 메시지를 추가로 보냄.
- 모든 서버 측 헬로 메시지가 끝났음을 알리는 ServerHelloDone으로 마무리임.

3.	ClientKeyExchange + ChangeCipherSpec + Finished
- 클라이언트가 서버 공개키에 기반해 PreMasterSecret(혹은 그와 동등한 자료)을 전송임.
- ChangeCipherSpec를 통해 “이제부터 암호화된 통신을 시작하자”고 알림.
- Finished 메시지로 최초의 암호화 검증 데이터를 서버에 보냄.

4.	ChangeCipherSpec + Finished

- 서버도 마찬가지로 ChangeCipherSpec를 보내, 암호화 모드 진입을 명시임.
- Finished로 첫 암호화 검증을 클라이언트에 보냄.

## 인증서 서명 검증
- 서명 생성 과정 (CA가 서명할 때)
    - CA가 인증서의 특정 데이터(주로 TBSCertificate, “To Be Signed Certificate”)를 해싱(Hashing)
        - 예: SHA-256(인증서 데이터) = 해시값(H)
    - CA가 자신의 개인키(Private Key)로 해시값(H)을 암호화 (즉, 서명)
        - 서명 값 = Encrypt(H, CA의 Private Key)
    - 서명 값(Signature)을 인증서의 일부로 포함하여 서버에 발급
- 서명 검증 과정 (클라이언트가 검증할 때)
    - 인증서에서 서명 값(Signature)과 CA의 공개키(Public Key)를 가져옴
    - 서명 값(Signature)을 CA의 공개키로 복호화
        - Decrypted_H = Decrypt(서명 값, CA의 Public Key)
        - 여기서 나온 값이 원래 CA가 서명할 때 사용했던 **해시값(H)**이어야 함.
    - 인증서에서 서명된 데이터를 직접 해시(Hash)하여 비교
        - Calculated_H = SHA-256(인증서 데이터)
        - Decrypted_H와 Calculated_H가 일치하면 서명이 유효함
- 서명 값은 CA의 개인키로 “해시값”을 암호화한 것이고,
    - 이를 검증할 때 CA의 공개키로 복호화하여 원본 해시값과 비교하는 과정이 수행
        - 브라우저는 미리 신뢰할 수 있는 CA(인증기관) 리스트를 가지고 있고 여기에 public key도 포함됨