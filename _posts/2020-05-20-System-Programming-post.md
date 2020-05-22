---
layout: post
title:  "개인공부 시스템 프로그래밍"
author: "Chester"
comments: false
---


지금까지 시스템 프로그래밍 보면서 배웠던거 정리할려고 함

OCW에서 서강대의 서정연 교수님이 시스템 프로그래밍 강의 영상 올려주신걸 보고 정리함

이전과 마찮가지로 중요한 내용을 정리하진 않음

그건 너무 오래걸림

걍 내 안의 내용들을 정리하려함

참고로 책은 강의에서 쓴
L. L. Beck, “System Software : An Introduction to Systems
Programming”, 3rd Edition, Addison-Wesley Publishing, 1997


또한, 챕터 1부터 챕터2 초반의 내용까지만 읽었기에 딱 거기까지만 하겠음


## 본문
* [SIC와 SIC/XE 구조](#chapter-1)
* [SIC와 SIC/XE 명령어 구조](#chapter-2)
* [디바이스 명령](#chapter-3)
* [예제의 명령어를 Object Code로 만들기](#chapter-4)
* [Object Program](#chapter-5)
* [꼭 봐야할 것](#chapter-6)

-----------------------------------------------------

### 1. SIC와 SIC/XE 구조 <a id="chapter-1">

책에서는 386, 486, IBM 시대에 쓰던 컴퓨터 아키텍처를 다룸 (아니면 그 전인가?)

SIC, SIC/XE는 각각 3바이트 메모리, 4바이트 메모리 SIC/XE임  
둘다 워드/버스는 3바이트

레지스터는 CPU가 예약해놓은 전용 레지스터와 사용자가 사용하기위한 범용 레지스터가 있음

    A = 누산기 레지스터  
    X = 인덱스 레지스터  
    L = 링크 인덱스 (서브 점프했을 떄에 돌아갈 주소를 저장)  
    PC = 프로그램 카운터  
    SW = Status Word 인덱스 (CC를 포함하기에 비교연산의 결과를 저장하는데 사용됨)  

이하는 XE 머신에 추가된 레지스터  

    B = Base 레지스타
    T = S = 범용 레지스터
    F = 소수 연산을 위한 레지스터 (48bit)


-------------------------------------------------

### 2. SIC와 SIC/XE 명령어 구조 <a id="chapter-2">

SIC의 명령어는 3바이트 체계로 하나의 포맷만 존재함

    OP Code(1Byte) / X (1bit) / Address (15bit)


SIC/XE는 4개의 포맷이 존재함  
메모리가 4바이트로 늘어난건 좋은데, 머신의 버스가 여전히 한 번에 3바이트를 가져올 수 있다. 그럼으로서 3바이트 명령어안에 4바이트 주소를 담을 수 없는 이슈가 생김  
그래서 3바이트 포맷으로 4바이트 메모리를 참조하려는 포맷이 생김

1. 1바이트 포맷    
    OP Code (1Byte)
2. 2바이트 포맷  
    OP Code (1Byte) / R1(4BIt) / R2(4Bit)
3. 3바이트 포맷  
    OP Code (6bit) / imxbpe (6bit) / disp (12bit)
4. 4바이트 포맷  
    OP Code (6bit) / inxbpe (6bit) / address (20bit)


1,2 포맷은 설명안함
R1,R2는 걍 레지스터를 뜻함
레지스터는 4bit로 충분히 표현가능한 범위임

3,4 포맷은 imxbpe 비트로 Addressing Mode가 정해짐
Addressing Mode란 주소를 참조하는 방법을 말함

우선 in/x/bp/e 단위로 묶어서 봐야함
쉬운 것부터 설명함

X 비트는 SIC와 동일하게 인덱스 비트임, 인덱스 레지스터를 사용한다면 1이 됨  
E 비트는 4바이트 포맷의 사용 여부를 말함. 1이면 해당 명령어는 4바이트 포맷 명령어가 됨  
b/p 비트는 각 값에 따라 아래를 뜻함     

    b = 0 && p = 1 이면 pc-relative  
    b = 1 && p = 0 이면 base-relative  
    b = 0 && p = 0 이면 direct address

pc-relative는 pc를 이용함. disp = TA-PC임 걍 대부분 **TA-PC = disp**를 기본적으로 씀 SIC에선ㅇㅇ  
Base-Relative는 PC-TA를 하면 결과값이 12bit(-2048~2048)를 넘어감. 그때 쓰는게 Base-relative임 걍 PC-Relatvice가 안되면 이걸 쓰는거임. 이때, PC대신에 Base 레지스터를 사용함. **TA - B register = disp**를 하면 disp가 나옴  
둘다 0이면 걍 계산하지말고 operation address에 써있는 값을 그대로 가져가란 뜻임. 주로 4바이트 명령어 혹은 operation address에 있는 값을 그대로 쓸 때 사용함

im비트는 각 값에 따라 아래를 뜻함

 i = 1 && n = 0 이면 immediate address
 i = 0 && n = 1 이면 indirect address
 i = 1 && n = 1 이면 simple address
 i = 0 && n = 0 이면 SIC 머신

 immediate address는 disp or address에 있는 값을 바로 사용하라는 뜻
 indirect address는 disp or address의 있는 값에 해당하는 주소지에 있는 값을 사용하라는 뜻
 simple address는 disp or address를 볼 필요없이 direct address처럼 operation address에 있는 값을 쓰라는 뜻

 
 ---------------------------


### 3. 디바이스 명령 <a id="chapter-3">

 divece 명령이 있음
 주로 TD,RD,WD
 RD/WD는 이름 그대로 Device에 Read/Write하라는 뜻
 TD는 RD/WD를 하기전에 해당 디바이스가 IO할 수 있는 상태인지 체크하는거임
 Device 명령어의 결과는 A가 아닌 CC(SW 레지스터)에 저장됨.
 TD후에 SW 레지스터에 =가 있으면 사용할 준비가 되지 않았다는 뜻

-------------------------------

### 4. 예제의 명령어를 Object Code로 만들기  <a id="chapter-4">

Op Code는 하드웨어에서 정해놓았기에 OP Table은 프로그래머가 만들어도 값은 하드웨어를 따라야함
SIC와 SIC/XE에서 쓰는 OP 테이블은 다음과 같음


    // Operation of Assemble
    {"ADD",0x18,3,0},   {"ADDF",0x58,3,1},  {"ADDR",0x90,2,1},  {"AND",0x40,3,0},
    {"CLEAR",0xB4,2,1}, {"COMP",0x28,3,0},  {"COMPF",0x88,3,1}, {"COMPR",0xA0,2,1},
    {"DIV",0x24,3,0},   {"DIVF",0x64,3,1},  {"DIVR",0x9C,2,1},  {"FIX",0xC4,1,1},
    {"FLOAT",0xC0,1,1}, {"HIO",0xF4,1,1},   {"J",0x3C,3,0},     {"JEQ",0x30,3,0},
    {"JGT",0x34,3,0},   {"JLT",0x38,3,0},   {"JSUB",0x48,3,0},  {"LDA",0x00,3,0},
    {"LDB",0x68,3,1},   {"LDCH",0x50,3,0},  {"LDF",0x70,3,1},   {"LDL",0x08,3,0},
    {"LDS",0x6C,3,1},   {"LDT",0x74,3,1},   {"LDX",0x04,3,0},   {"LPS",0xD0,3,1},
    {"MUL",0x20,3,0},   {"MULF",0x60,3,1},  {"MULR",0x98,2,1},  {"NORM",0xC8,1,1},
    {"OR",0x44,3,0},    {"RD",0xD8,3,0},    {"RMO",0xAC,2,1},   {"RSUB",0x4C,3,0},
    {"SHIFTL",0xA4,2,1},{"SHIFTR",0xA8,2,1},{"SIO",0xF0,1,1},   {"SSK",0xEC,3,1},
    {"STA",0x0C,3,0},   {"STB",0x78,3,1},   {"STCH",0x54,3,0},  {"STF",0x80,3,1}, 
    {"STI",0xD4,3,1},   {"STL",0x14,3,0},   {"STS",0x7C,3,1},   {"STSW",0xE8,3,0},
    {"STT",0x84,3,1},   {"STX",0x10,3,0},   {"SUB",0x1C,3,0},   {"SUBF",0x5C,3,1},
    {"SUBR",0x94,2,1},  {"SVC",0xB0,2,1},   {"TD",0xE0,3,0},    {"TIO",0xF8,1,1}, 
    {"TIX",0x2C,3,0},   {"TIXR",0xB8,2,1},  {"WD",0xDC,3,0 },
    // 어셈블러 지시자
    {"BASE",0,0,0},  {"NOBASE",0,0,0}, {"BYTE",1,0,0}, {"END",2,0,0},   {"EQU",7,0,0},
    {"LTORG",8,0,0}, {"RESB",3,0,0},   {"RESW",4,0,0}, {"START",5,0,0}, {"WORD",6,0,0},
    {"USE",9,0,0},   {"CSECT",10,0,0}, {"EXTREF",11,0,0}, {"EXTDEF",12,0,0}


------------------------------------------------------

이건 SIC 머신의 명령 예제임
나중에 다시 봤을떄, 혹시 기억못하면 어떻게든 Object Code를 만들어보셈  
생각보다 도움많이 됨. 이렇게 명령어를 만든다는걸 알 수 있고 뭣보다 만들어보면 재밌음 ㅎㅎ


그리고 어짜피 SIC든 SIC/XE든 내 머리로는 분명 책을 보든 구글링을 하던 어떻게든 해야됨
내가 이걸 매일 해야할 직업은 아니니까 분명히 잊어먹을거임 걍 필요할떄가 오면 책이든 강의든 뭐든 보고 하셈

    symble  Layble      operation   operation address   Disp&Address

    1000    COPY        START       1000            
    1000    FIRST       STL         RETADR              141033  L레지스터의 값을 RETADR에 저장 / L레지스터란, 서브루틴이 끝나고 다시 돌아와야할 위치를 저장하는 레지스터
    1003    CLOOP       JSUB        RDREC               482039  RDREC로 점프, 점프할 때에 현재 PC를 L레지스터에 저장한 다음에 PC에 RDREC를 저장한다.
    1006                LDA         LENGTH              001036  LENGTH를 A레지에 저장
    1009                COMP        ZERO                281030  ZERO와 A레지를 비교해서 값을 SW 레지스터에 저장
    100C                JEQ         ENDFIL              301015  SW 레지가 =이라면 ENDFIL로 점프 (읽은 길이가 0라면), 0이라면 출력시키자 않고, RETADR에 저장된 위치로 이동함
    100F                JSUB        WRREC               482061  WRREC로 서브루틴. 읽은 값을 출력시키는 서브루틴
    1012                J           CLOOP               3C1003  출력시키고 난 후에는 CLOOP로 점프
    1015    ENDFIL      LDA         EOF                 00102A  EOF를 A레지에 저장
    1018                STA         BUFFER              0C1039  A레지값을 BUFFER에 저장 (BUFFER를 EOF로 변경)
    101B                LDA         THREE               00102D  THREE를 A레지에 저장
    101E                STA         LENGTH              0C1036  A레지를 LENGTH에 저장 (BUFFER길이를 3으로 변경, EOF가 3글자니까)
    1021                JSUB        WRREC               482061  WRREC로 점프
    1024                LDL         RETADR              081033  RETADR를 L레지에 저장
    1027                RSUB                            4C0000  서브루틴을 끝내고, L레지에 저장된 값으로 점프
    102A    EOF         BYTE        C'EOF'              454F46  C'EOF'의 아스키코드
    102D    THREE       WORD        3                   000003
    1030    ZERO        WORD                            000000
    1033    RETADR      RESW        1                           WORD만큼 공간 할당. 공간할당에는 값이 없음
    1036    LENGTH      RESW        1                           
    1039    BUFFER      RESB        4096                        4096 바이트만큼의 공간 할당
            4096 BYTE Alloc                         
    2039    RDREC       LDX         ZERO                041030  ZERO를 X레지스터에 저장
    203C                LDA         ZERO                001030  ZERO를 A레지스터에 저장
    203F    RLOOP       TD          INPUT               E0205D  X'F1' 디바이스에 TestDevice, 결과는 SW 레지스터(CC)에 저장
    2042                JEQ         RLOOP               30203F  cc의 값이 =일때에, RLoop로 점프
    2045                RD          INPUT               D8205D  Device에서 읽은 1바이트 값을 Input에 저장하고 A레지에 저장(오른쪽 바이트 우선)
    2048                COMP        ZERO                281030  A레지와 Zeror와 비교. >,<,=의 값을 SW레지스터에 저장
    204B                JEQ         EXIT                302057  A레지와 SW와 비교하여, 같다면 EXIT로 점프 (Device에서 읽어온 값이 0라면, 더이상 읽지 않음)
    204E                STCH        BUFFER, X           549039  A레지의 (오른쪽 바이트)CHAR하나를 Buffer의 X비트 위치에 저장
    2051                TIX         MAXLEN              2C205E  X레지스터를 하나 증가키고 MAXLEN과 비교, 결과를 SW레지스터에 저장
    2054                JLT         RLOOP               38203F  SW레지스터의 값이 <라면, RLOOP로 점프(지금까지 읽은 횟수가 MAXLEN이상이라면 더이상 읽지 않음(버퍼의 길이 이상으로 읽음))
    2057    EXIT        STX         LENGTH              101036  X비트를 LENGTH에 저장(읽은 길이, 횟수)
    205A                RSUB                            4C0000  L레지스터의 값으로 점프(서브 루틴이 끝나고 난 뒤에 L레지스터에 저장된 위치로 돌아감), 이 경우 CLOOP에 해당
    205D    INPUT       BYTE        X'F1'               F1
    205E    MAXLEN      WORD        4096                001000

    2061    WRREC       LDX         ZERO                041030 ZERO를 X레지에 저장
    2064    WLOOP       TD          OUTPUT              E02079 OUTPUT 디바이스에 TestDevice, 결과는 SW 레지스터에 저장
    2067                JEQ         WLOOP               302064 SW의 값이 =일때에, WLOOP로 점프. TestDevice에서 준비완료 응답이 올 때까지 루프
    206A                LDCH        BUFFER,X            509039 Buffer의 X위치에서 CHAR하나만큼의 데이터를 A레지에 저장
    206D                WD          OUTPUT              DC2079 A레지를 OUTPUT Device에 씀
    2070                TIX         LENGTH              2C1036 X레지를 1 증가시키고 LENGTH와 비교, 결과는 SW에 저장
    2073                JLT         WLOOP               382064 SW레지가 <라면, WLOOP로 저장
    2076                RSUB                            4C0000 서브루틴을 끝내고, L레지에 저장된 값으로 점프
    2079    OUTPUT      BYTE        X'05'               05
                        END         FIRST               

------------------------------------------------------

이건 SIC/XE 머신의 명령 예제임  
동작하는건 위와 똑같음
이것도 해보는걸 추천한다만, 몇가지 알려주겠음

SIC/XE의 OP Code는 SIC에 비해 2bit 적음
그래서 Operation Table에 있는 값의 맨 오른쪽(끝)의 2bit를 짜르고 거기를 i/m bit로 써야함

또 여기선 기호가 있는데 
#은 immdiate address를 뜻함
@은 indirect adress를 뜻함
+는 4바이트 명령어로 확장을 뜻함
ㅅㄱ링


    symble  layble      operation   operation address   Disp&Address

    0000    COPY        START       0                   0   
    0000    FIRST       STL         RETADR              17202D    
    0003                LDB         #LENGTH             69202D        
                        BASE        LENGTH              
    0006    CLOOP       +JSUB       RDREC               4B101036
    000A                LDA         LENGTH              032026
    000D                COMP        #0                  290000
    0010                JEQ         ENDFIL              332007
    0013                +JSUB       WRREC               4B10105D
    0017                J           CLOOP               3F2FEC
    001A    ENDFIL      LDA         EOF                 032010
    001D                STA         BUFFER              0F2016
    0020                LDA         #3                  010003
    0023                STA         LENGTH              0F200D
    0026                +JSUB       WRREC               4B10105D
    002A                J           @RETADR             3D2003
    002D    EOF         BYTE        C'EOF'              454F46
    0030    RETADR      RESW        1     
    0033    LENGTH      RESW        1     
    0036    BUFFER      RESB        4096  
        4096 BYTE Alloc               
    1036    RDREC       CLEAR       X                   B410   
    1038                CLEAR       A                   B400
    103A                CLEAR       S                   B440
    103C                +LDT        #4096               75101000
    1040    RLOOP       TD          INPUT               E32019
    1043                JEQ         RLOOP               332FFA
    1046                RD          INPUT               DB2013
    1049                COMPR       A,S                 A004
    104B                JEQ         EXIT                322008
    104E                STCH        BUFFER, X           57C003
    1051                TIXR        T                   B850
    1053                JLT         RLOOP               3B2FEA
    1056    EXIT        STX         LENGTH              134000
    1059                RSUB                            4F0000
    105C    INPUT       BYTE        X'F1'               F1   

    105D    WRREC       CLEAR       X                   B410
    105F                LDT         LENGTH              734000
    1062    WLOOP       TD          OUTPUT              E32011
    1065                JEQ         WLOOP               322FFA
    1068                LDCH        BUFFER,X            53C003
    106B                WD          OUTPUT              DF2008
    106E                TIXR        T                   B850
    1070                JLT         WLOOP               3B2FEF
    1073                RSUB                            4F0000           
    1076    OUTPUT      BYTE        X'05'               05  
            END         FIRST                    


-----------------------------------------

### 5. Object Program  <a id="chapter-5">

SIC/XE 어셈블러가 위의 코드를 실행시키면 나오는 결과 파일을 말함
이 나중에 프로그램을 메모리에 올리는 로더가 Object Program 파일을 보고 메모리를 올림  
사실상 exe 파일

여튼 정해진 포맷이 있으나 간략하게 설명하자면

3개의 영역이 있음

Headr / Text / End


    1. Headr
        프로그램 이름, 프로그램이 시작할 메모리 위치, 프로그램의 길이(사용할 메모리 영역 범위)를 저장함
    2. Text
        명령어들이 모여있음. 실제 이걸 보고 동작함.
    3. End
        프로그램이 시작할 첫번쨰 명령의 주소를 저장함

자세한건 책이나 강의를 보셈

------------------------------------------

### 6. 꼭 봐야할 것  <a id="chapter-6">

길어질 것같아서 그냥 스킵하겠는데
어셈블러에 포함되어야할 기능들을 적어놓은게 있음

    1. 2 Pass
        어셈블러가 기계어로 번역할 떄에 2개의 공정을 거침
        1패스에서 주로 유요한 명령어 검사, 다이렉트 명령어 실행, 2 패스에서 사용할 레이블 할당
        2패스에서는 명령어 번역, 메모리에 데이터 정의, 패스1에서 남은 다이렉트 명령어 실행, object program 생산
    2. 위의 2패스를 실행시키기 위해 어셈블러에서 가지고 있어야할 메모리 영역이 있음
        첫 번쨰는 Location Counter(LOCCTR) : 현재 번역한 코드 위치
        두 번쨰는 Operation Table (OPTABLE) : 명령어 목록. 주로 해시 테이블 많이 쓴다함
        세 번째는 Symbol Table (SYMTABLE) : 레이블 저장함 이것도 해시 테이블로 만든다함 

나머진 강의랑 책봐라
중요한 부분이다.
