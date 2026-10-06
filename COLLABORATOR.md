# Project Collaboration Guide & Architecture Contract

## 📌 Project Overview
**Offline Voice Authentication System for Windows 11** (Windows Hello style). It allows users to unlock their Windows laptop or PC from the lock screen using a spoken passphrase and biometric speaker verification without internet or a dedicated GPU.

- **GitHub Repository**: [https://github.com/ithyaash3090/Voice-Authentication](https://github.com/ithyaash3090/Voice-Authentication)
- **Target OS**: Windows 11 (64-bit, x86_64, Intel Core & AMD Ryzen)
- **Resource Budget**: CPU-only inference, < 150 MB RAM, < 500 ms latency
- **Tech Nature**: 100% Free & Open-Source (Zero cloud/paid APIs)

---

## 👥 Team Work Division

| Team Member | Assigned Phases | Primary Work Directories |
| :--- | :--- | :--- |
| **Rohit (Member B)** | **Phase 1: Biometric & AI Core Engine**<br>**Phase 2: Windows Background Service & Audio Daemon**<br>**Phase 5: Security Hardening & Anti-Spoofing** | `core_engine/`<br>`service/` |
| **Ithyaash (Member A)** | **Phase 3: Windows Credential Provider (Lock Screen DLL)**<br>**Phase 4: User Voice Enrollment & Settings Desktop GUI** | `credential_provider/`<br>`enrollment_app/` |

> ⚠️ **Collaboration Rule**: Keep all work inside your designated directories to avoid merge conflicts.

---

## 🏗️ System Lifecycle & Restart Behavior

```
[ Power On / Restart Laptop ]
              │
              ▼ (Windows Kernel Boots)
[ 1. Windows Background Service Starts Automatically ]
     - Configured as StartupType: Automatic (LocalSystem)
     - Loads offline ONNX models into RAM (~100 MB) in < 1.5s
     - Initializes WASAPI microphone capture in standby mode
              │
              ▼ (Windows Lock Screen appears)
[ 2. Windows LogonUI loads Custom Credential Provider DLL ]
     - Renders "Voice Recognition" tile on the lock screen
     - User selects tile -> Credential Provider sends CMD:START_LISTEN via Named Pipe
              │
              ▼ (User speaks passphrase)
[ 3. Biometric Pipeline Verifies Voice & Passphrase ]
     - WebRTC VAD isolates voice frames
     - Offline ASR verifies spoken passphrase
     - ECAPA-TDNN computes 192-dim voice embedding vector
     - Cosine similarity checked against user_voiceprint.dat
     - Spectral liveness filter rejects replay attacks
              │
              ▼ (Confidence > Threshold)
[ 4. Service Decrypts Logon Token via DPAPI ]
     - Returns encrypted logon payload to Credential Provider
     - Credential Provider calls Windows LSA with MSV1_0_INTERACTIVE_LOGON
     - Desktop unlocks directly to Home Screen!
```

---

## 🛠️ Open-Source Tech Stack Details

### Rohit's Components (Phases 1, 2, 5):
1. **Biometric & AI Core Engine (`core_engine/`)**:
   - **Speaker Verification**: Pretrained **ECAPA-TDNN** or **3D-Speaker** quantized to INT8 ONNX (~25MB) producing 192-dim voice embeddings.
   - **Passphrase Recognition**: **Sherpa-ONNX** (Zipformer/Whisper-tiny INT8) or **Vosk** offline ASR.
   - **Audio Processing**: 16kHz mono, **WebRTC VAD**, bandpass filter (80 Hz - 7.5 kHz).
   - **Runtime**: **ONNX Runtime (CPU Execution Provider)** with AVX2 acceleration.
2. **Windows Background Service (`service/`)**:
   - **Daemon**: C# (.NET 8 Native AOT) or C++ Win32 Service running under `NT AUTHORITY\LocalSystem`.
   - **Audio in Session 0**: Direct **WASAPI** / **NAudio** pre-logon microphone capture.
   - **IPC Server**: Local **Named Pipe Server** (`\\.\pipe\VoiceAuthPipe`).
   - **Credential Vault**: **Windows DPAPI** (`CryptProtectData`) hardware-backed encryption.
3. **Security Hardening (`core_engine/anti_spoofing.py`)**:
   - **Anti-Replay / Liveness**: High-frequency spectral roll-off and phase jitter filter to distinguish live voice from speaker playback.
   - **Lockout**: 3 failed attempts fallback to Windows PIN/Password.

### Ithyaash's Components (Phases 3, 4):
1. **Windows Credential Provider (`credential_provider/`)**:
   - **Framework**: C++ Win32 COM API (`ICredentialProvider`, `ICredentialProviderCredential2`).
   - **Lock Screen UI**: Voice Recognition tile, microphone animation, status strings.
   - **Logon Submission**: Windows Local Security Authority (`LSA`) with `MSV1_0_INTERACTIVE_LOGON`.
   - **IPC Client**: Named pipe client communicating with `\\.\pipe\VoiceAuthPipe`.
2. **User Enrollment & Settings App (`enrollment_app/`)**:
   - **GUI**: Python (**CustomTkinter**) or C# WPF dark-mode desktop app.
   - **Features**: Microphone test & volume calibration, 3-5 sample voice recording wizard, passphrase selection, sensitivity sliders (Balanced / Strict / Relaxed).

---

## 🔌 Shared Interface Contract (IPC Protocol & File Formats)

### 1. Named Pipe IPC: `\\.\pipe\VoiceAuthPipe`
- **Commands sent to Service**:
  - `CMD:START_LISTEN`
  - `CMD:CANCEL`
  - `CMD:START_ENROLLMENT`
  - `CMD:STORE_CREDENTIAL:<USERNAME>:<BASE64_PASSWORD>`
- **Responses returned by Service**:
  - `STATUS:READY`
  - `STATUS:LISTENING`
  - `STATUS:PROCESSING`
  - `AUTH:SUCCESS:<ENCRYPTED_PAYLOAD>`
  - `AUTH:FAILED:SPEAKER_MISMATCH:<SIMILARITY_SCORE>`
  - `AUTH:FAILED:PHRASE_MISMATCH`
  - `AUTH:FAILED:SPOOF_DETECTED`

### 2. Voiceprint File: `user_voiceprint.dat`
```json
{
  "username": "ithyaash",
  "passphrase": "open my computer",
  "embedding": [0.0421, -0.0182, 0.1294, ...],
  "threshold": 0.75
}
```

---

## 📂 Repository Directory Layout

```
Voice-Authentication/
│
├── .gitignore
├── README.md
├── COLLABORATOR.md             <-- This specification document
│
├── core_engine/                <-- [ROHIT: Phases 1 & 5]
│   ├── requirements.txt
│   ├── audio_capture.py
│   ├── speaker_verifier.py
│   ├── phrase_recognizer.py
│   ├── anti_spoofing.py
│   ├── engine_pipeline.py
│   ├── test_biometrics.py
│   └── models/
│       └── download_models.py
│
├── service/                    <-- [ROHIT: Phase 2]
│   ├── VoiceAuthService.cs
│   ├── AudioSessionCapture.cs
│   ├── NamedPipeServer.cs
│   └── CredentialVault.cs
│
├── credential_provider/        <-- [ITHYAASH: Phase 3]
│   ├── VoiceCredentialProvider.cpp
│   ├── VoiceCredential.cpp
│   ├── NamedPipeClient.cpp
│   ├── dllmain.cpp
│   ├── VoiceCredentialProvider.def
│   └── resources/
│
└── enrollment_app/             <-- [ITHYAASH: Phase 4]
    ├── app.py
    ├── ui_theme.py
    ├── audio_meter.py
    ├── enrollment_wizard.py
    └── settings_panel.py
```

---

## 🔄 Recommended Git Workflow

```bash
# Rohit's branch:
git checkout -b feature/ai-service-rohit

# Ithyaash's branch:
git checkout -b feature/ui-credential-ithyaash

# When pushing:
git add .
git commit -m "Descriptive commit message"
git push origin <branch-name>
```
