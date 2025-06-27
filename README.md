# Introduction to BrightSign NPU

BrightSign players equipped with Neural Processing Units (NPUs) enable advanced machine learning and AI capabilities directly on the device. These capabilities allow developers to deploy models for tasks such as object detection, face recognition, and more, leveraging the power of the NPU for real-time inference.

BrightSign Model Packages (BSMP) are delivered as BrightSign OS (BOS) "extensions." Extensions are firmware update files installed during a reboot. These extensions, implemented as Linux squashfs file systems, extend the firmware to include the BSMP. For more details, visit our [Extension Template Repository](https://github.com/brightsign/extension-template).

## Example BrightSign Model Packages

Currently there is one reference example available:

* [Gaze Detection](https://github.com/brightsign/brightsign-npu-gaze-extension)

## What is an NPU?

A Neural Processing Unit (NPU) is a specialized hardware accelerator designed to efficiently execute machine learning and artificial intelligence tasks. Unlike general-purpose CPUs or GPUs, NPUs are optimized for operations commonly used in neural networks, such as matrix multiplications and convolutions. This makes them highly effective for real-time inference tasks, enabling faster processing with lower power consumption. For more technical details, refer to the [Rockchip NPU Documentation](https://opensource.rock-chips.com/wiki_NPU).  Developers can also explore the [Rockchip Developer Guide](https://opensource.rock-chips.com/wiki_Main_Page) for additional resources and tools.

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

## Model Licensing

When using models with BrightSign NPUs, it is essential to adhere to the licensing terms of the models included in your BSMP. Many models, such as those from the [Rockchip model zoo](https://github.com/airockchip/rknn_model_zoo), come with specific licensing conditions that must be followed.

For example:
- **Apache 2.0 License**: Permissive, allows commercial use with proper attribution.
- **GPL 3.0 License**: Strong copyleft, requires derivative works to be open-sourced.
- **AGPL 3.0 License**: Extends GPL 3.0 to include network-based usage, requiring source disclosure for SaaS applications.

For a detailed review of model licenses and their compatibility with commercial use, refer to the [Model Licenses Documentation](model-licenses.md).

**Disclaimer**: Always verify licensing terms and consult legal advice if necessary.
