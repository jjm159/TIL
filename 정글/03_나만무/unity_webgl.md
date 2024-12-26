# Unity WebGL

## 뭔데? 
- WebGL은 웹 그래픽 라이브러리. 3D 렌더링 가능케 함
- Unity에서는 Web 환경에서도 게임을 배포할 수 있도록 WebGL 빌드 기능을 제공

## 심각한 문제
- https://docs.unity3d.com/kr/2019.4/Manual/webgl-networking.html
- 웹에서는 소켓 통신이 애초에 되지 않음
- 정리
    - 애초에 웹에서는 소켓이 사용 불가
    - 대신에 웹 표준에서 정의하고 있는 네트워크 통신 방법을 사용하는데, 이게 WebSocket이랑 WebRTC
    - 걍 Phaser를 써도 소켓 통신은 안되고 저 두 개 기술 써야 함
- 대안
    - TCP 통신기반인 Websocket으로 먼저 해보다가, 성능 후달리면 UDP 기반의 Webrtc로 갈아탄다