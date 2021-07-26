아래 경고 수정은 'Visual Studio 2017 Professional' 기준 입니다.

`"warning C4819: 현재 코드 페이지(949)에서 표시할 수 없는 문자가 파일에 들어 있습니다."`  

&nbsp;

> 구글링 결과, Visual Studio 한글판을 사용하면 발생하는 문제라는게 대부분.  
해당 Header의 인코딩 방식이 잘못되어 있다는 건데,  
보통 주석을 한글로 쓰거나 하면 문제가 생길 것 같은데..  
문제가 발생한 Header는 Google Test Include 파일 이었다.  
"gtest-internal.h" 이놈.  

내가 사용한 방법은 Visual Studio에서 해당 Header를  
`Open` -> `다른 이름으로 저장` -> `인코딩 하여 저장` -> `기존 파일 덮어쓰기` 였다.  
사라지긴 하는데, 이래도 되나 싶네.

&nbsp;

***

`"warning C4996 : tr1 ..."`  

&nbsp;

*[참고 블로그](https://m.blog.naver.com/hirit808/221615764755)*  

> C 언어 입문자도 볼 수 있는 Warning 임.  
scanf라는 표준 함수를 사용하면 최근 Visual Studio에서 보안이슈로 인해 위 경고가 뜬다.  
해당 경고에 걸리는 함수들은 대부분 BOF(버퍼 오버플로우)의 위험을 지니고 있으며,  
BOF는 시스템해킹에 악용되는 취약점을 지니고 있어서 보안을 강화하기 위해 scanf 대신,  
scanf_s라는 표준 함수를 사용 해야 한다. (이를 위한 경고인가)  
내가 발생한 문제는 또 Google Test인데, 난 scanf를 쓰지 않았..  

외부로 노출되는 코드도 아니고 내부에서 단위테스팅용으로만 사용하는 거니께  
프로젝트의 [모든 구성 Win32/x64]에 명령줄(/wd4996)을 추가해서 경고 자체를 막아버림.  
`프로젝트 속성` -> `구성 속성` -> `C/C++` -> `명령줄` -> `추가 옵션` -> `/wd4996` 추가  

&nbsp;

***

`_WIN32_WINNT not defined. Defaulting to _WIN32_WINNT_MAXVER (see WinSDKVer.h)`

&nbsp;

> 딱히 이유는 못 찾았고 Visual Studio에 32 bit의 Version 정보가 있나보다.

미리 컴파일된 헤더(pch.h or stdafx.h)에 `#include <sdkddkver.h>`을 추가했다.

&nbsp;

***

`D9035 : 'Gm'옵션은 더 이상 사용되지 않으며 향후 릴리스에서 제거 될 예정입니다.`

&nbsp;

> 예전에는 컴파일 속도를 올리기 위해 __최소 다시 빌드 기능__ 을 이용했던 것 같은데,  
이제는 없어 졌나 보다.. 향후 릴리스에서는 제거 할 예정이니까 끄라는 이야기 인듯.

`프로젝트 속성` -> `구성 속성` -> `C/C++` -> `코드 생성` -> `최소 다시 빌드 기능` -> `아니요 (/Gm-)` 으로 변경.  

&nbsp;

***

`warning C4477: 'fwprintf' : 서식 문자열 '%12s'에 'wchar_t *' 형식의 인수가 필요하지만 variadic 인수 5의 형식이 'const _Elem *'입니다.`

&nbsp;

> 실제 사용처에서는 _ftprintf를 사용했지만 컴파일러가 변환하는 과정에서 fwprintf로 표기하게 된다. (한국어니까 유니코드로 되서 그른가..)
현재 사용하고 있는 struct의 변수가 std::string 이니께 c_str() 뽑아오면 const _Elem* 는 결국 const char*가 될테고..
근데 함수는 wide character를 원하고 있으니 경고를 내주는게 당연.

강제 캐스팅이라 이리 해도 될지는 모르겠으나.. `(const TCHAR*)(std::string.c_str())`으로 해결.

&nbsp;


***

`warning C4723: 0의 나누기가 발생할 수 있습니다.`
`warning C4723: potential divided by 0`

&nbsp;

> 로직에 0을 넣어두고 0으로 나누려고 해서 발생하는 경고  
대부분이 개발자 에러 일 듯 하고..  
SafeDiv를 사용하기 때문에 출력창의 경고를 더블클릭해서 바로 코드로 접근하는 건 불가능하고  
SafeDiv를 임시 매크로 #define __test__(a, b) a / b로 치환하고  
실제 Code Line에 접근해서 로직을 하나씩 뜯어보면서 문제를 수정해야함 ㅠㅠ

임시 매크로를 만들어서 치환 후에 해당 프로젝트 Rebuild, 문제 Code에 직접 접근해서 로직 파악 후 하나씩 수정.

&nbsp;
