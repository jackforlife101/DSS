# Building on Apple Silicon with Homebrew (jack/main)

An alternative to "Setup MacOS.txt" that avoids sudo installs into /usr/local
and the Qt online installer (no Qt account needed).

## One-time setup

```sh
sudo xcodebuild -license accept          # Xcode must be installed first
brew install cmake ninja pkgconf libomp qt
git clone https://github.com/microsoft/vcpkg.git ~/vcpkg
~/vcpkg/bootstrap-vcpkg.sh -disableMetrics
```

The chmlib source URL used by vcpkg (jedrea.com) is dead. Pre-seed the
download cache with the archived copy (vcpkg verifies its SHA512):

```sh
curl -L -o ~/vcpkg/downloads/chmlib-0.40.zip \
  "https://web.archive.org/web/2019id_/http://www.jedrea.com/chmlib/chmlib-0.40.zip"
```

Create `CMakeUserPresets.json` (gitignored) in the repo root with `mac-debug`
and `mac-release` presets. They set VCPKG_ROOT=~/vcpkg,
QTDIR=/opt/homebrew/opt/qt, OpenMP_ROOT=/opt/homebrew/opt/libomp, and add
libomp's include directory via CMAKE_{C,CXX}_STANDARD_INCLUDE_DIRECTORIES
(the kernel passes -fopenmp directly, so the OpenMP target's include path
never reaches it).

## Build and run

```sh
cmake --preset mac-debug                 # first run builds vcpkg deps (~5 min)
cmake --build --preset mac-debug
./build/debug/DeepSkyStacker/DeepSkyStacker.app/Contents/MacOS/DeepSkyStacker
```

The Debug build has AddressSanitizer and UBSan enabled; run the executable
from a terminal to see their reports. Use `mac-release` for real processing.

Do not run the install step: it invokes macdeployqt with the upstream
maintainer's code-signing identity.

## Local changes on jack/main
- DSSVersion.h: beta expiry moved to 2099 so local builds keep launching.
- StackingEngine.cpp: explicit discard of std::future<bool>::get() results,
  which newer libc++ marks [[nodiscard]] (and -Werror rejects).
