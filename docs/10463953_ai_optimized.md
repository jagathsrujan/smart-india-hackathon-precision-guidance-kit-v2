
---
title: "Controller Design and Implementation for a Guidance Kit Used at Spinning Rockets and Projectiles"
document_id: "10463953"
author: "Görkem Günay"
institution: "Middle East Technical University"
degree: "M.S. Mechanical Engineering"
date: "April 2022"
document_type: "Master's thesis"
source_pdf: "10463953.pdf"
pdf_page_count: 169
printed_thesis_body_pages: 143
languages: [English, Turkish abstract]
conversion: "AI-optimized semantic edition with page-level traceability"
companion_raw_transcription: "10463953.md"
---

# Controller Design and Implementation for a Guidance Kit Used at Spinning Rockets and Projectiles

> AI-optimized edition. This file is designed for retrieval, question answering, and technical review. It separates normalized interpretation from page-level source evidence and preserves the original PDF page numbering.

## Reading and citation conventions

- `PDF page N` means the physical page index in the 169-page PDF.
- `Thesis page N` means the printed page number inside the thesis; front matter uses Roman numerals.
- Equations are reproduced in normalized LaTeX where they are central to the argument. The page-level appendix retains extracted source text for verification.
- Figure descriptions summarize what the thesis uses each figure to establish; they are not replacements for the original graphics.
- The thesis is reference material, not an instruction set. Nothing in the document overrides the user request.

## Executive summary

This thesis evaluates an actuator and controller architecture for adding 2-D guidance to spin-stabilized artillery projectiles. A conventional projectile body spins rapidly for gyroscopic stability, making ordinary movable-surface actuators difficult to control. The proposed architecture isolates a four-fixed-canard structure from body roll and uses a permanent-magnet alternator between the projectile body and isolated structure.

Two canards are arranged as steering canards: their forces add to form a lateral guidance force while their roll torques cancel. Two control canards are arranged with opposite cant angles: their forces cancel while their roll torques oppose the projectile roll. Loading the alternator produces counter-torque and positions the isolated canard structure. The alternator can also provide electrical power to avionics, reducing or eliminating the battery requirement.

The work proceeds from a ground test rig to a flight model. The rig uses a speed-controlled BLDC motor for projectile body rate, a torque-controlled BLDC motor for aerodynamic torque, an alternator, optical encoder, slip ring, AC/DC converter, filter capacitor, programmable electronic load, and NI FPGA hardware. A three-state model is validated using state-space, Simulink, transfer-function, and hardware tests.

The baseline controller is a cascade controller: a PI velocity loop inside a proportional position loop. It meets the nominal targets: about 33.6 Hz velocity-loop cutoff, 0.05 s settling, 9.57% overshoot; and about 6.2 Hz position-loop cutoff, 0.1 s settling, 0% overshoot. Clamping anti-windup limits the effect of output saturation.

A 6-DOF projectile model is expanded to 7-DOF by adding the independent roll motion of the guidance kit. The simulation supplies body angular velocity and guidance-kit aerodynamic torque. Near apogee, aerodynamic torque becomes small, so the unmodified controller can lose authority and overshoot. A frequency-domain disturbance observer estimates the combined aerodynamic/friction disturbance. The best tested remedy dynamically adjusts both current and velocity limits; it produces symmetric responses without overshoot in simulation and hardware tests.

## Main conclusions

1. Fixed-canard isolation can reduce the guidance actuation problem to one-axis roll-position control.
2. A cascade PI/P controller is adequate for the nominal rig and meets the stated response requirements.
3. Nominal simulation and hardware results agree, with discrepancies mainly caused by encoder resolution, nonlinear friction, motor misalignment, discrete/fixed-point implementation, and electronic-load dynamics.
4. A constant controller is not robust to changing flight torque and body rate, especially near apogee.
5. The combined adaptive saturation strategy is the strongest result: it preserves control authority, avoids overshoot, and keeps positive/negative responses symmetric.
6. The feasibility result is promising but not a complete guidance demonstration: no navigation/guidance law or closed-loop trajectory correction is implemented.

## End-to-end system model

```text
Projectile body roll p_body(t)
        |
        v
Speed motor / alternator stator ------------------+
                                                   | relative electrical/mechanical plant
Guidance-kit rotor + fixed canards <--------------+
        | p_gk, theta_gk
        v
Optical encoder -> FPGA controller -> electronic load current I_load
                                      ^
                                      | disturbance estimate from p_gk and I_load
Aerodynamic torque T_aero and bearing friction T_f
```

## Core architecture and torque logic

### Canard force decomposition

- Steering canards at 0 and 180 degrees have the same cant angle. Their aerodynamic forces point in the same direction, so the net steering force is approximately twice the force of one steering canard; their roll torques cancel.
- Control canards at 90 and -90 degrees have opposite cant angles. Their aerodynamic forces cancel; their roll torques add to form `T_aero` in the direction opposite projectile roll.
- Alternator loading creates `T_alt` in the projectile-roll direction. The isolated canard structure is positioned by balancing alternator torque, aerodynamic torque, and bearing friction.

### Torque balance used throughout the thesis

$$
T_{+x}=T_{\mathrm{alt}}+T_f-T_{\mathrm{aero}}
$$

$$
T_{-x}=-T_{\mathrm{alt}}-T_f+T_{\mathrm{aero}}
$$

The thesis notes that control in the `-X` direction requires bearing friction to be smaller than available aerodynamic torque. This constraint is the reason changing flight conditions require controller saturation adaptation.

## Notation glossary

| Symbol | Meaning | Units / role |
|---|---|---|
| `p_body` | Projectile/rocket body angular velocity about X | rad/s |
| `p_gk` | Guidance-kit angular velocity about X | rad/s |
| `J_body` | Projectile-body roll inertia | kg m^2 |
| `J_gk` | Rotating guidance-kit inertia | kg m^2 |
| `b` | Viscous friction coefficient | N m/(rad/s) |
| `T_aero` | Aerodynamic torque on the fixed-canard structure | N m |
| `T_f` | Bearing/friction torque | N m |
| `T_alt` | Alternator electromagnetic torque | N m |
| `T_ad` | Aerodynamic damping torque due to kit rate | N m |
| `K_t` | Alternator torque constant | N m/A |
| `K_v` | Alternator voltage/speed constant | V/(rad/s) |
| `I_a` | Alternator phase current | A |
| `I_load` | Programmable electronic-load current | A |
| `L_a` | Alternator phase-to-phase inductance | H |
| `R_a` | Alternator phase-to-phase resistance | ohm |
| `C_f` | Filter capacitance | F |
| `V_Cf` | Filter-capacitor voltage | V |
| `alpha` | Incidence angle | rad |
| `beta` | Sideslip angle | rad |
| `rho` | Air density | kg/m^3 |
| `V` | Projectile speed magnitude | m/s |
| `S` | Reference area | m^2 |
| `d` | Projectile reference diameter | m |
| `p,q,r` | Body angular rates about x, y, z | rad/s |
| `u,v,w` | Body linear velocity components | m/s |
| `L,M,N` | Moments about x, y, z | N m |
| `X,Y,Z` | Forces along x, y, z | N |
| `theta_gk` | Guidance-kit roll angle | rad |
| `DOF` | Degree of freedom | model order |
| `G_vc` | Velocity-controller transfer function | inner loop |
| `G_olp` | Open-loop position transfer function | outer loop |
| `Q(s)` | Low-pass filter in disturbance observer | causal filter |


## Key quantitative facts

| Area | Value | Interpretation |
|---|---|---|
| Projectile | 155 mm | Reference artillery projectile |
| Launch loads | 15000 g axial; 5000 g radial | Guidance electronics/actuator environment |
| Typical body roll | 10000-20000 rpm; about 270 Hz in flight discussion | Reason conventional actuators need isolation |
| Rig controller rate | 1 kHz | FPGA control/update loop |
| Encoder resolution | 2500 CPT; about 24 rpm velocity resolution at 1 ms | Measurement granularity |
| Nominal velocity loop | 33.6 Hz cutoff; 0.05 s settling; 9.57% overshoot | PI inner-loop result |
| Nominal position loop | 6.2 Hz cutoff; 0.1 s settling; 0% overshoot | P outer-loop result |
| Maximum rig alternator load | 8 A in Chapter 6 analysis | Sets maximum modeled alternator torque |
| Maximum alternator torque | 5.76 N m using K_t about 0.72 N m/A | 8 A capability |
| Measured friction | 0.1 N m at the tested relative rate | Minimum disturbance floor |
| Selected test body rate | 800 rpm; 83.7 rad/s maximum | Scaled from flight rate for rig limits |
| Combined adaptive solution | No overshoot and symmetric response in tests | Preferred controller modification |


## Nominal mathematical model

The test-system state and input vectors are:

$$
x = \begin{bmatrix}p_{gk} & V_{Cf} & I_{La}\end{bmatrix}^{T}, \qquad
u = \begin{bmatrix}p_{body} & T_{aero} & I_{load}\end{bmatrix}^{T}
$$

$$\dot{x}=Ax+Bu$$

$$
A=\begin{bmatrix}
-b/J_{gk} & 0 & K_t/J_{gk} \\
0 & 0 & 1/C_f \\
-K_v/L_a & -1/L_a & -R_a/L_a
\end{bmatrix},
\qquad
B=\begin{bmatrix}
b/J_{gk} & 1/J_{gk} & 0 \\
0 & 0 & -1/C_f \\
K_v/L_a & 0 & 0
\end{bmatrix}.
$$

Equivalent state derivatives from Appendix A are:

$$\dot{p}_{gk}=\frac{b}{J_{gk}}p_{body}-\frac{b}{J_{gk}}p_{gk}+\frac{K_t}{J_{gk}}I_{La}+\frac{T_{aero}}{J_{gk}}$$

$$\dot{V}_{Cf}=\frac{I_{La}-I_{load}}{C_f},\qquad
\dot{I}_{La}=\frac{-K_vp_{gk}+K_vp_{body}-V_{Cf}-R_aI_{La}}{L_a}.$$

### Transfer functions

All three input-to-output transfer functions share a third-order denominator:

$$D(s)=\sigma_3s^3+\sigma_2s^2+\sigma_1s+\sigma_0.$$

For load current to guidance-kit rate:

$$G_{I_{load}\to p_{gk}}(s)=\frac{\beta_0}{D(s)},\qquad \beta_0=\frac{K_t}{L_aC_fJ_{gk}}.$$

The source defines `sigma_3=1`, `sigma_0=b/(L_a C_f J_gk)`, `sigma_1=(K_v K_t C_f+bR_a C_f+J_gk)/(L_a C_f J_gk)`, and `sigma_2=(bL_a C_f+R_a C_f J_gk)/(L_a C_f J_gk)`.

For aerodynamic torque to guidance-kit rate:

$$G_{T_{aero}\to p_{gk}}(s)=\frac{\beta_2s^2+\beta_1s+\beta_0}{D(s)},$$

where `beta_2=1/J_gk`, `beta_1=R_a/(J_gk L_a)`, and `beta_0=1/(J_gk L_a C_f)`.

For body angular velocity to guidance-kit rate, the numerator is `beta_2 s^2+beta_1 s+beta_0` with `beta_2=b/J_gk`, `beta_1=(bR_a+K_tK_v)/(L_aJ_gk)`, and `beta_0=b/(L_aC_fJ_gk)`.

## Controller design and validation

### Cascade structure

- Outer loop: position error `theta_gk,r - theta_gk` -> proportional position controller -> velocity reference.
- Inner loop: velocity error -> PI velocity controller -> electronic-load current command.
- Disturbance inputs: `T_aero` and `p_body` enter the plant and are rejected primarily by the inner loop.
- Saturation handling: clamping anti-windup prevents integrator accumulation when the current command is limited.
- Minimum-effort logic wraps the angular error so the canard structure rotates along the shorter path.

### Design requirements versus achieved results

| Loop | Requirement | Achieved |
|---|---|---|
| Position | Cutoff >=4 Hz; settling <=0.1 s; overshoot <=5%; zero step error | 6.2 Hz; 0.1 s; 0%; infinite static error constant |
| Velocity | Cutoff >=20 Hz; 5% settling <=0.05 s; overshoot <=25%; zero step error | 33.6 Hz; 0.05 s; 9.57%; infinite static error constant |


### Controller laws

Velocity PI controller:

$$G_{vc}(s)=K_{pv}+\frac{K_{iv}}{s}=0.29+\frac{7.82}{s}.$$

Position controller: proportional gain `K_pp=29.7`. The position loop has an integrator from the velocity-to-position plant, so no additional position integrator is required for zero step error.

### Hardware test interpretation

- Step test: simulation and hardware track the reference; stationary oscillations are attributed to encoder measurement error and nonlinear friction.
- Chirp test: 0.01-7.5 Hz over 15 s, amplitude `pi/8`, `pi` offset; measured cutoff is about 6.23 Hz, close to the 6.2 Hz design.
- Velocity/output differences are larger than position differences because the inner loop compensates for plant uncertainty; major uncertainties are Coulomb friction and torque/speed-motor centerline misalignment.

## Flight-dynamics model

### Coordinate frames

- Body frame rolls with the projectile and is computationally expensive at high roll rate.
- Body Fixed Plane frame imposes zero roll angle/rate and uses `p_F=-r tan(theta)`; its Y-axis remains parallel to the Earth, which is useful for guidance.
- Body non-spinning frame sets `p_F=0` and is simpler but is not the frame selected for the main simulation.

### Modeling assumptions

- Flat Earth; Earth curvature omitted.
- Gravity variation with altitude omitted.
- Aerodynamic coefficients treated as Mach-dependent only for the feasibility study; alpha/beta variation is omitted.
- Coefficients are sparse at Mach 0.9, 0.95, 1.1, 1.5, and 2.0, producing sudden nonphysical plot changes.

### 6-DOF and 7-DOF equations

The non-inertial translational and rotational equations are:

$$
\mathbf{F}=m\left(\left.\frac{d\mathbf{V}}{dt}\right|_{BFP}+\boldsymbol{\omega}_{BFP}\times\mathbf{V}\right),
\qquad
\mathbf{M}=\mathbf{I}\left.\frac{d\boldsymbol{\omega}}{dt}\right|_{BFP}+\boldsymbol{\omega}_{BFP}\times(\mathbf{I}\boldsymbol{\omega}).
$$

For 7-DOF, the angular-rate state is extended to `[p_gk, p_body, q, r]^T`; the inertia matrix separates `I_xgk` and `I_xbody`, and the moment vector receives the total guidance-kit roll torque `L_gk-7DOF`.

### Aerodynamic force and torque model

$$\alpha=\tan^{-1}\left(\frac{w}{u}\right),\qquad \beta=\tan^{-1}\left(\frac{v}{\sqrt{u^2+w^2}}\right).$$

$$\mathbf{F}_{7DOF}=\mathbf{F}_{6DOF}+\mathbf{F}_c,\qquad \mathbf{M}_{7DOF}=\mathbf{M}_{6DOF}+\mathbf{M}_c.$$

$$T_{aero}=\frac{1}{2}\rho V^2 S l_{gk} C_{rc},\qquad T_{ad}=\frac{1}{2}\rho S d^2 V p_{gk} C_{rdgk}.$$

$$L_{gk-7DOF}=T_{aero}+T_{ad}.$$

### Flight-model outputs used by the controller study

- Aerodynamic torque is largest near launch and end of flight and approaches zero near apogee.
- Body angular velocity decreases throughout the selected trajectory.
- The free-rotating guidance kit produces little change in Earth-fixed projectile position/velocity in the ballistic 7-DOF comparison, but it produces relevant control forces, moments, and roll torque.

## Adaptive controller modification

### Why the nominal controller fails

Near apogee, `T_aero` is too small to brake the isolated canard structure in one direction. The position response can make full rotations, which introduces nonlinear wrap-around and large overshoot. The rig also scales body rate down, changing the ratio between body rate and guidance-kit rate relative to the real projectile.

### Disturbance observer

The thesis selects a frequency-domain observer because the available hardware cannot measure DC-load voltage and current simultaneously. The inverse plant is filtered to keep the observer causal:

$$\hat{G}(s)=\frac{588.4}{s+1.04},\qquad Q(s)=\frac{1}{0.5s+1}.$$

The estimated disturbance current tracks the real disturbance current well because aerodynamic and friction torques do not contain high-frequency components.

### Three solutions

| Solution | Mechanism | Result |
|---|---|---|
| 1. Dynamic current limit | Set velocity-controller saturation from estimated disturbance current, approximately twice the observed current for symmetry. | Control remains available but overshoot remains when current is low and velocity limit is high. |
| 2. Dynamic velocity limit | Set position-controller velocity saturation proportional to observed disturbance current; 5.2 rad/s at 0.2 A gives scale factor 26 rad/(A s). | No overshoot in simulation, but positive/negative responses are asymmetric. |
| 3. Combined limits | Apply both dynamic current and dynamic velocity limits. | No overshoot, symmetric response, and alternator limit not reached; best result. |


## Hardware implementation of adaptive controllers

The disturbance observer and filter are discretized at 1 kHz with First-Order Hold:

$$G_{observer}(z)=\frac{0.003397z-0.003394}{z-0.998},\qquad Q(z)=\frac{0.0009993z+0.0009987}{z-0.998}.$$

Hardware results show:

- Unmodified controller: larger loss of control and overshoot than simulation because the programmable load cannot instantly provide demanded current and because of fixed-point, analog, and discrete effects.
- Dynamic current limit: close to simulation when the alternator is not saturated; observer error grows at high torque.
- Dynamic velocity limit: still affected by relative-speed-dependent alternator loading; present relative speed could be used for an additional saturation.
- Combined solution: best measured response, no overshoot, symmetric behavior, and alternator capacity not reached.

## Data tables

### Table 2.1 - 155 mm projectile launch and final rates

| Powder charge | Initial velocity (m/s) | Initial angular velocity (Hz) | Final angular velocity (Hz) |
|---|---|---|---|
| 3M | 546.4 | 176.3 | 128.2 |
| 4M | 676.7 | 218.3 | 158.8 |
| 5M | 812.4 | 262.1 | 190.6 |
| 6M | 943.5 | 304.4 | 221.3 |


### Table 3.1 - Open-circuit validation

| p_body (rad/s) | V_Cf test (V) | V_Cf simulation (V) | K_v test (rad/s per V) |
|---|---|---|---|
| 10.5 | 7.8 | 7.92 | 0.74 |
| 20.9 | 16.0 | 15.7 | 0.76 |
| 31.4 | 24.6 | 23.6 | 0.78 |
| 41.9 | 33.3 | 31.4 | 0.79 |
| 52.4 | 41.9 | 39.3 | 0.80 |
| 62.8 | 50.4 | 47.1 | 0.80 |
| 73.3 | 58.9 | 55.0 | 0.80 |


### Table 3.2 - Closed-circuit validation

| p_body (rad/s) | V_Cf test (V) | V_Cf simulation (V) | T_gk (N m) | I_load (A) | K_t test (N m/A) |
|---|---|---|---|---|---|
| 20.9 | 12.0 | 12.6 | 0.37 | 0.5 | 0.73 |
| 20.9 | 9.2 | 9.5 | 0.72 | 1.0 | 0.72 |
| 20.9 | 6.7 | 6.4 | 1.03 | 1.5 | 0.69 |
| 41.9 | 28.3 | 28.3 | 0.39 | 0.5 | 0.77 |
| 41.9 | 25.2 | 25.2 | 0.74 | 1.0 | 0.74 |
| 41.9 | 22.2 | 22.2 | 1.07 | 1.5 | 0.71 |
| 41.9 | 19.6 | 19.0 | 1.44 | 2.0 | 0.72 |
| 41.9 | 16.6 | 15.9 | 1.78 | 2.5 | 0.71 |
| 62.8 | 44.6 | 44.0 | 0.37 | 0.5 | 0.75 |
| 62.8 | 41.2 | 40.9 | 0.73 | 1.0 | 0.73 |
| 62.8 | 38.0 | 37.8 | 1.08 | 1.5 | 0.72 |
| 62.8 | 34.8 | 34.7 | 1.42 | 2.0 | 0.71 |
| 62.8 | 31.8 | 31.6 | 1.75 | 2.5 | 0.70 |
| 83.8 | 60.4 | 59.7 | 0.37 | 0.5 | 0.73 |
| 83.8 | 56.7 | 56.6 | 0.72 | 1.0 | 0.72 |
| 83.8 | 53.4 | 53.5 | 1.06 | 1.5 | 0.71 |
| 83.8 | 50.2 | 50.4 | 1.42 | 2.0 | 0.71 |
| 83.8 | 46.5 | 47.5 | 1.75 | 2.5 | 0.70 |


### Key hardware specifications

| Component | Specification |
|---|---|
| Speed motor | Kollmorgen AKM63M; 480 VAC; 1.23 N m/A; 4.95 kW; 4500 rpm; 10.5 N m; 24 kg cm^2 rotor inertia |
| Torque motor | Kollmorgen AKM-33H; 240 VAC; 0.51 N m/A; 1.31 kW; 5500 rpm; 2.27 N m; 0.85 kg cm^2 rotor inertia |
| Encoder | FENAC optical incremental with index; 2500 CPT |
| Alternator | Volt Defence; 0.716 N m/A; 0.749 V/(rad/s); 7 pole pairs; 6.2 ohm; 8.8 mH; 1.2 kW |
| Slip ring | 12 circuits; 5000 rpm maximum; 5 A maximum |
| Filter capacitor | TDK; 50 microfarad; 800 V maximum |
| AC/DC converter | Vishay VS-36MT140; 25 A continuous DC; 1400 V RRM |
| Power supply | Aim TTI CPX 400D; 20 A; 60 V; 420 W |
| Programmable DC load | Aim TTI LD400P; 80 A; 80 V; 400 W continuous; 600 W short term |
| NI modules | NI-9063 controller; NI-9205 analog input; NI-9264 analog output; NI-9411 encoder digital input |


## Figures and visual evidence index

Figures are preserved in the source PDF. The following index explains their role for agents that cannot inspect the graphics directly:

| Figure group | PDF pages | What it establishes |
|---|---|---|
| Figures 1.1-1.8 | 28-32 | Compare drag-brake, movable-canard, fixed-canard, prior control structures, test rigs, and dual-spin projectile models. |
| Figures 2.1-2.5 | 35-39 | Show projectile axes, guidance-kit construction, rotor/stator isolation, force/torque decomposition, and torque directions. |
| Figures 2.6-2.14 | 40-53 | Show rig schematic/photos, actuator cross-sections, hardware/software interfaces, encoder evaluation, and FPGA controller. |
| Figures 3.1-3.9 | 55-65 | Show physical model, linear graph/tree, Simulink blocks, state comparison, free-body diagram, and transfer-function validation. |
| Figures 4.1-4.24 | 70-92 | Show cascade structure, root loci, step/Bode/disturbance responses, controller blocks, step tests, and chirp tests. |
| Figures 5.1-5.11 | 93-105 | Show torques, reference frames, 6-DOF/7-DOF Simulink results, control forces/moments, aerodynamic torque and damping. |
| Figures 6.1-6.14 | 109-119 | Show flight torque/body-rate profiles, nominal-controller failure, observer match, and three adaptive-limit simulations. |
| Figures 7.1-7.8 | 122-127 | Show unmodified and adaptive-controller hardware/simulation comparisons. |
| Figures B.1-B.4 | 141-146 | Show electrical circuit and free-body diagrams used to derive transfer functions. |
| Figures F.1-F.3 | 162-163 | Compare full/reduced plant and show filter Bode/step response. |


## Limitations, ambiguities, and source-quality notes

- The PDF has 169 physical pages but the abstract says 143 pages; the difference is front matter, references, appendices, and blank pages.
- The flight model omits Earth curvature, gravity variation with altitude, alpha/beta-dependent aerodynamic coefficients, and a complete guidance strategy.
- Aerodynamic coefficient data are sparse at a few Mach values, which creates sudden nonphysical changes in torque plots.
- The ground rig applies simulated aerodynamic torque directly but scales projectile body angular velocity to stay within speed-motor/electronic-load limits.
- Hardware discrepancies are linked to nonlinear Coulomb friction, mechanical misalignment, continuous/discrete effects, fixed-point arithmetic, analog I/O, and electronic-load dynamics.
- Some original equation/table subscripts are visually ambiguous in text extraction; use the source PDF for final symbol verification. The raw companion transcription retains the extracted form.
- Appendix E calls the modified cross-product matrix “Figure E.4” even though it is presented as an equation-like matrix; this appears to be a source-labeling issue.

## Page-level evidence map

The map below is deliberately one entry per physical PDF page so an agent can cite or retrieve evidence without guessing the printed-page offset.

### PDF page 1 | thesis page n/a
<!-- source: pdf_page=1; thesis_page=n/a -->
- **Section:** Front matter
- **Focus:** CONTROLLER DESIGN AND IMPLEMENTATION FOR A GUIDANCE KIT

### PDF page 2 | thesis page n/a
<!-- source: pdf_page=2; thesis_page=n/a -->
- **Section:** Front matter
- **Focus:** Blank page.

### PDF page 3 | thesis page n/a
<!-- source: pdf_page=3; thesis_page=n/a -->
- **Section:** Front matter
- **Focus:** CONTROLLER DESIGN AND IMPLEMENTATION FOR A GUIDANCE

### PDF page 4 | thesis page iv
<!-- source: pdf_page=4; thesis_page=iv -->
- **Section:** Front matter
- **Focus:** I hereby declare that all information in this document has been obtained and

### PDF page 5 | thesis page v
<!-- source: pdf_page=5; thesis_page=v -->
- **Section:** Front matter
- **Focus:** ABSTRACT CONTROLLER DESIGN AND IMPLEMENTATION FOR A GUIDANCE

### PDF page 6 | thesis page vi
<!-- source: pdf_page=6; thesis_page=vi -->
- **Section:** Front matter
- **Focus:** of an alternator. Since the actuation problem is reduced to one-axis position control,

### PDF page 7 | thesis page vii
<!-- source: pdf_page=7; thesis_page=vii -->
- **Section:** Front matter
- **Focus:** ÖZ DÖNEN ROKET VE MÜH˙IMMATLARDA KULLANILAN B˙IR GÜDÜM

### PDF page 8 | thesis page viii
<!-- source: pdf_page=8; thesis_page=viii -->
- **Section:** Front matter
- **Focus:** alternatörün pozisyon kontrolüdür. Temel tahrik problemi tek eksende pozisyon kontrolüne indirgendi˘ginden dolayı gerekli elektronik sistemler ve eyleyiciler basitle¸ stirilmi¸ stir. Ayrıca alternatörden elde edilen gücü batarya yerine kullanmak mümkündür.

### PDF page 9 | thesis page ix
<!-- source: pdf_page=9; thesis_page=ix -->
- **Section:** Front matter
- **Focus:** If you want to find the secrets of the universe, think in terms of energy, frequency

### PDF page 10 | thesis page x
<!-- source: pdf_page=10; thesis_page=x -->
- **Section:** Front matter
- **Focus:** ACKNOWLEDGMENTS Firstly, I want to give my deepest thanks to my supervisor Prof. Dr. Raif Tuna

### PDF page 11 | thesis page xi
<!-- source: pdf_page=11; thesis_page=xi -->
- **Section:** Front matter
- **Focus:** TABLE OF CONTENTS 1 INTRODUCTION . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 1

### PDF page 12 | thesis page xii
<!-- source: pdf_page=12; thesis_page=xii -->
- **Section:** Front matter
- **Focus:** Speed Motor . . . . . . . . . . . . . . . . . . . . . . . . . . . . 17

### PDF page 13 | thesis page xiii
<!-- source: pdf_page=13; thesis_page=xiii -->
- **Section:** Front matter
- **Focus:** 3.5.2 Comparison of the Closed-Circuit Test Results . . . . . . . . . 40

### PDF page 14 | thesis page xiv
<!-- source: pdf_page=14; thesis_page=xiv -->
- **Section:** Front matter
- **Focus:** REFERENCES . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 107 6.4 Possible Solution Approaches . . . . . . . . . . . . . . . . . . . . . 87

### PDF page 15 | thesis page xv
<!-- source: pdf_page=15; thesis_page=xv -->
- **Section:** Front matter
- **Focus:** G . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 139

### PDF page 16 | thesis page xvi
<!-- source: pdf_page=16; thesis_page=xvi -->
- **Section:** Front matter
- **Focus:** LIST OF TABLES Table 2.1 Axial and Angular Velocities of 155 mm Spin-Stabilized Projectile . 9 Table 2.2 Mechanical Parts of the Actuator Part of Test Set-up . . . . . . . . . 17

### PDF page 17 | thesis page xvii
<!-- source: pdf_page=17; thesis_page=xvii -->
- **Section:** Front matter
- **Focus:** Table 4.1 Requirements of the Position Controller . . . . . . . . . . . . . . . 45 Table 4.2 Requirements of the Velocity Controller . . . . . . . . . . . . . . . 45

### PDF page 18 | thesis page xviii
<!-- source: pdf_page=18; thesis_page=xviii -->
- **Section:** Front matter
- **Focus:** LIST OF FIGURES Figure 1.1 An Example Design of 1-D Control Approach by Using Drag Figure 1.2 An Example Design Which Uses Movable Aerodynamic Surfaces . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 3 Brake . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 2

### PDF page 19 | thesis page xix
<!-- source: pdf_page=19; thesis_page=xix -->
- **Section:** Front matter
- **Focus:** Figure 2.8 Micro-controller Part . . . . . . . . . . . . . . . . . . . . . . . 16 Figure 2.9 AC/DC Converter and Filter Capacitor . . . . . . . . . . . . . . 16 Simulink Model . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 35

### PDF page 20 | thesis page xx
<!-- source: pdf_page=20; thesis_page=xx -->
- **Section:** Front matter
- **Focus:** Figure 4.5 Bode Plot of Velocity Loop . . . . . . . . . . . . . . . . . . . . 51 Figure 4.6 Error Due to Step Disturbance at Taero . . . . . . . . . . . . . . 52 View . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 60

### PDF page 21 | thesis page xxi
<!-- source: pdf_page=21; thesis_page=xxi -->
- **Section:** Front matter
- **Focus:** Figure 5.1 Torques on the Guidance Kit . . . . . . . . . . . . . . . . . . . 67 Figure 5.2 Flat Earth Inertial Reference Frame . . . . . . . . . . . . . . . . 69

### PDF page 22 | thesis page xxii
<!-- source: pdf_page=22; thesis_page=xxii -->
- **Section:** Front matter
- **Focus:** Figure 6.11 Velocity Controller Result for Dynamic Velocity Limit - Zoomed Figure 6.12 Input and Output Positions for Combined Solution . . . . . . . . 92 View . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 91

### PDF page 23 | thesis page xxiii
<!-- source: pdf_page=23; thesis_page=xxiii -->
- **Section:** Front matter
- **Focus:** Figure F.1 Comparison of Real and Reduced Transfer Functions . . . . . . 136 Figure F.2 Bode Plot of the Filter . . . . . . . . . . . . . . . . . . . . . . . 137

### PDF page 24 | thesis page xxiv
<!-- source: pdf_page=24; thesis_page=xxiv -->
- **Section:** Front matter
- **Focus:** LIST OF ABBREVIATIONS pbody: Angular velocity of the projectile or rocket body

### PDF page 25 | thesis page xxv
<!-- source: pdf_page=25; thesis_page=xxv -->
- **Section:** Front matter
- **Focus:** X, Y , Z: Forces in x, y, z direction

### PDF page 26 | thesis page xxvi
<!-- source: pdf_page=26; thesis_page=xxvi -->
- **Section:** Front matter
- **Focus:** F: Total aerodynamic force for 6-DOF modeling

### PDF page 27 | thesis page 1
<!-- source: pdf_page=27; thesis_page=1 -->
- **Section:** Chapter 1 - Introduction
- **Focus:** CHAPTER 1 1.1 Motivation and Problem Definition

### PDF page 28 | thesis page 2
<!-- source: pdf_page=28; thesis_page=2 -->
- **Section:** Chapter 1 - Introduction
- **Focus:** Figure 1.1: An Example Design of 1-D Control Approach by Using Drag Brake In the existing works, there are two commonly used approaches to convert existing

### PDF page 29 | thesis page 3
<!-- source: pdf_page=29; thesis_page=3 -->
- **Section:** Chapter 1 - Introduction
- **Focus:** Figure 1.2: An Example Design Which Uses Movable Aerodynamic Surfaces There is also an interesting approach that uses fixed aerodynamic surfaces isolated

### PDF page 30 | thesis page 4
<!-- source: pdf_page=30; thesis_page=4 -->
- **Section:** Chapter 1 - Introduction
- **Focus:** Figure 1.3: Orientation of Fixed Canard Structure Figure 1.4: An Example Design Which Uses Fixed Aerodynamic Surfaces to get the position of the fixed-wing structure with respect to the projectile body [7].

### PDF page 31 | thesis page 5
<!-- source: pdf_page=31; thesis_page=5 -->
- **Section:** Chapter 1 - Introduction
- **Focus:** Figure 1.5: Schematics of Zhang, Li, Xiao and Tian’s Control Structure Figure 1.7: Yin, Jia and Yu’s Control Structure Flight dynamics of a projectile which has an isolated part from body rotation of the

### PDF page 32 | thesis page 6
<!-- source: pdf_page=32; thesis_page=6 -->
- **Section:** Chapter 1 - Introduction
- **Focus:** Figure 1.6: Zhang, Li, Xiao and Tian’s Test Set-up Figure 1.8: Costello’s Projectile Schematic forward body and aft body can rotate relatively.

### PDF page 33 | thesis page 7
<!-- source: pdf_page=33; thesis_page=7 -->
- **Section:** Chapter 1 - Introduction
- **Focus:** artillery projectile by using fixed canards [10].

### PDF page 34 | thesis page 8
<!-- source: pdf_page=34; thesis_page=8 -->
- **Section:** Chapter 1 - Introduction
- **Focus:** 7-DOF to include the additional degree of freedom.

### PDF page 35 | thesis page 9
<!-- source: pdf_page=35; thesis_page=9 -->
- **Section:** Chapter 2 - Test system
- **Focus:** CHAPTER 2 Table 2.1: Axial and Angular Velocities of 155 mm Spin-Stabilized Projectile 2.1 Explanation of System Aim to be Tested

### PDF page 36 | thesis page 10
<!-- source: pdf_page=36; thesis_page=10 -->
- **Section:** Chapter 2 - Test system
- **Focus:** Figure 2.1: An Example Projectile with Guidance Kit Figure 2.2: Preliminary Design of Guidance Kit Section view of Figure 2.2 is given in Figure 2.3. Guidance kit body and nose fixed

### PDF page 37 | thesis page 11
<!-- source: pdf_page=37; thesis_page=11 -->
- **Section:** Chapter 2 - Test system
- **Focus:** Figure 2.3: Section View of Preliminary Design of an Example Guidance Kit green part. Since there is no stationary part in the system, it is not possible to use

### PDF page 38 | thesis page 12
<!-- source: pdf_page=38; thesis_page=12 -->
- **Section:** Chapter 2 - Test system
- **Focus:** Figure 2.4: Steering and Control Canards and Axis System The meanings of the symbols in Figure 2.4 are given below.

### PDF page 39 | thesis page 13
<!-- source: pdf_page=39; thesis_page=13 -->
- **Section:** Chapter 2 - Test system
- **Focus:** Figure 2.5. Figure 2.5: Torques at Fixed Canard Structure and the rotation of the projectile body (+X-axis) are opposite to each other. Therefore,

### PDF page 40 | thesis page 14
<!-- source: pdf_page=40; thesis_page=14 -->
- **Section:** Chapter 2 - Test system
- **Focus:** Figure 2.9. • Tf: Friction torque at the bearings. It is in the same direction with projectile

### PDF page 41 | thesis page 15
<!-- source: pdf_page=41; thesis_page=15 -->
- **Section:** Chapter 2 - Test system
- **Focus:** Figure 2.6: Schematic of the Test Set-up Figure 2.7: General View of Test Set-up

### PDF page 42 | thesis page 16
<!-- source: pdf_page=42; thesis_page=16 -->
- **Section:** Chapter 2 - Test system
- **Focus:** Figure 2.8: Micro-controller Part Figure 2.9: AC/DC Converter and Filter Capacitor This set-up simulates aerodynamic torque on the guidance kit with a BLDC motor

### PDF page 43 | thesis page 17
<!-- source: pdf_page=43; thesis_page=17 -->
- **Section:** Chapter 2 - Test system
- **Focus:** Table 2.2: Mechanical Parts of the Actuator Part of Test Set-up of the alternator is fixed to the part that rotates with the speed motor. There is an

### PDF page 44 | thesis page 18
<!-- source: pdf_page=44; thesis_page=18 -->
- **Section:** Chapter 2 - Test system
- **Focus:** Figure 2.10: Section View of the Test Set-up Table 2.3: Speed Motor Specifications

### PDF page 45 | thesis page 19
<!-- source: pdf_page=45; thesis_page=19 -->
- **Section:** Chapter 2 - Test system
- **Focus:** Figure 2.11: Zoomed View of Actuator Part Table 2.4: Torque Motor Specifications Torque Motor This is used to simulate the aerodynamic torque at the system. Related specifications with torque motor are given in Table 2.4.

### PDF page 46 | thesis page 20
<!-- source: pdf_page=46; thesis_page=20 -->
- **Section:** Chapter 2 - Test system
- **Focus:** Table 2.5: Speed Motor Driver Specifications Table 2.6: Torque Motor Driver Specifications Speed Motor Driver This component is used to control the speed motor. Related

### PDF page 47 | thesis page 21
<!-- source: pdf_page=47; thesis_page=21 -->
- **Section:** Chapter 2 - Test system
- **Focus:** Table 2.7: Encoder Specifications Table 2.8: Alternator Specifications Encoder Type Incremental with index NA

### PDF page 48 | thesis page 22
<!-- source: pdf_page=48; thesis_page=22 -->
- **Section:** Chapter 2 - Test system
- **Focus:** Table 2.9: Slip Ring Specifications Table 2.10. Filter Capacitor This component is used to filter transient components of voltage

### PDF page 49 | thesis page 23
<!-- source: pdf_page=49; thesis_page=23 -->
- **Section:** Chapter 2 - Test system
- **Focus:** Table 2.11: AC/DC Converter Specifications Table 2.12: Power Supply Specifications Maximum continuous DC current 25 A

### PDF page 50 | thesis page 24
<!-- source: pdf_page=50; thesis_page=24 -->
- **Section:** Chapter 2 - Test system
- **Focus:** Table 2.13: Programmable DC Electronic Load Specifications Table 2.14: Micro-controller Components Micro-controller Components These components are used for data acquisition

### PDF page 51 | thesis page 25
<!-- source: pdf_page=51; thesis_page=25 -->
- **Section:** Chapter 2 - Test system
- **Focus:** Figure 2.12: User Interface of the Test Software Figure 2.13. The second part is programmed by using FPGA target of the NI-9063 to have fast and

### PDF page 52 | thesis page 26
<!-- source: pdf_page=52; thesis_page=26 -->
- **Section:** Chapter 2 - Test system
- **Focus:** Figure 2.13: Encoder Evaluation Part of the Test Software to apply minimum control effort. The software part which is responsible for the

### PDF page 53 | thesis page 27
<!-- source: pdf_page=53; thesis_page=27 -->
- **Section:** Chapter 2 - Test system
- **Focus:** Figure 2.14: Controller Part of the Test Software

### PDF page 54 | thesis page 28
<!-- source: pdf_page=54; thesis_page=28 -->
- **Section:** Chapter 2 - Test system
- **Focus:** Blank page.

### PDF page 55 | thesis page 29
<!-- source: pdf_page=55; thesis_page=29 -->
- **Section:** Chapter 3 - Test-system model
- **Focus:** CHAPTER 3 Figure 3.1: Physical Model of Test Set-up MATHEMATICAL MODEL OF THE TEST SYSTEM

### PDF page 56 | thesis page 30
<!-- source: pdf_page=56; thesis_page=30 -->
- **Section:** Chapter 3 - Test-system model
- **Focus:** Figure 3.2: Linear Graph Model of Test Set-up The Linear Graph Method is used to find the system model systematically. The linear

### PDF page 57 | thesis page 31
<!-- source: pdf_page=57; thesis_page=31 -->
- **Section:** Chapter 3 - Test-system model
- **Focus:** Figure 3.3: System Tree of the Linear Graph Model • Step 2: Include as many as A-type elements with one branch of the transformer.

### PDF page 58 | thesis page 32
<!-- source: pdf_page=58; thesis_page=32 -->
- **Section:** Chapter 3 - Test-system model
- **Focus:** The state space model of the system is simulated with Matlab/Simulink environment

### PDF page 59 | thesis page 33
<!-- source: pdf_page=59; thesis_page=33 -->
- **Section:** Chapter 3 - Test-system model
- **Focus:** Figure 3.4: Block Diagram of Simulink Model To be sure of the mathematical model that is found by using the state space approach,

### PDF page 60 | thesis page 34
<!-- source: pdf_page=60; thesis_page=34 -->
- **Section:** Chapter 3 - Test-system model
- **Focus:** Figure 3.5: Inside of Mechanical System Block Figure 3.6: Inside of Electrical System Block At the Electrical System block, relative velocity and Iload(t) are taken as inputs and

### PDF page 61 | thesis page 35
<!-- source: pdf_page=61; thesis_page=35 -->
- **Section:** Chapter 3 - Test-system model
- **Focus:** Figure 3.7: The Differences of State Variables Between State Space and Simulink The differences between the state variables are given in Figure 3.7. As it is expected,

### PDF page 62 | thesis page 36
<!-- source: pdf_page=62; thesis_page=36 -->
- **Section:** Chapter 3 - Test-system model
- **Focus:** Figure 3.8: Free Body Diagram ofJgk The torques at theJgk are explained in the list below:

### PDF page 63 | thesis page 37
<!-- source: pdf_page=63; thesis_page=37 -->
- **Section:** Chapter 3 - Test-system model
- **Focus:** 3.4.2 Transfer Function Between Taero andPgk

### PDF page 64 | thesis page 38
<!-- source: pdf_page=64; thesis_page=38 -->
- **Section:** Chapter 3 - Test-system model
- **Focus:** 3.4.3 Transfer Function Between Pbody andPgk

### PDF page 65 | thesis page 39
<!-- source: pdf_page=65; thesis_page=39 -->
- **Section:** Chapter 3 - Test-system model
- **Focus:** Figure 3.9: Comparison Between Transfer Function Models and State Space Results 3.5 Comparison of the Model Results and Test Results

### PDF page 66 | thesis page 40
<!-- source: pdf_page=66; thesis_page=40 -->
- **Section:** Chapter 3 - Test-system model
- **Focus:** Table 3.1: Open-Circuit Test Results (rad/s) Test Result (V) Simulation Result (V) Test Result (rad/s

### PDF page 67 | thesis page 41
<!-- source: pdf_page=67; thesis_page=41 -->
- **Section:** Chapter 3 - Test-system model
- **Focus:** Table 3.2: Closed-Circuit Test Results (rad/s) Test Result(V) Simulation Result(V) (Nm) Test Result(A) (N/m

### PDF page 68 | thesis page 42
<!-- source: pdf_page=68; thesis_page=42 -->
- **Section:** Chapter 3 - Test-system model
- **Focus:** Blank page.

### PDF page 69 | thesis page 43
<!-- source: pdf_page=69; thesis_page=43 -->
- **Section:** Chapter 4 - Controller design
- **Focus:** CHAPTER 4 There are many controller types used in servo control and other control problems.

### PDF page 70 | thesis page 44
<!-- source: pdf_page=70; thesis_page=44 -->
- **Section:** Chapter 4 - Controller design
- **Focus:** Figure 4.1: Cascade Control Structure noise, derivative action will be the last choice in the controller design.

### PDF page 71 | thesis page 45
<!-- source: pdf_page=71; thesis_page=45 -->
- **Section:** Chapter 4 - Controller design
- **Focus:** Table 4.1: Requirements of the Position Controller Table 4.2: Requirements of the Velocity Controller dynamics in the reference position is required when the projectile is on the required

### PDF page 72 | thesis page 46
<!-- source: pdf_page=72; thesis_page=46 -->
- **Section:** Chapter 4 - Controller design
- **Focus:** input R(s) and output C(s) is given in Equation 4.3. The dynamics of a second-order

### PDF page 73 | thesis page 47
<!-- source: pdf_page=73; thesis_page=47 -->
- **Section:** Chapter 4 - Controller design
- **Focus:** Static position error constant is found in Equation 4.8 by using the Final Value Theorem and replacing the Laplace transform of the unit step function, 1

### PDF page 74 | thesis page 48
<!-- source: pdf_page=74; thesis_page=48 -->
- **Section:** Chapter 4 - Controller design
- **Focus:** Since reference input equated to zero, the error can be found by multiplying the transfer function found in Equation 4.11 with the disturbancePbody. Equation of error due

### PDF page 75 | thesis page 49
<!-- source: pdf_page=75; thesis_page=49 -->
- **Section:** Chapter 4 - Controller design
- **Focus:** Figure 4.2: Root Locus Graph ofGIloadPgk Root-Locus of the GIloadPgk is given in Figure 4.2. The transfer function has three

### PDF page 76 | thesis page 50
<!-- source: pdf_page=76; thesis_page=50 -->
- **Section:** Chapter 4 - Controller design
- **Focus:** Figure 4.3: Root Locus Graph of Compensated Velocity Loop zero should be added near the pole to compensate for the effect of the integrator,

### PDF page 77 | thesis page 51
<!-- source: pdf_page=77; thesis_page=51 -->
- **Section:** Chapter 4 - Controller design
- **Focus:** Figure 4.4: Step Response of Velocity Loop Figure 4.5: Bode Plot of Velocity Loop The step response of the designed controller is given in Figure 4.4 and Bode graph is

### PDF page 78 | thesis page 52
<!-- source: pdf_page=78; thesis_page=52 -->
- **Section:** Chapter 4 - Controller design
- **Focus:** Figure 4.6: Error Due to Step Disturbance atTaero Also, we need to check the behavior of the controller to system disturbances. The

### PDF page 79 | thesis page 53
<!-- source: pdf_page=79; thesis_page=53 -->
- **Section:** Chapter 4 - Controller design
- **Focus:** Figure 4.7: Error Due to Step Disturbance atpbody Table 4.3: Result of Velocity Controller Design Parameter Requirement Result Unit

### PDF page 80 | thesis page 54
<!-- source: pdf_page=80; thesis_page=54 -->
- **Section:** Chapter 4 - Controller design
- **Focus:** Desired closed-loop pole locations of the position loop can be found by making

### PDF page 81 | thesis page 55
<!-- source: pdf_page=81; thesis_page=55 -->
- **Section:** Chapter 4 - Controller design
- **Focus:** Figure 4.8: Root Locus Graph ofGolp Locations of the open-loop poles are listed below.

### PDF page 82 | thesis page 56
<!-- source: pdf_page=82; thesis_page=56 -->
- **Section:** Chapter 4 - Controller design
- **Focus:** Figure 4.9: Step Response of Position Loop Table 4.4: Result of Position Controller Design The step response of the position loop is given in Figure 4.9 and the Bode plot of the

### PDF page 83 | thesis page 57
<!-- source: pdf_page=83; thesis_page=57 -->
- **Section:** Chapter 4 - Controller design
- **Focus:** Figure 4.10: Bode Plot of Position Loop Figure 4.11: Block Diagram of Simulation 4.4 Test Results and Comparison

### PDF page 84 | thesis page 58
<!-- source: pdf_page=84; thesis_page=58 -->
- **Section:** Chapter 4 - Controller design
- **Focus:** Figure 4.12: Velocity Controller Block Figure 4.13: Position Controller Block Inside of the velocity controller block is given in Figure 4.12. Clamping integration

### PDF page 85 | thesis page 59
<!-- source: pdf_page=85; thesis_page=59 -->
- **Section:** Chapter 4 - Controller design
- **Focus:** Figure 4.14: Inside of Calculate Minimum Control Effort Block Table 4.5: Test Conditions The inputs and saturation points in the test case are given in Table 4.5.

### PDF page 86 | thesis page 60
<!-- source: pdf_page=86; thesis_page=60 -->
- **Section:** Chapter 4 - Controller design
- **Focus:** Figure 4.15: Input, Simulation, and Test Results for Step Test Figure 4.16: Input, Simulation and Test Results for Step Test with Zoomed View Position controller outputs for test and simulation results are given in Figure 4.17 and

### PDF page 87 | thesis page 61
<!-- source: pdf_page=87; thesis_page=61 -->
- **Section:** Chapter 4 - Controller design
- **Focus:** Figure 4.17: Output of Position Controller at Step Test Figure 4.18: Output of Position Controller at Step Test with Zoomed View

### PDF page 88 | thesis page 62
<!-- source: pdf_page=88; thesis_page=62 -->
- **Section:** Chapter 4 - Controller design
- **Focus:** Figure 4.19:pgk at Simulation and Test Results pgk at simulation and test results are given in Figure 4.19 and zoomed view of this

### PDF page 89 | thesis page 63
<!-- source: pdf_page=89; thesis_page=63 -->
- **Section:** Chapter 4 - Controller design
- **Focus:** Figure 4.20:pgk at Simulation and Test Results with Zoomed View Figure 4.21: Input, Simulation, and Test Results for Chip Test

### PDF page 90 | thesis page 64
<!-- source: pdf_page=90; thesis_page=64 -->
- **Section:** Chapter 4 - Controller design
- **Focus:** Figure 4.22: Input, Simulation, and Test Results for Chip Test with Zoomed View The output of the position controller is compared in Figure 4.23. Also, feedback

### PDF page 91 | thesis page 65
<!-- source: pdf_page=91; thesis_page=65 -->
- **Section:** Chapter 4 - Controller design
- **Focus:** Figure 4.23: Output of the Position Controller Results for Chip Test

### PDF page 92 | thesis page 66
<!-- source: pdf_page=92; thesis_page=66 -->
- **Section:** Chapter 4 - Controller design
- **Focus:** Figure 4.24:pgk at Simulation and Test Results

### PDF page 93 | thesis page 67
<!-- source: pdf_page=93; thesis_page=67 -->
- **Section:** Chapter 5 - Flight dynamics
- **Focus:** CHAPTER 5 Figure 5.1: Torques on the Guidance Kit FLIGHT DYNAMICS OF THE ARTILLERY PROJECTILE

### PDF page 94 | thesis page 68
<!-- source: pdf_page=94; thesis_page=68 -->
- **Section:** Chapter 5 - Flight dynamics
- **Focus:** The torques at the +X and -X directions are given in Equation 5.1 and Equation 5.2.

### PDF page 95 | thesis page 69
<!-- source: pdf_page=95; thesis_page=69 -->
- **Section:** Chapter 5 - Flight dynamics
- **Focus:** Figure 5.2: Flat Earth Inertial Reference Frame times. Therefore defining a middle coordinate system that has no rolling motion is

### PDF page 96 | thesis page 70
<!-- source: pdf_page=96; thesis_page=70 -->
- **Section:** Chapter 5 - Flight dynamics
- **Focus:** 5.1.2 Body Fixed Plane Coordinate System

### PDF page 97 | thesis page 71
<!-- source: pdf_page=97; thesis_page=71 -->
- **Section:** Chapter 5 - Flight dynamics
- **Focus:** 5.2 Mathematical Model and Simulation of 6-DOF Projectile Dynamics

### PDF page 98 | thesis page 72
<!-- source: pdf_page=98; thesis_page=72 -->
- **Section:** Chapter 5 - Flight dynamics
- **Focus:** The equations which represent the kinematics of the system are given in Equation 5.9

### PDF page 99 | thesis page 73
<!-- source: pdf_page=99; thesis_page=73 -->
- **Section:** Chapter 5 - Flight dynamics
- **Focus:** Figure 5.3: Overall Structure of 6-DOF Simulation Table 5.1. The physical properties of the simulated projectile and initial conditions are given in

### PDF page 100 | thesis page 74
<!-- source: pdf_page=100; thesis_page=74 -->
- **Section:** Chapter 5 - Flight dynamics
- **Focus:** Figure 5.4: Positions at Earth Fixed Reference Frame 5.3 Mathematical Model and Simulation of 7-DOF Projectile Dynamics

### PDF page 101 | thesis page 75
<!-- source: pdf_page=101; thesis_page=75 -->
- **Section:** Chapter 5 - Flight dynamics
- **Focus:** Figure 5.5: Velocities at Earth Fixed Reference Frame The inertia matrix in Equation 5.12, I, is given in Equation 5.14.

### PDF page 102 | thesis page 76
<!-- source: pdf_page=102; thesis_page=76 -->
- **Section:** Chapter 5 - Flight dynamics
- **Focus:** Figure 5.6: Positions at Earth Fixed Reference Frame Since the kinematics of the system is not change with the addition of 1-DOF, it is

### PDF page 103 | thesis page 77
<!-- source: pdf_page=103; thesis_page=77 -->
- **Section:** Chapter 5 - Flight dynamics
- **Focus:** Figure 5.7: Velocities at Earth Fixed Reference Frame Figure 5.8: Control Forces at Body Fixed Plane Reference Frame The control forces and moments that arise from the addition of the guidance kit are

### PDF page 104 | thesis page 78
<!-- source: pdf_page=104; thesis_page=78 -->
- **Section:** Chapter 5 - Flight dynamics
- **Focus:** Figure 5.9: Control Moments at Body Fixed Plane Reference Frame Aerodynamic torqueTaero and aerodynamic damping torque Tad on the guidance kit

### PDF page 105 | thesis page 79
<!-- source: pdf_page=105; thesis_page=79 -->
- **Section:** Chapter 5 - Flight dynamics
- **Focus:** Figure 5.10: Aerodynamic TorqueTaero on Guidance Kit Figure 5.11: Aerodynamic Damping TorqueTad on Guidance Kit

### PDF page 106 | thesis page 80
<!-- source: pdf_page=106; thesis_page=80 -->
- **Section:** Chapter 5 - Flight dynamics
- **Focus:** Blank page.

### PDF page 107 | thesis page 81
<!-- source: pdf_page=107; thesis_page=81 -->
- **Section:** Chapter 6 - Adaptive controller
- **Focus:** CHAPTER 6 MODIFICATION OF THE CONTROLLER FOR CHANGING

### PDF page 108 | thesis page 82
<!-- source: pdf_page=108; thesis_page=82 -->
- **Section:** Chapter 6 - Adaptive controller
- **Focus:** Table 6.1: Initial Conditions modified for low aerodynamic torque inputs.

### PDF page 109 | thesis page 83
<!-- source: pdf_page=109; thesis_page=83 -->
- **Section:** Chapter 6 - Adaptive controller
- **Focus:** Figure 6.1: Aerodynamic Torque for Selected Flight Conditions Figure 6.2: Body Angular Velocity for Selected Flight Conditions

### PDF page 110 | thesis page 84
<!-- source: pdf_page=110; thesis_page=84 -->
- **Section:** Chapter 6 - Adaptive controller
- **Focus:** Figure 6.3: Input and Output Positions for Unmodified Controller 6.2 Results for Current Control Structure

### PDF page 111 | thesis page 85
<!-- source: pdf_page=111; thesis_page=85 -->
- **Section:** Chapter 6 - Adaptive controller
- **Focus:** Figure 6.4: Input and Output Velocities for Unmodified Controller From the results of the unmodified controller it is possible to state that near to apogee

### PDF page 112 | thesis page 86
<!-- source: pdf_page=112; thesis_page=86 -->
- **Section:** Chapter 6 - Adaptive controller
- **Focus:** Figure 6.5. Since this model uses the inverse of the plant, the input current and output Figure 6.5: Block Diagram of Disturbance Observer velocity need to be filtered to make the system physically applicable, which means the

### PDF page 113 | thesis page 87
<!-- source: pdf_page=113; thesis_page=87 -->
- **Section:** Chapter 6 - Adaptive controller
- **Focus:** Figure 6.6: Real and Observed Disturbance Current the system is given in Figure 6.6. Since aerodynamic torque and friction torque do

### PDF page 114 | thesis page 88
<!-- source: pdf_page=114; thesis_page=88 -->
- **Section:** Chapter 6 - Adaptive controller
- **Focus:** Figure 6.7: Input and Output Positions for Dynamic Current Limit be equal to two times of the disturbance observer result to have symmetric responses

### PDF page 115 | thesis page 89
<!-- source: pdf_page=115; thesis_page=89 -->
- **Section:** Chapter 6 - Adaptive controller
- **Focus:** Figure 6.8: Velocity Controller Result for Dynamic Current Limit and Observed Current 6.4.2 Changing Velocity Limit Dynamically with Aerodynamic Torque

### PDF page 116 | thesis page 90
<!-- source: pdf_page=116; thesis_page=90 -->
- **Section:** Chapter 6 - Adaptive controller
- **Focus:** Figure 6.9: Input and Output Positions for Dynamic Velocity Limit the changes of the reference angle at 40th s and 42th s are the same but the shape of

### PDF page 117 | thesis page 91
<!-- source: pdf_page=117; thesis_page=91 -->
- **Section:** Chapter 6 - Adaptive controller
- **Focus:** Figure 6.10: Velocity Controller Result for Dynamic Velocity Limit Figure 6.11: Velocity Controller Result for Dynamic Velocity Limit - Zoomed View

### PDF page 118 | thesis page 92
<!-- source: pdf_page=118; thesis_page=92 -->
- **Section:** Chapter 6 - Adaptive controller
- **Focus:** Figure 6.12: Input and Output Positions for Combined Solution 6.4.3 Changing Velocity and Current Limit Dynamically with Aerodynamic

### PDF page 119 | thesis page 93
<!-- source: pdf_page=119; thesis_page=93 -->
- **Section:** Chapter 6 - Adaptive controller
- **Focus:** Figure 6.13: Velocity Controller Result for Combined Solution Figure 6.14: Velocity Controller Result for Combined Solution - Zoomed View

### PDF page 120 | thesis page 94
<!-- source: pdf_page=120; thesis_page=94 -->
- **Section:** Chapter 6 - Adaptive controller
- **Focus:** Blank page.

### PDF page 121 | thesis page 95
<!-- source: pdf_page=121; thesis_page=95 -->
- **Section:** Chapter 7 - Hardware results
- **Focus:** CHAPTER 7 Figure 7.1. Zoomed views of this figure are given in Figure 7.2 and Figure 7.3. In the TEST RESULTS FOR MODIFIED CONTROLLERS

### PDF page 122 | thesis page 96
<!-- source: pdf_page=122; thesis_page=96 -->
- **Section:** Chapter 7 - Hardware results
- **Focus:** Figure 7.1: Input Position, Simulation, and Test Results for Unmodified Controller require fixed-point numbers. The losses at the conversion between floating point and

### PDF page 123 | thesis page 97
<!-- source: pdf_page=123; thesis_page=97 -->
- **Section:** Chapter 7 - Hardware results
- **Focus:** Figure 7.2: Input Position, Simulation, and Test Results for Unmodified Controller- Figure 7.3: Input Position, Simulation, and Test Results for Unmodified Controller-

### PDF page 124 | thesis page 98
<!-- source: pdf_page=124; thesis_page=98 -->
- **Section:** Chapter 7 - Hardware results
- **Focus:** Figure 7.4: Alternator Saturation Points and Velocity Controller Results Figure 7.6. Since the results of the disturbance observer are nearly the same for all 7.2 Test Results for Changing Current Limit

### PDF page 125 | thesis page 99
<!-- source: pdf_page=125; thesis_page=99 -->
- **Section:** Chapter 7 - Hardware results
- **Focus:** Figure 7.5: Input Position, Simulation, and Test Results for Changing Current Limit Figure 7.6: Comparison of the Disturbance Observer Results

### PDF page 126 | thesis page 100
<!-- source: pdf_page=126; thesis_page=100 -->
- **Section:** Chapter 7 - Hardware results
- **Focus:** Figure 7.7: Input Position, Simulation, and Test Results for Changing Velocity Limit Figure 7.8 for input and output positions. It is the best solution because we have no 7.3 Test Results for Changing Velocity Limit

### PDF page 127 | thesis page 101
<!-- source: pdf_page=127; thesis_page=101 -->
- **Section:** Chapter 7 - Hardware results
- **Focus:** Figure 7.8: Input Position, Simulation, and Test Results for Combined Solution The best solution is the combined solution due to symmetric response characteristics in both directions. Also in this solution, the transient response is consistent with

### PDF page 128 | thesis page 102
<!-- source: pdf_page=128; thesis_page=102 -->
- **Section:** Chapter 7 - Hardware results
- **Focus:** Blank page.

### PDF page 129 | thesis page 103
<!-- source: pdf_page=129; thesis_page=103 -->
- **Section:** Chapter 8 - Conclusion and future work
- **Focus:** CHAPTER 8 In this thesis subject, the feasibility of a guidance method for spin-stabilized munitions is examined.

### PDF page 130 | thesis page 104
<!-- source: pdf_page=130; thesis_page=104 -->
- **Section:** Chapter 8 - Conclusion and future work
- **Focus:** In chapter 6, the aerodynamic torque and body angular velocity, which are found in

### PDF page 131 | thesis page 105
<!-- source: pdf_page=131; thesis_page=105 -->
- **Section:** Chapter 8 - Conclusion and future work
- **Focus:** Aerodynamic torque found by simulations is applied directly to the test system. To

### PDF page 132 | thesis page 106
<!-- source: pdf_page=132; thesis_page=106 -->
- **Section:** Chapter 8 - Conclusion and future work
- **Focus:** Blank page.

### PDF page 133 | thesis page 107
<!-- source: pdf_page=133; thesis_page=107 -->
- **Section:** References
- **Focus:** REFERENCES [1] Y . Wang, X.-m. Wang, and J.-y. Yu, “Influence of control strategy on stability

### PDF page 134 | thesis page 108
<!-- source: pdf_page=134; thesis_page=108 -->
- **Section:** References
- **Focus:** [10] X. Zhang, Y . Xiaoxian, and Q. Zheng, “Impact point prediction guidance based

### PDF page 135 | thesis page 109
<!-- source: pdf_page=135; thesis_page=109 -->
- **Section:** References
- **Focus:** [21] S. Li, J. Yang, W.-H. Chen, and X. Chen, Disturbance observer-based control:

### PDF page 136 | thesis page 110
<!-- source: pdf_page=136; thesis_page=110 -->
- **Section:** References
- **Focus:** Blank page.

### PDF page 137 | thesis page 111
<!-- source: pdf_page=137; thesis_page=111 -->
- **Section:** Appendix A - Linear graph
- **Focus:** Appendix A To be consistent with flight dynamics notation, angular velocity on X-axis is presented

### PDF page 138 | thesis page 112
<!-- source: pdf_page=138; thesis_page=112 -->
- **Section:** Appendix A - Linear graph
- **Focus:** Continuity equations of the system are listed below:

### PDF page 139 | thesis page 113
<!-- source: pdf_page=139; thesis_page=113 -->
- **Section:** Appendix A - Linear graph
- **Focus:** By using the 16 equations listed above, the derivative of system states can be expressed in terms of input and states of the system. These equations are given below.

### PDF page 140 | thesis page 114
<!-- source: pdf_page=140; thesis_page=114 -->
- **Section:** Appendix A - Linear graph
- **Focus:** Blank page.

### PDF page 141 | thesis page 115
<!-- source: pdf_page=141; thesis_page=115 -->
- **Section:** Appendix B - Transfer functions
- **Focus:** Appendix B Figure B.1: Circuit of the Electrical Part To be consistent with flight dynamics notation, angular velocity on X-axis is presented

### PDF page 142 | thesis page 116
<!-- source: pdf_page=142; thesis_page=116 -->
- **Section:** Appendix B - Transfer functions
- **Focus:** Figure B.2: Free Body Diagram ofJgk forIload -Kv(Pbody-Pgk) +RaIa +LasIa + 1

### PDF page 143 | thesis page 117
<!-- source: pdf_page=143; thesis_page=117 -->
- **Section:** Appendix B - Transfer functions
- **Focus:** We need to check the units of the parts added atσ1 andσ2.

### PDF page 144 | thesis page 118
<!-- source: pdf_page=144; thesis_page=118 -->
- **Section:** Appendix B - Transfer functions
- **Focus:** Figure B.3: Free Body Diagram ofJgk forTaero Since there will be a voltage change at the alternator due to changing pgk we need

### PDF page 145 | thesis page 119
<!-- source: pdf_page=145; thesis_page=119 -->
- **Section:** Appendix B - Transfer functions
- **Focus:** Figure B.4: Free Body Diagram ofJgk forpbody There is no need to check units at this transfer function.

### PDF page 146 | thesis page 120
<!-- source: pdf_page=146; thesis_page=120 -->
- **Section:** Appendix B - Transfer functions
- **Focus:** The equation of motion is given in Equation B.14 in Laplace domain.

### PDF page 147 | thesis page 121
<!-- source: pdf_page=147; thesis_page=121 -->
- **Section:** Appendix B - Transfer functions
- **Focus:** We need to check the units of added terms inβ1.

### PDF page 148 | thesis page 122
<!-- source: pdf_page=148; thesis_page=122 -->
- **Section:** Appendix B - Transfer functions
- **Focus:** Blank page.

### PDF page 149 | thesis page 123
<!-- source: pdf_page=149; thesis_page=123 -->
- **Section:** Appendix C - MATLAB code
- **Focus:** Appendix C J_holder = 1.678 *10^-4; % in kg *m^2

### PDF page 150 | thesis page 124
<!-- source: pdf_page=150; thesis_page=124 -->
- **Section:** Appendix C - MATLAB code
- **Focus:** % Form system and input matrices

### PDF page 151 | thesis page 125
<!-- source: pdf_page=151; thesis_page=125 -->
- **Section:** Appendix C - MATLAB code
- **Focus:** stateSpaceModel = ss(A, B, C, D)

### PDF page 152 | thesis page 126
<!-- source: pdf_page=152; thesis_page=126 -->
- **Section:** Appendix C - MATLAB code
- **Focus:** denum2 = [sigma3, sigma2, sigma1, sigma0];

### PDF page 153 | thesis page 127
<!-- source: pdf_page=153; thesis_page=127 -->
- **Section:** Appendix C - MATLAB code
- **Focus:** rlocus(G_Iload_pgk*((s+26.98)/s))

### PDF page 154 | thesis page 128
<!-- source: pdf_page=154; thesis_page=128 -->
- **Section:** Appendix C - MATLAB code
- **Focus:** %Position controller requirements

### PDF page 155 | thesis page 129
<!-- source: pdf_page=155; thesis_page=129 -->
- **Section:** Appendix D - Reference-frame derivation
- **Focus:** Appendix D Equation of motion in a non-inertial reference frame is given in Equation D.1.

### PDF page 156 | thesis page 130
<!-- source: pdf_page=156; thesis_page=130 -->
- **Section:** Appendix D - Reference-frame derivation
- **Focus:** ωBFP : Angular velocity vector of the Body Fixed Plane Reference Frame.

### PDF page 157 | thesis page 131
<!-- source: pdf_page=157; thesis_page=131 -->
- **Section:** Appendix D - Reference-frame derivation
- **Focus:** If we take the derivative of the angular velocity vector in Equation D.1 to the left side

### PDF page 158 | thesis page 132
<!-- source: pdf_page=158; thesis_page=132 -->
- **Section:** Appendix D - Reference-frame derivation
- **Focus:** cosθ cosψ sinϕ sinθ cosψ- cosϕ sinψ cosϕ sinθ cosψ + sinϕ sinψ

### PDF page 159 | thesis page 133
<!-- source: pdf_page=159; thesis_page=133 -->
- **Section:** Appendix E - 7-DOF modification
- **Focus:** Appendix E With the addition of 1-DOF to the system, the inertia matrix, moment vector, angular

### PDF page 160 | thesis page 134
<!-- source: pdf_page=160; thesis_page=134 -->
- **Section:** Appendix E - 7-DOF modification
- **Focus:** The skew-symmetric matrix version of the cross product of ωBFP needs to be modified to be consistent with the dimensions of the other vectors. Also, the first row of

### PDF page 161 | thesis page 135
<!-- source: pdf_page=161; thesis_page=135 -->
- **Section:** Appendix F - Plant reduction/filter
- **Focus:** Appendix F The numerical value of the transfer function betweenIload andpgk is given in Equation

### PDF page 162 | thesis page 136
<!-- source: pdf_page=162; thesis_page=136 -->
- **Section:** Appendix F - Plant reduction/filter
- **Focus:** Figure F.1: Comparison of Real and Reduced Transfer Functions The main parameter in the first-order low pass filter design is the time constant. A

### PDF page 163 | thesis page 137
<!-- source: pdf_page=163; thesis_page=137 -->
- **Section:** Appendix F - Plant reduction/filter
- **Focus:** Figure F.2: Bode Plot of the Filter Figure F.3: Step Response of the Filter

### PDF page 164 | thesis page 138
<!-- source: pdf_page=164; thesis_page=138 -->
- **Section:** Appendix F - Plant reduction/filter
- **Focus:** Blank page.

### PDF page 165 | thesis page 139
<!-- source: pdf_page=165; thesis_page=139 -->
- **Section:** Appendix G - Aerodynamic equations
- **Focus:** Appendix G G.1 Aerodynamic Forces And Moments

### PDF page 166 | thesis page 140
<!-- source: pdf_page=166; thesis_page=140 -->
- **Section:** Appendix G - Aerodynamic equations
- **Focus:** G.1.1.6 Pitch / Yaw Damping Moment

### PDF page 167 | thesis page 141
<!-- source: pdf_page=167; thesis_page=141 -->
- **Section:** Appendix G - Aerodynamic equations
- **Focus:** Total aerodynamic forces on the projectile are given in Equation G.10.

### PDF page 168 | thesis page 142
<!-- source: pdf_page=168; thesis_page=142 -->
- **Section:** Appendix G - Aerodynamic equations
- **Focus:** These are the moments that arise from the addition of the guidance kit to the projectile.

### PDF page 169 | thesis page 143
<!-- source: pdf_page=169; thesis_page=143 -->
- **Section:** Appendix G - Aerodynamic equations
- **Focus:** G.1.2.4 Aerodynamic Damping Torque on Guidance Kit


## Appendix A - Page-ordered extracted source text

This appendix is an extraction layer, not the normalized interpretation above. It retains page-level wording, equations, table rows, and captions. For exact visual layout, consult the source PDF; for reasoning and retrieval, use the structured sections and evidence map first.

### PDF page 1 | thesis page n/a
<!-- raw_extraction: pdf_page=1; thesis_page=n/a -->
```text
CONTROLLER DESIGN AND IMPLEMENTATION FOR A GUIDANCE KIT
USED AT SPINNING ROCKETS AND PROJECTILES
A THESIS SUBMITTED TO
THE GRADUATE SCHOOL OF NATURAL AND APPLIED SCIENCES
OF
MIDDLE EAST TECHNICAL UNIVERSITY
BY
GÖRKEM GÜNAY
IN PARTIAL FULFILLMENT OF THE REQUIREMENTS
FOR
THE DEGREE OF MASTER OF SCIENCE
IN
MECHANICAL ENGINEERING
APRIL 2022
```

### PDF page 2 | thesis page n/a
<!-- raw_extraction: pdf_page=2; thesis_page=n/a -->
*(Blank page.)*

### PDF page 3 | thesis page n/a
<!-- raw_extraction: pdf_page=3; thesis_page=n/a -->
```text
Approval of the thesis:
CONTROLLER DESIGN AND IMPLEMENTATION FOR A GUIDANCE
KIT USED AT SPINNING ROCKETS AND PROJECTILES
submitted by GÖRKEM GÜNAY in partial fulfillment of the requirements for the
degree of Master of Science in Mechanical Engineering Department, Middle
East Technical University by,
Prof. Dr. Halil Kalıpçılar
Dean, Graduate School of Natural and Applied Sciences
Prof. Dr. Mehmet Ali Sahir Arıkan
Head of Department, Mechanical Engineering
Prof. Dr. Raif Tuna Balkan
Supervisor, Mechanical Engineering
Assist. Prof. Dr. Hakan Çalışkan
Co-supervisor, Mechanical Engineering
Examining Committee Members:
Assoc. Prof. Dr. Ulaş Yaman
Mechanical Engineering, METU
Prof. Dr. Raif Tuna Balkan
Mechanical Engineering, METU
Assist. Prof. Dr. Orkun Özşahin
Mechanical Engineering, METU
Assist. Prof. Dr. Hakan Çalışkan
Mechanical Engineering, METU
Assoc. Prof. Dr. Sinan Kılıçaslan
Mechanical Engineering, Gazi University
Date:28.04.2022
```

### PDF page 4 | thesis page iv
<!-- raw_extraction: pdf_page=4; thesis_page=iv -->
```text
I hereby declare that all information in this document has been obtained and
presented in accordance with academic rules and ethical conduct. I also declare
that, as required by these rules and conduct, I have fully cited and referenced all
material and results that are not original to this work.
Name, Surname: Görkem Günay
Signature :
```

### PDF page 5 | thesis page v
<!-- raw_extraction: pdf_page=5; thesis_page=v -->
```text
ABSTRACT
CONTROLLER DESIGN AND IMPLEMENTATION FOR A GUIDANCE
KIT USED AT SPINNING ROCKETS AND PROJECTILES
Günay, Görkem
M.S., Department of Mechanical Engineering
Supervisor: Prof. Dr. Raif Tuna Balkan
Co-Supervisor: Assist. Prof. Dr. Hakan Çalışkan
April 2022, 143 pages
Traditional spin-stabilized projectiles have disadvantages due to low accuracy and
precision with respect to guided munitions. There are two main solutions to eliminate
these disadvantages. The first one uses a drag brake mechanism to make 1-D guidance by making range correction. The second solution makes 2-D guidance. Since
rotation dynamics of a spin-stabilized projectile is faster than existing servo actuators
this solution requires isolation of the guidance kit from the rotation of the projectile
body. It is possible to use traditional control actuation systems with movable aerodynamic surfaces for 2-D guidance but it requires complicated electronics and a battery.
In the scope of this thesis, the feasibility of an actuation solution to 2-D guidance
problem of spin-stabilized projectiles is examined. The examined solution uses a
fixed canard structure which is isolated from body rotation. The actuator part consists
of an alternator. The stator of the alternator rotates with the projectile body and the
rotor of the alternator rotates with a fixed canard structure. There is a net torque and
a net force generated by fixed canards and guidance can be done by controlling the
direction of the net force. Therefore, the main actuation problem is position control
```

### PDF page 6 | thesis page vi
<!-- raw_extraction: pdf_page=6; thesis_page=vi -->
```text
of an alternator. Since the actuation problem is reduced to one-axis position control,
required driver electronics and actuators are simplified. And also, it is possible to use
power generated by the alternator instead of using a battery.
A test system is developed to simulate the rotation of the projectile body and aerodynamic torque on the guidance kit. Mathematical model of the test system with the
alternator is created. To make position control of the alternator, a cascade PI controller is designed and applied in the test system. Requirements of the controller are
satisfied with PI controller in the velocity loop and P controller in the position loop.
Clamping anti-windup method is used to overcome the overshoot problem caused by
the limitation of the controller output in the velocity loop.
Also, in the scope of this thesis, flight dynamics of a spin-stabilized artillery projectile
is examined. Firstly, the model is established in 6-DOF, then it is extended to 7-
DOF with the addition of 1-DOF generated by the free rotating part of the guidance
kit. This 7-DOF flight simulation model is used to find the angular velocity of the
projectile body and aerodynamic torque on the guidance kit.
The designed controller is tested with changing aerodynamic torque and body angular velocity. The aerodynamic torque found by using 7-DOF simulation is directly
applied to the test system but the angular velocity of the body is scaled down to be
inside of the limits of the test system. A disturbance observer is designed to be able to
find the sum of the aerodynamic torque and friction torque generated by body angular velocity. The saturation parameters are updated with respect to total disturbance
torque on the system to have control ability through to fight time.
Keywords: Cascade PI Control, Disturbance Observer, Projectile Guidance Kit, Projectile Flight Dynamics, Spinning Projectiles
```

### PDF page 7 | thesis page vii
<!-- raw_extraction: pdf_page=7; thesis_page=vii -->
```text
ÖZ
DÖNEN ROKET VE MÜH˙IMMATLARDA KULLANILAN B˙IR GÜDÜM
K˙IT˙I ˙IÇ˙IN KONTROLCÜ TASARIMI VE UYGULANMASI
Günay, Görkem
Yüksek Lisans, Makina Mühendisli˘gi Bölümü
Tez Yöneticisi: Prof. Dr. Raif Tuna Balkan
Ortak Tez Yöneticisi: Dr. Ö˘gr. Üyesi. Hakan Çalışkan
Nisan 2022 , 143 sayfa
Geleneksel dönü kararlı mermiler dü¸ sük hassasiyet ve do˘gruluk sebebi ile güdümlü
mühimmatlara kıyasla dezavantaja sahiptir. Bu dezavantajı gidermek için iki ana çözüm bulunmaktadır.˙Ilk çözüm bir fren mekanizması yardımıyla menzilce tek boyutlu
düzeltme yapar. ˙Ikinci çözüm iki boyutlu düzeltme yapmaktadır. Dönü kararlı mermilerin dinami˘gi günümüzde bulunan eyleyicilerin dinami˘ginden yüksek oldu˘gu için
bu çözümde güdüm kitini mermi gövdesinden yalıtmak gerekir.˙Iki boyutlu düzeltme
için hareket edebilir aerodinamik yüzeyler içeren geleneksel kontrol tahrik sistemlerini kullanmak mümkündür ancak bu karma¸ sık elektronik bile¸ senler ve batarya gerektirir. Bu tezde dönü kararlı mermi ve roketlerin güdümlendirilme problemine çözüm
olabilecek bir yöntem çalı¸ sılmı¸ stır. Çalı¸ sılan çözüm mermi gövdesinden ba˘gımsız dönebilen sabit kanatlı bir yapı kullanmaktadır. Eyleyici kısmı sarımları mermi gövdesi
ile birlikte, mıknatısları sabit kanatlı yapı ile birlikte dönen bir alternatörden olu¸ sur.
Sistemde sabit kanatlar tarafından olu¸ san net kuvvet ve net tork bulunmaktadır ve
net kuvvetin yönü ayarlanarak güdüm yapılabilir. Dolayısıyla ana tahrik problemi bir
```

### PDF page 8 | thesis page viii
<!-- raw_extraction: pdf_page=8; thesis_page=viii -->
```text
alternatörün pozisyon kontrolüdür. Temel tahrik problemi tek eksende pozisyon kontrolüne indirgendi˘ginden dolayı gerekli elektronik sistemler ve eyleyiciler basitle¸ stirilmi¸ stir. Ayrıca alternatörden elde edilen gücü batarya yerine kullanmak mümkündür.
Mermi gövdesinin dönüsünü ve güdüm kiti üzerindeki aerodinamik torku simule etmek için bir test sistemi geli¸ stirilmi¸ stir. Alternatör ile birle¸ stirilen test sisteminin matematiksel modeli elde edilmi¸ stir. Alternatörün pozisyon kontrolü için bir kaskad PI
denetleyici tasarlanmı¸ s ve test sisteminde uygulanmı¸ stır. Denetleyici isterleri hız döngüsünde PI denetleyici, pozisyon döngüsünde P denetleyici ile kar¸ sılanmı¸ stır. Hız
döngüsünde denetleyici çıktısının sınırlandırılması nedeniyle olu¸ san a¸ sım problemini
çözmek için "Clamping anti-windup" yöntemi kullanılmı¸ stır.
Bu tezin içeri ˘gi kapsamında dönü kararlı top mermilerinin uçu¸ s dinami ˘gi de incelenmi¸ stir. Uçu¸ s dinami˘gi ilk olarak 6 serbestlik derecesi ile incelenmi¸ s, daha sonra
güdüm kiti tarafından sisteme eklenen 1 serbestlik derecesi de dahil edilerek benzetim 7 serbestlik derecesine çıkarılmı¸ stır. 7 serbestlik derecesi içeren uçu¸ s modeli top
mermisi gövdesinin açısal hızını ve güdüm kiti üzerinde olu¸ san aerodinamik torku
bulmak için kullanılmı¸ stır.
Tasarlanan denetleyici de˘gi¸ sken aerodinamik tork ve gövde açısal hızı için test edilmi¸ stir. 7 serbestlik dereceli uçu¸ s simulasyonu kullanılarak bulunan aerodinamik tork
test sisteminde de˘gi¸ stirilmeden uygulanmı¸ stır ancak gövde açısal hızı test sisteminin
limitlerinin içinde kalabilmek için oransal olarak azaltılmı¸ stır. Güdüm kiti üzerinde
olu¸ san aerodinamik tork ve gövde dönüsü dolayısıyla olu¸ san sürtünme torkunun toplam de ˘geri bozucu kestirimi yoluyla elde edilmi¸ stir. Denetleyicilerin doyum noktası
de˘gi¸ skenleri sistemdeki toplam bozucu etkiye göre uçu¸ s boyunca denetleme kabiliyetinin korunması amacıyla güncellenmi¸ stir.
Anahtar Kelimeler: Kaskad PI Kontrolcü, Bozucu Kestirimi, Top Mermisi Güdüm
Kiti, Top Mermisi Uçu¸ s Mekani˘gi, Dönü Kararlı Top Mermisi
```

### PDF page 9 | thesis page ix
<!-- raw_extraction: pdf_page=9; thesis_page=ix -->
```text
If you want to find the secrets of the universe, think in terms of energy, frequency
and vibration.
```

### PDF page 10 | thesis page x
<!-- raw_extraction: pdf_page=10; thesis_page=x -->
```text
ACKNOWLEDGMENTS
Firstly, I want to give my deepest thanks to my supervisor Prof. Dr. Raif Tuna
Balkan, and my co-supervisor Assist. Prof. Dr. Hakan Çalışkan for the knowledge
and experience they share with me and their positive attitude to me during this thesis
period.
I also want to thank Roketsan and my colleagues at this company, for their friendship
and their technical support to me during this thesis process. Especially, I want to
thank Berkan Bilir who is my first manager at this company.
I owe a thank you to my friends Banu Çiçek Büyüker and Ali Mert Vural for their
support during the thesis process. Also, I want to thank Okan Günel, Ali Nuri Güne¸ sdo˘gdu, Önder Gezer, Cemal Gönen, and Emre Ersoy who are my friends from Mechanical Engineering Department.
Lastly, I am thankful to my family and all my friends for their endless support to me
in this thesis and in my life.
```

### PDF page 11 | thesis page xi
<!-- raw_extraction: pdf_page=11; thesis_page=xi -->
```text
TABLE OF CONTENTS
ABSTRACT . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . v
ÖZ . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . vii
ACKNOWLEDGMENTS . . . . . . . . . . . . . . . . . . . . . . . . . . . . . x
TABLE OF CONTENTS . . . . . . . . . . . . . . . . . . . . . . . . . . . . . xi
LIST OF TABLES . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . xvi
LIST OF FIGURES . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . xviii
LIST OF ABBREVIATIONS . . . . . . . . . . . . . . . . . . . . . . . . . . . xxiv
CHAPTERS
1 INTRODUCTION . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 1
1.1 Motivation and Problem Definition . . . . . . . . . . . . . . . . . . 1
1.2 Literature Survey . . . . . . . . . . . . . . . . . . . . . . . . . . . . 2
1.3 Contributions and Novelties . . . . . . . . . . . . . . . . . . . . . . 7
1.4 The Outline of the Thesis . . . . . . . . . . . . . . . . . . . . . . . . 7
2 EXPLANATION OF THE TEST SYSTEM . . . . . . . . . . . . . . . . . 9
2.1 Explanation of System Aim to be Tested . . . . . . . . . . . . . . . . 9
2.2 Explanation of Test Hardware . . . . . . . . . . . . . . . . . . . . . 14
2.2.1 General Description of the Test Set-up . . . . . . . . . . . . . 14
2.2.2 Specifications of the Test Hardware . . . . . . . . . . . . . . . 17
```

### PDF page 12 | thesis page xii
<!-- raw_extraction: pdf_page=12; thesis_page=xii -->
```text
Speed Motor . . . . . . . . . . . . . . . . . . . . . . . . . . . . 17
Torque Motor . . . . . . . . . . . . . . . . . . . . . . . . . . . 19
Speed Motor Driver . . . . . . . . . . . . . . . . . . . . . . . . 20
Torque Motor Driver . . . . . . . . . . . . . . . . . . . . . . . 20
Encoder . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 20
Alternator . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 21
Slip Ring . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 21
Filter Capacitor . . . . . . . . . . . . . . . . . . . . . . . . . . 22
AC/DC Converter . . . . . . . . . . . . . . . . . . . . . . . . . 22
Power Supply . . . . . . . . . . . . . . . . . . . . . . . . . . . 23
Programmable DC Electronic Load . . . . . . . . . . . . . . . . 23
Micro-controller Components . . . . . . . . . . . . . . . . . . . 24
2.3 Explanation of the Test Software . . . . . . . . . . . . . . . . . . . . 24
3 MATHEMATICAL MODEL OF THE TEST SYSTEM . . . . . . . . . . . 29
3.1 State Space Model . . . . . . . . . . . . . . . . . . . . . . . . . . . 30
3.2 Simulink Model . . . . . . . . . . . . . . . . . . . . . . . . . . . . 33
3.3 Comparison of the System Models . . . . . . . . . . . . . . . . . . . 34
3.4 Transfer Functions Between Inputs and Pgk . . . . . . . . . . . . . . 35
3.4.1 Transfer Function Between Iload andPgk . . . . . . . . . . . . 36
3.4.2 Transfer Function Between Taero andPgk . . . . . . . . . . . . 37
3.4.3 Transfer Function Between Pbody andPgk . . . . . . . . . . . 38
3.5 Comparison of the Model Results and Test Results . . . . . . . . . . 39
3.5.1 Comparison of the Open-Circuit Test Results . . . . . . . . . 39
```

### PDF page 13 | thesis page xiii
<!-- raw_extraction: pdf_page=13; thesis_page=xiii -->
```text
3.5.2 Comparison of the Closed-Circuit Test Results . . . . . . . . . 40
4 CONTROLLER DESIGN . . . . . . . . . . . . . . . . . . . . . . . . . . . 43
4.1 Requirements for Position and Velocity Controllers . . . . . . . . . . 44
4.2 Velocity Controller Design . . . . . . . . . . . . . . . . . . . . . . . 45
4.3 Position Controller Design . . . . . . . . . . . . . . . . . . . . . . . 54
4.4 Test Results and Comparison . . . . . . . . . . . . . . . . . . . . . . 57
4.4.1 Comparison of the Step Test Results . . . . . . . . . . . . . . 59
4.4.2 Comparison of the Chirp Test Results . . . . . . . . . . . . . 62
5 FLIGHT DYNAMICS OF THE ARTILLERY PROJECTILE . . . . . . . . 67
5.1 Coordinate Frames . . . . . . . . . . . . . . . . . . . . . . . . . . . 68
5.1.1 Body Coordinate Frame . . . . . . . . . . . . . . . . . . . . . 68
5.1.2 Body Fixed Plane Coordinate System . . . . . . . . . . . . . . 70
5.1.3 Body Non-Spinning Frame . . . . . . . . . . . . . . . . . . . 70
5.2 Mathematical Model and Simulation of 6-DOF Projectile Dynamics . 71
5.2.1 6-DOF Dynamical Model in Body Fixed Plane Coordinate
System . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 71
5.2.2 6-DOF Simulink Model and Results . . . . . . . . . . . . . . 72
5.3 Mathematical Model and Simulation of 7-DOF Projectile Dynamics . 74
5.3.1 Simulink Model and Results . . . . . . . . . . . . . . . . . . 76
6 MODIFICATION OF THE CONTROLLER FOR CHANGING AERODY-
NAMIC TORQUE AND BODY ANGULAR VELOCITY . . . . . . . . . . 81
6.1 Flight Scenario to be Tested . . . . . . . . . . . . . . . . . . . . . . 82
6.2 Results for Current Control Structure . . . . . . . . . . . . . . . . . 84
6.3 Disturbance Observer Design . . . . . . . . . . . . . . . . . . . . . 85
```

### PDF page 14 | thesis page xiv
<!-- raw_extraction: pdf_page=14; thesis_page=xiv -->
```text
6.4 Possible Solution Approaches . . . . . . . . . . . . . . . . . . . . . 87
6.4.1 Changing Current Limit Dynamically with Aerodynamic Torque 87
6.4.2 Changing Velocity Limit Dynamically with Aerodynamic Torque 89
6.4.3 Changing Velocity and Current Limit Dynamically with Aerodynamic Torque . . . . . . . . . . . . . . . . . . . . . . . . . 92
7 TEST RESULTS FOR MODIFIED CONTROLLERS . . . . . . . . . . . . 95
7.1 The Results for Unmodified Controller . . . . . . . . . . . . . . . . 95
7.2 Test Results for Changing Current Limit . . . . . . . . . . . . . . . . 98
7.3 Test Results for Changing Velocity Limit . . . . . . . . . . . . . . . 100
7.4 Test Results for Combined Solution . . . . . . . . . . . . . . . . . . 100
8 CONCLUSION AND FUTURE WORK . . . . . . . . . . . . . . . . . . . 103
8.1 Conclusion . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 103
8.2 Future Work . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 104
REFERENCES . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 107
A . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 111
B . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 115
B.1 Derivation of the Transfer Function Between Iload andPgk . . . . . . 116
B.2 Derivation of the Transfer Function Between Taero(t) andPgk . . . . 117
B.3 Derivation of the Transfer Function Between Pbody andPgk . . . . . 119
C . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 123
D . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 129
E . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 133
F . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 135
```

### PDF page 15 | thesis page xv
<!-- raw_extraction: pdf_page=15; thesis_page=xv -->
```text
G . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 139
G.1 Aerodynamic Forces And Moments . . . . . . . . . . . . . . . . . . 139
G.1.1 Aerodynamic Forces And Moments for 6-DOF Model . . . . . 139
G.1.1.1 Lift Force . . . . . . . . . . . . . . . . . . . . . . . . . 139
G.1.1.2 Drag Force . . . . . . . . . . . . . . . . . . . . . . . . 140
G.1.1.3 Magnus Force . . . . . . . . . . . . . . . . . . . . . . 140
G.1.1.4 Pitching/Yawing Moment . . . . . . . . . . . . . . . . 140
G.1.1.5 Magnus Moment . . . . . . . . . . . . . . . . . . . . . 140
G.1.1.6 Pitch / Yaw Damping Moment . . . . . . . . . . . . . . 140
G.1.1.7 Roll Damping Moment . . . . . . . . . . . . . . . . . . 141
G.1.1.8 Gravitational Force . . . . . . . . . . . . . . . . . . . . 141
G.1.2 Aerodynamic Forces And Moments for 7-DOF Model . . . . . 141
G.1.2.1 Control Forces . . . . . . . . . . . . . . . . . . . . . . 141
G.1.2.2 Control Moments . . . . . . . . . . . . . . . . . . . . . 142
G.1.2.3 Aerodynamic Torque on Guidance Kit . . . . . . . . . 142
G.1.2.4 Aerodynamic Damping Torque on Guidance Kit . . . . 143
```

### PDF page 16 | thesis page xvi
<!-- raw_extraction: pdf_page=16; thesis_page=xvi -->
```text
LIST OF TABLES
TABLES
Table 2.1 Axial and Angular Velocities of 155 mm Spin-Stabilized Projectile . 9
Table 2.2 Mechanical Parts of the Actuator Part of Test Set-up . . . . . . . . . 17
Table 2.3 Speed Motor Specifications . . . . . . . . . . . . . . . . . . . . . . 18
Table 2.4 Torque Motor Specifications . . . . . . . . . . . . . . . . . . . . . 19
Table 2.5 Speed Motor Driver Specifications . . . . . . . . . . . . . . . . . . 20
Table 2.6 Torque Motor Driver Specifications . . . . . . . . . . . . . . . . . . 20
Table 2.7 Encoder Specifications . . . . . . . . . . . . . . . . . . . . . . . . 21
Table 2.8 Alternator Specifications . . . . . . . . . . . . . . . . . . . . . . . 21
Table 2.9 Slip Ring Specifications . . . . . . . . . . . . . . . . . . . . . . . . 22
Table 2.10 Filter Capacitor Specifications . . . . . . . . . . . . . . . . . . . . 22
Table 2.11 AC/DC Converter Specifications . . . . . . . . . . . . . . . . . . . 23
Table 2.12 Power Supply Specifications . . . . . . . . . . . . . . . . . . . . . 23
Table 2.13 Programmable DC Electronic Load Specifications . . . . . . . . . . 24
Table 2.14 Micro-controller Components . . . . . . . . . . . . . . . . . . . . . 24
Table 3.1 Open-Circuit Test Results . . . . . . . . . . . . . . . . . . . . . . . 40
Table 3.2 Closed-Circuit Test Results . . . . . . . . . . . . . . . . . . . . . . 41
```

### PDF page 17 | thesis page xvii
<!-- raw_extraction: pdf_page=17; thesis_page=xvii -->
```text
Table 4.1 Requirements of the Position Controller . . . . . . . . . . . . . . . 45
Table 4.2 Requirements of the Velocity Controller . . . . . . . . . . . . . . . 45
Table 4.3 Result of Velocity Controller Design . . . . . . . . . . . . . . . . . 53
Table 4.4 Result of Position Controller Design . . . . . . . . . . . . . . . . . 56
Table 4.5 Test Conditions . . . . . . . . . . . . . . . . . . . . . . . . . . . . 59
Table 5.1 Physical Properties and Initial Conditions . . . . . . . . . . . . . . 73
Table 6.1 Initial Conditions . . . . . . . . . . . . . . . . . . . . . . . . . . . 82
```

### PDF page 18 | thesis page xviii
<!-- raw_extraction: pdf_page=18; thesis_page=xviii -->
```text
LIST OF FIGURES
FIGURES
Figure 1.1 An Example Design of 1-D Control Approach by Using Drag
Brake . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 2
Figure 1.2 An Example Design Which Uses Movable Aerodynamic Surfaces . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 3
Figure 1.3 Orientation of Fixed Canard Structure . . . . . . . . . . . . . . 4
Figure 1.4 An Example Design Which Uses Fixed Aerodynamic Surfaces . 4
Figure 1.5 Schematics of Zhang, Li, Xiao and Tian’s Control Structure . . 5
Figure 1.7 Yin, Jia and Yu’s Control Structure . . . . . . . . . . . . . . . . 5
Figure 1.6 Zhang, Li, Xiao and Tian’s Test Set-up . . . . . . . . . . . . . . 6
Figure 1.8 Costello’s Projectile Schematic . . . . . . . . . . . . . . . . . . 6
Figure 2.1 An Example Projectile with Guidance Kit . . . . . . . . . . . . 10
Figure 2.2 Preliminary Design of Guidance Kit . . . . . . . . . . . . . . . 10
Figure 2.3 Section View of Preliminary Design of an Example Guidance Kit 11
Figure 2.4 Steering and Control Canards and Axis System . . . . . . . . . . 12
Figure 2.5 Torques at Fixed Canard Structure . . . . . . . . . . . . . . . . 13
Figure 2.6 Schematic of the Test Set-up . . . . . . . . . . . . . . . . . . . 15
Figure 2.7 General View of Test Set-up . . . . . . . . . . . . . . . . . . . 15
```

### PDF page 19 | thesis page xix
<!-- raw_extraction: pdf_page=19; thesis_page=xix -->
```text
Figure 2.8 Micro-controller Part . . . . . . . . . . . . . . . . . . . . . . . 16
Figure 2.9 AC/DC Converter and Filter Capacitor . . . . . . . . . . . . . . 16
Figure 2.10 Section View of the Test Set-up . . . . . . . . . . . . . . . . . 18
Figure 2.11 Zoomed View of Actuator Part . . . . . . . . . . . . . . . . . . 19
Figure 2.12 User Interface of the Test Software . . . . . . . . . . . . . . . . 25
Figure 2.13 Encoder Evaluation Part of the Test Software . . . . . . . . . . 26
Figure 2.14 Controller Part of the Test Software . . . . . . . . . . . . . . . 27
Figure 3.1 Physical Model of Test Set-up . . . . . . . . . . . . . . . . . . 29
Figure 3.2 Linear Graph Model of Test Set-up . . . . . . . . . . . . . . . . 30
Figure 3.3 System Tree of the Linear Graph Model . . . . . . . . . . . . . 31
Figure 3.4 Block Diagram of Simulink Model . . . . . . . . . . . . . . . . 33
Figure 3.5 Inside of Mechanical System Block . . . . . . . . . . . . . . . 34
Figure 3.6 Inside of Electrical System Block . . . . . . . . . . . . . . . . 34
Figure 3.7 The Differences of State Variables Between State Space and
Simulink Model . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 35
Figure 3.8 Free Body Diagram of Jgk . . . . . . . . . . . . . . . . . . . . . 36
Figure 3.9 Comparison Between Transfer Function Models and State Space
Results . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 39
Figure 4.1 Cascade Control Structure . . . . . . . . . . . . . . . . . . . . 44
Figure 4.2 Root Locus Graph of GIloadPgk . . . . . . . . . . . . . . . . . . 49
Figure 4.3 Root Locus Graph of Compensated Velocity Loop . . . . . . . . 50
Figure 4.4 Step Response of Velocity Loop . . . . . . . . . . . . . . . . . 51
```

### PDF page 20 | thesis page xx
<!-- raw_extraction: pdf_page=20; thesis_page=xx -->
```text
Figure 4.5 Bode Plot of Velocity Loop . . . . . . . . . . . . . . . . . . . . 51
Figure 4.6 Error Due to Step Disturbance at Taero . . . . . . . . . . . . . . 52
Figure 4.7 Error Due to Step Disturbance at pbody . . . . . . . . . . . . . . 53
Figure 4.8 Root Locus Graph of Golp . . . . . . . . . . . . . . . . . . . . 55
Figure 4.9 Step Response of Position Loop . . . . . . . . . . . . . . . . . 56
Figure 4.10 Bode Plot of Position Loop . . . . . . . . . . . . . . . . . . . . 57
Figure 4.11 Block Diagram of Simulation . . . . . . . . . . . . . . . . . . . 57
Figure 4.12 Velocity Controller Block . . . . . . . . . . . . . . . . . . . . . 58
Figure 4.13 Position Controller Block . . . . . . . . . . . . . . . . . . . . . 58
Figure 4.14 Inside of Calculate Minimum Control Effort Block . . . . . . . 59
Figure 4.15 Input, Simulation, and Test Results for Step Test . . . . . . . . . 60
Figure 4.16 Input, Simulation and Test Results for Step Test with Zoomed
View . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 60
Figure 4.17 Output of Position Controller at Step Test . . . . . . . . . . . . 61
Figure 4.18 Output of Position Controller at Step Test with Zoomed View . . 61
Figure 4.19 pgk at Simulation and Test Results . . . . . . . . . . . . . . . . 62
Figure 4.20 pgk at Simulation and Test Results with Zoomed View . . . . . . 63
Figure 4.21 Input, Simulation, and Test Results for Chip Test . . . . . . . . 63
Figure 4.22 Input, Simulation, and Test Results for Chip Test with Zoomed
View . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 64
Figure 4.23 Output of the Position Controller Results for Chip Test . . . . . 65
Figure 4.24 pgk at Simulation and Test Results . . . . . . . . . . . . . . . . 66
```

### PDF page 21 | thesis page xxi
<!-- raw_extraction: pdf_page=21; thesis_page=xxi -->
```text
Figure 5.1 Torques on the Guidance Kit . . . . . . . . . . . . . . . . . . . 67
Figure 5.2 Flat Earth Inertial Reference Frame . . . . . . . . . . . . . . . . 69
Figure 5.3 Overall Structure of 6-DOF Simulation . . . . . . . . . . . . . . 73
Figure 5.4 Positions at Earth Fixed Reference Frame . . . . . . . . . . . . 74
Figure 5.5 Velocities at Earth Fixed Reference Frame . . . . . . . . . . . . 75
Figure 5.6 Positions at Earth Fixed Reference Frame . . . . . . . . . . . . 76
Figure 5.7 Velocities at Earth Fixed Reference Frame . . . . . . . . . . . . 77
Figure 5.8 Control Forces at Body Fixed Plane Reference Frame . . . . . . 77
Figure 5.9 Control Moments at Body Fixed Plane Reference Frame . . . . . 78
Figure 5.10 Aerodynamic Torque Taero on Guidance Kit . . . . . . . . . . . 79
Figure 5.11 Aerodynamic Damping Torque Tad on Guidance Kit . . . . . . 79
Figure 6.1 Aerodynamic Torque for Selected Flight Conditions . . . . . . . 83
Figure 6.2 Body Angular Velocity for Selected Flight Conditions . . . . . . 83
Figure 6.3 Input and Output Positions for Unmodified Controller . . . . . . 84
Figure 6.4 Input and Output Velocities for Unmodified Controller . . . . . 85
Figure 6.5 Block Diagram of Disturbance Observer . . . . . . . . . . . . . 86
Figure 6.6 Real and Observed Disturbance Current . . . . . . . . . . . . . 87
Figure 6.7 Input and Output Positions for Dynamic Current Limit . . . . . . 88
Figure 6.8 Velocity Controller Result for Dynamic Current Limit and Observed Current . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 89
Figure 6.9 Input and Output Positions for Dynamic Velocity Limit . . . . . 90
Figure 6.10 Velocity Controller Result for Dynamic Velocity Limit . . . . . 91
```

### PDF page 22 | thesis page xxii
<!-- raw_extraction: pdf_page=22; thesis_page=xxii -->
```text
Figure 6.11 Velocity Controller Result for Dynamic Velocity Limit - Zoomed
View . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 91
Figure 6.12 Input and Output Positions for Combined Solution . . . . . . . . 92
Figure 6.13 Velocity Controller Result for Combined Solution . . . . . . . . 93
Figure 6.14 Velocity Controller Result for Combined Solution - Zoomed View 93
Figure 7.1 Input Position, Simulation, and Test Results for Unmodified
Controller . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 96
Figure 7.2 Input Position, Simulation, and Test Results for Unmodified
Controller-Zoomed View . . . . . . . . . . . . . . . . . . . . . . . . . 97
Figure 7.3 Input Position, Simulation, and Test Results for Unmodified
Controller-Zoomed View . . . . . . . . . . . . . . . . . . . . . . . . . 97
Figure 7.4 Alternator Saturation Points and Velocity Controller Results . . . 98
Figure 7.5 Input Position, Simulation, and Test Results for Changing Current Limit . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 99
Figure 7.6 Comparison of the Disturbance Observer Results . . . . . . . . 99
Figure 7.7 Input Position, Simulation, and Test Results for Changing Velocity Limit . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 100
Figure 7.8 Input Position, Simulation, and Test Results for Combined Solution . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 101
Figure B.1 Circuit of the Electrical Part . . . . . . . . . . . . . . . . . . . . 115
Figure B.2 Free Body Diagram of Jgk forIload . . . . . . . . . . . . . . . . 116
Figure B.3 Free Body Diagram of Jgk forTaero . . . . . . . . . . . . . . . . 118
Figure B.4 Free Body Diagram of Jgk forpbody . . . . . . . . . . . . . . . . 119
```

### PDF page 23 | thesis page xxiii
<!-- raw_extraction: pdf_page=23; thesis_page=xxiii -->
```text
Figure F.1 Comparison of Real and Reduced Transfer Functions . . . . . . 136
Figure F.2 Bode Plot of the Filter . . . . . . . . . . . . . . . . . . . . . . . 137
Figure F.3 Step Response of the Filter . . . . . . . . . . . . . . . . . . . . 137
```

### PDF page 24 | thesis page xxiv
<!-- raw_extraction: pdf_page=24; thesis_page=xxiv -->
```text
LIST OF ABBREVIATIONS
pbody: Angular velocity of the projectile or rocket body
pgk: Angular velocity of the guidance kit
Jbody: Inertia of the projectile or rocket body
Jgk: Inertia of the rotating parts of the guidance kit
b: Viscous friction coefficient
Taero:Aerodynamic torque at the guidance kit
Kt: Torque constant of the alternator
Kv: V oltage constant of the alternator
Ia: Current at the alternator
La: Phase to phase inductance of the alternator
Ra: Phase to phase resistance of the alternator
Cf: Filter capacitance
Iload: Load current
VCf ,VLa,VRa: V oltage drop atCf,La,Ra
δV : Resolution of velocity measurement
Kp: Static position error constant
Gvc: Transfer function of velocity controller
Golp: Open loop transfer function of position loop
d: Projectile reference diameter
p, q, r: Angular velocities in x, y, z direction
u, v, w: Linear velocities
L, M, N: Moments in x, y, z direction
Lgk-7DOF,LB-7DOF : Total torque in x-axis in 7-DOF
M7DOF,N 7DOF : Moments in y, z direction in 7-DOF
```

### PDF page 25 | thesis page xxv
<!-- raw_extraction: pdf_page=25; thesis_page=xxv -->
```text
X, Y , Z: Forces in x, y, z direction
X7DOF,Y 7DOF,Z 7DOF : Forces in x, y, z direction in 7-DOF
CLα: Lift force coefficient
CD Drag force coefficient
Cypα: Magnus force coefficient
Cmα: Pitching / Yawing moment coefficient
Cnpα: Magnus moment coefficient
Cmq: Ptich / Yaw damping moment coefficient
Clp: Roll damping moment coefficient
Crd: Roll damping moment coefficient of guidance kit
Crc: Roll moment coefficient of guidance kit
CNδ : Normal force coefficient of guidance kit
δs: Deflection angle of the steering canards
α: Incidence angle
β: Sideslip angle
ρ: Air density
θgk: Roll angle of the guidance kit
V: Magnitude of the projectile velocity
S: Reference area
L: Lift force
D: Drag force
K: Magnus force
Mpy: Pitching/Yawing moment
Mm: Magnus moment
Mpyd: Pitch/Yaw damping moment
Mrd: Roll damping moment
Fg: Gravitational force
```

### PDF page 26 | thesis page xxvi
<!-- raw_extraction: pdf_page=26; thesis_page=xxvi -->
```text
F: Total aerodynamic force for 6-DOF modeling
M: Total aerodynamic moment for 6-DOF modeling
Fc: Control forces generated by guidance kit
Mc: Control moments generated by guidance kit
lCG: Distance between pressure center of the steering canards
and center of mass of projectile
F7DOF : Total aerodynamic force for 7-DOF modeling
M7DOF : Total aerodynamic moment for 7-DOF modeling
Tad: Damping torque due to the angular velocity of the guidance kit at roll axis
Lgk-7DOF : Total aerodynamic torque on the guidance kit at the
roll axis
Crdgk: Roll damping moment coefficient of the guidance kit
ϕ,θ,ζ : Euler angles
DOF: Degree of freedom
m: Mass of the projectile
Q: Filter transfer function
ˆG: Reduced transfer function betweenIload andPgk
```

### PDF page 27 | thesis page 1
<!-- raw_extraction: pdf_page=27; thesis_page=1 -->
```text
CHAPTER 1
INTRODUCTION
1.1 Motivation and Problem Definition
Preventing civilian losses and having more precise strikes are increasing trends in
modern warfare environments [1]. Traditional artillery projectiles have disadvantages
due to low accuracy and precision in comparison to missiles and guided rockets [2].
And also having more precise strikes by using artillery projectiles requires adjustment
fire and observation and evaluation of the impact point location to correct the firing
parameters at the coming fire. This adjustment fire significantly decreases the chance
of destroying the target because of the time it consumes and it produces a risk at the
secrecy of the operation.
Developing a guided projectile requires navigation and guidance components which
need to be functional after gun launch and operational during flight. At the gun
launch, there is 15000g axial and 5000g radial acceleration exist and at the flight
phase, the projectile has an angular velocity of about 270 Hz [3].
The motivation behind this thesis is to develop a test system for the feasibility study
of the actuation system of a guidance kit that can be used at spin-stabilized munitions.
Also this thesis aims to develop a flight simulation of a spin-stabilized projectile to
have the necessary information for guidance kit design.
```

### PDF page 28 | thesis page 2
<!-- raw_extraction: pdf_page=28; thesis_page=2 -->
```text
1.2 Literature Survey
In the existing works, there are two commonly used approaches to convert existing
spin-stabilized artillery projectiles to guided munitions. One of them is a 1-D approach which has a brake mechanism at its structure. In this approach, the projectile
is aimed intentionally behind the target and the reduction existed by the use of the
drag brake in the range of the projectile is used to make a range correction [4]. One
of the examples of this 1-D correction drag brake is given in Figure 1.1. This is a
simple approach but it does not have the control ability to make side corrections.
Figure 1.1: An Example Design of 1-D Control Approach by Using Drag Brake
The other approach is making 2-D guidance by using movable canards or fixed canards. At the spinning projectiles, it is not possible to use conventional actuators due
to the high bandwidth requirement which is proportional with the angular velocity of
the projectile[4]. Therefore, it is found to be necessary to isolate the rotation of the
part that has actuators from the projectile body. We can divide the design that makes
2-D guidance into two. The first uses movable aerodynamic surfaces to have necessary forces and moments for guidance. An example of this is given in Figure 1.2.
This solution requires complicated actuators and drivers. Also, a battery is necessary
to supply power to control actuation system components. These are disadvantages
due to volume and mass limitations.
```

### PDF page 29 | thesis page 3
<!-- raw_extraction: pdf_page=29; thesis_page=3 -->
```text
Figure 1.2: An Example Design Which Uses Movable Aerodynamic Surfaces
There is also an interesting approach that uses fixed aerodynamic surfaces isolated
from the projectile body to make 2-D guidance [5]. This alternative uses 4 fixed
canards attached to an isolated part from the projectile body. The cant angles of
these canards are arranged such that 2 of them produce an aerodynamic torque in the
opposite direction of the angular velocity of the projectile body. The others produce a
net force that is required for guidance. The arrangement of the fixed canard structure
is shown in Figure 1.3 and an example of this type of design is given in Figure 1.4.
The actuator of this system is an alternator its rotor is attached to the fixed canard
structure and its stator is attached to the projectile body. This approach simplifies the
actuation problem to one axis and reduces the complexity of required components.
Since it is possible to generate power for other avionic components by using the
alternator, the battery requirement can be eliminated.
The actuator part of the fixed canard structure is examined by Zhang, Li, Xiao, and
Tian in 2016 for a mortar projectile [6]. The schematic of the used control structure is
given in Figure 1.5. Also in this work, the platform of the control system simulation
is explained. They used two motors to simulate the angular velocity of the mortar
munition and aerodynamic torque at the canards. They measured the angular velocity
of the fixed canard structure by using a gyroscope and used fourth-order Runge-Kutta
integral to calculate roll angle. The test platform they used is given in Figure 1.6.
A similar actuation mechanism for artillery projectiles is examined and test results
are given by Yin, Jia, and Yu in 2018. In this work, roll angle information of the
guidance kit body is taken by using a geomagnetic sensor, and a hall sensor is used
```

### PDF page 30 | thesis page 4
<!-- raw_extraction: pdf_page=30; thesis_page=4 -->
```text
Figure 1.3: Orientation of Fixed Canard Structure
Figure 1.4: An Example Design Which Uses Fixed Aerodynamic Surfaces
to get the position of the fixed-wing structure with respect to the projectile body [7].
A cascade structure is used to control the velocity and position of the fixed canard
structure. They used a proportional controller in the position loop and a proportionalintegral controller in the velocity loop. The closed-loop control system they used is
given in Figure 1.7.
```

### PDF page 31 | thesis page 5
<!-- raw_extraction: pdf_page=31; thesis_page=5 -->
```text
Figure 1.5: Schematics of Zhang, Li, Xiao and Tian’s Control Structure
Figure 1.7: Yin, Jia and Yu’s Control Structure
Flight dynamics of a projectile which has an isolated part from body rotation of the
projectile is examined by Costello in 1998 [8]. The projectile body is modeled as two
inertia elements that can rotate relative to each other and connected with bearings.
The projectile schematic used is given in Figure 1.8. In this figure, ΦF represents the
roll angle of the forward body and ΦA represents the roll angle of the aft body. And
```

### PDF page 32 | thesis page 6
<!-- raw_extraction: pdf_page=32; thesis_page=6 -->
```text
Figure 1.6: Zhang, Li, Xiao and Tian’s Test Set-up
forward body and aft body can rotate relatively.
Figure 1.8: Costello’s Projectile Schematic
In 2010, Wernert, Theodoulis, and Morel work on the equation of motion of a dualspin projectile in different reference frames [9]. They compared the computational
work required at every reference frame to choose the optimum frame for guidance
purposes in real-life. In 2012, Costello presented his work with some modifications
which make it possible to use an impact point prediction method for guidance purposes. In 2019, Zhang, Yao, and Zheng presented their work on the guidance of an
```

### PDF page 33 | thesis page 7
<!-- raw_extraction: pdf_page=33; thesis_page=7 -->
```text
artillery projectile by using fixed canards [10].
1.3 Contributions and Novelties
The thesis contributes to the literature in the parts listed below.
• Developing a test system to make the feasibility analysis of a 2-D guidance kit
actuation system that uses a fixed canard structure. A controller is designed and
tested for the actuator part of the guidance kit which mainly consists of position
control of an alternator.
• Dynamics of the spin-stabilized projectile is modeled and simulated in 6-DOF
and 7-DOF. By using this 7-DOF simulation, aerodynamic torque and body
angular velocity are found and they are applied using the test set-up to test
control ability through flight.
1.4 The Outline of the Thesis
Introduction, literature survey, and contributions to literature are given in Chapter 1.
Guidance methods for artillery projectiles are listed. The advantage and disadvantages of guidance methods are explained.
Explanation of the test system, list of the test hardware, and explanations of test
software are given in Chapter 2. The relationships between components of the system
are also explained in this section.
The mathematical model and simulations of the actuation system are given in Chapter
3. The results are presented and different modeling techniques are compared.
Controller design for the actuation system is given in Chapter 4. In this chapter,
the test results of the designed controller and the comparison of the test results with
simulation are also presented.
Modeling and simulation of flight dynamics of a spin-stabilized projectile are given
in Chapter 5. First, a 6-DOF model is established and the 6-DOF model is updated to
```

### PDF page 34 | thesis page 8
<!-- raw_extraction: pdf_page=34; thesis_page=8 -->
```text
7-DOF to include the additional degree of freedom.
The impact of the changing aerodynamic torque and body angular velocity are examined in the simulation environment in Chapter 6. Three different solutions are proposed to make the controller more robust to changes in the torque and body angular
velocity. The simulation results of the solutions are also explained in this chapter.
The test results of the modified controllers for changing aerodynamic torque and velocity are presented in Chapter 7.
Conclusion and future work are given in Chapter 8.
```

### PDF page 35 | thesis page 9
<!-- raw_extraction: pdf_page=35; thesis_page=9 -->
```text
CHAPTER 2
EXPLANATION OF THE TEST SYSTEM
2.1 Explanation of System Aim to be Tested
A projectile that contains a guidance kit is given in Figure 2.1 with axis convention. A
general spin-stabilized 155 mm artillery projectile has a large angular velocity on Xaxis to have gyroscopic stabilization. The angular velocity can change between 10000
rpm and 20000 rpm with the powder charge of the firing. The linear velocity of the
projectile increases with increasing powder charge and there is a linear relationship
between linear velocity and angular velocity at the barrel outlet. Angular velocities
of the projectile with respect to powder charge are given in Table 2.1. The direction
of the projectile rotation is +X-axis.
Table 2.1: Axial and Angular Velocities of 155 mm Spin-Stabilized Projectile
Powder Charge Initial Velocity (m/s) Initial Angular Velocity (Hz) Final Angular Velocity (Hz)
3M 546.4 176.3 128.2
4M 676.7 218.3 158.8
5M 812.4 262.1 190.6
6M 943.5 304.4 221.3
Preliminary design of a guidance kit is shown in Figure 2.2. The guidance kit mainly
consists of 3 parts which are the guidance kit body (Green part in Figure 2.2), the
nose of the projectile (Orange part in Figure 2.2), and the part that contains 4 fixed
canards (Blue part in Figure 2.2).
```

### PDF page 36 | thesis page 10
<!-- raw_extraction: pdf_page=36; thesis_page=10 -->
```text
Figure 2.1: An Example Projectile with Guidance Kit
Figure 2.2: Preliminary Design of Guidance Kit
Section view of Figure 2.2 is given in Figure 2.3. Guidance kit body and nose fixed
to each other. The green part is connected to the projectile body and rotates with the
body. The blue part is isolated from the projectile body by using two bearings and it
can rotate freely on X-axis. A permanent magnet synchronous alternator is attached
to the system as an actuator and power source for avionic components. The rotor of
the alternator is fixed to the blue part and the stator of the alternator is fixed to the
```

### PDF page 37 | thesis page 11
<!-- raw_extraction: pdf_page=37; thesis_page=11 -->
```text
green part. Since there is no stationary part in the system, it is not possible to use
classical rotor and stator convention. The part which has permanent magnets is called
as rotor and the part which has coils is called as stator. There are four fixed canards on
the outside of the rotor. The canards are fixed to the rotor, so they rotate with the blue
part. Therefore, the rotation of the canards on the X-axis is free from the projectile
body.
Figure 2.3: Section View of Preliminary Design of an Example Guidance Kit
A closer look at the canards is given in Figure 2.4. Two of the canards have the
same cant angle. These canards are shown at 0° and 180° in Figure 2.4. These
canards are called steering canards. The aerodynamic forces on these canards are
also shown in Figure 2.4 with yellow color. Since the forces on these canards have
the same direction, the torques generated by these forces on the X-axis cancel each
other and they do not form any net torque but they form a net force that is double of
the aerodynamic force at one of the steering canards. This net force will be used to
make 2-D guidance.
The other two canards have opposite cant angles which are shown at 90° and -90°
in Figure 2.4. The aerodynamic forces are also shown in Figure 2.4 with red color.
Since the directions of these two forces are opposite, they cannot form a net force but
they form a net torque on the X-axis.
```

### PDF page 38 | thesis page 12
<!-- raw_extraction: pdf_page=38; thesis_page=12 -->
```text
Figure 2.4: Steering and Control Canards and Axis System
The meanings of the symbols in Figure 2.4 are given below.
• Faero@steering canard: Total aerodynamic force acting on one of the steering
canards at the center of pressure.
• Faero@control canard: Total aerodynamic force acting on one of the control
canards at the center of pressure.
• r: The distance from the center of pressure of the control canard to the X-axis.
• R: The distance of the center of pressures of control canards.
The cant angles of the canards are adjusted such that the direction of Taero (-X-axis)
```

### PDF page 39 | thesis page 13
<!-- raw_extraction: pdf_page=39; thesis_page=13 -->
```text
and the rotation of the projectile body (+X-axis) are opposite to each other. Therefore,
we can form a torque in the opposite direction of theTaero by loading the alternator.
The guidance kit aims to adjust the direction of the net force at steering canards with
respect to the earth fixed reference frame by using the torques which are given in
Figure 2.5.
Figure 2.5: Torques at Fixed Canard Structure
The meanings of the symbols in Figure 2.5 and explanations of them are given below.
• Taero: Total aerodynamic torque formed by control canards. It is at -X direction.
Taero =f(ProjectileVelocity,Altitude ).
It depends on the altitude because the air density is significantly changing with
altitude.
```

### PDF page 40 | thesis page 14
<!-- raw_extraction: pdf_page=40; thesis_page=14 -->
```text
• Tf: Friction torque at the bearings. It is in the same direction with projectile
angular velocity. (+X)
Tf =f(RelativeVelocity, RadialandAxialLoad )
• Talt: The torque generated by the alternator by taking current on it. It is in the
same direction with projectile angular velocity.(+X)
Talt =f(LoadonAlternator )
This test set-up aims to make a feasibility analysis of the actuation system of the
guidance kit which mainly consists of position control of an alternator. The stator of
the alternator is connected to the projectile body and the rotor can rotate freely, under
the torques which are shown in Figure 2.5.
2.2 Explanation of Test Hardware
2.2.1 General Description of the Test Set-up
The main purpose of the test set-up is to have a test system that can simulate aerodynamic torque on the guidance kit and body angular velocity of the projectile or rocket;
therefore having a ground test opportunity for the actuation system of the guidance
kit.
The schematic of the test set-up is given in Figure 2.6. Also, the pictures of the test
set-up with the name of the components used is given in Figure 2.7, Figure 2.8, and
Figure 2.9.
```

### PDF page 41 | thesis page 15
<!-- raw_extraction: pdf_page=41; thesis_page=15 -->
```text
Figure 2.6: Schematic of the Test Set-up
Figure 2.7: General View of Test Set-up
```

### PDF page 42 | thesis page 16
<!-- raw_extraction: pdf_page=42; thesis_page=16 -->
```text
Figure 2.8: Micro-controller Part
Figure 2.9: AC/DC Converter and Filter Capacitor
This set-up simulates aerodynamic torque on the guidance kit with a BLDC motor
used in torque control mode. Also, the angular velocity of the projectile or rocket
body is simulated with another BLDC motor used in speed control mode. The stator
```

### PDF page 43 | thesis page 17
<!-- raw_extraction: pdf_page=43; thesis_page=17 -->
```text
of the alternator is fixed to the part that rotates with the speed motor. There is an
optical encoder between the rotor and the stator of the alternator to find the relative
position between the projectile or rocket body and the guidance kit. A slip ring is
used to transform the power of the alternator and encoder signals from rotating to the
stationary part. Data acquisition of feedback signals from motor drivers and encoder,
evaluation of the data to find control input and generation of output signals is done
using the micro-controller in the system.
2.2.2 Specifications of the Test Hardware
The section view of the test set-up is given in Figure 2.10 and a zoomed view of the
actuator part is given in Figure 2.11. The mechanical parts which have an inertial load
at the alternator are numbered and numerical values are given in Table 2.2.
Table 2.2: Mechanical Parts of the Actuator Part of Test Set-up
Part Number Value/Name/Type Inertia ( kgm 2)
1 Coupling 4.069× 10-5
2 Holding Part 1.678× 10-4
3 Alternator Body 7.55× 10-4
4 Alternator Magnets 1.485× 10-4
The components used in this set-up are listed below with their basic properties.
Speed Motor This is used to simulate the angular velocity of the projectile body.
Related specifications with speed motor are given in Table 2.3.
```

### PDF page 44 | thesis page 18
<!-- raw_extraction: pdf_page=44; thesis_page=18 -->
```text
Figure 2.10: Section View of the Test Set-up
Table 2.3: Speed Motor Specifications
Parameter Value/Name/Type Unit
Manufacturer Kollmorgen NA
Model AKM63M NA
V oltage 480 VAC
Torque Constant 1.23 Nm
A
Rated Power 4.95 kW
Rated Speed 4500 rpm
Rated Torque 10.5 Nm
Rotor Inertia 24 kgcm 2
```

### PDF page 45 | thesis page 19
<!-- raw_extraction: pdf_page=45; thesis_page=19 -->
```text
Figure 2.11: Zoomed View of Actuator Part
Torque Motor This is used to simulate the aerodynamic torque at the system. Related specifications with torque motor are given in Table 2.4.
Table 2.4: Torque Motor Specifications
Parameter Value/Name/Type Unit
Manufacturer Kollmorgen NA
Model AKM-33H NA
V oltage 240 VAC
Torque Constant 0.51 Nm
A
Rated Power 1.31 kW
Rated Speed 5500 rpm
Rated Torque 2.27 Nm
Rotor Inertia 0.85 kgcm 2
```

### PDF page 46 | thesis page 20
<!-- raw_extraction: pdf_page=46; thesis_page=20 -->
```text
Speed Motor Driver This component is used to control the speed motor. Related
specifications with speed motor driver are given in Table 2.5.
Table 2.5: Speed Motor Driver Specifications
Parameter Value/Name/Type Unit
Manufacturer Kollmorgen NA
Model AKD-P02407 NA
Continuous Current 24 A
Peak Current 48 A
Continuous Power 16 kW
Torque Motor Driver This component is used to control the torque motor. Related
specifications with torque motor driver are given in Table 2.6.
Table 2.6: Torque Motor Driver Specifications
Parameter Value/Name/Type Unit
Manufacturer Kollmorgen NA
Model AKD-P02207 NA
Continuous Current 12 A
Peak Current 30 A
Continuous Power 8 kW
Encoder This is used to measure the relative angle and velocity between the rotor
and the stator of the alternator. Related specifications with the encoder are given in
Table 2.7.
```

### PDF page 47 | thesis page 21
<!-- raw_extraction: pdf_page=47; thesis_page=21 -->
```text
Table 2.7: Encoder Specifications
Parameter Value/Name/Type Unit
Manufacturer FENAC NA
Sensing Type Optic NA
Encoder Type Incremental with index NA
Resolution 2500 CPT
Alternator This component is the actuator part of the guidance kit. Related specifications with this component are given in Table 2.8.
Table 2.8: Alternator Specifications
Parameter Value/Name/Type Unit
Manufacturer V olt Defence NA
Torque constant 0.716 Nm
A
Speed constant 0.749 V
rad/sec
Number of pole pairs 7 NA
Resistance(ph-ph) 6.2 Ohm
Inductance(ph-ph) 8.8 mH
Continuous Power 1.2 kW
Inertia of magnets 1.485 kgcm 2
Slip Ring This component is used to transfer alternator power and encoder signals
from the rotary part to the stationary part. Related specifications with slip ring are
given in Table 2.9.
```

### PDF page 48 | thesis page 22
<!-- raw_extraction: pdf_page=48; thesis_page=22 -->
```text
Table 2.9: Slip Ring Specifications
Parameter Value/Name/Type Unit
Manufacturer Senring NA
Number of circuit 12 NA
Maximum speed 5000 rpm
Maximum current 5 A
Filter Capacitor This component is used to filter transient components of voltage
which is produced due to the effect of the inductance of the alternator under sudden
changes in the current. The specifications related to the filter capacitor are given in
Table 2.10.
Table 2.10: Filter Capacitor Specifications
Parameter Value/Name/Type Unit
Manufacturer TDK NA
Maximum voltage 800 V
Capacitance 50 µF
Capacitance tolerance code K NA
AC/DC Converter This component is used to convert the AC voltage generated by
the alternator to DC voltage. Related specifications with AC/DC converter are given
in Table 2.11.
```

### PDF page 49 | thesis page 23
<!-- raw_extraction: pdf_page=49; thesis_page=23 -->
```text
Table 2.11: AC/DC Converter Specifications
Parameter Value/Name/Type Unit
Manufacturer Vishay NA
Model VS-36MT140 NA
Maximum continuous DC current 25 A
VRRM 1400 V
Power Supply This component is used to supply the required power for logic supplies of motor drivers, the micro controller, and the encoder. Related specifications
with power supply are given in Table 2.12.
Table 2.12: Power Supply Specifications
Parameter Value/Name/Type Unit
Manufacturer Aim TTI NA
Model CPX 400D NA
Maximum current 20 A
Maximum voltage 60 V
Continuous Power 420 W
Programmable DC Electronic Load This component is used to load the alternator
at the required current. The specifications related to this component are given in Table
2.13.
```

### PDF page 50 | thesis page 24
<!-- raw_extraction: pdf_page=50; thesis_page=24 -->
```text
Table 2.13: Programmable DC Electronic Load Specifications
Parameter Value/Name/Type Unit
Manufacturer Aim TTI NA
Model LD400P NA
Maximum current 80 A
Maximum voltage 80 V
Continuous Power 400 W
Short term power 600 W
Micro-controller Components These components are used for data acquisition
from other components, executing controller software, and producing control input.
Related specifications with these components are given in Table 2.14
Table 2.14: Micro-controller Components
Manufacturer Model Function
National Instruments NI-9063 Main controller
National Instruments NI-9205 Analog Input
National Instruments NI-9264 Analog Output
National Instruments NI-9411 Digital Input (Encoder)
2.3 Explanation of the Test Software
The test software is written by using Labview program which is developed by National Instruments company. The software consists of two main parts. The first part
is executed at the test computer and it is responsible for taking input and presenting
output data. The user interface of this part is given in Figure 2.12.
```

### PDF page 51 | thesis page 25
<!-- raw_extraction: pdf_page=51; thesis_page=25 -->
```text
Figure 2.12: User Interface of the Test Software
The second part is programmed by using FPGA target of the NI-9063 to have fast and
real-time execution. This part of the program is mainly responsible for finding the
velocity and position of the guidance kit by using encoder signals and speed motor
position data; executing the control algorithm and generating input signals for DC
electronic load, speed motor driver, and torque motor driver.
Data acquisition for encoder signals is done with NI-9411 digital input module. While
evaluating encoder data, in order not to take noises as a physical signal an input line
filter of 30µs is used while evaluating A and B line signals of the encoder; and 45µs
input line filter is used for the index signal. The measurement of the velocity is done
with 1 ms time intervals. The resolution of this measurement is given in Equation 2.1.
The software part which is responsible for the evaluation of encoder data is given in
Figure 2.13.
δV = 1
2500× 1000× 2π(rad/s)
δV = 24(rpm)
(2.1)
The controller algorithm is executed at 1kHz and the output is also updated at the
same frequency. The controller designed in Chapter 4 is applied in real-life by using
this part with saturation dynamics and determination of the direction of the error
```

### PDF page 52 | thesis page 26
<!-- raw_extraction: pdf_page=52; thesis_page=26 -->
```text
Figure 2.13: Encoder Evaluation Part of the Test Software
to apply minimum control effort. The software part which is responsible for the
execution of the control loop is given in Figure 2.14.
```

### PDF page 53 | thesis page 27
<!-- raw_extraction: pdf_page=53; thesis_page=27 -->
```text
Figure 2.14: Controller Part of the Test Software
```

### PDF page 54 | thesis page 28
<!-- raw_extraction: pdf_page=54; thesis_page=28 -->
*(Blank page.)*

### PDF page 55 | thesis page 29
<!-- raw_extraction: pdf_page=55; thesis_page=29 -->
```text
CHAPTER 3
MATHEMATICAL MODEL OF THE TEST SYSTEM
A detailed mathematical model of the test system is constructed and simulated in this
chapter. Different modeling techniques are used and the results are compared with
each other. Lastly, required transfer functions to make controller design is found and
the results of the transfer function model and the state space model are compared.
The representations of the physical components of the test system are given in Figure
3.1. Alternator at the system modeled as a voltage source with series connection of
resistance and inductance of the alternator. Filter capacitor connected to alternator
circuit in parallel connection. Viscous friction between rotating parts is modeled as a
damper and the damping coefficient is found by using test results. Lastly, the current
at DC electronic load is modeled as a current source at the system. At the real artillery
projectile, the angular velocity of the body is an initial condition. Since the test setup simulates this with a speed controlled BLDC motor, the angular velocity of the
projectile body is modeled as a source element.
Figure 3.1: Physical Model of Test Set-up
```

### PDF page 56 | thesis page 30
<!-- raw_extraction: pdf_page=56; thesis_page=30 -->
```text
3.1 State Space Model
The Linear Graph Method is used to find the system model systematically. The linear
graph model of the system is shown in Figure 3.2. To be consistent with notation used
in flight dynamics, angular velocities at the system are presented withp.
Figure 3.2: Linear Graph Model of Test Set-up
Parameters required to form the tree are given below:
• (Number of nodes)N = 7
• (Number of distinct graph)Nd= 2
• (Total number of branches)B = 11
• (Number of branches at tree)BT =N-Nd = 7- 2 = 5
• (Number of link)BL =B-BT = 11- 5 = 6
The procedure followed to form the tree is given below:
• Step 1: Include all across variable sources as tree branches.
pbody(t) is included. (1 branch)
```

### PDF page 57 | thesis page 31
<!-- raw_extraction: pdf_page=57; thesis_page=31 -->
```text
• Step 2: Include as many as A-type elements with one branch of the transformer.
Jgk,Cf, andT1 are included. (3 branches)
• Step 3: Complete the tree with resistance elements.
Ra is included and the required 5 branches are completed.
The tree of the system is given in Figure 3.3; tree branches are shown in red color,
and links are shown in black color.
Figure 3.3: System Tree of the Linear Graph Model
All primary and secondary variables of the system, elemental equations, compatibility
and continuity equations are given in Appendix A. The states of the system are pgk,
VCf (A-type energy storage elements across variables on the tree branches ), and ILa
(T-type energy storage element through variables on the links ). pgk refers to the
angular velocity of the guidance kit,VCf refers to the voltage difference between high
and low side of the capacitor, and ILa refers to the current on the alternator. The
equations in commonly used matrix form are given in Equation 3.1, Equation 3.2,
Equation 3.3, Equation 3.4, and Equation 3.5.
˙x = Ax + Bu (3.1)
```

### PDF page 58 | thesis page 32
<!-- raw_extraction: pdf_page=58; thesis_page=32 -->
```text
x =


pgk
VCf
ILa


(3.2)
u =


pbody(t)
Taero(t)
Iload(t)


(3.3)
A =


-b
Jgk
0 Kt
Jgk
0 0 1
Cf
-Kv
La
-1
La
-Ra
La


(3.4)
B =


b
Jgk
1
Jgk
0
0 0 -1
Cf
Kv
La
0 0


(3.5)
The state space model of the system is simulated with Matlab/Simulink environment
and the comparison of the results with other modeling techniques is given in Section
3.3. Related Matlab code is given in Appendix C.
```

### PDF page 59 | thesis page 33
<!-- raw_extraction: pdf_page=59; thesis_page=33 -->
```text
3.2 Simulink Model
To be sure of the mathematical model that is found by using the state space approach,
a Simulink model is also formed and the results of these models are compared in
Section 3.3.
The block diagram of the Simulink model is given in Figure 3.4. The model consists
of two main blocks called Mechanical System and Electrical System.
Figure 3.4: Block Diagram of Simulink Model
At the Mechanical System block, the torques on theJgk are taken as input and relative
angular velocity betweenpbody(t) andpgk is given as output. The inside of this block
is given in Figure 3.5.
```

### PDF page 60 | thesis page 34
<!-- raw_extraction: pdf_page=60; thesis_page=34 -->
```text
Figure 3.5: Inside of Mechanical System Block
At the Electrical System block, relative velocity and Iload(t) are taken as inputs and
VCf andILa are calculated and given as outputs. The inside of the Electrical System
block is given in Figure 3.6.
Figure 3.6: Inside of Electrical System Block
3.3 Comparison of the System Models
The Simulink model and the state space model are simulated with the same inputs and
the results are compared for system states which arepgk,VCf , andILa. The inputs of
the system are given in the list below.
```

### PDF page 61 | thesis page 35
<!-- raw_extraction: pdf_page=61; thesis_page=35 -->
```text
• pbody(t) = 100rad/sec
• Taero(t) =-0.75Nm
• Iload(t) = 1A
The differences between the state variables are given in Figure 3.7. As it is expected,
the Simulink model and the state space model are consistent with each other.
Figure 3.7: The Differences of State Variables Between State Space and Simulink
Model
3.4 Transfer Functions Between Inputs and Pgk
The free body diagram of theJgk is given in Figure 3.8.
```

### PDF page 62 | thesis page 36
<!-- raw_extraction: pdf_page=62; thesis_page=36 -->
```text
Figure 3.8: Free Body Diagram ofJgk
The torques at theJgk are explained in the list below:
• Talt(t) is the torque generated by loading the alternator.
Talt =KtIa and sinceIa depends on alternator voltage andIload(t) the function
becomes:
Talt =Talt(pbody(t),Taero(t),Iload(t))
• Tf is the total friction torque on theJgk.
Tf =Tf(pbody(t),Taero(t),Iload(t))
• Taero is the aerodynamic torque at the system. It is shown here in the positive
direction. Due to this, it will have negative values in the simulations.
Taero =Taero(t)
For control purposes, we need to find the transfer function between Iload and Pgk.
To simulate the system correctly, we also need to find the transfer functions between
system disturbances and system output. Therefore, we also need to find the transfer
functions betweenPbody andPgk, andTaero andPgk.
3.4.1 Transfer Function Between Iload andPgk
Derivation of the transfer function between Iload and Pgk is given in Appendix B.
Also, in this appendix consistency of the units is checked for coefficients of the same
power of s. The transfer function is given in Equation 3.6.
```

### PDF page 63 | thesis page 37
<!-- raw_extraction: pdf_page=63; thesis_page=37 -->
```text
GIloadPgk = β0
σ3s3 +σ2s2 +σ1s +σ0
β0 = Kt
LaCfJgk
σ0 = b
LaCfJgk
σ1 = KvKtCf +bRaCf +Jgk
LaCfJgk
σ2 = bLaCf +RaCfJgk
LaCfJgk
σ3 = 1
(3.6)
3.4.2 Transfer Function Between Taero andPgk
Derivation of the transfer function betweenTaero andPgk is given in Appendix B. The
transfer function is given in Equation 3.7.
GTaeroPgk = β2s2 +β1s +β0
σ3s3 +σ2s2 +σ1s +σ0
β0 = 1
JgkLaCf
β1 = Ra
JgkLa
β2 = 1
Jgk
σ0 = b
JgkLaCf
σ1 = bRaCf +KtKvCf +Jgk
JgkLaCf
σ2 = JgkRa +Lab
JgkLa
σ3 = 1
(3.7)
```

### PDF page 64 | thesis page 38
<!-- raw_extraction: pdf_page=64; thesis_page=38 -->
```text
3.4.3 Transfer Function Between Pbody andPgk
Derivation of the transfer function between Pbody and Pgk is given in Appendix B.
The transfer function is given in Equation 3.8.
Gpbody(t)Pgk = β2s2 +β1s +β0
σ3s3 +σ2s2 +σ1s +σ0
β0 = b
LaCfJgk
β1 = bRa +KtKv
LaJgk
β2 = b
Jgk
σ0 = b
LaCfJgk
σ1 = Jgk +bRaCf +KtKvCf
LaCfJgk
σ2 = RaJgk +bLa
LaJgk
σ3 = 1
(3.8)
The difference at pgk between the transfer function model and the state space model
is compared for the inputs given in Section 3.3. The result is given in Figure 3.9. As
expected, the results are consistent with each other.
```

### PDF page 65 | thesis page 39
<!-- raw_extraction: pdf_page=65; thesis_page=39 -->
```text
Figure 3.9: Comparison Between Transfer Function Models and State Space Results
3.5 Comparison of the Model Results and Test Results
3.5.1 Comparison of the Open-Circuit Test Results
In this case,pgk is kept at 0 and the speed motor is rotated at different speeds.Iload(t)
is also kept at 0 and the voltage at the output of the filterVCf is recorded with respect
topbody(t). The comparison of test results and the results of the simulations are given
in Table 3.1. In general test and simulation results are consistent with each other but
there is an increasing difference with relative velocity. It is possible to update the
model for changing Kv but since it is not important for control purposes it is kept
constant.
```

### PDF page 66 | thesis page 40
<!-- raw_extraction: pdf_page=66; thesis_page=40 -->
```text
Table 3.1: Open-Circuit Test Results
pbody VCf VCf Kv
(rad/s) Test Result (V) Simulation Result (V) Test Result (rad/s
V )
10.5 7.8 7.92 0.74
20.9 16.0 15.7 0.76
31.4 24.6 23.6 0.78
41.9 33.3 31.4 0.79
52.4 41.9 39.3 0.8
62.8 50.4 47.1 0.8
73.3 58.9 55.0 0.8
3.5.2 Comparison of the Closed-Circuit Test Results
In this case,pgk is kept at 0 and the speed motor is rotated at different speeds.Iload(t)
is also controlled. The voltage at the output of the filter VCf andTgk is recorded with
respect topbody(t) andIload(t).
The comparison of the test results and the results of the simulations is given in Table
3.2. In this table, Tgk refers to the total torque generated by the alternator, which
means the friction torque is eliminated. In the test results, there is a small change in
Kt with changing current. Since the current measurements of the torque motor driver
are used to estimate the torque generated by the alternator, the reason behind this can
be offset and non-linearity at current measurement.
```

### PDF page 67 | thesis page 41
<!-- raw_extraction: pdf_page=67; thesis_page=41 -->
```text
Table 3.2: Closed-Circuit Test Results
pbody VCf VCf Tgk Iload Kt
(rad/s) Test Result(V) Simulation Result(V) (Nm) Test Result(A) (N/m
A )
20.9 12.0 12.6 0.37 0.5 0.73
20.9 9.2 9.5 0.72 1 0.72
20.9 6.7 6.4 1.03 1.5 0.69
41.9 28.3 28.3 0.39 0.5 0.77
41.9 25.2 25.2 0.74 1 0.74
41.9 22.2 22.2 1.07 1,5 0.71
41.9 19.6 19 1.44 2 0.72
41.9 16.6 15.9 1.78 2.5 0.71
62.8 44.6 44.0 0.37 0.5 0.75
62.8 41.2 40.9 0.73 1 0.73
62.8 38.0 37.8 1.08 1.5 0.72
62.8 34.8 34.7 1.42 2 0.71
62.8 31.8 31.6 1.75 2.5 0.70
83.8 60.4 59.7 0.37 0.5 0.73
83.8 56.7 56.6 0.72 1 0.72
83.8 53.4 53.5 1.06 1.5 0.71
83.8 50.2 50.4 1.42 2 0.71
83.8 46.5 47.5 1.75 2.5 0.70
```

### PDF page 68 | thesis page 42
<!-- raw_extraction: pdf_page=68; thesis_page=42 -->
*(Blank page.)*

### PDF page 69 | thesis page 43
<!-- raw_extraction: pdf_page=69; thesis_page=43 -->
```text
CHAPTER 4
CONTROLLER DESIGN
There are many controller types used in servo control and other control problems.
Traditional PID controllers are still the most common controller type in the industry
despite many other controller types being erased after that [11]. It is possible to tune
PID controllers using traditional time and frequency domain approaches such as root
locus and bode plots, and in addition to these, it is possible to tune PID controllers
on-site by using some automatic tuning algorithms. PID controllers generally give
not the best but sufficient results with system dynamics, actuator saturation, and the
existence of the disturbance [12].
In general, PID controllers have three terms which are P - proportional with the error
signal, I - proportional with the integral of the error signal, and D - proportional
with the derivative of the error signal. I - action is generally used for eliminating
steady-state error and D - action is generally used for improving the stability of the
closed-loop system [13].
The time-domain control input for a PID controller is given in Equation 4.1.
u(t) =Kpe(t) +Ki
Z
e(t)dt +Kd
de(t)
dt (4.1)
Equation 4.1 in the Laplace domain is given in Equation 4.2.
U(s) =KpE(s) +Ki
E(s)
s +KdsE(s) (4.2)
Since derivative control amplifies measurement noise [14] it is generally filtered with
a first or second-order low pass filter. To prevent amplification of the measurement
```

### PDF page 70 | thesis page 44
<!-- raw_extraction: pdf_page=70; thesis_page=44 -->
```text
noise, derivative action will be the last choice in the controller design.
PID controller structure has some modifications in applications such as cascade control. It is possible to use a cascade control structure if there are more than one measurable states in the system but there is only one of them wanted to be controlled [15]. It
is also a suitable control method if it is wanted to eliminate disturbances in the inner
loop before they have an impact on the controlled parameter [16]. A cascade control
structure is suitable for our system because we want to control the position ofJgk and
we want to eliminate the impact of the disturbances which are Taero(t) andpbody(t).
The cascade structure which will be used at the control of the θgk is given in Figure
4.1. In this block diagram, Gd1 represents the transfer function between Taero and
Pgk, Gd2 represents the transfer function between Pbody andPgk, and Gu represents
the transfer function betweenIload andPgk.
Figure 4.1: Cascade Control Structure
4.1 Requirements for Position and Velocity Controllers
While tuning the cascade controllers, to have sufficient results in the outer loop, the
dynamics of the inner loop need to be at least 5 times faster than the outer loop [17].
So, we first need to determine the requirements of the position controller and need
to find the requirements of the velocity controller with respect to position controller
requirements.
The settling time of the position controller is given as 0.1 s in the literature [10]. Also
at this reference, the position, θgk is examined during flight. And it states that high
```

### PDF page 71 | thesis page 45
<!-- raw_extraction: pdf_page=71; thesis_page=45 -->
```text
dynamics in the reference position is required when the projectile is on the required
trajectory, and no need to make guidance to hit the target. Therefore, the position
of the fixed canard structure does not need to be controlled after this point and it
is allowed to rotate freely. Therefore, a relatively slow cut-off frequency, 4 Hz is
sufficient for the position loop. Also, overshoot is not desired at the position loop and
steady-state error needs to be zero for a step input. The requirements of the position
loop are given in Table 4.1.
Table 4.1: Requirements of the Position Controller
Parameter Value Unit
Minimum cut-off frequency 4 Hz
Static position error constant ∞ NA
Maximum settling time 0.1 s
Maximum overshoot 5 %
Requirements of the velocity controller are determined by referencing position loop
requirements. The overshoot of the system is not as critical as the position loop. The
requirements for the velocity loop are given in Table 4.2.
Table 4.2: Requirements of the Velocity Controller
Parameter Value Unit
Minimum cut-off frequency 20 Hz
Static position error constant ∞ NA
Maximum settling time, %5 (Ts) 0.05 s
Maximum overshoot (Mp) 25 %
4.2 Velocity Controller Design
We can determine the locations of the dominant poles by using a second-order system
assumption. The standard form of a second-order system transfer function between
```

### PDF page 72 | thesis page 46
<!-- raw_extraction: pdf_page=72; thesis_page=46 -->
```text
input R(s) and output C(s) is given in Equation 4.3. The dynamics of a second-order
system can be expressed in terms ofωn andζ.
C(s)
R(s) = ω2
n
s2 + 2ζωns +ω2
n
(4.3)
We can find ζ by using the maximum overshoot requirement and the relationship
given in Equation 4.4.
Mp =e
-( σ
ωd
)π
σ =ζωn
ωd =wn
p
1-ζ 2
(4.4)
ωn can be found by using the settling time requirement and the relationship is given
in Equation 4.5.
ts = 3
σ
σ =ζωn
(4.5)
The closed-loop transfer function of the velocity loop and the transfer function between error and reference velocity are given in Equation 4.6.
Gclv =
Pgkf
Pgkr
= GvcGIloadPgk
1 +GvcGIloadPgk
Ev
Pgkr
= 1-
Pgkf
Pgkr
= 1
1 +GvcGIloadPgk
Ev = Pgkr
1 +GvcGIloadPgk
(4.6)
To choose the velocity controller type, steady-state requirements need to be checked.
The Final Value Theorem is a useful tool for checking steady-state requirements in
the Laplace domain [11]. This theorem states that the final value of a function can be
found by using Equation 4.7.
```

### PDF page 73 | thesis page 47
<!-- raw_extraction: pdf_page=73; thesis_page=47 -->
```text
lim
s→0
sF (s) = lim
t→∞
f(t) (4.7)
Static position error constant is found in Equation 4.8 by using the Final Value Theorem and replacing the Laplace transform of the unit step function, 1
s, with Pgkr in
Equation 4.6.
lim
s→0
sF (s) = lim
s→0
s1
s
1
1 +GvcGIloadpgk
Kp = 1
Gvc(0)GIloadpgk(0)
(4.8)
If Equation 3.6, which gives GIloadPgk, written in Equation 4.8, static position error
constant,Kp becomes:
Kp = 1
(Gcv(0))Kt
b
(4.9)
To have zero steady-state error for a unit step input, Kp needs to be∞. Therefore
there should be a free integrator at the controller, so a PI controller is a suitable choice.
The transfer function of the velocity controller is given in Equation 4.10.
Gvc =Kpv + Kiv
s (4.10)
Sincepbody(t) andTaero(t) act on the system as disturbances, it is required to check
the steady-state error of the system due to these disturbances. Closed-loop transfer
functions between Pbody and Pgk are found in Equation 4.11 by equating reference
input to zero.
Pgkf = (Pgkr-Pgkf )GvcGIloadPgk +PbodyGPbodyPgk
Pgkf (1 +GcvGIloadPgk) =PbodyGPbodyPgk
Pgkf
Pbody
= GPbodyPgk
1 +GvcGIloadPgk
(4.11)
```

### PDF page 74 | thesis page 48
<!-- raw_extraction: pdf_page=74; thesis_page=48 -->
```text
Since reference input equated to zero, the error can be found by multiplying the transfer function found in Equation 4.11 with the disturbancePbody. Equation of error due
toPbody is given in Equation 4.12.
Ev(s) =- GPbodyPgk
1 +GvcGIloadPgk
Pbody (4.12)
If we use Equation 3.6 and Equation 3.8 in Equation 4.12 then apply Final Value
Theorem for a step disturbance input the following equation will be found.
lim
s→0
sEv(s) =-s GPbodyPgk(0)
1 +Gvc(0)GIloadPgk(0)
1
s = 0
GPbodyPgk(0) = 1
GIloadPgk(0) = Kt
b
Gvc(0) =∞
(4.13)
Since there is a free integrator at the velocity controller, the disturbance error due to
pbody is zero at steady-state.
If the same procedure is applied forTaero, Equation 4.14 is found.
Ev(s) =- GTaeroPgk
1 +GvcGIloadPgk
Taero (4.14)
Using Equation 3.6 and Equation 3.7 in Equation 4.14 and applying Final Value Theorem for a step input toTaero following equation for steady-state error can be found.
lim
s→0
sEv(s) =-s GTaeroPgk(0)
1 +Gvc(0)GIloadTaero(0)
1
s = 0
GTaeroPgk(0) = 1
b
GIloadPgk(0) = Kt
b
Gvc(0) =∞
(4.15)
As it could be expected, the disturbance error due toTaero is zero at steady-state.
```

### PDF page 75 | thesis page 49
<!-- raw_extraction: pdf_page=75; thesis_page=49 -->
```text
Root-Locus of the GIloadPgk is given in Figure 4.2. The transfer function has three
poles and the locations of the poles are written below.
Figure 4.2: Root Locus Graph ofGIloadPgk
• s1 = -1.04
• s2 = -352.28 + 1483.20i
• s3 = -352.28 - 1483.20i
If we make a second-order system assumption,ζ andωn can be found by using Equation 4.4 and Equation 4.5. The desired locations of the closed-loop poles are given in
Equation 4.16.
s1 =-ζωn +ωn
p
1-ζ 2i =-60 + 136i
s2 =-ζωn-ωn
p
1-ζ 2i =-60- 136i
(4.16)
PI controller requires the addition of an integrator and a zero to the open-loop transfer
function. Since the addition of the integrator shifts the root-locus plot to the right,
```

### PDF page 76 | thesis page 50
<!-- raw_extraction: pdf_page=76; thesis_page=50 -->
```text
zero should be added near the pole to compensate for the effect of the integrator,
and the general shape of the root-locus plot will not change. The pole located at -
1.03 will move towards to -∞ with increasing gain. The desired locations of the
closed-loop poles which are found with the second-order system assumption are less
dominant than the pole located at -1.03 if the gain is not high enough to go beyond
-60, therefore we need to check the step response of the system to see the effect of
the designed controller. With the addition of a zero at -26.98 with an integrator, the
root-locus graph of the system is given in Figure 4.3.
Figure 4.3: Root Locus Graph of Compensated Velocity Loop
If we select gain as 0.29 we would have 2 real poles and 2 complex conjugate poles, so
the system becomes a combination of one overdamped and one underdamped system.
The pole locations of the closed-loop system are given below.
• s1 = -33.17
• s2 = -144.78
• s3 = -263.82 + 1468.48i
```

### PDF page 77 | thesis page 51
<!-- raw_extraction: pdf_page=77; thesis_page=51 -->
```text
• s3 = -263.82 - 1468.48i
The step response of the designed controller is given in Figure 4.4 and Bode graph is
given in Figure 4.5.
Figure 4.4: Step Response of Velocity Loop
Figure 4.5: Bode Plot of Velocity Loop
```

### PDF page 78 | thesis page 52
<!-- raw_extraction: pdf_page=78; thesis_page=52 -->
```text
Also, we need to check the behavior of the controller to system disturbances. The
errors due to step disturbance for Taero andPbody are given in Figure 4.6 and Figure
4.7.
Figure 4.6: Error Due to Step Disturbance atTaero
The errors due to disturbances are zero at steady-state as it is expected.
Since the sources and capability of the actuators are limited, the output of the controller needs to be saturated. Limitation of the controller output can cause overshoot
in the system, therefore an anti-windup technique should be used. There are two
mainly used integral windup techniques in the literature which are "Back Calculation
and Tracking Technique" and "Clamping Technique" [12]. The clamping integration
windup technique is selected due to its easy-to-apply algorithm. The windup block is
given in Section 4.4.
The comparison of the requirements and designed controller is given in Table 4.3. As
can be seen from the table, the designed controller satisfies the requirements.
```

### PDF page 79 | thesis page 53
<!-- raw_extraction: pdf_page=79; thesis_page=53 -->
```text
Figure 4.7: Error Due to Step Disturbance atpbody
Table 4.3: Result of Velocity Controller Design
Parameter Requirement Result Unit
Minimum cut-off frequency 20 33.6 Hz
Static Position Error Constant ∞ ∞ NA
Maximum settling time, %5 (Ts) 0.05 0.05 s
Maximum overshoot (Mp) 25 9.57 %
The transfer function of the velocity controller is given in Equation 4.17.
Gvc =Kpv + Kiv
s
Gvc = 0.29 + 7.82
s
(4.17)
```

### PDF page 80 | thesis page 54
<!-- raw_extraction: pdf_page=80; thesis_page=54 -->
```text
4.3 Position Controller Design
Desired closed-loop pole locations of the position loop can be found by making
second-order system assumption and using Equation 4.4 and Equation 4.5. The resulting closed loop-pole locations are given below.
s1 =-ζωn +ωn
p
1-ζ 2i =-30.00 + 31.47i
s2 =-ζωn-ωn
p
1-ζ 2i =-30.00- 31.47i
(4.18)
The open-loop transfer function between input and output position is given in Equation 4.19.
Golp =GpcGclv
1
s (4.19)
Since there is a free integrator in the open-loop transfer function according to Final
Value Theorem, the static position error constant is∞. Therefore there is no need to
add an integrator to the controller. Also, the impacts of the disturbances are eliminated
in the inner velocity loop, therefore there is no need to check the position controller
for disturbance rejection. So, a proportional controller is enough for the position loop.
Root-locus graph for the position loop is given in Figure 4.8.
```

### PDF page 81 | thesis page 55
<!-- raw_extraction: pdf_page=81; thesis_page=55 -->
```text
Figure 4.8: Root Locus Graph ofGolp
Locations of the open-loop poles are listed below.
• s1 = 0
• s2 = -33.17
• s3 = -144.78
• s4 = -263.82 + 1468.48i
• s5 = -263.82 - 1468.48i
As it can be seen from Figure 4.8 one of the closed-loop poles will always remain on
the right side of the desired pole locations, therefore the behavior of the system will
be dominated by this pole. Gain is adjusted such that the location of this pole will not
slow down the system, with a time constant smaller than 0.05. If we adjust the gain
to 29.7, the locations of the closed-loop poles are given in the list below.
• s1 = -21.03
• s2 = -78.00 + 26.7i
• s3 = -78.00 - 26.7i
```

### PDF page 82 | thesis page 56
<!-- raw_extraction: pdf_page=82; thesis_page=56 -->
```text
• s4 = -264.29 + 1466.71i
• s5 = -264.29 - 1466.71i
The step response of the position loop is given in Figure 4.9 and the Bode plot of the
position loop is given in Figure 4.10.
Figure 4.9: Step Response of Position Loop
The comparison of the position controller requirements and designed position controller is given in Table 4.4. Since the system is dominated by the pole near to imaginary axis, the behaviour of the system is like a first-order system.
Table 4.4: Result of Position Controller Design
Parameter Requirement Result Unit
Minimum cut-off frequency 4 6.2 Hz
Static Position Error Constant ∞ ∞ NA
Maximum settling time, %5 (Ts) 0.1 0.1 s
Maximum overshoot (Mp) 5 0 %
```

### PDF page 83 | thesis page 57
<!-- raw_extraction: pdf_page=83; thesis_page=57 -->
```text
Figure 4.10: Bode Plot of Position Loop
4.4 Test Results and Comparison
The designed controller is simulated with Matlab / Simulink. The block diagram of
the simulation is given in Figure 4.11. The block diagram mainly consists of the position and velocity controllers, and the transfer functions represent system dynamics.
Figure 4.11: Block Diagram of Simulation
```

### PDF page 84 | thesis page 58
<!-- raw_extraction: pdf_page=84; thesis_page=58 -->
```text
Inside of the velocity controller block is given in Figure 4.12. Clamping integration
windup algorithm is also implemented in this block with saturation points.
Figure 4.12: Velocity Controller Block
The inside of the position controller block is given in Figure 4.13. This block also
contains a part which is called "Calculate Minimum Control Effort" which determines
the direction that needs to be rotated to have minimum control effort. Also, the inside
of this block is given in Figure 4.14.
Figure 4.13: Position Controller Block
```

### PDF page 85 | thesis page 59
<!-- raw_extraction: pdf_page=85; thesis_page=59 -->
```text
Figure 4.14: Inside of Calculate Minimum Control Effort Block
The inputs and saturation points in the test case are given in Table 4.5.
Table 4.5: Test Conditions
Input Value Unit
pbody 83.8 rad/s
pgkmax 20.9 rad/s
pgkmin -20.9 rad/s
Iloadmax 1.9 A
Iloadmin 0 A
4.4.1 Comparison of the Step Test Results
At this test, step input positions are given to the system. Input position, test results,
and simulation results are given in Figure 4.15. A zoomed view of this figure is given
in Figure 4.16. As it can be seen from these figures, simulation results and test results
are consistent with each other. While the system is stationary, there are oscillations at
the position in test results that do not exist in simulation results. The reason behind
this is measurement error in encoder signals. Also, non-linear friction in the test
set-up can cause this situation.
```

### PDF page 86 | thesis page 60
<!-- raw_extraction: pdf_page=86; thesis_page=60 -->
```text
Figure 4.15: Input, Simulation, and Test Results for Step Test
Figure 4.16: Input, Simulation and Test Results for Step Test with Zoomed View
Position controller outputs for test and simulation results are given in Figure 4.17 and
a zoomed view of this figure is given in Figure 4.18.
```

### PDF page 87 | thesis page 61
<!-- raw_extraction: pdf_page=87; thesis_page=61 -->
```text
Figure 4.17: Output of Position Controller at Step Test
Figure 4.18: Output of Position Controller at Step Test with Zoomed View
```

### PDF page 88 | thesis page 62
<!-- raw_extraction: pdf_page=88; thesis_page=62 -->
```text
pgk at simulation and test results are given in Figure 4.19 and zoomed view of this
figure is given in Figure 4.20. The results are consistent with each other when the
system is moving but there are fluctuations in test results while the system is stationary. The reasons for this behavior are measurement errors and the resolution of the
velocity measurements.
Figure 4.19:pgk at Simulation and Test Results
4.4.2 Comparison of the Chirp Test Results
At this test, a chirp signal is given to the system as a reference position. The amplitude
of the given signal is pi/8 and the frequency of the signal is changing from 0.01 to 7.5
Hz in 15s. The sinusoidal signal is applied with pi rad offset. Input position, output
position of simulation, and test results for the chirp test are given in Figure 4.21.
A zoomed view of Figure 4.21 is given in Figure 4.22. Test results are consistent
with simulation results. The amplitude at the cut-off frequency is 3.4192 rad with
the addition of pi rad offset. The response of the system reduced to this amplitude
after 12.5s. The corresponding frequency is 6.23 Hz which is consistent with design
results.
```

### PDF page 89 | thesis page 63
<!-- raw_extraction: pdf_page=89; thesis_page=63 -->
```text
Figure 4.20:pgk at Simulation and Test Results with Zoomed View
Figure 4.21: Input, Simulation, and Test Results for Chip Test
```

### PDF page 90 | thesis page 64
<!-- raw_extraction: pdf_page=90; thesis_page=64 -->
```text
Figure 4.22: Input, Simulation, and Test Results for Chip Test with Zoomed View
The output of the position controller is compared in Figure 4.23. Also, feedback
velocities are compared in Figure 4.24. The difference between test results and simulation results is larger than the position difference since as an inner loop, the velocity
controller compensates for model uncertainties. The uncertainties mainly come from
non-linear Coulomb friction and the change in friction coefficient due to misalignment of center lines of torque and velocity motors.
.
```

### PDF page 91 | thesis page 65
<!-- raw_extraction: pdf_page=91; thesis_page=65 -->
```text
Figure 4.23: Output of the Position Controller Results for Chip Test
```

### PDF page 92 | thesis page 66
<!-- raw_extraction: pdf_page=92; thesis_page=66 -->
```text
Figure 4.24:pgk at Simulation and Test Results
```

### PDF page 93 | thesis page 67
<!-- raw_extraction: pdf_page=93; thesis_page=67 -->
```text
CHAPTER 5
FLIGHT DYNAMICS OF THE ARTILLERY PROJECTILE
The torques on the fixed canard structure are given in Figure 5.1. It is possible to
neglect aerodynamic damping torque while the angular velocity of the guidance kit is
low and it is the case when position control is active. As it can be seen from Figure
5.1, the torque in the direction of body angular velocity is the sum of the friction
torque and alternator torque. It is possible to control the torque generated by the
alternator, but there is no control ability on friction torque. In the opposite direction
with body angular velocity, aerodynamic torque is available but it is the output of the
flight conditions that we have no control.
Figure 5.1: Torques on the Guidance Kit
```

### PDF page 94 | thesis page 68
<!-- raw_extraction: pdf_page=94; thesis_page=68 -->
```text
The torques at the +X and -X directions are given in Equation 5.1 and Equation 5.2.
T+x =Talt +Tf-Taero (5.1)
T-x =-Talt-Tf +Taero (5.2)
As it can be seen from equations 5.1 and 5.2, friction torque needs to be smaller than
aerodynamic torque to have control ability in the -X direction.
To find aerodynamic torque and friction torque in the system, a 6-DOF mathematical
model is constructed and the model is extended to 7-DOF.
5.1 Coordinate Frames
As an inertial frame, a flat Earth reference frame is used. This frame does not include
the curvature of the Earth’s surface. Earth Fixed Reference Frame which is used as
the inertial frame is given in Figure 5.2.
There are three possible coordinate systems to make 6-DOF simulation of a spinning
projectile. These coordinate systems and their advantages & disadvantages are given
in the subsections below.
5.1.1 Body Coordinate Frame
In general, to model an air vehicle, the Body Coordinate System is used. In this
coordinate system, the origin is placed at the Cg of the vehicle. X-axis points directly
to the nose, Y-axis is perpendicular to X-axis, and Z-axis is found by using the righthand rule. This frame is rolling with the projectile therefore it has the same angular
and linear velocity with the projectile. Since the computational power required to
simulate a model directly depends on the highest frequency of the system, the change
in roll angle while using the body coordinate frame leads to very long simulation
```

### PDF page 95 | thesis page 69
<!-- raw_extraction: pdf_page=95; thesis_page=69 -->
```text
Figure 5.2: Flat Earth Inertial Reference Frame
times. Therefore defining a middle coordinate system that has no rolling motion is
useful[9] [18].
The rotational velocity of a reference frame can be expressed by using following
ways:
⃗ wF = ˙ϕ⃗ x3 + ˙θ⃗ y2 + ˙ψ⃗ z1
⃗ wF =pF⃗ xBR +qF⃗ yBR +rF⃗ zBR
F subscript indicates "Frame", BR indicates "Body Reference Frame", 1, 2, and 3
indicate "Middle reference frames" in Euler rotation sequences.
No rolling can be guaranteed by using two approaches. The first is having zero roll at
angular velocity components of the reference frame and the other is having zero roll
rate at the Euler angles [19].
```

### PDF page 96 | thesis page 70
<!-- raw_extraction: pdf_page=96; thesis_page=70 -->
```text
5.1.2 Body Fixed Plane Coordinate System
This coordinate system has no change in the Euler roll angle. In other words, if the
initial roll angle is equal to zero,ϕ and ˙ϕ are zero through to flight time.
The relationship between change in Euler angles and angular velocity of a reference
frame is given below [20].


˙ϕ
˙θ
˙ψ

 =


1 tan θ sinϕ tanθ cosϕ
0 cos ϕ sinϕ
0 sec θ sinϕ secθ cosϕ




pF
qF
rF

 (5.3)
If we extract ˙ϕ from the Equation 5.3:
˙ϕ =pF +qF tanθ sinϕ +rF tanθ cosϕ (5.4)
If we substitute 0 toϕ and ˙ϕ in the equation above we get:
0 =pF +rF tanθ
pF =-rF tanθ
(5.5)
WhereqF andrF are the same with angular velocities of the body which is written
in the Body Fixed Plane coordinate system. So, it is possible to eliminate subscript F
and use q, and r for angular velocity components of the Y and Z axis of the coordinate
frame.
The advantage of using this reference frame is Y-axis always remains parallel to the
Earth’s surface. This property will be useful for guidance and control purposes, therefore this reference frame will be used for simulation [9].
5.1.3 Body Non-Spinning Frame
This reference frame assumes thatpF = 0. This is simpler than the non-rolling body
frame and requires less computational time.
```

### PDF page 97 | thesis page 71
<!-- raw_extraction: pdf_page=97; thesis_page=71 -->
```text
5.2 Mathematical Model and Simulation of 6-DOF Projectile Dynamics
The mathematical model of spin-stabilized projectile dynamics is contracted with
some simplifications. The Earth’s surface is assumed to be flat which means the
curvature of the Earth’s surface is neglected. Also, the change in gravitational force
with respect to altitude is neglected. In general, aerodynamic coefficients change
with changing velocity, incidence angle, and sideslip angle but it is assumed that the
coefficients are only changing with respect to Mach number. Since the aim of these
simulations is to find aerodynamic torque and body velocity, it is possible to make
these assumptions. To use this model for guidance purposes these simplifications
need to be included in the model.
5.2.1 6-DOF Dynamical Model in Body Fixed Plane Coordinate System
The explanations and formulas related to forces and moments are given in Appendix
G. These equations are also valid in the Non-rolling Body Fixed Plane Reference
Frame, but now incidence and sideslip angle need to be calculated in the non-rolling
Body Reference Frame.
Linear and angular equation of motion in a non-inertial frame can be written as:
F =m
dV
dt




BFP
+ ωBFP× V

M = Idω
dt




BFP
+ ωBFP× Iω
(5.6)
where V represents the body velocity vector, ω represents the body angular velocity
vector in 6DOF, ωBFP represents the angular velocity vector of the reference Body
Fixed Plane coordinate system, and I represents the inertia matrix.
The equations which explain the dynamics of the spinning projectile are given in
Equation 5.7 and Equation 5.8. In these equations; L, M, and N represent total moment on X, Y , and Z axis; X, Y , and Z represent the total force on X, Y , and Z axis.
The derivation of the equations is given in Appendix D.
```

### PDF page 98 | thesis page 72
<!-- raw_extraction: pdf_page=98; thesis_page=72 -->
```text


˙p
˙q
˙r

 = I-1




L
M
N

-


0 -r q
r 0 r tanθ
-q -r tanθ 0

 I


p
q
r



 (5.7)


˙u
˙v
˙w

 =


X
m
Y
m
Z
m

-


0 -r q
r 0 r tanθ
-q -r tanθ 0




u
v
w

 (5.8)
The equations which represent the kinematics of the system are given in Equation 5.9
and Equation 5.10. The derivation of these equations is given in Appendix D.


˙θ
˙ψ

 =

1 0
0 1
cosθ



q
r

 (5.9)


˙xE
˙yE
˙zE

 =


cosθ cosψ - sinψ sinθ cosψ
cosθ sinψ cosψ sinθ sinψ
- sinθ 0 cos θ




u
v
w

 (5.10)
5.2.2 6-DOF Simulink Model and Results
The overall structure of the Simulink model is given in Figure 5.3. This model is
formed by modifying Simulink Aerospace Block-set as it will work at Body Fixed
Plane Coordinate System.
```

### PDF page 99 | thesis page 73
<!-- raw_extraction: pdf_page=99; thesis_page=73 -->
```text
Figure 5.3: Overall Structure of 6-DOF Simulation
The physical properties of the simulated projectile and initial conditions are given in
Table 5.1.
Table 5.1: Physical Properties and Initial Conditions
Parameter Value Unit
Projectile diameter (d) 155 mm
Projectile mass (M) 43 kg
Ixx 0.144 kgm 2
Vi 800 m/s
pqri [1600, 0, -3.5] rad/sec
Firing elevation angleθi 63 °
Iyy =Iyy 1.216 kgm 2
The results of the simulation for the position and velocity of the projectile in the Earth
Fixed Reference Frame are given in Figure 5.4 and Figure 5.5. Since the positive zdirection is inside the Earth’s surface, position and velocity in Z-axis are given after
multiplying with -1 in the figures.
```

### PDF page 100 | thesis page 74
<!-- raw_extraction: pdf_page=100; thesis_page=74 -->
```text
Figure 5.4: Positions at Earth Fixed Reference Frame
5.3 Mathematical Model and Simulation of 7-DOF Projectile Dynamics
Since a projectile with the guidance kit has one additional DOF due to the free rotating
part, we need to take this into account. Linear and angular equation of motion for 7-
DOF is given in Equation 5.11.
F7DOS =m
dV
dt




BFP
+ ωBFP× V

M7DOF = Idω7DOF
dt




BFP
+ ωBFP× Iω7DOF
(5.11)
Equations to represent 7-DOF dynamics of the guided projectile are presented below
in Equation 5.12 and Equation 5.13. The reasons for the modifications done to the
equation of motion of the system are given in Appendix E.
```

### PDF page 101 | thesis page 75
<!-- raw_extraction: pdf_page=101; thesis_page=75 -->
```text
Figure 5.5: Velocities at Earth Fixed Reference Frame


˙pF
˙pA
˙q
˙r


= I-1




Lgk-7DOF
LB-7DOF
M7DOF
N7DOF


-


0 0 -r q
0 0 -r q
0 r 0 r tanθ
0 -q -r tanθ 0


I


pgk
pbody
q
r




(5.12)


˙u
˙v
˙w

 =


X7DOF
m
Y7DOF
m
Z7DOF
m

-


0 -r q
r 0 r tanθ
-q -r tanθ 0




u
v
w

 (5.13)
The inertia matrix in Equation 5.12, I, is given in Equation 5.14.


Ixgk 0 0 0
0 Ixbody 0 0
0 0 Iy 0
0 0 0 Iz


(5.14)
```

### PDF page 102 | thesis page 76
<!-- raw_extraction: pdf_page=102; thesis_page=76 -->
```text
Since the kinematics of the system is not change with the addition of 1-DOF, it is
possible to use the same equations with the 6-DOF model. The kinematic equations
are given in Equation 5.15 and Equation 5.16.


˙θ
˙ψ

 =

1 0
0 1
cosθ



q
r

 (5.15)


˙xE
˙yE
˙zE

 =


cosθ cosψ - sinψ sinθ cosψ
cosθ sinψ cosψ sinθ sinψ
- sinθ 0 cos θ




u
v
w

 (5.16)
5.3.1 Simulink Model and Results
The same initial conditions are used with the 6-DOF model. There is no control
input is applied to the guidance kit. It is a ballistic simulation of a projectile with the
guidance kit.
There is no significant change in positions and velocities at Earth Fixed Reference
Frame since the guidance kit rotates freely through the flight. Positions and velocities
at Earth Fixed Reference Frame are given in Figure 5.6 and Figure 5.7.
Figure 5.6: Positions at Earth Fixed Reference Frame
```

### PDF page 103 | thesis page 77
<!-- raw_extraction: pdf_page=103; thesis_page=77 -->
```text
Figure 5.7: Velocities at Earth Fixed Reference Frame
The control forces and moments that arise from the addition of the guidance kit are
shown in Figure 5.8 and Figure 5.9. The magnitude of the forces and moments are
higher at the beginning of the flight and end of the flight because of the high velocity
and air density.
Figure 5.8: Control Forces at Body Fixed Plane Reference Frame
```

### PDF page 104 | thesis page 78
<!-- raw_extraction: pdf_page=104; thesis_page=78 -->
```text
Figure 5.9: Control Moments at Body Fixed Plane Reference Frame
Aerodynamic torqueTaero and aerodynamic damping torque Tad on the guidance kit
are given in Figure 5.10 and Figure 5.11. In Figure 5.10, the results are given for
different firing elevation angles to show the change in the aerodynamic torque. These
torques are high at the beginning and end of the flight due to high velocity and air
density. At the apogee point, which is the point that has the highest altitude, aerodynamic torque reduces to nearly zero due to low air density and velocity. Therefore, if
the proposed guidance kit is used for guidance, the friction torque at this point should
be smaller than the aerodynamic torque to have control ability. If the friction torque
exceeds the aerodynamic torque at the apogee point, it is needed to begin the guidance
after that point.
In Figure 5.10 and 5.11, there are some sudden changes that are not possible physically. The reason is related to the aerodynamic coefficient set that is found in the
literature. Since the coefficients are available only at 0.9, 0.95, 1.1, 1.5, and 2.0
Mach, the data set is not comprehensive enough to cover all parts of the flight.
```

### PDF page 105 | thesis page 79
<!-- raw_extraction: pdf_page=105; thesis_page=79 -->
```text
Figure 5.10: Aerodynamic TorqueTaero on Guidance Kit
Figure 5.11: Aerodynamic Damping TorqueTad on Guidance Kit
```

### PDF page 106 | thesis page 80
<!-- raw_extraction: pdf_page=106; thesis_page=80 -->
*(Blank page.)*

### PDF page 107 | thesis page 81
<!-- raw_extraction: pdf_page=107; thesis_page=81 -->
```text
CHAPTER 6
MODIFICATION OF THE CONTROLLER FOR CHANGING
AERODYNAMIC TORQUE AND BODY ANGULAR VELOCITY
To have a symmetric response for positive and negative directions, aerodynamic
torque needs to be less than or equal to half of the total friction torque generated
by body angular velocity and alternator torque. Maximum alternator torque is determined by the current capacity of the programmable DC electronic load which is 8A.
Maximum aerodynamic torque is found in Equation 6.1. Friction torque is found by
testing for 800 rpm relative angular velocity between alternator rotor and stator.
Taeromax = Taltmax +Tfriction
2
Taltmax =Kt× 8 = 5.76
Tfriction = 0.1
Taeromax = 2.93Nm
(6.1)
Minimum aerodynamic torque needs to be bigger than the friction torque generated
by body angular velocity. The condition for minimum aerodynamic torque is given
in Equation 6.2.
Taeromin =Tfriction = 0.1Nm (6.2)
Since we have no control over aerodynamic torque, the system can make large overshoots if the aerodynamic torque is not high enough to slow down the system. Large
overshoots can lead to instability in the system because of the non-linearity formed
by full rotation. To overcome these problems, the designed controller needs to be
```

### PDF page 108 | thesis page 82
<!-- raw_extraction: pdf_page=108; thesis_page=82 -->
```text
modified for low aerodynamic torque inputs.
6.1 Flight Scenario to be Tested
Initial conditions for the selected flight scenario are given in Table 6.1.
Table 6.1: Initial Conditions
Initial Condition Value Unit
Vinitial 800 m/s
pqrinitial [1600, 0, -3.5] rad/s
θinitial 0.925 rad
Also, the aerodynamic torque and angular velocity of the projectile body are given
in Figure 6.1 and Figure 6.2. The flight scenario is selected such that aerodynamic
torque is always bigger than friction torque in order to test the control ability through
flight. The maximum angular velocity of the projectile body is nearly 15500 rpm
which is 1623.2 rad/s. It is possible to apply aerodynamic torque directly but it is
not possible to use body angular velocity directly because of the limits of the speed
motor and programmable DC electronic load. Therefore, it is needed to scale body
angular velocity. The scale factor is determined by the maximum voltage of the DC
electronic load which is 80 V . Scale factor, C, is given in Equation 6.3.
prelativemax = 80/Kv = 106.6rad/s
pbodymax =prelativemax-pgkmax
= 106.6- 31.42 = 75.18
C = 1623/75.18 = 21.6
(6.3)
Since the angular velocity of the projectile body is decreasing with respect to time
the scale factor is updated such that the maximum angular velocity of the body is 800
rpm, 83.7 rad/s.
```

### PDF page 109 | thesis page 83
<!-- raw_extraction: pdf_page=109; thesis_page=83 -->
```text
Figure 6.1: Aerodynamic Torque for Selected Flight Conditions
Figure 6.2: Body Angular Velocity for Selected Flight Conditions
```

### PDF page 110 | thesis page 84
<!-- raw_extraction: pdf_page=110; thesis_page=84 -->
```text
6.2 Results for Current Control Structure
Aerodynamic torque and scaled body angular velocity are applied to the simulation
with respect to time. The output of the velocity controller is saturated at 8 A to use
the full capacity of the system under large torque values. The output of the position
controller is saturated at 31.42 rad/s because of the voltage limit of the programmable
DC electronic load.
A square wave signal is given to the system through the flight period to examine the
response of the system to different torque and body angular velocity conditions. The
amplitude of the signal is pi/4 with pi offset. The response of the system for this
reference signal is given in Figure 6.3. From Figure 6.3 it is possible to observe that
control ability is lost between the 30s and 50s because at this region aerodynamic
torque is not enough to slow down the system and therefore the system makes very
large overshoots. Also, the comparison of the reference velocity and actual velocity is
given in Figure 6.4. It is found that the velocity controller is not capable of responding
to the output of the position controller in the direction of aerodynamic torque.
Figure 6.3: Input and Output Positions for Unmodified Controller
```

### PDF page 111 | thesis page 85
<!-- raw_extraction: pdf_page=111; thesis_page=85 -->
```text
Figure 6.4: Input and Output Velocities for Unmodified Controller
From the results of the unmodified controller it is possible to state that near to apogee
point, the response of the system needs to be adjusted in the direction of the alternator
torque. Therefore, the total disturbance torques in the system, friction, and aerodynamic torque, need to be observed.
6.3 Disturbance Observer Design
There are two main approaches to observe the disturbance in a dynamic system. The
first one is the frequency-domain approach which is designed in the Laplace domain.
The second is time-domain approach. The frequency-domain approach only uses the
input and output of the system. It is simple to apply but it gives less precise results
with respect to the time-domain approach which uses all the state variables to estimate
disturbance. Also, being able to use the time-domain approach requires measurement
or estimation of all the state variables [21]. In this case, since it is not possible to
measure voltage and current at the DC electronic load at the same time, the frequencydomain approach is used. The block diagram of the disturbance observer is given in
```

### PDF page 112 | thesis page 86
<!-- raw_extraction: pdf_page=112; thesis_page=86 -->
```text
Figure 6.5. Since this model uses the inverse of the plant, the input current and output
velocity need to be filtered to make the system physically applicable, which means the
product of the inverse of the system and the filter transfer function is proper, therefore
the system is causal.
Figure 6.5: Block Diagram of Disturbance Observer
To have a simple filter design and spend less computational time in real-life applications, the transfer function of the plant is simplified. The simplified transfer function
betweenIload andpgk is given in Equation 6.4. A first-order low pass filter is selected
such that the rise time of the filter 1.1 seconds and the transfer function of the filter is
given in Equation 6.5. A relatively large rise time is selected to filter the inputs due
to measurement noise. A relatively large time constant is not a problem for examined
system because the change of the aerodynamic torque does not have high-frequency
components. The details of the simplification of the transfer function, the comparison of the original transfer function with the reduced transfer function, and the filter
design are given in Appendix F.
ˆG = 588.4
s + 1.04 (6.4)
Q = 1
0.5s + 1 (6.5)
The comparison of the designed disturbance observer and real disturbance current at
```

### PDF page 113 | thesis page 87
<!-- raw_extraction: pdf_page=113; thesis_page=87 -->
```text
the system is given in Figure 6.6. Since aerodynamic torque and friction torque do
not have high-frequency components, the designed observer is enough for changing
the limits of the current and velocity during flight.
Figure 6.6: Real and Observed Disturbance Current
6.4 Possible Solution Approaches
Three possible solutions are examined to overcome this problem. The first solution
is limiting the output of the velocity controller with respect to changing aerodynamic
torque such that the control ability of the system is adjusted symmetrically in the direction of the alternator torque and in the direction of the aerodynamic torque. The
second solution is limiting the output of the position controller with respect to changing aerodynamic torque. The third solution is the combination of the first and second
approaches.
6.4.1 Changing Current Limit Dynamically with Aerodynamic Torque
This approach changes the upper saturation point of the velocity controller with respect to time by using the estimated disturbance current. The saturation point needs to
```

### PDF page 114 | thesis page 88
<!-- raw_extraction: pdf_page=114; thesis_page=88 -->
```text
be equal to two times of the disturbance observer result to have symmetric responses
in both directions.
The input and output positions are given in Figure 6.7. Also, the output of the velocity
controller with the result of the disturbance observer is given in Figure 6.8. It is
possible to observe from these figures that the system is controllable throughout the
flight. The control ability is always equal in positive and negative directions but since
we have no control at the velocity limit it is not possible to control the system without
having overshoots due to the low current limit and the high velocity limit which is
300 rpm. It is possible to observe this situation at the time between 35s and 55s.
Figure 6.7: Input and Output Positions for Dynamic Current Limit
```

### PDF page 115 | thesis page 89
<!-- raw_extraction: pdf_page=115; thesis_page=89 -->
```text
Figure 6.8: Velocity Controller Result for Dynamic Current Limit and Observed Current
6.4.2 Changing Velocity Limit Dynamically with Aerodynamic Torque
This approach changes the upper and lower saturation points of the position controller
with respect to time by using the result of the disturbance observer. The maximum
value of the velocity limit at the minimum aerodynamic torque is found by using the
simulation of the system and which is 5.2 rad/s for 0.2 A disturbance current. The
maximum and minimum limit of the position controller is scaled such that the output
is equal to 5.2 rad/s for 0.2 A disturbance current. The velocity scale factor is given
in Equation 6.6. To be inside of the capability of the load resistor, the output of the
position controller is saturated at 31.4 rad/s if the output of the velocity limit found
by using disturbance current is higher than this.
The input and output positions for this approach are given in Figure 6.9. It can be
observed from Figure 6.9 that it is possible to control the system through to flight
time without having any overshoot. Also, the output of the velocity controller is
given in Figure 6.10 and a closed view of this figure is given in Figure 6.11. As it can
be seen from Figure 6.11, the system has asymmetric responses to both sides since
```

### PDF page 116 | thesis page 90
<!-- raw_extraction: pdf_page=116; thesis_page=90 -->
```text
the changes of the reference angle at 40th s and 42th s are the same but the shape of
the current demands are different.
VSF = 5.2(rad/s)
0.2(A) = 26rad
As
VL = 26rad
As
ˆI(A)
Where VSF stands for "Velocity Scale Factor", and VL stands for "Velocity Limit".
(6.6)
Figure 6.9: Input and Output Positions for Dynamic Velocity Limit
```

### PDF page 117 | thesis page 91
<!-- raw_extraction: pdf_page=117; thesis_page=91 -->
```text
Figure 6.10: Velocity Controller Result for Dynamic Velocity Limit
Figure 6.11: Velocity Controller Result for Dynamic Velocity Limit - Zoomed View
```

### PDF page 118 | thesis page 92
<!-- raw_extraction: pdf_page=118; thesis_page=92 -->
```text
6.4.3 Changing Velocity and Current Limit Dynamically with Aerodynamic
Torque
This solution approach is formed to have symmetric control capacity in both directions and at the same time to have no overshot through to flight. It is the combination
of the first and the second solutions.
Input and output positions for this case are given in Figure 6.12. It can be seen from
this figure that we have control ability through to flight without having any overshoot.
The output of the velocity controller is given in Figure 6.13. A zoomed view of this
figure is given in Figure 6.14 which shows the same part with Figure 6.11. It is clear
that the asymmetric shape of the current demand is eliminated.
Figure 6.12: Input and Output Positions for Combined Solution
```

### PDF page 119 | thesis page 93
<!-- raw_extraction: pdf_page=119; thesis_page=93 -->
```text
Figure 6.13: Velocity Controller Result for Combined Solution
Figure 6.14: Velocity Controller Result for Combined Solution - Zoomed View
```

### PDF page 120 | thesis page 94
<!-- raw_extraction: pdf_page=120; thesis_page=94 -->
*(Blank page.)*

### PDF page 121 | thesis page 95
<!-- raw_extraction: pdf_page=121; thesis_page=95 -->
```text
CHAPTER 7
TEST RESULTS FOR MODIFIED CONTROLLERS
All the three solutions are tested in real-life and the results are presented in the related
sections. The transfer functions of the disturbance observer and filter are converted
to discrete-time to be applicable in real-life. At this operation Matlab "c2d" function
is used with "First-Order Hold" method and the time step is 1/1000s. Discretized
transfer functions of the observer and filter are given in Equation 7.1 and 7.2.
Gobserver(z) = 0.003397z- 0.003394
z- 0.998 (7.1)
Q(z) = 0.0009993z + 0.0009987
z- 0.998 (7.2)
These transfer functions are implemented to FPGA target and the results are presented
in the next three sections. Also, the test results for the unmodified controller are given
in this chapter.
7.1 The Results for Unmodified Controller
The input and output positions for test results and simulation results are presented in
Figure 7.1. Zoomed views of this figure are given in Figure 7.2 and Figure 7.3. In the
test case, the control ability is lost at a larger part of the flight in comparison to the
simulation. Possible reasons for this situation are the dynamics of the programmable
DC electronic load, continuous & discrete-time differences, and non-linear friction in
the mechanical test system. Also, in the FPGA code, it is needed to use fixed-point
numbers to implement the transfer functions. And also analog inputs and outputs
```

### PDF page 122 | thesis page 96
<!-- raw_extraction: pdf_page=122; thesis_page=96 -->
```text
require fixed-point numbers. The losses at the conversion between floating point and
fixed point can be a part of these differences. Also, in the test results, there are large
overshoots that do not exist in the simulation. The reason behind this is the ability of
the alternator to generate the required current with respect to relative velocity. The
maximum current can be taken from the alternator, and the output of the velocity
controller are given in Figure 7.4. This figure shows that the overshoot points match
with the time that the velocity controller demands the maximum current that can be
taken from the alternator. In real-life, this phenomenon causes an accumulation of
error at the controller of the programmable DC load and it requires time to begin
loading the alternator properly. Therefore, it causes overshoots in the system.
Figure 7.1: Input Position, Simulation, and Test Results for Unmodified Controller
```

### PDF page 123 | thesis page 97
<!-- raw_extraction: pdf_page=123; thesis_page=97 -->
```text
Figure 7.2: Input Position, Simulation, and Test Results for Unmodified Controller-
Zoomed View
Figure 7.3: Input Position, Simulation, and Test Results for Unmodified Controller-
Zoomed View
```

### PDF page 124 | thesis page 98
<!-- raw_extraction: pdf_page=124; thesis_page=98 -->
```text
Figure 7.4: Alternator Saturation Points and Velocity Controller Results
7.2 Test Results for Changing Current Limit
This case is applied at the test set-up and the results for input and output positions
are presented in Figure 7.5. Since we are not at the limit of the alternator, the test
results are consistent with the simulation results. Also in this part, the test results
and simulation results for the disturbance observer are compared and it is given in
Figure 7.6. Since the results of the disturbance observer are nearly the same for all
test conditions, it is not presented in the other sections. The result of the disturbance
observer is consistent with simulation results at low aerodynamic torques. There is a
difference between simulation and test results for high aerodynamic torque values.
```

### PDF page 125 | thesis page 99
<!-- raw_extraction: pdf_page=125; thesis_page=99 -->
```text
Figure 7.5: Input Position, Simulation, and Test Results for Changing Current Limit
Figure 7.6: Comparison of the Disturbance Observer Results
```

### PDF page 126 | thesis page 100
<!-- raw_extraction: pdf_page=126; thesis_page=100 -->
```text
7.3 Test Results for Changing Velocity Limit
The comparison of the test results and simulation results for changing the velocity
limit is given in Figure 7.7. In this case, the same problem mentioned in section
7.1 is also happened. There are overshoots in the test results due to the load limit
of the alternator with respect to the relative velocity between its rotor and stator.
When the guidance kit rotates in the same direction with the body, the results of the
velocity controller exceed the limit of the alternator and it causes large overshoots.
To eliminate this, the output of the controller can also be saturated to be inside of the
alternator load limit and it can be found by using present relative velocity which is an
available data.
Figure 7.7: Input Position, Simulation, and Test Results for Changing Velocity Limit
7.4 Test Results for Combined Solution
The combined solution is also applied in real-life and the results are presented in
Figure 7.8 for input and output positions. It is the best solution because we have no
overshoot and we have symmetric responses in both directions. Also, since we are
limiting the current with respect to aerodynamic torque, the limit of the alternator is
```

### PDF page 127 | thesis page 101
<!-- raw_extraction: pdf_page=127; thesis_page=101 -->
```text
not reached.
Figure 7.8: Input Position, Simulation, and Test Results for Combined Solution
The best solution is the combined solution due to symmetric response characteristics in both directions. Also in this solution, the transient response is consistent with
design requirements which means there is no overshoot. The first solution is not
applicable at the guidance of artillery projectiles in real-life because of the large overshoot which is an unwanted property for this application. The second solution can
be applied to the real artillery projectile since the angular velocity of the projectile
body always dominate the angular velocity of the guidance kit. The overshoot problem arises at the test setup since the velocity of the projectile body is scaled down.
This makes the ratio of guidance kit angular velocity to the projectile body angular
velocity larger.
```

### PDF page 128 | thesis page 102
<!-- raw_extraction: pdf_page=128; thesis_page=102 -->
*(Blank page.)*

### PDF page 129 | thesis page 103
<!-- raw_extraction: pdf_page=129; thesis_page=103 -->
```text
CHAPTER 8
CONCLUSION AND FUTURE WORK
8.1 Conclusion
In this thesis subject, the feasibility of a guidance method for spin-stabilized munitions is examined.
In Chapter 1, the problem definition and motivation behind this study are explained.
Also, the literature survey related to the solution of the guidance problem of spinstabilized munitions and the flight dynamics of these munitions is presented. In addition, the outline of the thesis and contribution to the literature are explained in the
first chapter.
In Chapter 2, the test set-up which is contracted to test the feasibility of the actuation
system is explained. Specifications of the components that are used in the set-up are
given in this chapter. The test software is also explained in Chapter 2.
In chapter 3, the mathematical model of the test set-up is established and different
modeling techniques are compared. Also, test result related to the mathematical
model is given in this chapter.
In chapter 4, controller design is explained. A cascade control structure is used to
control the position of the guidance kit. Test results of the designed controller are
given in this chapter.
In chapter 5, 6-DOF and 7-DOF dynamics of a spin-stabilized artillery projectile are
modeled. Established models are simulated with Matlab / Simulink and results are
given. The aerodynamic torque on the guidance kit and body angular velocity are
```

### PDF page 130 | thesis page 104
<!-- raw_extraction: pdf_page=130; thesis_page=104 -->
```text
found in this section.
In chapter 6, the aerodynamic torque and body angular velocity, which are found in
Chapter 5, are applied to the controller in the simulation environment. It is found that
the system has large overshoots when the aerodynamic torque is low. At the apogee
point, it is possible to lose the control ability of the system. 3 possible solutions are
applied in the simulation environment and the results are compared in this section.
The first solution is changing the current limit dynamically with respect to disturbance current. This solution solves the instability problem but it can not solve the
overshoot problem. The second solution is changing the velocity limit dynamically
with respect to the disturbance current. This solution solves instability and overshoot
problems but the system responds asymmetrically in positive and negative directions.
The third solution is the combination of the first and second solutions. This approach
solves overshoot and instability problems. In the third solution, the system also has
symmetric responses.
In chapter 7, the proposed solutions in Chapter 6 are applied to the test system. The
results are compared with simulation results. As it is expected, the best solution is
to change the saturation points of the velocity and current controller with respect to
disturbance current. For this case, there is no overshoot through to flight and the
system has symmetric behavior in both directions.
8.2 Future Work
By referencing the work done in this thesis, it is possible to state that the studied
actuation method can be applied to spin-stabilized munitions.
In the scope of this thesis, the flight dynamics of the projectile is modeled at 6-DOF
and 7-DOF. But Earth curvature, change in gravity with respect to altitude, and change
in aerodynamic coefficients with incidence and sideslip angles are neglected. And
guidance strategy is not worked in this thesis. The simulations can be extended to
include these assumptions, and the guidance algorithm can be implemented in the
future.
```

### PDF page 131 | thesis page 105
<!-- raw_extraction: pdf_page=131; thesis_page=105 -->
```text
Aerodynamic torque found by simulations is applied directly to the test system. To
be inside the limit of the test system, body velocity is scaled down. In the future, it is
possible to design a test system and an alternator to simulate flight conditions directly.
In Chapter 7, the aerodynamic torque and body velocity are applied to the test system
but the change in flight conditions due to guidance kit position is not considered. In
the future, it is possible to make hardware-in-the-loop (HIL) simulations to include
the impact of the guidance kit position on the flight parameters. And the guidance
algorithm can also be included in HIL simulations to test the guidance algorithm with
real actuator performance.
```

### PDF page 132 | thesis page 106
<!-- raw_extraction: pdf_page=132; thesis_page=106 -->
*(Blank page.)*

### PDF page 133 | thesis page 107
<!-- raw_extraction: pdf_page=133; thesis_page=107 -->
```text
REFERENCES
[1] Y . Wang, X.-m. Wang, and J.-y. Yu, “Influence of control strategy on stability
of dual-spin projectiles with fixed canards,” Defence technology, vol. 14, no. 6,
pp. 709–719, 2018.
[2] J. Spagni, S. Theodoulis, and P. Wernert, “Flight control for a class of 155 mm
spin-stabilized projectile with reciprocating canards,” in AIAA Guidance, Navigation, and Control Conference, p. 4685, 2012.
[3] A. Barton, “155-mm m795 aerofuze test at the kofa range, yuma proving
ground, arizona, 19 may 2015,” tech. rep., ARDEC, METC, RDAR-MEF-I Picatinny Arsenal United States, 2017.
[4] J. W. Robinson and F. Berefelt, On Guidance and Control for Guided Artillery
Projectiles, Part 1: General Considerations. Defence & Security, Systems and
Technology, Swedish Defence Research Agency . . . , 2011.
[5] J. A. Clancy, T. D. Bybee, and W. A. Friedrich, “Fixed canard 2-d guidance of
artillery projectiles,” Jan. 3 2006. US Patent 6,981,672.
[6] Y . Zhang, P. Li, L. Xiao, and F. Tian, “Research on control scheme of dualspin projectile with fixed canards,” in 2016 IEEE International Conference on
Mechatronics and Automation, pp. 144–148, IEEE, 2016.
[7] T. Yin, F. Jia, and J. Yu, “Research on roll control system for fixed canard rudder
of the dual-spin trajectory correction projectile,” Wireless Personal Communications, vol. 103, no. 1, pp. 83–98, 2018.
[8] M. Costello, “Modeling and simulation of a differential roll projectile,” in AIAA
Modeling and Simulation Technologies Conference and Exhibit, p. 4543, 1998.
[9] P. Wernert, S. Theodoulis, and Y . Morel, “Flight dynamics properties of 155
mm spin-stabilized projectiles analyzed in different body frames,” in AIAA atmospheric flight mechanics conference, p. 7640, 2010.
```

### PDF page 134 | thesis page 108
<!-- raw_extraction: pdf_page=134; thesis_page=108 -->
```text
[10] X. Zhang, Y . Xiaoxian, and Q. Zheng, “Impact point prediction guidance based
on iterative process for dual-spin projectile with fixed canards,”Chinese Journal
of Aeronautics, vol. 32, no. 8, pp. 1967–1981, 2019.
[11] K. Ogata, Modern Control Engineering . Instrumentation and controls series,
Prentice Hall, 2010.
[12] T. H. K. J. Astrom, Advanced PID Control. ISA Instrumentation, Systems and
Automation Society, 2006.
[13] T. H. Karl J. Astrom, PID Controllers. International Society for Measurement
and Con, 2 sub ed., 1995.
[14] A. Visioli, Practical PID Control . Advances in Industrial Control, Springer,
2006 ed., 2006.
[15] D. W. Smith, J. Crawford, and P. Moore, Marine auxiliary machinery. Elsevier,
2016.
[16] D. M. Soares, H. A. M. Calil, and R. M. Stephan, “Cascade control vs full-state
feedback,” in 2019 IEEE 15th Brazilian Power Electronics Conference and 5th
IEEE Southern Power Electronics Conference (COBEP/SPEC), pp. 1–6, 2019.
[17] U. Karapinar, Z. Esen, M. ¸ Sahin, and F. Kanburo˘glu, “Cascaded controller algorithm design for a brushless dc motor with matlab/simulink and mcu implementation,” in 2013 21st Signal Processing and Communications Applications
Conference (SIU), pp. 1–8, 2013.
[18] P. Wernert and S. Theodoulis, “Modelling and stability analysis for a class of
155 mm spin-stabilized projectiles with course correction fuse (ccf),” in AIAA
atmospheric flight mechanics conference, p. 6269, 2013.
[19] P. Wernert and S. Theodoulis, “Modelling and stability analysis for a class of
155 mm spin-stabilized projectiles with course correction fuse (ccf),” in AIAA
atmospheric flight mechanics conference, p. 6269, 2011.
[20] N. Sinha and N. Ananthkrishnan, Advanced Flight Dynamics with Elements of
Flight Control. CRC Press, 2017.
```

### PDF page 135 | thesis page 109
<!-- raw_extraction: pdf_page=135; thesis_page=109 -->
```text
[21] S. Li, J. Yang, W.-H. Chen, and X. Chen, Disturbance observer-based control:
methods and applications. CRC press, 2014.
[22] J. Guan and W. Yi, “Modeling of dual-spinning projectile with canard and trajectory filtering,” International Journal of Aerospace Engineering , vol. 2018,
2018.
[23] M. Khalil, H. Abdalla, et al., “Dispersion analysis for spinning artillery projectile,” in International Conference on Aerospace Sciences and Aviation Technology, vol. 13, pp. 1–12, The Military Technical College, 2009.
```

### PDF page 136 | thesis page 110
<!-- raw_extraction: pdf_page=136; thesis_page=110 -->
*(Blank page.)*

### PDF page 137 | thesis page 111
<!-- raw_extraction: pdf_page=137; thesis_page=111 -->
```text
Appendix A
To be consistent with flight dynamics notation, angular velocity on X-axis is presented
withp in all the equations.
Primary variables (Across-variables of tree branches and through variables of the
links) and secondary variables (Through variables of the branches and across variables of the links) are listed below:
• Primary variables: pbody(t),pgk,VT1,VRa,VCf ,Iload(t),ILa,Tb,TT 2,Taero(t),
Tbody
• Secondary variables:Tbody(t),TJgk,IT 1,IRa,ICF ,VLoad(t),VLa,pb,pT 2,pTaero,
pbody
So the system states become (Across variables of A-Type elements on the normal
tree, and through variables of T-type elements on the link)pgk,VCf andILa.
The elemental equations which relate across and through variables of the system are
listed below:
dpgk
dt = 1
Jgk
TJgk (A.1)
VT 1 =-KvpT 2 (A.2)
VRa =IRaRa (A.3)
```

### PDF page 138 | thesis page 112
<!-- raw_extraction: pdf_page=138; thesis_page=112 -->
```text
ICf =Cf
dVCf
dt (A.4)
dILa
dt = 1
La
VLa (A.5)
TT 2 =KtIT 1 (A.6)
Tb =bpb (A.7)
dpbody
dt = 1
Jbody
Tbody (A.8)
Continuity equations of the system are listed below:
IRa =ILa (A.9)
IRa-ICf-Iload(t) = 0 (A.10)
IT 1 +ILa = 0 (A.11)
-TJgk +Taero(t)-Tb-TT 2 = 0 (A.12)
Compatibility equations of the system are listed below:
VRa +VCf-VT 1 +VLa = 0 (A.13)
-pgk +pT 2 +pbody(t) = 0 (A.14)
```

### PDF page 139 | thesis page 113
<!-- raw_extraction: pdf_page=139; thesis_page=113 -->
```text
pbody(t)-pbody = 0 (A.15)
pb +pbody(t)-pgk = 0 (A.16)
By using the 16 equations listed above, the derivative of system states can be expressed in terms of input and states of the system. These equations are given below.
dpgk
dt = b
Jgk
pbody(t)- b
Jgk
pgk + KtILa
Jgk
+ Taero(t)
JGK
(A.17)
dVCf
dt = ILa
Cf
- Iload(t)
Cf
(A.18)
dILa
dt =-Kvpgk
La
+ Kvpbody(t)
La
- VCf
La
- RaILa
La
(A.19)
```

### PDF page 140 | thesis page 114
<!-- raw_extraction: pdf_page=140; thesis_page=114 -->
*(Blank page.)*

### PDF page 141 | thesis page 115
<!-- raw_extraction: pdf_page=141; thesis_page=115 -->
```text
Appendix B
To be consistent with flight dynamics notation, angular velocity on X-axis is presented
withp in all the equations.
The circuit at the electrical part is given in Figure B.1.
Figure B.1: Circuit of the Electrical Part
If we apply Kirchhoff’s current law at N1 and take the Laplace transform:
Iload =Ia-ICf (B.1)
If we apply Kirchhoff’s voltage law and take the Laplace transform:
-Va +RIa +LasIa + 1
CsICf = 0
-Kv(Pbody-Pgk) +RIa +LasIa + 1
CsICf = 0
(B.2)
If we extractICf from Equation B.1 and use it in Equation B.2:
```

### PDF page 142 | thesis page 116
<!-- raw_extraction: pdf_page=142; thesis_page=116 -->
```text
-Kv(Pbody-Pgk) +RaIa +LasIa + 1
Cfs(Ia-Iload) = 0 (B.3)
B.1 Derivation of the Transfer Function Between Iload andPgk
The free body diagram ofJgk forIload is given in Figure B.2.
Figure B.2: Free Body Diagram ofJgk forIload
Equation of the motion of the system is found as:
Talt-Tf =Jgk
dpgk
dt
Talt =KtIa
Tf =bpgk
(B.4)
If we take Laplace transform of Equation B.4 and extractIa in terms ofPgk:
Ia = Pgk(b +Jgks)
Kt
(B.5)
If we take the input other than Iload as zero and make some algebraic manipulations
in Equation B.3 we can find the following equation:
KvPgk +Ia(LaCfs2 +RaCfs + 1
Cfs ) = Iload
Cfs (B.6)
If we use Equation B.5 and write Ia in terms of Pgk in Equation B.6 we can find the
following transfer function:
```

### PDF page 143 | thesis page 117
<!-- raw_extraction: pdf_page=143; thesis_page=117 -->
```text
GIloadPgk = β0
σ3s3 +σ2s2 +σ1s +σ0
β0 = Kt
LaCfJgk
σ0 = b
LaCfJgk
σ1 = KvKtCf +bRaCf +Jgk
LaCfJgk
σ2 = bLaCf +RaCfJgk
LaCfJgk
σ3 = 1
(B.7)
We need to check the units of the parts added atσ1 andσ2.
KvKtCf = (Vs )(Nm
A )(F ) = (kgm 2
As3 s)(kgm
s2 m 1
A)(s4A2
m2kg )
=kgm 2
bRaCf = (kgm
s2 ms)(kgm 2
s3A2 )(s4A2
m2kg )
=kgm 2
Jgk =kgm 2
(B.8)
bLaCf = (kgm
s2 ms)(kgm 2
s2A2 )(s4A2
m2kg )
=kgm 2s
RaCfJgk = (kgm 2
s3A2 )(s4A2
m2kg )(kgm 2)
=kgm 2s
(B.9)
The units of the added terms are consistent with each other. At the other transfer
functions, the additive components of the denominator will not be checked since they
are the same for the same systems.
B.2 Derivation of the Transfer Function Between Taero(t) andPgk
The free body diagram ofJgk forTaero is given in Figure B.3.
```

### PDF page 144 | thesis page 118
<!-- raw_extraction: pdf_page=144; thesis_page=118 -->
```text
Figure B.3: Free Body Diagram ofJgk forTaero
Since there will be a voltage change at the alternator due to changing pgk we need
to include this torque in the free body diagram. Equation of motion of is given in
Equation B.10 in Laplace domain.
Taero +KtIa =Pgk(b +Jgks) (B.10)
If we take Pbody and Iload in Equation B.3 to zero and find Ia in therms of Pgk and
system variables, this gives Equation B.11.
Ia = -KvPgkCfs
LaCfs2 +RaCfs + 1 (B.11)
If we writeIa as it is found in Equation B.11 in Equation B.10 it gives Equation B.12.
Taero =Pgk(b +Jgks + KtKvCfs
LaCfs2 +RaCfs + 1) (B.12)
And the transfer function becomes:
```

### PDF page 145 | thesis page 119
<!-- raw_extraction: pdf_page=145; thesis_page=119 -->
```text
GTaeroPgk = β2s2 +β1s +β0
σ3s3 +σ2s2 +σ1s +σ0
β0 = 1
JgkLaCf
β1 = Ra
JgkLa
β2 = 1
Jgk
σ0 = b
JgkLaCf
σ1 = bRaCf +KtKvCf +Jgk
JgkLaCf
σ2 = JgkRa +Lab
JgkLa
σ3 = 1
(B.13)
There is no need to check units at this transfer function.
B.3 Derivation of the Transfer Function Between Pbody andPgk
The free body diagram ofJgk forpbody is given in Figure B.4.
Figure B.4: Free Body Diagram ofJgk forpbody
In Figure B.4,Talt represents the torque generated by the alternator due to the change
in voltage with respect to pbody andpgk,Tf represents friction due to velocity of pgk
andTfb represents the friction due to velocity of thepbody. The expressions related to
these torques are given in Equation B.14.
```

### PDF page 146 | thesis page 120
<!-- raw_extraction: pdf_page=146; thesis_page=120 -->
```text
Talt =KtIa
Tfb =bpbody
Tf =bpgk
(B.14)
The equation of motion is given in Equation B.14 in Laplace domain.
bPbody +KtIa =Pgk(b +Jgks) (B.15)
If we takeIload as zero and findIa in terms ofPgk andPbody in Equation B.3 it gives
Equation B.16.
Ia = CfsKv(Pbody-Pgk)
LaCfs2 +RaCfs + 1 (B.16)
If we put the result of Equation B.16 into Equation B.15 it gives Equation B.17.
Pbody(b+ KtKvCfs
LaCfs2 +RaCfs + 1) =Pgk(b+s(Jgk + KtKvCf
LaCfs2 +RaCfs + 1)) (B.17)
The transfer function becomes:
GPbodyPgk = β2s2 +β1s +β0
σ3s3 +σ2s2 +σ1s +σ0
β0 = b
LaCfJgk
β1 = bRa +KtKv
LaJgk
β2 = b
Jgk
σ0 = b
LaCfJgk
σ1 = Jgk +bRaCf +KtKvCf
LaCfJgk
σ2 = RaJgk +bLa
LaJgk
σ3 = 1
(B.18)
```

### PDF page 147 | thesis page 121
<!-- raw_extraction: pdf_page=147; thesis_page=121 -->
```text
We need to check the units of added terms inβ1.
bRa = (Nms)(Ohm)
= ((kgm
s2 )ms)(kgm 2
s3A2 )
= kg 2m4
s4A2
KtKv = (Nm
A )(Vs )
= (kgm
s2 m 1
A)(kgm 2
As3 s)
= kg 2m4
A2s4
(B.19)
The units are consistent with each other.
```

### PDF page 148 | thesis page 122
<!-- raw_extraction: pdf_page=148; thesis_page=122 -->
*(Blank page.)*

### PDF page 149 | thesis page 123
<!-- raw_extraction: pdf_page=149; thesis_page=123 -->
```text
Appendix C
Contents
• Velocity Controller Design
• Position Controller Design
clc
clear all
close all
% System parameters
Cf = 5 *(10^-5); % in Farad
La = 8.8 *10^-3; % in Hennry
Ra = 6.2; % in ohm
J_holder = 1.678 *10^-4; % in kg *m^2
J_alternator_body = 7.55 *10^-4; % in kg *m^2
J_coupling = 4.069 *10^-5; % in kg *m^2
J_magnets = 1.485 *10^-4; % in kg *m^2
J_motor = 0.85 *10^-4; % in kg *m^2
Jgk = J_holder + J_alternator_body +J_coupling...
+ J_magnets +J_motor ;
b = 0.2/(1500/60 *2*pi) % in Nm/(rad/sec)
Kt = 0.72 ; % in Nm/A
Kv = 0.75 ; % V/(rad/sec)
```

### PDF page 150 | thesis page 124
<!-- raw_extraction: pdf_page=150; thesis_page=124 -->
```text
% Form system and input matrices
A1_1 = -b/Jgk;
A1_2 = 0;
A1_3 = Kt/Jgk;
A2_1 = 0;
A2_2 = 0;
A2_3 = 1/Cf;
A3_1 = -Kv/La;
A3_2 = -1/La;
A3_3 = -Ra/La;
A = [A1_1, A1_2, A1_3;
A2_1, A2_2, A2_3;
A3_1, A3_2, A3_3];
B1_1 = b/Jgk;
B1_2 = 1/Jgk;
B1_3 = 0;
B2_1 = 0;
B2_2 = 0;
B2_3 = -1/Cf;
B3_1 = Kv/La;
B3_2 = 0;
B3_3 = 0;
B = [B1_1, B1_2, B1_3;
B2_1, B2_2, B2_3;
B3_1, B3_2, B3_3];
% Form system output matrices - OmegaGk
```

### PDF page 151 | thesis page 125
<!-- raw_extraction: pdf_page=151; thesis_page=125 -->
```text
C =[1, 0, 0];
D =[0, 0, 0];
stateSpaceModel = ss(A, B, C, D)
G_sys = tf(stateSpaceModel)
% Transfer function between Iload(t) and Omega_Jgk
beta0= Kt/(La *Cf*Jgk) ;
sigma0 = b/(La *Cf*Jgk) ;
sigma1_1 = Kv *Kt*Cf/(La*Cf*Jgk) ;
sigma1_2 = b *Ra*Cf/(La*Cf*Jgk) ;
sigma1_3 = Jgk/(La *Cf*Jgk) ;
sigma1 = sigma1_1+sigma1_2+sigma1_3 ;
sigma2_1 = b *La*Cf/(La*Cf*Jgk) ;
sigma2_2 = Ra *Cf*Jgk/(La*Cf*Jgk) ;
sigma2 = sigma2_1+sigma2_2 ;
sigma3 = 1;
num1 = [beta0] ;
denum1 = [sigma3, sigma2, sigma1, sigma0] ;
G_Iload_pgk = tf(num1, denum1)
% Transfer function between Taero(t) and Omega_Jgk
beta0 = 1/(Jgk *La*Cf);
beta1 = Ra/(Jgk *La);
beta2 = 1/Jgk;
sigma0 = b/(Jgk *La*Cf);
sigma1 = (b *Ra*Cf+Kt*Kv*Cf+Jgk)/(Jgk*La*Cf);
sigma2 = (Jgk *Ra+La*b)/(Jgk*La);
```

### PDF page 152 | thesis page 126
<!-- raw_extraction: pdf_page=152; thesis_page=126 -->
```text
sigma3 = 1;
num2= [beta2, beta1, beta0];
denum2 = [sigma3, sigma2, sigma1, sigma0];
G_Taero_pgk = tf(num2, denum2)
% Transfer function between Omega_body(t) and Omega_Jgk
beta0 = b/(Jgk *La*Cf);
beta1 = (b *Ra+Kt*Kv)/(Jgk*La);
beta2 = b/Jgk;
sigma0 = b/(Jgk *La*Cf);
sigma1 = (b *Ra*Cf+Kt*Kv*Cf+Jgk)/(Jgk*La*Cf);
sigma2 = (Jgk *Ra+La*b)/(Jgk*La);
sigma3 = 1;
num3= [beta2, beta1, beta0];
denum3 = [sigma3, sigma2, sigma1, sigma0];
G_OmegaBody_pgk = tf(num3, denum3)
Velocity Controller Design
s = tf(’s’);
pole_G_Iload_pgk = pole(G_Iload_pgk)
rlocus(G_Iload_pgk)
title(’G_Iload_pgk’)
xlim([-500 100])
ylim([-2000 2000])
% Velocity Controller Reuirements
syms zeta wn
ts_v = 0.05 % %5 settling time
Mp = 25 % Max overshoot, %
Eq1 = 25 == 100 *(exp(-(zeta/(sqrt(1-zeta^2)))*pi))
zeta_v = double(solve(Eq1))
zeta_v = max(zeta_v)
wn_v = 3/(ts_v *zeta_v)
```

### PDF page 153 | thesis page 127
<!-- raw_extraction: pdf_page=153; thesis_page=127 -->
```text
sgrid(zeta_v, wn_v)
figure
rlocus(G_Iload_pgk*((s+26.98)/s))
title(’Compansated G_Iload_pgk ’)
xlim([-500 100])
ylim([-2000 2000])
sgrid(zeta_v, wn_v)
figure
Gvc = 0.289860532841818 *(s+26.98)/s
Gol_v = G_Iload_pgk *Gvc
Gcl_v = feedback(Gol_v,1)
pole_clv = pole(Gcl_v)
step(Gcl_v)
title(’Closed Loop Velocity’)
figure
bode(Gcl_v)
title(’Bode-Closed Loop Velocity’)
figure
% Check disturbance rejection
Ev_Taero = -G_Taero_pgk/(1+Gvc *G_Iload_pgk)
step(Ev_Taero)
title(’Step Response for Step T_a_e_r_o Disturbance’)
figure
%
%
Ev_OmegaGk = -G_OmegaBody_pgk/(1+Gvc *G_Iload_pgk)
step(Ev_OmegaGk )
title(’Step Response for Step P_b_o_d_y Disturbance’)
figure
```

### PDF page 154 | thesis page 128
<!-- raw_extraction: pdf_page=154; thesis_page=128 -->
```text
Position Controller Design
Gol_p = Gcl_v *1/s
pole_olp = pole(Gol_p)
rlocus(Gol_p)
title(’Open Loop Position’)
%Position controller requirements
ts_p = 0.1 % %5 settling time
Mp = 5 % Max overshoot
Eq1 = Mp == 100 *(exp(-(zeta/(sqrt(1-zeta^2)))*pi))
zeta_p = double(solve(Eq1))
zeta_p = max(zeta_p)
wn_p = 3/(ts_p *zeta_p)
sgrid(zeta_p, wn_p)
xlim([-350 50])
ylim([-1550, 1550])
figure
Gcl_p = feedback(Gol_p *29.7,1)
step(Gcl_p)
title(’Closed Loop Position’)
figure
pole_clp=pole(Gcl_p)
bode(Gcl_p)
title(’Bode-Closed Loop Position’);
```

### PDF page 155 | thesis page 129
<!-- raw_extraction: pdf_page=155; thesis_page=129 -->
```text
Appendix D
Equation of motion in a non-inertial reference frame is given in Equation D.1.
F =m
dV
dt




BFP
+ ωBFP× V

M = Idω
dt




BFP
+ ωBFP× Iω
(D.1)
The symbols used in Equation D.1 are explained in the equations below.
F: Total force vector acting on projectile body written in Body Fixed Plane Reference
Frame.
F =


X
Y
Z

 (D.2)
M: Total moment vector acting on projectile body written at Body Fixed Plane Reference Frame.
M =


L
M
N

 (D.3)
V: Linear velocity vector of the projectile body written at Body Fixed Plane Reference
Frame.
V =


u
v
w

 (D.4)
```

### PDF page 156 | thesis page 130
<!-- raw_extraction: pdf_page=156; thesis_page=130 -->
```text
ωBFP : Angular velocity vector of the Body Fixed Plane Reference Frame.
ωBFP =


pBFP
q
r

 (D.5)
ω: Angular velocity vector of the projectile body written at Body Fixed Plane Reference Frame.
ω6DOF =


p
q
r

 (D.6)
I: Inertia matrix of the projectile body.
I =


Ix 0 0
0 Iy 0
0 0 IZ

 (D.7)
Cross product of ωBFP can be written in matrix form and if we replace pBFP in
this matrix form with-r tanθ, which is found in Equation 5.5, the skew-symmetric
matrix form of the cross product is given in Equation D.8.
ωBFP× =


0 -r q
r 0 r tanθ
-q -r tanθ 0

 (D.8)
If we take the derivative of the linear velocity vector in Equation D.1 to the left side
and replace the symbols with Equations D.2, D.4, D.5, and D.8; the equation of the
motion of the linear velocity can be found as given in Equation D.9.


˙u
˙v
˙w

 =


X
m
Y
m
Z
m

-


0 -r q
r 0 r tanθ
-q -r tanθ 0




u
v
w

 (D.9)
```

### PDF page 157 | thesis page 131
<!-- raw_extraction: pdf_page=157; thesis_page=131 -->
```text
If we take the derivative of the angular velocity vector in Equation D.1 to the left side
and replace the symbols with Equations D.3, D.5, D.6, D.7, and D.8; the equation of
the motion of the angular velocity can be found as given in Equation D.10.


˙p
˙q
˙r

 = I-1




L
M
N

-


0 -r q
r 0 r tanθ
-q -r tanθ 0

 I


p
q
r



 (D.10)
The transformation of the body axis angular angular velocity vector to the rate of
Euler angles is given in Equation D.11.


˙ϕ
˙θ
˙ψ

 =


1 sin ϕ tanθ cosϕ tanθ
0 cos ϕ - sinϕ
0 sinϕ
cosθ
cosϕ
sinθ




pF
qF
rF

 (D.11)
Since ˙ϕ andϕ are equated to zero by equatingpF to-r tanθ, Equation D.11 reduces
to Equation D.12. Note that, the rotation of the Body Fixed Plane Reference Frame
is the same with the angular velocity of the projectile except pF . Therefore, qF and
rF is replaced with q and r.


˙θ
˙ψ

 =

1 0
0 1
cosθ



q
r

 (D.12)
The transformation of linear velocities of the body to Earth Fixed Reference Frame is
given in Equation D.13.


˙xE
˙yE
˙zE

 = G


u
v
w

 (D.13)
where transformation matrix G is given in Equation D.14.
```

### PDF page 158 | thesis page 132
<!-- raw_extraction: pdf_page=158; thesis_page=132 -->
```text
G =


cosθ cosψ sinϕ sinθ cosψ- cosϕ sinψ cosϕ sinθ cosψ + sinϕ sinψ
cosθ sinψ sinϕ sinθ sinψ + cosϕ cosψ cosϕ sinθ sinψ- sinϕ cosψ
- sinθ sinϕ cosθ cosϕ cosθ


(D.14)
Since we equatedϕ to 0, Equation D.14 reduces to Equation D.15.
G =


cosθ cosψ - sinψ sinθ cosψ
cosθ sinψ cosψ sinθ sinψ
- sinθ 0 cos θ

 (D.15)
```

### PDF page 159 | thesis page 133
<!-- raw_extraction: pdf_page=159; thesis_page=133 -->
```text
Appendix E
With the addition of 1-DOF to the system, the inertia matrix, moment vector, angular
velocity vector, and the skew-symmetric matrix form of the cross product of ωBFP
need to be modified. [22]
Since the system has an independent degree of freedom at the X-axis, the inertia matrix should be updated to include two different inertia which can rotate independently.
The modified inertia matrix is given in Equation E.1.


Ixgk 0 0 0
0 Ixbody 0 0
0 0 Iy 0
0 0 0 Iz


(E.1)
Torques at the guidance kit need to be added to the moment vector. We need to add a
row to M7DOF to take total aerodynamic torque in the guidance kit,Lgk-7DOF which
is given in Equation G.18, into account. The modified moment vector is given in
Equation E.2.
M7DOF =


Lgk-7DOF
Lb-7DOF
M7DOF
N7DOF


(E.2)
The angular velocity vector of the projectile needs to be modified to include the angular velocity of the guidance kit,pgk. The modified angular velocity vector is given
```

### PDF page 160 | thesis page 134
<!-- raw_extraction: pdf_page=160; thesis_page=134 -->
```text
in Equation E.3.
ω7DOF =


pgk
pbody
q
r


(E.3)
The skew-symmetric matrix version of the cross product of ωBFP needs to be modified to be consistent with the dimensions of the other vectors. Also, the first row of
the matrix needs to be doubled since pbody andpgk should be multiplied by the same
row. Since they are independent angular velocities at the same axis, the row they need
to be multiplied by should be the same to take the derivative in the non-inertial frame
correctly. The modified version is given in Figure E.4.
ωBFP× =


0 0 -r q
0 0 -r q
0 r 0 r tanθ
0 -q -r tanθ 0


(E.4)
```

### PDF page 161 | thesis page 135
<!-- raw_extraction: pdf_page=161; thesis_page=135 -->
```text
Appendix F
The numerical value of the transfer function betweenIload andpgk is given in Equation
F.1. Also in this equation locations of the poles are presented.
GIloadPgk = 1.367e9
s3 + 705.6s2 + 2.325e6s + 2.418e6
p1 =-1.04
p2 =-352.28 + 1483.20i
p3 =-352.28- 1483.20i
(F.1)
Since the complex conjugate pole pair is 338.73 times far away from the imaginary
axis, it is possible to neglect the dynamics coming from these poles. By using partial
fraction expansion we can find residues related to poles. Residues and pole pairs
are found by using Matlab "residue" function and they are listed in Equation F.2.
Residues for complex conjugate poles are not in the traditional form since Matlab
gives complex residues for complex conjugate pole pairs.
GIloadPgk = 1.367e9
s3 + 705.6s2 + 2.325e6s + 2.418e6
r1 = 588.4
r2 =-294.2 + 69.7i
r3 =-294.2- 69.7i
(F.2)
The reduced transfer function is given in Equation F.3. The comparison of the step
responses for original and reduced transfer functions is given in Figure F.1. As it is
expected, the results are nearly the same.
```

### PDF page 162 | thesis page 136
<!-- raw_extraction: pdf_page=162; thesis_page=136 -->
```text
ˆG = 588.4
s + 1.04 (F.3)
Figure F.1: Comparison of Real and Reduced Transfer Functions
The main parameter in the first-order low pass filter design is the time constant. A
relatively large time constant, 0.5, is selected in order not to take the responses due
to measurement noise. Since aerodynamic torque and friction torque does not have
high-frequency component it is a proper choice for this control case. Step response
and Bode plot of the designed filter are given in Figure F.3 and Figure F.2. The
transfer function of the designed filter is given in Equation F.4.
Q = 1
0.5s + 1 (F.4)
```

### PDF page 163 | thesis page 137
<!-- raw_extraction: pdf_page=163; thesis_page=137 -->
```text
Figure F.2: Bode Plot of the Filter
Figure F.3: Step Response of the Filter
```

### PDF page 164 | thesis page 138
<!-- raw_extraction: pdf_page=164; thesis_page=138 -->
*(Blank page.)*

### PDF page 165 | thesis page 139
<!-- raw_extraction: pdf_page=165; thesis_page=139 -->
```text
Appendix G
G.1 Aerodynamic Forces And Moments
Aerodynamic forces and moments on a spin-stabilized projectile are explained in
this section. Numerical values of the coefficients are taken from [23]. Aerodynamic
forces and moments are calculated at the reference frame where incidence angle α
and sideslip angleβ are calculated. Definitions of the α andβ are given in Equation
G.1 where u, v, and w are linear velocity components of the center of mass.
α = tan-1(w
u )
β = tan-1( v√
u2 +w2 )
(G.1)
G.1.1 Aerodynamic Forces And Moments for 6-DOF Model
G.1.1.1 Lift Force
L = 1
2ρSV 2CLα×


sinβ2 + cosβ2 sinα2
- sinβ cosα cosβ
- cosβ2 sinα cosα

 (G.2)
```

### PDF page 166 | thesis page 140
<!-- raw_extraction: pdf_page=166; thesis_page=140 -->
```text
G.1.1.2 Drag Force
D =-1
2ρSV 2CD×


cosα cosβ
sinβ
sinα cosβ

 (G.3)
G.1.1.3 Magnus Force
K = 1
2ρSV 2Cypα
pd
V ×


0
- sinα cosβ
sinβ

 (G.4)
G.1.1.4 Pitching/Yawing Moment
MPY = 1
2ρsdV 2CMα×


0
sinα cosβ
- sinβ

 (G.5)
G.1.1.5 Magnus Moment
Mm = 1
2ρSdV 2(-Cnpα)pd
V ×


0
- sinβ
- sinα cosβ

 (G.6)
G.1.1.6 Pitch / Yaw Damping Moment
Mpyd = 1
2ρSdV 2Cmq
d
V ×


0
q
r

 (G.7)
```

### PDF page 167 | thesis page 141
<!-- raw_extraction: pdf_page=167; thesis_page=141 -->
```text
G.1.1.7 Roll Damping Moment
Mrd = 1
2ρSdV 2Clp
pd
V ×


1
0
0

 (G.8)
G.1.1.8 Gravitational Force
Fg =mg×


- sinθ
sinϕ cosθ
cosϕ cosθ

 (G.9)
Total aerodynamic forces on the projectile are given in Equation G.10.
F =


X
Y
Z

 = L + D + K + Fg (G.10)
Total aerodynamic moments on the projectile body are given in Equation G.11.
M =


L
M
N

 = MPY + Mm + Mpyd + Mrd (G.11)
G.1.2 Aerodynamic Forces And Moments for 7-DOF Model
Explanations of aerodynamic forces and moments which are formed by the guidance
kit are given below. Numerical values of these coefficients are taken from [1].
G.1.2.1 Control Forces
These are the forces that arise from the addition of the guidance kit to the projectile.
Fc = ρSV 2
2 CNδ

δS


0
cosθgk
sinθgk

 +


0
α
β



 (G.12)
```

### PDF page 168 | thesis page 142
<!-- raw_extraction: pdf_page=168; thesis_page=142 -->
```text
G.1.2.2 Control Moments
These are the moments that arise from the addition of the guidance kit to the projectile.
MC = ρSV 2lCG
2 CNδ

δS


0
- sinpF
cospF

 +


0
-β
α



 (G.13)
To find total aerodynamic force and moments on the guidance kit, it is required to
add control forces and control moments to Equation G.10 and Equation G.11. Total
aerodynamic force on the projectile with the addition of the guidance kit is given in
Equation G.14.
F7DOF =


X7DOF
Y7DOF
Z7DOF

 = F6DOF + Fc (G.14)
Total moment on the projectile with the addition of guidance kit is given in Equation
G.15. Since there will be another equation for added 7th DOF to the system related
to angular velocity about the X-axis, the component of the total moment on the body
of the projectile labeled asLB-7DOF .
M7DOF =


LB-7DOF
M7DOF
N7DOF

 = M6DOF + Mc (G.15)
G.1.2.3 Aerodynamic Torque on Guidance Kit
This is the torque due to the cant angle of the control canards. The direction of this
torque is in the opposite direction to the projectile angular velocity.
Taero = 1
2ρV 2SlgkCrc (G.16)
```

### PDF page 169 | thesis page 143
<!-- raw_extraction: pdf_page=169; thesis_page=143 -->
```text
G.1.2.4 Aerodynamic Damping Torque on Guidance Kit
This is the damping torque due to the angular velocity of the guidance kit. It is in the
opposite direction of the angular velocity of the guidance kit.
Tad = 1
2ρSd2VpgkCrdgk (G.17)
Total aerodynamic torque on the guidance kit is given in Equation G.18.
Lgk-7DOF =Taero +Tad (G.18)
```


## Appendix B - Compact chapter/page map

| Section | PDF pages | Printed thesis pages | Purpose |
|---|---|---|---|
| Front matter | 1-26 | Roman pages / unnumbered | Approval, abstracts, contents, lists, abbreviations |
| Chapter 1 - Introduction | 27-34 | 1-8 | Motivation, literature, contributions, thesis outline |
| Chapter 2 - Test system | 35-54 | 9-28 | Guidance-kit architecture, hardware, software, rig limits |
| Chapter 3 - Test-system model | 55-68 | 29-42 | State-space, Simulink, transfer functions, validation |
| Chapter 4 - Controller design | 69-92 | 43-66 | Cascade PI/P design, anti-windup, step/chirp tests |
| Chapter 5 - Flight dynamics | 93-106 | 67-80 | Coordinate frames, 6-DOF/7-DOF model, torque profiles |
| Chapter 6 - Adaptive controller | 107-120 | 81-94 | Observer and dynamic saturation strategies |
| Chapter 7 - Hardware results | 121-128 | 95-102 | Discretization and tests of all controller variants |
| Chapter 8 - Conclusion/future work | 129-132 | 103-106 | Conclusions, limitations, HIL and guidance next steps |
| References | 133-136 | 107-110 | Sources [1]-[23] |
| Appendix A | 137-140 | 111-114 | Linear-graph equations |
| Appendix B | 141-148 | 115-122 | Transfer-function derivations |
| Appendix C | 149-154 | 123-128 | MATLAB controller-design code |
| Appendix D | 155-158 | 129-132 | Non-inertial frame derivation |
| Appendix E | 159-160 | 133-134 | 7-DOF equation modifications |
| Appendix F | 161-164 | 135-138 | Plant reduction and filter design |
| Appendix G | 165-169 | 139-143 | Aerodynamic force/moment equations |


## Companion files

- `10463953.md`: raw page-ordered Markdown transcription.
- `10463953_ai_optimized.md`: this structured, normalized edition.
