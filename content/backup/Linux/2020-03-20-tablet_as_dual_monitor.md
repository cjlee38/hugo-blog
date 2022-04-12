---
categories: linux
date: "2020-03-20T05:37:00Z"
tags: ['null']
title: '# 다시 우분투로, 그리고 태블릿을 듀얼 모니터로 활용하기'
---

# 1. 왜 다시 우분투로?
리눅스를 지원한다는 프로그램들도, 대부분 데비안이나 레드햇계열의 확장자만 제공해주고, 아치 리눅스는 외면받는 현상을 정말 많이 목격했다. 예를들면 팀뷰어라던가.. 슬랙이라던가.. 덤으로 압도적으로 차이나는 구글링 결과 숫자도.

기어코 구글링을 할때에도

for/on manjaro로 검색하던 것을 for/on linux/manjaro로 검색하는 꼴이 되버리니 이게 뭐하는 짓인가 싶더라.

결국 다시 우분투로 갈아타니, 상대적으로 이렇게 편할수가 없더라. deb확장자를 볼때에는 눈물이 다 나더라.

# 2. 태블릿을 듀얼 모니터로 활용 - 실패담
해야 할 공부는 안하고, 공부를 할 환경만 이것저것 세팅하는데 재미를 붙이다보니, 자주 활용하지 않던 예전 태블릿

(galaxy tab A(2016) with s pen) 을 듀얼 모니터로 활용할 수 있을까 하는 생각이 들었다.


별로 그렇게 어려울 것 같지도 않겠다 하는 생각에, 대충 찾아보니 태블릿을 듀얼모니터로 활용하는 프로그램은 세가지가 있었다.

1. spacedesk2. 태블릿을 듀얼 모니터로 활용하기
2. Twomon(SE, USB, Air)
3. displaylink

spacedesk는 무료로 제공되는 프로그램이다. 근데 아무래도 무선으로 활용하는 방식이라 뭔가 많이 느릴 것 같았고, 윈도우에서만 지원하기에, 또 다시 wine으로 골머리 썩기 싫어서(설치파일이 msi 확장자여서) 그냥 스킵했다. 

Twomon은 유료이긴 하지만, 정말 많이 활용되는 소프트웨어인 것 같았다. 영어로 구글링하면 거의 반절 이상은 twomon으로 쏟아져 나왔다. 근데 얘도 윈도우와 맥에서만 지원하길래, 그나마 exe 확장자인걸 보고 와인으로 한번 시도 정도는 해보았으나 (플레이스토어/앱스토어 에서 5000원/10000원으로 판매, 데스크탑 프로그램은 무료인듯)  와인을 윈도우10, 64빗으로 세팅해도 버전이 낮다고 안되길래 "역시구나" 하면서 스킵했다. 아마 와인 사용법도 제대로 모르는게 흠인듯.

displaylink는 리눅스도 지원한다길래, 옳다구나 하고 바로 시도해보았는데, 얘는 소프트웨어가아니라 드라이버인것 같더라. 장장 8시간 가까이를 시도했는데도, 결국 실패하고 말았다.

이것저것 구글링해본 최종 결과는 "USB Host Mode"가 안되서 인 것 같았다. "충전중일 때에는 호스트 모드가 안되고, 호스트 모드가 안되면 사용을 못합니다" 라고 하는 것 같았다. 태블릿이 아닌 내 스마트폰으로 시도해보았는데도 안되더라. 후..

아이패드를 사용하시는 분은 [여기](http://www.kwangsiklee.com/2017/11/asus-mb169b-ubuntu-14-04-5-lts-displaylink-%ED%99%98%EA%B2%BD%EC%84%A4%EC%A0%95-%ED%95%98%EA%B8%B0/)를 참고하면 좋을 것 같다.

결국 외면하고 외면하던 VNC라는 것을 활용해보기로 했다.

# 3. 태블릿을 듀얼 모니터로 활용하기 - 성공담
VNC가 뭔가 하고 찾아보니, Virtual Network Programming의 약자로, 한마디로 원격제어로 이해했다.

원격제어랑 듀얼모니터랑 무슨 상관이지? 싶었는데, 이게 일종의 꼼수인 것 같더라. 무슨말이냐 하면, 내 모니터로 내보내는 해상도를 가상으로 확장시킨 다음에, 빈공간을 원격제어로 접근하는 것이다.

예를 들어, 내가 현재 모니터를 800*600으로 사용하고 있는데, 얘를 강제로 1600*600으로 확장해놓고 보면, 내 모니터에서는 800*600만 보여주게 되고, 나머지 빈공간의 800*600을 원격제어로 접근하는 것이다. 참 기발하더라.

[Extending desktop over VNC, the easy way / Community Contributions / Arch Linux Forums](https://bbs.archlinux.org/viewtopic.php?id=191555)


처음에는 위의 링크를 따라해보았으나, 똑같이 진행했는데도 에러가 나고, 구글링을 해봐도 별 소득이 없어서 포기할까 하려는 찰나, 한 은인을 뵙게 되었다.

[Android tablet as second monitoron Ubuntu 16.04](http://pavatechpit.blogspot.com/2017/04/android-tablet-as-second-monitor-on.html)  

[mrenrich84/vnc_virtual_display_linker](https://github.com/mrenrich84/vnc_virtual_display_linker)


Apparently, nobody took this seriously.

So I decided to create a script that configures your X11 server and creates a VNC server automatically.  

nobody took this seriously.... 참 공감되는 말이다.
아무튼, 첫번째 링크에 있는 커맨드를 파이썬으로 잘 조립해놓으셨다.
사용하는 방법은 정말 쉽다.

써있는대로

```
$ sudo apt install x11vnc
$ x11vnc -storepasswd #비밀번호 설정
$ git clone https://github.com/mrenrich84/vnc_virtual_display_linker
$ cd vnc_virtual_display_linker
$ ./vnc_virtual_display_linker.py
```

를 순서대로 진행한다.


그리고 현재 나는 우분투 18.04.04를 사용하고 있으므로, 다음도 진행해줬다.

```
The solution would be to create the following file:
sudo vim /usr/share/X11/xorg.conf.d/20-intel.conf
with the following content:

Section "Device"
Identifier "intelgpu0"
Driver "intel"
Option "VirtualHeads" "2"
EndSection

and the logout/re-login.
```

그리고, C를 타이핑해서 내가쓰고있는 모니터의 width와 height를 설정한뒤, 내가 사용할 가상 모니터의 width와 height를 설정한다.

그리고 N 타이핑 -> 엔터, 마지막으로 S 타이핑.

(A의 경우 케이블을 이용하는 방식인 것 같은데, 제대로 안해봐서 모르겠다.)

이러면 이제 원격제어를 할 수 있는 서버가 열린거다.

다음으로 VNC Viewer, 즉 클라이언트 역할을 하는 어플리케이션을 플레이스토어에서 다운받았다.

나는 'VNC Viewer - Remote Desktop' 이라는 어플을 다운받았다.

어플을 실행한 뒤, 우측 하단의 + 버튼을 누르면 아이피와 포트, 그리고 임의로 정하는 이름을 입력하는 창이 등장한다.

아이피는 터미널에서
```
$ ip addr show
```
를 통해서 inet에 해당하는 아이피 번호를,

포트는 서버를 열었을 때 보이는 포트 (나는 5900이었다) 를 입력하고, 이름은 대충 아무거나 써서 진행했다.

그리고나서 몇가지 물어보는 부분에 대답을 해주고 나면......

결실을 맺는다.



# 4. 자주 쓸까?
그런데 몇가지 문제점이 있다.

첫째로는 refreshing rate가 너무 낮다. 생각으로는 유튜브나 강의영상을 태블릿에 띄워놓으려고 했는데, 렉이 너무걸린다. 아마 케이블을 연결하는 방식이 있었으니 이를 활용하면 어떻게 잘 될지도 모르겠다.

둘째로는 내가 설정해놓은 resolution이 좀 어긋났다. 이건 아마 이것저것 테스트를 해봐야 할 것 같다.

셋째는... 첫 번째 문제랑도 연관이 있는데, 결국 인터넷을 사용하는 방식이기 때문에 와이파이가 없으면 사용하기가 어렵다. 이 역시 케이블을 이용해봐야 할 것 같다.

마지막으로는, 갑자기 팬 소음이 엄청 시끄러워졌다. 그만큼 cpu를 많이 잡아먹는다는 이야기인 것 같은데... 그럴만 한가? 아무튼, 총합 약 12시간에 걸친 듀얼모니터를 반만 성공했다. 나머지는 다음에 하는걸로..

다음 포스팅은 겸사겸사 진행했던, 주피터 노트북으로 서버 여는 방법으로 적어야 겠다.