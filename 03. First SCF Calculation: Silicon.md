# 첫 번째 SCF 계산 — 실리콘(Si) 결정

Quantum ESPRESSO를 이용해 실리콘 결정의 전자밀도와 총에너지를 계산합니다.

- 사전 준비: Quantum ESPRESSO 설치 및 `pw.x` 실행 경로 등록 완료
- 계산 대상: 다이아몬드 구조의 실리콘 결정
- 계산 셀: Si 원자 2개를 포함한 원시 단위셀
- 계산 방법: DFT / PBE / Ultrasoft 의사퍼텐셜
- **모든 명령어는 Ubuntu 터미널에서 실행합니다.**

> 이 예제는 첫 계산을 익히기 위한 실습입니다. 연구에 사용할 결과는 에너지 컷오프와 k-point에 대한 수렴성 검증이 필요합니다.

## 1. SCF 계산이란?

**SCF(Self-Consistent Field)**는 전자밀도가 자기일관성을 만족할 때까지 계산을 반복하는 과정입니다.

1. 초기 전자밀도를 설정합니다.
2. 전자밀도로부터 유효 퍼텐셜을 만듭니다.
3. 전자 상태를 계산하고 새로운 전자밀도를 얻습니다.
4. 전자밀도를 갱신하며 수렴할 때까지 반복합니다.

이 예제에서는 **원자 위치와 격자를 고정한 상태에서 전자구조를 계산**합니다. 구조 최적화는 수행하지 않습니다.

## 2. 실행 명령어 확인

```bash
command -v pw.x
```

출력 예시:

```text
/home/사용자이름/software/qe-7.5/bin/pw.x
```

경로가 표시되지 않으면 Quantum ESPRESSO 설치 및 PATH 설정을 먼저 확인합니다.

## 3. 계산용 폴더 만들기

```bash
mkdir -p ~/qe-examples/01-si-scf/pseudo
mkdir -p ~/qe-examples/01-si-scf/tmp
cd ~/qe-examples/01-si-scf
```

계산에 사용할 폴더를 생성하고 이동합니다.

| 폴더 | 용도 |
|---|---|
| `01-si-scf` | 이번 실습의 작업 폴더 |
| `pseudo` | 의사퍼텐셜 파일 보관 |
| `tmp` | 전자밀도·파동함수 등 계산 데이터 저장 |

현재 위치를 확인합니다.

```bash
pwd
```

출력 예시:

```text
/home/사용자이름/qe-examples/01-si-scf
```

> 이후 명령어는 이 작업 폴더에서 실행합니다. 입력 파일의 `./pseudo`와 `./tmp`는 명령어를 실행하는 현재 폴더를 기준으로 해석됩니다.

## 4. 실리콘 의사퍼텐셜 다운로드

의사퍼텐셜은 원자핵과 내부 전자가 원자가전자에 미치는 효과를 계산하기 쉽게 나타낸 데이터입니다.

이번 실습에서는 공식 사이트에서 제공하는 다음 파일을 사용합니다.

```text
Si.pbe-n-rrkjus_psl.1.0.0.UPF
```

### 다운로드 도구 설치

```bash
sudo apt install wget
```

### 의사퍼텐셜 다운로드

```bash
wget -O pseudo/Si.pbe-n-rrkjus_psl.1.0.0.UPF \
  https://pseudopotentials.quantum-espresso.org/upf_files/Si.pbe-n-rrkjus_psl.1.0.0.UPF
```

| 항목 | 의미 |
|---|---|
| `wget` | URL에서 파일 다운로드 |
| `-O` | 저장할 파일 경로와 이름 지정 |
| `\` | 긴 명령어를 다음 줄까지 이어서 작성 |

> 두 줄을 함께 복사해서 실행합니다. `\` 뒤에 공백을 추가하지 않습니다.

### 다운로드 확인

```bash
ls -lh pseudo/Si.pbe-n-rrkjus_psl.1.0.0.UPF
```

파일 앞부분을 확인합니다.

```bash
head -n 15 pseudo/Si.pbe-n-rrkjus_psl.1.0.0.UPF
```

다음과 같은 정보를 확인할 수 있습니다.

```text
<UPF version="2.0.1">
```

- 원소: Si
- 의사퍼텐셜 종류: USPP
- 교환·상관 함수: PBE

> 다운로드에 실패했거나 HTML 오류 페이지가 저장된 경우에는 계산을 진행하지 말고 파일을 다시 다운로드합니다.

## 5. SCF 입력 파일 만들기

다음 코드 블록 전체를 Ubuntu 터미널에 붙여 넣습니다.

```bash
cat > si.scf.in <<'EOF'
&CONTROL
  calculation = 'scf',
  restart_mode = 'from_scratch',
  prefix = 'si',
  pseudo_dir = './pseudo',
  outdir = './tmp',
/
&SYSTEM
  ibrav = 2,
  A = 5.43,
  nat = 2,
  ntyp = 1,
  ecutwfc = 50.0,
  ecutrho = 400.0,
  occupations = 'fixed',
/
&ELECTRONS
  conv_thr = 1.0d-8,
  mixing_beta = 0.7,
/
ATOMIC_SPECIES
Si 28.0855 Si.pbe-n-rrkjus_psl.1.0.0.UPF

ATOMIC_POSITIONS crystal
Si 0.00 0.00 0.00
Si 0.25 0.25 0.25

K_POINTS automatic
4 4 4 0 0 0
EOF
```

| 항목 | 의미 |
|---|---|
| `cat > si.scf.in` | 입력한 내용을 `si.scf.in` 파일에 저장 |
| `<<'EOF'` | 다음 줄부터 종료 표시인 `EOF`까지를 파일 내용으로 사용 |
| 마지막 `EOF` | 파일에 저장할 내용의 끝 |

> 같은 이름의 파일이 있으면 덮어씁니다. 마지막 `EOF`는 줄 맨 앞에 입력합니다.

생성한 파일을 확인합니다.

```bash
cat si.scf.in
```

## 6. 입력 파일 이해하기

### `&CONTROL` — 계산 종류와 저장 위치

| 설정 | 의미 |
|---|---|
| `calculation = 'scf'` | SCF 계산 수행 |
| `restart_mode = 'from_scratch'` | 새로운 계산 시작 |
| `prefix = 'si'` | 저장 데이터의 이름 구분 |
| `pseudo_dir = './pseudo'` | 의사퍼텐셜이 있는 폴더 |
| `outdir = './tmp'` | 계산 데이터 저장 폴더 |

### `&SYSTEM` — 결정 구조와 계산 조건

| 설정 | 의미 |
|---|---|
| `ibrav = 2` | 면심입방(FCC) 브라베 격자 사용 |
| `A = 5.43` | 관용 입방셀의 격자상수: 5.43 Å |
| `nat = 2` | 계산 셀에 포함된 원자 수 |
| `ntyp = 1` | 원소 종류 수 |
| `ecutwfc = 50.0` | 파동함수의 평면파 에너지 컷오프: 50 Ry |
| `ecutrho = 400.0` | 전하밀도·퍼텐셜의 에너지 컷오프: 400 Ry |
| `occupations = 'fixed'` | 고정된 전자 점유수 사용 |

실리콘의 다이아몬드 구조는 FCC 격자에 두 원자의 기저를 배치해 표현합니다. 이 입력에서는 원시 단위셀에 원자 2개를 넣습니다.

에너지 컷오프를 높이면 더 많은 평면파를 사용하므로 일반적으로 계산량이 증가합니다.

> 이번 의사퍼텐셜에 기재된 최소 권장 컷오프는 파동함수 약 44 Ry, 전하밀도 약 175 Ry입니다. 실습에서는 각각 50 Ry와 400 Ry를 사용합니다. 이 값이 모든 연구 목적에 수렴한 조건이라는 뜻은 아닙니다.

### `&ELECTRONS` — 전자 계산의 수렴 조건

| 설정 | 의미 |
|---|---|
| `conv_thr = 1.0d-8` | 추정 SCF 에너지 오차의 수렴 기준: `10^-8 Ry` |
| `mixing_beta = 0.7` | 전자밀도 갱신 시 새 전자밀도를 반영하는 혼합 계수 |

`1.0d-8`은 Fortran에서 사용하는 지수 표기이며, `1 × 10^-8`을 뜻합니다.

> `conv_thr`는 연속한 두 단계의 총에너지 차이를 직접 지정하는 값과는 다릅니다.

### `ATOMIC_SPECIES` — 원소 정보

```text
Si 28.0855 Si.pbe-n-rrkjus_psl.1.0.0.UPF
```

왼쪽부터 다음 정보를 입력합니다.

1. 원소를 구분하는 이름
2. 원자 질량
3. 사용할 의사퍼텐셜 파일 이름

> 파일 이름은 `pseudo` 폴더에 저장한 이름과 정확히 일치해야 합니다. Ubuntu는 대문자와 소문자를 구분합니다.

### `ATOMIC_POSITIONS crystal` — 원자 위치

```text
Si 0.00 0.00 0.00
Si 0.25 0.25 0.25
```

`crystal`은 격자벡터를 기준으로 한 분율 좌표입니다. Å 단위의 직교 좌표가 아닙니다.

### `K_POINTS automatic` — k-point 격자

```text
4 4 4 0 0 0
```

| 값 | 의미 |
|---|---|
| 앞의 `4 4 4` | 역공간의 세 방향을 각각 4개로 나누는 격자 |
| 뒤의 `0 0 0` | 각 방향에서 격자 이동 없음 |

대칭성을 적용하면 실제 계산하는 k-point 수는 전체 격자점 수보다 줄어들 수 있습니다.

## 7. SCF 계산 실행

먼저 작업 폴더로 이동합니다.

```bash
cd ~/qe-examples/01-si-scf
```

다음 명령어를 실행합니다.

```bash
OMP_NUM_THREADS=1 pw.x -in si.scf.in > si.scf.out 2>&1
```

| 항목 | 의미 |
|---|---|
| `OMP_NUM_THREADS=1` | OpenMP 스레드 수를 1로 지정 |
| `pw.x` | 전자구조 계산 프로그램 실행 |
| `-in si.scf.in` | 읽을 입력 파일 지정 |
| `> si.scf.out` | 출력을 파일에 저장 |
| `2>&1` | 오류 출력도 같은 파일에 저장 |

출력을 파일로 보내므로 터미널에는 계산 과정이 표시되지 않습니다. 계산이 끝나면 명령어를 입력할 수 있는 프롬프트가 다시 나타납니다.

> 같은 출력 파일 이름으로 다시 실행하면 기존 `si.scf.out` 내용은 덮어씁니다.

### 계산 진행 상황 확인

계산 중 내용을 보려면 Ubuntu 터미널을 하나 더 열고 다음 명령어를 실행합니다.

```bash
tail -f ~/qe-examples/01-si-scf/si.scf.out
```

확인을 마치면 `Ctrl+C`를 누릅니다.

> 두 번째 터미널에서 `tail -f`를 종료해도 첫 번째 터미널의 계산은 계속 진행됩니다.

## 8. 계산 결과 확인

### SCF 수렴 및 정상 종료 확인

```bash
grep -E 'convergence has been achieved|convergence NOT achieved|JOB DONE' si.scf.out
```

성공 여부를 판단할 때는 다음 두 가지를 함께 확인합니다.

- `convergence has been achieved`: SCF 수렴 달성
- `JOB DONE.`: 프로그램 정상 종료

`convergence NOT achieved`가 표시되면 수렴하지 않은 상태입니다.

> `JOB DONE.`만으로 SCF 수렴을 판단하지 않습니다.

### 총에너지 확인

```bash
grep '!' si.scf.out
```

수렴한 SCF 총에너지 줄에는 일반적으로 `!`가 표시됩니다.

```text
!    total energy = ... Ry
```

`...`는 실제 출력된 수치로 표시됩니다. 여기서는 실행으로 확인하지 않은 에너지 값을 예시로 제시하지 않습니다.

| 출력 | 의미 |
|---|---|
| `total energy` | 이번 계산 셀의 총에너지 |
| `Ry` | Rydberg 에너지 단위 |

이 입력의 총에너지는 **Si 원자 2개를 포함한 계산 셀 전체**에 대한 값입니다.

> 총에너지를 원자 수로 나눈 값이 곧 응집에너지가 되는 것은 아닙니다. 응집에너지를 구하려면 별도의 기준 상태 계산이 필요합니다.

### 마지막 출력 확인

```bash
tail -n 30 si.scf.out
```

계산 종료 부분과 시간 정보 등을 확인합니다.

### 저장 데이터 확인

```bash
ls tmp/si.save
```

정상적으로 데이터가 저장되면 전자밀도와 구조·계산 정보를 담은 파일 등을 확인할 수 있습니다.

> `tmp`는 이 예제에서 지정한 폴더 이름입니다. 후속 계산에 사용할 데이터가 있으므로 필요한 동안 보관합니다.

## 9. 선택 사항 — MPI 프로세스 2개로 실행

기본 계산이 성공한 후 병렬 실행을 확인할 수 있습니다.

앞선 계산이 끝난 상태에서 다음 명령어를 실행합니다.

```bash
OMP_NUM_THREADS=1 mpirun -np 2 pw.x -in si.scf.in > si.scf.mpi.out 2>&1
```

| 항목 | 의미 |
|---|---|
| `mpirun` | MPI 병렬 프로그램 실행 |
| `-np 2` | MPI 프로세스 2개 사용 |
| `si.scf.mpi.out` | 병렬 실행의 출력 파일 |

결과를 확인합니다.

```bash
grep -E 'convergence has been achieved|convergence NOT achieved|JOB DONE' si.scf.mpi.out
grep '!' si.scf.mpi.out
```

동일한 입력을 사용하므로 총에너지는 수치 오차 범위에서 일치해야 합니다.

> 이처럼 작은 계산은 병렬 실행의 통신 비용 때문에 더 빨라지지 않을 수도 있습니다. 같은 `prefix`와 `outdir`를 사용하는 계산을 동시에 실행하지 않습니다.

## 10. 자주 발생하는 문제

### 의사퍼텐셜 파일을 찾지 못하는 경우

작업 위치와 파일을 확인합니다.

```bash
pwd
ls -lh pseudo
```

다음 사항을 확인합니다.

- 현재 위치가 `~/qe-examples/01-si-scf`인지
- 의사퍼텐셜 파일이 `pseudo` 폴더에 있는지
- 입력 파일의 파일 이름과 실제 이름이 일치하는지

### 입력 파일을 읽지 못하는 경우

```bash
cat si.scf.in
```

다음 사항을 확인합니다.

- 따옴표가 `'` 형태인지
- `&CONTROL`, `&SYSTEM`, `&ELECTRONS`가 각각 `/`로 끝나는지
- 원자 위치가 2줄인지
- Markdown 표시인 백틱이나 마지막 `EOF`가 파일에 들어가지 않았는지

### SCF가 수렴하지 않은 경우

먼저 의사퍼텐셜과 입력 내용이 예제와 같은지 확인합니다.

필요하면 `&ELECTRONS` 부분을 다음과 같이 수정해 다시 계산합니다.

```text
&ELECTRONS
  conv_thr = 1.0d-8,
  mixing_beta = 0.3,
  electron_maxstep = 200,
/
```

- `mixing_beta`를 낮추어 전자밀도 갱신 폭을 줄입니다.
- `electron_maxstep`으로 최대 SCF 반복 횟수를 늘립니다.

> 반복 횟수를 늘리는 것만으로 모든 수렴 문제가 해결되지는 않습니다.

### MPI 실행 슬롯이 부족한 경우

먼저 MPI 없이 기본 실행 명령어로 계산을 완료합니다.

```bash
OMP_NUM_THREADS=1 pw.x -in si.scf.in > si.scf.out 2>&1
```

## 11. GitHub에 올릴 파일

학습 내용을 공유하려면 다음 파일을 올리면 됩니다.

| 파일 | 용도 |
|---|---|
| `README.md` | 실습 설명 |
| `si.scf.in` | 계산 입력 파일 |
| `.gitignore` | 계산 중 생성되는 파일의 업로드 제외 설정 |

의사퍼텐셜은 이 문서의 공식 다운로드 링크를 제공합니다.

`.gitignore`는 다음 내용으로 만들 수 있습니다.

```gitignore
# Quantum ESPRESSO calculation data
tmp/

# Downloaded pseudopotentials
pseudo/*.UPF
pseudo/*.upf

# Calculation outputs
*.out
```

> 결과를 함께 공유하려면 확인한 출력 파일을 별도 이름으로 보관하고 `.gitignore` 규칙을 조정합니다. 큰 파동함수와 임시 계산 데이터는 저장소에 올리지 않는 편이 좋습니다.

## 실습 완료

다음 사항을 확인하면 첫 SCF 실습이 완료됩니다.

- [ ] 실리콘 의사퍼텐셜 다운로드
- [ ] `si.scf.in` 생성
- [ ] SCF 계산 실행
- [ ] `convergence has been achieved` 확인
- [ ] `JOB DONE.` 확인
- [ ] 총에너지 확인

**다음 실습에서는 에너지 컷오프와 k-point를 바꾸면서 총에너지가 얼마나 달라지는지 확인합니다.**

## 참고 자료

- [Quantum ESPRESSO 공식 홈페이지](https://www.quantum-espresso.org/)
- [pw.x 공식 입력 파일 설명](https://www.quantum-espresso.org/Doc/INPUT_PW.html)
- [공식 실리콘 의사퍼텐셜 목록](https://pseudopotentials.quantum-espresso.org/legacy_tables/ps-library/si)
- [실습용 Si 의사퍼텐셜 다운로드](https://pseudopotentials.quantum-espresso.org/upf_files/Si.pbe-n-rrkjus_psl.1.0.0.UPF)
