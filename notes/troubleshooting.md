# SunPy and JSOC Troubleshooting Notes

This document records issues encountered during the SunPy warm-up and Project 1, together with the solutions that worked.

## 1. `NoMatchError` in a JSOC Query

### Symptom

`Fido.search()` raised:

```text
NoMatchError: This query was not understood by any clients.
```

### Cause and solution

A generic instrument query was combined with JSOC-specific attributes. Use a JSOC series explicitly and provide the notification email through `a.jsoc.Notify`.

```python
from getpass import getpass
from sunpy.net import Fido, attrs as a

jsoc_email = getpass("Enter your registered JSOC email: ")

result = Fido.search(
    a.Time(start_time, end_time),
    a.jsoc.Series("aia.lev1_euv_12s"),
    a.jsoc.Wavelength(171 * u.angstrom),
    a.jsoc.Sample(5 * u.min),
    a.jsoc.Segment("image"),
    a.jsoc.Notify(jsoc_email),
)
```

Using `getpass()` avoids exposing an email address in a public notebook.

## 2. Fixed-cadence Sampling Missed an Observation

### Symptom

One channel returned fewer files, even though nearby observations existed.

### Cause

JSOC sampling uses a channel-specific time grid. If no record falls on the requested grid point, the corresponding interval may appear to be missing. AIA channels also have different timestamp phases.

### Solution

Search a narrow interval around the missing target without `a.jsoc.Sample`, inspect all returned `T_REC` values, and select the nearest observation. Always report the actual observation time rather than the nominal target time.

## 3. An Extra or Duplicate Observation Appeared

Cadence queries may include an endpoint or a manually recovered file may overlap an existing sample. Build the final sequence by assigning one nearest observation to each target time, and remove duplicate paths or timestamps before analysis.

## 4. Image and `spikes` Files Were Both Downloaded

If both segments are exported, every observation can produce a large image FITS file and a small spikes FITS file. Restrict the query to:

```python
a.jsoc.Segment("image")
```

When loading local data, also use a filename pattern such as:

```python
sorted(directory.glob("*.image_lev1.fits"))
```

## 5. Partial Download or `ContentLengthError`

### Symptom

One file failed with `ConnectionResetError` or an incomplete response payload.

### Solution

Retry the original `Results` object returned by `Fido.fetch()`:

```python
retried_files = Fido.fetch(
    downloaded_files,
    progress=False,
    max_conn=1,
)

print(len(retried_files.errors))
```

`Fido.fetch()` can retry only the failed entries stored in `.errors`. Confirm completion by counting local `*.image_lev1.fits` files and, when needed, opening them with Astropy or SunPy.

## 6. Excessive Progress Output and Notebook Lag

Use:

```python
Fido.fetch(result, progress=False, max_conn=1)
```

This hides transfer progress bars and limits parallel connections. JSOC and DRMS status logs may still appear because they are logger messages rather than progress bars.

If JupyterLab reports a server connection error while a download is running, verify the files from a terminal before restarting the notebook. The browser connection can fail even after the Python process has written the files successfully.

## 7. `KeyError: "Keyword 'DATE-OBS' not found"`

### Cause

`fits.getval(file, "DATE-OBS")` reads the primary header by default, but the relevant time keyword may be stored in another HDU or under a different AIA keyword.

### Preferred solution

Use the time interpreted by SunPy:

```python
observation_time = sunpy.map.Map(file).date
```

This is more robust than assuming a specific FITS keyword and extension.

## 8. `sunpy.timeseries` Missing Dependencies

### Symptom

Importing `sunpy.timeseries` failed because packages such as `h5netcdf` or `h5py` were absent.

### Solution

Install the TimeSeries extras in the active environment, then restart the kernel:

```bash
python -m pip install "sunpy[timeseries]"
```

For the GOES netCDF files used here, `h5netcdf` and `h5py` are required.

## 9. `IProgress not found` Warning

This warning affects the notebook-style progress display, not the data query. Either keep `progress=False` or install/update `ipywidgets` and restart Jupyter.

## 10. JSOC Export Stayed Pending

`status=1` or `status=2` means the server is preparing the export. Large or repeated requests can take time. Keep requests small, avoid launching duplicate exports, and reuse already downloaded local files.

## 11. VSO or Provider Search Failure

Archive services can be temporarily unavailable. Prefer the client appropriate for the data source, keep the time range narrow, and retry later. For AIA Level 1 data, the JSOC client gave more reproducible control over wavelength, cadence, and segment selection.

## 12. Variables Disappeared After a Kernel Restart

A notebook output is not the same as live Python state. After restarting, run cells from the beginning in dependency order. Keep downloads optional and let analysis cells discover existing local files so the notebook remains rerunnable.

## 13. Relative-path Errors

Relative paths are evaluated from the notebook's working directory. Check it with:

```python
from pathlib import Path
print(Path.cwd())
```

When the notebook is run from `notebooks/`, the project directories can be referenced as:

```python
data_dir = Path("../data")
figure_dir = Path("../figures")
```

## 14. Bright White Pixels Versus Saturation

White pixels in a rendered map may result from the colormap or clipping and do not by themselves prove detector saturation. Check the numerical data and relevant quality information. Saturated pixels can bias means and integrated intensities, so report their count or fraction and compare robust statistics such as the median.

## 15. Reproducibility Checklist

- Keep credentials and email addresses out of committed cells and outputs.
- Record the query interval, series, wavelength, segment, and cadence.
- Save and report actual observation times.
- Count downloaded image files and inspect `.errors`.
- Restart the kernel and run the analysis cells in order.
- Keep large FITS files out of Git.
- Document manual recovery, deletion, or replacement of observations.

