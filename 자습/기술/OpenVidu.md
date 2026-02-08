#WebRTC

---
출처: https://savingbe.tistory.com/22

# WebRTC
WebRTC(Web Real-Time Communication)
웹 브라우저와 기기 간 실시간 음성, 텍스트 및 화상 통신을 가능하도록 하는 오픈소스 프로젝트
웹 브라우저를 통해 직접 P2P 통신을 하기 때문에, 
통신을 이용하는 기기에서는 추가적인 플러그인이나 어플리케이션을 설치하지 않아도 통신이 가능

WebRTC는 서버와 peer의 형태가 어떻게 되어있느냐에 따라 Mesh, SFU, MCU같은 형식으로 나뉘고, 
서버를 구축할 때에는 시그널링 서버, STUN 서버, TURN 서버 등을 만들어야 하는 번거로움이 있음
# OpenVidu
WebRTC를 기반으로 여러 화상통화 서비스를 개발하기 편하게 해주는 플랫폼
[LiveKit](https://livekit.io/)이라는 WebRTC infrastructure를 기반으로 함

###### v2와 v3 차이
**SFU 서버의 변경**
v2는 Kurento라는 SFU 서버를 기반으로 함
Kurento도 openVidu 개발자들이 제작한 SFU였고, 
그 위에 openVidu라는 플랫폼을 개발해 사용을 용이하게 만든 것이라고 함
그런데 이들이 판단하기에 Kurento는 낡은 프로그램이 되었고, 명백한 성능의 한계를 가지게 됨
그래서 v3에서는, 내부 서버를 Kurento에서 mediasoup로 변경하기로 했다고 함

**LiveKit의 적용**
v2는 LiveKit이 적용되어있지 않지만, v3에는 적용되어 있음
그 덕에 v3는 LiveKit을 기반으로 설계된 서버와 클라이언트라면 그대로 사용할 수 있음 
LiveKit은 chatGPT로 유명한 openAI에서도 사용하고 있는 오픈 소스

