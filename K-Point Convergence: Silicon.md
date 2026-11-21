# K-Point Convergence: Silicon

실리콘 결정의 k-point 격자를 단계적으로 촘촘하게 바꾸고, 총에너지의 변화를 비교합니다.

- 사전 준비: 첫 SCF 계산 및 에너지 컷오프 수렴성 실습 완료
- 계산 대상: Si 원자 2개를 포함한 원시 단위셀
- 계산 방법: DFT / PBE / Ultrasoft 의사퍼텐셜
- **모든 명령어는 Ubuntu 터미널에서 실행합니다.**

## 1. 실습 목표

앞선 실습에서는 에너지 컷오프를 바꾸었습니다.

이번에는 컷오프와 구조를 고정하고, **k-point 격자만 바꿉니다.**

| 실습 | 바꾸는 조건 | 확인하는 내용 |
|---|---|---|
| 에너지 컷오프 수렴성 | `ecutwfc`, `ecutrho` | 평면파 표현이 충분한가? |
| k-point 수렴성 | `K_POINTS` | 역공간의 표본점이 충분한가? |

> 한 번에 비교할 조건을 구분하면 결과가 달라진 원인을 이해하기 쉽습니다.

## 2. k-point란?

결정에서는 원자 배열이 주기적으로 반복됩니다.

전자 상태를 계산할 때는 이 주기성을 이용해 **역공간의 브릴루앙 영역(Brillouin zone)**을 다룹니다. 이 영역의 적분을 유한한 표본점으로 근사할 때 사용하는 점이 k-point입니다.

k-point 격자를 촘촘하게 하면 일반적으로 적분의 정확도가 높아지지만 계산량도 증가합니다.

> k-point는 원자를 배치하는 점이 아닙니다. 원자 위치는 실제 공간, k-point는 역공간의 정보입니다.

### 입력 형식

```text
K_POINTS automatic
4 4 4 0 0 0
```

| 값 | 의미 |
|---|---|
| 앞의 `4 4 4` | 역격자 세 방향의 격자 분할 수 |
| 뒤의 `0 0 0` | 각 방향에서 격자 이동 없음 |

전체 격자는 64개 점으로 구성되지만, 대칭성을 적용하면 실제 계산하는 점의 수는 줄어들 수 있습니다.

## 3. 이번 실습의 조건

| 항목 | 설정 |
|---|---|
| 구조 | 다이아몬드 구조의 실리콘 |
| 격자상수 | 5.43 Å |
| 원자 수 | 2 |
| 의사퍼텐셜 | `Si.pbe-n-rrkjus_psl.1.0.0.UPF` |
| `ecutwfc` | 예시 고정값 80 Ry |
| `ecutrho` | 예시 고정값 640 Ry |
| SCF 수렴 기준 | `1.0d-8 Ry` |
| k-point 격자 | `2 × 2 × 2`부터 `12 × 12 × 12`까지 |
| 격자 이동 | 모든 계산에서 `0 0 0` |

이번 실습은 앞선 컷오프 시험의 상한값인 80 Ry와 640 Ry를 예시로 사용합니다.

> 이 값들이 충분히 수렴했다고 미리 가정하지 않습니다. 앞선 실습에서 더 높은 값이 필요했다면 스크립트 상단의 두 컷오프를 수정하세요. 이번 비교 중에는 동일한 값을 유지합니다.

### 비교할 격자

| 격자 | 입력값 |
|---|---|
| `2 × 2 × 2` | `2 2 2 0 0 0` |
| `4 × 4 × 4` | `4 4 4 0 0 0` |
| `6 × 6 × 6` | `6 6 6 0 0 0` |
| `8 × 8 × 8` | `8 8 8 0 0 0` |
| `10 × 10 × 10` | `10 10 10 0 0 0` |
| `12 × 12 × 12` | `12 12 12 0 0 0` |

## 4. 작업 폴더 준비

```bash
mkdir -p ~/qe-examples/03-kpoint-convergence/pseudo
cd ~/qe-examples/03-kpoint-convergence
```

첫 SCF 실습에서 사용한 의사퍼텐셜을 복사합니다.

```bash
cp ~/qe-examples/01-si-scf/pseudo/Si.pbe-n-rrkjus_psl.1.0.0.UPF pseudo/
```

파일과 실행 경로를 확인합니다.

```bash
ls -lh pseudo/
command -v pw.x
```

> 복사에 실패하면 첫 실습 폴더의 위치와 의사퍼텐셜 파일 이름을 확인합니다.

## 5. 자동 계산 스크립트 만들기

다음 코드 블록 전체를 Ubuntu 터미널에 붙여 넣습니다.

```bash
cat > run_kpoints.sh <<'EOF'
#!/usr/bin/env bash
set -euo pipefail

cd -- "$(dirname -- "${BASH_SOURCE[0]}")"

# 모든 계산에 동일하게 적용할 컷오프
ecutwfc=80
ecutrho=640

pseudo_file="Si.pbe-n-rrkjus_psl.1.0.0.UPF"

if ! command -v pw.x >/dev/null 2>&1; then
    echo "오류: pw.x를 찾지 못했습니다. PATH를 확인하세요." >&2
    exit 1
fi

if [[ ! -s "pseudo/$pseudo_file" ]]; then
    echo "오류: pseudo/$pseudo_file 파일이 없습니다." >&2
    exit 1
fi

mkdir -p inputs outputs tmp

printf 'k_grid,ecutwfc_Ry,ecutrho_Ry,total_energy_Ry\n' > energies.csv

for k in 2 4 6 8 10 12
do
    input_file="inputs/si_k${k}.in"
    output_file="outputs/si_k${k}.out"

    mkdir -p "tmp/k${k}"

    cat > "$input_file" <<INPUT
&CONTROL
  calculation = 'scf',
  restart_mode = 'from_scratch',
  prefix = 'si',
  pseudo_dir = './pseudo',
  outdir = './tmp/k${k}',
/
&SYSTEM
  ibrav = 2,
  A = 5.43,
  nat = 2,
  ntyp = 1,
  ecutwfc = ${ecutwfc},
  ecutrho = ${ecutrho},
  occupations = 'fixed',
/
&ELECTRONS
  conv_thr = 1.0d-8,
  mixing_beta = 0.7,
/
ATOMIC_SPECIES
Si 28.0855 ${pseudo_file}

ATOMIC_POSITIONS crystal
Si 0.00 0.00 0.00
Si 0.25 0.25 0.25

K_POINTS automatic
${k} ${k} ${k} 0 0 0
INPUT

    echo "계산 시작: ${k} × ${k} × ${k}"

    if ! OMP_NUM_THREADS=1 pw.x -in "$input_file" > "$output_file" 2>&1; then
        echo "오류: 계산 실행에 실패했습니다. $output_file 확인" >&2
        exit 1
    fi

    if ! grep -q 'convergence has been achieved' "$output_file" ||
       ! grep -q 'JOB DONE' "$output_file" ||
       grep -q 'convergence NOT achieved' "$output_file"; then
        echo "오류: SCF 수렴 또는 정상 종료를 확인하지 못했습니다." >&2
        echo "$output_file 파일을 확인하세요." >&2
        exit 1
    fi

    energy=$(awk '/^[[:space:]]*![[:space:]]+total energy/ {value=$5} END {print value}' "$output_file")

    if [[ -z "$energy" ]]; then
        echo "오류: $output_file에서 총에너지를 찾지 못했습니다." >&2
        exit 1
    fi

    printf '%s,%s,%s,%s\n' \
        "$k" "$ecutwfc" "$ecutrho" "$energy" >> energies.csv

    echo "계산 완료: 총에너지 = ${energy} Ry"
done

echo "모든 계산이 완료되었습니다."
echo "결과 파일: energies.csv"
EOF
```

### 스크립트가 하는 일

1. k-point 격자별 입력 파일을 만듭니다.
2. 6개의 SCF 계산을 순서대로 실행합니다.
3. 각 계산의 수렴과 정상 종료를 확인합니다.
4. 총에너지를 추출해 `energies.csv`에 기록합니다.

| 코드 | 의미 |
|---|---|
| `for k in 2 4 6 8 10 12` | 격자 크기를 순서대로 변경 |
| `${k} ${k} ${k} 0 0 0` | 해당 격자를 입력 파일에 적용 |
| `outdir = './tmp/k${k}'` | 계산별 저장 위치 구분 |
| `> "$output_file" 2>&1` | 일반 출력과 오류를 같은 파일에 저장 |

> 다시 실행하면 기존 입력·출력 파일과 `energies.csv`를 덮어씁니다.

## 6. 계산 실행

```bash
bash run_kpoints.sh
```

화면에 현재 계산하는 격자가 표시됩니다.

```text
계산 시작: 2 × 2 × 2
```

계산이 완료되면 다음 격자로 넘어갑니다.

> 격자가 촘촘해지면 계산 시간이 늘어날 수 있습니다.

### 진행 상황 확인

다른 Ubuntu 터미널에서 다음 명령어를 실행합니다.

```bash
tail -f ~/qe-examples/03-kpoint-convergence/outputs/si_k2.out
```

확인할 계산에 따라 파일 이름을 `si_k4.out`, `si_k6.out` 등으로 변경합니다.

`Ctrl+C`를 누르면 출력 확인을 종료합니다.

### 오류가 발생한 경우

실패한 출력 파일의 마지막 부분을 확인합니다.

```bash
tail -n 50 outputs/si_k2.out
```

파일 이름은 실패한 계산에 맞게 변경합니다.

> 중간에 실패하면 CSV에는 성공한 계산까지만 저장됩니다. 원인을 해결한 뒤 전체 계산을 다시 실행합니다.

## 7. 계산 결과 확인

```bash
cat energies.csv
```

각 열의 의미는 다음과 같습니다.

| 열 이름 | 의미 |
|---|---|
| `k_grid` | 각 방향의 격자 분할 수 |
| `ecutwfc_Ry` | 모든 계산에 적용한 파동함수 컷오프 |
| `ecutrho_Ry` | 모든 계산에 적용한 전하밀도 컷오프 |
| `total_energy_Ry` | Si 원자 2개를 포함한 셀의 총에너지 |

각 출력에서 실제 계산에 사용된 k-point 수도 확인할 수 있습니다.

```bash
grep 'number of k points' outputs/si_k*.out
```

> 출력되는 수는 대칭성을 적용한 뒤의 k-point 수일 수 있으므로, 입력한 `k³`과 반드시 같지는 않습니다.

## 8. 기준 계산과 비교

가장 촘촘한 `12 × 12 × 12` 계산을 임시 기준으로 사용합니다.

```text
에너지 차이 = |현재 계산의 총에너지 - 기준 계산의 총에너지|
```

원자당 meV로 변환합니다.

```text
에너지 차이(meV/atom)
= 에너지 차이(Ry/cell) × 13.605693 × 1000 ÷ 2
```

마지막에 2로 나누는 이유는 이번 계산 셀의 원자 수가 2개이기 때문입니다.

### 비교 스크립트 만들기

Python이 없다면 설치합니다.

```bash
sudo apt install python3
```

다음 코드를 실행해 분석 파일을 만듭니다.

```bash
cat > analyze_kpoints.py <<'PY'
import csv
import math
from pathlib import Path

WORKDIR = Path(__file__).resolve().parent
EXPECTED_GRIDS = [2, 4, 6, 8, 10, 12]
RY_TO_MEV = 13.605693 * 1000
ATOM_COUNT = 2

with (WORKDIR / "energies.csv").open(newline="") as stream:
    rows = list(csv.DictReader(stream))

grids = [int(row["k_grid"]) for row in rows]

if grids != EXPECTED_GRIDS:
    raise SystemExit("오류: 6개 계산을 모두 완료한 뒤 분석하세요.")

cutoff_pairs = {
    (float(row["ecutwfc_Ry"]), float(row["ecutrho_Ry"]))
    for row in rows
}

if len(cutoff_pairs) != 1:
    raise SystemExit("오류: 계산 사이의 컷오프가 서로 다릅니다.")

energies = [
    float(row["total_energy_Ry"].replace("D", "E").replace("d", "e"))
    for row in rows
]

if not all(math.isfinite(value) for value in energies):
    raise SystemExit("오류: 유효하지 않은 에너지 값이 있습니다.")

reference = energies[-1]
result_path = WORKDIR / "convergence.csv"

print("기준 격자: 12 × 12 × 12")
print("격자 / 총에너지(Ry/cell) / 기준과의 차이(meV/atom)")

with result_path.open("w", newline="") as stream:
    writer = csv.writer(stream)
    writer.writerow([
        "k_grid",
        "total_energy_Ry",
        "abs_delta_meV_per_atom",
    ])

    for k, energy in zip(grids, energies):
        delta = abs(energy - reference) * RY_TO_MEV / ATOM_COUNT

        writer.writerow([
            k,
            f"{energy:.10f}",
            f"{delta:.6f}",
        ])

        print(f"{k}x{k}x{k} / {energy:.10f} / {delta:.6f}")

print(f"비교 결과 저장: {result_path.name}")
PY
```

실행합니다.

```bash
python3 analyze_kpoints.py
```

화면과 `convergence.csv`에 비교 결과가 표시됩니다.

## 9. 결과 해석

이번 실습의 학습용 기준은 다음과 같습니다.

```text
12 × 12 × 12 기준 계산과의 에너지 차이 ≤ 1 meV/atom
```

예를 들어 `8 × 8 × 8`과 `10 × 10 × 10`이 모두 기준을 만족한다면, `8 × 8 × 8`을 후보로 검토할 수 있습니다.

단, 실제 계산 결과를 확인한 뒤 판단해야 합니다.

### 확인할 사항

- 후보 격자와 그보다 촘촘한 격자에서 모두 차이가 작은가?
- 기준 격자보다 더 촘촘한 계산에서도 결과가 안정적인가?
- 컷오프를 포함한 나머지 조건이 동일한가?
- 모든 SCF 계산이 수렴했는가?

> k-point에 따른 총에너지는 반드시 한 방향으로만 변하지는 않습니다. 에너지가 계속 감소하는지보다 차이가 충분히 작아지고 안정되는지를 확인합니다.

### 기준 계산도 검증하기

`12 × 12 × 12`의 에너지 차이가 0인 것은 기준으로 선택했기 때문입니다.

그 자체가 충분히 수렴했다는 증거는 아닙니다.

필요하면 계산 목록에 다음 격자를 추가합니다.

```text
14 × 14 × 14
16 × 16 × 16
```

추가 실습에서는 다음 두 곳을 함께 수정합니다.

1. `run_kpoints.sh`의 반복 목록
2. `analyze_kpoints.py`의 `EXPECTED_GRIDS`

예시:

```bash
for k in 2 4 6 8 10 12 14 16
```

```python
EXPECTED_GRIDS = [2, 4, 6, 8, 10, 12, 14, 16]
```

분석 코드의 기준 격자 안내 문구도 `16 × 16 × 16`으로 변경합니다. 분석은 목록의 마지막 계산을 기준으로 수행합니다.

## 10. 그래프 만들기

`convergence.csv`를 Excel이나 Google 스프레드시트로 열고 산점도를 만듭니다.

| 항목 | 설정 |
|---|---|
| 가로축 | `k_grid` |
| 세로축 | `abs_delta_meV_per_atom` |
| 그래프 제목 | `K-Point Convergence of Silicon` |
| 가로축 이름 | `k-point Grid Size (n × n × n)` |
| 세로축 이름 | `Absolute Energy Difference (meV/atom)` |

가로축의 `8`은 `8 × 8 × 8` 격자를 의미합니다.

> 가로축 값은 대칭성을 적용한 뒤의 실제 k-point 수가 아닙니다.

## 11. 컷오프와 k-point를 함께 확인하기

이번 실습에서 k-point 후보를 정했다면, 그 격자에서 컷오프도 다시 확인하는 것이 좋습니다.

앞선 컷오프 실습은 `4 × 4 × 4` 격자에서 수행했기 때문입니다.

권장 순서는 다음과 같습니다.

1. 초기 k-point 격자에서 컷오프 후보를 찾습니다.
2. 충분한 컷오프에서 k-point 후보를 찾습니다.
3. 선택한 k-point 격자에서 컷오프의 충분성을 다시 확인합니다.
4. 필요한 경우 조건을 조정하고 재확인합니다.

> 최종 조건은 이번 구조와 의사퍼텐셜에 대한 결과입니다. 다른 물질이나 셀에도 같은 숫자를 그대로 적용하지 않습니다.

## 12. GitHub에 저장할 파일

| 파일 | 내용 |
|---|---|
| `README.md` | 실습 설명 |
| `run_kpoints.sh` | 자동 계산 스크립트 |
| `analyze_kpoints.py` | 결과 비교 스크립트 |
| `energies.csv` | 실제 계산한 총에너지 |
| `convergence.csv` | 기준과의 에너지 차이 |

`.gitignore` 예시:

```gitignore
# 계산 데이터
tmp/

# 자동 생성된 입력 및 출력
inputs/
outputs/

# 다운로드한 의사퍼텐셜
pseudo/*.UPF
pseudo/*.upf
```

## 실습 과제

1. `4 × 4 × 4`와 `8 × 8 × 8`에서 실제 사용된 k-point 수를 비교하세요.
2. SCF 수렴과 k-point 수렴의 차이를 설명하세요.
3. `1 meV/atom` 기준을 만족하는 격자 후보를 검토하세요.
4. 기준 격자가 충분히 촘촘한지 확인할 방법을 설명하세요.
5. 최종 격자에서 컷오프를 다시 확인해야 하는 이유를 설명하세요.

## 실습 완료

- [ ] 6가지 k-point 격자 계산 완료
- [ ] 모든 계산의 SCF 수렴 및 정상 종료 확인
- [ ] 총에너지 추출
- [ ] 원자당 에너지 차이 비교
- [ ] k-point 격자 후보 검토
- [ ] 기준 격자와 컷오프의 추가 검증 필요성 이해

**다음 실습에서는 계산 조건을 고정하고, 실리콘의 격자상수를 바꾸며 에너지가 가장 낮은 구조를 찾습니다.**

## 참고 자료

- [Quantum ESPRESSO 공식 입력 파일 설명](https://www.quantum-espresso.org/Doc/INPUT_PW.html)
- [실습용 Si 의사퍼텐셜](https://pseudopotentials.quantum-espresso.org/upf_files/Si.pbe-n-rrkjus_psl.1.0.0.UPF)
