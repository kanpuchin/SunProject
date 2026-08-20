# SunProject

SunPy와 공개 태양 관측 자료를 이용하여 태양 영상 분석의 기초를 학습하고, 이후 태양 활동영역 시계열 분석으로 발전시키기 위한 개인 프로젝트입니다.

현재 단계에서는 SDO/AIA 자료의 검색과 다운로드, FITS 메타데이터 확인, 태양 좌표계 기반 영상 처리, 다파장 비교 및 ROI intensity 분석을 수행했습니다.

## Project Goals

- SunPy를 이용한 태양 관측 FITS 자료 처리
- 태양 좌표계와 WCS 이해
- SDO/AIA 다파장 영상 비교
- 활동영역과 quiet Sun의 정량적 intensity 비교
- 재현 가능한 분석 노트북과 문서 작성
- 향후 활동영역 시계열 및 플레어 분석으로 확장

## Repository Structure

```text
SunProject/
├── data/
│   └── .gitkeep
├── figures/
│   ├── aia_multiwavelength_comparison.png
│   └── aia171_roi_intensity_profiles.png
├── notebooks/
│   ├── 01_sunpy_warmup.ipynb
│   └── SDO_AIA_warmup.ipynb
├── notes/
│   ├── next_questions.md
│   ├── sunpy_cheatsheet.md
│   ├── sunpy_warmup_summary.md
│   └── troubleshooting.md
├── .gitignore
└── README.md
```

## Notebooks

### `01_sunpy_warmup.ipynb`

SunPy의 기본 기능을 익히기 위한 워밍업 노트북입니다.

주요 내용:

- FITS 파일 불러오기
- SunPy Map 생성
- 관측 메타데이터 확인
- WCS 기반 태양 영상 시각화
- helioprojective 좌표 사용
- `submap()`을 이용한 부분 영역 추출

### `SDO_AIA_warmup.ipynb`

SDO/AIA 자료를 이용한 다파장 비교 및 정량 분석 노트북입니다.

주요 내용:

- `Fido`를 이용한 AIA 자료 검색
- JSOCClient를 이용한 직접 export
- AIA 171 Å, 193 Å, 304 Å 영상 비교
- 동일 활동영역의 다파장 crop
- 파장별 대표 형성 온도와 구조 비교
- active-region 및 quiet-Sun ROI 설정
- ROI intensity 통계량 계산
- 1차원 intensity profile 작성
- 포화 픽셀 확인

## Main Results

### AIA Multi-wavelength Comparison

![AIA multi-wavelength comparison](figures/aia_multiwavelength_comparison.png)

2011년 6월 7일 약 06:33 UTC에 관측된 AIA 171 Å, 193 Å, 304 Å 영상을 비교했습니다.

각 채널은 서로 다른 온도 범위와 태양 대기 구조를 강조합니다.

| Channel | Main line | Representative temperature | Main structures |
|---|---|---:|---|
| AIA 171 Å | Fe IX | 약 0.6 MK | Quiet corona, coronal loops |
| AIA 193 Å | Fe XII | 약 1.3 MK | Hotter and diffuse corona |
| AIA 304 Å | He II | 약 0.05 MK | Chromosphere, transition region, prominence |

AIA 영상의 색은 실제 가시광선 색이 아니라 파장별 구조를 구분하기 위한 false color입니다.

### Active-region and Quiet-Sun Intensity Comparison

![AIA 171 intensity profiles](figures/aia171_roi_intensity_profiles.png)

AIA 171 Å 영상에서 같은 크기의 active-region ROI와 quiet-Sun ROI를 비교했습니다.

주요 결과:

- Mean intensity ratio (active / quiet): **6.94**
- Median intensity ratio (active / quiet): **4.70**
- Saturated pixels in the active-region ROI: **167**
- Saturated-pixel fraction: **0.595%**

활동영역의 intensity profile에는 coronal loop와 밝은 국소 구조에 대응하는 큰 변동과 peak가 나타났습니다. Quiet-Sun profile은 상대적으로 낮고 균일했습니다.

포화 픽셀은 평균과 표준편차를 증가시킬 수 있으므로, 두 영역의 전형적인 밝기를 비교할 때 중앙값도 함께 사용했습니다.

## Notes

- [`sunpy_warmup_summary.md`](notes/sunpy_warmup_summary.md): 워밍업에서 학습한 주요 개념
- [`sunpy_cheatsheet.md`](notes/sunpy_cheatsheet.md): 자주 사용하는 SunPy 명령어
- [`troubleshooting.md`](notes/troubleshooting.md): 실습 중 발생한 오류와 해결 방법
- [`next_questions.md`](notes/next_questions.md): 다음 분석 단계에서 확인할 질문

## Data

분석에는 JSOC에서 내려받은 SDO/AIA FITS 자료를 사용했습니다.

FITS 파일은 크기가 크고 필요할 때 다시 내려받을 수 있으므로 Git 저장소에는 포함하지 않습니다. 다운로드한 자료는 로컬의 다음 디렉토리에 저장합니다.

```text
data/
└── aia_multiwavelength/
```

노트북은 `notebooks/` 디렉토리에서 실행되며, 데이터와 figure 경로는 다음과 같은 상대경로를 사용합니다.

```python
data_dir = Path("../data")
figure_dir = Path("../figures")
```

## Environment

분석에는 다음 Python 패키지를 사용했습니다.

- Python
- SunPy
- Astropy
- NumPy
- Matplotlib
- DRMS
- Jupyter

기존 가상환경을 활성화한 뒤 Jupyter Notebook을 실행합니다.

```bash
conda activate sun_env
jupyter notebook
```

VS Code에서는 `sun_env`를 Jupyter kernel로 선택하여 노트북을 실행할 수 있습니다.

## Reproducibility

모든 노트북은 커널을 재시작한 뒤 첫 번째 셀부터 순서대로 실행하여 재현성을 확인했습니다.

단, JSOC 자료 export는 외부 서버 상태와 기존 pending request의 영향을 받을 수 있습니다. 자료를 이미 다운로드한 경우에는 로컬 FITS 파일을 이용해 분석 부분을 다시 실행할 수 있습니다.

## Next Step

다음 단계에서는 여러 시각의 SDO/AIA 영상을 이용하여 활동영역의 intensity light curve를 만들고 시간 변화를 분석할 예정입니다.

추후에는 다음 분석으로 확장할 수 있습니다.

- 태양 자전을 고려한 ROI 추적
- 다파장 light curve 비교
- GOES X-ray flux와 AIA intensity 비교
- 플레어 전후의 상승·peak·감쇠 시간 측정
- PSP 또는 태양 전파 관측 자료와의 연계
