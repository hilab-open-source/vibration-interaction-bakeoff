# Piezo Vibration Classification Starter

## Overview

This course starter lets you collect examples of vibrations, train a classifier, and run live inference on a laptop. A piezo sensor connected to the laptop's audio input acts as a contact microphone. A Dear PyGui desktop application calculates and displays FFT features, records labeled examples, trains a scikit-learn model, and shows predictions.

Unlike the other two audio starters, this project does not require an ESP32-S3 to collect data or run the provided application. A trained Linear SVM can be exported as a C++ header for later use in an ESP32 project.

## What you will build

You will create a custom vibration recognizer—for example, a classifier for tapping, scratching, knocking, or interactions with different surfaces. You will:

1. Capture vibrations through the laptop audio input.
2. Visualize their frequency spectra.
3. Record and label examples in the desktop application.
4. Train and evaluate a classifier on the laptop.
5. Optionally export a Linear SVM for ESP32.

![Vibration classification interface showing live FFT features](assets/fft_gui.png)

## How this starter differs from the other audio starters

| Starter | Audio input | Model | Where inference runs |
| --- | --- | --- | --- |
| Sound classification | INMP441 on ESP32-S3 | Student-trained Linear SVM | Laptop or ESP32-S3 |
| Speech recognition | INMP441 on ESP32-S3 | Pretrained ESP-SR MultiNet6 | ESP32-S3 |
| **Vibration interaction (this project)** | Piezo sensor on laptop audio input | Student-trained scikit-learn classifier | Laptop; Linear SVM can be exported |

## Hardware

- Piezo sensor or piezo contact microphone
- Audio cable or adapter connecting the piezo signal to the laptop audio input
- A surface or object whose vibrations you want to classify

Select the piezo/audio input device in the application. If the laptop has no compatible analog input, use a USB audio interface with a microphone or line input. Begin with low input gain: piezo elements can produce large voltage spikes when struck. Use appropriate input protection or conditioning for your audio hardware.

## Software setup

Install Python 3.10 or later and [`uv`](https://docs.astral.sh/uv/getting-started/installation/). On macOS, install PortAudio first:

```bash
brew install portaudio
```

Then install the locked environment:

```bash
uv sync
```

Alternatively, create a Python 3.10 virtual environment and install the project:

```bash
python3.10 -m venv .venv
. .venv/bin/activate
pip install .
```

## Run the starter

With `uv`:

```bash
uv run visualizer.py
```

Or with the virtual environment active:

```bash
python visualizer.py
```

Choose the input device connected to the piezo sensor. The application remembers the selected device between sessions.

## Use the application

The application plots the live FFT spectrum with frequency in hertz on the x-axis and amplitude on the y-axis. A peak readout shows the strongest frequency and its current amplitude.

To train and use a model:

1. Choose a model. `LinearSVC` is the default and can be exported for ESP32.
2. Add at least two classes, such as `tap` and `scratch`.
3. Select a class and click **Record Data**.
4. Repeat the interaction under varied conditions, then stop recording.
5. Collect examples for every class and click **Train Model**.
6. Switch to **Infer** to view live predictions.
7. Optionally save the trained `.model` file.

Collect all classes with the same sensor mounting, input gain, and object setup. Physical changes can shift the vibration spectrum and reduce accuracy.

## How it works

1. The laptop audio input captures mono samples from the piezo sensor.
2. `StreamAnalyzer` divides the stream into windows and calculates FFT amplitudes.
3. `get_audio_features()` returns 512 frequency bins for visualization and data collection.
4. `preprocess_data()` prepares each feature vector; the starter currently returns it unchanged.
5. The selected scikit-learn classifier learns from the labeled vectors.
6. The same feature pipeline feeds the model during live laptop inference.

The graph uses a fixed maximum amplitude of `500000` to make signal levels easier to compare. Input devices may have different sample rates, gains, and analog conditioning, so data may not transfer directly between setups.

## Customize the starter

Most project-specific changes belong in `models.py`, which contains interfaces for features, preprocessing, training, prediction, persistence, and export.

- Modify `get_ear()` or `get_audio_features()` to experiment with FFT settings and features.
- Modify `preprocess_data()` to normalize or transform feature vectors.
- Modify `train_model()` to change the classifier or its parameters.
- Keep training and inference preprocessing identical.
- Retrain after changing FFT size, sample rate, bin count, preprocessing, sensor mounting, or input gain.

### Export a Linear SVM for ESP32

Train with `LinearSVC`, save the `.model` file, and convert it to a C++ header:

```bash
uv run export_model_header.py path/to/model.model vibration_model.h
```

Include the header in an ESP32 project and supply a feature vector with exactly the same length and preprocessing used for Python training:

```cpp
#include "vibration_model.h"

float features[VIBRATION_MODEL_NUM_FEATURES];

int class_index = vibration_model_predict_index(features);
const char *class_name = vibration_model_predict_name(features);
```

The export creates model code only. Your ESP32 firmware must separately sample the sensor and reproduce the laptop's FFT feature pipeline.

## Test and verify

This starter does not currently include an automated test suite. For a functional check, launch the application, confirm the selected input produces a changing FFT plot, record two classes, train a model, and verify that **Infer** displays predictions.

## Troubleshooting

### No input device appears

- Connect the piezo or USB audio interface before starting the application.
- Grant microphone permission to the terminal or application running Python.
- On macOS, confirm PortAudio is installed.
- Restart after connecting a new audio device.

### The FFT plot does not react

- Select the piezo input rather than the built-in microphone.
- Check the audio adapter, input gain, and piezo wiring.
- Try a gentle tap and watch the peak-amplitude readout.

### Predictions are inconsistent

- Collect more varied examples for every class.
- Keep sensor mounting and input gain fixed.
- Include a background class if the model must distinguish idle periods.
- Improve the features in `models.py` if classes have similar spectra.

## Project layout

| Path | Purpose |
| --- | --- |
| `visualizer.py` | Desktop training and inference application |
| `models.py` | Features, models, persistence, and export |
| `src/stream_analyzer.py` | Audio buffering and FFT analysis |
| `src/stream_reader.py` | Audio-device discovery and capture |
| `src/utils.py` | Signal-processing utilities |
| `export_model_header.py` | Converts a Linear SVM to C++ |
| `assets/` | Interface screenshots |
| `metadata/` | Saved application preferences |
| `pyproject.toml` / `uv.lock` | Python environment |
| `LICENSE` | MIT license |
