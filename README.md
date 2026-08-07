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
  * *Kinematic Displacement:* Relative motion between the scene and the sensor (whether from camera structural jitter, tracking drift, or deliberate spatial skewing) drags point sources of light across physical pixels, turning discrete point targets into continuous spatial trails defined by a time-parameterized kinematic trajectory curve $\gamma(t)$.
  * *Wavelength-Dependent Refraction (Dispersion):* Optical elements bend varying frequencies of light at slightly different angles, forcing white-light point sources to physically fracture into continuous spectral dispersion vectors.
  * Together, these phenomena destroy the spatial-spectral integrity of the raw scene, transforming high-contrast point data into a degraded, overlapping convolution map.
* **The Forward Equation (Mathematical Modeling):** Formalizing image formation as a linear system affected by convolution and additive noise. In continuous spatial coordinates $(x,y)$, the optical degradation process is expressed as:
  $$I(x,y) = [S(x,y) * K(x,y)] + N(x,y)$$
  where:
  * $I(x,y)$ represents the final degraded, motion-smeared, and chromatically dispersed image recorded by the digital sensor.
  * $S(x,y)$ denotes the ideal, pristine source scene (the ground truth prior to optical corruption).
  * $K(x,y)$ is the composite system kernel (Point Spread Function / Line Spread Function) parameterized by sensor kinematics, optical geometry, and spatial skewing along the motion path.
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

## 3. System Hardware Architecture (Cumulative Modular Tiers)
The physical architecture is structured as a cumulative, modular hierarchy, allowing users to scale complexity and capability based on their resources and institutional backing—ranging from software-only processing to advanced, dual-path research systems:

* **Tier 1: The Software-Only Tier (Beginner / Classroom Level)**
  * *Required Components:* A standard consumer camera (smartphone, webcam, or existing DSLR) + a computer running the vector-gradient deconvolution engine.
  * *Operational Scope:* Zero hardware modifications required. Processes pre-existing legacy footage, motion-blurred terrestrial video, or simulated synthetic data using pure blind and semi-blind software-driven kernel estimation.
* **Tier 2: The Instrumented Single-Axis Rig (Intermediate / Robotics Club Level)**
  * *Includes all Tier 1 capabilities, plus:*
  * *Hardware Additions:* A low-cost microcontroller (ESP32/Arduino) mounted via a 3D-printed hot-shoe or lens bracket, paired with an external high-frequency IMU module and a small stepper motor for controlled translation/slew.
  * *Operational Scope:* Enables active dwell-time modulation and deterministic telemetry logging. For makers using existing DSLRs, the hot-shoe mounted IMU logs real-time micro-jitters directly to a companion file (that may require high-frequency filtering to isolate structural shutter-shock transients during actuation), feeding an exact motion trace for PSF reconstruction to the software without altering internal optics.
* **Tier 3: The Dual-Path Research Rig (Advanced / Makerspace & Institutional Level)**
  * *Includes all Tier 1 & Tier 2 capabilities, plus:*
  * *Hardware Additions & Architectural Paths:* An optical non-polarizing beam splitter cube and a modular 3D-printed optical chassis, deployable via two implementation paths depending on resources:
    * *Path A (Single-Sensor Split-Field / Maker-Space Optimized):* Projects Channel A (anchor) and Channel B (dispersive) onto two distinct zones of a single high-resolution, global-shutter monochrome CMOS sensor (no IR-cut filter). This natively eliminates clock drift, optical path length discrepancies, USB bus latency, and multi-sensor synchronization hurdles.
    * *Path B (Dual-Sensor / Institutional Research Grade):* Utilizes dual synchronized global-shutter monochrome CMOS sensors paired with a redundant multi-IMU array (2–3 synchronized inertial sensors utilizing basic sensor fusion or master clock alignment to isolate structural flex and vibration).
  * *Operational Scope:* The full dual-path architecture. Splits a single optical wavefront into parallel channels—Channel A serving as a sharp spatial anchor, and Channel B capturing dispersed/defocused data simultaneously for high-precision, non-blind astrophysical and spatial-spectral reconstruction.
 
### Telescope Mount Telemetry Integration (Optional Motion Source)
Motorized GoTo or tracking telescope mounts can serve as an alternative motion-guidance source for astronomical sweeps. When commanded through ASCOM or INDI, the mount can execute smooth, programmable sky-scan trajectories and provide a deterministic macro-motion path $\gamma(t)$ derived from RA/DEC updates or encoder-level telemetry.

* **Smooth Macro-Motion Advantage:** Telescope sweeps are mechanically stable and typically smoother than handheld or IMU-driven rigs. This makes them well-suited for controlled dwell-time modulation, spectral streaking, and long-exposure motion-based reconstruction where clean, low-frequency motion is preferred.

* **Logging Integration:** ASCOM/INDI motor traces can be timestamped and aligned with the camera’s exposure metadata, allowing the motion path $\gamma(t)$ to be synchronized with the imaging sequence. This provides a deterministic sweep profile without requiring external inertial hardware.

This integration path is optional and intended for users already operating motorized mounts; it complements Tier 2 and Tier 3 by offering a low-noise, programmable macro-motion source for sky-sweep experiments.

---

## 4. Active Dwell-Time Modulation (Hardware-Level HDR)

* **Active Kinematics and Slew Mechanics:**  
  The dispersion mechanism (IMU-guided rig, stepper translation stage, or telescope mount) varies its instantaneous velocity $v(t)$ during the sweep.  
  Dwell time at spatial or spectral coordinate $x$ scales inversely with velocity:  
  $$\tau(x) \propto \frac{\Delta x}{v(x)}.$$  
  Sweep velocity becomes the direct control variable for photon integration.

* **Dwell-Time Engineering:**  
  The accumulated signal at position $x$ is  
  $$N_{\text{sig}}(x) = \Phi(x)\,\tau(x) = \Phi(x)\,\frac{\Delta x}{v(x)},$$  
  where $\Phi(x)$ is the local photon flux.  
  Two physical bounds define the feasible velocity band:
  * *Saturation limit:*  
    $$v(x) \ge \frac{\Phi(x)\Delta x}{N_{\text{FWC}}}$$  
    to avoid exceeding full-well capacity.
  * *Read-noise limit:*  
    $$v(x) \le \frac{\Phi(x)\Delta x}{k\,\sigma_{\text{read}}}$$  
    to keep faint features above detection threshold $k$.  
  The velocity curve must remain between these limits, turning qualitative guidance into a concrete design equation.

* **Effective Dynamic Range Gain:**  
  The dynamic-range extension achievable through modulation alone is approximately  
  $$DR_{\text{eff}} \approx DR_{\text{sensor}} \times \frac{v_{\max}}{v_{\min}}.$$  
  This ties HDR performance directly to the mechanical velocity ratio the hardware can sustain.

* **Control Mode (Open-Loop or Closed-Loop):**  
  Dwell-time modulation requires a velocity profile $v(x)$ that can be generated either in open-loop form (precomputed from a flux estimate) or closed-loop form (adjusted in real time from sensor feedback).  
  The architecture has not yet selected a control mode, and the hardware implications are deferred to Section 8.

* **Single-Shot Dynamic Range Advantage:**  
  Traditional bracketed HDR requires multiple frames, introducing ghosting, registration error, and PSF inconsistency.  
  Dwell-time modulation performs HDR within a single continuous sweep, producing one coherent motion trace and one unified PSF model.

* **Pixel Dilution Avoidance and Photon Statistics:**  
  Multi-frame averaging dilutes faint features by mixing them with noise from brighter frames.  
  Dwell-time modulation shapes photon arrival rate directly during acquisition.  
  Because photon noise follows Poisson statistics,  
  $$\text{SNR} \propto \sqrt{N_{\text{sig}}},$$  
  increasing dwell time on faint absorption lines by a factor of $n$ improves SNR by $\sqrt{n}$.

* **Link to Section 2 Reconstruction Weight $w(t)$:**  
  The weighting term $w(t)$ used in the reconstruction model originates physically from dwell-time modulation:  
  $$w(t) \propto \tau(t) = \frac{\Delta x}{v(t)}.$$  
  This closes the loop between mechanical control and mathematical reconstruction.

* **Integration with Tier 2 and Tier 3 Hardware:**  
  Tier 2 rigs vary sweep velocity deterministically and log motion traces for reconstruction.  
  Tier 3 dual-path systems synchronize dwell-time modulation across anchor and dispersive channels.  
  Telescope-mount sweeps provide smooth macro-motion profiles that can be timestamp-aligned with exposure metadata to drive modulation without external inertial hardware.  
  In all cases, the velocity and dwell-time curve become hardware-level metadata passed to the reconstruction engine.


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

### 8. Conclusion and Future Horizons
* **Summary of Contributions:** This work demonstrates that computational telemetry, vector-gradient field analysis, and closed-loop spatial-spectral modeling can successfully substitute for high-cost physical engineering. By treating motion blur and chromatic dispersion as "compressed multidimensional measurements" rather than data failures, we establish a robust, low-cost instrument architecture.
* **Computational Simplicity (From Calculus to Arithmetic):** By replacing heavy, continuous frequency-domain integral equations with discrete spatial vector mapping and flux reassignment, the algorithm bypasses complex calculus. It functions fundamentally through high-speed spatial binning and coordinate addition/subtraction, opening the door for real-time edge-computing execution on lightweight microcontrollers.
* **The Space-Sweep Vision (Wide-Field Surveys):** The ultimate evolution of this architecture points toward wide-field space-based survey cameras. By embracing intentional, continuous sensor sweeps, a single exposure can simultaneously capture high-precision astrometry and multi-object slitless spectroscopy for thousands of stars, asteroids, and transients concurrently—eliminating the need for expensive mechanical tracking mounts or physical spectrograph slits.
* **Quantum-Stabilized Motion Truth (QGPS Integration):** Looking toward next-generation aerospace systems, the future integration of Quantum GPS (QGPS) via cold-atom interferometry addresses the ultimate bottleneck of inertial tracking: sensor drift. By providing absolute, drift-free motion telemetry for $\gamma(t)$, quantum-stabilized telemetry eliminates inversion uncertainty, locking the reconstruction into a fully deterministic, singularity-resistant solution.
* **Solid-State & Networked Scaling:** Scaling toward solid-state Optical Phased Arrays (OPAs) to eliminate moving parts entirely, enabling real-time spectral video streams, and deploying distributed citizen-science sensor networks.
* **Open-Loop vs Closed-Loop Control Mode:** A key unresolved design question is whether dwell-time modulation should operate in open-loop form (precomputed velocity profiles derived from flux estimates) or closed-loop form (real-time velocity correction based on sensor feedback). This decision requires quantitative analysis of sensor readout latency, mechanical response time, and IMU bandwidth, and will shape the hardware requirements for future Tier 2 and Tier 3 prototypes.
* **Broader Field Horizons:** Future applications may also extend into microscopy, archival astronomy, and forensic imaging, where flawed exposures contain recoverable multidimensional structure.

---

## References
* Cho, T. S., Paris, S., Horn, B. K. P., & Freeman, W. T. (2011). [Blur Kernel Estimation Using the Radon Transform](https://people.csail.mit.edu/sparis/publi/2011/cvpr_radon/Cho_11_Blur_Kernel_Estimation.pdf). *Proceedings of the IEEE Conference on Computer Vision and Pattern Recognition (CVPR)*.

*[Bibliography and instrumentation schemas to be added]*
