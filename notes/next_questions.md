# Questions for the Next Stage

## 1. AIA 데이터 전처리

- JSOC에서 받은 AIA Level 1 자료에 `aia_prep()` 또는 `aiapy.calibrate`를 적용해야 하는 이유는 무엇인가?
- Level 1과 Level 1.5 자료는 좌표 정렬, pixel scale, 회전 보정 측면에서 어떻게 다른가?
- 서로 다른 시각과 파장의 영상을 정량적으로 비교하려면 어떤 보정이 필요한가?
- AIA intensity를 비교할 때 노출시간으로 나누어 DN/s 단위로 변환해야 하는가?

## 2. 시계열 자료 정렬

- 여러 시각의 AIA 영상에서 동일한 태양 영역을 어떻게 계속 추적할 수 있는가?
- 태양의 differential rotation은 짧은 시계열과 긴 시계열 분석에 각각 어느 정도 영향을 미치는가?
- `differential_rotate()` 또는 `propagate_with_solar_surface()`는 어떤 경우에 사용해야 하는가?
- 관측 시각이 정확히 일치하지 않는 다파장 영상을 어느 정도 시간 차이까지 같은 시각의 자료로 간주할 수 있는가?

## 3. ROI와 intensity 측정

- 고정된 helioprojective 좌표의 ROI와 태양 자전을 따라가는 ROI 중 어느 방법이 적절한가?
- 활동영역의 크기나 형태가 시간에 따라 변할 때 ROI를 어떻게 정해야 하는가?
- 평균, 중앙값, 총합 중 어떤 통계량이 활동영역의 시간 변화를 가장 잘 나타내는가?
- 포화 픽셀이나 결측 픽셀이 포함된 시계열은 어떻게 처리해야 하는가?
- 배경 코로나의 변화를 제거하기 위해 quiet-Sun ROI를 어떻게 활용할 수 있는가?

## 4. AIA 채널의 물리적 해석

- AIA 채널의 대표 형성 온도와 실제 temperature response function은 어떻게 다른가?
- 하나의 AIA 채널에 여러 방출선이 기여할 때 intensity 증가를 온도 상승으로 해석할 수 있는가?
- 171 Å, 193 Å, 304 Å에서 동일한 구조가 서로 다르게 보이는 이유를 어떻게 정량화할 수 있는가?
- 플레어 또는 분출 과정에서 각 파장 채널의 peak 시각 차이는 어떤 물리적 의미를 가질 수 있는가?

## 5. 다음 분석 목표

- 활동영역의 AIA intensity light curve를 만들려면 적절한 cadence와 전체 관측 시간은 어느 정도인가?
- AIA light curve와 GOES X-ray flux를 어떻게 시간적으로 정렬하고 비교할 수 있는가?
- 플레어 전후의 intensity 증가율, peak time, decay time을 어떤 방식으로 측정할 수 있는가?
- 단순한 ROI 분석에서 더 참신한 연구 질문으로 발전시키려면 어떤 추가 자료나 분석법이 필요한가?
- 이후 SDO 분석을 PSP 또는 태양 전파 관측 자료와 어떻게 연결할 수 있는가?
