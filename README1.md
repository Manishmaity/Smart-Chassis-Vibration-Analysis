# Smart Chassis Vibration Analysis System

**Project Module-4 · Chassis Structures & Safety Telltales**

A self-contained Jupyter/Colab notebook that simulates vibration on a car
chassis, extracts the same features a microcontroller would compute,
detects and classifies faults (crack vs. loose joint), drives a
debounced green/red telltale, and includes a ready-to-adapt bridge to
real accelerometer hardware. No physical sensors are required to run it
— all vibration signals are generated numerically.

## How the notebook maps to the project brief

| Project concept | Section |
|---|---|
| Accelerometers on key chassis points | §2 — simulated front/middle/rear sensor data |
| MCU finds abnormal frequency patterns | §3 feature extraction (FFT/PSD) + §4 ML classifier |
| Warning telltale on threshold breach | §5 threshold + debounce logic (MCU-style) |
| Model predicting structural issues | §6 live drive simulation + §9 summary |

## What's inside

| Section | What it does |
|---|---|
| 1. Setup | Imports, fixed RNG seed, sample rate (1 kHz), 1-second analysis windows |
| 2. Simulated sensor data | `simulate_window()` builds 3-channel acceleration from 3 chassis modes (18/45/90 Hz) plus engine harmonics and road noise. **Crack**: modes shift down and amplitude rises (stiffness loss, less damping), plus "breathing-crack" harmonics at 2×/3× the first mode. **Loose joint**: random high-frequency rattle impacts are injected |
| 3. Feature extraction | 9 features per sensor (RMS, peak, crest factor, kurtosis, dominant frequency, spectral centroid, low/mid/high band energy) → 27 features per window across 3 sensors |
| 4. ML classifier | A `RandomForestClassifier` distinguishes Healthy / Crack / Loose joint, reported with a classification report, confusion matrix, and top-10 feature-importance chart |
| 5. Threshold + debounce telltale | Learns a healthy baseline for RMS, kurtosis, and dominant frequency, computes a max-z-score anomaly score, and only switches the telltale ON after `N_ON` consecutive abnormal windows and OFF after `N_OFF` normal ones (hysteresis, to ignore a single pothole) |
| 6. Drive simulation | Simulates 90 seconds with a fault developing gradually from t = 30 s, plotting true severity, anomaly score vs. threshold (with telltale-on shading), and the classifier's live class probabilities. Includes a small red/green dashboard-circle view |
| 7. Sensitivity study | Detection-rate-vs-severity curves for both fault types, showing how small a fault the threshold rule can still catch |
| 8. Bridge to real hardware | Drop-in guidance for wiring an MPU6050/ADXL345 to an Arduino/ESP32, logging CSV, and reusing `channel_features()` / `anomaly_score()` on real data, plus Arduino-style pseudocode for the same debounce logic |
| 9. Summary | Restates the detection logic and states the honest limitations (see below) |

## Requirements

```
numpy
scipy
matplotlib
scikit-learn
```

No `ipywidgets` or other interactive-widget dependency — every cell runs
headlessly in plain Jupyter, Colab, or `jupyter nbconvert --execute`.

```bash
pip install numpy scipy matplotlib scikit-learn notebook
jupyter notebook
```

## Sample output

With the notebook's fixed random seed, one run produced:

```
Dataset: (900, 27) -> samples x features

              precision    recall  f1-score   support
     Healthy       0.94      0.97      0.95        75
       Crack       1.00      1.00      1.00        75
 Loose joint       0.97      0.93      0.95        75
    accuracy                           0.97       225

Healthy anomaly score: mean=0.75, 99th pct=2.71 (threshold=4.0)
Telltale first lit at t=36s -> severity at that moment ~0.25
```

Exact numbers can shift slightly across scikit-learn versions, but should
land close to this.

## Honest caveat for the report

The classifier is trained and tested on windows from the same synthetic
generator, so 97% accuracy shows the feature set and pipeline logic work
end-to-end — it isn't evidence of real-world performance on an actual
vehicle with real sensor noise and mounting variation the simulator
doesn't model. The notebook's own §9 says as much; keep that framing in
your write-up rather than quoting the accuracy number on its own.

## Suggested repo structure

```
├── Smart_Chassis_Vibration_Analysis.ipynb
└── README.md   # rename this file to README.md when you add it to the repo
```

## Limitations

- All data is simulated; the notebook's own §8 explains how to substitute
  logged MPU6050/ADXL345 data if you later add real hardware.
- Thresholds (`THRESH`, `N_ON`, `N_OFF`) are illustrative starting points —
  the notebook itself notes a real vehicle needs baselines re-learned per
  vehicle, and recalibration across speeds, loads, and road types.
- The sensitivity study (§7) characterizes detection of the simulator's
  own fault model, not a validated real-world detection limit.

## License

Add a license of your choice here (MIT is a common default for coursework
repos) — e.g. `MIT License, Copyright (c) 2026 <your name>`.

## Author

<your name> — <course / project name>
