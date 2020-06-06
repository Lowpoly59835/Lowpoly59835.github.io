windbg 명령에서 몇몇 명령어는 다음과 같은 메세지와 함께 실행되지 않는다.

<span style="color:red"> No export process found </span>

내가 겪은건 두 가지 이유가 있었다.  
커널에대한 권한과 디버깅 확장 프로그램 미설치였다.

## sos.dll

구글에서 소개하는 명령어중 좀 쓸만한건 전부 sos.dll을 통환 확장 명령어였다..

sos.dll도 .Net Framework 버전/플랫폼 별로 선택해서 로드해야한다는 미친 버전 관리를 보여준다.  
.Net Framework 버전은 그렇다쳐도 플랫폼이란건 무슨 귀신씌나락 까먹는 소리냐했더니 덤프파일이 발생한 머신의 os에 맞는 버전을 써야한다고 하더라.  
진짜 버전/플랫폼 별로 sos.dll이 따로 있더라.(http://www.mskbfiles.com/sos.dll.php)

하지만, 여러번 삽질을 해본 결과 큰 차이가 없는 버전에선 굳이 맞출 필요는 없다. 권장사항일뿐 필수사항은 아니었다.

그래서 내가 사용할 버전은 뭘로해야하나했더니, 닷넷 프레임워크에 이미 설치가 되어있더라.

경로는 다음과 같다.

    C:\Windows\Microsoft.NET\Framework64\v4.0.30319

해당 폴더에 sos.dll이 있다.

그후에 windbg에서 !load 명령을 통해서 해당 sos.dll을 로드하면 된다.

    !load " C:\Windows\Microsoft.NET\Framework64\v4.0.30319\SOS.dll"
---
layout: post
title:  "개인공부 시스템 프로그래밍"
author: "Chester"
comments: false
---



주의사항으로는 windbg의 확장 폴더가 있는데, 여기에 sos.dll을 넣어서는 안된다.

    C:\Program Files (x86)\Windows Kits\10\Debuggers\x64\winext

왜냐하면, sos.dll은 단독 라이브러리가 아니라 clr.dll이나 다른 dll을 참조해야하는데, 그것들을 일일히 같이 가져와야하기 때문이다.


## kernal mode

windbg를 로컬 커널 모드로 실행시키는 방법은 두 가지가 있다.

하나는 file-kernal debug-local-ok
하나는 .attach -k 후에 g 명령를 내리는 것이다.

하지만, 무작정 실행시키면 에러와 함께 로컬 커널 모드로 진입할 수가 없다.


커널 모드로 진입하기 전에 관리자 권한으로 프롬프트를 실행시켜서 bcdedit /dbgsettings local를 입력하고 재부팅을 해야한다.



SOS.dll 확장 모듈을 버전 별로 구하는 방법 (https://www.sysnet.pe.kr/2/0/1654)  
windbg ".loadby sos" 명령어 (https://www.sysnet.pe.kr/2/0/943)  
Failed to find runtime DLL (mscorwks.dll), 0x80004005 (https://www.sysnet.pe.kr/Default.aspx?mode=2&sub=0&detail=1&pageno=0&wid=1020&rssMode=1&wtype=0)  
Windbg - Local Kernel Debug 모드 (https://www.sysnet.pe.kr/2/0/934)  
windbg - 닷넷 메모리 덤프에서 전역 객체의 내용을 조사하는 방법(https://www.sysnet.pe.kr/2/0/11460)   
Setting Up Local Kernel Debugging of a Single Computer Manually(https://docs.microsoft.com/ko-kr/windows-hardware/drivers/debugger/setting-up-local-kernel-debugging-of-a-single-computer-manually)
