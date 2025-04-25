---
title: (GCC, C/C++) 임의의 디렉토리에 있는 외부 라이브러리 링킹
date: 2024-10-23 21:20:00 +0900
categories: [개발, C/C++]
tags: [c, c++, gcc, 링커, 링킹, linux] # TAG names should always be lowercase
math: true
---

이걸 하게된 이유는 두가지가 있습니다.      

하나는 예전에 라이브러리 가져다 쓸 때 자꾸 라이브러리를 찾지 못한다는 에러를 띄워대서 힘들었던 경험   
다른 이유는 이번 중간고사 준비에 필요했기 때문입니다.   

제 중간고사 중 리눅스 + C언어로 보는 실기시험이 있습니다. 그리고 시험을 볼 리눅스 환경에 아무 파일이나 미리 준비해서 넣을 수 있게 되었는데, '그러면 그냥 자료구조같은건 오픈소스 라이브러리 가져다가 쓰면 편하지 않나?' 라는 생각이 들었습니다.   

근데 보통 오픈소스 라이브러리는 `make install` 하면 /usr/local/ 에 설치되는게 디폴트죠. 그런데 sudo 를 쓸 수 없는 계정 + cmake가 없어서 라이브러리를 빌드할 수 없는 실습 환경이라 빌드되어서 만들어진 `.so`(shared library) 나 `.a`(static library) 파일을 가져다가 홈디렉토리에 놓고 써야했습니다.   

아무튼 그런 이유에서 아무 디렉토리에 `(.so .a)`라이브러리 파일을 두고 쓰는 방법을 정리하게 되었습니다.   
   
## 우선 라이브러리를 빌드하자
---
   
각자 잘 빌드하면 되는데, 그래도 예시를 하나 가져오겠습니다.   
<https://github.com/srdja/Collections-C>
   
제가 쓰고자 한 임의의 타입`(void*)`을 지원하는 자료구조 라이브러리입니다.   
   
라이브러리들은 거의 다 `README.md` 같은데에 빌드 방법을 적어놓습니다. 여기서도 그대로 따라하면 됩니다.   
   
단계별 설명 대신에 과정을 정리만 해보겠습니다.
```
! 필요하면 sudo를 붙일것 !

-- github 오픈소스 레포지토리 기준 공통과정 --
1. git clone <repository 주소> # 라이브러리 소스코드를 다운로드 받습니다
2. cd <repository 이름> # 다운로드 받은 라이브러리 소스코드 디렉토리에 들어갑니다
-- 해당 라이브러리의 빌드 방법 --
3. mkdir build 
4. cd build
5. cmake ..-DSHARED=false # 동적 라이브러리로 만들고 싶다면 -DSHARED=true
6. make
7. make install 

8. 디폴트로는 
/usr/local/lib/libcollectc.a # shared library 라면 .so
/usr/local/lib/pkgconfig/collectc.어쩌구
/usr/local/include/collectc/ # 헤더 파일

가 만들어질 것입니다.
```
근데 얘는 static library 만들면 헤더 파일을 안만들어줬어서 저는 build/ 디렉토리 지우고 shared library 로 한 번 더 설치해서 `/usr/local/include/collectc/` 까지 챙겨줬습니다.
   
## 우리의 프로젝트에 가져오기
---
이제 만들어진 라이브러리 파일`(.so / .a)` 과 헤더 파일이 들어있는 디렉토리를 원하는 디렉토리에 가져와줍시다.   
예를 들어, 프로젝트 루트 폴더에 두겠습니다.   
예시는 위에서 빌드한 라이브러리를 쓰겠습니다.
``` sh
$ tree
.
├── Makefile
├── include
│   └── collectc
│       ├── cc_array.h
│       ├── cc_array_sized.h
│       ├── cc_common.h
│       └── cc_tsttable.h
...
│
├── libs
│   └── libcollectc.a
└── src
    └── main.c
```

이런 프로젝트 구조라면, main.c 를 아래와 같이 빌드해서 라이브러리를 링킹할 수 있습니다.
``` bash
gcc main.c -o main -lcollectc -I./include -L./libs
```

이제 이 라이브러리는 잘 링킹되어서 `#include <collectc/헤더 파일 이름.h>` 으로 쓸 수 있게 되었습니다.

shared library 의 경우에는 유사하게 세팅한 후에 링킹 옵션을 하나 추가하면 됩니다. `gcc ... -Wl,-rpath=./libs`
```bash
gcc main.c -o main -lcollectc -I./include -L./libs -Wl,-rpath=./libs
```
이 경우는 참고 자료중 `Using a Dynamic Library with GCC` 에 자세히 나와있습니다.


---

### 참고 자료
<https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/7/html/developer_guide/gcc-using-libraries#gcc-using-libraries_using-static-library-gcc   >

<https://github.com/srdja/Collections-C>