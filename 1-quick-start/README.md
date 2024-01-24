# workshop-css
Workshop for Tencent Cloud Cloud Streaming Service


CSS는 쉽고 간편하게 고품질 저지연의 라이브 스트리밍을 안정적으로 제공하기 위한 완전 관리형 서비스입니다. CSS는 라이브 채널을 빠르고 쉽게 생성하고 삭제할 수 있는 템플릿을 구성하여, 라이브 스트리밍 라이프사이클을 빠르고 손쉽게 구성할 수 있도록 도와줍니다.
템플릿을 사용하여 CSS 채널을 구성한 후 라이브 피드를 클라우드에 입수하면, 재생 URL을 얻을 수 있습니다. 즉, CSS가 트랜스코딩, 패키징을 관리하고 CSS CDN을 통해 시청자에게 스트리밍 비디오를 안정적으로 제공한다는 건데요.

![CSS 아키텍처](./assets/images/css_architecture.png)

이번 lab 에서는, Cloud Streaming Service 를 이용해 live feed 를 클라우드로 입수하고, 디바이스에서 플레이 할 수 있는 엔드 투 앤드 파이프라인을 구성하는 방법을 다룰 예정 입니다.

# 1. Pre-requisites
텐센트 클라우드의 Cloud Streaming Service 를 이용하기 위해서는, 별도의 Push 및 Play Domain 등록이 필요합니다.

# 2. Create a push domain


1. CSS 콘솔 메인 화면 왼쪽 탭에서 *Domain* 탭을 클릭합니다. 그러면 현재 구성되어 있는 CSS 도메인들을 확인할 수 있습니다.
2. 메인 메뉴 위쪽에 위치한 *Add Domain* 버튼을 클릭 합니다.
![도메인추가](./../assets/images/3-css-console-new-domain.png)
3. 도메인 추가 팝업이 뜨면, 아래와 같이 Push Domain을 위한 값을 입력해 줍니다. 입력이 완료되면 *Add Domain* 버튼을 클릭해서 다음으로 이동합니다.
Type : Push domain
Domain Name : push 용으로 사용할 도메인 이름
![push도메인추가](./../assets/images/4-1-css-push-domain.png)

4. CNAME Configuration 메뉴를 참고해서, DNS 서비스에서 host 에 대한 CNAME 값을 추가해 줍니다.
*CNAME Configuration 메뉴*
![CNAME Configuration 메뉴](./../assets/images/4-1-css-push-domain-cname-configuration.png)

*DNS 에 CNAME 레코드 추가*
![CNAME 레코드 추가](./../assets/images/4-1-css-push-domain-cname.png)

*CNAME 레코드 검증*
![레코드 검증](./../assets/images/4-1-css-push-domain-cname-verify.png)

# 3. Create a play domain

1. CSS 콘솔 메인 화면 왼쪽 탭에서 *Domain* 탭을 클릭합니다. 그러면 현재 구성되어 있는 CSS 도메인들을 확인할 수 있습니다.
2. 메인 메뉴 위쪽에 위치한 *Add Domain* 버튼을 클릭 합니다.
![도메인추가](./../assets/images/3-css-console-new-domain.png)
3. 도메인 추가 팝업이 뜨면, 아래와 같이 Type 을 Playback Domain 으로 변경하고, 아래와 같은 값을 입력해 줍니다. 입력이 완료되면 *Add Domain* 버튼을 클릭해서 다음으로 이동합니다.
Type : Play domain
Acceleralation region : Outside Chinese mainland 
Domain Name : playback 용으로 사용할 도메인 이름
![play도메인추가](./../assets/images/4-2-css-play-domain.png)

4. CNAME Configuration 메뉴를 참고해서, DNS 서비스에서 host 에 대한 CNAME 값을 추가해 줍니다.
*CNAME Configuration 메뉴*
![CNAME Configuration 메뉴](./../assets/images/4-2-css-play-domain-cname-configuration.png)

*DNS 에 CNAME 레코드 추가*
![도메인추가](./../assets/images/4-2-css-play-domain-cname.png)

*CNAME 레코드 검증*
![레코드 검증](./../assets/images/4-2-css-play-domain-cname-verify.png)

May I have a quick question? 
- In the console, there are 2 menus : Live transcoding, Adaptive Bitrate.
- What are difference between 2 features? And what would be the purpose of live transcoding (for recording & relay?)
- Because basically for the CSS live streaming, the transcoded stream of ABR configuration might be used, then no need for transcoding configuration.

Live transcoding is NON-ABR transcoding
ABR is not available for all protocols, for the customers who ate using HTTP-FLV/RTMP, they just need to do single resolution transcoding

# 4. Create a Adaptive Bitrate Domain

1. CSS 콘솔 메인 화면 왼쪽 탭에서 *Feature Configuration -> Adaptive Bitrate* 탭을 클릭합니다.
2. Adaptive Bitrate 콘솔 상단에 *Create Template* 을 버튼을 클릭 합니다.
3. CSS 에서는 Template 을 구성해서 Bitrate, 해상도 등을 구성하고 ABR 래더를 생성할 수 있습니다.

3-1. Template Name 은 *testABR* 을 입력합니다. 이는 임의의 값을 입력 가능 합니다.
![abr-config](./../assets/images/5-1-abr-config.png)

3-2. *Streams* 메뉴에는 ABR 래더를 구성하고 레더별 트랜스코딩 방법, 해상도, 비트레이트 등을 결정 할 수 있습니다.
이번 랩에서는 Standard Transcoding 을 활용한 기본 구성으로 진행할 예정 입니다.
다음과 같이 3개의 Stream 을 Adaptive Bitrate 으로 구성하고,
각각의 Stream Name 은 HD, SD, Smooth 로 구성합니다.
Stream Quality 은 각각 HD, SD, Smooth 을 선택합니다. 그러면 자동으로 resolution, bitrate 값들이 지정됩니다.
Note: 이번 랩에서는 기본값을 선택하지만, bitrate, resolution, transcoding type 등 유연하게 지정 가능합니다.
![abr-config-hd](./../assets/images/5-1-abr-config-hd.png)
![abr-config-sd](./../assets/images/5-1-abr-config-sd.png)
![abr-config-smooth](./../assets/images/5-1-abr-config-smooth.png)

3-3. Stream 구성까지 완료한 이후 *Save* 버튼을 클릭 합니다. 그러면 구성한 Adaptive Bitrate 템플릿을 Domain 과 Binding 하라는 팝업이 뜨는데요. 아래와 같이, 3번에서 구성한 Playback Domain 을 선택합니다.
![bind-abr-config-menu](./../assets/images/5-2-bind-abr-config-menu.png)
![bind-abr-config-to-play](./../assets/images/5-2-bind-abr-config-to-play.png)

# 5. Test and verify 
