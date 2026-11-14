# VisionText AI

VisionText AI is a Python-based OCR application for extracting printed text from image files using OpenCV, `pytesseract`, and the separately installed Tesseract OCR engine. The project includes both a command-line interface and a Streamlit dashboard for image upload, OCR processing, result export, and benchmark comparison against a preserved baseline.

## Overview

This project processes PNG, JPG, and JPEG images and extracts readable text from printed documents. It validates uploaded file size and decoded image dimensions before OCR, applies a consistent preprocessing pipeline, filters recognized words by Tesseract confidence, and saves the extracted text along with supporting output files.

The application is designed for straightforward use in local development environments. It supports single-image processing through `main.py` and interactive upload-based processing through `app.py`. The repository also includes tests and a benchmark runner that compares the active OCR pipeline with the original baseline implementation.

## Features

- PNG, JPG, and JPEG input support
- Input validation for file size and decoded image dimensions
- Normalization of grayscale, BGR, and transparent BGRA images
- Original preprocessing pipeline using grayscale conversion and adaptive thresholding
- Confidence-based filtering of recognized words
- Output generation for extracted text, JSON report, preprocessed image, and annotated image
- Optional CLI configuration for confidence threshold and page segmentation mode
- Streamlit dashboard for uploading images and reviewing OCR results
- Automated unit and integration tests
- Benchmarking against a preserved baseline using CER/WER evaluation against independent ground-truth text

## Technology Stack

| Technology | Purpose |
|---|---|
| Python 3.10+ | Application runtime |
| OpenCV | Image loading, normalization, preprocessing, and annotation |
| `pytesseract` | Python interface to Tesseract OCR |
| Tesseract OCR | OCR engine used for text recognition |
| NumPy | Image and numeric processing |
| Pillow | Image validation and decoding |
| Streamlit | Web-based upload and review dashboard |
| unittest | Automated validation |

## Project Structure

```text
VisionText_Ai/
├── app.py
├── main.py
├── ocr_engine.py
├── requirements.txt
├── README.md
├── LICENSE
├── .gitignore
├── outputs/
├── samples/
├── tests/
├── benchmarks/
│   ├── baseline/
│   ├── ground_truth/
│   ├── results/
│   └── run_benchmark.py
└── __pycache__/
```

### Important files

- `main.py` runs the OCR pipeline from the command line.
- `app.py` provides the Streamlit dashboard for interactive image processing.
- `ocr_engine.py` contains image validation, preprocessing, OCR execution, result filtering, and output generation.
- `requirements.txt` lists the Python dependencies.
- `tests/test_ocr_engine.py` contains unit tests and real-Tesseract integration checks.
- `benchmarks/run_benchmark.py` compares the current implementation with the preserved baseline.
- `samples/sample_document.png` is the included sample image used by the project and benchmark workflow.

## System Architecture

```text
Image file
  ↓
Python loader and validators
  ↓
OpenCV preprocessing
  ├── grayscale conversion
  └── adaptive thresholding
  ↓
Tesseract OCR engine
  ↓
Confidence filtering and word selection
  ↓
Annotated image + extracted text + JSON report
```

The project uses a simple pipeline: input images are read and validated, normalized into a supported format, processed by OpenCV, passed to Tesseract for OCR, and filtered by the configured confidence threshold. The selected variant is the one that produces the most accepted words, with mean confidence used as the tiebreaker.

## Requirements

- Python 3.10 or newer
- Tesseract OCR installed separately from Python packages
- English Tesseract data available for the installed engine
- Dependencies from `requirements.txt`

### Install Tesseract

Tesseract must be installed outside of Python. On Windows, the standard installation path is typically:

```powershell
C:\Program Files\Tesseract-OCR\tesseract.exe
```

After installation, confirm the engine is available:

```powershell
tesseract --version
```

If Tesseract is not on `PATH`, set it for the current terminal session:

```powershell
$env:TESSERACT_EXE = 'C:\Program Files\Tesseract-OCR\tesseract.exe'
```

## Installation

Clone the repository and create a virtual environment:

```powershell
py -3 -m venv .venv
.
.venv\Scripts\python.exe -m pip install --upgrade pip
.
.venv\Scripts\python.exe -m pip install -r requirements.txt
```

If Tesseract is not already on `PATH`, configure it in the same terminal before running OCR commands:

```powershell
$env:TESSERACT_EXE = 'C:\Program Files\Tesseract-OCR\tesseract.exe'
```

## Configuration

The application accepts runtime environment settings for upload size and decoded pixel limits. These values must be positive integers.

```powershell
$env:VISIONTEXT_MAX_UPLOAD_MB = '20'
$env:VISIONTEXT_MAX_IMAGE_PIXELS = '30000000'
```

The default limits are:

- 15 MB maximum upload size
- 25,000,000 maximum decoded pixels

If an uploaded file or decoded image exceeds these limits, the application raises a validation error before OCR is attempted.

## Running the Project

### Command-line usage

Run OCR on the included sample image:

```powershell
.
.venv\Scripts\python.exe main.py samples\sample_document.png
```

Use a custom confidence threshold and page segmentation mode:

```powershell
.
.venv\Scripts\python.exe main.py samples\sample_document.png --confidence 80 --psm 6
```

The CLI writes output files to the `outputs/` directory:

- `outputs/extracted_text.txt`
- `outputs/recognized_words.png`
- `outputs/preprocessed.png`
- `outputs/report.json`

### Streamlit dashboard

Start the dashboard from the project root:

```powershell
.
.venv\Scripts\python.exe -m streamlit run app.py
```

The dashboard allows the user to upload an image, set the minimum word confidence, choose a page segmentation mode, and review:

- original image
- preprocessed image
- annotated result image
- extracted text
- accepted word counts and confidence metrics

## Application Usage

1. Open the project in a Python environment with Tesseract installed.
2. Upload a PNG, JPG, or JPEG image through the CLI or the Streamlit dashboard.
3. The system validates the file type, file size, and image dimensions.
4. The image is normalized and preprocessed using the project’s grayscale or adaptive-threshold pipeline.
5. Tesseract recognizes text and returns word-level confidence values.
6. Words below the configured confidence threshold are excluded.
7. The accepted words are assembled into text output and saved for review.

## Benchmarking

The project includes a benchmark utility for comparing the current OCR pipeline with the preserved baseline implementation. The benchmark requires a reference text file for the target image stored under `benchmarks/ground_truth/` with the same stem as the image name.

Run the benchmark from the project root:

```powershell
.
.venv\Scripts\python.exe benchmarks\run_benchmark.py --repeats 3
```

This workflow:

- loads the original baseline module from `benchmarks/baseline/`
- loads the active OCR implementation from `ocr_engine.py`
- processes each variant using the same sample image
- calculates CER and WER using case-folded whitespace tokenization and Levenshtein edit distance
- writes benchmark results to `benchmarks/results/benchmark_results.json` and `benchmark_results.csv`

## Testing

The repository includes unit tests and Tesseract-backed integration tests.

Run the full suite:

```powershell
.
.venv\Scripts\python.exe -m unittest discover -s tests -v
```

The integration tests are skipped automatically if Tesseract is not installed.

## Known Limitations

- The project is built around a single clean sample document rather than a broad real-world OCR benchmark set.
- It does not include deskewing, rotation correction, resizing, denoising, or contrast enhancement by default.
- Tesseract confidence is an OCR engine estimate, not a verified measure of recognition accuracy.
- Tesseract must be installed separately; `pytesseract` is only the Python interface.
- File-size and pixel limits can be adjusted, but they are still validation constraints rather than unlimited processing.

## License

This project is released under the MIT License. See the `LICENSE` file for the full license text.

## Author

SHOJIB-80

## Project Status

The repository contains a complete local OCR workflow for image-based text extraction, including CLI execution, dashboard usage, validation logic, tests, and benchmarking. The implementation is focused on the actual OCR behavior present in the codebase and intentionally preserves the original preprocessing logic while adding input safety and reporting improvements.
