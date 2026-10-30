# SecondSight Project Rules

## Project Overview

SecondSight is an XR assistive perception application for Meta Quest, Ray-Ban Meta glasses, and Android. It detects non-verbal signals (gaze, facial actions, posture, prosody) and provides real-time feedback.

**Key Principles:**
- On-device processing only (no cloud)
- Probabilistic outputs with confidence levels
- No identity tracking or face recognition
- Privacy-first architecture

---

## Tech Stack

| Component | Technology |
|-----------|------------|
| Core | C++17 |
| UI (Quest) | Kotlin + Meta Spatial SDK |
| CV | Ocean CV |
| ML | TensorFlow Lite |
| Audio | Oboe |
| Build | Gradle + CMake |

---

## Quick Commands

### Build

```bash
# Debug build
./gradlew assembleDebug

# Release build
./gradlew assembleRelease

# Clean build
./gradlew clean assembleDebug

# Native code only
./gradlew externalNativeBuildDebug
```

### Install & Run (Quest)

```bash
# Install to Quest
./gradlew installDebug

# Run app
adb shell am start -n com.secondsight/.MainActivity

# Stop app
adb shell am force-stop com.secondsight

# Restart app
adb shell am force-stop com.secondsight && adb shell am start -n com.secondsight/.MainActivity
```

### Logs

```bash
# App logs
adb logcat -s SecondSight:V

# Native logs
adb logcat -s secondsight:V

# All logs (filtered)
adb logcat | grep -E "(SecondSight|secondsight)"
```

### Testing

```bash
# Unit tests
./gradlew test

# Specific test
./gradlew test --tests "com.secondsight.core.PipelineTest"

# Instrumented tests (on device)
./gradlew connectedAndroidTest

# Native C++ tests
cd app/src/main/cpp && mkdir -p build && cd build
cmake .. -DBUILD_TESTS=ON && make -j$(nproc) && ctest --output-on-failure
```

### Debugging

```bash
# Connect Quest via ADB
adb devices

# ADB over WiFi
adb tcpip 5555 && adb connect <quest-ip>:5555

# Performance profiling
adb shell dumpsys gfxinfo com.secondsight

# Screenshot
adb shell screencap /sdcard/debug.png && adb pull /sdcard/debug.png
```

### Deployment

```bash
# Build release APK
./gradlew assembleRelease

# Sideload to Quest
adb install -r app/build/outputs/apk/release/app-release.apk
```

---

## Project Structure

```
secondSight/
├── app/                    # Android app (Kotlin + C++)
│   └── src/main/
│       ├── java/           # Kotlin UI + bridge
│       ├── cpp/            # C++ native code
│       └── assets/models/  # TFLite models
├── core/                   # Shared C++ library
├── models/                 # ML model files
└── doc/                    # Documentation
```

---

## Key Interfaces

```cpp
// Sensor abstraction (platform-agnostic)
class ISensorAdapter {
    virtual SensorCapabilities getCapabilities() = 0;
    virtual std::optional<RawCameraFrame> getCameraFrame() = 0;
    virtual std::optional<RawIMUReading> getIMUReading() = 0;
};

// ML runtime abstraction
class IMLRuntime {
    virtual bool loadModel(const std::string& path) = 0;
    virtual InferenceResult run(const std::vector<Tensor>& inputs) = 0;
};
```

---

## C ABI Bridge

```cpp
extern "C" {
    int secondsight_init(const char* config_path);
    void secondsight_shutdown(void);
    int secondsight_poll_cues(SafeCueC* out_cues, int max_cues);
    void secondsight_set_enabled(bool enabled);
    void secondsight_emergency_disable(void);
}
```

---

## Performance Targets

| Metric | Target |
|--------|--------|
| Lightweight cues | < 250 ms |
| Heavy perception | < 500 ms |
| Frame rate | ≥ 25 fps |
| Thermal | Stable 15+ min |

---

## Documentation

- [Design Doc](doc/design.md) - Full system design
- [Quest Implementation](doc/implementation/quest-implementation-plan.md) - Quest build guide
- [Device Compatibility](doc/low-level-design/4.1-device-compatibility.md) - HAL design

---

## Code Style

- C++17 with `-std=c++17`
- Kotlin for Android UI
- Use `std::optional` for nullable returns
- All floats normalized to 0.0–1.0 range
- Confidence/reliability always included in outputs
