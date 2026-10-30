# Meta Quest Implementation Plan

> **Status:** Draft implementation plan. This repository does not yet contain the application or build scripts shown below. SDK setup, sensor access, and dependency versions require validation against public documentation.

---

## Table of Contents

- [1. Overview](#1-overview)
- [2. Environment Setup](#2-environment-setup)
- [3. Project Structure](#3-project-structure)
- [4. Build System](#4-build-system)
- [5. Development Workflow](#5-development-workflow)
- [6. Testing Strategy](#6-testing-strategy)
- [7. Deployment](#7-deployment)
- [8. Implementation Phases](#8-implementation-phases)

---

## 1. Overview

This document outlines the implementation plan for SecondSight on **Meta Quest** devices (Quest 3, Quest Pro). Quest is a candidate development platform; the required camera and tracking access must be verified for each device and public API.

### Target Devices

| Device | Native Tracking | Key Features |
|--------|-----------------|--------------|
| **Quest 3** | Body, Hand | Passthrough cameras, depth sensor, high compute |
| **Quest Pro** | Face, Eye, Body, Hand | Full facial tracking, eye tracking |

### Tech Stack Summary

| Component | Technology |
|-----------|------------|
| **Core Logic** | C++17 |
| **UI Layer** | Kotlin + Meta Spatial SDK |
| **CV Processing** | Ocean CV |
| **ML Inference** | TensorFlow Lite |
| **Audio** | Oboe |
| **Build System** | Gradle + CMake |
| **IDE** | Android Studio |

---

## 2. Environment Setup

### 2.1 Prerequisites

#### System Requirements

- **OS**: macOS 12+ / Ubuntu 22.04+ / Windows 11
- **RAM**: 16GB minimum (32GB recommended)
- **Storage**: 50GB free space
- **GPU**: For shader compilation and debugging

#### Required Accounts

- Meta Developer Account ([developer.meta.com](https://developer.meta.com))
- Quest device registered as Developer Device

### 2.2 Tool Installation

#### Android Studio

```bash
# Download Android Studio (Hedgehog or later)
# https://developer.android.com/studio

# After installation, open SDK Manager and install:
# - Android SDK Platform 33 (Android 13)
# - Android SDK Build-Tools 33.0.0
# - Android NDK 25.2.9519653 (required for C++)
# - CMake 3.22.1+
```

#### Verify Android SDK

```bash
# Add to ~/.zshrc or ~/.bashrc
export ANDROID_HOME=$HOME/Library/Android/sdk
export PATH=$PATH:$ANDROID_HOME/platform-tools
export PATH=$PATH:$ANDROID_HOME/ndk/25.2.9519653

# Verify
adb --version
```

#### Meta Spatial SDK

```bash
# Clone or download Spatial SDK
# https://developer.meta.com/docs/spatial-sdk

# Add to local.properties in project root:
# spatial.sdk.dir=/path/to/spatial-sdk
```

#### Ocean CV (Meta's Computer Vision Library)

```bash
# Build from the public open-source repository:
git clone https://github.com/facebookresearch/ocean.git
cd ocean
mkdir build && cd build
cmake .. -DCMAKE_TOOLCHAIN_FILE=$ANDROID_HOME/ndk/25.2.9519653/build/cmake/android.toolchain.cmake \
         -DANDROID_ABI=arm64-v8a \
         -DANDROID_PLATFORM=android-29
make -j$(nproc)
```

#### TensorFlow Lite

```bash
# Download TFLite AAR for Android
# https://www.tensorflow.org/lite/guide/android

# Add to build.gradle:
# implementation 'org.tensorflow:tensorflow-lite:2.14.0'
# implementation 'org.tensorflow:tensorflow-lite-gpu:2.14.0'
```

#### Oboe Audio Library

```bash
# Add to build.gradle (app level)
# implementation 'com.google.oboe:oboe:1.8.0'
```

### 2.3 Quest Device Setup

```bash
# Enable Developer Mode on Quest device
# 1. Open Meta Quest app on phone
# 2. Go to Devices > Your Quest > Developer Mode
# 3. Enable Developer Mode

# Connect Quest via USB
adb devices  # Should show Quest device

# Enable ADB over WiFi (optional but recommended)
adb tcpip 5555
adb connect <quest-ip>:5555
```

---

## 3. Project Structure

```
secondSight/
├── app/                          # Android application module
│   ├── src/
│   │   ├── main/
│   │   │   ├── java/             # Kotlin/Java code
│   │   │   │   └── com/secondsight/
│   │   │   │       ├── MainActivity.kt
│   │   │   │       ├── ui/       # Spatial SDK UI components
│   │   │   │       ├── bridge/   # JNI bridge classes
│   │   │   │       └── service/  # Background services
│   │   │   ├── cpp/              # C++ native code
│   │   │   │   ├── CMakeLists.txt
│   │   │   │   ├── core/         # Core pipeline
│   │   │   │   ├── hal/          # Hardware abstraction
│   │   │   │   ├── ml/           # ML inference
│   │   │   │   ├── cv/           # Computer vision (Ocean)
│   │   │   │   └── bridge/       # JNI implementation
│   │   │   ├── assets/
│   │   │   │   └── models/       # TFLite models
│   │   │   ├── res/              # Android resources
│   │   │   └── AndroidManifest.xml
│   │   └── test/                 # Unit tests
│   └── build.gradle.kts
├── core/                         # Shared C++ library (portable)
│   ├── src/
│   │   ├── pipeline/             # Pipeline orchestration
│   │   ├── preprocessing/        # Frame preprocessing
│   │   ├── features/             # Feature extraction
│   │   ├── inference/            # Signal inference
│   │   ├── fusion/               # Signal fusion
│   │   ├── policy/               # Policy gate
│   │   └── types/                # Data types
│   ├── include/                  # Public headers
│   └── CMakeLists.txt
├── models/                       # ML model files
│   ├── face_landmark.tflite
│   ├── pose_estimation.tflite
│   └── emotion_classifier.tflite
├── doc/                          # Documentation
├── scripts/                      # Build/deploy scripts
├── build.gradle.kts              # Root build file
├── settings.gradle.kts
├── gradle.properties
└── local.properties              # Local SDK paths (not in git)
```

---

## 4. Build System

### 4.1 Gradle Configuration

**`settings.gradle.kts`**

```kotlin
pluginManagement {
    repositories {
        google()
        mavenCentral()
        gradlePluginPortal()
    }
}

dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        mavenCentral()
    }
}

rootProject.name = "SecondSight"
include(":app")
```

**`build.gradle.kts` (root)**

```kotlin
plugins {
    id("com.android.application") version "8.2.0" apply false
    id("org.jetbrains.kotlin.android") version "1.9.20" apply false
}

task("clean", Delete::class) {
    delete(rootProject.buildDir)
}
```

**`app/build.gradle.kts`**

```kotlin
plugins {
    id("com.android.application")
    id("org.jetbrains.kotlin.android")
}

android {
    namespace = "com.secondsight"
    compileSdk = 33

    defaultConfig {
        applicationId = "com.secondsight"
        minSdk = 29
        targetSdk = 33
        versionCode = 1
        versionName = "1.0.0"

        ndk {
            abiFilters.add("arm64-v8a")
        }

        externalNativeBuild {
            cmake {
                cppFlags += listOf("-std=c++17", "-O3", "-fPIC")
                arguments += listOf(
                    "-DANDROID_STL=c++_shared",
                    "-DANDROID_ARM_NEON=TRUE"
                )
            }
        }
    }

    buildTypes {
        release {
            isMinifyEnabled = false
            proguardFiles(
                getDefaultProguardFile("proguard-android-optimize.txt"),
                "proguard-rules.pro"
            )
        }
        debug {
            isDebuggable = true
            isJniDebuggable = true
        }
    }

    externalNativeBuild {
        cmake {
            path = file("src/main/cpp/CMakeLists.txt")
            version = "3.22.1"
        }
    }

    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_17
        targetCompatibility = JavaVersion.VERSION_17
    }

    kotlinOptions {
        jvmTarget = "17"
    }
}

dependencies {
    // Meta Spatial SDK
    implementation(files("libs/spatial-sdk.aar"))

    // TensorFlow Lite
    implementation("org.tensorflow:tensorflow-lite:2.14.0")
    implementation("org.tensorflow:tensorflow-lite-gpu:2.14.0")
    implementation("org.tensorflow:tensorflow-lite-support:0.4.4")

    // Oboe Audio
    implementation("com.google.oboe:oboe:1.8.0")

    // Kotlin
    implementation("androidx.core:core-ktx:1.12.0")
    implementation("androidx.appcompat:appcompat:1.6.1")
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-android:1.7.3")

    // Testing
    testImplementation("junit:junit:4.13.2")
    androidTestImplementation("androidx.test.ext:junit:1.1.5")
    androidTestImplementation("androidx.test.espresso:espresso-core:3.5.1")
}
```

### 4.2 CMake Configuration

**`app/src/main/cpp/CMakeLists.txt`**

```cmake
cmake_minimum_required(VERSION 3.22.1)
project(secondsight VERSION 1.0.0 LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Find dependencies
find_package(oboe REQUIRED CONFIG)

# Ocean CV (adjust path as needed)
set(OCEAN_DIR "${CMAKE_SOURCE_DIR}/../../../../../third_party/ocean")
add_subdirectory(${OCEAN_DIR} ocean)

# TensorFlow Lite
set(TFLITE_DIR "${CMAKE_SOURCE_DIR}/../../../../../third_party/tflite")

# Core library sources
set(CORE_SOURCES
    core/pipeline_orchestrator.cpp
    core/sensor_normalizer.cpp
    hal/spatial_sensor_adapter.cpp
    cv/preprocessor_ocean.cpp
    ml/tflite_runtime.cpp
    features/face_feature_extractor.cpp
    features/body_feature_extractor.cpp
    inference/face_signal_model.cpp
    inference/body_signal_model.cpp
    fusion/signal_fuser.cpp
    policy/policy_gate.cpp
    bridge/jni_bridge.cpp
)

# Build shared library
add_library(secondsight SHARED ${CORE_SOURCES})

target_include_directories(secondsight PRIVATE
    ${CMAKE_SOURCE_DIR}/include
    ${OCEAN_DIR}/include
    ${TFLITE_DIR}/include
)

target_link_libraries(secondsight
    android
    log
    oboe::oboe
    ocean_cv
    tensorflowlite_jni
)

# Enable NEON optimizations
target_compile_options(secondsight PRIVATE -mfpu=neon)
```

### 4.3 Build Commands

```bash
# Full build (debug)
./gradlew assembleDebug

# Full build (release)
./gradlew assembleRelease

# Clean build
./gradlew clean assembleDebug

# Build only native code
./gradlew externalNativeBuildDebug

# Install to connected Quest
./gradlew installDebug

# Build + Install
./gradlew installDebug

# Generate APK bundle
./gradlew bundleRelease
```

---

## 5. Development Workflow

### 5.1 Daily Development Commands

```bash
# Build and install to Quest
./gradlew installDebug

# Run app on Quest
adb shell am start -n com.secondsight/.MainActivity

# View logs
adb logcat -s SecondSight:V

# View native logs
adb logcat -s secondsight:V

# Stop app
adb shell am force-stop com.secondsight

# Restart app
adb shell am force-stop com.secondsight && \
adb shell am start -n com.secondsight/.MainActivity
```

### 5.2 Debugging

```bash
# Native debugging (requires LLDB)
# In Android Studio: Run > Debug 'app'

# Remote debugging via ADB
adb forward tcp:8600 tcp:8600

# Capture frame for analysis
adb shell screencap /sdcard/debug_frame.png
adb pull /sdcard/debug_frame.png

# Performance profiling
adb shell dumpsys gfxinfo com.secondsight
```

### 5.3 Hot Reload Workflow

```bash
# For Kotlin/Java changes only (not native)
./gradlew installDebug -Pandroid.injected.build.api=30

# For native changes, full rebuild required
./gradlew clean installDebug
```

---

## 6. Testing Strategy

### 6.1 Unit Tests

```bash
# Run all unit tests
./gradlew test

# Run specific test class
./gradlew test --tests "com.secondsight.core.PipelineTest"

# Run with coverage
./gradlew testDebugUnitTestCoverage
```

### 6.2 Native Tests (C++)

```bash
# Build and run native tests
cd app/src/main/cpp
mkdir build && cd build
cmake .. -DBUILD_TESTS=ON
make -j$(nproc)
ctest --output-on-failure
```

### 6.3 Instrumented Tests (on Quest)

```bash
# Run instrumented tests
./gradlew connectedAndroidTest

# Run specific test
./gradlew connectedAndroidTest \
    -Pandroid.testInstrumentationRunnerArguments.class=com.secondsight.SensorIntegrationTest
```

### 6.4 Manual Testing Checklist

| Test | Description | Pass Criteria |
|------|-------------|---------------|
| **Sensor Init** | All sensors initialize | No errors in logcat |
| **Camera Feed** | Passthrough cameras active | Frame rate > 25 fps |
| **Face Tracking** | Face detected (Quest Pro) | FAUs generated |
| **Body Tracking** | Body pose detected | Keypoints visible |
| **Hand Tracking** | Hand gestures recognized | Gesture events fire |
| **Latency** | End-to-end cue latency | < 250 ms |
| **Thermal** | Extended use (15 min) | No throttling |

---

## 7. Deployment

### 7.1 Build Release APK

```bash
# Generate signed release APK
./gradlew assembleRelease

# APK location
ls app/build/outputs/apk/release/
```

### 7.2 Sideload to Quest

```bash
# Install release APK
adb install -r app/build/outputs/apk/release/app-release.apk

# Verify installation
adb shell pm list packages | grep secondsight
```

### 7.3 App Lab Submission

1. Build release APK with signing
2. Create App Lab listing at [developer.meta.com](https://developer.meta.com)
3. Upload APK and complete metadata
4. Submit for review

---

## 8. Implementation Phases

### Phase 1: Foundation (Weeks 1-3)

| Task | Deliverable |
|------|-------------|
| Project scaffolding | Gradle + CMake build working |
| Spatial SDK integration | Basic app runs on Quest |
| JNI bridge skeleton | Kotlin ↔ C++ calls working |
| Sensor adapter (Spatial SDK) | Camera, IMU data flowing |

### Phase 2: Core Pipeline (Weeks 4-7)

| Task | Deliverable |
|------|-------------|
| Ocean CV preprocessing | Frame stabilization working |
| TFLite runtime integration | Model loading, inference |
| Face feature extraction | FAUs extracted |
| Body feature extraction | Pose keypoints extracted |

### Phase 3: Inference & Fusion (Weeks 8-10)

| Task | Deliverable |
|------|-------------|
| Face signal model | Emotion/intent signals |
| Body signal model | Posture signals |
| Signal fusion | Combined FusedSignalOutput |
| Temporal aggregation | Smoothed signals |

### Phase 4: Policy & Output (Weeks 11-13)

| Task | Deliverable |
|------|-------------|
| Policy gate | Blocked signals filtered |
| Visual cue rendering | Spatial SDK UI cues |
| Audio cue rendering | Oboe audio feedback |
| User controls | Settings panel working |

### Phase 5: Polish & Testing (Weeks 14-16)

| Task | Deliverable |
|------|-------------|
| Performance optimization | < 250 ms latency |
| Thermal management | Stable 15+ min use |
| Error handling | Graceful degradation |
| Integration testing | All scenarios pass |

---

## References

- [Meta Spatial SDK Documentation](https://developer.meta.com/docs/spatial-sdk)
- [Meta Quest Developer Hub](https://developer.meta.com/quest)
- [TensorFlow Lite Android Guide](https://www.tensorflow.org/lite/android)
- [Oboe Audio Library](https://github.com/google/oboe)
- [Ocean CV Library](https://github.com/facebookresearch/ocean)
