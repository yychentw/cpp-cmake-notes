# C/C++ 專案開發環境設置

課程中介紹如何在 Windows 主機使用 WSL（Windows Subsystem for Linux）和 Visual Studio Code，達到和在 Linux 主機上差不多的開發 workflow。

以下依照課程介紹的環境設置流程為大綱，並依照目前最新的官方建議進行微調修改，設置流程如下：

## 安裝 Visual Studio 2022

在[此處](https://visualstudio.microsoft.com/zh-hant/downloads/)可免費下載 Visual Studio Community 2022。

執行安裝程式後，進到 Visual Studio Installer，確認是否安裝「使用 C++ 的桌面開發」，以確保其支援核心的 C/C++ 開發。

詳細可參考 [Install C and C++ support in Visual Studio](https://learn.microsoft.com/en-us/cpp/build/vscpp-step-0-installation?view=msvc-170)。

## 安裝 Visual Studio Code（VS Code）

在[此處](https://code.visualstudio.com/download)下載安裝檔，並依照安裝程式中的步驟進行安裝即可～

可先在 VS Code 中安裝 C/C++ extensions。

## 安裝 Windows Subsystem for Linux（WSL）

在安裝 WSL 前，首先要確認系統是 Windows 10 或 11。

過去要安裝 WSL 可能需要一些繁瑣的步驟，不過現在只需要幾個簡單的步驟和一行指令即可完成：
1. 以系統管理員身分執行 Windows PowerShell 或 Windows Command Prompt
2. 執行以下指令：`wsl --install`
3. 待安裝完成後，重新開機
4. 在安裝後首次執行 WSL 時，會需要設定使用者資訊，包含帳號和密碼，之後即可開始使用 WSL
5. 要在 VS Code 中使用 WSL，要在 VS Code 中安裝 WSL extension 或 Remote Development extension pack

詳細可參考 [How to install Linux on Windows with WSL](https://learn.microsoft.com/en-us/windows/wsl/install)。

## 安裝 C/C++ 專案開發相關套件

課程中建議在 WSL 用 apt 安裝相關套件，而在安裝套件前，通常會先更新套件資訊並將已安裝到套件升級到最新版本：
```bash
sudo apt update
sudo apt upgrade
```

### 安裝 C/C++ 編譯工具

```bash
sudo apt install build-essential gdb
```
build-essential 包含 gcc 和 g++。

### 安裝 CMake

```bash
sudo apt install make cmake
```

### 安裝 Git

```bash
sudo apt install git-all
```

### 安裝 Python

建議安裝 Python 3.8 或更新的版本。

```bash
sudo apt-get install python3 python3-pip
```

### 安裝 Doxygen

```bash
sudo apt install doxygen
```

### Optional

```bash
sudo apt install lcov gcovr
sudo apt install ccache
sudo apt install cppcheck
sudo apt install llvm clang-format clang-tidy
sudo apt install curl zip unzip tar
sudo apt install graphviz
```
