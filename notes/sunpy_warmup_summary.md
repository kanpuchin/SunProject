# SunPy Study and Project 1 Summary

This note summarizes the main concepts learned during the SunPy warm-up and the methods and results developed in Project 1.

## 1. FITS Files and SunPy Maps

FITS (Flexible Image Transport System) is a standard file format for astronomical observations. A FITS file can contain both an image and metadata such as the observation time, instrument, wavelength, exposure time, and coordinate-system information.

SunPy loads solar FITS images as `Map` objects:

```python
import sunpy.map

aia_map = sunpy.map.Map("aia_file.fits")

print(aia_map.data.shape)
print(aia_map.date)
print(aia_map.wavelength)
print(aia_map.exposure_time)
print(aia_map.coordinate_frame)
```

A map contains WCS (World Coordinate System) information, which connects image pixels to solar coordinates. This project primarily used helioprojective Cartesian coordinates in arcseconds.

## 2. Visualization and Spatial Selection

SunPy maps work with Matplotlib and WCSAxes:

```python
import astropy.units as u
import matplotlib.pyplot as plt

fig = plt.figure()
ax = fig.add_subplot(projection=aia_map)

aia_map.plot(
    axes=ax,
    clip_interval=(1, 99.9) * u.percent,
)
aia_map.draw_limb(axes=ax)

plt.show()
```

Displayed white regions are not automatically saturated pixels. They can also result from the plotting normalization or clipping range.

A region can be defined in helioprojective coordinates and extracted with `submap()`:

```python
from astropy.coordinates import SkyCoord

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

## 3. Searching for and Downloading Solar Data

`Fido` provides a common interface to several solar-physics data services. The warm-up used VSO and JSOC searches, while Project 1 used JSOC directly for AIA data.

The following lessons were important:

- A successful VSO search does not guarantee a successful download.
- A JSOC query should explicitly specify the series, wavelength, image segment, and registered email address.
- `T_REC` is a nominal JSOC record time and may differ from the physical observation time.
- `Map.date`, derived from the FITS metadata, was used for temporal comparisons.
- Slicing a JSOC response is not a reliable way to export only selected records. A narrow search interval should be used instead.
- A failed file transfer can be retried using the returned download-results object without submitting a new export.

## 4. AIA Multi-wavelength Observations

SDO/AIA observes the solar atmosphere through multiple EUV passbands. Each channel has a broad and sometimes multi-thermal temperature response, so its representative temperature should be treated as an approximate diagnostic rather than an exact plasma temperature.

| Channel | Major contribution | Representative temperature | Commonly observed structures |
|---|---|---:|---|
| AIA 171 Å | Fe IX | approximately 0.6 MK | Quiet corona and coronal loops |
| AIA 193 Å | Fe XII, with hot flare contributions | approximately 1.3 MK in the quiet corona | Coronal structures and flare emission |
| AIA 211 Å | Fe XIV | approximately 2 MK | Hotter coronal structures |
| AIA 304 Å | He II | approximately 0.05 MK | Chromosphere, transition region, and prominences |

The colors used for AIA images are conventional false colors, not visible-light colors. SunPy applies wavelength-specific colormaps such as `sdoaia171`, `sdoaia193`, `sdoaia211`, and `sdoaia304`.

## 5. Warm-up ROI Analysis

The warm-up compared equal-sized active-region and quiet-Sun ROIs in an AIA 171 Å image.

Results:

- Mean intensity ratio, active region divided by quiet Sun: **6.94**
- Median intensity ratio, active region divided by quiet Sun: **4.70**
- Pixels at the adopted saturation threshold in the active-region ROI: **167**
- Corresponding pixel fraction: **0.595%**

The active-region intensity profile contained large variations associated with coronal loops and compact bright structures. The quiet-Sun profile was fainter and more uniform. Because extreme pixels strongly affect the mean and standard deviation, the median was also used to compare typical ROI intensities.

## 6. Project 1: Flare-event Selection

Project 1 extended the single-image warm-up into a time-series analysis of the 2011 June 7 M2.5 flare in NOAA active region 11226.

| Property | Value |
|---|---|
| Flare start | 06:16 UTC |
| GOES peak | 06:41 UTC |
| Flare end | 06:59 UTC |
| Approximate location | Solar-X = 715.5 arcsec, Solar-Y = −339.8 arcsec |

The event was selected from HEK records because its GOES class, active-region number, timing, and approximate helioprojective location were available.

## 7. AIA Time-series Selection

Project 1 used AIA 171, 193, 211, and 304 Å images from approximately 06:05 to 07:10 UTC with a target interval of five minutes.

Fixed-time sampling did not always return one image for every target time. Two cases were encountered:

- The 304 Å observations were offset from the initial sampling grid, so the search interval was shifted to match their record times.
- The 171 Å search omitted the 06:30 target even though nearby observations existed. An image at 06:30:14 was found with an unsampled local search.

For each target time, the nearest available observation was selected using `Map.date`. The final dataset contained 14 observations for each channel. Small inter-channel time offsets were acceptable for the five-minute trend analysis but cannot be used to measure sub-minute physical delays.

## 8. Exposure-normalized ROI Light Curves

A fixed photometric ROI covered Solar-X = 600–820 arcsec and Solar-Y = −430 to −260 arcsec.

For each image, the mean ROI intensity was divided by the exposure time. Each channel was then divided by its own pre-flare baseline, defined as the mean of the first two observations near 06:05 and 06:10 UTC.

The sampled maxima were:

| Channel | Sampled peak time | Peak intensity relative to baseline |
|---:|:---:|---:|
| 171 Å | 06:25:12 UTC | 1.89 |
| 193 Å | 06:40:10 UTC | 1.99 |
| 211 Å | 06:35:03 UTC | 1.63 |
| 304 Å | 06:25:32 UTC | 5.25 |

The 171 and 304 Å sampled maxima occurred in the same five-minute interval. Their 20-second difference is below the temporal resolution of the analysis and does not establish a peak order.

## 9. Comparison with GOES Soft X-ray Flux

One-minute averaged GOES-15 XRS-B observations in the 1–8 Å band were compared with the AIA ROI light curves.

- The 171 Å maximum occurred approximately **15.8 minutes before** the GOES maximum.
- The 304 Å maximum occurred approximately **15.5 minutes before** the GOES maximum.
- The 211 Å maximum occurred approximately **6.0 minutes before** the GOES maximum.
- The 193 Å maximum occurred approximately **0.8 minutes before** the GOES maximum and was effectively coincident with it at the AIA sampling resolution.

The 171 and 304 Å sampled maxima occurred during the rapid rise of the soft X-ray flux. The 211 Å maximum occurred as the X-ray flux approached its broad maximum, and the 193 Å maximum was most closely associated with the GOES maximum.

## 10. Interpretation and Limitations

The channel-dependent light curves indicate that different EUV-emitting structures evolved during different phases of the soft X-ray event. However, this result does not by itself establish a simple heating or cooling sequence.

Important limitations include:

- AIA passbands have broad and multi-thermal temperature responses.
- The AIA curves were normalized to separate baselines, so their amplitudes are not direct measurements of relative emitted energy.
- GOES/XRS measures full-disk irradiance, while the AIA curves represent a selected active-region ROI.
- The five-minute AIA sampling interval cannot resolve short-lived variations or sub-minute delays.
- The measured evolution depends on the ROI definition.
- Bright flare-core pixels may be saturated and require a dedicated check.
- The analysis used AIA Level 1 data. More precise spatial comparisons require pointing correction, registration, and calibration to Level 1.5.

## 11. Current Outcome

The warm-up and Project 1 established an end-to-end workflow for selecting a solar event, downloading multi-instrument observations, matching images in time, extracting a common ROI, constructing exposure-normalized light curves, and comparing EUV evolution with GOES soft X-ray flux.

Project 1 is complete as a preliminary analysis. The next stage will focus on calibration, robustness tests, and the transition toward AIA–HMI and Parker Solar Probe projects.
