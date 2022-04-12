---
categories: linux
date: "2020-03-09T21:24:00Z"
tags: ['null']
title: '# 만자로 기본 설정'
---

# 1. 한글 설정
리눅스에는 여러가지 한글 입력기가 있지만, 대체로 많이 알려진 것은 ibus와 uim인듯 하다. 우분투를 처음 깔았었을 당시만 하더라도 뭣도 모르고 두개를 동시에 다 깔았고, ibus로 한글입력을 했었지만, 이번에 다시 한번 찾아보니 대체로 uim이 오류가 적다고 해서, uim으로 설치했다.

![1](/assets/images/2020-08-27-05-34-02_2020-08-27-linux_1.md.png){: .alignCenter}

```
$ sudo pacman -S uim
```

명령어를 통해서 uim을 설치한 뒤에, vim으로 .xprofile을 다음과 같이 수정하였다. ( xprofile을 처음 열어보는 거라면, 아무 것도 없어서 '내가 잘못열은건가?' 할 수도 있다. 사실, "수정"이라는 단어가 아니라 "생성"으로 봐도 무방하다. )

```
$ vim ~/.xprofile

export GTK_IM_MODULE=uim
export QT_IM_MODULE=uim
uim-xim &
export XMODIFIERS=@im=uim
```


그리고, 아래의 명령어로 설정창을 열어준 뒤에, 다음 사진들과 같이 세팅을 해주고 재부팅을 하면, 끝이다. 간단하다.
```
$ uim-pref-gtk
```
![2](/assets/images/2020-08-27-05-39-27_2020-08-27-linux_1.md.png){: .alignCenter}  
![3](/assets/images/2020-08-27-05-39-51_2020-08-27-linux_1.md.png){: .alignCenter}  
![4](/assets/images/2020-08-27-05-40-07_2020-08-27-linux_1.md.png){: .alignCenter}  



저번에 우분투를 썼을 당시에, 한글 키 입력에 고생을 좀 많이해서, 한영변환을 컨트롤+스페이스로 하는것에 익숙해져있어서 딱히 수정하지 않았는데 (이번에도 또 골머리 아플 것 같아서..)



만약 이에 익숙해있지 않은 사람이라면, 다음을 테스트해보면 좋을 것 같다. 잘 되는지는 안해봐서 모르겠다.


```
$ vim ~/.xprofile

xmodmap -e 'remove mod1 = Alt_R'
xmodmap -e 'keycode 108 = Hangul'
xmodmap -e 'remove control = Control_R'
xmodmap -e 'keycode 105 = Hangul_Hanja'

:wq
```

그리고 벼루 키 바인딩 1에서 벼루on, off를 기존에 사용하던 한글 버튼으로 grab해주면 된다.

# 2. 터치패드 제스처
아무래도 랩탑을 사용하다 보니, 터치패드+마우스를 동시에 사용하는데 적응 되어있었고, 또 윈도우에서도 맥 못지않게 여러손가락으로 터치패드를 사용하도록 제공해줘서 편리하게 사용했는데, 리눅스에서는 이런 부분이 좀 부족하다.

[libinput-gestures](https://github.com/bulletmark/libinput-gestures/blob/master/README.md) 라는 녀석이 멀티터치 제스쳐를 제공해준다고 한다.

아주 친절하게 설치 방법을 알려주므로, 고대로 따라하면 된다. 이거보다 더 쉽게 설명하기는 어려울 듯

# 3. 카카오톡
사실 카카오톡이 우분투를 메인os로 사용하다가 버리게 된 가장 큰 이유일 것이다. 걸핏하면 죽어버리고, 안되는 것도 많고.. 그렇다고 리눅스를 버릴지언정 카카오톡을 버릴순 없으니, 이번에 다시 한번 깔아보았다. 문제가 생기는건 여전하지만;

일단 기본적으로 카카오톡은 리눅스를 지원하지 않는다. 해줄게요~ 라는 말이 나온지가 약 2년전인거같은데, 아직도 안해주는거보니까 그냥 말만 해놓고.... 아무튼, 그래서 wine이라는 녀석이 필요한데 얘는 "리눅스에서 윈도우 프로그램을 실행하게 해주는 놈"이라고 생각하면 된다. 물론 완벽하게 호환이 되지는 않는다.

아무튼, 그래서 설치 방법은


```

$ sudo pacman -S wine winetricks cabextract
$ WINEARCH=win32 WINEPREFIX=~/.wine wine wineboot
$ wget https://raw.githubusercontent.com/Winetricks/winetricks/master/src/winetricks
$ chmod 777 winetricks
$ winetricks
-> Select the default wineprefixaa 체크
-> Install a Windows DLL or component 체크
-> gdiplus, msxml6, riched30, wmp9 체크한 뒤에 설치 (찾기가 상당히 귀찮다)

```

이렇게까지 마치고 난 뒤에, 카카오톡 홈페이지에 가서 윈도우용 설치파일을 다운받는다.

설치된 경로로 이동한 뒤에, 터미널에서

```
$ wine KakaoTalk_Setup.exe

```

를 하면 익숙한 설치 절차가 진행된다.

그런데, 나처럼 영문판을 기본으로 쓰고있다면 한글이 이상하게 깨진다던지, 혹은 한글 자체는 멀쩡하게 보이는데 한글입력기가 제대로 먹지 않는다. 이럴 때에는
```
$ cd ~/.local/share/applications/wine/Programs/KakaoTalk
$ vim KakaoTalk.desktop
```

그리고 굵은 글씨로 표시된 부분을 대충 집어넣어주자.



[Desktop Entry]
Name=KakaoTalk
Exec=env WINEPREFIX="/home/cjlee/.wine" LANG="ko_KR.UTF-8" wine C:\\\\windows\\\\command\\\\start.exe /Unix /home/cjlee/.wine/dosdevices/c:/ProgramData/Microsoft/Windows/Start\\ Menu/Programs/KakaoTalk/KakaoTalk.lnk
Type=Application
StartupNotify=true
Path=/home/cjlee/.wine/dosdevices/c:/Program Files/Kakao/KakaoTalk
Icon=12CF_KakaoTalk.0
StartupWMClass=kakaotalk.exe



이렇게하면 애지간하면 잘 될것으로 보인다.

근데, 문제는 와인 업데이트하고나니까 갑자기 한글이 안된다... 한숨이 절로 나오더라.

그리고 아직 많이 써보지 못했지만, 두 가지의 크리티컬한 문제점을 발견했는데,

첫 번째는, 알림이 오면 카톡이 죽어버린다. 카톡에서 알림을 꺼놨는데도 자꾸 죽어버리더라

두 번째는, 사진 전송이 안된다; 파일 전송까지는 괜찮은데 사진을 전송할 수가 없다.



두 번째 문제야 어떻게 하더라도, 첫 번째 문제가 아주 골머리가 아파서, 아직은 미뤄두고 있는 상태이긴 한데, 시도해볼만한 것으로는 1. PlayOnLinux를 깔고 그 위에 카카오톡을 설치하는 것, 2. (많이 비효율적이지만) 가상머신을 돌리는것.



사실 가상머신을 띄워놓으면 왠만한 윈도우 프로그램을 사용하는데에는 문제가 없겠지만, 그럼 리눅스를 쓰는 의미가 퇴색되는 것 같아서 최후의 보루로 미뤄놓고 있는 상태이긴 한데, 지금같은 흐름이라면... 많이 좋지는 않다.



# 4. 오피스
사실 wine 을 이용해서 오피스를 설치하려고 했는데, 설치방법 찾아보기도 귀찮고 해서, 나름 마소 오피스와 잘 호환이 된다는 오픈오피스를 사용해보려고 한다. 물론 다시 학기가 시작되고, 여러가지 문서작업이나 pdf에 주석다는 짓거리를 하다보면 언제 또 욕나오는 상황에 맞닥드릴지는 모르겠지만 ..

# 5. 배터리 관리
정신없이 이것저것 설치하다보면, 어느새 박살나있는 배터리를 보고 "원래 이랬나?" 싶은 숫자가 눈앞에 나타나있다. 리눅스 자체에서 배터리 관리에 크게 신경을 쓰지 않는다니, 뭐..( DIY 정신이 너무 많이 깃들어있는거 아닌가 싶기도 한데)

아무튼, 자세히 잘 알지도 못하고, 알고 싶지도 않아서, 대충 tlp라는 놈을 설치했다

```
$ sudo pacman -S tlp
$ sudo tlp start
```


한 번 실행해놓으면 다음부터 자동으로 실행된다니, 잊어버리고 살아도 되는 듯 하다.

이정도로 대충 기본적인 세팅 정도는 해두었는데, 쓰면서 얼마나 많은 윈도우 프로그램과 마주하고, 또 와인을 얼마나 많이 파악해둬야 할지 감이 안오니, 이건 뭐 가상머신을 진짜 띄워야 하나.. 물론 아직 써보지는 않았지만, 각종 은행 혹은 정부기관의 공인인증서나, 혹은 여러가지 아주 진드기같은 보안 프로그램들을 설치할 상상을 하니 벌써부터 행복해서 죽을 것 같다.

