# Architecture

How on-device AI runs on a BrightSign player, from silicon to presentation.

## Layers

```mermaid
flowchart TB
    subgraph L1["1. Hardware"]
        direction LR
        RK3588["RK3588 - 3-core, 6 TOPS<br/>XT5"]
        RK3576["RK3576 - 2-core, 4 TOPS<br/>XS6 / XD6"]
        RK3568["RK3568 - 1-core, 1 TOPS<br/>LS5 / HS5"]
    end
    subgraph L2["2. BrightSignOS + RKNN runtime"]
        RKNN["librknnrt.so<br/>loads .rknn models"]
    end
    subgraph L3["3. BSMP extension (squashfs)"]
        direction LR
        CPP["C++ / Go inference engine"]
        MODELS[".rknn models<br/>YOLOX, RetinaFace,<br/>ByteTrack, Whisper, RKLLM"]
    end
    subgraph L4["4. Transport"]
        direction LR
        UDP["UDP localhost<br/>5000 BrightScript / 5002 JSON"]
        MQTT["MQTT<br/>analytics/v7.0"]
        PROM["Prometheus<br/>/metrics"]
        IMG["/tmp/output.jpg<br/>+ image-stream-server"]
    end
    subgraph L5["5. Consumer / presentation"]
        direction LR
        HTMLAPP["HTML5 app (roHtmlWidget)"]
        BAC["BrightAuthor:connected"]
        DASH["Go dashboard / Grafana"]
        SUBS["External MQTT subscribers"]
    end
    L1 --> L2 --> L3
    CPP --> UDP --> HTMLAPP
    CPP --> UDP --> BAC
    CPP --> MQTT --> SUBS
    CPP --> PROM --> DASH
    CPP --> IMG --> HTMLAPP
```

### 1. Hardware

BrightSign Series 5 players use Rockchip SoCs with an integrated NPU. The NPU
accelerates the matrix multiplications and convolutions that dominate neural-network
inference, at much lower power than a CPU or GPU.

Per-SoC specs below are from the Argus extension READMEs (the current source of truth).

| SoC | NPU | Example players | Parallel models (Argus) |
| --- | --- | --- | --- |
| RK3588 | 3-core, 6 TOPS | XT5 | 2-3 models |
| RK3576 | 2-core, 4 TOPS | XS6 / XD6 (reported as "XS156") | 2 models |
| RK3568 | 1-core, 1 TOPS | LS5 / HS5 | 1 model |

Capacity scales with the NPU: the audience engine runs 2-3 models in parallel on an
XT5 but only 1 on an LS5/HS5. Feature coverage also varies by SoC -- the voice
extension and the LLM design currently target RK3588 (XT5) only.

### 2. BrightSignOS and the RKNN runtime

Inference uses Rockchip's RKNN runtime, `librknnrt.so`, shipped with BrightSignOS.
Models must be in Rockchip's `.rknn` format; they are compiled ahead of time from
ONNX with `rknn-toolkit2` (see [build-and-deploy.md](build-and-deploy.md)). LLM
workloads use the separate RKLLM runtime.

The Python CV extension requires BrightSignOS 9.1.79.3 or newer for its bundled
`librknnrt.so`.

### 3. The BSMP extension

A BrightSign Model Package bundles one or more `.rknn` models with the engine that
runs them. It is delivered as a BrightSignOS **extension**: a Linux squashfs
filesystem that extends the firmware and auto-starts at boot via `bsext_init`.

Engines are written in C++ (the inference pipelines) and, in the Argus family, Go
(supervisors, dashboards, and MQTT/metrics services). A typical vision engine uses a
producer-consumer threading model: one thread pulls camera frames, another runs NPU
inference, a third formats and publishes results.

### 4. Transport

Extensions do not render UI. They publish results, and separate consumers subscribe.

| Channel | Default | Payload | Consumed by |
| --- | --- | --- | --- |
| UDP localhost | 5000 | BrightScript-friendly text | `autorun.brs`, BA:connected |
| UDP localhost | 5002 | JSON | HTML5 presentations |
| MQTT (Argus) | 1883 (TCP), 9001 (WebSockets) | `analytics/v7.0` JSON | external subscribers, dashboard |
| Prometheus | 9101 (Argus) / 9400 (NPU exporter) | metrics exposition | Prometheus / Grafana |
| Image | `/tmp/output.jpg` + port 20200 | decorated JPEG | presentations, visual debug |
| Dashboard | 20300 | HTML "mission control" / zone config | browser |
| Health (rtsp-watcher) | 9200 `/healthz` | per-stream state / capacity | operators |

Ports vary by project; the project README is authoritative. The gaze/object/voice
extensions use the UDP 5000/5002 contract; the Argus family uses MQTT + Prometheus
instead. `argus-rtsp-watcher` serves per-stream MJPEG at `20200 + stream index`. Some
design documents (LPR, LLM) use 5001/5003 instead of 5000/5002.

### 5. Consumer / presentation

The visible experience runs in one of:

- An **HTML5 app** in `roHtmlWidget`, launched by `autorun.brs`, that polls
  `/tmp/output.jpg` and listens on a UDP port to overlay results or switch content.
- A **BrightAuthor:connected** presentation that switches content based on the UDP
  values (for example, play a different video when one vs. more than one person looks
  at the screen).
- A **dashboard** -- the Argus Go `dashboard-server`, or an external Prometheus /
  Grafana stack -- for analytics and operations.

## Runtime data flow

```mermaid
sequenceDiagram
    participant Cam as Camera (USB / CSI / RTSP)
    participant Eng as NPU engine (C++)
    participant NPU as RKNN runtime
    participant Pres as Presentation (HTML / BA:connected)
    participant Ext as External subscribers (MQTT / Prometheus)

    Cam->>Eng: frame
    Eng->>NPU: preprocessed tensor
    NPU-->>Eng: detections (boxes, classes, landmarks)
    Eng->>Eng: track IDs, gaze, dwell, direction
    Eng->>Pres: UDP 5000 / 5002 results
    Eng->>Pres: /tmp/output.jpg (decorated)
    Eng->>Ext: MQTT analytics/v7.0
    Eng->>Ext: Prometheus /metrics
    Pres->>Pres: overlay boxes / switch content
```

## Build pipeline

Example repos share one cross-compilation pipeline so the heavy toolchain and model
assets are built once and reused.

```mermaid
flowchart LR
    SRC["BrightSignOS source<br/>(S3)"] --> SDKB["brightsign-sdk-builder<br/>provisions cobra aarch64 SDK<br/>+ RKNN toolkit"]
    SDKB --> CACHE["argus-build-cache<br/>shared SDK sysroots +<br/>compiled .rknn models"]
    ZOO["ONNX models<br/>(rknn_model_zoo)"] --> COMPILE["rknn-toolkit2<br/>ONNX -> .rknn"]
    COMPILE --> CACHE
    CACHE --> EXT["Extension repo build<br/>C++/Go cross-compile +<br/>package"]
    EXT --> BSFW[".bsfw (signed, production)"]
    EXT --> LVM["LVM ext zip (dev)"]
    BSFW --> PLAYER["BrightSign player"]
    LVM --> PLAYER
```

- **brightsign-sdk-builder** provisions a build host once with the aarch64
  cross-compile SDK and the RKNN toolkit, driven by a Makefile and Yocto /
  OpenEmbedded recipes.
- **argus-build-cache** holds the shared, box-level assets: compiled SDK sysroots and
  per-SoC `.rknn` models (YOLOX, RetinaFace, Whisper, a MobileNetV2 embedding, an
  RKLLM Qwen2.5 model, and a Piper TTS runtime). Consumer repos point at it with
  `BS_CACHE_DIR` / `ARGUS_CACHE_DIR` (default `../argus-build-cache`).
- Each **extension repo** cross-compiles its engine against the cache, compiles or
  reuses the needed models, and packages a `.bsfw` (production) and an LVM extension
  zip (dev).

The three model extensions (gaze, object, voice) follow an identical sequence:
`./setup` -> `./compile-models` -> `./build --extract-sdk` -> `./build-apps` ->
`./package`. Full detail in [build-and-deploy.md](build-and-deploy.md).

## Repository ecosystem

```mermaid
flowchart TB
    GEN["brightsign-npu-general<br/>(docs hub)"]
    TMPL["extension-template"]
    SDKB["brightsign-sdk-builder"]
    CACHE["argus-build-cache"]

    subgraph BSMP["Model packages"]
        GAZE["gaze-extension"]
        OBJ["object-extension"]
        VOICE["voice-extension"]
        ARGUS["argus-audience-measurement"]
        RTSP["argus-rtsp-watcher"]
    end
    subgraph PRES["Presentations / apps"]
        SHTML["simple-gaze-detection-html"]
        SPRES["simple-gaze-detection-presentation"]
        ATTN["argus-attention-demo-html"]
    end
    subgraph DEV["Dev platforms / tools"]
        PYCV["python-cv-dev-extension"]
        DOCKER["npu-docker-extension"]
        METRICS["npu--metrics-exporter"]
        IMGSRV["bs-image-stream-server"]
    end

    TMPL --> GAZE & OBJ & VOICE
    SDKB --> CACHE
    CACHE --> GAZE & OBJ & VOICE & ARGUS & RTSP
    ARGUS --> RTSP
    GAZE --> SHTML & SPRES
    ARGUS --> ATTN
    GAZE & OBJ & VOICE & ARGUS --> IMGSRV
```

Argus is the most complete family: `argus-audience-measurement-extension` is the core
engine, `argus-rtsp-watcher` reuses it for multiple RTSP cameras, and the HTML demos
are its signage front-ends.

## Design decisions worth knowing

- **Privacy by design.** Vision engines process frames on-device and publish only
  derived metrics (counts, gaze, dwell). No images leave the player, and the audience
  engine does no facial recognition.
- **Separation of inference and presentation.** The UDP/MQTT contract lets the same
  BSMP drive an HTML app, a BA:connected presentation, or an external dashboard
  without changing the engine.
- **Shared build cache.** Cross-compiling the SDK and models is expensive; building
  once and sharing via `argus-build-cache` keeps per-project builds fast.
