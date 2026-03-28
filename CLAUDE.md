# GaussianBeam Modernization for Apple Silicon

## Goal
Rebuild GaussianBeam to run natively on modern macOS (Apple Silicon) with contemporary Qt framework.

## Project Context
- **Original**: C++/Qt GUI app for Gaussian laser beam simulations (written ~2007)
- **Current State**: Works on Windows; needs modernization for M1/M2/M3 Macs
- **Build System**: Dual (qmake + CMake); we'll use **qmake** as primary
- **Architecture**: Currently targets i386/ppc; needs arm64 + x86_64 universal binary

## Key Updates Needed

### 1. Qt Framework Upgrade
- **From**: Qt4 (ancient, unsupported)
- **To**: Qt6 (latest) or Qt5 (stable fallback)
- **Files to change**: `GaussianBeam.pro`, `CMakeLists.txt`
- **Dependencies**: `QT += core gui xml xmlpatterns` → add `widgets` for Qt6

### 2. macOS Architecture & Deployment Target
- **Current**: i386, ppc, deployment target 10.4
- **Target**: arm64 (Apple Silicon) + x86_64 (Intel), deployment target 11.0+
- **Qmake changes**:
  - Remove `macx:CONFIG += x86 ppc`
  - Add: `macx:QMAKE_APPLE_DEVICE_SPECIFIER = generic/macos` or let Qt handle it
- **CMake changes**:
  - Update `CMAKE_OSX_ARCHITECTURES` to `arm64;x86_64`
  - Update `CMAKE_OSX_DEPLOYMENT_TARGET` to `11.0` (minimum for arm64)

### 3. Platform Dependencies
- Remove hardcoded `/usr/bin`, `/usr/share` paths (these may vary on modern macOS)
- Consider making install paths configurable

## Environment Setup

### Prerequisites to Install
```bash
# 1. Xcode Command Line Tools (includes compiler, cmake)
xcode-select --install

# 2. Homebrew (if not installed)
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# 3. Qt6 via Homebrew
brew install qt@6

# 4. (Optional) CMake via Homebrew (Xcode includes older version)
brew install cmake
```

### Verify Installation
```bash
# Check Qt6 location
brew --prefix qt@6
# Typical output: /usr/local/opt/qt@6

# Check compiler
clang --version
cmake --version
```

## Build & Test Strategy

### Phase 1: Update Build Files
1. Update `GaussianBeam.pro` with Qt6 + modern macOS settings
2. Update `CMakeLists.txt` similarly
3. Commit changes (incremental)

### Phase 2: Initial Build (qmake)
```bash
# Set Qt6 in path
export PATH="$(brew --prefix qt@6)/bin:$PATH"

# Run qmake (creates Makefile)
qmake GaussianBeam.pro

# Compile
make -j$(nproc)

# Run
./gaussianbeam
```

### Phase 3: Create App Bundle
- qmake auto-generates `.app` bundle on macOS (if configured correctly)
- Alternatively use `macdeployqt` tool to bundle dependencies

### Phase 4: Test & Package
- Verify runs on native Apple Silicon
- Test on Intel Mac if available (universal binary)
- (Optional) Code sign & notarize for distribution

## Common Issues & Solutions

**Issue**: `qt: command not found`
- **Fix**: `export PATH="$(brew --prefix qt@6)/bin:$PATH"`

**Issue**: `moc: command not found`
- **Fix**: Same as above; ensure Qt6 bin is in PATH

**Issue**: Compilation errors about Qt4-specific headers
- **Fix**: Update includes; Qt6 removed some modules (replaced with modern equivalents)

**Issue**: App won't run on Apple Silicon
- **Fix**: Verify CMake/qmake built for arm64, not just x86_64

## References
- Qt Migration guide: https://doc.qt.io/qt-6/porting-to-qt-6.html
- Qt on macOS: https://doc.qt.io/qt-6/macos.html
- Homebrew Qt: `brew info qt@6`

## Success Criteria
- [ ] GaussianBeam.pro updated for Qt6
- [ ] CMakeLists.txt updated for Qt6 + arm64/x86_64
- [ ] Builds successfully with qmake
- [ ] App runs natively on Apple Silicon
- [ ] (Optional) Universal binary runs on Intel Macs too
- [ ] (Optional) App is codesigned & notarized
