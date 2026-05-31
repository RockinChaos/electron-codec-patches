# 🎬 Electron Codec Patches

Electron patches enabling **HEVC**, **DTS**, **AC3**, **E-AC3**, **TrueHD**, and **MLP** codec support for media playback.

---

## 📦 Supported Versions

### ✅ Electron [v42.0.0](https://releases.electronjs.org/release/v42.0.0) → [v42.3.0](https://releases.electronjs.org/release/v42.3.0)

**Patch Version:** v42.0.0  
**Tested Range:** v42.0.0 – v42.3.0

| Platform | Architecture | Build Requirements |
|----------|-------------|-------------------|
| 🪟 **Windows** | x64 | Node 24.15.0 • Python ≥ 3.9 |
| 🐧 **Linux** | x64 | Node 24.15.0 • Python ≥ 3.9 |
| 🍎 **macOS** | x64 | Node 24.15.0 • Xcode 26.3 • MetalToolChain • Python ≥ 3.9 |
| 🍎 **macOS** | arm64 | Node 24.15.0 • Xcode 26.3 • MetalToolChain • Python ≥ 3.9 |

> [!NOTE]
> **macOS:** You may need to launch Xcode settings, navigate to Components, and remove/reinstall MetalToolChain if issues arise.

---

## 🚀 Build Guide

> **Source:** [Official Electron Build Instructions](https://www.electronjs.org/docs/latest/development/build-instructions-gn)

### 0️⃣ Install Prerequisites

Before building Electron, you must install the required system dependencies for your operating system. These dependencies vary by platform and are essential for a successful build.

> [!IMPORTANT]
> Use the **Build Requirements** listed in the [Supported Versions](#-supported-versions) table above. These are the **tested and verified versions** for this patch. While the Electron documentation provides general prerequisites, the specific versions listed above are required for compatibility with these patches.

**📖 Refer to the official Electron documentation for your OS:**
- [Windows Build Prerequisites](https://www.electronjs.org/docs/latest/development/build-instructions-windows)
- [Linux Build Prerequisites](https://www.electronjs.org/docs/latest/development/build-instructions-linux)
- [macOS Build Prerequisites](https://www.electronjs.org/docs/latest/development/build-instructions-macos)

> [!CAUTION]
> Make sure all system dependencies are installed before proceeding with the steps below.

---

### 1️⃣ Install depot_tools

First, clone the Chromium [depot tools](https://commondatastorage.googleapis.com/chrome-infra-docs/flat/depot_tools/docs/html/depot_tools_tutorial.html#_setting_up) repository:
```bash
git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git
```

#### Add depot_tools to PATH

**🐧 Linux & 🍎 macOS:**
```bash
export PATH="`pwd`/depot_tools:$PATH"
```

**🪟 Windows:**

**Command Prompt:**
```cmd
set PATH=%cd%\depot_tools;%PATH%
```

**PowerShell:**
```powershell
$env:Path = "$(Get-Location)\depot_tools;$env:Path"
```

> [!TIP]
> This adds depot_tools to your PATH for the current session. To make this permanent, see the [Optional: Permanent Environment Variables](#%EF%B8%8F-optional-permanent-environment-variables) section.

#### Windows-Specific Setup
Set the environment variable `DEPOT_TOOLS_WIN_TOOLCHAIN=0`:
1. Open **Control Panel** → **System and Security** → **System** → **Advanced system settings**
2. Add a system variable `DEPOT_TOOLS_WIN_TOOLCHAIN` with value `0`

> **Why?** This tells depot_tools to use your locally installed Visual Studio instead of attempting to download a Google-internal version.

---

### 2️⃣ Getting the Code
```bash
mkdir electron && cd electron
gclient config --name "src/electron" --unmanaged https://github.com/electron/electron
gclient sync --with_branch_heads --with_tags
```

> ⏰ **This will take a while—grab a coffee!** ☕

---

### 3️⃣ Update Git Remote Configuration

Configure the repository for `git pull` operations:
```bash
cd src/electron
git remote remove origin
git remote add origin https://github.com/electron/electron
git checkout main
git fetch
git branch --set-upstream-to=origin/main
cd -
```

> **📝 Understanding gclient:** The `gclient` tool checks the `DEPS` file in `src/electron/` for dependencies (Chromium, Node.js, etc.). Running `gclient sync -f` ensures all dependencies match the specifications in that file.

---

### 4️⃣ Pulling Updates

To update to a specific Electron version:
```bash
cd src/electron
git pull
git checkout v42.0.0
# Use -f to force sync, -D to delete files no longer in the repository
gclient sync -f
```

---

### 5️⃣ Apply the Patches

#### File Placement
- Move `chromium-hevc-hd-audio.patch` → `src/`
- Move `electron-hevc-hd-audio.patch` → `src/electron/`
- Move `ffmpeg-hevc-hd-audio.patch` → `src/third_party/ffmpeg/`

#### Apply Each Patch
```bash
# Apply Chromium patch
cd src
git apply chromium-hevc-hd-audio.patch
```
```bash
# Apply Electron patch
cd src/electron
git apply electron-hevc-hd-audio.patch
```
```bash
# Apply FFmpeg patch
cd src/third_party/ffmpeg
git apply ffmpeg-hevc-hd-audio.patch
```

---

### 6️⃣ Generate Release Configuration

> ⏰ **This will take a while!**

#### 🐧 Linux & 🍎 macOS

**Native Build (x64):**
```bash
gn gen out/Release --args="import(\"//electron/build/args/release.gn\")"
```

#### 🍎 macOS Cross-Compilation (x64 → arm64)

**Cross-compile for arm64 (Apple Silicon) from x64 (Intel):**
```bash
gn gen out/Release --args="import(\"//electron/build/args/release.gn\") target_cpu=\"arm64\""
```

> [!NOTE]
> macOS x64 → arm64 cross-compilation has been successfully tested and is recommended.

> [!WARNING]
> Cross-compiling from arm64 → x64 is not well-supported and not recommended.

#### 🪟 Windows

**Command Prompt:**
```cmd
gn gen out/Release --args="import(\"//electron/build/args/release.gn\")"
```

**PowerShell:**
```powershell
gn gen out/Release --args="import(\`"//electron/build/args/release.gn\`")"
```

> [!WARNING]
> **Cross-OS Compilation:** Cross-compiling for Linux or Windows from other operating systems is technically possible but **not tested or recommended**. Building natively on the target OS is strongly preferred for stability and compatibility.

---

### 7️⃣ Build Electron

Compile the release configuration:
```bash
ninja -C out/Release electron
```

> ⏰ **Expect this to take significant time depending on your hardware!**

> [!TIP]
> **Performance:** You can use the `-j` flag to control parallel build jobs and reduce system load. For example, `ninja -j 4 -C out/Release electron` will use only 4 parallel jobs instead of all available CPU cores. This is useful if you need to use your computer for other tasks during the build. Without the `-j` flag, ninja will automatically use all available CPU cores for maximum build speed.

---

### 8️⃣ Package the Build

#### Create Distribution Archive
```bash
ninja -C out/Release electron:electron_dist_zip
```

> 🎉 **Your patched Electron build is now ready!**

---

## 🔄 Updating and Rebuilding

### Reset Previous Changes

Clean your working directories before rebuilding:
```bash
cd src/ && git reset --hard HEAD && git clean -df
cd src/electron/ && git reset --hard HEAD && git clean -df
cd src/third_party/ffmpeg/ && git reset --hard HEAD && git clean -df
```

> [!IMPORTANT]
> Before updating to a new version, ensure you have the correct dependencies installed for that version. Check the **Build Requirements** in the [Supported Versions](#-supported-versions) section for the target Electron version and update Node, Python, Xcode, or other dependencies as needed.

### Update to New Version
```bash
cd src/electron
git pull
# Replace $VERSION with the matching patch version
git checkout v$VERSION
# Use -f to force sync, -D to delete files no longer in the repository
gclient sync -f
```

> After resetting and updating, repeat steps 5-8 to apply patches and rebuild.

---

## ⚙️ Optional: Permanent Environment Variables

Setting environment variables permanently can save time across multiple build sessions. This is **completely optional** but recommended for frequent builders.

### 🐧 Linux

Add to your shell configuration file (`~/.bashrc`, `~/.zshrc`, or `~/.profile`):
```bash
# Add depot_tools to PATH
export PATH="/path/to/depot_tools:$PATH"
```

Then reload your shell:
```bash
source ~/.bashrc  # or ~/.zshrc, ~/.profile
```

---

### 🍎 macOS

Add to your shell configuration file (`~/.zshrc` for Zsh or `~/.bash_profile` for Bash):
```bash
# Add depot_tools to PATH
export PATH="/path/to/depot_tools:$PATH"
```

Then reload your shell:
```bash
source ~/.zshrc  # or ~/.bash_profile
```

---

### 🪟 Windows

#### Using System Environment Variables (Recommended)

1. Open **Control Panel** → **System and Security** → **System** → **Advanced system settings**
2. Click **Environment Variables**
3. Under **System variables** (or **User variables** for current user only):
    - Select **Path** and click **Edit**
    - Click **New** and add `C:\path\to\depot_tools`
4. Click **OK** to save all changes
5. **Restart your terminal/PowerShell** for changes to take effect

#### Using PowerShell Profile (Alternative)

Edit your PowerShell profile:
```powershell
notepad $PROFILE
```

Add these lines:
```powershell
# Add depot_tools to PATH
$env:Path += ";C:\path\to\depot_tools"
```

Save and reload:
```powershell
. $PROFILE
```

> [!NOTE]
> Replace `/path/to/` or `C:\path\to\` with your actual installation paths.

---

## 📄 License

This project is licensed under the GPL-3.0 License. See [LICENSE](https://github.com/RockinChaos/electron-codec-patches/blob/master/LICENSE) for details.

---
⭐️ **Star this repo if you find it useful!** **Made with ❤️ for the Electron community**