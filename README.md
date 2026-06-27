# Text Encryption Tool (文本加密工具)

[![License](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Kotlin](https://img.shields.io/badge/Kotlin-1.9+-purple)](https://kotlinlang.org/)
[![API](https://img.shields.io/badge/API-21%2B-brightgreen)](https://developer.android.com/)
[![Platform](https://img.shields.io/badge/Platform-Android-green)](https://www.android.com/)
[![GitHub last commit](https://img.shields.io/github/last-commit/BI3BJU/Text-Encryption-Tool-Java)]



## Screenshots (截图)

| 主界面 (Main) | 
| :---: |
| ![主界面](./main.jpg) | 

## 中文版

### 简介

一款轻量级 Android 文本加密/解密应用，使用 **AES‑256‑GCM**（伽罗瓦/计数器模式）进行对称加密。支持基于密码短语的密钥派生、随机密钥生成，密文输出支持 **Base64** 或 **十六进制** 格式。实时统计、剪贴板集成以及长按上下文菜单让操作更加便捷。

### 功能特点

- 🔐 **AES‑256‑GCM 加密**，带身份验证（保证完整性和机密性）。
- 🔑 **密码短语 → 密钥**（SHA‑256 派生），也可使用生成的随机密钥。
- 🎲 **一键生成随机密钥**（32 字节，Base64 编码）。
- 📋 **加密/解密/生成密钥后自动复制到剪贴板**。
- 📊 **实时统计**：
  - 明文：UTF‑8 字节数。
  - 密文：Base64 字符长度或十六进制字节数（根据格式切换）。
- 🔄 **输出格式切换**：Base64 或十六进制。
- 🖱️ **长按上下文菜单**（每个输入框）：复制、粘贴、剪切、全选、清空，以及“加密当前明文”/“解密当前密文”/“生成密钥”快捷操作。
- 🛡️ **智能错误提示** – 密钥错误或数据损坏时给出友好提示。

### 快速开始

#### 前提条件
- Android Studio（最新稳定版）
- JDK 11+
- Android SDK（API 21+）

#### 安装
1. 使用命令 `git clone https://github.com/BI3BJU/Text-Encryption-Tool-Java.git` 克隆仓库。
2. 在 Android Studio 中打开项目，并在模拟器或真机上运行。

### 使用说明
1. 在顶部输入框输入**密钥**（密码短语），或点击“生成密钥”创建随机密钥。
2. 在**明文**框输入要加密的文本。
3. 选择输出格式：**Base64** 或 **十六进制**。
4. 点击**加密** → 密文显示在**密文**框中，并自动复制到剪贴板。
5. 解密时，将密文粘贴或输入到**密文**框中，点击**解密** → 明文会恢复并复制到剪贴板。

> **提示**：长按任意输入框可调出快捷菜单（例如直接加密当前明文）。

### 技术栈
- **语言**：Kotlin
- **加密**：Java Cryptography Extension (JCE) – `AES/GCM/NoPadding`
- **密钥派生**：SHA‑256（`MessageDigest`）
- **编码**：`Base64` 和自定义十六进制转换
- **界面**：Android XML 布局，配合 `AppCompat` 和 `WindowInsets` 处理

### 许可证
本项目采用 **MIT License** 许可证。详见 `LICENSE` 文件。

## 🌐 跨平台互通性 (Cross-Platform Interoperability)

本项目提供的 **Java 客户端** 与 **Python 客户端（https://github.com/BI3BJU/Text-Encryption-Tool-Python）** 拥有完全一致且对齐的底层加密架构。
这意味着：**您在 Python 端加密的密文，可以直接在 Android 端解密；反之亦然。**

### 🔐 核心加密对齐标准

两端在进行加解密时，严格遵循以下数学与密码学规范：

1. **密钥派生 (Key Derivation)**：
   双方均采用 **SHA-256** 算法。用户输入的任意长度密码都会先转换为 `UTF-8` 字节流，随后派生成固定 `32 字节 (256-bit)` 的强密钥，用于后续的 AES 加密。
2. **加密算法 (Encryption)**：
   采用国际标准的 **AES-256-GCM** 认证加密模式（无填充 `NoPadding`，认证标签 Tag 长度为 `128-bit / 16字节`）。
3. **随机盐 (Nonce/IV)**：
   每次加密均使用系统级安全随机数生成器（Python 的 `os.urandom` 与 Android 的 `SecureRandom`）生成一个全新的 **12 字节 Nonce**。即使密码和明文相同，每次生成的密文也是完全不同的。

---

### 📦 密文数据结构 (Data Layout)

为了确保两端生成的二进制数据能够顺畅互通，密文在传输/展示前进行了严格的字节拼接。

无论是 Base64 还是 Hex（十六进制）格式，解包后的原始二进制字节流（Raw Bytes）结构如下：

| 字节范围 (Bytes) | 数据类型 | 作用 |
| :--- | :--- | :--- |
| `0 ~ 11` (前 12 字节) | **Nonce (IV)** | 初始化向量，用于防重放与差异化密文 |
| `12 ~ 结尾` (剩余字节) | **Ciphertext + Auth Tag** | 真正的加密密文以及末尾 16 字节的 GCM 认证标签 |



解密时，两端程序都会自动切片读取前 12 字节作为 Nonce，提取剩余字节作为密文与 Tag 进行完整性校验及解密。

---

### 🔄 传输格式支持

两端程序均内置了**密文格式自动识别引擎**。当您复制密文准备解密时，无需手动选择输入格式，工具会自动判断并解析以下两种表现层编码：
* **Base64 字符串**（例如：`aBc1...==`）
* **Hex / 十六进制字符串**（例如：`61626331...`）

免责声明
本应用按“原样”提供，仅供实验和教育用途。作者不对因使用本软件造成的任何误用或损害负责。
---


## English

### Overview

A lightweight Android app for symmetric encryption/decryption of text using **AES-256-GCM** (Galois/Counter Mode). It supports custom passphrase‑based key derivation, random key generation, and outputs ciphertext in **Base64** or **Hex** format. Real‑time statistics, clipboard integration, and a long‑press context menu make it easy to use.

### Features

- 🔐 **AES‑256‑GCM encryption** with authenticated encryption (integrity & confidentiality).
- 🔑 **Passphrase‑to‑key** via SHA‑256 (or use a generated random key).
- 🎲 **One‑click random key generation** (32‑byte, Base64‑encoded).
- 📋 **Automatic copy to clipboard** after encryption/decryption/key generation.
- 📊 **Real‑time statistics**:
  - Plain text: UTF‑8 byte count.
  - Cipher text: length in Base64 characters or bytes (Hex mode).
- 🔄 **Output format switch** between Base64 and Hex.
- 🖱️ **Long‑press context menu** on each input field: Copy, Paste, Cut, Select All, Clear, plus `Encrypt current`/`Decrypt current`/`Generate key` shortcuts.
- 🛡️ **Error‑aware decryption** – shows friendly messages for wrong keys or corrupted data.

### Getting Started

#### Prerequisites
- Android Studio (latest stable)
- JDK 11+
- Android SDK (API 21+)

#### Installation
1. Clone the repo using `git clone https://github.com/BI3BJU/Text-Encryption-Tool-Java.git`.
2. Open the project in Android Studio and run it on an emulator or physical device.

### Usage
1. Enter a key (passphrase) in the top field, or tap `Generate Key` to create a random one.
2. Type your plain text in the **Plain Text** field.
3. Choose output format: **Base64** or **Hex**.
4. Tap **Encrypt** – ciphertext appears in the **Cipher Text** field and is copied to clipboard.
5. To decrypt, paste or type ciphertext into the **Cipher Text** field and tap **Decrypt** – the plain text is restored and copied.

> **Tip**: Long‑press any field to access quick actions (e.g., encrypt the current plain text directly).

### Technical Stack
- **Language**: Kotlin
- **Cryptography**: Java Cryptography Extension (JCE) – `AES/GCM/NoPadding`
- **Key Derivation**: SHA‑256 (via `MessageDigest`)
- **Encoding**: `Base64` and custom Hex converter
- **UI**: Android XML layouts with `AppCompat` and `WindowInsets` handling

### License
Distributed under the **MIT License**. See `LICENSE` for more information.

## 🌐 Cross-Platform Interoperability

The **Java client** provided in this project and the **Python client (https://github.com/BI3BJU/Text-Encryption-Tool-Python)** share a fully consistent and aligned underlying encryption architecture.
This means: **Ciphertext encrypted on the Python side can be directly decrypted on the Android side, and vice versa.**

### 🔐 Core Encryption Standards

Both implementations strictly adhere to the following mathematical and cryptographic specifications during encryption and decryption:

1. **Key Derivation**:
Both sides utilize the **SHA-256** algorithm. User-provided passwords of any length are first converted into a `UTF-8` byte stream and then derived into a fixed-length, strong **32-byte (256-bit)** key for subsequent AES encryption.
2. **Encryption Algorithm**:
Uses the industry-standard **AES-256-GCM** authenticated encryption mode (`NoPadding`; authentication tag length: `128-bit / 16 bytes`).
3. **Random Salt (Nonce/IV)**:
For every encryption operation, a unique **12-byte Nonce** is generated using a system-level cryptographically secure random number generator (Python's `os.urandom` and Android's `SecureRandom`). Even if the password and plaintext remain identical, the resulting ciphertext differs every time.

---

### 📦 Ciphertext Data Structure

To ensure seamless interoperability of binary data between the two platforms, the ciphertext undergoes strict byte concatenation before transmission or display. Regardless of whether the format is Base64 or Hex (hexadecimal), the structure of the unpacked raw binary byte stream is as follows:

| Byte Range | Data Type | Purpose |
| :--- | :--- | :--- |
| `0 ~ 11` (First 12 bytes) | **Nonce (IV)** | Initialization Vector; used to prevent replay attacks and ensure ciphertext uniqueness |
| `12 ~ End` (Remaining bytes) | **Ciphertext + Auth Tag** | The actual encrypted ciphertext followed by the 16-byte GCM authentication tag |

During decryption, the programs on both ends automatically slice the input to read the first 12 bytes as the Nonce, while extracting the remaining bytes as the ciphertext and tag for integrity verification and decryption.

---

### 🔄 Transmission Format Support

Both programs feature a built-in **ciphertext format auto-detection engine**. When you copy the ciphertext for decryption, there is no need to manually select the input format; the tool automatically identifies and parses the following two encoding formats:
* **Base64 string** (e.g., `aBc1...==`)
* **Hex / Hexadecimal string** (e.g., `61626331...`)

Disclaimer
This app is provided as‑is for experimental and educational purposes. The author is not responsible for any misuse or damage caused by this software.
---
