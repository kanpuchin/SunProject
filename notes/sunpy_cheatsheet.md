# SunPy Project Cheatsheet

Reusable commands and patterns from the SunPy warm-up and Project 1.

## 1. Imports

```python
from pathlib import Path
from getpass import getpass

import astropy.units as u
import matplotlib.pyplot as plt
import numpy as np
import pandas as pd
import sunpy.map

from astropy.coordinates import SkyCoord
from astropy.time import Time
from sunpy.net import Fido, attrs as a
```

## 2. Project Paths

```python
data_dir = Path("../data")
figure_dir = Path("../figures")
figure_dir.mkdir(parents=True, exist_ok=True)
```

## 3. Find Local AIA Image Files

```python
files = sorted(
    Path("../data/aia/20110607/171").glob("*.image_lev1.fits")
)
print(len(files))
```

The explicit pattern excludes JSOC `spikes` segments.

## 4. Create and Inspect a SunPy Map

```python
aia_map = sunpy.map.Map(files[0])

print(aia_map.date)
print(aia_map.wavelength)
print(aia_map.exposure_time)
print(aia_map.coordinate_frame)
print(aia_map.data.shape)
```

## 5. Plot a Map

```python
fig = plt.figure(figsize=(8, 8))
ax = fig.add_subplot(projection=aia_map)

aia_map.plot(
    axes=ax,
    clip_interval=(1, 99.9) * u.percent,
)
aia_map.draw_limb(axes=ax)
ax.grid(alpha=0.3)

plt.show()
```

## 6. Define and Extract a Helioprojective ROI

```python
bottom_left = SkyCoord(
    600 * u.arcsec,
    -430 * u.arcsec,
    frame=aia_map.coordinate_frame,
)
top_right = SkyCoord(
    820 * u.arcsec,
    -260 * u.arcsec,
    frame=aia_map.coordinate_frame,
)

roi_map = aia_map.submap(
    bottom_left,
    top_right=top_right,
)
```

## 7. Draw the ROI on a Full-disk Map

```python
fig = plt.figure(figsize=(8, 8))
ax = fig.add_subplot(projection=aia_map)
aia_map.plot(axes=ax)
aia_map.draw_quadrangle(
    bottom_left,
    top_right=top_right,
    axes=ax,
    edgecolor="cyan",
    linewidth=2,
)
```

## 8. Search HEK for a Flare

```python
hek_result = Fido.search(
    a.Time("2011-06-07 05:30", "2011-06-07 08:00"),
    a.hek.EventType("FL"),
)
```

Inspect start, peak, end, GOES class, NOAA region number, and helioprojective position before choosing the event.

## 9. Search JSOC for Sampled AIA Images

```python
jsoc_email = getpass("Enter your registered JSOC email: ")

result = Fido.search(
    a.Time(observation_start, observation_end),
    a.jsoc.Series("aia.lev1_euv_12s"),
    a.jsoc.Wavelength(171 * u.angstrom),
    a.jsoc.Sample(5 * u.min),
    a.jsoc.Segment("image"),
    a.jsoc.Notify(jsoc_email),
)
```

Do not commit a literal email address to a public notebook.

## 10. Search Around a Missing Target Time

Remove `a.jsoc.Sample` and query a short interval:

```python
nearby = Fido.search(
    a.Time("2011-06-07 06:29", "2011-06-07 06:31"),
    a.jsoc.Series("aia.lev1_euv_12s"),
    a.jsoc.Wavelength(171 * u.angstrom),
    a.jsoc.Segment("image"),
    a.jsoc.Notify(jsoc_email),
)
```

Then select the observation nearest to the target.

## 11. Download Quietly and Retry Failures

```python
downloaded = Fido.fetch(
    result,
    path=str(download_dir / "{file}"),
    progress=False,
    max_conn=1,
)

if downloaded.errors:
    downloaded = Fido.fetch(
        downloaded,
        progress=False,
        max_conn=1,
    )
```

## 12. Select the Nearest Local Observation

```python
def find_closest_observation(files, target_time):
    target = Time(target_time)
    candidates = []

    for file in files:
        observation_time = sunpy.map.Map(file).date
        difference = abs((observation_time - target).to_value(u.s))
        candidates.append((difference, file, observation_time))

    _, closest_file, closest_time = min(candidates, key=lambda item: item[0])
    return closest_file, closest_time
```

Use `Map.date` instead of assuming that `DATE-OBS` is in the primary FITS header.

## 13. Exposure-normalized ROI Intensity

```python
def exposure_normalized_mean(aia_map, bottom_left, top_right):
    roi = aia_map.submap(bottom_left, top_right=top_right)
    return np.nanmean(roi.data) / aia_map.exposure_time.to_value(u.s)
```

For each channel, normalize the time series to a defined pre-flare baseline:

```python
baseline = np.mean(intensities[:2])
relative_intensity = np.asarray(intensities) / baseline
```

Record precisely which samples define the baseline.

## 14. Basic ROI Statistics

```python
data = roi_map.data.astype(float)

statistics = {
    "mean": np.nanmean(data),
    "median": np.nanmedian(data),
    "std": np.nanstd(data),
    "min": np.nanmin(data),
    "max": np.nanmax(data),
}
```

## 15. Saturation Check

```python
saturation_level = aia_map.meta.get("SATVAL", 16383)
saturated = roi_map.data >= saturation_level

print("Saturated pixels:", np.count_nonzero(saturated))
print("Fraction:", np.mean(saturated))
```

Treat the threshold as instrument- and processing-dependent and inspect data-quality metadata as well.

## 16. Find a Sampled Peak

```python
peak_index = int(np.nanargmax(relative_intensity))
peak_time = observation_times[peak_index]
peak_value = relative_intensity[peak_index]
```

The result is the maximum among sampled observations, not necessarily the exact physical peak.

## 17. Download and Read GOES XRS Data

```python
from sunpy import timeseries as ts

goes_result = Fido.search(
    a.Time("2011-06-07 05:30", "2011-06-07 07:30"),
    a.Instrument("XRS"),
)

goes_avg_result = goes_result[0][goes_result[0]["Resolution"] == "avg1m"]
goes_files = Fido.fetch(
    goes_avg_result,
    path=str(goes_directory / "{file}"),
    progress=False,
)

goes_ts = ts.TimeSeries(goes_files, source="XRS")
goes_df = goes_ts.to_dataframe()
```

For the 1–8 Å band, use `xrsb`. Filter or inspect `xrsb_quality` before quantitative work.

## 18. Restrict GOES Data to an Interval

```python
goes_interval = goes_df.loc[
    "2011-06-07 05:30":"2011-06-07 07:30"
]
good = goes_interval[goes_interval["xrsb_quality"] == 0]
```

## 19. Compare AIA and GOES on Twin Axes

```python
fig, ax_aia = plt.subplots(figsize=(12, 7))
ax_goes = ax_aia.twinx()

for wavelength, curve in light_curves.items():
    ax_aia.plot(
        curve["times"],
        curve["relative_intensity"],
        marker="o",
        label=f"AIA {wavelength} Å",
    )

ax_goes.semilogy(
    good.index,
    good["xrsb"],
    color="black",
    label="GOES-15 XRS-B (1–8 Å)",
)
```

Keep the two y-axis units explicit: relative AIA intensity on the left and physical GOES flux on the right.

## 20. Save a Figure

```python
fig.savefig(
    figure_dir / "figure_name.png",
    dpi=200,
    bbox_inches="tight",
)
```

Save the figure after all ROI, annotation, and layout changes have been applied.

