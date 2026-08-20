# SunPy Cheat Sheet

## 1. 기본 import

```python
from pathlib import Path

import numpy as np
import matplotlib.pyplot as plt
import astropy.units as u

from astropy.coordinates import SkyCoord
from astropy.time import Time

import sunpy.map
from sunpy.net import Fido
from sunpy.net import attrs as a
```

## 2. FITS 파일 찾기

```python
data_dir = Path("../data")
fits_files = sorted(data_dir.rglob("*.fits"))

print(f"FITS files: {len(fits_files)}")

for filepath in fits_files:
    print(filepath)
```

- `glob("*.fits")`: 현재 디렉토리에서 검색
- `rglob("*.fits")`: 하위 디렉토리까지 검색
- 노트북이 `notebooks/`에 있으면 `../data`가 프로젝트 루트의 `data/`를 가리킨다.

## 3. SunPy Map 생성

```python
smap = sunpy.map.Map(fits_files[0])
```

여러 파일을 한 번에 불러올 때:

```python
maps = [sunpy.map.Map(filepath) for filepath in fits_files]
```

## 4. Map 정보 확인

```python
print(smap)
print(smap.data.shape)
print(smap.date)
print(smap.wavelength)
print(smap.coordinate_frame)
print(smap.scale)
print(smap.meta)
```

특정 메타데이터 확인:

```python
print(smap.meta.get("exptime"))
print(smap.meta.get("instrume"))
print(smap.meta.get("date-obs"))
```

## 5. 기본 영상 그리기

```python
fig = plt.figure(figsize=(8, 8))
ax = fig.add_subplot(projection=smap)

smap.plot(axes=ax)
smap.draw_limb(axes=ax, color="white")

plt.tight_layout()
plt.show()
```

밝기 범위를 percentile로 조정:

```python
smap.plot(
    axes=ax,
    clip_interval=(1, 99.9) * u.percent
)
```

로그 스케일 사용:

```python
from matplotlib.colors import LogNorm

smap.plot(
    axes=ax,
    norm=LogNorm()
)
```

## 6. AIA colormap 확인

```python
print(smap.plot_settings["cmap"].name)
```

AIA Map에는 파장에 따라 다음과 같은 false-color colormap이 자동으로 적용된다.

```text
sdoaia171
sdoaia193
sdoaia304
```

## 7. 태양 좌표 정의

```python
point = SkyCoord(
    500 * u.arcsec,
    -400 * u.arcsec,
    frame=smap.coordinate_frame
)
```

사각형 영역 정의:

```python
bottom_left = SkyCoord(
    450 * u.arcsec,
    -750 * u.arcsec,
    frame=smap.coordinate_frame
)

top_right = SkyCoord(
    1100 * u.arcsec,
    -100 * u.arcsec,
    frame=smap.coordinate_frame
)
```

## 8. Submap 생성

```python
submap = smap.submap(
    bottom_left,
    top_right=top_right
)
```

## 9. ROI 표시

```python
fig = plt.figure(figsize=(8, 8))
ax = fig.add_subplot(projection=smap)

smap.plot(
    axes=ax,
    clip_interval=(1, 99.9) * u.percent
)

smap.draw_quadrangle(
    bottom_left,
    top_right=top_right,
    axes=ax,
    edgecolor="cyan",
    linewidth=2,
    label="ROI"
)

ax.legend()
plt.show()
```

## 10. 여러 Map 나란히 그리기

```python
maps = [map_171, map_193, map_304]
wavelengths = [171, 193, 304]

fig = plt.figure(figsize=(18, 6))

for i, (smap, wavelength) in enumerate(
    zip(maps, wavelengths),
    start=1
):
    ax = fig.add_subplot(1, 3, i, projection=smap)

    smap.plot(
        axes=ax,
        clip_interval=(1, 99.9) * u.percent
    )

    smap.draw_limb(
        axes=ax,
        color="white"
    )

    ax.set_title(f"AIA {wavelength} Å")

plt.tight_layout()
plt.show()
```

## 11. Map 선택하기

특정 파장의 Map만 선택:

```python
candidates = [
    smap for smap in maps
    if round(smap.wavelength.to_value(u.angstrom)) == 171
]
```

목표 시각과 가장 가까운 Map 선택:

```python
target_time = Time("2011-06-07T06:33:33")

nearest_map = min(
    candidates,
    key=lambda smap: abs((smap.date - target_time).sec)
)
```

## 12. Fido로 자료 검색

```python
result = Fido.search(
    a.Time(
        "2011-06-07 06:33:00",
        "2011-06-07 06:34:00"
    ),
    a.Instrument.aia,
    a.Wavelength(171 * u.angstrom)
)

print(result)
```

## 13. JSOC 직접 검색

```python
jsoc_email = "your_email@example.com"

result = Fido.search(
    a.Time(
        "2011-06-07 06:33:25",
        "2011-06-07 06:33:27"
    ),
    a.jsoc.Series("aia.lev1_euv_12s"),
    a.Wavelength(171 * u.angstrom),
    a.jsoc.Segment.image,
    a.jsoc.Notify(jsoc_email)
)

print(result)
```

JSOC 검색 결과에는 다음 문구가 나타나야 한다.

```text
Results from the JSOCClient
```

## 14. 자료 다운로드

```python
download_dir = Path("../data/aia")

download_dir.mkdir(
    parents=True,
    exist_ok=True
)

files = Fido.fetch(
    result,
    path=download_dir
)

print(files)
```

주의: JSOC 검색 결과를 slicing한 뒤 `Fido.fetch()`에 전달해도 선택한 행만 다운로드된다고 보장되지 않는다. 필요한 record만 검색되도록 시간 범위를 좁히는 편이 안전하다.

## 15. ROI 통계량 계산

```python
def intensity_stats(smap):
    finite_data = smap.data[
        np.isfinite(smap.data)
    ]

    return {
        "mean": np.mean(finite_data),
        "median": np.median(finite_data),
        "std": np.std(finite_data),
        "min": np.min(finite_data),
        "max": np.max(finite_data),
        "n_pixels": finite_data.size,
    }

stats = intensity_stats(submap)
print(stats)
```

## 16. 포화 픽셀 확인

```python
saturation_level = 16383

n_saturated = np.sum(
    submap.data >= saturation_level
)

n_finite = np.sum(
    np.isfinite(submap.data)
)

saturated_fraction = (
    100 * n_saturated / n_finite
)

print("Saturated pixels:", n_saturated)
print(f"Saturated fraction: {saturated_fraction:.3f}%")
```

## 17. 1차원 intensity profile

세로 방향으로 평균을 내어 가로 profile 생성:

```python
profile = np.nanmean(
    submap.data,
    axis=0
)

x = np.linspace(
    0,
    100,
    profile.size
)

fig, ax = plt.subplots(figsize=(10, 5))

ax.plot(x, profile)
ax.set_xlabel("Relative position within ROI (arcsec)")
ax.set_ylabel("Intensity (DN)")
ax.grid(alpha=0.3)

plt.tight_layout()
plt.show()
```

## 18. Figure 저장

```python
figure_dir = Path("../figures")
figure_dir.mkdir(exist_ok=True)

output_path = figure_dir / "figure.png"

fig.savefig(
    output_path,
    dpi=300,
    bbox_inches="tight"
)

print(f"Saved to: {output_path}")
```

`fig.savefig()`는 일반적으로 `plt.show()`보다 먼저 실행하는 것이 안전하다.
