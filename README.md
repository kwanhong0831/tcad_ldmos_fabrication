# LDMOS 소자 공정 시뮬레이션 (Silvaco TCAD ATHENA)

nLDMOS(LOCOS 기반) 소자를 Silvaco TCAD **ATHENA**(DeckBuild)로 공정 시뮬레이션한 커맨드 덱입니다. Well 형성 → LOCOS 형성 → Gate/LDD 형성 → Source/Drain 및 Body 형성 순으로 4단계에 걸쳐 소자 구조를 생성합니다.

## 파일 설명

| 파일 | 설명 |
|---|---|
| `tcad_code.str` | ATHENA 커맨드 덱 (텍스트). **주의**: 확장자는 `.str`이지만 실제 내용은 구조 파일이 아니라 실행 스크립트입니다. Silvaco 관례상 커맨드 덱은 보통 `.in` 확장자를 쓰므로, 실제로 DeckBuild에서 실행할 때는 `.in`으로 복사/저장해서 사용하는 것을 권장합니다. |
| `LDMOS 소자 설계 및 특성 분석 연구.pdf` | 시뮬레이션이 참고한 논문/설계 근거 |
| `26-1_반도체소자시뮬레이션_9조_발표자료.pdf` | 발표자료 |
| `26-1_반도체소자시뮬레이션_9조_회의록.pdf` | 회의록 |
| `(양식)최종보고서_9조.pdf` | 최종보고서 |

## 실행 환경

- **Silvaco TCAD** (ATHENA 모듈 포함 라이선스 필요)
- **DeckBuild** (ATHENA 커맨드 덱 실행용 GUI/런타임)
- **TonyPlot** (결과 구조 파일 시각화용, DeckBuild와 함께 설치됨)

> 상용 라이선스 소프트웨어이므로 저장소에는 시뮬레이션 코드만 포함되어 있습니다. 실행하려면 Silvaco TCAD가 설치·라이선스된 환경이 필요합니다.

## 실행 방법

1. DeckBuild를 실행하고 `tcad_code.str`(또는 `.in`으로 리네임한 사본)을 엽니다.
2. `Run` 버튼으로 전체를 순차 실행하거나, 아래 4개 파트를 구간별로 나눠 하나씩 실행합니다.
   - 각 파트는 `go athena`로 시작해 `quit`으로 끝나며, 이전 파트가 만든 구조 파일(`init infile=...`)을 입력으로 이어받습니다.
   - 구간별로 나눠 실행할 경우, 이전 파트의 출력 `.str` 파일이 같은 작업 폴더에 있어야 합니다.
3. 각 파트 끝의 `tonyplot ...` 명령으로 해당 단계 결과 구조를 바로 확인할 수 있습니다.

## 공정 단계 요약

| Part | 내용 | 주요 조건 | 입력 구조 파일 | 출력 구조 파일 |
|---|---|---|---|---|
| 1. Well Formation | p-well / n-well 형성 (LOCOS 기반 nLDMOS) | N-well: P 1.3e13 cm⁻², 150 keV / P-well: B 3.7e13 cm⁻², 160 keV / Drive-in 1150℃ 120min | 초기 실리콘 (Boron 5e14 cm⁻³) | `ldmos_01_well_fix.str` |
| 2. LOCOS Formation | Pad oxide → Nitride 마스크 → LOCOS 습식 산화 (960℃, 240min, 3구간 분할) | 최종 LOCOS 두께 목표 787.4 nm | `ldmos_01_well_fix.str` | `ldmos_02_locos_fix.str` |
| 3. Vth Adjust / Gate Oxide / Poly Gate / LDD / Spacer | Vth 임플란트, 게이트 산화막(900℃, 40min), Poly 게이트 증착, Boron/Arsenic LDD, 스페이서 형성 | Poly: 0.5 μm, P 5e19 cm⁻³ / 스페이서 약 150 nm | `ldmos_02_locos_fix.str` | `ldmos_03_gate_ldd_fix_v3.str` |
| 4. N+ Source/Drain & P+ Body | Screen oxide → N+ S/D 임플란트 → 활성화 어닐 → P+ Body(BF₂) 임플란트 → 어닐 | S/D: As 5e15 cm⁻², 60 keV / Body: BF₂ 3e15 cm⁻², 80 keV | `ldmos_03_gate_ldd_fix_v3.str` | `ldmos_final_figure2_clean_v4.str` |

## 알려진 이슈

- **Part 4 중복 블록**: 원본 파일의 Part 4 구간(`3번째` 이후) 안에 작성 중 중단된 초안(`method compress` 뒤 `d`로 끊긴 부분, 393~470행)이 남아 있고, 곧이어 완성된 본문(471~603행)이 다시 이어집니다. **실제로 실행되는 것은 두 번째(완성된) 블록**이며, 첫 번째 미완성 블록은 무시해도 됩니다.
- 마찬가지로 파일 맨 끝(605~662행)에는 이미 `quit`(603행) 이후에 위치한 P+ Body 관련 명령이 한 번 더 중복되어 남아 있습니다. 이 역시 이전 편집 과정에서 남은 잔여 텍스트로, 실행에는 영향을 주지 않지만 정리 시 삭제해도 무방합니다.
- 위 두 중복 구간을 정리한 버전을 별도로 두고 싶다면 `tcad_code_clean.in`처럼 별도 파일로 정리해서 올리는 것을 권장합니다.

## 참고

- BF₂ 임플란트가 사용 버전에서 오류를 낼 경우, 코드 주석에 안내된 대로 `implant boron dose=3.0e15 energy=40 tilt=7 pearson`으로 대체 가능합니다.
- 자세한 공정 근거와 결과 분석은 `LDMOS 소자 설계 및 특성 분석 연구.pdf` 및 최종보고서를 참고하세요.
