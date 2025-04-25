---
title: emscripten 찍먹기 - 1
date: 2024-09-21 20:55:00 +0900
categories: [개발공부, emscripten]
tags: [emscripten, wasm, c++, c]     # TAG names should always be lowercase
---

요즘 프로젝트로 WebAssembly(wasm)로 웹개발을 하게 될 것 같아서, 일단 한 번 찍어먹어봐야지 싶어서 쓰는 글입니다.   
   
그래서 C++로 개발하는게 일단 익숙하니 emscripten을 써보려고 합니다. 이 글은 일단 emscripten을 써보기 위해 <https://emscripten.org/docs/getting_started/> 튜토리얼 문서를 정리하려 합니다. 문서의 일부 내용만 정리하였습니다.
<br/><br/>
# 설치   
<https://emscripten.org/docs/getting_started/downloads.html>

이 글에서는 linux(내지 WSL) 환경만 정리하였습니다. 참고로 저는 WSL ubuntu 24.04 를 쓰고 있습니다.   
   
### 사전 설치
1. python3 (3.6 버전 이상)   
2. cmake (optional)
3. git (튜토리얼에서는 깃을 통해 SDK를 다운로드 받습니다)

    
   
설치 방법은 아래와 같습니다.   
   
```bash
# Get the emsdk repo
git clone https://github.com/emscripten-core/emsdk.git

# Enter that directory
cd emsdk
```   
우선 위의 방법으로 SDK를 다운로드 받습니다. 이후 다운로드 받은 emsdk 디렉토리로 이동합니다.  

```bash
# Fetch the latest version of the emsdk (not needed the first time you clone)
git pull

# Download and install the latest SDK tools.
./emsdk install latest

# Make the "latest" SDK "active" for the current user. (writes .emscripten file)
./emsdk activate latest

# Activate PATH and other environment variables in the current terminal
source ./emsdk_env.sh
```   
`git pull`은 방금 clone 했을 때는 이미 repo의 최신 버전을 가져왔으므로 안해도 됩니다.   
이제 `./emsdk install latest` 를 통해 최신 버전의 sdk 도구를 설치하고, `./emsdk activate latest` 를 통해 현재 유저에게 sdk를 활성화합니다.   
   
마지막으로, emscripten 컴파일러 등을 쓰기 위해 환경변수를 지정해줍니다. `source ./emsdk_env.sh` 해당 셸 스크립트로 환경변수를 설정해줍니다. 그러면 **"현재 터미널"** 에서 도구들을 사용할 수 있습니다.   
   
### 설치 확인   
`source ./emsdk_env.sh` 를 실행해서 환경변수를 설정하였다면,    
```bash
emcc -v
```   
위 명령어를 통해 emcc 가 설치되어 있는지 확인합니다.      
<br/><br/>
# hello world
<https://emscripten.org/docs/getting_started/Tutorial.html>

정리하고 싶은 부분만 정리하였습니다.   
   
일단 저는 환경변수 문제인지 잘 모르겠는데, 제 환경에선 ./emcc 대신 emcc로만 emcc 사용이 가능했습니다. 리눅스 뉴비라 이유는 모르겠습니다.   
      
### 컴파일 및 실행
`emcc <소스코드> -o <파일명>.html `  
위 명령어로 html 파일을 만들 수 있는데, 그냥 실행하면 XHR Request 라는 것을 크롬, 사파리 등에서 지원하지 않아 그런 브라우저에서 실행하면 필요한 파일들이 로드되지 않는다고 합니다.    
따라서 파일들을 웹서버로 제공하는 방법이 제시되어 있습니다.   
`python -m http.server` 예제에서 만든 `hello.html` 이 있는 디렉토리에서 이것을 실행합니다. 그런 다음 `http://localhost:8000/` 로 브라우저를 통해 접속하면 됩니다.
   
위 로컬 웹서버 대신 간단한 로컬 테스팅을 위한 다른 방법으론 컴파일 할 때 `-sSINGLE_FILE` 옵션을 이용하는 것입니다.   
      

### 그래픽스 라이브러리
emscripten 에서는 SDL과 [기타](https://emscripten.org/docs/porting/multimedia_and_graphics/index.html) 멀티미디어 API를 제공합니다. SDL은 별다른 설치나 링킹없이 사용가능 한 것 같습니다.   
   
### 파일 접근   
emscripten 은 파일 시스템을 시뮬레이팅하므로, `fopen`, `fclose` 등의 파일시스템에 접근하는 표준 libc API를 사용 가능합니다.   
하지만 웹 환경은 파일들이 비동기적으로 다운로드 된단 점 때문에 웹개발을 한번이라도 해보셨다면 아실 수 있다시피 `fopen` 이 실행되는 타이밍에 해당 파일이 아직 다운로드 되지 않았을 수 있습니다.   
   
emscripten 에서는 컴파일 시 이를 두가지 방법으로 해결합니다.   
1. `--embed-file <file>`    
이 옵션을 이용하면 생성된 WebAssembly Moduel에 파일들이 삽입된다고 설명되어 있습니다. 아래의 두번째 방법은 실행시간에 파일들이 다시 복사되어지므로 이 방법이 일반적으로 더 효율적이라고 설명되어 있습니다. `<file>` 에는 디렉토리를 포함할 수 있고, 디렉토리를 인자로 주면 디렉토리의 모든 내용이 포함된다고 합니다.
2. `preload-file <file>`    
이 옵션을 사용하면 **filename.html** 이 메인 파일로 만들어진다고 했을 때, **filename.data** 에 preload 된 파일들이 저장된다고 합니다.   

위의 두가지 방법으로 컴파일 시간에 가상 파일 시스템이 원하는 구조로 생성됩니다. (즉 네이티브에서 `fopen` 쓰듯이 접근 가능합니다) 이때, 파일 시스템은 컴파일러를 실행하는 위치를 기반한 상대 경로로 만들어집니다. 예를 들어, `<file>` 에 `dir/something.txt` 를 준다면, `fopen("dir/somthing.txt", ...)` 를 통해 접근해야합니다.   
   
더 자세한 설명은 원문에서 제공하는 링크가 있습니다.   
[파일 시스템 개요](https://emscripten.org/docs/porting/files/file_systems_overview.html#file-system-overview)   
[파일 시스템 API (emscripten 라이브러리)](https://emscripten.org/docs/api_reference/Filesystem-API.html#filesystem-api)   
[Synchronous Virtual XHR Backed File System Usage](https://emscripten.org/docs/porting/files/Synchronous-Virtual-XHR-Backed-File-System-Usage.html#synchronous-virtual-xhr-backed-file-system-usage)   


### 원문의 다른 주제   
테스트 코드 작성을 통한 테스팅을 위한 툴 `test/runner`   
테스팅 툴 + 벤치마킹 기능도 제공합니다.   
   
<https://emscripten.org/docs/api_reference/emscripten.h.html#emscripten-h>   
emscripten 이 제공하는 (emscripten.h) 라이브러리 레퍼런스         

https://emscripten.org/docs/porting/index.html#integrating-porting-index   
기존 C/C++ 코드 이식 관련