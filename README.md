# Digital Line Coding and Power Spectral Density Analysis

This repository contains a Python implementation of a digital communications experiment that generates and analyzes six common line codes: **Unipolar NRZ**, **Polar NRZ**, **Polar RZ**, **Manchester**, **Differential Manchester**, and **AMI** (Alternate Mark Inversion). The code runs entirely in a single Google Colab cell using standard scientific libraries.

The experiment is designed to meet the following objectives:

1. Generate the six line codes from a random bit sequence.
2. Compare their DC content, bandwidth, and clocking properties.
3. Estimate the power spectral density (PSD) using Welch’s method.
4. Compute the average level and running digital sum (RDS).
5. Test the behaviour with a long run of identical bits.

All results are presented through aligned waveform plots, normalized PSD plots, RDS curves, and long‑run behaviour visualisations. A validation step encodes a fixed 8‑bit word and counts transitions to verify the encoders.

---

## Table of Contents

- [Line Coding Schemes](#line-coding-schemes)
- [Power Spectral Density (PSD)](#power-spectral-density-psd)
- [Running Digital Sum (RDS) and DC Content](#running-digital-sum-rds-and-dc-content)
- [Clock Recovery and Transitions](#clock-recovery-and-transitions)
- [Code Overview](#code-overview)
- [How to Run](#how-to-run)
- [Results and Discussion](#results-and-discussion)
  - [Observations](#observations)
  - [Discrepancies and Why They Occur](#discrepancies-and-why-they-occur)
- [Improvements and Extensions](#improvements-and-extensions)
- [References](#references)

---

## Line Coding Schemes

Line coding maps binary data (0s and 1s) to electrical signals. Each scheme has distinct properties:

| Scheme | Description | DC Content | Clocking | Bandwidth |
|--------|-------------|------------|----------|-----------|
| **Unipolar NRZ** | 1 → +V, 0 → 0 | Non‑zero (V/2 for balanced data) | Poor (long zeros) | Narrow |
| **Polar NRZ** | 1 → +V, 0 → –V | Zero (for balanced data) | Poor (long runs) | Narrow |
| **Polar RZ** | 1 → +V for half bit, then 0; 0 → –V for half, then 0 | Zero (if balanced) | Better (return to zero) | Wider than NRZ |
| **Manchester** | 1 → +V then –V; 0 → –V then +V | Zero | Excellent (transition every bit) | Double NRZ bandwidth |
| **Differential Manchester** | Transition at bit boundary always; 0 → mid‑bit transition, 1 → no mid‑bit transition (or vice‑versa) | Zero | Excellent | Same as Manchester |
| **AMI** | 1 → alternating ±V; 0 → 0 | Zero (for alternating ones) | Poor (long zeros) | Narrow, no DC |

---

## Power Spectral Density (PSD)

The PSD shows how the signal power is distributed over frequency. It is estimated using **Welch’s method**, which averages periodograms of overlapping segments to reduce variance. The plots are normalised to the maximum PSD and expressed in dB.

Key features:
- **DC component** appears as a peak at zero frequency.
- **Bandwidth** is indicated by the main lobe width and the presence of high‑frequency components.
- **Spectral nulls** occur at multiples of the bit rate (e.g., Manchester has a null at DC).

---

## Running Digital Sum (RDS) and DC Content

The **Running Digital Sum** is the cumulative sum of the signal levels (averaged over each bit period). It is a measure of the **DC wander**:

- A bounded RDS indicates no DC drift (good for transformer coupling).
- A linearly growing RDS indicates a DC component (problematic for AC‑coupled channels).

We compute the RDS from the per‑bit average values, which are obtained by averaging the waveform over each bit period.

---

## Clock Recovery and Transitions

Clock recovery at the receiver is easier when the signal contains frequent transitions. Schemes like Manchester and Differential Manchester guarantee at least one transition per bit, making them self‑clocking. Others (Unipolar NRZ, Polar NRZ) may have long runs without transitions, causing timing drift.

In the validation step, we count the number of level changes (both sample‑level and bit‑boundary transitions) for an 8‑bit test word to quantify this property.

---

## Code Overview

The code is written in Python and uses **NumPy**, **SciPy**, and **Matplotlib**. It is structured as follows:

1. **Parameters** – set amplitude `V`, samples per bit `spb`, bit period `Tb`, and the number of random bits.
2. **Encoder functions** – each returns the full waveform and per‑bit average for the input bit list.
3. **Waveform generation** – run all encoders on the same random bit sequence and store results.
4. **Visualisations**:
   - Aligned waveforms (all codes overlaid on separate subplots).
   - Normalised PSD (Welch estimate, plotted against normalised frequency `f/fb`).
   - RDS curves.
   - Long‑run behaviour (100 consecutive ones) – waveforms and RDS.
5. **Metrics** – print average level and final RDS.
6. **Validation** – encode a fixed 8‑bit word (`10110010`) and count level changes.
7. **Interpretation** – a printed summary comparing observations with theory.

All plotting is done with tight layouts and clear labels. The code runs in a single Colab cell, making it easy to execute and modify.

---

## How to Run

1. Open [Google Colab](https://colab.research.google.com/).
2. Create a new notebook.
3. Paste the entire code block from this repository into a single cell.
4. Run the cell. All plots and outputs will appear inline.

Alternatively, you can run the script locally with Python 3.6+ and the required libraries installed (`numpy`, `scipy`, `matplotlib`).

---

## Results and Discussion

### Observations

- **Unipolar NRZ**: DC level ≈ +V/2 for random bits; RDS grows steadily; PSD shows a strong DC peak and a main lobe up to ~1/Tb.
- **Polar NRZ**: Average level ≈ 0; RDS stays bounded for random data; PSD has a null at DC and lobes extending to ~1/Tb.
- **Polar RZ**: Average level ≈ 0; RDS oscillates around zero but can drift for long runs; PSD has a wider main lobe (double the bandwidth) due to the narrower pulse width.
- **Manchester**: Average exactly 0; RDS bounded (almost zero); PSD shows a deep null at DC and a main lobe peaking around fb, with a width of ~2fb (consistent with the sinusoidal shape).
- **Differential Manchester**: Similar to Manchester; no DC; RDS bounded; PSD similar to Manchester.
- **AMI**: Average ≈ 0 (for random data); RDS bounded and tends to cancel; PSD shows a null at DC and a spectral shape resembling a raised‑cosine.

These observations largely agree with established theory. For long runs of ones:
- Unipolar NRZ RDS grows linearly; Polar NRZ RDS stays near zero (alternating ±V); RZ grows linearly but with half slope; Manchester and diff‑Manchester remain zero; AMI alternates between +1 and –1, so RDS stays bounded.

### Discrepancies and Why They Occur

1. **PSD estimation variance**  
   Welch’s method is an estimator; with a finite number of bits (here 50), the estimated PSD is noisy and may not perfectly match the theoretical closed‑form spectra. Increasing the number of bits or using a longer FFT segment reduces variance.

2. **Normalisation**  
   We normalise each PSD to its own maximum. This makes comparisons of absolute power across codes impossible, but highlights the shape and relative peaks. If absolute comparison is needed, we should use the same scaling factor (e.g., based on total signal power).

3. **Edge effects**  
   The waveforms are not repeated periodically; the Welch method implicitly applies a window, which introduces spectral leakage. Using a longer bit sequence and a suitable window (e.g., Hann) reduces leakage.

4. **RDS calculation**  
   We use the per‑bit average voltage (not the actual instantaneous signal). This is a simplified measure; the true DC component is the time‑average of the waveform, which for RZ is lower than the peak level. Our RDS tracks the average levels, which still correctly indicates DC wander.

5. **Transition counting**  
   We count sample‑level changes using a threshold of 0.1V. For Manchester and differential Manchester, the number of transitions is high, as expected. However, due to the sampled representation, a single bit transition may produce multiple sample changes; we also count per‑bit boundary changes to give a more meaningful comparison.

6. **Long‑run waveforms**  
   For AMI with all ones, the waveform alternates ±V, so the average level is zero. Our RDS for AMI shows a sawtooth pattern alternating between +1 and 0 (or -1), which is correct because the per‑bit average is +V for a ‘1’ and then –V for the next, giving an average of 0 over a pair. Our RDS plot actually shows alternating steps, confirming the bounded nature.

**How to reduce discrepancies:**

- Increase the number of bits (e.g., 200 or 500) for better statistical stability.
- Use a longer segment length in `welch` (e.g., `nperseg=512`) and overlap more.
- Apply a window (e.g., Hann) to reduce leakage.
- Instead of normalising to the maximum, normalise to the total average power or to a reference (e.g., the PSD of a single ideal pulse) for absolute comparisons.
- For transition counting, use the bit‑level transitions only (by examining per‑bit average changes) to avoid sampling artefacts.

---

## Improvements and Extensions

- **Add more line codes** – e.g., B8ZS, HDB3, or MLT‑3.
- **Implement theoretical PSD formulas** and overlay them on the Welch estimates for direct validation.
- **Compute the exact DC component** by taking the time average of the waveform, not just per‑bit averages.
- **Include eye diagrams** for clock recovery visualisation.
- **Add a GUI or interactive controls** to change bit sequences or parameters on the fly.
- **Export results** to CSV or PDF for reporting.

---

## References

1. B. Sklar, *Digital Communications: Fundamentals and Applications*, 2nd ed. Prentice Hall, 2001.
2. J. G. Proakis, *Digital Communications*, 4th ed. McGraw‑Hill, 2001.
3. Welch, P. D. (1967). "The use of fast Fourier transform for the estimation of power spectra: A method based on time averaging over short, modified periodograms". *IEEE Trans. Audio Electroacoust.* AU‑15: 70–73.

---

## License

This project is open‑source and available under the MIT License.

---

*This README accompanies the experimental code for educational purposes. Feel free to use, modify, and distribute.*
