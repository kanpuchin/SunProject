# Next Questions and Analysis Roadmap

This note records the questions resolved in the SunPy warm-up and Project 1, together with the most useful next steps.

## Questions Resolved in Project 1

### How should observations from different AIA channels be time-matched?

Fixed-cadence JSOC sampling does not guarantee identical timestamps across channels. Project 1 therefore selected the locally available observation nearest to each target time using the observation time stored in the SunPy map. Small channel-to-channel differences of several seconds are acceptable for this preliminary five-minute-cadence comparison, but the actual timestamps must be reported.

### What should be done when a target sample is missing?

Search a short interval around the target without cadence sampling, then select the nearest observation. This recovered a suitable 171 Å image near 06:30 UTC. Duplicate observations should be removed so that every target time contributes at most one image per channel.

### How should AIA intensities be compared?

The mean data number in a fixed helioprojective ROI was divided by the exposure time and then normalized to a pre-flare baseline. This makes temporal changes easier to compare, but it does not constitute a full physical calibration between passbands.

### How do the AIA peaks compare with the GOES soft X-ray peak?

In the five-minute samples, 171 and 304 Å peaked about 16 minutes before the 06:41 UTC GOES peak, 211 Å peaked about 6 minutes before it, and 193 Å peaked within about 1 minute of it. These offsets describe the sampled ROI light curves and should not be interpreted as exact physical delays.

## Immediate Validation Tasks

- Repeat the analysis with a finer cadence around 06:15–06:45 UTC.
- Process the AIA maps to a consistent Level 1.5 geometry using `aiapy.calibrate` before pixel-by-pixel comparison.
- Inspect data-quality keywords and quantify saturated or invalid pixels at every time step.
- Test several ROI sizes and positions, including a background-subtracted measurement.
- Compare mean, median, and integrated intensity to determine sensitivity to bright or saturated pixels.
- Estimate timing uncertainty from both cadence and channel-to-channel timestamp offsets.

## Physical Questions

- Why does the 304 Å ROI show the largest relative enhancement?
- Which structures dominate the early 171 and 304 Å peaks?
- Does the later 193 Å maximum reflect hotter plasma, a different structure, or the broad temperature response of the channel?
- How much of each curve is controlled by plasma evolution versus changing morphology inside the fixed ROI?
- Would a differential emission measure analysis change the temperature interpretation?

## Possible Project 2 Directions

### Active-region evolution with AIA and HMI

- Track the active region while accounting for solar rotation.
- Compare EUV intensity evolution with HMI line-of-sight magnetic field.
- Measure magnetic-flux evolution before and after a selected flare.
- Investigate whether morphological changes correspond to magnetic restructuring.

### Higher-cadence flare timing

- Download a shorter interval at substantially higher cadence.
- Compare AIA derivatives or channel ratios with GOES XRS flux.
- Measure rise, peak, and decay times with uncertainties.
- Separate impulsive and gradual-phase behavior where possible.

### Multi-instrument extension

- Investigate available solar-radio observations for the event.
- Examine whether an appropriate Parker Solar Probe interval offers a meaningful connection.
- Define a scientific question before combining remote-sensing and in-situ data.

## Questions for Research Guidance

- Is ROI photometry an appropriate starting point for the intended scientific question?
- Which calibration and uncertainty checks are essential before treating the timing differences quantitatively?
- Would AIA–HMI active-region evolution or a higher-cadence flare study be the stronger next project?
- Which result from Project 1 is most worth developing into a research-grade analysis?

