# Quantum ESPRESSO 다운로드 및 설치

Ubuntu에서 Quantum ESPRESSO의 소스 코드를 다운로드하고, 컴파일한 뒤 실행 여부를 확인합니다.

- 사전 준비: Ubuntu 및 컴파일 도구·계산 라이브러리 설치 완료
- 인터넷 연결 필요
- **이 문서의 모든 명령어는 Ubuntu 터미널에서 실행합니다.**
- 예시 버전: Quantum ESPRESSO 7.5
- 설치 방식: CPU를 사용하는 소스 코드 컴파일

> **소스 코드**는 사람이 작성한 프로그램 원본이며, **컴파일**은 이 코드를 컴퓨터가 실행할 수 있는 프로그램으로 변환하는 과정입니다.

## 1. 추가 도구 설치

```bash
sudo apt update
sudo apt install git ca-certificates libblas-dev openmpi-bin
```

소스 코드 다운로드와 컴파일·병렬 실행에 사용할 도구를 보완합니다.

| 패키지 | 역할 |
|---|---|
| `git` | 저장소에서 소스 코드 다운로드 |
| `ca-certificates` | HTTPS 연결에 사용하는 인증서 |
| `libblas-dev` | BLAS 라이브러리를 프로그램과 연결하는 데 필요한 개발 파일 |
| `openmpi-bin` | `mpirun` 등 MPI 실행 명령어 제공 |

> 앞서 설치한 `libblas3`는 실행 라이브러리입니다. 여기서는 컴파일에 필요한 `libblas-dev`를 추가합니다. 이미 설치된 패키지는 다시 설치할 필요 없이 설치 상태가 확인됩니다.

## 2. 설치할 폴더 만들기

```bash
mkdir -p ~/software
cd ~/software
```

Ubuntu 홈 디렉터리에 `software` 폴더를 만들고 이동합니다.

| 명령어 | 의미 |
|---|---|
| `mkdir` | 폴더 생성 |
| `-p` | 필요한 상위 폴더도 함께 생성하며, 이미 있는 폴더는 그대로 사용 |
| `~` | 현재 Ubuntu 사용자의 홈 디렉터리 |
| `cd` | 지정한 폴더로 이동 |

현재 위치를 확인합니다.

```bash
pwd
```

출력 예시:

```text
/home/사용자이름/software
```

> 이 안내에서는 Ubuntu 홈 디렉터리 아래에 설치합니다. 이후 명령어의 경로도 이 위치를 기준으로 합니다.

## 3. Quantum ESPRESSO 소스 코드 다운로드

```bash
git clone --branch qe-7.5 --depth 1 --recurse-submodules --shallow-submodules https://gitlab.com/QEF/q-e.git qe-7.5
```

Quantum ESPRESSO 공식 저장소에서 7.5 버전의 소스 코드와 관련 하위 저장소를 다운로드합니다.

| 항목 | 의미 |
|---|---|
| `git clone` | 저장소를 내 컴퓨터에 복사 |
| `--branch qe-7.5` | `qe-7.5` 태그에 해당하는 버전 선택 |
| `--depth 1` | 전체 개발 이력 대신 선택한 버전에 필요한 이력만 다운로드 |
| `--recurse-submodules` | 연결된 하위 저장소도 함께 다운로드 |
| `--shallow-submodules` | 하위 저장소의 다운로드 이력도 최소화 |
| 마지막 `qe-7.5` | 내 컴퓨터에 생성할 폴더 이름 |

> 다운로드 중 `detached HEAD` 안내가 표시될 수 있습니다. 특정 릴리스 태그를 선택했기 때문이며, 설치를 진행하는 데 문제가 되지 않습니다.

다운로드가 완료되면 해당 폴더로 이동합니다.

```bash
cd ~/software/qe-7.5
```

파일과 폴더를 확인합니다.

```bash
ls
```

`configure`, `PW`, `Modules`, `Makefile` 등의 항목이 보이면 소스 코드가 다운로드된 상태입니다.

> `destination path ... already exists` 오류가 나오면 같은 이름의 폴더가 이미 있다는 뜻입니다. 기존 다운로드가 완료된 상태라면 해당 폴더로 이동해서 다음 단계를 진행합니다.

## 4. 컴파일러 및 MPI 확인

```bash
gfortran --version
```

Fortran 컴파일러의 버전을 확인합니다.

```bash
mpif90 --version
```

MPI용 Fortran 컴파일 명령어가 작동하는지 확인합니다.

```bash
mpirun --version
```

MPI 실행 도구의 버전을 확인합니다.

| 명령어 | 역할 |
|---|---|
| `gfortran` | Fortran 코드 컴파일 |
| `mpif90` | MPI 프로그램 컴파일에 필요한 옵션과 라이브러리를 연결 |
| `mpirun` | 여러 MPI 프로세스로 프로그램 실행 |

> `mpif90 --version`에서 GNU Fortran 정보가 표시되는 것은 정상입니다. `mpif90`이 내부적으로 `gfortran`을 사용할 수 있기 때문입니다.

## 5. 컴파일 환경 설정

Quantum ESPRESSO 소스 코드 폴더에서 다음 명령어를 실행합니다.

```bash
cd ~/software/qe-7.5
./configure
```

`configure`는 컴퓨터에 설치된 컴파일러와 계산 라이브러리를 확인하고, 컴파일에 사용할 설정 파일을 생성합니다.

| 항목 | 의미 |
|---|---|
| `.` | 현재 폴더 |
| `./configure` | 현재 폴더에 있는 `configure` 스크립트 실행 |

이 과정에서는 다음과 같은 사항을 확인합니다.

- C 및 Fortran 컴파일러
- MPI 병렬 계산 환경
- BLAS 및 LAPACK
- FFT 라이브러리
- 기타 컴파일에 필요한 기능

> 이 단계는 **컴파일 준비 단계**입니다. 아직 Quantum ESPRESSO 실행 파일이 만들어지는 것은 아닙니다.

### MPI 병렬 설정 확인

설정 결과를 확인합니다.

```bash
grep -E '^[[:space:]]*(MPIF90|DFLAGS)[[:space:]]*=' make.inc
```

MPI가 활성화된 경우 일반적으로 다음 항목을 확인할 수 있습니다.

- `MPIF90`에 `mpif90` 또는 MPI 컴파일러 경로가 지정됨
- `DFLAGS`에 `-D__MPI`가 포함됨

실제 출력 형식과 다른 옵션은 환경에 따라 달라질 수 있습니다.

| 명령어·파일 | 의미 |
|---|---|
| `grep` | 파일에서 특정 조건에 맞는 줄 검색 |
| `make.inc` | 컴파일러·옵션·라이브러리 연결 설정을 담은 파일 |
| `MPIF90` | 컴파일에 사용할 MPI Fortran 명령어 |
| `-D__MPI` | MPI 기능을 활성화하는 전처리 옵션 |

> ScaLAPACK과 ELPA는 설치되어 있어도 자동으로 모두 활성화되는 것은 아닙니다. 이 안내에서는 기본 설정으로 설치를 진행하며, 해당 라이브러리의 추가 최적화는 별도로 다룹니다.

## 6. Quantum ESPRESSO 컴파일

### 기본 계산 프로그램 만들기

```bash
make -j2 pw
```

전자구조 계산에 사용하는 `pw.x`와 필요한 구성 요소를 컴파일합니다.

| 항목 | 의미 |
|---|---|
| `make` | 설정과 의존관계에 따라 컴파일 수행 |
| `-j2` | 최대 2개의 컴파일 작업을 동시에 수행 |
| `pw` | PWscf 계산 프로그램을 만드는 대상 지정 |

처음 설치할 때는 비교적 부담이 적은 `-j2`로 시작합니다.

> `-j2`는 **프로그램을 만드는 과정의 병렬 작업 수**입니다. 나중에 계산을 실행할 때 사용하는 MPI 프로세스 수와는 별개입니다.

컴파일 중에는 많은 메시지가 출력됩니다. 컴퓨터 성능과 인터넷 연결 상태에 따라 시간이 걸릴 수 있습니다. 필요한 외부 구성 요소를 추가로 다운로드하는 경우도 있습니다.

### 다른 프로그램도 설치하려면

기본 컴파일이 성공한 뒤, 필요한 프로그램을 추가로 컴파일할 수 있습니다.

```bash
make -j2 ph
```

포논 계산 프로그램을 컴파일합니다.

```bash
make -j2 pp
```

계산 결과의 후처리에 사용하는 프로그램을 컴파일합니다.

전체 기본 빌드 대상을 컴파일하려면 다음 명령어를 사용합니다.

```bash
make -j2 all
```

| 명령어 | 주요 용도 |
|---|---|
| `make -j2 pw` | 전자구조 계산 |
| `make -j2 ph` | 포논 계산 |
| `make -j2 pp` | 후처리 |
| `make -j2 all` | 전체 기본 빌드 대상 컴파일 |

> 처음에는 `pw`만 컴파일해도 기본적인 SCF 계산을 시작할 수 있습니다. 컴파일에는 `sudo`를 붙이지 않습니다.

## 7. 실행 파일 생성 확인

```bash
ls -l ~/software/qe-7.5/bin/pw.x
```

`pw.x` 파일 또는 해당 실행 파일을 가리키는 링크가 표시되는지 확인합니다.

실행 가능한 상태인지 확인합니다.

```bash
test -x ~/software/qe-7.5/bin/pw.x && echo "pw.x is ready"
```

정상 출력:

```text
pw.x is ready
```

| 항목 | 의미 |
|---|---|
| `test -x` | 지정한 파일을 실행할 수 있는지 확인 |
| `&&` | 앞 명령어가 성공하면 뒤 명령어 실행 |
| `echo` | 화면에 문구 출력 |

> 이 명령어가 아무것도 출력하지 않으면 실행 파일이 없거나 실행할 수 없는 상태입니다. 컴파일 오류와 경로를 확인합니다.

## 8. 실행 경로 등록

현재 상태에서는 `pw.x`를 실행할 때 전체 경로를 입력해야 합니다.

```bash
~/software/qe-7.5/bin/pw.x -h
```

어느 폴더에서든 `pw.x`만 입력해 실행할 수 있도록 `PATH`에 등록합니다.

### 설정 파일에 경로 추가

다음 명령어는 **한 번만 실행합니다.**

```bash
echo 'export PATH="$HOME/software/qe-7.5/bin:$PATH"' >> ~/.bashrc
```

| 항목 | 의미 |
|---|---|
| `export` | 환경변수를 현재 셸과 이후 실행하는 프로그램에 전달 |
| `PATH` | 명령어의 실행 파일을 검색하는 폴더 목록 |
| `$HOME` | 현재 Ubuntu 사용자의 홈 디렉터리 |
| `$PATH` | 기존 실행 경로 목록 |
| `>>` | 파일 끝에 내용 추가 |
| `~/.bashrc` | Ubuntu의 대화형 Bash 터미널이 시작될 때 읽는 설정 파일 |

> 작은따옴표 안의 내용을 그대로 입력합니다. 위 명령어는 기존 `PATH`를 유지하면서 Quantum ESPRESSO의 `bin` 폴더를 앞에 추가합니다.

### 현재 터미널에 적용

```bash
source ~/.bashrc
```

변경한 설정을 현재 터미널에 적용합니다.

### 등록 결과 확인

```bash
command -v pw.x
```

출력 예시:

```text
/home/사용자이름/software/qe-7.5/bin/pw.x
```

다른 경로가 표시되면 기존에 설치한 다른 버전이 선택되어 있을 수 있으므로 경로를 확인합니다.

> 앞으로 설치 폴더를 이동하거나 삭제하면 등록한 경로도 수정해야 합니다.

## 9. 프로그램 실행 확인

### 기본 실행 확인

```bash
pw.x -h
```

프로그램의 시작 정보와 도움말이 표시되는지 확인합니다.

출력에는 다음과 같은 정보가 포함될 수 있습니다.

```text
Program PWSCF ...
```

버전과 출력 형식은 설치한 버전에 따라 달라질 수 있습니다.

### MPI 병렬 실행 확인

```bash
OMP_NUM_THREADS=1 mpirun -np 2 pw.x -h
```

MPI 프로세스 2개로 실행을 시작할 수 있는지 확인합니다.

| 항목 | 의미 |
|---|---|
| `OMP_NUM_THREADS=1` | 이 실행에서 프로세스당 OpenMP 스레드 수를 1로 지정 |
| `mpirun` | MPI 프로그램 실행 |
| `-np 2` | MPI 프로세스 2개 사용 |
| `pw.x` | 전자구조 계산 프로그램 |
| `-h` | 도움말 표시 |

시작 정보에서 프로세스 수가 2로 표시되는지 확인합니다.

> 도움말 실행은 프로그램이 시작되는지 확인하는 간단한 점검입니다. **실제 계산의 정상 수행 여부는 입력 파일과 의사퍼텐셜을 준비한 뒤 별도로 확인해야 합니다.**

## 10. 자주 발생하는 문제

### `git: command not found`

Git이 설치되어 있지 않은 상태입니다.

```bash
sudo apt install git
```

### 하위 저장소 다운로드가 중단된 경우

소스 코드 폴더에서 다시 다운로드합니다.

```bash
cd ~/software/qe-7.5
git submodule update --init --recursive
```

### `mpif90: command not found`

MPI 개발 패키지를 확인합니다.

```bash
sudo apt install libopenmpi-dev openmpi-bin
```

설치 후 다시 확인합니다.

```bash
mpif90 --version
```

### MPI가 감지되지 않은 경우

MPI 명령어가 정상 작동하는지 먼저 확인합니다. 이후 소스 코드 폴더에서 다음과 같이 설정할 수 있습니다.

```bash
cd ~/software/qe-7.5
./configure MPIF90=mpif90
```

이미 컴파일을 진행한 상태에서 MPI 설정을 변경했다면 기존 컴파일 결과를 정리한 뒤 다시 진행합니다.

```bash
make clean
./configure MPIF90=mpif90
make -j2 pw
```

> OpenMPI가 사용할 Fortran 컴파일러를 명시해야 하는 환경에서는 `export OMPI_FC=gfortran`을 설정한 뒤 다시 진행합니다. 기본 설정이 정상 작동하면 추가할 필요가 없습니다.

### 컴파일 중 메모리 부족이 의심되는 경우

`Killed` 등의 메시지가 나오면 메모리 부족일 수 있습니다. 동시에 수행하는 작업 수를 줄여 재시도합니다.

```bash
make -j1 pw
```

### `pw.x: command not found`

실행 파일이 존재하는지 확인합니다.

```bash
ls -l ~/software/qe-7.5/bin/pw.x
```

파일이 있다면 경로 설정을 적용합니다.

```bash
source ~/.bashrc
command -v pw.x
```

### MPI에서 실행 슬롯이 부족하다고 나오는 경우

Ubuntu에서 사용할 수 있는 CPU 수를 확인합니다.

```bash
nproc
```

사용 가능한 자원에 맞게 프로세스 수를 줄입니다.

```bash
OMP_NUM_THREADS=1 mpirun -np 1 pw.x -h
```

> MPI 실행은 일반 Ubuntu 사용자 계정으로 진행합니다. `sudo mpirun`을 사용하지 않습니다.

### 컴파일이 실패한 경우

화면 마지막의 `Error 1`, `Error 2`만으로는 원인을 알기 어렵습니다. 그보다 앞에 표시된 실제 오류 메시지를 확인합니다.

원인을 찾기 어렵다면 작업 하나로 재실행하여 출력을 확인합니다.

```bash
make -j1 pw > build.log 2>&1
```

최근 출력 내용을 확인합니다.

```bash
tail -n 80 build.log
```

| 항목 | 의미 |
|---|---|
| `> build.log` | 일반 출력을 파일에 저장하며, 기존 파일은 덮어씀 |
| `2>&1` | 오류 출력도 같은 파일에 저장 |
| `tail -n 80` | 파일의 마지막 80줄 확인 |

## 설치 완료

여기까지 진행하면 다음 상태가 됩니다.

- Quantum ESPRESSO 소스 코드 다운로드 완료
- 컴파일 환경 설정 완료
- `pw.x` 컴파일 완료
- 실행 경로 등록 완료
- 기본 실행 및 MPI 실행 시작 확인 완료

**다음 단계에서는 계산용 폴더, 의사퍼텐셜, 입력 파일을 준비하고 첫 번째 SCF 계산을 수행합니다.**

## 참고 자료

- [Quantum ESPRESSO 공식 홈페이지](https://www.quantum-espresso.org/)
- [Quantum ESPRESSO 공식 소스 저장소](https://gitlab.com/QEF/q-e)
- [Quantum ESPRESSO 7.5 릴리스](https://gitlab.com/QEF/q-e/-/releases/qe-7.5)
- [공식 설치 안내 — make를 이용한 컴파일](https://www.quantum-espresso.org/Doc/user_guide/node11.html)
- [공식 PWscf 사용 안내](https://www.quantum-espresso.org/Doc/pw_user_guide/node7.html)
