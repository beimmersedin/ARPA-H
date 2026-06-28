# ARPA-H: CRC 단일세포 MSI vs MSS Consensus DEG 분석

대장암(Colorectal Cancer, CRC) Atlas 단일세포 RNA-seq 데이터에서 **세포타입별 차등발현 유전자(DEG)**를 분석하고, 여러 통계 방법의 결과를 **consensus(rank aggregation)** 로 종합하여 **MSI(Microsatellite Instability-high) vs MSS(Microsatellite Stable)** 표현형을 구분하는 마커 유전자를 발굴하는 프로젝트입니다.

## 분석 파이프라인

```
원본 CRC Atlas 발현 행렬
        │
        ▼
  세포타입별 분할 / 추출           (split*.ipynb)
        │
        ▼
  세포타입별 DEG 검정
  (Wilcoxon, limmatrend,
   RISCQP, MAST, monocle)
        │
        ▼
  Consensus rank aggregation       (raw_consensus_*, fillter_consensus_*)
  + logFC 방향 투표로 MSI/MSS 분류
        │
        ▼
  MSI/MSS 특이 마커 추출 & 시각화    (0527, 0528_msi, 0528_mss,
                                     consensus_celltype_specific3_mss)
        │
        ▼
  결과 (MSI/MSS_specific_*.xlsx)
```

## 디렉터리 구성

### 1. 데이터 분할 / 탐색 — `split*.ipynb`
- 9개 관심 유전자(VSIG4, LYVE1, CLEC5A, CX3CR1, FN1, IL11, NRAP, SDC1, SCG5)의 세포타입별 발현 행렬을 로드하고, MSI/MSS 그룹별 요약 통계 및 박스플롯을 생성합니다.
- `split.ipynb` (기본), `split_yeonnu.ipynb`, `split_정은이.ipynb` (팀원별 변형본)

### 2. 세포타입별 DEG 결과 처리 — `h5ad_{CellType}.ipynb` (14개)
동일 템플릿으로 세포타입별 Wilcoxon DEG 결과를 처리합니다.
- 입력: `{CellType}_Wilcoxon_DEG_crcatlas.csv` (약 28,476 유전자; names, scores, logfoldchanges, pvals, pvals_adj)
- 처리: adjusted p-value로 랭크 → 9개 관심 유전자 필터 → logFC 부호로 MSI/MSS 판정 → `-log10(adj p)` 계산
- 대상 세포타입: B_cell, T_cell, Cancer_cell, Epithelial, ILC, Mast, Myeloid, Neutrophil, NK, Plasma, Schwann_cell, Stromal, Mini_10percent

### 3. Consensus 분석 — `raw_consensus_*` / `fillter_consensus_*` (각 8개)
여러 DEG 방법의 결과를 **랭크 합산(rank_sum)** 으로 종합하고, logFC 방향 투표(3개 이상 방법 일치)로 `MSI_high` / `MSS_high` 를 분류합니다.

| 구분 | 유전자 수 | 사용 방법 | 파일 접미사 |
|------|-----------|-----------|-------------|
| **raw** | 전체 (~28,476) | 4개 (Wilcoxon, limmatrend, RISCQP, MAST) | `_raw.csv` |
| **fillter** | 사전 필터 (~1,295–3,215) | 5개 (+ monocle) | `_re.csv` |

- 대상 세포타입: B_cell, epithelial(`eptheilal`), mast, myeloid, neutrophil, plasma, stromal, random10(대조군)

### 4. MSI/MSS 특이 마커 분석
- `0527.ipynb` — 세포타입별 raw/filtered 유전자 통계 탐색 (mean/median)
- `consensus_celltype_specific3_mss.ipynb` — rank_sum 임계값으로 세포타입 특이 유전자 추출 (myeloid: 397개 = MSS 333 / MSI 64)
- `0528_msi.ipynb` / `0528_mss.ipynb` — 엑셀 마커 리스트 로드 → myeloid 세포에서 MSI/MSS 발현 박스플롯
- `more.ipynb` — 입력 파일 경로 및 Wilcoxon 통계 레퍼런스

### 5. 결과 파일
- `MSI_specific_with_median_expression.xlsx` — MSI 특이 유전자 41개 (S100A8, S100A9, IFI30, FCN1, VCAN 등)
- `MSS_specific_with_median_expression.xlsx` — MSS 특이 유전자 95개 (C1QA, C1QB, HLA-DPB1, CD63, NAMPT 등)
- 각 파일은 세포타입별 `rank_sum` 과 MSI_high / MSS_high 중앙값 발현 비율을 포함합니다.

## 주요 데이터 경로

> 노트북은 아래 외부 스토리지 경로를 참조합니다. 저장소에는 노트북과 결과 엑셀만 포함되며, 원본 데이터는 포함되지 않습니다.

- 원본 / DEG 데이터: `/nas/arpa_h_repository/public_data/CRC_Atlas_1/CRC-Atlas-Split/crc-atlas/Deg_data/`
- 분할 행렬: `/data/project/arpa_h/raw_data/split/`
- Consensus 병합 결과: `/data/project/yeonu/consensus/celltype_rank_sum_merged.csv`

## 사용 라이브러리

- pandas, numpy
- matplotlib
- scanpy / anndata (h5ad, 단일세포 처리)

## 핵심 개념

- **MSI / MSS**: 대장암의 마이크로새틀라이트 불안정성 상태. 면역치료 반응성 등 임상적으로 중요한 분자 표현형.
- **Consensus DEG**: 단일 검정법의 편향을 줄이기 위해 여러 방법의 유전자 랭크를 합산하여 일관되게 상위에 오르는 유전자를 robust 마커로 선택하는 방법.
