# A Convolution–Deconvolution Framework for Low-Cost Spectrometry and 3D Imaging

**Author:** John Kirby  
**Date:** 08/06/2026  
**Repository:** spectral-deconvolution-engine  
**Status:** Conceptual Instrumentation & Computational Pipeline Architecture  

---

## Abstract

This framework presents an open-source, low-cost instrumentation design that bypasses traditional, expensive optical hardware ($15k+ lab units) by leveraging computational optics and telemetry. Operating under the principle that **"flawed images are not failures — they are compressed multidimensional measurements,"** the system implements a closed-loop **convolution–deconvolution** pipeline. It couples a dual-path monochrome sensor layout with active dwell-time modulation and multi-IMU hardware telemetry to extract high-resolution 3D depth and quantitative spectroscopy at ~80% of laboratory performance at <1% of the cost. The system embodies **computational self-awareness**, using its own telemetry-derived PSF as a corrective model.

---

## 1. Introduction
* **The "Streak-as-Data" Paradigm Shift:** Challenging the modern astronomical dogma that light streaks (such as satellite constellations or tracking drift) represent ruined data. By reframing motion and chromatic dispersion as multi-dimensional geometric encodings, this architecture turns every accidental or deliberate light trace into a simultaneous point source and high-resolution spectrum—unlocking multi-object spectroscopy without a single moving optical part.
* **The High Cost of Precision:** Traditional spectroscopy and hyperspectral imaging are restricted by expensive, mechanically complex instrumentation.
* **The Democratization of Science:** Applying the Pareto principle (80/20 rule) via open-source hardware, consumer-grade monochrome sensors, and computational correction.
* **Distinction from Heuristic Image Editing:** Establishing this system as a rigorous, radiometric physical inversion framework rather than a perceptual image-editing tool (e.g., raster filters). Unlike consumer software, the architecture preserves raw radiometric flux conservation, ingests hardware IMU telemetry, and performs quantitative spatial-spectral reconstruction.
* **The "Flaw-as-Feature" Paradigm:** Harnessing chromatic dispersion and motion tracking as intentional data-encoding dimensions rather than discarding them.
* **Computational Self-Awareness:** Replacing mechanical rigidity with a system that *knows its own convolution kernel* and corrects itself using telemetry-driven PSF estimation.
* **Scientific Lineage:** This work extends the lineage of *coded apertures* and *compressed sensing* into low-cost instrumentation.

---

## 2. Theoretical Foundations: The Convolutional Forward Model
* **Defining the "Crime" (The Physics of Optical Corruption):** Modeling how pristine incoming radiance is systematically corrupted before reaching digitization. The "crime" occurs across three distinct physical dimensions during the finite integration window (shutter open time $T$):
  * *Temporal Integration:* The camera shutter remains open over time $T$, acting as a temporal integrator that captures a continuous accumulation of light while motion occurs.
  * *Kinematic Displacement:* Relative motion between the scene and the sensor (whether from camera structural jitter, tracking drift, or deliberate spatial skewing) drags point sources of light across physical pixels, turning discrete point targets into continuous spatial trails.
  * *Wavelength-Dependent Refraction (Dispersion):* Optical elements bend varying frequencies of light at slightly different angles, forcing white-light point sources to physically fracture into continuous spectral dispersion vectors.
  * Together, these phenomena destroy the spatial-spectral integrity of the raw scene, transforming high-contrast point data into a degraded, overlapping convolution map.
* **The Forward Equation (Mathematical Modeling):** Formalizing image formation as a linear system affected by convolution and additive noise. In continuous spatial coordinates $(x,y)$, the optical degradation process is expressed as:
  $$I(x,y) = [S(x,y) * K(x,y)] + N(x,y)$$
  where:
  * $I(x,y)$ represents the final degraded, motion-smeared, and chromatically dispersed image recorded by the digital sensor.
  * $S(x,y)$ denotes the ideal, pristine source scene (the ground truth prior to optical corruption).
  * $K(x,y)$ is the composite system kernel (Point Spread Function / Line Spread Function) dictated by sensor kinematics, optical geometry, and spatial skewing.
  * $N(x,y)$ accounts for stochastic sensor noise, including photon shot noise, electronic read noise, and thermal dark current.
  * $*$ denotes the 2D spatial convolution operator.
  * In the discrete digital domain of the camera sensor, this translates to a pixel-grid operation where every individual pixel value in $I$ is a weighted linear superposition of surrounding source pixels distributed across space by $K$.
* **Defining the Point Spread Function (PSF):** Formally defining the PSF as the spatial impulse response of the optical and digital imaging system—describing mathematically how a theoretical, infinitely small point source of light is "spread" or blurred across adjacent pixels due to diffraction, lens aberrations, and sensor limitations.
* **The "Family of PSFs" (From Defect to Data):** *A perfect image gives you one PSF. A flawed image gives you a family of PSFs.* In traditional static imaging, a uniform PSF is treated as an optical defect to be minimized or removed to collapse data back into a singular point. In contrast, our framework leverages a structured, multi-dimensional *family of PSFs*. Intentional spatial motion (jitter/translation) and chromatic dispersion transform the PSF from a static blur blob into an information-rich geometric manifold that explicitly encodes directional motion vectors, temporal evolution, and continuous spectral gradients.
* **The Composite Kernel ($K$):** Mathematically defining the total system kernel as a structured superposition (or cascaded convolution) of two distinct physical domains:
  * *Spatial Convolution ($K_{\text{spatial}}$):* The kinematic component governing geometric distribution. It maps the temporal trajectory of camera translation, structural jitter, and user-guided spatial skewing across the 2D sensor grid.
  * *Spectral Convolution ($K_{\text{spectral}}$):* The optical component governing chromatic distribution. It maps radiant energy from a source across spatial coordinates as a continuous function of wavelength ($\lambda$).
  * Together, these phenomena fuse into a unified composite kernel $K(x,y)$, where every point in the image space is simultaneously subjected to geometric displacement and wavelength-dependent dispersion.
* **Kernel Separability:** Establishing the mathematical condition that allows the composite kernel to be factored into decoupled or sequentially independent components—specifically, treating spatial motion correction and spectral dispersion extraction as orthogonal or cascading operations. Under the constraint of high-frequency hardware telemetry and controlled dispersion geometry, kernel separability drastically reduces computational complexity, transforming an otherwise intractable multi-dimensional deconvolution problem into stable, sequential 1D and 2D inversions that cleanly feed our modular processing pipeline.
* **Discrete Vector-Gradient Inversion (Arithmetic-Based Reconstruction):** Unlike classical deconvolution frameworks relying on continuous integral equations or Fourier-domain division (which risk instability near zero-frequency components), our inverse pipeline operates entirely in the discrete spatial domain. Representing the motion path as a parametric curve $\gamma(t) = (x(t), y(t))$, reconstruction proceeds via spatial-temporal flux reassignment:
  $$S_{\text{est}}(x_0, y_0) = \sum_{t \in \gamma^{-1}(x_0, y_0)} I(x(t), y(t)) \cdot w(t)$$
  where $w(t)$ incorporates exposure-time normalization, logarithmic scaling, and finite PSF support. This formulation bypasses continuous spectral division, reducing inversion to stable coordinate remapping and additive/subtractive flux binning.

---

## 3. System Hardware Architecture
* **The Dual-Path Split-Sensor Design:** 
  * Utilizing a low-cost optical beam splitter to divide a single lens input across two parallel monochrome channels.
  * **Channel A (The Sharp Anchor):** Provides un-smeared structural boundaries and high-precision spatial coordinates ($X, Y$).
  * **Channel B (The Dispersive/Defocused Channel):** Houses the intentional chromatic dispersion profile for spectroscopy and depth-from-defocus calculations.
* **Cross-Channel Geometric Registration (Homography):** Computational mapping to mathematically align and register the Sharp Anchor layer over the Dispersive layer, compensating for mechanical tolerances.
* **Component Selection:** Consumer-grade CMOS/CCD sensors, 3D-printed modular housings, and open-source microcontrollers chosen to maximize performance per dollar.
* **Control Ecosystem:** Low-cost microcontrollers (e.g., ESP32) ensuring microsecond-precise hardware timing and shutter synchronization.

---

## 4. Active Dwell-Time Modulation (Hardware-Level HDR)
* **Active Kinematics & Slew Mechanics:** Dynamically modulating the speed of the dispersion mechanism during data collection.
* **Dwell-Time Engineering:** Speeding up over blindingly bright continuum peaks to prevent pixel saturation, and slowing down over weak absorption lines to elevate faint photons above the read-noise floor.
* **Single-Shot Dynamic Range Advantage:** Unlike exposure bracketing, which introduces motion-ghosting and alignment artifacts when targets shift between frames, dwell-time modulation performs a continuous single sweep. The velocity curve itself manages dynamic range, eliminating multi-frame registration errors entirely.
* **Pixel Dilution Avoidance:** Prevents faint spectral features from being buried under noise due to multi-frame averaging.

---

## 5. Multi-IMU Redundancy & Telemetry Logging
* **Passive Kinematics & High-Hz Tracking:** Bypassing OS smoothing filters by using independent, high-frequency external IMUs (1,000+ Hz) to log raw physical motion.
* **Redundant Sensor Fusion:** Cross-checking multi-IMU arrays rigidly locked to the optical block to isolate structural flex and jitter.
* **Structural Flex Mapping:** Telemetry reveals micro-scale bending or torsion in the optical block, enabling correction without rigid mounts.
* **Telemetry as a Deterministic PSF:** Using real-time error logs to replace expensive mechanical rigidity with self-aware computational reconstruction.

---

## 6. Computational Processing: The Vector-Gradient & Deconvolution Pipeline
* **Initial Wavelength Calibration:** Mapping pixel position to exact wavelengths ($\lambda$) using a known baseline reference source before processing targets.
* **Inverting the Forward Model:** Translating the forward convolution into a solvable inverse problem across both telemetry-driven (non-blind) and image-driven (blind) operational modes.
* **Vector-Gradient Field Analysis:** Bypassing classical edge-fitting transforms in favor of spatial-spectral vector-gradient analysis. By computing directional derivatives, magnitude, and orientation fields, the engine extracts motion vectors, chromatic spread, and stellar PSF signatures from data.
* **Module 1: User-Guided Geometry & Path Rectification:** An interactive interface where the operator maps complex translation vectors or non-linear squiggles, allowing the software to perform Inverse Spatial Coordinate Mapping and unwarp distorted pixel geometry along the motion path.
* **Module 2: Software-Guided Spectra & Automatic Isolation:** Employing spatial-spectral vector-gradient analysis to automatically sample intensity profiles, identify emission lines, and isolate spectral slices without relying on manual user drawing.
* **Local Baseline Continuum Calibration (Stray-Light Subtraction):** A background-sampling mechanism that records static environment baselines (e.g., sampling ambient wall or sky flux) and subtracts that background floor from the smeared streak data before execution, ensuring background integrity is maintained during target collapse.
* **Deterministic vs. Estimated Deconvolution:** Utilizing synchronized IMU/VCM telemetry logs (non-blind mode) or vector-gradient estimated kernels (blind mode) combined with perceptual log-scaling to execute Trajectory-Based Spatial Re-integration and collapse smeared energy back to its true source coordinates.
* **Channel Realignment & Reconstruction:** Using homography and PSF-derived geometric correction to overlay anchor channels, reconstruct faint absorption lines, and translate monochrome intensity logs into quantitative 1D spectra, paralleling NASA narrowband imaging pipelines.

---

## 7. Cost-Benefit and Scientific Utility Analysis
* **Performance Benchmarking:** Evaluating the output of a sub-$500 DIY dual-path rig against mid-tier prosumer ($4k–$18k) and laboratory-grade ($15k+) systems.
* **The Sensitivity-Resolution Trade-off:** Analyzing photon dispersion limits relative to the sensor's read-noise floor.
* **Citizen-Science Deployment:** Low-cost, mass-producible architecture enables distributed environmental monitoring, educational spectroscopy, and global-scale scientific participation.

---

## 8. Conclusion and Future Horizons
* **Summary of Contributions:** Proving that computational telemetry and closed-loop modeling can substitute for high-cost physical engineering.
* **Future Horizons:** 
  * Scaling toward solid-state Optical Phased Arrays (OPAs), real-time spectral video, and distributed citizen-science deployment networks.
  * *Future applications may also extend into microscopy, archival astronomy, and forensic imaging, where flawed exposures contain recoverable multidimensional structure.*

---

## References

*[Bibliography and instrumentation schemas to be added]*
