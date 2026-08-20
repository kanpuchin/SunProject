# SunPy Warm-up Summary

## 1. FITS와 SunPy Map

FITS(Flexible Image Transport System)는 천문 관측 자료에 널리 사용되는 파일 형식이다. 하나의 FITS 파일에는 관측 영상뿐 아니라 관측 시각, 관측기기, 파장, 노출시간 및 좌표계 등의 메타데이터가 함께 저장된다.

SunPy에서는 `sunpy.map.Map()`을 사용하여 태양 영상 FITS 파일을 불러올 수 있다. 생성된 Map 객체는 영상 배열인 `data`, FITS 메타데이터인 `meta`, 관측 시각인 `date`, 관측 파장인 `wavelength` 등의 정보를 제공한다.

```python
import sunpy.map

aia_map = sunpy.map.Map("aia_file.fits")

print(aia_map.data.shape)
print(aia_map.date)
print(aia_map.wavelength)
print(aia_map.meta)
```

Map은 WCS(World Coordinate System) 정보를 포함하므로 픽셀 좌표를 태양 관측 좌표로 변환할 수 있다. 이번 실습에서는 helioprojective 좌표를 사용했으며, 좌표 단위는 주로 arcsec였다.

## 2. 태양 영상 시각화와 부분 영역 추출

SunPy Map은 Matplotlib과 연동하여 관측 좌표가 포함된 영상을 그릴 수 있다. `clip_interval`을 사용하면 극단적인 픽셀값의 영향을 줄여 영상 구조를 더 잘 보이게 할 수 있다.

```python
import matplotlib.pyplot as plt
import astropy.units as u

fig = plt.figure()
ax = fig.add_subplot(projection=aia_map)

aia_map.plot(
    axes=ax,
    clip_interval=(1, 99.9) * u.percent
)

aia_map.draw_limb(axes=ax)

plt.show()
```

`SkyCoord`로 경계 좌표를 지정하고 `submap()`을 적용하면 동일한 태양 좌표 범위를 추출할 수 있다.

```python
from astropy.coordinates import SkyCoord

bottom_left = SkyCoord(
    450 * u.arcsec,
    -750 * u.arcsec,
    frame=aia_map.coordinate_frame
)

top_right = SkyCoord(
    1100 * u.arcsec,
    -100 * u.arcsec,
    frame=aia_map.coordinate_frame
)

cropped_map = aia_map.submap(
    bottom_left,
    top_right=top_right
)
```

## 3. 태양 관측 자료 검색과 다운로드

SunPy의 `Fido`는 여러 태양물리 데이터 제공처를 통합하여 검색하는 인터페이스이다. 검색 조건은 `sunpy.net.attrs`를 사용해 관측 시각, 관측기기 및 파장 등으로 지정한다.

VSO를 통한 검색은 여러 제공처의 자료를 통합하여 찾기에 편리하지만, 다운로드 서버 상태에 따라 오류가 발생할 수 있다. 이번 실습에서는 VSO 다운로드가 실패하여 JSOC에 직접 export 요청을 제출하는 방식으로 AIA Level 1 자료를 내려받았다.

JSOC 검색 결과에서 `T_REC`는 일정한 cadence에 배치된 record time이다. 이것은 FITS 메타데이터에 기록된 실제 관측 시각과 몇 초 정도 다를 수 있다.

또한 JSOC 검색 결과를 slicing해도 선택한 행만 다운로드되는 것은 아니다. 특정 record 하나만 필요할 때는 검색 결과를 slicing하기보다 검색 시간 범위를 좁혀야 한다.

## 4. AIA 다파장 영상

SDO/AIA는 서로 다른 EUV 파장으로 태양 대기 구조를 관측한다. 각 채널은 특정 온도에만 반응하는 것이 아니라 넓은 온도 반응 함수를 가지므로, 대표 형성 온도는 근사적인 진단값으로 이해해야 한다.

| Channel | Main line | Representative temperature | Main structures |
|---|---|---:|---|
| AIA 171 Å | Fe IX | 약 0.6 MK | Quiet corona, coronal loops |
| AIA 193 Å | Fe XII | 약 1.3 MK | Hotter and diffuse corona |
| AIA 304 Å | He II | 약 0.05 MK | Chromosphere, transition region, prominence |

AIA 영상에 사용되는 색은 실제 가시광선 색이 아니라 파장을 구분하기 위한 false color이다.

SunPy는 FITS 메타데이터를 바탕으로 `sdoaia171`, `sdoaia193`, `sdoaia304` 등의 관례적인 colormap을 자동 적용한다.

## 5. ROI intensity 분석

AIA 171 Å 영상에서 동일한 크기의 active-region ROI와 quiet-Sun ROI를 정하고 평균, 중앙값, 표준편차 등의 기초 통계량을 계산했다.

분석 결과 active-region ROI는 quiet-Sun ROI보다 평균 intensity가 약 6.94배, 중앙값이 약 4.70배 높았다.

활동영역의 intensity profile에는 coronal loop와 밝은 국소 구조에 대응하는 큰 변동과 여러 peak가 나타났다. 반면 quiet-Sun profile은 상대적으로 낮고 균일했다.

활동영역에는 16383 DN에 도달한 포화 픽셀이 167개 있었으며, 이는 유효 픽셀의 약 0.595%였다. 포화 픽셀은 평균과 표준편차를 증가시킬 수 있으므로, 이 경우 중앙값이 두 영역의 전형적인 밝기를 비교하는 더 안정적인 통계량이다.

이번 분석을 통해 영상의 색과 밝기를 정성적으로 관찰하는 것에서 나아가, ROI 통계와 1차원 intensity profile을 이용해 활동영역과 quiet Sun의 차이를 정량적으로 비교했다.