# VisionAI — Comprehensive Project Documentation & Academic Defense

## 1. Executive Summary

**Project Name:** VisionAI  
**Subtitle:** Multimodal Visual Question Answering & Computer Vision  
**Target Platform:** Native Android (API Level 26+ / Android 8.0 through Android 15+)  
**Runtime Architecture:** 100% Standalone Client-Side Application (Zero Python / Zero Server Dependency)  

VisionAI addresses the challenge of building a fully autonomous, mobile-first computer vision and multimodal reasoning system. Unlike traditional machine learning projects that require a tethered desktop machine running a Python backend (such as FastAPI, Flask, or Uvicorn), VisionAI performs real-time preprocessing, object detection, text recognition, and statistical image analysis entirely on the smartphone hardware, while interacting directly with Google's Gemini multimodal foundation model via direct HTTPS REST calls.

---

## 2. Academic Project Mapping & Technical Equivalences

In academic settings, projects often originate as desktop Python scripts using packages like OpenCV, YOLO, PyTorch, and EasyOCR. This section documents how each fundamental computer vision and machine learning concept was ported into production-grade, native Android components:

### 2.1 Image Preprocessing: OpenCV (`cv2`) ➔ Android Graphics & ExifInterface
- **Desktop/Python Origin:** `cv2.imread()`, `cv2.resize()`, `cv2.cvtColor()`
- **Android Native Implementation:** `LocalCVRepository.preprocessImage()`
- **Algorithm & Engineering:**
  - Reads image metadata via `androidx.exifinterface.media.ExifInterface` to extract the true sensor orientation (tags 1, 3, 6, 8 for 0°, 180°, 90°, 270° rotations).
  - Implements two-pass memory-safe decoding (`inJustDecodeBounds = true` followed by calculated power-of-two `inSampleSize`) preventing `OutOfMemoryError` even on 108-megapixel mobile sensors.
  - Applies a 2D affine transformation `android.graphics.Matrix` to rectify orientations before neural inference.

### 2.2 Object Detection: YOLO / Ultralytics ➔ Google ML Kit On-Device Object Detection
- **Desktop/Python Origin:** `from ultralytics import YOLO; model = YOLO('yolov8n.pt')`
- **Android Native Implementation:** `com.google.mlkit:object-detection`
- **Algorithm & Engineering:**
  - Utilizes Google's on-device MobileNetV2-SSD neural architecture packaged as pre-compiled native TFLite binaries.
  - Executes directly on the mobile device CPU/GPU/NPU without internet connectivity.
  - Extracts normalized bounding coordinates $[x_{min}, y_{min}, x_{max}, y_{max}] \in [0.0, 1.0]$, object category classifications, and probabilistic confidence scores.
  - Normalized boxes are drawn in Jetpack Compose using dynamic `androidx.compose.foundation.Canvas` overlay layers that scale cleanly across any display density (dp) or screen orientation.

### 2.3 Deep Learning Model Inference: PyTorch / TensorFlow ➔ ML Kit Native C++/TFLite Runtimes
- **Desktop/Python Origin:** `import torch`, `torch.nn`, `import tensorflow as tf`
- **Android Native Implementation:** Google Play Services ML Core & TensorFlow Lite native binaries bundled on Android.
- **Algorithm & Engineering:**
  - Zero Python wrapper overhead.
  - Memory-mapped model weights and hardware-accelerated quantization (INT8/FP16) ensuring sub-100ms inference times on modern mobile chipsets.

### 2.4 Text Recognition: EasyOCR ➔ Google ML Kit On-Device Text Recognition
- **Desktop/Python Origin:** `import easyocr; reader = easyocr.Reader(['en'])`
- **Android Native Implementation:** `com.google.mlkit:text-recognition`
- **Algorithm & Engineering:**
  - On-device neural OCR model capable of segmenting text into structural hierarchies: Blocks $\rightarrow$ Lines $\rightarrow$ Elements.
  - Fully offline execution with bounding box detection and instantaneous clipboard integration.

### 2.5 Optical & Statistical Image Analysis: scikit-image ➔ Native Kotlin Algorithms
- **Desktop/Python Origin:** `from skimage import exposure, filters, color`
- **Android Native Implementation:** `LocalCVRepository.analyzeImageMetrics()`
- **Algorithm & Engineering:**
  - **Luminance:** Calculated per ITU-R BT.601 standard:
    $$Y = 0.299 \cdot R + 0.587 \cdot G + 0.114 \cdot B$$
  - **RMS Dynamic Contrast:** Standard deviation of pixel intensities across the normalized image grid:
    $$\sigma_{RMS} = \sqrt{\frac{1}{N} \sum_{i=1}^{N} (Y_i - \bar{Y})^2}$$
  - **Sharpness / Focus Variance:** High-pass neighborhood difference gradient (Laplacian/Sobel proxy):
    $$\nabla Y = \sqrt{\left(\frac{\partial Y}{\partial x}\right)^2 + \left(\frac{\partial Y}{\partial y}\right)^2}$$
  - **Color Clustering:** Average channel distributions ($R, G, B$) and automatic dominant hue categorization (e.g., Deep Charcoal, Indigo, Warm Amber, Emerald Green).

### 2.6 Multimodal Visual Question Answering: Gemini Multimodal ➔ Direct REST Client
- **Desktop/Python Origin:** `google-generativeai` Python SDK
- **Android Native Implementation:** `GeminiRepository` with `gemini-3.5-flash` via OkHttp 4 + Moshi
- **Algorithm & Engineering:**
  - Direct secure HTTPS client connection to `https://generativelanguage.googleapis.com/v1beta/models/gemini-3.5-flash:generateContent`.
  - Base64 JPEG payload compression (constrained to $\le 1024$px dimension at 85% quality) optimizing network latency and memory bandwidth.
  - Strict system instructions enforcing factual, grounded visual reasoning without hallucinations.
  - Conversational context preservation across multi-turn queries.

---

## 3. System Architecture & Component Interactions

```
+---------------------------------------------------------------------------------+
|                                VisionAI Android App                             |
|                                                                                 |
|  +--------------------+     +------------------------------------------------+  |
|  |   UI Layer         |     | State Layer                                    |  |
|  | - VisionAIScreen   | <-> | - VisionViewModel                              |  |
|  | - ImagePreview     |     | - VisionUiState (StateFlow)                    |  |
|  | - Canvas Overlays  |     +------------------------------------------------+  |
|  | - QuestionInput    |                            |                            |
|  | - AnswerCard       |                            v                            |
|  +--------------------+             +-----------------------------+             |
|                                     | Repository Layer            |             |
|                                     +-----------------------------+             |
|                                     |                             |             |
|                                     v                             v             |
|                        +-----------------------+     +-----------------------+  |
|                        | LocalCVRepository     |     | GeminiRepository      |  |
|                        | (100% On-Device)      |     | (Direct HTTPS Client) |  |
|                        +-----------------------+     +-----------------------+  |
|                        | - EXIF & Subsampling  |     | - gemini-3.5-flash    |  |
|                        | - ML Kit Object Detect|     | - OkHttp + Moshi      |  |
|                        | - ML Kit Neural OCR   |     | - Multi-turn VQA      |  |
|                        | - Optical Algorithms  |     | - Factual Prompts     |  |
|                        +-----------------------+     +-----------------------+  |
+------------------------------------------------------------------|--------------+
                                                                   | HTTPS REST
                                                                   v
                                                  +-------------------------------+
                                                  | Google Gemini Cloud Endpoints |
                                                  +-------------------------------+
```

---

## 4. Google Play Policy Compliance & Security

1. **Zero Broad Storage Permissions:** Does not request `READ_EXTERNAL_STORAGE` or `MANAGE_EXTERNAL_STORAGE`. Instead, it uses Android's zero-permission **Photo Picker** (`ActivityResultContracts.PickVisualMedia`), complying strictly with modern Android security standards.
2. **Safe Camera Usage:** Declares `android.permission.CAMERA` with `android.hardware.camera.any` marked as optional (`required="false"`), and verifies runtime permission prior to launch.
3. **API Key Isolation:** API keys are never hardcoded. Keys are loaded from `BuildConfig.GEMINI_API_KEY` or user-configured through an encrypted in-memory dialog.

---

## 5. Verification & Testing

- **Compilation:** Clean Kotlin Gradle DSL build via `compile_applet` and Gradle `assembleDebug`.
- **UI Responsiveness:** Tested across compact, standard, and wide portrait orientations with Material 3 dynamic padding and edge-to-edge system insets.
- **Fail-Safe Operation:** In the absence of an internet connection or Gemini API key, all on-device Computer Vision features (Object Detection, OCR, and Image Analysis) remain 100% functional.
