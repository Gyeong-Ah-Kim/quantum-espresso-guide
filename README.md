# Ubuntu 설치 및 Quantum ESPRESSO 실행 환경 준비

Windows에서 WSL 2를 이용해 Ubuntu를 설치하고, Quantum ESPRESSO 컴파일에 필요한 도구를 준비합니다.

- 대상: Windows 11 또는 Windows 10 버전 2004(빌드 19041) 이상
- 인터넷 연결 필요
- 명령어의 실행 위치가 **PowerShell**인지 **Ubuntu 터미널**인지 확인하세요.

## 1. WSL 및 Ubuntu 설치

시작 메뉴에서 **PowerShell**을 검색한 후, 마우스 오른쪽 버튼을 눌러 **관리자 권한으로 실행**합니다.

```powershell
wsl --install -d Ubuntu
```

Windows에서 Linux를 사용할 수 있도록 WSL과 Ubuntu를 설치합니다.

| 명령어 | 의미 |
|---|---|
| `wsl` | Windows의 Linux 환경 관리 |
| `--install` | WSL 설치 |
| `-d Ubuntu` | 설치할 Linux 배포판을 Ubuntu로 지정 |

설치가 끝나면 **컴퓨터를 재시작**합니다.

## 2. Ubuntu 계정 생성

시작 메뉴에서 **Ubuntu**를 실행합니다.

최초 실행 시 사용자 이름과 비밀번호를 설정합니다.

```text
Enter new UNIX username: 사용자 이름 입력
New password: 비밀번호 입력
Retype new password: 비밀번호 재입력
```

- Windows 계정과 별도로 사용하는 Ubuntu 계정입니다.
- 비밀번호 입력 시 화면에 글자나 별표가 표시되지 않는 것은 정상입니다.
- 이 비밀번호는 이후 관리자 권한으로 명령어를 실행할 때 사용합니다.

## 3. WSL 2 설치 확인

**실행 위치: PowerShell**

```powershell
wsl --list --verbose
```

설치된 Linux 배포판과 WSL 버전을 확인합니다.

```text
  NAME      STATE           VERSION
* Ubuntu    Stopped         2
```

`VERSION`이 **2**이면 WSL 2를 사용하는 상태입니다. `STATE`는 실행 여부에 따라 달라집니다.

만약 `VERSION`이 1이면 다음 명령어로 변경합니다.

```powershell
wsl --set-version Ubuntu 2
```

> 배포판 이름이 `Ubuntu-24.04`처럼 표시되면 `Ubuntu` 대신 해당 이름을 입력합니다.

## 4. Ubuntu 업데이트

이 단계부터는 **Ubuntu 터미널**에서 실행합니다.

### 소프트웨어 목록 갱신

```bash
sudo apt update
```

설치 가능한 소프트웨어의 목록과 버전 정보를 최신 상태로 갱신합니다.

### 설치된 소프트웨어 업그레이드

```bash
sudo apt upgrade
```

현재 설치된 소프트웨어를 업그레이드합니다. 진행 여부를 물으면 `Y`를 입력하고 Enter를 누릅니다.

| 명령어 | 의미 |
|---|---|
| `sudo` | 관리자 권한으로 실행 |
| `apt` | Ubuntu의 소프트웨어 관리 도구 |
| `update` | 소프트웨어 목록 갱신 |
| `upgrade` | 설치된 소프트웨어 업그레이드 |

> 비밀번호를 요청하면 2단계에서 설정한 Ubuntu 비밀번호를 입력합니다.

## 5. 기본 컴파일 도구 및 계산 라이브러리 설치

**실행 위치: Ubuntu 터미널**

```bash
sudo apt install autoconf build-essential gfortran libblas3 libc6 libfftw3-dev liblapack-dev
```

Quantum ESPRESSO의 소스 코드를 실행 가능한 프로그램으로 만들고, 수학 계산을 수행하는 데 필요한 도구를 설치합니다.

| 패키지 | 역할 |
|---|---|
| `autoconf` | 설치 환경을 확인하는 설정 스크립트 생성 |
| `build-essential` | C/C++ 컴파일러와 `make` 등 기본 빌드 도구 |
| `gfortran` | Fortran 소스 코드를 실행 프로그램으로 변환 |
| `libblas3` | 벡터·행렬 계산용 실행 라이브러리 |
| `libc6` | 프로그램 실행에 필요한 기본 C 라이브러리 |
| `libfftw3-dev` | 푸리에 변환 계산용 개발 라이브러리 |
| `liblapack-dev` | 방정식·행렬 계산용 개발 라이브러리 |

> `apt install`은 뒤에 나열된 패키지를 설치한다는 뜻입니다.

## 6. 병렬 계산 라이브러리 설치

**실행 위치: Ubuntu 터미널**

```bash
sudo apt install libopenmpi-dev libscalapack-openmpi-dev libelpa-dev
```

여러 프로세스가 계산을 나누어 처리하고, 큰 행렬을 효율적으로 계산하는 데 필요한 라이브러리를 설치합니다.

| 패키지 | 역할 |
|---|---|
| `libopenmpi-dev` | 병렬 계산 중 프로세스 간 통신 지원 |
| `libscalapack-openmpi-dev` | 큰 행렬 계산을 여러 프로세스에 분산 |
| `libelpa-dev` | 행렬의 고유값·고유벡터 계산 가속 |

> 라이브러리를 설치하는 것만으로 병렬 계산이 자동 적용되지는 않습니다. 이후 Quantum ESPRESSO의 컴파일 설정과 실행 방법에 따라 사용됩니다.

## 7. 설치 결과 확인

**실행 위치: Ubuntu 터미널**

```bash
cat /etc/os-release
```

Ubuntu 이름과 버전을 확인합니다.

```bash
gfortran --version
```

Fortran 컴파일러의 설치 여부와 버전을 확인합니다.

```bash
make --version
```

컴파일 작업을 관리하는 `make`의 설치 여부와 버전을 확인합니다.

## 8. Ubuntu 실행 및 종료

### 실행 — PowerShell에서 입력

```powershell
wsl -d Ubuntu
```

Ubuntu 터미널을 실행합니다. 시작 메뉴에서 Ubuntu를 실행해도 됩니다.

### 터미널 종료 — Ubuntu에서 입력

```bash
exit
```

현재 Ubuntu 터미널 세션을 종료합니다.

## 준비 완료

여기까지 진행하면 Ubuntu와 Quantum ESPRESSO 컴파일에 필요한 기본 환경이 준비됩니다.

**Quantum ESPRESSO 자체는 아직 설치되지 않았습니다.** 다음 단계에서 소스 코드를 다운로드하고, 컴파일 환경 설정 및 컴파일을 진행합니다.

## 참고 자료

- [Microsoft WSL 설치 안내](https://learn.microsoft.com/ko-kr/windows/wsl/install)
- [Microsoft WSL 기본 명령어](https://learn.microsoft.com/ko-kr/windows/wsl/basic-commands)
- [Quantum ESPRESSO 공식 홈페이지](https://www.quantum-espresso.org/)
