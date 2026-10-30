# 👓 SecondSight

**A little more context. A little less guesswork.**

![Stage: Architecture and planning](https://img.shields.io/badge/Stage-Architecture%20%26%20planning-8b5cf6)
![Inference: On-device](https://img.shields.io/badge/Inference-On--device-10b981)
![Core: C++17](https://img.shields.io/badge/Core-C%2B%2B17-2563eb)
![Platforms: XR and Android](https://img.shields.io/badge/Platforms-XR%20%26%20Android-f59e0b)

SecondSight is an assistive perception project for wearable and mobile devices. It is designed to help people notice non-verbal cues during everyday conversations — changes in gaze, facial movement, posture, and voice — through subtle visual and audio feedback.

The goal: make communication more accessible, with local processing, clear uncertainty, and control in the user's hands.

## ✨ Key features

- 👁️ **Multimodal perception** — independent face, gaze, body, and audio paths combine observations with confidence and input-quality scores.
- 🧩 **Portable core** — C++17 interfaces, sensor adapters, replaceable ML runtimes, and a C ABI separate the pipeline from platform integration.
- ⚡ **Responsive processing** — latency budgets, frame scheduling, and adaptive model selection account for limited compute and thermal constraints.
- 🛡️ **Privacy-first operation** — on-device inference, no identity recognition or cross-session identity linking, and no raw recording in default assistive mode.
- 🎚️ **Subtle feedback** — temporal smoothing and hysteresis stabilize cues; a policy gate filters outputs before presentation.
- 🛟 **Graceful degradation** — missing sensors and unreliable signals reduce what the system reports, with an emergency disable control always available.

## 🔄 How it works

```mermaid
flowchart LR
    S["📷 Sensors"] --> P["⚙️ Pre-processing"]
    P --> M["🧠 Perception models"]
    M --> F["🧩 Signal fusion"]
    F --> T["⏱️ Temporal smoothing"]
    T --> G["🛡️ Policy gate"]
    G --> U["💬 Visual / audio cues"]

    classDef input fill:#dbeafe,stroke:#2563eb,color:#172554
    classDef processing fill:#ede9fe,stroke:#8b5cf6,color:#2e1065
    classDef control fill:#d1fae5,stroke:#059669,color:#064e3b
    classDef output fill:#fef3c7,stroke:#d97706,color:#78350f
    class S input
    class P,M,F,T processing
    class G control
    class U output
```

## 🛠️ Planned stack

| Layer | Technologies |
|-------|--------------|
| 🧩 Core & platform integration | C++17, Kotlin, Android NDK, OpenXR |
| 🧠 ML inference | TensorFlow Lite / ONNX Runtime |
| 🔊 Audio | Oboe |
| 📦 Build | Gradle, CMake |

## 📚 Documentation

- 🗺️ [System design](doc/design.md) — requirements, architecture, trade-offs, and data contracts.
- 🔌 [Hardware abstraction](doc/low-level-design/4.1-device-compatibility.md) — adapters and capability discovery.
- 🧠 [Perception models](doc/low-level-design/4.4-perception-models.md) · [Signal fusion](doc/low-level-design/4.5-signal-fusion.md)
- 🛡️ [Policy gate](doc/low-level-design/4.7-policy-gate.md) · [Privacy & consent](doc/low-level-design/4.12-privacy-consent-ux.md)
- ⚡ [Performance](doc/low-level-design/4.13-performance-latency.md) · [Error handling](doc/low-level-design/4.14-error-handling.md)

## 🚧 Roadmap

Current stage: architecture and implementation planning. Application code and measured benchmarks are still to come.

- [ ] Build a minimal sensor-to-cue prototype.
- [ ] Validate public SDK access, model licenses, and runtime compatibility.
- [ ] Measure latency, sustained throughput, and thermal behavior.
- [ ] Evaluate cue usefulness and model bias with consenting participants.

Observable cues are the starting point. Emotion, intent, and deception inference discussed in the design remain unvalidated research ideas; confidence scores alone do not establish their validity. Optional recording modes require separate consent and retention controls.
