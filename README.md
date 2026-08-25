# Introduction to BrightSign NPU

BrightSign players equipped with Neural Processing Units (NPUs) enable advanced machine learning and AI capabilities directly on the device. These capabilities allow developers to deploy models for tasks such as object detection, face recognition, and more, leveraging the power of the NPU for real-time inference.

BrightSign Model Packages (BSMP) are delivered as BrightSign OS (BOS) "extensions." Extensions are firmware update files installed during a reboot. These extensions, implemented as Linux squashfs file systems, extend the firmware to include the BSMP. For more details, visit our [Extension Template Repository](https://github.com/BrightDevelopers/extension-template).

> **Looking for a complete solution?**
> [**Argus**](https://github.com/brightsign/argus-audience-measurement-extension) is BrightSign's
> reference audience-measurement application: person counting, gaze detection, dwell time,
> entry/exit events, and movement analytics, published over MQTT and Prometheus. This repository
> provides background on BrightSign NPU model packages and links to the individual extensions below.
>
> *Argus is the complete reference application; the extensions below are single-model building blocks.*

## Example BrightSign Model Packages

Start with **Argus**, the complete reference application. The others are single-model building
blocks, useful when you want to understand or reuse one piece in isolation.

| Package | Status | What it does |
|---|---|---|
| [**Argus Audience Measurement**](https://github.com/brightsign/argus-audience-measurement-extension) | BETA | Complete audience analytics: person count, gaze, dwell, entry/exit, direction, speed — over MQTT and Prometheus |
| [Gaze Detection](https://github.com/brightsign/brightsign-npu-gaze-extension) | ALPHA | RetinaFace face detection plus "is this person looking at the screen?" |
| [Object Detection](https://github.com/brightsign/brightsign-npu-object-extension) | ALPHA | Object detection with selectable classes and confidence thresholds |
| [Voice Detection](https://github.com/brightsign/brightsign-npu-voice-extension) | ALPHA | Gaze-triggered speech-to-text using a Whisper encoder-decoder model |

## What is an NPU?

A Neural Processing Unit (NPU) is a specialized hardware accelerator designed to efficiently execute machine learning and artificial intelligence tasks. Unlike general-purpose CPUs or GPUs, NPUs are optimized for operations commonly used in neural networks, such as matrix multiplications and convolutions. This makes them highly effective for real-time inference tasks, enabling faster processing with lower power consumption. For more technical details, refer to the [Rockchip RKNN Toolkit](https://github.com/airockchip/rknn-toolkit2).  Developers can also explore the [Rockchip RKNN Model Zoo](https://github.com/airockchip/rknn_model_zoo) for additional resources and tools.

NPUs are particularly well-suited for edge computing scenarios, where AI models need to run locally on devices without relying on cloud-based resources. This ensures low latency, enhanced privacy, and reduced bandwidth usage.

## Common Use Cases

BrightSign NPUs can be utilized across a wide range of applications. Here are some common use cases:

### Machine Vision
- Object detection and tracking
- Face recognition and gaze detection
- License plate recognition
- Pose estimation for human activity analysis
- Gesture recognition for user interaction

### Audio Processing
- Speech recognition and transcription
- Audio classification (e.g., detecting specific sounds or events)
- Text-to-speech synthesis

### Natural Language Processing
- Sentiment analysis
- Language translation
- Chatbot and conversational AI

### Other Applications
- Predictive maintenance using sensor data
- Anomaly detection in industrial systems
- Edge AI for IoT devices

## The Repository Family

| Repository | Role |
|---|---|
| [argus-audience-measurement-extension](https://github.com/brightsign/argus-audience-measurement-extension) | Complete audience-measurement application — **start here** |
| [brightsign-npu-general](https://github.com/brightsign/brightsign-npu-general) | This document: concepts, use cases, model licensing |
| [brightsign-npu-gaze-extension](https://github.com/brightsign/brightsign-npu-gaze-extension) | Single-model BSMP: gaze detection |
| [brightsign-npu-object-extension](https://github.com/brightsign/brightsign-npu-object-extension) | Single-model BSMP: object detection |
| [brightsign-npu-voice-extension](https://github.com/brightsign/brightsign-npu-voice-extension) | Two-model BSMP: gaze-triggered speech-to-text |
| [simple-gaze-detection-html](https://github.com/brightsign/simple-gaze-detection-html) | HTML5 demo app for the gaze BSMP |
| [simple-voice-detection-html](https://github.com/brightsign/simple-voice-detection-html) | HTML5 demo app for the voice BSMP |
| [simple-gaze-detection-presentation](https://github.com/brightsign/simple-gaze-detection-presentation) | BrightAuthor:connected demo for the gaze BSMP |
| [simple-object-detection-presentation](https://github.com/brightsign/simple-object-detection-presentation) | BrightAuthor:connected demo for the object BSMP |
| [bs-image-stream-server](https://github.com/brightsign/bs-image-stream-server) | Dev tool: browser view of annotated model output |
| [extension-template](https://github.com/BrightDevelopers/extension-template) | Starter template for any BrightSign extension |
| [bs-workshop-extension](https://github.com/BrightDevelopers/bs-workshop-extension) | Hands-on workshop: build, package, deploy, iterate |
| [bs-extension-workshop-html-app](https://github.com/BrightDevelopers/bs-extension-workshop-html-app) | Companion HTML app for the workshop |
| [howto-git](https://github.com/brightsign/howto-git) | GitHub for non-coders — for document collaborators |

### A note on GitHub organizations

Shipping model packages, demos, and tools live under
[`brightsign`](https://github.com/brightsign). Developer education material — the extension
template and the workshop — lives under
[`BrightDevelopers`](https://github.com/BrightDevelopers). The `BrightSign-Playground`
organization is retired; links to it survive only as redirects and should not be used.

## Model Licensing

When using models with BrightSign NPUs, it is essential to adhere to the licensing terms of the models included in your BSMP. Many models, such as those from the [Rockchip model zoo](https://github.com/airockchip/rknn_model_zoo), come with specific licensing conditions that must be followed.

For example:
- **Apache 2.0 License**: Permissive, allows commercial use with proper attribution.
- **GPL 3.0 License**: Strong copyleft, requires derivative works to be open-sourced.
- **AGPL 3.0 License**: Extends GPL 3.0 to include network-based usage, requiring source disclosure for SaaS applications.

For a detailed review of model licenses and their compatibility with commercial use, refer to the [Model Licenses Documentation](model-licenses.md).

**Disclaimer**: Always verify licensing terms and consult legal advice if necessary.
