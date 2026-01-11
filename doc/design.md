# SecondSight — System Design

---

## Table of Contents

- [Intro](#intro)
- [0. Documentation Style & Technical Writing Guidelines](#0-documentation-style--technical-writing-guidelines)
- [1. Problem & Project Scope](#1-problem--project-scope)
- [2. High-Level Design](#2-high-level-design)
- [3. Data Model](#3-data-model)
- [4. Low Level Design](#4-low-level-design)
  - [4.1 Device Compatibility & Hardware Abstraction](#41-device-compatibility--hardware-abstraction)
  - [4.2 Perception Models](#42-perception-models)
  - [4.3 Raw Data Storage (Mode-Gated)](#43-raw-data-storage-mode-gated)
- [Appendix A: Signal Vocabulary](#appendix-a-signal-vocabulary)
- [Appendix B: Facial Action Units (FAUs) Reference](#appendix-b-facial-action-units-faus-reference)
- [Appendix C: Terminology](#appendix-c-terminology)
- [Appendix D: Bibliography & Key Concepts](#appendix-d-bibliography--key-concepts)

---

<details id="intro">
<summary><strong>Intro</strong></summary>

SecondSight is an XR assistive perception application for devices such as Meta Quest, Ray-Ban Meta smart glasses, and Android smartphones. It augments a user's real-world interactions by detecting observable non-verbal signals such as gaze direction patterns, facial action cues, posture shifts, and turn-taking or voice prosody where available — and delivering subtle, non-authoritative hints through visual, audio, or multimodal feedback depending on device capabilities.

### Scope and Intent

**SecondSight is designed to:**

- Detect internal user states (emotions, intent, deception indicators) from observable cues
- Use movement detection (FAUs, body language, prosody) as intermediate signals for state inference
- Provide real-time feedback to improve situational awareness of non-verbal communication
- Operate in a way that prioritizes privacy, consent, and responsible use

**SecondSight is not designed to:**

- Identify people — no face recognition, no identity linking
- Record or store raw sensor streams by default
- Replace professional assessment (clinical, legal, security)

### Ethical and Legal Guardrails

- **Probabilistic outputs** — all state inferences include confidence levels; never absolute claims
- **Privacy first** — prefer on-device processing, no raw storage
- **Ephemeral inference** — real-time cues, no replay, no sharing
- **Transparency** — always communicate uncertainty and limitations
- **User control** — easy disable, adjustable sensitivity, clear feedback

</details>

---

<details id="0-documentation-style--technical-writing-guidelines">
<summary><strong>0. Documentation Style & Technical Writing Guidelines</strong></summary>

- Use **clear, concise, and unambiguous language**
- Prefer **bullet points** over long paragraphs
- Highlight **key terms** in **bold**
- Use **tables** for comparisons, trade-offs, and structured data
- Use **Mermaid** for architecture diagrams with high contrast colors and very short text in boxes. Add comments below the diagram if necessary.
- Keep diagrams **high-level first**, add detail in later sections
- Explicitly document **assumptions**, **constraints**, and **out-of-scope** items
- Mark open questions with **TBD**

</details>

---

<details id="1-problem--project-scope">
<summary><strong>1. Problem & Project Scope</strong></summary>

### Core Problem

Most of the inter-personal communication we do is non-verbal. We communicate through our facial expressions, body language, and tone of voice. We do this unconsciously, and often without realizing it. We use these non-verbal cues to understand others, to express ourselves, and to build relationships. But sometimes, we can't read the cues we're getting from others. We can't understand what they're feeling, what they're thinking, or what they're trying to communicate. This can lead to miscommunication, confusion, and even conflict.

**SecondSight** provides real-time, on-device detection of internal states (emotions, intent, deception indicators) from observable non-verbal signals.

### Functional Requirements

| ID | Requirement | Priority |
|----|-------------|----------|
| **F1** | No-go inference enforcement (hard block on recognition/identity) | P0 |
| **F2** | Privacy and consent UX (onboarding, explicit toggles) | P0 |
| **F3** | Real-time perception pipeline (camera, IMU, optional mic) | P0 |
| **F4** | Signal categorization and temporal aggregation with confidence bounds | P0 |
| **F5** | Subtle, non-authoritative user feedback with user controls | P0 |
| **F6** | Multi-device support (Quest, Ray-Ban Meta glasses, Android smartphones) with visual/audio/multimodal feedback | P0 |
| **F7** | Graceful degradation when sensors or compute are limited | P1 |
| **F8** | Operating modes: Assistive (default), Training/Reflection, Research (opt-in) | P2 |

**Priority Key:** P0 = Critical | P1 = High | P2 = Medium

### Non-Functional Requirements

| Category | Requirement |
|----------|-------------|
| **Latency** | < 250 ms lightweight cues, < 500 ms heavier perception |
| **Performance** | Dynamic quality scaling based on device thermal state |
| **Privacy** | On-device inference, no raw sensor storage, opt-in anonymized telemetry only |
| **Reliability** | Conservative behavior under poor signal quality; explicit "insufficient data" states |
| **Transparency** | Explainable cues; persistent reminder of probabilistic nature |
| **Safety** | Low salience defaults; immediate disable mechanism |
| **Compliance** | Documentation-ready data flows; DPIA-style review support |

### Scope Agreement

#### In Scope

- Real-time detection of **internal states**: emotions, intent indicators, deception cues
- **Observable signals as intermediary**: gaze proxy, facial action, pose, prosody
- On-device processing with privacy-first architecture
- Assistive, Training, and Research modes
- **1:1 interaction model** (UserA ↔ single UserB)

#### Out of Scope

- Face recognition or identity tracking
- Cloud-based inference or raw data upload
- Individual scoring, ranking, or profiling
- Clinical diagnosis or legal/security assessment
- **Multi-party tracking** (UserC, UserD, etc.) — future scope

### Ethical & Legal Guardrails

| Guardrail | Description |
|-----------|-------------|
| Probabilistic outputs | All inferences include confidence levels; never absolute claims |
| Privacy first | On-device processing, no raw storage |
| Ephemeral inference | Real-time cues only; no replay, no sharing |
| Transparency | Always communicate uncertainty and limitations |
| No identity tracking | Never identify individuals or link across sessions |

### 1.5 State of the Art: Real-Time Social Perception Applications

This section surveys existing applications for real-time non-verbal signal detection, covering both **facial analysis** and **body language** (pose, gesture, posture). Understanding this landscape is critical for positioning SecondSight.

### Facial Analysis — XR/VR

| Product | Platform | Claims | Status | Notes |
|---------|----------|--------|--------|-------|
| **Meta Quest Pro Face Tracking** | Quest Pro | Real-time facial expression tracking for avatar animation | Commercial (2022+) | FACS-based; focuses on animation, not emotion |
| **Emteq Pro** | Pico, HTC Vive (face mask) | EMG, PPG, eye tracking for emotional well-being | Commercial reference design | Research-oriented; limited consumer traction |
| **Floreo VR** | Various VR | VR learning tools for autism; social/communication skills | Commercial | Used by schools, therapy; NIH studies |

### Facial Analysis — Smart Glasses & Mobile

| Product | Platform | Claims | Status | Notes |
|---------|----------|--------|--------|-------|
| **Google Superpower Glass** | Google Glass | Emoji overlays for emotion recognition; autism assistance | Research prototype (Stanford) | Limited follow-up after initial research |
| **Brain Power (Empowered Brain)** | Google Glass | Autism emotional recognition via emojis | Commercial (limited) | Partnership with Affectiva |
| **Affectiva Emotion AI SDK** | iOS/Android | Facial expression recognition and emotion classification | Commercial SDK | Acquired by Smart Eye (2021); bias concerns |
| **Microsoft Azure Face API** | Cloud | Emotion detection from facial images | **Deprecated (2022)** | Removed citing scientific validity concerns |

### Body Language & Pose Estimation — XR/VR

| Product | Platform | Claims | Status | Notes |
|---------|----------|--------|--------|-------|
| **Meta Quest Body Tracking** | Quest 3, Quest Pro | Full-body pose estimation from headset cameras | Commercial (2023+) | Inside-out tracking; powers avatar animation |
| **Meta Hand Tracking** | Quest, Ray-Ban Meta | Real-time hand pose and gesture recognition | Commercial | Supports microgestures on smart glasses |
| **Apple Vision Pro** | visionOS | Hand and eye tracking for spatial input | Commercial (2024) | Gesture-based UI; no body tracking |
| **HTC Vive Full Body Trackers** | Vive | Additional trackers for legs/feet | Commercial | Requires external trackers |

### Body Language & Pose Estimation — Mobile/Desktop

| Product | Platform | Claims | Status | Notes |
|---------|----------|--------|--------|-------|
| **MediaPipe Holistic** | Cross-platform | Real-time face, hand, and body landmarks (540+ keypoints) | Open source (Google) | Runs on mobile; widely used in research |
| **OpenPose** | Desktop | Multi-person pose estimation | Open source (CMU) | Pioneering; GPU-intensive |
| **BlazePose GHUM** | Mobile | Real-time 3D body + hand landmarks | Research (Google) | Optimized for mobile devices |
| **MoveNet** | TensorFlow | Fast pose estimation for fitness/sports | Open source (Google) | Thunder/Lightning variants for speed/accuracy |

### Gesture Recognition

| Product | Platform | Claims | Status | Notes |
|---------|----------|--------|--------|-------|
| **Meta Ray-Ban Hand Gestures** | Ray-Ban Meta glasses | CV-based hand gestures for music, AI control | Prototype (2025) | Usability challenges noted in research |
| **Leap Motion Controller** | Desktop/VR | High-precision hand tracking | Commercial | Acquired by Ultraleap |
| **Manomotion** | Mobile | Gesture recognition SDK | Commercial | AR/VR applications |

### Presentation & Communication Coaching

| Product | Platform | Claims | Status | Notes |
|---------|----------|--------|--------|-------|
| **Orai** | iOS/Android | AI speech coach; analyzes pace, filler words, energy | Commercial | Focus on public speaking |
| **Yoodli** | Web/Mobile | AI communication coach; body language feedback | Commercial | Analyzes posture, gestures, eye contact |
| **Poised** | Desktop | Real-time meeting coach | Commercial | Analyzes speech patterns, filler words |
| **Speeko** | iOS/Android | Speech and presentation training | Commercial | Gamified learning |

### Research Prototypes (Body/Pose)

| Project | Organization | Focus | Status |
|---------|--------------|-------|--------|
| **Stanford Autism Glass** | Stanford | Behavioral coaching via Google Glass | Published research (2016–2020) |
| **XR-Poser** | Meta/KAUST | Egocentric body tracking for AR/VR | Research (2025) |
| **LabGraph PosAI** | Meta (open source) | Real-time pose recognition tool | Research prototype |
| **DiffusionPoser** | Academic | Body pose from sparse IMU sensors | CVPR 2024 |
| **IMUPoser** | Academic | Full-body pose from phones/watches/earbuds | CHI 2023 |

### Lie Detection / Deception Products

| Finding |
|---------|
| **No credible consumer products identified.** Academic research confirms humans perform at chance level for lie detection. The scientific basis for automated deception detection remains unsupported. |

### Key Criticisms and Controversies

| Issue | Details |
|-------|---------|
| **Scientific validity** | Microsoft deprecated its emotion API (2022) citing concerns about inferring internal states from facial expressions. |
| **Demographic bias** | Facial and body models perform inconsistently across ethnicities, ages, genders, and body types. |
| **Privacy concerns** | Always-on sensing raises concerns for bystanders who haven't consented. |
| **Regulatory landscape** | EU AI Act (2024) classifies emotion recognition in certain contexts as high-risk or prohibited. |

### Lessons for SecondSight: Learning from Industry Approaches

| Industry Challenge | How SecondSight Addresses It |
|-------------------|------------------------------|
| **Scientific validity concerns** (Microsoft deprecated API) | **Probabilistic outputs with confidence levels.** All state inferences include uncertainty. Never absolute claims. |
| **Face-only analysis** (most emotion AI) | **Face + Body fusion.** Independent models for face and body, combined at fusion. Either can work alone (graceful degradation). |
| **Cloud processing** (Affectiva, Azure) | **100% on-device.** No raw data leaves the device. No cloud dependency. Privacy by architecture, not policy. |
| **Black-box outputs** (most commercial APIs) | **Transparent uncertainty.** Every inference includes reliability score. Clear communication of confidence levels. |
| **Poor demographic generalization** (bias in training data) | **Fine-tunable models.** Open-source approach enables retraining on diverse datasets for fairness. |
| **Always-on surveillance framing** | **Assistive framing.** Opt-in, user-controlled, ephemeral. Designed as a personal tool, not surveillance. |
| **No identity protection** | **No identity tracking.** Never identify individuals or link across sessions. Face detection, not recognition. |

---

### Apps to Try (Hands-On Research)

The following apps are recommended for hands-on exploration of the current state of the art.

#### Pose & Body Tracking

| App | Platform | What to Evaluate |
|-----|----------|------------------|
| **MediaPipe Holistic Demo** | Web | Body + hand + face landmark accuracy; latency |
| **Move.ai** | iOS | Markerless motion capture from phone |
| **Plask** | Web | AI motion capture from video |
| **Rokoko Video** | iOS/Android | Free motion capture from video |

#### Gesture Recognition Apps

| App | Platform | What to Evaluate |
|-----|----------|------------------|
| **Meta Quest Hand Tracking Demos** | Quest | Gesture recognition accuracy; latency |
| **Leap Motion Playground** | Desktop | High-precision hand tracking |
| **FingerPose** | Web (JS) | Custom gesture recognition |

#### Communication Coaching

| App | Platform | What to Evaluate |
|-----|----------|------------------|
| **Yoodli** | Web/iOS | Body language feedback quality |
| **Orai** | iOS/Android | Speech pattern analysis |
| **Poised** | Desktop | Real-time meeting feedback |
| **Speeko** | iOS/Android | Presentation training UX |

#### Autism/Social Skills

| App | Platform | What to Evaluate |
|-----|----------|------------------|
| **Floreo** | VR | Therapeutic VR for social skills |
| **Brain Power (Empowered Brain)** | Google Glass | Emoji-based emotion feedback |

#### Developer SDKs

| SDK | Platform | What to Evaluate |
|-----|----------|------------------|
| **MediaPipe** | Cross-platform | Pose, face, hand tracking APIs |
| **OpenPose** | Desktop | Multi-person pose estimation |
| **Meta Movement SDK** | Quest | Body and hand tracking APIs |
| **ARKit Body Tracking** | iOS | Apple's body tracking capabilities |
| **ARCore Augmented Faces** | Android | Google's face mesh tracking |

</details>

---

<details id="2-high-level-design">
<summary><strong>2. High-Level Design</strong></summary>

### Core Idea

SecondSight is a real-time perception pipeline with a **mandatory policy gate** between inference and user feedback. All processing happens on-device. No raw sensor data is stored.

The system uses **four independent perception paths**:

- **Face signals** — Facial Action Units (FAUs), expression dynamics
- **Eye signals** — Gaze direction, attention patterns
- **Body signals** — Pose, gestures, posture shifts, orientation
- **Audio signals** — Prosody, tempo, turn-taking cues

Each path has its own dedicated model. Results are combined at the Signal Fusion stage before the Policy Gate.

### Architecture Diagram

```mermaid
flowchart TB
    subgraph INPUTS["📡 Inputs"]
        CAM["📷 Camera"]
        IMU["🔄 IMU"]
        MIC["🎤 Microphone"]
    end

    subgraph PROCESSING["⚙️ Perception Pipeline"]
        direction TB
        PRE["🔧 Pre-processing"]

        subgraph PERCEPTION["🧠 Perception"]
            subgraph FEAT["📊 Feature Extraction"]
                FACE_FEAT["😐 Face"]
                EYE_FEAT["👁️ Gaze"]
                BODY_FEAT["🧍 Body"]
                AUDIO_FEAT["🎵 Audio"]
            end

            subgraph INF["🤖 Inference"]
                FACE_INF["😐 Face Model"]
                EYE_INF["👁️ Gaze Model"]
                BODY_INF["🧍 Body Model"]
                AUDIO_INF["🎵 Audio Model"]
            end

            FUSION["🔀 Signal Fusion"]
        end

        TEMP["⏱️ Temporal Aggregation"]
    end

    subgraph SAFETY["🛡️ Policy & Safety"]
        GATE["🚦 Policy Gate"]
    end

    subgraph OUTPUT["💬 User Feedback"]
        VIS["👓 Visual"]
        AUD["🔊 Audio"]
    end

    CAM --> PRE
    IMU --> PRE
    MIC --> PRE

    PRE --> FACE_FEAT --> FACE_INF --> FUSION
    PRE --> EYE_FEAT --> EYE_INF --> FUSION
    PRE --> BODY_FEAT --> BODY_INF --> FUSION
    PRE --> AUDIO_FEAT --> AUDIO_INF --> FUSION

    FUSION --> TEMP --> GATE
    GATE --> VIS
    GATE --> AUD

    style GATE fill:#cc0000,stroke:#000,stroke-width:3px,color:#fff
    style SAFETY fill:#660000,stroke:#cc0000,stroke-width:2px,color:#fff
    style PERCEPTION fill:#0066cc,stroke:#000,stroke-width:2px,color:#fff
    style FACE_INF fill:#004d99,stroke:#000,color:#fff
    style EYE_INF fill:#660066,stroke:#000,color:#fff
    style BODY_INF fill:#006600,stroke:#000,color:#fff
    style AUDIO_INF fill:#996600,stroke:#000,color:#fff
    style FUSION fill:#cc6600,stroke:#000,color:#fff
    style INPUTS fill:#333,stroke:#fff,color:#fff
    style PROCESSING fill:#1a1a1a,stroke:#fff,color:#fff
    style OUTPUT fill:#333,stroke:#fff,color:#fff
    style TEMP fill:#444,stroke:#fff,color:#fff
    style PRE fill:#444,stroke:#fff,color:#fff
```

### Key Design Decisions

| Decision | Rationale | Alternatives Rejected |
|----------|-----------|----------------------|
| On-device processing | Privacy first, no cloud dependency, low latency | Cloud inference (privacy risk, latency) |
| Policy gate as hard requirement | Prevents forbidden inferences (identity, absolute claims) from reaching user | Soft filtering (risk of leakage) |
| Probabilistic outputs with confidence | Acknowledges inherent uncertainty in state detection | Absolute claims (scientifically unsound) |
| Temporal smoothing | Reduces noise, prevents jittery feedback | Raw frame-by-frame (too noisy) |
| Graceful degradation | Works across device capabilities | Fail-hard (poor UX) |

> **Design note:** ML models are **independent** — each can run, fail, or degrade separately. Signal Fusion combines their outputs but does not require both. This enables graceful degradation (e.g., body-only when face is occluded).

### Optional Modules

| Module | Description | Privacy Note |
|--------|-------------|--------------|
| Local Summaries | Post-session reflection for training mode | No raw media stored |
| Telemetry | Performance metrics | Opt-in, aggregated, anonymized only |
| **Raw Data Storage** | Dev/eval only — stores sensor frames and pipeline outputs | ⚠️ **DEV ONLY** — disabled in production builds |

### Sequence Diagram — Real-Time Perception Flow

```mermaid
sequenceDiagram
    participant S as Sensors
    participant P as Pre-processing
    participant F as Feature Extraction
    participant I as Inference
    participant G as Policy Gate
    participant U as User Feedback

    loop Every frame (~30ms)
        S->>P: Raw frame + IMU
        P->>P: Quality check
        alt Good quality
            P->>F: Stabilized frame
            F->>I: Features (pose, gaze, FACs)
            I->>G: Signal + confidence
            G->>G: No-go check
            alt Allowed signal
                G->>U: Safe cue
            else Blocked signal
                G--xU: (suppressed)
            end
        else Poor quality
            P--xF: (frame dropped)
        end
    end
```

### Assumptions

- Device has sufficient compute for on-device ML inference
- User has completed onboarding and consented to processing

#### User Controls

Users interact with the system through `UserSettings` (persistent preferences) and `UserControls` (runtime actions). Key capabilities:

- **Master toggle** — Enable/disable all processing
- **Mode selection** — Assistive, training, or research mode
- **Feedback intensity** — Adjust cue salience
- **Emergency disable** — Immediate, persistent off

</details>

---

<details id="3-data-model">
<summary><strong>3. Data Model</strong></summary>

This section defines all data types and contracts for the SecondSight perception pipeline. All communication is on-device; there is no external network API. See **Appendix C** for terminology (UserA, UserB).

> ⚠️ **Critical Clarification:** All perception (face tracking, body tracking, hand tracking, gaze estimation) refers to **UserB** — the person being observed. UserA (the device wearer) is the consumer of the feedback, not the subject of perception. The only data from UserA used in the pipeline is IMU data for motion compensation/stabilization.

### Pipeline Data Flow

```mermaid
flowchart TB
    subgraph INPUT["📡 Sensors"]
        CAM["📷 Camera"]
        IMU["🔄 IMU"]
        MIC["🎤 Mic"]
    end

    SF["SensorFrame"]

    PF["ProcessedFrame"]

    subgraph FEATURES["📊 Feature Extraction"]
        FFV["FaceFeatureVector"]
        BFV["BodyFeatureVector"]
        PRS["ProsodyFeatures"]
    end

    subgraph INFERENCE["🤖 Inference"]
        FSO["FaceSignalOutput"]
        BSO["BodySignalOutput"]
    end

    FUSO["FusedSignalOutput"]

    GATE["🚦 Policy Gate"]

    SC["SafeCue"]

    subgraph OUTPUT["💬 User Feedback"]
        VIS["👓 Visual"]
        AUD["🔊 Audio"]
    end

    CAM & IMU & MIC --> SF
    SF --> PF
    PF --> FFV & BFV & PRS
    FFV --> FSO
    BFV --> BSO
    PRS --> FUSO
    FSO & BSO --> FUSO
    FUSO --> GATE
    GATE --> SC
    SC --> VIS & AUD

    style INPUT fill:#333,stroke:#fff,color:#fff
    style SF fill:#444,stroke:#fff,color:#fff
    style PF fill:#555,stroke:#fff,color:#fff
    style FEATURES fill:#0066cc,stroke:#000,color:#fff
    style FFV fill:#004d99,stroke:#000,color:#fff
    style BFV fill:#006600,stroke:#000,color:#fff
    style PRS fill:#996600,stroke:#000,color:#fff
    style INFERENCE fill:#005500,stroke:#000,color:#fff
    style FSO fill:#004d99,stroke:#000,color:#fff
    style BSO fill:#006600,stroke:#000,color:#fff
    style FUSO fill:#cc6600,stroke:#000,color:#fff
    style GATE fill:#cc0000,stroke:#000,stroke-width:3px,color:#fff
    style SC fill:#cc6600,stroke:#000,color:#fff
    style OUTPUT fill:#333,stroke:#fff,color:#fff
```

### Quality & Reliability Flow

All feature vectors and signals include quality/reliability scores (0.0–1.0) that propagate through the pipeline:

| Stage | Metric | Description |
|-------|--------|-------------|
| Sensor | `signal_quality` | Composite quality (lighting, occlusion, blur) |
| Features | `face_quality` / `body_quality` | Modality-specific quality (visibility, angle, truncation) |
| Inference | `reliability` | Model confidence × input quality |
| Output | `salience` | Presentation weight (strength × reliability) |

---

### Core Data Types

#### Input Types

**SensorFrame** — Raw sensor data captured at each frame interval, observing UserB.

| Attribute | Type | Description |
|-----------|------|-------------|
| `timestamp_ns` | int64 | Device monotonic clock in nanoseconds (boot time reference) |
| `camera_frames` | CameraFrame[] | Array of camera images (1 for mono, 2 for stereo) |
| `depth_frame` | DepthFrame? | Optional depth map (available on Quest, some phones) |
| `imu_data` | IMUReading | Accelerometer + gyroscope for UserA head motion |
| `audio_chunk` | AudioBuffer? | Optional audio buffer (UserB speech, if mic enabled) |

**CameraFrame** — A single camera image with associated metadata.

| Attribute | Type | Description |
|-----------|------|-------------|
| `image` | RGBFrame | Raw RGB pixel data |
| `camera_id` | CameraPosition | Which camera: `left`, `right`, or `center` |
| `width` | int32 | Image width in pixels |
| `height` | int32 | Image height in pixels |
| `intrinsics` | CameraIntrinsics | Focal length, principal point, distortion coefficients |
| `extrinsics` | CameraPose? | Position and orientation relative to device origin |

**CameraIntrinsics** — Camera calibration parameters (pinhole + distortion model).

| Attribute | Type | Description |
|-----------|------|-------------|
| `fx` | float | Focal length in pixels (x-axis) |
| `fy` | float | Focal length in pixels (y-axis) |
| `cx` | float | Principal point x-coordinate |
| `cy` | float | Principal point y-coordinate |
| `distortion_model` | DistortionModel | Model type: `none`, `radial`, `fisheye` |
| `distortion_coeffs` | float[] | Distortion coefficients (model-dependent) |

**CameraPose** — 6-DoF camera pose relative to device origin.

| Attribute | Type | Description |
|-----------|------|-------------|
| `position` | Vec3 | Translation [x, y, z] in meters |
| `orientation` | Quaternion | Rotation as [w, x, y, z] quaternion |

**DepthFrame** — Depth map with per-pixel distance values.

| Attribute | Type | Description |
|-----------|------|-------------|
| `depth_data` | float[] | Per-pixel depth values in meters (row-major) |
| `width` | int32 | Depth map width in pixels |
| `height` | int32 | Depth map height in pixels |
| `min_depth` | float | Minimum valid depth in meters |
| `max_depth` | float | Maximum valid depth in meters |
| `depth_type` | DepthType | Source: `stereo_computed`, `tof`, `lidar`, `structured_light` |
| `confidence_map` | float[]? | Optional per-pixel confidence (0.0–1.0) |
| `intrinsics` | CameraIntrinsics | Depth sensor intrinsics (for 3D unprojection) |

#### Processing Types

**ProcessedFrame** — Cleaned and stabilized frame of UserB, ready for feature extraction.

| Attribute | Type | Description |
|-----------|------|-------------|
| `timestamp_ns` | int64 | Preserved from source SensorFrame |
| `stabilized_frame` | RGBFrame | Motion-compensated image of UserB |
| `signal_quality` | float (0.0–1.0) | Composite quality score (lighting, occlusion, blur) |

#### Feature Types

**FaceFeatureVector** — Extracted facial features from UserB's face (independent of body).

| Attribute | Type | Description |
|-----------|------|-------------|
| `timestamp_ns` | int64 | Preserved from source frame |
| `facial_actions` | FAUVector | Facial Action Units from UserB (see Appendix B) |
| `gaze_proxy` | GazeEstimate | UserB head orientation + eye region cues |
| `face_quality` | float (0.0–1.0) | Face-specific quality (visibility, occlusion, angle) |

**BodyFeatureVector** — Extracted body features from UserB (independent of face).

| Attribute | Type | Description |
|-----------|------|-------------|
| `timestamp_ns` | int64 | Preserved from source frame |
| `pose` | PoseKeypoints | UserB body keypoints (17+ joints: shoulders, elbows, wrists, hips, etc.) |
| `hand_pose` | HandKeypoints? | UserB hand keypoints (21 per hand, if visible) |
| `head_orientation` | HeadPose | UserB head position and rotation (yaw, pitch, roll) |
| `gestures` | GestureVector? | Detected gesture primitives from UserB (hand movements, arm positions) |
| `body_quality` | float (0.0–1.0) | Body-specific quality (visibility, truncation, occlusion) |

**ProsodyFeatures** — Audio-derived features from UserB's speech (optional, if mic enabled).

| Attribute | Type | Description |
|-----------|------|-------------|
| `timestamp_ns` | int64 | Preserved from source audio chunk |
| `pitch` | float | Fundamental frequency (F0) of UserB's voice |
| `tempo` | float | Speech rate (syllables/sec) |
| `energy` | float | Volume/intensity |
| `voice_quality` | float (0.0–1.0) | Audio signal quality (SNR, clipping) |

#### Inference Types

**SignalOutput** — Raw inference from any perception model. Used for face, gaze, body, and audio signals.

| Attribute | Type | Description |
|-----------|------|-------------|
| `timestamp_ns` | int64 | Preserved from source features |
| `modality` | SignalModality | Source modality: `face`, `gaze`, `body`, `audio` |
| `signal_type` | SignalType | Detected signal category (see Appendix A) |
| `intensity` | float (0.0–1.0) | Magnitude of the detected signal |
| `reliability` | float (0.0–1.0) | Model confidence × input quality |

> **Note:** Each perception model produces `SignalOutput` instances. The `modality` field identifies the source (face, gaze, body, audio), while `signal_type` specifies the detected signal within that modality's vocabulary.

**FusedSignalOutput** — Combined signals from multiple modalities, ready for policy filtering.

| Attribute | Type | Description |
|-----------|------|-------------|
| `timestamp_ns` | int64 | Preserved from source signals |
| `signal_type` | SignalType | Unified signal category (see Appendix A) |
| `intensity` | float (0.0–1.0) | Fused magnitude (weighted by reliability) |
| `reliability` | float (0.0–1.0) | Combined reliability across contributing modalities |
| `contributing_sources` | SignalModality[] | Which modalities contributed to this fused signal |
| `source_weights` | SourceWeights | Per-modality contribution weights used in fusion |

**SourceWeights** — Per-modality weights used in signal fusion.

| Attribute | Type | Description |
|-----------|------|-------------|
| `face_weight` | float (0.0–1.0) | Weight applied to face signal (0.0 if unavailable) |
| `gaze_weight` | float (0.0–1.0) | Weight applied to gaze signal (0.0 if unavailable) |
| `body_weight` | float (0.0–1.0) | Weight applied to body signal (0.0 if unavailable) |
| `audio_weight` | float (0.0–1.0) | Weight applied to audio signal (0.0 if unavailable) |

#### Output Types

**SafeCue** — Policy-approved cue about UserB, safe to present to UserA.

| Attribute | Type | Description |
|-----------|------|-------------|
| `timestamp_ns` | int64 | Preserved from source signal |
| `cue_type` | CueType | Mapped to allowed vocabulary only (see Appendix A) |
| `salience` | float (0.0–1.0) | Combined strength × reliability for presentation |
| `modality` | enum | Output channel: `visual`, `audio`, or `both` |

### Configuration Types

**UserSettings** — Persistent preferences for UserA's feedback experience.

| Attribute | Type | Description |
|-----------|------|-------------|
| `enabled` | bool | Master on/off toggle for all processing |
| `mode` | enum | Operating mode: `assistive`, `training`, or `research` |
| `feedback_intensity` | float (0.0–1.0) | Global multiplier for cue salience |
| `feedback_modality` | enum | Preferred output: `visual`, `audio`, or `both` |
| `mute_until_ns` | int64? | Timestamp until which feedback is suppressed |

**UserControls** — Runtime actions available to UserA.

| Method | Description |
|--------|-------------|
| `toggle_enabled()` | Switch processing on or off |
| `set_mode(mode)` | Change operating mode |
| `adjust_intensity(delta)` | Increase or decrease feedback salience |
| `mute_for(duration_ns)` | Temporarily suppress all cues |
| `emergency_disable()` | Immediate, persistent off — requires explicit re-enable |

### Enumeration Types

| Enum | Values | Description |
|------|--------|-------------|
| `SignalModality` | `face`, `gaze`, `body`, `audio` | Source modality that produced the signal |
| `SignalType` | See Appendix A | Unified signal vocabulary across all modalities |
| `CueType` | See Appendix A | Allowed output vocabulary for user feedback |
| `OperatingMode` | `assistive`, `training`, `research` | System operating modes |
| `FeedbackModality` | `visual`, `audio`, `both` | Output channel preference |
| `CameraPosition` | `left`, `right`, `center` | Which camera in a multi-camera setup |
| `DistortionModel` | `none`, `radial`, `fisheye` | Lens distortion model type |
| `DepthType` | `stereo_computed`, `tof`, `lidar`, `structured_light` | Depth sensor technology |

</details>

---

<details id="4-low-level-design">
<summary><strong>4. Low Level Design</strong></summary>

This section provides detailed technical design, including class diagrams, interface definitions, and technology choices.

### Open Questions (TBD)

- Exact latency budgets per pipeline stage

<details id="41-device-compatibility--hardware-abstraction">
<summary><strong>4.1 Device Compatibility & Hardware Abstraction</strong></summary>

#### Supported Devices

| Device Class | Examples | Primary Use Case |
|--------------|----------|------------------|
| **VR Headsets** | Meta Quest 3, Quest Pro | Immersive training, research mode |
| **Smart Glasses** | Ray-Ban Meta | Everyday assistive use |
| **Smartphones** | Android devices | Accessible entry point, training mode |

#### Platform-Specific Tech Stacks

SecondSight supports two distinct platform families with different underlying SDKs:

| Platform Family | Devices | SDK / Framework | Key Capabilities |
|-----------------|---------|-----------------|------------------|
| **VR Devices** | Quest 3, Quest Pro | **OpenXR** | Full body tracking, hand tracking, face tracking, eye tracking, spatial anchors |
| **Wearables & Mobile** | Ray-Ban Meta glasses, Android smartphones | [**Meta Wearables Device Access Toolkit**](https://developers.meta.com/blog/introducing-meta-wearables-device-access-toolkit/) | Camera access, IMU, microphone, limited compute |

> **Reference:**

#### Hardware Abstraction Layer (HAL)

Despite different platform SDKs, the **Perception Layer receives identical input** regardless of device. This is achieved through a Hardware Abstraction Layer (HAL) that normalizes sensor data.

```mermaid
flowchart TB
    subgraph DEVICES["Device Layer"]
        direction LR
        VR["🥽 VR Headsets<br/>(OpenXR)"]
        GLASSES["🕶️ Smart Glasses<br/>(Wearables SDK)"]
        PHONE["📱 Smartphones<br/>(Wearables SDK)"]
    end

    subgraph HAL["Hardware Abstraction Layer"]
        direction TB
        VR_ADAPTER["OpenXR Adapter"]
        WEAR_ADAPTER["Wearables SDK Adapter"]
        NORMALIZER["Sensor Normalizer"]
    end

    subgraph PERCEPTION["🧠 Perception Layer"]
        UNIFIED["Unified SensorFrame"]
    end

    VR --> VR_ADAPTER
    GLASSES --> WEAR_ADAPTER
    PHONE --> WEAR_ADAPTER
    VR_ADAPTER --> NORMALIZER
    WEAR_ADAPTER --> NORMALIZER
    NORMALIZER --> UNIFIED

    style HAL fill:#663399,stroke:#000,stroke-width:2px,color:#fff
    style VR_ADAPTER fill:#4d2673,stroke:#000,color:#fff
    style WEAR_ADAPTER fill:#4d2673,stroke:#000,color:#fff
    style NORMALIZER fill:#336699,stroke:#000,color:#fff
    style PERCEPTION fill:#0066cc,stroke:#000,stroke-width:2px,color:#fff
    style UNIFIED fill:#004d99,stroke:#000,color:#fff
    style DEVICES fill:#1a1a1a,stroke:#fff,stroke-width:2px,color:#fff
```

#### HAL Design Principles

| Principle | Description |
|-----------|-------------|
| **Uniform Input Contract** | All platform adapters produce the same `SensorFrame` structure |
| **Graceful Degradation** | Missing sensors are represented as null/optional; Perception layer handles absence |
| **Quality Metadata** | Each adapter reports sensor quality scores for downstream reliability weighting |
| **No Platform Leakage** | Perception layer code has zero knowledge of underlying SDK |

#### Sensor Availability by Device

| Sensor | Quest 3/Pro | Ray-Ban Meta | Android Phone | Notes |
|--------|-------------|--------------|---------------|-------|
| **RGB Camera** | ✅ Full | ✅ Limited FOV | ✅ Full | Extract RGB from passthrough/front-facing cameras |
| **Face Tracking** | ✅ Quest Pro (native FAUs) | ❌ ML models | ❌ ML models | Native: HAL maps to FAU vector; ML: Perception layer runs inference |
| **Body Tracking** | ✅ Native keypoints | ❌ ML models | ❌ ML models | Native: HAL converts XR body pose; ML: Perception layer runs inference |
| **Hand Tracking** | ✅ Native pose + gestures | ⚠️ Limited | ❌ ML models | Native: HAL extracts hand pose; ML: Perception layer runs inference |
| **IMU** | ✅ Head tracking | ✅ Accel + Gyro | ✅ Accel + Gyro | Normalized to IMU-equivalent for motion compensation |
| **Microphone** | ✅ | ✅ | ✅ | Extract audio buffer for prosody analysis (UserB speech) |

> **Note:** When native tracking is unavailable (❌), the Perception layer runs on-device ML models on camera frames. This is more compute-intensive but produces equivalent `SensorFrame` data. All tracking (face, body, hand) refers to **UserB** — the observed person.

#### Implications for Perception Layer

| Scenario | Behavior |
|----------|----------|
| VR with native face tracking | HAL provides pre-computed FAUs → Face Feature Extraction bypassed |
| Glasses without native tracking | HAL provides raw frames → Face/Body models run full pipeline |
| Phone in training mode | HAL provides raw frames + audio → Full ML inference |
| Any device with degraded sensor | HAL sets low `signal_quality` → Perception layer applies higher uncertainty |

</details>

<details id="42-perception-models">
<summary><strong>4.2 Perception Models</strong></summary>

This section evaluates model options

This section evaluates model options for the core perception pipeline. SecondSight requires on-device, real-time inference for facial action units, body pose, gaze estimation, and audio prosody.

#### Evaluation Criteria

| Criterion | Weight | Description |
|-----------|--------|-------------|
| **On-Device Capability** | Critical | Must run locally; no cloud dependency |
| **Real-Time Performance** | Critical | ≥15 FPS on target devices (Quest, Ray-Ban Meta, Android) |
| **FAU/Signal Output** | Critical | Must output observable signals, not emotion labels |
| **Model Size** | High | <100MB preferred for mobile/XR deployment |
| **License** | High | Permissive (MIT, Apache 2.0); no GPL/AGPL |
| **Privacy** | High | No data exfiltration; inference-only |
| **Accuracy** | Medium | Good enough for assistive cues; not clinical-grade |
| **Fine-Tuning** | Medium | Ability to adapt to diverse populations |

#### Solution Options Overview

```mermaid
flowchart TB
    subgraph OPTIONS["Perception Model Options"]
        direction TB
        COMMERCIAL["💰 Commercial SDKs"]
        OPENSOURCE["🆓 Open-Source Models"]
        CUSTOM["🔧 Custom Implementation"]
    end

    subgraph COMMERCIAL_LIST["Commercial"]
        AFF["Affectiva/Smart Eye"]
        VISAGE["Visage Technologies"]
        NOLDUS["Noldus FaceReader"]
    end

    subgraph OPENSOURCE_LIST["Open-Source"]
        MP["MediaPipe Holistic"]
        IBUG["OpenIBUG AU Detection"]
        ARIA["Aria ET (Gaze)"]
    end

    subgraph CUSTOM_LIST["Custom"]
        CUST["Train from scratch"]
    end

    COMMERCIAL --> COMMERCIAL_LIST
    OPENSOURCE --> OPENSOURCE_LIST
    CUSTOM --> CUSTOM_LIST

    style COMMERCIAL fill:#996600,stroke:#000,color:#fff
    style OPENSOURCE fill:#006600,stroke:#000,color:#fff
    style CUSTOM fill:#660066,stroke:#000,color:#fff
```

#### Commercial SDK Comparison

| SDK | Status | Pricing | On-Device | FAU Support | Body/Pose | Real-Time | Privacy | Notes |
|-----|--------|---------|-----------|-------------|-----------|-----------|---------|-------|
| **Affectiva/Smart Eye** | ✅ Active (acquired 2021) | Per-seat license | ✅ Yes | ✅ 21 AUs | ❌ No | ✅ Yes | ✅ On-device option | Industry leader; automotive focus |
| **iMotions** | ✅ Active | Annual subscription | ❌ Cloud platform | ✅ Uses Affectiva (21 AUs) | ❌ Integrates 3P | ✅ Yes | ⚠️ Cloud-based | Research platform; sensor integration |
| **Noldus FaceReader** | ✅ Active | Per-seat license | ✅ Local | ✅ FACS-based | ❌ Limited | ✅ Yes | ✅ On-premise | Academic focus |
| **Visage Technologies** | ✅ Active | Custom licensing | ✅ Yes | ✅ FACS-based | ⚠️ Limited | ✅ Yes | ✅ Privacy-focused | Unity integration; mobile SDK |
| **Kairos** | ✅ Active | Usage-based API | ❌ Cloud-only | ⚠️ Limited | ❌ No | ✅ Yes | ⚠️ Cloud | Privacy concerns |
| **Amazon Rekognition** | ✅ Active | Pay-per-use | ❌ Cloud-only | ⚠️ Emotions only | ❌ No | ✅ Yes (video) | ⚠️ AWS | Not FAU-based |
| **Google Cloud Vision** | ✅ Active | Pay-per-use | ❌ Cloud-only | ❌ Detection only | ❌ No | ✅ Yes | ⚠️ Cloud | No emotion/FAU output |
| **Microsoft Azure Face** | ❌ **Deprecated** (2023) | — | — | — | — | — | — | Retired due to responsible AI concerns |
| **RealEyes** | ✅ Active | Enterprise license | ❌ Cloud | ⚠️ Attention only | ❌ No | ✅ Yes | ⚠️ Cloud | Attention measurement focus |

#### Open-Source Model Comparison

| Model | License | Size | Speed | Platforms | Output | On-Device | Maintenance |
|-------|---------|------|-------|-----------|--------|-----------|-------------|
| **MediaPipe Holistic** | Apache 2.0 | ~15MB | 30-60 FPS | TFLite, ONNX, Web | 540+ keypoints (face + body + hands) | ✅ Yes | ✅ Active (Google) |
| **OpenIBUG AU Detection** | MIT | ~100MB | 25-30 FPS | PyTorch, ONNX | 12-18 Action Units | ✅ Yes | ✅ Active (academic) |
| **Aria ET (Gaze)** | MIT | ~50MB | 10 FPS | PyTorch, ONNX | Gaze yaw/pitch + vergence | ✅ Yes | ✅ Active (Meta) |
| **BlazePose GHUM** | Apache 2.0 | ~10MB | 30 FPS | TFLite | 33 body keypoints | ✅ Yes | ✅ Active (Google) |
| **OpenPose** | Apache 2.0 | ~200MB | 5-10 FPS | C++, Python (GPU) | Multi-person pose | ⚠️ GPU only | ⚠️ Slower updates |
| **MoveNet** | Apache 2.0 | ~5MB | 50+ FPS | TFLite | 17 body keypoints | ✅ Yes | ✅ Active (Google) |
| **Wav2Vec2** | MIT | ~300MB | 10 FPS | PyTorch, HF | Audio embeddings | ⚠️ Needs quant | ✅ Active (Meta) |

#### Full Solution Comparison (Including Custom)

| Criterion | Commercial (Affectiva) | Open-Source (MediaPipe + OpenIBUG) | Custom Implementation |
|-----------|------------------------|-------------------------------------|----------------------|
| **Development Cost** | 🟩 Low (SDK integration) | 🟨 Medium (integration + tuning) | 🟥 High (R&D + training) |
| **Licensing Cost** | 🟥 High (per-seat) | 🟩 Free (MIT/Apache) | 🟩 Free |
| **Time to MVP** | 🟩 Fast (weeks) | 🟨 Medium (1-2 months) | 🟥 Slow (6+ months) |
| **On-Device** | 🟩 Yes | 🟩 Yes | 🟩 Yes |
| **FAU Output** | 🟩 21 AUs | 🟩 12-18 AUs | 🟩 Customizable |
| **Body Tracking** | 🟥 No (separate SDK) | 🟩 Yes (MediaPipe) | 🟩 Customizable |
| **Gaze Estimation** | 🟨 Limited | 🟩 Yes (Aria ET) | 🟩 Customizable |
| **Audio Prosody** | 🟥 No | 🟨 Partial (Wav2Vec2) | 🟩 Customizable |
| **Model Size** | 🟨 ~100MB | 🟩 ~50-100MB combined | 🟨 Variable |
| **Fine-Tuning** | 🟥 No (black box) | 🟩 Yes | 🟩 Full control |
| **Bias Control** | 🟥 No control | 🟨 Retrain possible | 🟩 Full control |
| **Privacy** | 🟩 On-device option | 🟩 On-device | 🟩 On-device |
| **Vendor Lock-in** | 🟥 High | 🟩 None | 🟩 None |
| **Long-term Maintenance** | 🟨 Vendor-dependent | 🟨 Community-dependent | 🟥 Self-maintained |

**Legend:** 🟩 Strong | 🟨 Acceptable | 🟥 Weak

#### Recommended Architecture

Based on the comparison, the **hybrid open-source approach** is recommended for SecondSight:

```mermaid
flowchart LR
    subgraph FACE["😐 Face Pipeline"]
        MP_FACE["MediaPipe Face Mesh<br/>(468 landmarks)"]
        IBUG["OpenIBUG AU Detector<br/>(12-18 AUs)"]
        MP_FACE --> IBUG
    end

    subgraph BODY["🧍 Body Pipeline"]
        MP_BODY["MediaPipe Holistic<br/>(33 pose + hands)"]
    end

    subgraph GAZE["👁️ Gaze Pipeline"]
        ARIA["Aria ET / RGB Gaze<br/>(yaw, pitch)"]
    end

    subgraph AUDIO["🎤 Audio Pipeline"]
        LIBROSA["Librosa Features<br/>(pitch, tempo, energy)"]
    end

    FACE --> FUSION["Signal Fusion"]
    BODY --> FUSION
    GAZE --> FUSION
    AUDIO --> FUSION

    style FACE fill:#004d99,stroke:#000,color:#fff
    style BODY fill:#006600,stroke:#000,color:#fff
    style GAZE fill:#660066,stroke:#000,color:#fff
    style AUDIO fill:#996600,stroke:#000,color:#fff
    style FUSION fill:#cc6600,stroke:#000,color:#fff
```

| Component | Model | Size | License | Priority |
|-----------|-------|------|---------|----------|
| **Face Landmarks** | MediaPipe Face Mesh | 15MB | Apache 2.0 | P0 |
| **Facial Action Units** | OpenIBUG AU Detection | 100MB | MIT | P0 |
| **Body Pose** | MediaPipe Holistic | 10MB | Apache 2.0 | P0 |
| **Hand Tracking** | MediaPipe Hands | 5MB | Apache 2.0 | P1 |
| **Gaze Estimation** | Aria ET / RGB Gaze | 50MB | MIT | P0 |
| **Audio Features** | Librosa + MLP | 5MB | BSD | P1 |
| **Total** | — | **~185MB** | — | — |

#### Rationale for Open-Source Approach

| Factor | Decision |
|--------|----------|
| **Cost** | No licensing fees; critical for research/non-profit use cases |
| **Privacy** | Full on-device; no vendor telemetry |
| **Flexibility** | Can fine-tune for demographic fairness |
| **Body Tracking** | Commercial face SDKs don't include body; would need two vendors |
| **Transparency** | Open weights enable bias auditing |
| **Vendor Risk** | Microsoft deprecated Azure Face API; Affectiva was acquired |
| **Signals-Only** | Can ensure FAU output, not emotion labels (harder with black-box SDKs) |

#### When to Consider Commercial SDK

| Scenario | Recommendation |
|----------|----------------|
| Faster time-to-market required | Consider Affectiva for face + MediaPipe for body |
| Enterprise support needed | Visage Technologies offers good support |
| Automotive/embedded focus | Smart Eye (Affectiva) has automotive expertise |
| No ML expertise on team | Commercial SDK reduces ML burden |

#### Model Deployment Strategy

| Phase | Focus | Models |
|-------|-------|--------|
| **Phase 1 (MVP)** | Core signals | MediaPipe Holistic + OpenIBUG AU |
| **Phase 2 (Enhanced)** | Gaze + audio | Add Aria ET + Librosa features |
| **Phase 3 (Production)** | Optimization | Quantization (TFLite, ONNX), model pruning |
| **Phase 4 (Custom)** | Fine-tuning | Retrain on diverse datasets for fairness |

#### Perception Model Open Questions (TBD)

- [ ] Benchmark OpenIBUG AU vs MediaPipe Blendshapes for signal quality
- [ ] Evaluate Aria ET performance on RGB (vs IR eye tracking)
- [ ] Test combined model size and latency on Ray-Ban Meta
- [ ] Assess demographic bias in selected models

</details>

<details id="43-raw-data-storage-mode-gated">
<summary><strong>4.3 Raw Data Storage (Mode-Gated)</strong></summary>

> ⚠️ **Raw data storage availability depends on the operating mode. Each mode has different storage policies based on its purpose.**

Raw data storage is gated by **operating mode**. This reflects the reality that storage requirements differ based on user intent and use case.

#### Storage Policies by Mode

| Mode | Storage Allowed | Consent Required | Data Retention | Use Case |
|------|-----------------|------------------|----------------|----------|
| **Assistive** | ❌ None | — | — | Real-time feedback only; privacy-first |
| **Training** | ⚠️ Session summaries only | Implicit (mode selection) | Session-scoped (auto-delete on exit) | Self-improvement; no raw media |
| **Research** | ✅ Full (opt-in) | Explicit consent dialog | Configurable (default: 30 days) | Model development, evaluation, labeling |

#### Purpose by Mode

**Assistive Mode** — No storage. Ephemeral processing only.
- Real-time perception and feedback
- Privacy-maximizing: nothing persists beyond the current frame
- Default mode for everyday use

**Training Mode** — Session summaries only. No raw media.
- Post-session reflection and self-improvement
- Stores aggregated statistics (e.g., "attention patterns over time")
- No raw camera frames, audio, or sensor data
- Auto-deleted when session ends

**Research Mode** — Full storage with explicit consent.
| Use Case | Description |
|----------|-------------|
| **Model Training** | Collect labeled examples for training/fine-tuning perception models |
| **Offline Evaluation** | Run pipeline on recorded data to measure accuracy, latency, regressions |
| **Debugging** | Reproduce issues by replaying exact sensor inputs |
| **A/B Testing** | Compare model versions on identical input data |

#### What Gets Stored

```mermaid
flowchart LR
    subgraph STORAGE["📦 Raw Data Storage"]
        direction TB
        SENSOR["SensorFrame"]
        PROC["ProcessedFrame"]
        FEAT["Feature Vectors"]
        SIG["Signal Outputs"]
        META["Session Metadata"]
    end

    subgraph PIPELINE["Pipeline"]
        S["Sensors"] --> P["Pre-proc"] --> F["Features"] --> I["Inference"]
    end

    S -.->|tap| SENSOR
    P -.->|tap| PROC
    F -.->|tap| FEAT
    I -.->|tap| SIG

    style STORAGE fill:#993300,stroke:#ff6600,stroke-width:2px,color:#fff
    style SENSOR fill:#663300,stroke:#000,color:#fff
    style PROC fill:#663300,stroke:#000,color:#fff
    style FEAT fill:#663300,stroke:#000,color:#fff
    style SIG fill:#663300,stroke:#000,color:#fff
    style META fill:#663300,stroke:#000,color:#fff
```

| Data Type | Contents | Storage Format |
|-----------|----------|----------------|
| `SensorFrame` | Raw camera frames, IMU readings, audio chunks | Binary (frames as PNG/JPEG, IMU/audio as protobuf) |
| `ProcessedFrame` | Stabilized frames, quality scores | Binary |
| `FaceFeatureVector` | FAUs, gaze proxy, quality | Protobuf/JSON |
| `BodyFeatureVector` | Pose keypoints, gestures, quality | Protobuf/JSON |
| `SignalOutput` | Face/body signals with confidence | Protobuf/JSON |
| `SessionMetadata` | Device info, timestamps, settings, annotations | JSON |

#### Implementation Note: External Tools for Raw Media

> **💡 Implementation Option:** For raw media capture (camera frames, audio), the `DataRecorder` implementation can delegate to external tools like **FFmpeg**, **GStreamer**, or platform-specific recorders. These tools provide efficient encoding, compression, and mature file handling.
>
> The `DataRecorder` class remains the **API layer** responsible for:
> - Mode-gating (blocking Assistive, limiting Training)
> - Semantic data (feature vectors, signals, annotations)
> - Session management (metadata, auto-delete policies)
> - Privacy controls (consent, anonymization)
>
> This hybrid approach combines external tool efficiency with SecondSight's semantic recording requirements.

#### Storage API

**DataRecorder** — Mode-gated class for capturing pipeline data. Enforces storage policies based on operating mode.

##### Sequence Diagram — Recording Session Lifecycle

```mermaid
sequenceDiagram
    participant App as Application
    participant DR as DataRecorder
    participant FS as FileSystem
    participant Pipeline as Perception Pipeline

    Note over App,Pipeline: Session Start
    App->>DR: new DataRecorder(mode, config)
    DR->>DR: validate mode (reject Assistive)
    DR->>DR: validate config against mode
    App->>DR: start_session("session_001")
    DR->>FS: create session directory
    DR->>FS: write metadata.json
    DR-->>App: session started

    Note over App,Pipeline: Recording Loop
    loop Every Frame (~33ms)
        Pipeline->>DR: record_sensor_frame(frame)
        alt Research Mode
            DR->>FS: write raw frame
        else Training Mode
            DR->>DR: extract summary stats only
        end
        Pipeline->>DR: record_features(face, body)
        DR->>FS: write feature vectors
        Pipeline->>DR: record_signals(signals)
        DR->>FS: write signal outputs
    end

    Note over App,Pipeline: Optional: Pause/Resume
    App->>DR: pause()
    DR-->>App: recording paused
    Note right of DR: Frames discarded while paused
    App->>DR: resume()
    DR-->>App: recording resumed

    Note over App,Pipeline: Session End
    App->>DR: end_session()
    DR->>DR: flush buffers
    DR->>FS: finalize metadata
    DR-->>App: SessionSummary
```

##### Sequence Diagram — Export Flow (Research Mode Only)

```mermaid
sequenceDiagram
    participant User as User/App
    participant DR as DataRecorder
    participant Anon as Anonymizer
    participant FS as FileSystem

    User->>DR: export_session("session_001", options)
    DR->>DR: check mode == Research
    alt Not Research Mode
        DR-->>User: ModeNotAllowedError
    else Research Mode
        DR->>FS: read session data
        alt options.anonymize_faces
            DR->>Anon: blur faces in frames
            Anon-->>DR: anonymized frames
        end
        alt options.strip_audio
            DR->>DR: remove audio buffers
        end
        DR->>FS: write export (zip/tar/dir)
        DR-->>User: ExportResult
    end
```

##### State Diagram — DataRecorder States

```mermaid
stateDiagram-v2
    [*] --> Uninitialized

    Uninitialized --> Ready: constructor(mode, config)
    Uninitialized --> Error: mode == Assistive

    Ready --> Recording: start_session()
    Ready --> [*]: destructor

    Recording --> Paused: pause()
    Recording --> Ready: end_session()
    Recording --> Recording: record_*()

    Paused --> Recording: resume()
    Paused --> Ready: end_session()

    Error --> [*]

    note right of Recording
        Active recording
        Frames saved to disk
    end note

    note right of Paused
        Session active but
        frames discarded
    end note
```

##### Activity Diagram — Mode-Based Recording Behavior

```mermaid
flowchart TB
    START((Start)) --> CHECK_MODE{Check Mode}

    CHECK_MODE -->|Assistive| REJECT[❌ Reject: ModeNotAllowedError]
    CHECK_MODE -->|Training| TRAINING_PATH
    CHECK_MODE -->|Research| RESEARCH_PATH

    subgraph TRAINING_PATH["Training Mode"]
        T_SENSOR[record_sensor_frame] --> T_EXTRACT[Extract summary stats only]
        T_EXTRACT --> T_STORE[Store aggregates]
        T_FEATURES[record_features] --> T_STORE_FEAT[Store feature vectors]
        T_SIGNALS[record_signals] --> T_STORE_SIG[Store signals]
        T_ANNOT[add_annotation] --> T_NOOP[No-op: annotations disabled]
    end

    subgraph RESEARCH_PATH["Research Mode"]
        R_SENSOR[record_sensor_frame] --> R_STORE_RAW[Store full raw frame]
        R_FEATURES[record_features] --> R_STORE_FEAT[Store feature vectors]
        R_SIGNALS[record_signals] --> R_STORE_SIG[Store signals]
        R_ANNOT[add_annotation] --> R_STORE_ANNOT[Store annotation]
    end

    TRAINING_PATH --> SESSION_END[end_session]
    RESEARCH_PATH --> SESSION_END

    SESSION_END --> AUTO_DELETE{auto_delete_on_exit?}
    AUTO_DELETE -->|Yes: Training default| DELETE[Delete session data]
    AUTO_DELETE -->|No: Research default| KEEP[Keep session data]

    DELETE --> DONE((Done))
    KEEP --> DONE
    REJECT --> DONE

    style REJECT fill:#cc0000,stroke:#000,color:#fff
    style TRAINING_PATH fill:#996600,stroke:#000,color:#fff
    style RESEARCH_PATH fill:#006600,stroke:#000,color:#fff
    style T_NOOP fill:#666666,stroke:#000,color:#fff
```

##### Usage Example — Research Mode Recording

```mermaid
sequenceDiagram
    participant Lab as Research Lab App
    participant DR as DataRecorder
    participant UI as Consent UI

    Note over Lab,UI: Setup with Consent
    Lab->>UI: show consent dialog
    UI-->>Lab: user consents
    Lab->>DR: new DataRecorder(RESEARCH, config)
    Lab->>DR: start_session("eval_20260111")

    Note over Lab,UI: Controlled Recording
    loop Evaluation Scenario
        Lab->>DR: record_sensor_frame(frame)
        Lab->>DR: record_features(face, body)
        Lab->>DR: record_signals(signals)
        Lab->>DR: add_annotation(ts, "ground_truth_smile")
    end

    Lab->>DR: end_session()
    DR-->>Lab: SessionSummary{frames: 1800, duration: 60s}

    Note over Lab,UI: Later: Export for Analysis
    Lab->>DR: export_session("eval_20260111", {anonymize: true})
    DR-->>Lab: ExportResult{path: "export.zip", size: 450MB}
```

##### Usage Example — Training Mode Self-Reflection

```mermaid
sequenceDiagram
    participant User as User App
    participant DR as DataRecorder
    participant Summary as Session Summary

    Note over User,Summary: Personal Training Session
    User->>DR: new DataRecorder(TRAINING, config)
    Note right of DR: auto_delete_on_exit = true (default)
    User->>DR: start_session("practice_001")

    loop Interaction Practice
        User->>DR: record_features(face, body)
        User->>DR: record_signals(signals)
        Note right of DR: Only aggregates stored, no raw media
    end

    User->>DR: end_session()
    DR-->>User: SessionSummary

    User->>Summary: display reflection
    Summary-->>User: "Avg attention: 72%, Posture shifts: 12"

    Note over User,Summary: Session data auto-deleted
    DR->>DR: delete session (auto)
```

##### Class Diagram

```mermaid
classDiagram
    class DataRecorder {
        -OperatingMode mode
        -RecorderConfig config
        -string sessionId
        -bool active
        -bool paused
        -RecordingStats stats
        +DataRecorder(mode, config)
        +start_session(session_id) void
        +end_session() SessionSummary
        +record_sensor_frame(frame) void
        +record_processed_frame(frame) void
        +record_features(face, body) void
        +record_signals(signals) void
        +add_annotation(timestamp, label, metadata) void
        +pause() void
        +resume() void
        +flush() void
        +export_session(session_id, options) ExportResult
        +delete_session(session_id) void
        +is_active() bool
        +is_paused() bool
        +get_session_id() string
        +get_mode() OperatingMode
        +get_stats() RecordingStats
    }

    class RecorderConfig {
        +bool capture_sensor_frames
        +bool capture_processed_frames
        +bool capture_features
        +bool capture_signals
        +bool capture_annotations
        +int frame_rate_divisor
        +int max_session_duration_s
        +int max_storage_mb
        +bool auto_delete_on_exit
        +validate(mode) bool
        +get_default_for_mode(mode) RecorderConfig
    }

    class RecordingStats {
        +int frames_recorded
        +int64 bytes_written
        +float duration_s
        +int dropped_frames
        +get_storage_used_mb() float
        +get_frame_rate() float
        +reset() void
    }

    class SessionSummary {
        +string session_id
        +int total_frames
        +float duration_s
        +int64 storage_bytes
        +timestamp start_time
        +timestamp end_time
        +get_avg_frame_rate() float
    }

    class ExportOptions {
        +bool anonymize_faces
        +bool strip_audio
        +ExportFormat format
        +bool include_metadata
        +int jpeg_quality
        +bool compress_output
        +validate() bool
    }

    class ExportResult {
        +string export_path
        +int64 size_bytes
        +bool success
        +string error_message
        +int files_exported
        +float duration_s
    }

    class OperatingMode {
        <<enumeration>>
        ASSISTIVE
        TRAINING
        RESEARCH
    }

    class ExportFormat {
        <<enumeration>>
        ZIP
        TAR
        DIRECTORY
    }

    class SensorFrame {
        <<input>>
        +int64 timestamp_ns
        +CameraFrame[] camera_frames
        +DepthFrame depth_frame
        +IMUReading imu_data
        +AudioBuffer audio_chunk
    }

    class ProcessedFrame {
        <<input>>
        +int64 timestamp_ns
        +RGBFrame stabilized_frame
        +float signal_quality
    }

    class FaceFeatureVector {
        <<input>>
        +int64 timestamp_ns
        +FAUVector facial_actions
        +GazeEstimate gaze_proxy
        +float face_quality
    }

    class BodyFeatureVector {
        <<input>>
        +int64 timestamp_ns
        +PoseKeypoints pose
        +HandKeypoints hand_pose
        +HeadPose head_orientation
        +float body_quality
    }

    class SignalOutput {
        <<input>>
        +int64 timestamp_ns
        +SignalModality modality
        +SignalType signal_type
        +float intensity
        +float reliability
    }

    DataRecorder "1" *-- "1" RecorderConfig : configured by
    DataRecorder "1" *-- "1" RecordingStats : tracks
    DataRecorder "1" --> "1" OperatingMode : operates in
    DataRecorder ..> SessionSummary : returns
    DataRecorder ..> ExportResult : returns
    ExportOptions "1" --> "1" ExportFormat : uses
    DataRecorder ..> ExportOptions : accepts

    DataRecorder ..> SensorFrame : records
    DataRecorder ..> ProcessedFrame : records
    DataRecorder ..> FaceFeatureVector : records
    DataRecorder ..> BodyFeatureVector : records
    DataRecorder ..> SignalOutput : records

    style DataRecorder fill:#004d99,stroke:#000,color:#fff
    style RecorderConfig fill:#006600,stroke:#000,color:#fff
    style RecordingStats fill:#663399,stroke:#000,color:#fff
    style SessionSummary fill:#996600,stroke:#000,color:#fff
    style ExportOptions fill:#660066,stroke:#000,color:#fff
    style ExportResult fill:#996600,stroke:#000,color:#fff
    style OperatingMode fill:#cc0000,stroke:#000,color:#fff
    style ExportFormat fill:#666666,stroke:#000,color:#fff
    style SensorFrame fill:#444444,stroke:#000,color:#fff
    style ProcessedFrame fill:#444444,stroke:#000,color:#fff
    style FaceFeatureVector fill:#004d99,stroke:#000,color:#fff
    style BodyFeatureVector fill:#006600,stroke:#000,color:#fff
    style SignalOutput fill:#cc6600,stroke:#000,color:#fff
```

```pseudo
class DataRecorder:
    """
    Mode-gated data recorder for capturing pipeline data.
    Only instantiable in Training or Research modes.
    Assistive mode will raise ModeNotAllowedError.
    """

    # ─────────────────────────────────────────────────────────────
    # LIFECYCLE
    # ─────────────────────────────────────────────────────────────

    constructor(mode: OperatingMode, config: RecorderConfig)
        // Creates a new recorder for the given mode.
        // Raises ModeNotAllowedError if mode is Assistive.
        // Validates config against mode constraints (e.g., Training cannot store raw frames).

    destructor()
        // Auto-flushes pending data and closes file handles.
        // Ensures no data loss on unexpected termination.

    method start_session(session_id: string) -> void
        // Begins a new recording session with the given ID.
        // Creates session directory structure and metadata file.
        // Raises SessionAlreadyActiveError if a session is in progress.

    method end_session() -> SessionSummary
        // Finalizes the current session, flushes all buffers, writes metadata.
        // Returns summary with frame counts, duration, and storage used.
        // Safe to call multiple times (idempotent).

    # ─────────────────────────────────────────────────────────────
    # STATE PROPERTIES
    # ─────────────────────────────────────────────────────────────

    property is_active: bool [readonly]
        // True if a session is in progress and not paused.

    property is_paused: bool [readonly]
        // True if session is active but recording is paused.

    property session_id: string? [readonly]
        // Current session ID, or null if no session active.

    property mode: OperatingMode [readonly]
        // The operating mode this recorder was created with.

    property stats: RecordingStats [readonly]
        // Live statistics: frame count, bytes written, duration.

    # ─────────────────────────────────────────────────────────────
    # RECORDING METHODS (behavior depends on mode)
    # ─────────────────────────────────────────────────────────────

    method record_sensor_frame(frame: SensorFrame) -> void
        // Records raw sensor data (camera, IMU, audio).
        // Training mode: extracts and stores summary statistics only.
        // Research mode: stores full raw frame if config allows.
        // No-op if paused or no active session.

    method record_processed_frame(frame: ProcessedFrame) -> void
        // Records stabilized/pre-processed frame.
        // Training mode: stores quality scores only.
        // Research mode: stores full frame if config allows.

    method record_features(face: FaceFeatureVector?, body: BodyFeatureVector?) -> void
        // Records extracted feature vectors.
        // Both modes: stores feature data (no raw media).

    method record_signals(signals: List<SignalOutput>) -> void
        // Records inference outputs with confidence scores.
        // Both modes: stores signal data.

    method add_annotation(timestamp_ns: int64, label: string, metadata: Map?) -> void
        // Adds ground-truth label for offline evaluation.
        // Used to mark events (e.g., "user smiled", "deception cue").
        // Research mode only; no-op in Training mode.

    # ─────────────────────────────────────────────────────────────
    # CONTROL METHODS
    # ─────────────────────────────────────────────────────────────

    method pause() -> void
        // Temporarily stops recording without ending session.
        // Frames received while paused are discarded.

    method resume() -> void
        // Resumes recording after pause.

    method flush() -> void
        // Forces write of all buffered data to disk.
        // Useful before long operations or checkpoints.

    # ─────────────────────────────────────────────────────────────
    # EXPORT METHODS (Research mode only)
    # ─────────────────────────────────────────────────────────────

    method export_session(session_id: string, options: ExportOptions) -> ExportResult
        // Exports a recorded session to a portable format.
        // Can apply anonymization (blur faces, strip audio).
        // Raises ModeNotAllowedError if not in Research mode.

    method delete_session(session_id: string) -> void
        // Permanently deletes a recorded session.
        // Requires explicit confirmation in config.
```

**RecorderConfig** — Controls what data is captured and storage limits.

| Attribute | Type | Description |
|-----------|------|-------------|
| `capture_sensor_frames` | bool | Store raw camera/IMU/audio (Research only) |
| `capture_processed_frames` | bool | Store stabilized frames |
| `capture_features` | bool | Store feature vectors |
| `capture_signals` | bool | Store inference outputs |
| `capture_annotations` | bool | Allow ground-truth labeling (Research only) |
| `frame_rate_divisor` | int | Store every Nth frame (1 = all, 10 = 10%) |
| `max_session_duration_s` | int | Auto-stop after N seconds |
| `max_storage_mb` | int | Stop if storage exceeds limit |
| `auto_delete_on_exit` | bool | Delete session data when session ends (Training default) |

**RecordingStats** — Live statistics during recording.

| Attribute | Type | Description |
|-----------|------|-------------|
| `frames_recorded` | int | Total frames captured |
| `bytes_written` | int64 | Storage used in bytes |
| `duration_s` | float | Session duration in seconds |
| `dropped_frames` | int | Frames skipped due to rate limiting or pause |

**ExportOptions** — Controls session export behavior.

| Attribute | Type | Description |
|-----------|------|-------------|
| `anonymize_faces` | bool | Blur detected faces in exported frames |
| `strip_audio` | bool | Remove audio from export |
| `format` | ExportFormat | Output format: `zip`, `tar`, `directory` |
| `include_metadata` | bool | Include session metadata in export |

#### File Structure

```text
/data/secondsight/dev/recordings/
├── session_20260110_134500/
│   ├── metadata.json           # Session info, device, config
│   ├── sensor_frames/
│   │   ├── 000001.png          # Raw camera frame
│   │   ├── 000001.imu.pb       # IMU reading
│   │   └── ...
│   ├── processed_frames/
│   │   ├── 000001.png
│   │   └── ...
│   ├── features/
│   │   ├── 000001.face.json
│   │   ├── 000001.body.json
│   │   └── ...
│   ├── signals/
│   │   ├── 000001.json
│   │   └── ...
│   └── annotations.json        # Ground-truth labels
```

#### Privacy Considerations (Even in Dev)

| Concern | Mitigation |
|---------|------------|
| **Bystander capture** | Recording sessions should be in controlled environments only |
| **Data retention** | Auto-delete after configurable period (default: 30 days) |
| **Data sharing** | Recordings stay on-device; explicit export required |
| **Consent** | Dev builds display persistent "RECORDING" indicator |
| **Anonymization** | Export tools can blur faces, strip audio before sharing |

</details>

</details>

---

<details id="appendix-a-signal-vocabulary">
<summary><strong>Appendix A: Signal Vocabulary</strong></summary>

### Allowed Outputs

| Signal Type | Description | Example Cue |
|-------------|-------------|-------------|
| `engagement_shift` | Change in attention indicators | "Attention shifted" |
| `turn_taking_cue` | Pause or floor-yielding signal | "Opening to speak" |
| `tempo_change` | Speech rate variation | "Pace quickening" |
| `posture_shift` | Body orientation change | "Leaning away" |
| `gaze_aversion` | Eye contact pattern change | "Gaze shifted" |
| `expression_intensity` | Facial expressiveness level | "Expression intensified" |
| `emotional_state` | Detected emotion indicator | "Frustration detected (72%)" |
| `deception_indicator` | Behavioral inconsistency cue | "Possible deception (65%)" |
| `intent_signal` | Goal-directed behavior cue | "Hesitation detected" |

### Blocked Vocabulary (Never Output)

| Blocked Category | Examples | Rationale |
|------------------|----------|-----------|
| **Identity** | "Person X", face ID, tracking across sessions | No face recognition |
| **Absolute Claims** | "definitely lying", "100% angry", "is deceptive" | Always probabilistic |
| **Clinical Diagnosis** | "has anxiety disorder", "is depressed", "PTSD" | Not a clinical tool |
| **Character Judgments** | "untrustworthy person", "bad character", "manipulative personality" | Observe behavior, not traits |

> ⚠️ **Note:** All state inferences include confidence levels. The system outputs probabilistic indicators (e.g., "Frustration detected (72%)"), never absolute claims (e.g., "Is angry").

</details>

---

<details id="appendix-b-facial-action-units-faus-reference">
<summary><strong>Appendix B: Facial Action Units (FAUs) Reference</strong></summary>

### Overview

The **Facial Action Coding System (FACS)** is a taxonomy of human facial movements based on muscle activation, developed by Paul Ekman and Wallace Friesen (1978). SecondSight uses a subset of Action Units (AUs) relevant to non-verbal signal detection.

FACS describes **what the face is doing** — the observable movements that serve as intermediate signals for detecting internal states.

### Action Units Used

| AU | Name | Muscle | Observable Signal |
|----|------|--------|-------------------|
| AU1 | Inner Brow Raiser | Frontalis (pars medialis) | Surprise, concern indicators |
| AU2 | Outer Brow Raiser | Frontalis (pars lateralis) | Surprise indicators |
| AU4 | Brow Lowerer | Corrugator supercilii | Concentration, confusion indicators |
| AU5 | Upper Lid Raiser | Levator palpebrae | Attention, alertness |
| AU6 | Cheek Raiser | Orbicularis oculi | Expression intensity |
| AU7 | Lid Tightener | Orbicularis oculi | Engagement, focus |
| AU9 | Nose Wrinkler | Levator labii superioris | Discomfort indicator |
| AU10 | Upper Lip Raiser | Levator labii superioris | Expression intensity |
| AU12 | Lip Corner Puller | Zygomaticus major | Positive expression indicator |
| AU14 | Dimpler | Buccinator | Expression asymmetry |
| AU15 | Lip Corner Depressor | Depressor anguli oris | Expression shift |
| AU17 | Chin Raiser | Mentalis | Expression intensity |
| AU20 | Lip Stretcher | Risorius | Expression shift |
| AU23 | Lip Tightener | Orbicularis oris | Tension indicator |
| AU24 | Lip Pressor | Orbicularis oris | Suppression indicator |
| AU25 | Lips Part | Depressor labii | Speaking readiness |
| AU26 | Jaw Drop | Masseter (relaxed) | Speech, surprise |
| AU45 | Blink | Orbicularis oculi | Attention patterns |

### Intensity Coding

FACS uses a 5-point intensity scale:

| Code | Intensity | Description |
|------|-----------|-------------|
| A | Trace | Barely visible |
| B | Slight | Clearly present |
| C | Marked | Pronounced |
| D | Severe | Strong |
| E | Maximum | Extreme |

SecondSight normalizes these to a 0.0–1.0 float for `intensity` in SignalOutput.

### References

1. Ekman, P., & Friesen, W. V. (1978). *Facial Action Coding System: A Technique for the Measurement of Facial Movement*. Consulting Psychologists Press.
2. Ekman, P., Friesen, W. V., & Hager, J. C. (2002). *Facial Action Coding System: The Manual* (2nd ed.). Research Nexus.
3. Facial Action Coding System — Wikipedia: <https://en.wikipedia.org/wiki/Facial_Action_Coding_System>

</details>

---

<details id="appendix-c-terminology">
<summary><strong>Appendix C: Terminology</strong></summary>

| Term | Definition | Relevance to SecondSight |
|------|------------|--------------------------|
| **Affective Computing** | The study and development of systems that recognize, interpret, and simulate human emotions. | Domain foundation; SecondSight applies real-time, on-device affective inference with uncertainty. |
| **Cue** | A user-facing hint derived from one or more signals. Presented via visual/audio feedback. | Output presented to UserA after passing the Policy Gate; includes salience and uncertainty. |
| **Deception Detection** | Detection of behavioral cues associated with deception. | Provided as probabilistic indicators only; confidence and limitations are surfaced. |
| **Emotion Recognition** | Automated detection of emotional states from facial expressions, voice, and body language. | Core capability with probabilistic outputs and policy gate enforcement to prevent overreach. |
| **FAU** | Facial Action Unit — a FACS-based descriptor of facial muscle movement (see Appendix B). | Intermediate features for face signals; enable explainable, movement-based inference. |
| **FACS** | Facial Action Coding System — a taxonomy of human facial movements based on muscle activation. Developed by Ekman & Friesen (1978). | SecondSight uses FACS-based features (FAUs) as intermediate signals for detecting internal states. See Appendix B. |
| **Non-verbal Communication** | The transmission of messages without words — throxugh facial expressions, gestures, posture, gaze, and vocal tone. | Core domain; multimodal signals drive state inference and assistive cues. |
| **Physiognomy** | The pseudoscientific practice of judging character or personality from facial features (not expressions). Discredited. | Explicitly avoided; SecondSight detects dynamic signals, not static traits. |
| **Policy Gate** | The mandatory filtering layer that blocks forbidden inferences before they reach UserA. | Enforces ethical/legal guardrails (no identity, no absolute claims); hard block in pipeline. |
| **Reliability** | A composite score of model confidence and input data quality. | Propagates through pipeline; used to modulate cue salience and transparency. |
| **Salience** | How prominently a cue is presented; a function of signal strength and reliability. | Drives presentation intensity; user-adjustable and quality-weighted. |
| **Signal** | An observable, measurable non-verbal behavior (e.g., gaze shift, posture change). | Core input units for inference; fused across modalities before policy gating. |
| **UserA** | The person wearing the SecondSight device. Receives feedback cues. | Primary user receiving assistive feedback; controls settings and modes. |
| **UserB** | The interlocutor being observed by UserA. Source of non-verbal signals. | Source of observable signals; never identified; signals are processed with privacy safeguards. |

</details>

---
<details id="appendix-d-bibliography--key-concepts">
<summary><strong>Appendix D: Bibliography</strong></summary>


### Bibliography

1. Ekman, P., & Friesen, W. V. (1978). *Facial Action Coding System: A Technique for the Measurement of Facial Movement*. Consulting Psychologists Press.
2. Ekman, P., Friesen, W. V., & Hager, J. C. (2002). *Facial Action Coding System: The Manual* (2nd ed.). Research Nexus.
3. Picard, R. W. (1997). *Affective Computing*. MIT Press.
4. Todorov, A., Olivola, C. Y., Dotsch, R., & Mende-Siedlecki, P. (2015). Social attributions from faces: Determinants, consequences, accuracy, and functional significance. *Annual Review of Psychology*, 66, 519–545.
5. Bond, C. F., & DePaulo, B. M. (2006). Accuracy of deception judgments. *Personality and Social Psychology Review*, 10(3), 214–234.
6. Crawford, K., & Paglen, T. (2019). Excavating AI: The politics of training sets for machine learning. *AI Now Institute*.
