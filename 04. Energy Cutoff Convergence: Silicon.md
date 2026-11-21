# Energy Cutoff Convergence: Silicon

실리콘 결정의 에너지 컷오프를 단계적으로 높이고, 총에너지의 변화를 비교합니다.

- 사전 준비: `First SCF Calculation: Silicon` 실습 완료
- 계산 대상: Si 원자 2개를 포함한 원시 단위셀
- **모든 명령어는 Ubuntu 터미널에서 실행합니다.**

> 이번 실습에서는 입력 파일을 여러 개 만들고 순서대로 계산하는 간단한 Bash 스크립트를 사용합니다.

## 1. 실습 목표

첫 SCF 계산에서는 다음 값을 사용했습니다.

```text
ecutwfc = 50.0
ecutrho = 400.0
```

하지만 계산이 정상적으로 끝났다고 해서 이 값들이 충분하다고 판단할 수는 없습니다.

이번에는 컷오프를 높여도 총에너지가 거의 달라지지 않는지 확인합니다.

이렇게 계산 조건에 따른 결과의 변화를 확인하는 과정을 **수렴성 테스트(convergence test)**라고 합니다.

### 두 가지 수렴의 차이

| 구분 | 확인하는 내용 |
|---|---|
| SCF 수렴 | 주어진 계산 조건에서 전자 계산이 자기일관성에 도달했는가? |
| 컷오프 수렴 | 컷오프를 더 높여도 관심 있는 결과가 충분히 안정적인가? |

> SCF가 수렴해도 컷오프에 대한 수렴성은 별도로 확인해야 합니다.

## 2. 이번 실습의 계산 조건

| 항목 | 설정 |
|---|---|
| 구조 | 다이아몬드 구조의 실리콘 |
| 격자상수 | 5.43 Å |
| 원자 수 | 2 |
| 의사퍼텐셜 | `Si.pbe-n-rrkjus_psl.1.0.0.UPF` |
| k-point 격자 | `4 × 4 × 4`, 이동 없음 |
| SCF 수렴 기준 | `1.0d-8 Ry` |
| `ecutwfc` | 30, 40, 50, 60, 70, 80 Ry |
| `ecutrho` | 각 `ecutwfc`의 8배 |

계산할 컷오프 조합은 다음과 같습니다.

| `ecutwfc` (Ry) | `ecutrho` (Ry) |
|---:|---:|
| 30 | 240 |
| 40 | 320 |
| 50 | 400 |
| 60 | 480 |
| 70 | 560 |
| 80 | 640 |

> 30 Ry와 40 Ry는 이 의사퍼텐셜의 파동함수 최소 권장값인 약 44 Ry보다 낮습니다. 낮은 컷오프에서 결과가 어떻게 달라지는지 관찰하기 위해 포함합니다.

### 왜 두 컷오프를 함께 바꾸나요?

Ultrasoft 의사퍼텐셜을 사용하는 이번 입문 실습에서는 다음 비율로 두 값을 함께 높입니다.

```text
ecutrho = 8 × ecutwfc
```

이는 두 컷오프를 함께 높이는 경로에서 총에너지의 안정성을 확인하는 방법입니다.

> 8배가 모든 의사퍼텐셜에 적용되는 규칙은 아닙니다. 연구용 조건을 결정할 때는 충분히 높은 `ecutrho`에서 `ecutwfc`를 검사하고, 선택한 `ecutwfc`에서 `ecutrho`도 별도로 검사합니다.

## 3. 작업 폴더 준비

```bash
mkdir -p ~/qe-examples/02-cutoff-convergence/pseudo
cd ~/qe-examples/02-cutoff-convergence
```

첫 실습에서 사용한 의사퍼텐셜을 복사합니다.

```bash
cp ~/qe-examples/01-si-scf/pseudo/Si.pbe-n-rrkjus_psl.1.0.0.UPF pseudo/
```

복사 결과를 확인합니다.

```bash
ls -lh pseudo/
```

다음 파일이 보여야 합니다.

```text
Si.pbe-n-rrkjus_psl.1.0.0.UPF
```

> `No such file or directory`가 나오면 첫 실습 폴더의 위치와 파일 이름을 확인합니다.

## 4. 자동 계산 스크립트 만들기

다음 코드 전체를 Ubuntu 터미널에 붙여 넣습니다.

```bash
cat > run_cutoff.sh <<'EOF'
#!/usr/bin/env bash
set -euo pipefail

# Always work in the directory containing this script.
cd -- "$(dirname -- "${BASH_SOURCE[0]}")"

pseudo_file="Si.pbe-n-rrkjus_psl.1.0.0.UPF"

if ! command -v pw.x >/dev/null 2>&1; then
    echo "ERROR: pw.x was not found. Check your PATH." >&2
    exit 1
fi

if [[ ! -s "pseudo/$pseudo_file" ]]; then
    echo "ERROR: Missing pseudopotential: pseudo/$pseudo_file" >&2
    exit 1
fi

mkdir -p inputs outputs tmp

printf 'ecutwfc_Ry,ecutrho_Ry,total_energy_Ry\n' > energies.csv

for cutoff in 30 40 50 60 70 80
do
    density_cutoff=$((8 * cutoff))
    input_file="inputs/si_${cutoff}.in"
    output_file="outputs/si_${cutoff}.out"

    mkdir -p "tmp/ecut_${cutoff}"

    cat > "$input_file" <<INPUT
&CONTROL
  calculation = 'scf',
  restart_mode = 'from_scratch',
  prefix = 'si',
  pseudo_dir = './pseudo',
  outdir = './tmp/ecut_${cutoff}',
/
&SYSTEM
  ibrav = 2,
  A = 5.43,
  nat = 2,
  ntyp = 1,
  ecutwfc = ${cutoff},
  ecutrho = ${density_cutoff},
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
4 4 4 0 0 0
INPUT

    echo "Running: ecutwfc=${cutoff} Ry, ecutrho=${density_cutoff} Ry"

    if ! OMP_NUM_THREADS=1 pw.x -in "$input_file" > "$output_file" 2>&1; then
        echo "ERROR: Calculation failed. Check $output_file" >&2
        exit 1
    fi

    if ! grep -q 'convergence has been achieved' "$output_file" ||
       ! grep -q 'JOB DONE' "$output_file" ||
       grep -q 'convergence NOT achieved' "$output_file"; then
        echo "ERROR: SCF convergence or completion check failed." >&2
        echo "Check $output_file" >&2
        exit 1
    fi

    energy=$(awk '/^[[:space:]]*![[:space:]]+total energy/ {value=$5} END {print value}' "$output_file")

    if [[ -z "$energy" ]]; then
        echo "ERROR: Total energy was not found in $output_file" >&2
        exit 1
    fi

    printf '%s,%s,%s\n' "$cutoff" "$density_cutoff" "$energy" >> energies.csv
    echo "Completed: total energy = ${energy} Ry"
done

echo "All calculations completed."
echo "Results saved to energies.csv"
EOF
```

### 스크립트의 주요 기능

| 코드 | 역할 |
|---|---|
| `for cutoff in ...` | 지정한 컷오프를 순서대로 사용 |
| `$((8 * cutoff))` | `ecutrho` 계산 |
| `${cutoff}` | 현재 컷오프 값을 입력 파일에 삽입 |
| `pw.x -in ...` | 생성한 입력 파일로 SCF 계산 |
| `grep` | 수렴 및 정상 종료 문구 확인 |
| `awk` | 최종 총에너지 추출 |
| `energies.csv` | 컷오프별 총에너지 저장 |

각 계산은 서로 다른 `outdir`를 사용하므로 계산 데이터가 구분됩니다.

> 스크립트를 다시 실행하면 기존 입력·출력 파일과 `energies.csv`를 덮어씁니다.

## 5. 계산 실행

```bash
bash run_cutoff.sh
```

6개의 계산을 순서대로 실행합니다.

실행 중에는 다음과 같이 현재 계산 조건이 표시됩니다.

```text
Running: ecutwfc=30 Ry, ecutrho=240 Ry
```

각 계산이 성공하면 총에너지가 표시됩니다. 실제 수치는 계산 결과에 따라 출력됩니다.

### 진행 상황 확인

다른 Ubuntu 터미널에서 현재 계산의 출력을 확인할 수 있습니다.

```bash
tail -f ~/qe-examples/02-cutoff-convergence/outputs/si_30.out
```

30 Ry 계산이 끝났다면 확인할 파일 이름을 `si_40.out` 등으로 바꿉니다.

`Ctrl+C`를 누르면 출력 확인을 종료합니다.

## 6. 총에너지 표 확인

모든 계산이 완료되면 다음 명령어를 실행합니다.

```bash
cat energies.csv
```

파일의 각 열은 다음 의미입니다.

| 열 이름 | 의미 |
|---|---|
| `ecutwfc_Ry` | 파동함수 컷오프 |
| `ecutrho_Ry` | 전하밀도·퍼텐셜 컷오프 |
| `total_energy_Ry` | Si 원자 2개를 포함한 셀의 총에너지 |

> 스크립트가 중간에 멈추면 CSV에는 성공한 계산까지만 기록됩니다. 결과 해석 전에 6개 계산이 모두 완료되었는지 확인합니다.

## 7. 기준 계산과 에너지 차이 비교

이번 실습에서는 가장 높은 컷오프인 **80 Ry 계산**을 임시 기준으로 사용합니다.

각 계산의 에너지 차이는 다음과 같이 구합니다.

```text
에너지 차이 = |현재 계산의 총에너지 - 80 Ry 계산의 총에너지|
```

원자당 meV로 변환하려면 다음 식을 사용합니다.

```text
에너지 차이(meV/atom)
= 에너지 차이(Ry/cell) × 13.605693 × 1000 ÷ 2
```

- `1 Ry ≈ 13.605693 eV`
- `1 eV = 1000 meV`
- 이번 계산 셀에는 Si 원자 2개가 있으므로 2로 나눕니다.

### 비교표 생성

Python이 없다면 설치합니다.

```bash
sudo apt install python3
```

다음 코드 전체를 실행합니다.

```bash
python3 - <<'PY'
import csv
import math
from pathlib import Path

with Path("energies.csv").open(newline="") as stream:
    rows = list(csv.DictReader(stream))

expected = [30, 40, 50, 60, 70, 80]
actual = [int(row["ecutwfc_Ry"]) for row in rows]

if actual != expected:
    raise SystemExit("ERROR: Complete all six calculations before analysis.")

energies = [float(row["total_energy_Ry"].replace("D", "E").replace("d", "e"))
            for row in rows]

if not all(math.isfinite(energy) for energy in energies):
    raise SystemExit("ERROR: Invalid energy value.")

reference = energies[-1]
ry_to_mev = 13.605693 * 1000
atom_count = 2

with Path("convergence.csv").open("w", newline="") as stream:
    writer = csv.writer(stream)
    writer.writerow([
        "ecutwfc_Ry",
        "ecutrho_Ry",
        "total_energy_Ry",
        "abs_delta_meV_per_atom",
    ])

    print("Reference: ecutwfc = 80 Ry, ecutrho = 640 Ry")
    print(f"{'ecutwfc':>8} {'ecutrho':>8} {'Energy (Ry/cell)':>20} {'Delta (meV/atom)':>18}")

    for row, energy in zip(rows, energies):
        delta = abs(energy - reference) * ry_to_mev / atom_count
        writer.writerow([
            row["ecutwfc_Ry"],
            row["ecutrho_Ry"],
            f"{energy:.10f}",
            f"{delta:.6f}",
        ])
        print(
            f"{int(row['ecutwfc_Ry']):8d} "
            f"{int(row['ecutrho_Ry']):8d} "
            f"{energy:20.10f} "
            f"{delta:18.6f}"
        )

print("Comparison saved to convergence.csv")
PY
```
화면과 `convergence.csv`에 기준 계산과의 에너지 차이가 출력됩니다.

## 8. 결과 해석

연습용 판정 기준으로 다음을 설정합니다.

```text
80 Ry 기준 계산과의 차이가 1 meV/atom 이하
```

예를 들어 50 Ry, 60 Ry, 70 Ry에서 모두 이 기준을 만족하면, 50 Ry를 이번 시험 범위의 컷오프 후보로 검토할 수 있습니다.

다음 사항을 함께 확인합니다.

- 한 지점뿐 아니라 그보다 높은 여러 컷오프에서도 차이가 작은지 확인합니다.
- 80 Ry의 차이가 0인 이유는 해당 계산을 기준으로 선택했기 때문입니다.
- 80 Ry 자체가 충분히 높은지는 이 비교만으로 판단할 수 없습니다.
- 필요하면 90 Ry와 100 Ry 계산을 추가해 기준값의 안정성을 확인합니다.

> `1 meV/atom`은 이번 실습의 학습용 기준입니다. 필요한 정확도는 총에너지 차이, 힘, 응력 등 계산하려는 물리량에 따라 달라집니다.

이번 실습에서는 k-point 격자를 고정했으며, k-point에 대한 수렴성은 아직 확인하지 않았습니다.

**따라서 이번 결과는 지정한 구조, 컷오프 비율, k-point 조건에서의 수렴성 확인에 해당합니다.**

## 9. 그래프로 확인하기

`convergence.csv`를 Excel이나 Google 스프레드시트로 열고 산점도를 만듭니다.

| 축 | 사용할 열 |
|---|---|
| 가로축 | `ecutwfc_Ry` |
| 세로축 | `abs_delta_meV_per_atom` |

그래프 제목 예시:

```text
Energy Cutoff Convergence of Silicon
```

축 이름 예시:

```text
X: Wavefunction Cutoff (Ry)
Y: Absolute Energy Difference (meV/atom)
```

컷오프가 커질수록 기준 계산과의 에너지 차이가 작아지고 안정되는지 확인합니다.

## 10. GitHub에 저장할 파일

| 파일 | 내용 |
|---|---|
| `README.md` | 실습 설명 |
| `run_cutoff.sh` | 자동 계산 스크립트 |
| `energies.csv` | 실제 계산에서 얻은 총에너지 |
| `convergence.csv` | 기준 계산과의 비교 결과 |

> CSV에는 직접 계산해서 얻은 결과를 저장합니다.

계산 데이터와 다운로드 파일을 제외하는 `.gitignore` 예시:

```gitignore
# 계산 데이터
tmp/

# 자동 생성된 입력 및 출력 파일
inputs/
outputs/

# 다운로드한 의사퍼텐셜
pseudo/*.UPF
pseudo/*.upf
```

## 실습 과제

1. 모든 SCF 계산이 수렴하고 정상 종료되었는지 확인하세요.
2. 컷오프에 따라 총에너지가 어떻게 변하는지 설명하세요.
3. 학습용 기준인 `1 meV/atom`을 만족하는 컷오프 후보를 검토하세요.
4. 80 Ry가 충분한 기준인지 확인하려면 어떤 계산을 추가해야 할지 생각해 보세요.

## 실습 완료

- [ ] 6가지 조건의 SCF 계산 실행
- [ ] 모든 계산의 수렴 및 정상 종료 확인
- [ ] 컷오프별 총에너지 확인
- [ ] 원자당 에너지 차이 비교
- [ ] 이번 조건에서 사용할 컷오프 후보 검토

**다음 실습에서는 k-point 격자를 바꾸며 총에너지의 수렴성을 확인합니다.**

## 참고 자료

- [Quantum ESPRESSO 공식 입력 파일 설명](https://www.quantum-espresso.org/Doc/INPUT_PW.html)
- [실습용 Si 의사퍼텐셜](https://pseudopotentials.quantum-espresso.org/upf_files/Si.pbe-n-rrkjus_psl.1.0.0.UPF)
