# VisionAI — Multimodal Visual Question Answering & Computer Vision

VisionAI is a **standalone native Android application** combining on-device Computer Vision with Google Gemini multimodal Visual Question Answering (VQA).

---

## ⚡ Standalone Architecture — Zero Server Dependency

> **CRITICAL ARCHITECTURAL GUARANTEE:**
> VisionAI **does NOT require any Python backend, FastAPI, Flask, uvicorn, or PC server**.
> It installs directly as a standalone `.apk` on an Android phone.
> All local computer vision processing executes directly on the mobile device hardware, and VQA queries communicate directly with Google's Gemini multimodal REST endpoint.

---

## 🎓 Academic Project Mapping

For academic evaluations and project submissions, this project maps desktop/Python data science concepts to production-grade native Android runtimes:

| Original Desktop / Python Concept | Purpose / Role | Standalone Android Implementation Used |
| :--- | :--- | :--- |
| **OpenCV** (`cv2`) | Image Preprocessing & Transforms | **Android Native Graphics + Matrix EXIF Normalization** (`LocalCVRepository`): orientation matrix rotation, memory-safe subsampling, downscaling, color space mapping |
| **YOLO / Ultralytics** | Real-time Object Detection | **Google ML Kit On-Device Object Detection** (`com.google.mlkit:object-detection`): on-device MobileNet/SSD neural detector with label classification & bounding boxes |
| **PyTorch / TensorFlow** | Deep Learning Inference Engine | **ML Kit On-Device Neural Model Runtime**: native C++/TFLite binaries executing inference directly on phone CPU/NPU |
| **EasyOCR** | Optical Character Recognition | **Google ML Kit On-Device Text Recognition** (`com.google.mlkit:text-recognition`): on-device neural text recognizer extracting text blocks, lines, and bounding boxes |
| **scikit-image** | Image Analysis & Optics | **Native Kotlin Optical Algorithms**: Rec. 601 pixel luminance, RMS contrast standard deviation, Laplacian/Sobel edge gradient sharpness proxy, RGB color clustering |
| **Gemini Multimodal** | Visual Question Answering (VQA) | **Gemini 3.5 Flash REST API** (Direct Client via OkHttp + Moshi): multi-turn conversational VQA with direct base64 image transmission |
| **Streamlit / Web UI** | Frontend Interface | **Jetpack Compose + Material 3**: modern, reactive, accessible Android UI with custom Canvas bounding box rendering |

---

## 🚀 Key Features

1. **Dual Image Input**:
   - Android Photo Picker (zero-permission, privacy-first)
   - Live Camera photo capture with runtime permission verification
2. **On-Device Computer Vision**:
   - **Object Detection**: Detects multiple objects with confidence scores, labels, and real-time bounding box canvas overlay
   - **Text Recognition (OCR)**: Extracts visible text blocks and lines offline with 1-click clipboard copy
   - **Image Analysis**: Calculates resolution, aspect ratio, brightness classification, dynamic contrast range, sharpness/focus classification, and dominant color hue
3. **Multimodal Visual Question Answering (VQA)**:
   - Powered by `gemini-3.5-flash`
   - Ask natural language questions about any uploaded image
   - Quick questions suggestion chips for rapid queries
   - Full conversation history with timestamps, user/AI chat bubbles, and answer copy action
   - Multi-turn contextual question answering
4. **Resilient & Crash-Proof**:
   - Memory-safe bitmap decoding preventing `OutOfMemoryError` on 50MP+ photos
   - Non-blocking asynchronous coroutine execution
   - API key management dialog with runtime key override support
   - Interactive Architecture & Tech Info modal directly in the app

---

## 🛠️ Tech Stack & Dependencies

- **Language**: Kotlin 100%
- **UI Toolkit**: Jetpack Compose, Material 3
- **State Architecture**: MVVM (`ViewModel`, `StateFlow`, Coroutines)
- **On-Device ML**: Google ML Kit (`object-detection`, `text-recognition`)
- **Image Handling**: Android Graphics, `androidx.exifinterface:exifinterface`, `coil-compose`
- **Network / API**: OkHttp 4, Moshi (Kotlin Reflection)
- **AI Model**: Google Gemini 3.5 Flash (REST API v1beta)

---

## 📱 Running & Building the APK

1. In Google AI Studio:
   - Click **Run** or use the **Streaming Android Emulator** to interact with VisionAI.
   - Configure your Gemini API key in the AI Studio Secrets panel (`GEMINI_API_KEY`) or tap the key icon in the app header.
2. Generating the APK:
   - Export project via the Settings menu or run `gradle assembleDebug` to produce `app-debug.apk`.
   - Install the APK on any Android 8.0+ (API 26+) physical device.
