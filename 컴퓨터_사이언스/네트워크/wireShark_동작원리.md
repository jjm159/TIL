# WireShark 동작 원리

[D.4. dumpcap: Capturing with "dumpcap" for viewing with Wireshark](https://www.wireshark.org/docs/wsug_html_chunked/AppToolsdumpcap.html)

[pcap](https://en.wikipedia.org/wiki/Pcap)

와이어샤크는 pcap이라는 패킷을 읽어들이는 api를 사용한 dumpcap 프로그램을 사용하여 패킷을 수집, 캡쳐한다. 

사용자가 네트워크에 연결되면 네트워크 인터페이스 카드(NIC)와 링크 계층 드라이버를 통해 패킷을 송수신하게 된다. 패킷을 수집하려면 NIC카드를 통과하는 데이터를 읽을 수 있어야 한다. 이를 위해 pcap api를 사용한다. 이 api는 NIC에 오고 가는 패킷들을 읽을 수 있는 기능을 제공해준다. 

그리고 이 api를 사용하여 만든 프로그램이 dumpcap이다. 이 프로그램을 사용하면 NIC를 지나다니는 패킷들을 수집할 수 있다. 맥북에는 이미 설치되어 있어서 실행시켜보았다. 옵션을 주지 않고 실행시키면 Packets: number 와 같이 패킷이 얼마나 수집되었는지 개수가 계속 Counting된다. 

와이어샤크는 이 dumpcap을 엔진으로 사용하여 다른 기능들을 추가한 프로그램이다. 와이어샤크 또한 명령어로 실행시킬 수 있다.
