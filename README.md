# Windows 11 Offline Voice Authentication System

A local, offline voice biometrics authentication system designed for Windows 11 logon (Windows Hello style). It allows users to unlock their laptop or PC using a spoken passphrase and biometric speaker verification without requiring an internet connection or a dedicated GPU.

---

## 🌟 Key Features

- **100% Offline & Private:** All speech recognition, voiceprint feature extraction, and biometric matching occur entirely on the local machine.
- **Pure CPU Optimized:** Optimized INT8 ONNX models designed to run swiftly on budget Intel (Core i3/Celeron) and AMD (Ryzen 3/Athlon) CPUs (< 150MB RAM, < 500ms latency).
- **Two-Factor Voice Biometrics:**
  1. **Passphrase Verification (What is said):** Real-time offline keyword spotting/ASR.
  2. **Speaker Verification (Who said it):** Deep neural voiceprint embedding (ECAPA-TDNN) with cosine similarity comparison.
  3. **Liveness / Anti-Replay Defense:** Spectral analysis filter to prevent basic replay attacks from phone speakers.
- **Windows Logon Integration:** Windows Credential Provider (V2) COM interface registering a "Voice Recognition" tile directly on the Windows 11 lock screen.

---

## 💻 System Requirements

| Specification | Minimum | Recommended |
| :--- | :--- | :--- |
| **Operating System** | Windows 11 (64-bit, 21H2+) | Windows 11 (22H2 / 23H2+) |
| **Processor (Intel)**| Intel Core i3 (6th Gen+) / Celeron N4100+ | Intel Core i5 / i7 (8th Gen+) |
| **Processor (AMD)**  | AMD Ryzen 3 1200 / Athlon 3000G+ | AMD Ryzen 5 / 7 (3000+ Series) |
| **RAM**              | 4 GB | 8 GB+ |
| **RAM Footprint**    | ~80 MB – 150 MB during active inference | Peak < 200 MB |
| **Disk Space**       | ~250 MB | ~350 MB |
| **GPU**              | **None (Pure CPU Inference)** | N/A |
| **Audio**            | Standard built-in laptop mic or USB mic | Mic array with noise suppression |

---

## 🏗️ Architecture Overview

```
[ Windows 11 Lock Screen (LogonUI) ]
                 │
   (Selects "Voice Login" Tile)
                 ▼
[ Windows Credential Provider DLL (C++) ]
                 │ (Named Pipe IPC)
                 ▼
[ Voice Auth Windows Background Service (LocalSystem) ]
                 │ (WASAPI Capture)
                 ▼
┌─────────────────────────────────────────────────────────┐
│              Offline Local Biometric Pipeline           │
│ 1. Passphrase Match (Sherpa-ONNX / Vosk)                │
│ 2. Speaker Verification (ECAPA-TDNN Embedding Vector)   │
│ 3. Liveness & Anti-Replay Check                         │
└────────────────────────────┬────────────────────────────┘
                             │ (Confidence > Threshold)
                             ▼
[ Decrypt Stored Logon Token via Windows DPAPI ]
                             │
                             ▼
[ LSA / Winlogon Unlocks Desktop ]
```

---

## 🗺️ Project Roadmap & Phases

- [ ] **Phase 1: Biometric & AI Core Engine** (Offline speech recognition + speaker verification in Python/C++ ONNX).
- [ ] **Phase 2: Local Windows Audio Service** (WASAPI background capture + secure DPAPI credential management).
- [ ] **Phase 3: Windows Credential Provider DLL** (C++ COM component for Windows 11 Lock Screen UI).
- [ ] **Phase 4: User Enrollment & Settings Desktop App** (Voice enrollment wizard + threshold calibration).
- [ ] **Phase 5: Security Hardening & Benchmarking** (Spoof resistance + sub-second cold boot unlock).

---

## 📄 License
MIT License
