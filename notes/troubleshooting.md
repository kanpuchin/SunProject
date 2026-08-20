# SunPy Warm-up Troubleshooting

## 1. VSO 검색 결과 다운로드 실패

### 문제

`Fido.search()`를 사용한 AIA 자료 검색은 성공했지만, `Fido.fetch()`를 실행했을 때 VSO를 통한 다운로드가 반복적으로 실패했다.

검색 결과의 Provider가 JSOC로 표시되어도, 결과 표가 `Results from the VSOClient` 아래에 있다면 실제 검색 및 다운로드 요청은 VSOClient를 통해 처리된다.

### 해결

JSOC 속성을 명시하여 JSOCClient로 직접 검색했다.

```python
result = Fido.search(
    a.Time(start_time, end_time),
    a.jsoc.Series("aia.lev1_euv_12s"),
    a.Wavelength(171 * u.angstrom),
    a.jsoc.Segment.image,
    a.jsoc.Notify(jsoc_email)
)
```

검색 결과에 다음 문구가 표시되는지 확인했다.

```text
Results from the JSOCClient
```

## 2. VSO와 JSOC 검색 결과의 시각 차이

### 문제

동일한 자료를 검색했지만 VSO의 `Start Time`과 JSOC의 `T_REC`가 서로 다르게 나타났다.

### 원인과 해결

JSOC의 `T_REC`는 자료를 일정한 cadence에 배치하기 위한 명목상의 record time이다. 따라서 실제 관측 시각과 몇 초 정도 다를 수 있다.

다운로드 후 SunPy Map의 `date` 속성을 사용하여 FITS 메타데이터에 기록된 관측 시각을 확인했다.

```python
smap = sunpy.map.Map(filepath)
print(smap.date)
```

다파장 비교에서는 각 Map의 실제 관측 시각을 확인하고, 목표 시각과 가장 가까운 자료를 선택했다.

## 3. Sliced JSOC 결과 다운로드 문제

### 문제

다음과 같이 JSOC 검색 결과의 한 행을 선택한 뒤 다운로드하려 했다.

```python
selected = result[0, 0]
files = Fido.fetch(selected)
```

그러나 JSOCClient는 sliced query result의 일부만 다운로드하는 방식을 지원하지 않았다. 선택된 행과 관계없이 원래 검색 결과에 포함된 자료가 모두 export될 수 있었다.

### 해결

특정 record 하나만 검색되도록 처음부터 검색 시간 범위를 좁혔다.

```python
result = Fido.search(
    a.Time(
        "2011-06-07 06:33:25",
        "2011-06-07 06:33:27"
    ),
    ...
)
```

이미 여러 파일이 다운로드된 경우에는 모든 FITS 파일을 불러온 뒤 파장과 관측 시각을 기준으로 필요한 Map을 선택했다.

## 4. 커널 재시작 후 변수 소실

### 문제

노트북을 닫거나 커널을 재시작한 후 `files_171`, `map_171` 등의 변수가 정의되지 않았다는 오류가 발생했다.

### 원인과 해결

Python 변수는 커널 메모리에만 존재하므로 커널을 재시작하면 사라진다. 반면 다운로드된 FITS 파일은 디스크에 남아 있다.

따라서 파일 경로를 다시 검색하여 Map을 생성했다.

```python
download_dir = Path("../data/aia_multiwavelength")
fits_files = sorted(download_dir.glob("*.fits"))

all_maps = [
    sunpy.map.Map(filepath)
    for filepath in fits_files
]
```

## 5. 데이터 디렉토리 변경 후 경로 오류

### 문제

자료를 다음 위치에서 프로젝트 루트의 `data/`로 이동한 뒤 기존 상대경로가 작동하지 않았다.

```text
SunProject/notebooks/data
```

변경된 구조:

```text
SunProject/data
SunProject/notebooks
```

### 해결

노트북은 `notebooks/`를 기준으로 실행되므로 데이터 경로를 다음과 같이 수정했다.

```python
download_dir = Path("../data/aia_multiwavelength")
```

현재 작업 디렉토리와 실제 경로는 다음 명령으로 확인할 수 있다.

```python
from pathlib import Path

print(Path.cwd())
print(download_dir.resolve())
```

## 6. JSOC pending export 오류

### 문제

JSOC 다운로드 셀을 실행한 뒤 로컬에서 실행을 중단했다. 이후 새로운 export를 제출하려 하자 다음 오류가 발생했다.

```text
DrmsExportError:
User has 1 pending export requests;
please wait until at least one request has completed
before submitting a new one.
```

### 원인

노트북 셀을 중단해도 로컬의 대기 또는 다운로드 과정만 멈춘다. 이미 JSOC 서버에 제출된 export 작업은 계속 처리된다.

JSOC는 동일 사용자의 export 요청이 처리 중일 때 새로운 요청을 제한할 수 있다.

### 해결

오류 메시지에 표시된 request ID를 이용해 기존 요청의 상태를 확인했다.

```python
import drms

client = drms.Client()

request = client.export_from_id(
    "JSOC_REQUEST_ID"
)

print(request.status)
print(request.has_finished())
print(request.has_succeeded())
```

기존 요청이 완료될 때까지 `Fido.fetch()`를 반복 실행하지 않았다.

## 7. 활동영역의 포화 픽셀

### 문제

Active-region ROI의 최대 intensity가 `16383 DN`으로 나타났고 평균과 표준편차가 매우 크게 계산되었다.

### 원인

ROI 안에 AIA 영상의 포화 상한에 도달한 픽셀이 포함되어 있었다.

### 해결

포화 픽셀의 개수와 비율을 확인했다.

```python
saturation_level = 16383

n_saturated = np.sum(
    active_roi.data >= saturation_level
)

n_finite = np.sum(
    np.isfinite(active_roi.data)
)

fraction = 100 * n_saturated / n_finite

print(n_saturated)
print(f"{fraction:.3f}%")
```

포화 픽셀은 전체 유효 픽셀의 약 0.595%였다. 평균은 밝은 극단값의 영향을 받을 수 있으므로, 두 ROI의 전형적인 intensity를 비교할 때 중앙값 비율도 함께 사용했다.

## 8. 재현성 확인

모든 노트북에서 커널을 재시작한 후 셀을 처음부터 순서대로 실행했다.

확인 항목:

- 모든 셀이 순서대로 실행되는가
- 이전 커널에 남은 변수에 의존하지 않는가
- 데이터 상대경로가 올바른가
- 분석 결과와 figure가 다시 생성되는가
- 오류 없이 마지막 셀까지 실행되는가
