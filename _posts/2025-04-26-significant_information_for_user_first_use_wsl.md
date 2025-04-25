---
title: WSL 을 처음 사용하려고 할 때 몰랐어서 후회한 것들
date: 2025-04-26 02:00:00 +0900
categories: [개발, 기타]
tags: [wsl, 개발환경] # TAG names should always be lowercase
---

이 글은 제가 `WSL (Windows Subsystem for Linux)` 을 사용하면서, 처음 사용하려는 사용자가 알면 좋을 내용들을 정리하였습니다. ~~근데 사실 제목은 어그로입니다~~  
이 글에선 WSL2 를 기준으로 설명합니다. 글에서 `호스트` 라고 적은 내용들은 WSL 가 설치된 `윈도우` 운영체제를 뜻합니다. `윈도우` 라고 적기도 하였습니다.

# WSL(WSL2)

Hyper-V 를 이용하여 윈도우에서 듀얼 부팅없이 리눅스 환경을 사용할 수 있는 기능입니다. 실제 사용할 때 네이티브 리눅스와의 차이점을 느끼기 어렵습니다. 가상머신보다 세팅이 훨씬 간편하며, 듀얼 부팅과 비교하면 윈도우와 리눅스를 동시에 사용할 수 없다는 단점이 없습니다.

아래는 첫 줄에서 말한 `WSL` 을 처음 사용하려는 사용자가 알면 좋을 내용들을 열거하였습니다.

## Windows 10 과 11 의 차이?

11에서는 사용가능한데, 10에서는 사용할 수 없는 아주 다양한 기능들이 존재합니다. WSL 을 어느 정도로 사용하느냐에 따라 중요할 수 있을 것 같습니다. 하지만 제 개발환경에서는 윈도우 10을 사용하고 있어도 문제가 있던 적은 없습니다. 어차피 곧 윈도우 10 공식 지원 종료된다고 하니까 사소한 문제일 수는 있겠네요.

## 네이티브 리눅스와 WSL2 의 차이?

- 임베디드 프로그래밍을 할 때 외부 장치를 인식시키는 것이 어렵다고 들은 적이 있는 듯 합니다. 제가 경험한 내용은 아니라 잘 모릅니다.
- `systemd` 를 사용할 수 없다는 오해가 있습니다. 사실 조금 이전에는 사실이었으나 현재 WSL 의 `Ubuntu` 배포판에서는 기본으로 제공되며, 이외의 경우에서도 활성화 할 수 있다고 [공식 문서](https://learn.microsoft.com/ko-kr/windows/wsl/systemd)에 나와있습니다.
- 리눅스 커널: WSL 의 리눅스 배포판들은 자체적인 커스텀 커널을 가지고 있는 것으로 보입니다. 따라서 네이티브 리눅스보다 커널 선택이 제한됩니다. 따라서 특정 커널 버전에 관련된 기능이 있다면 사용하지 못할 수도 있습니다. [참고 자료](https://learn.microsoft.com/ko-kr/community/content/wsl-user-msft-kernel-v6)  
  특히 참고 자료에 따르면 여러 배포판에서 서로 다른 커널을 사용하도록 하는 것이 불가능하다고 나와있습니다.
- 이외에 여러 차이점이 있을 수도 있지만, 특별히 리눅스에 specific 한 작업이 아니라면 문제 없다고 생각합니다.

## WSL 에 할당된 자원

- 참고: <https://learn.microsoft.com/ko-kr/windows/wsl/wsl-config#main-wsl-settings>
- CPU 는 호스트 머신과 동일한 수의 코어, 스레드를 사용가능 합니다.
- 메모리는 호스트 머신의 전체 메모리의 50% 를 디폴트로 사용합니다. 이는 `.wslconfig` 파일을 통해 늘리거나 줄일 수 있습니다.
- [스왑](https://en.wikipedia.org/wiki/Memory_paging)에 사용할 수 있는 메모리는 호스트의 메모리의 25%가 디폴트이고 `.wslconfig` 에서 조절할 수 있습니다.
- WSL 은 호스트 머신의 GPU 를 인식할 수 있고, 사용할 수 있습니다. 다만 WSL 안에서 그래픽 드라이버를 설치할 필요가 있습니다.
- 보조 디스크는 디폴트로 최대 1TB 를 사용할 수 있고, `.wslconfig` 에서 조절 가능합니다.

### 메모리에 관해서

- WSL 은 한 번 사용한 메모리를 쉽게 반환하지 않는 문제가 있습니다. https://github.com/microsoft/WSL/issues/4166
- 8~16 GB 메모리를 사용하는 환경에서 큰 문제가 될 수도 있는데, 필요하면 WSL 에서 `echo 3 > /proc/sys/vm/drop_caches` 명령을 통해 메모리를 회수할 수 있습니다. (<https://github.com/microsoft/WSL/issues/4166#issuecomment-2327161215>)
- `.wslconfig` 설정에 대한 [문서](https://learn.microsoft.com/ko-kr/windows/wsl/wsl-config#experimental-settings)에 따르면, 실험적 기능이지만 `autoMemoryReclaim` 옵션을 통해 메모리를 자동으로 회수할 수 있다고 합니다. 다만 해당 설정으로 인해 WSL 에서 **`docker`** 를 사용할 때 문제가 생길 수도? 안생길 수도? 있다는 것 같습니다. <https://github.com/ddev/ddev/issues/5356>

## 네트워크

- 제가 WSL 와 호스트 윈도우 사이의 네트워크 구조에 대한 지식이 부족해서, 아래 내용은 제 경험 사례 내에서 이야기 한다고 생각해주세요.
- `WSL2` 는 호스트와 별도의 네트워크 인터페이스를 가지고 있습니다. 따라서 보통은 윈도우와 외부 연결을 WSL 에서 캡쳐할 수 없는 것 같고, 그 반대도 마찬가지로 보입니다.
- 윈도우에서 `WSL` 에 접속할 때에는 `localhost` 를 통해서 접근할 수 있습니다.
- `WSL` 에서 윈도우에 접속할 때에는 `localhost` 대신 IP 주소를 통해서 접근해야 합니다.
- `WSL` 의 네트워크 인터페이스 또한 자신의 IP 주소를 가지고 있습니다.
- (Windows 11) 호스트의 네트워크 인터페이스를 [`미러링`](https://learn.microsoft.com/ko-kr/windows/wsl/networking#mirrored-mode-networking) 할 수 있습니다. 특히 이 경우에 `localhost` 를 통해 `WSL` 에서 윈도우에 접근할 수 있다고 합니다.

## 리눅스 배포판 선택

- `Ubuntu 22.04` 만 지원하는 것이 아닙니다. 일단 WSL 이 설치되어 있다면 명령 프롬프트(cmd) 에서 `wsl -l -o` 을 입력하여 볼 수 있으며, 공식적으로 관리되진 않을 수 있어도 다양한 배포판이 인터넷에 있을 수도? 있습니다.
- 참고로 명령 프롬프트에서 설치가능한 배포판 중 일부만 나열하면, `Debian` `FedoraLinux-42` `SUSE-Linux-Enterprise-15-SP6` `Ubuntu-24.04` `archlinux` `OracleLinux_9_1` 등이 있습니다. (`wsl --install -d <설치할 배포판>` 명령 에서 입력 가능한 이름으로 적었습니다)
- 배포판을 지우는 것은 쉽습니다. `wsl --unregister <배포판>`
- 명령 프롬프트에서 `wsl` 명령으로 접속할 수 있는 디폴트 배포판을 설정할 수 있습니다. `wsl --set-default, -s <배포판>`

## WSL 배포판이 설치될 위치

- 명령 프롬프트(cmd) 에서 WSL 을 설치할 경우에, `wsl --install -d <설치할 배포판> --location <Location>` 을 통해 WSL 배포판이 설치될 위치를 지정할 수 있습니다. 정확히는 리눅스 파일 시스템이 저장된 가상 디스크(VHD) 파일의 위치입니다.
- 이미 설치된 배포판의 VHD 또한 쉽게 옮길 수 있습니다. `wsl --manage <배포판> --move <Location>` 을 통해 가능합니다.

## WSL 에서 윈도우 파일 시스템에 접근하기

- WSL 에서 `/mnt` 디렉토리에 `c` `d` `e` 와 같이 존재합니다. 아마 다른 디스크를 mount 하거나 unmount 하는 것도 가능한 것 같은데, 관련 작업을 해본 적이 없어서 잘 모릅니다.

## 이후에 다른 내용이 생각나면 글 업데이트 하겠습니다.
