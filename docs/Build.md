# Build Instructions

## Prerequisites

### macOS
```bash
brew install cmake
```

### Windows
```bash
# Install Visual Studio 2019/2022 with C++ Desktop development
# Install CMake 3.20+ from https://cmake.org/download/
```

### Linux
```bash
sudo apt install cmake
```

## Build

### Quick Build (macOS)
```bash
mkdir build && cd build
cmake .. -DCMAKE_BUILD_TYPE=Release
cmake --build . --config Release
```

### Build with JUCE Download (all platforms)
```bash
mkdir build && cd build
cmake .. -DCMAKE_BUILD_TYPE=Release -DJUCE_DOWNLOAD=ON
cmake --build . --config Release
```

## Install

### macOS
The plugin will be installed to `~/Library/Audio/Plug-Ins/VST3/`
```bash
cp build/Vst3Sine.plugin/Contents/MacOS/Vst3Sine ~/Library/Audio/Plug-Ins/VST3/
```

### Windows
```bash
copy /Y build\Vst3Sine.dll "C:\Program Files\Vst3Plugins\Vst3Sine.dll"
```

## Tests

```bash
cd build
ctest --output-on-failure
```

## Troubleshooting

### "JUCE not found"
The CMakeLists.txt will automatically download JUCE 7.0.14 from GitHub if not found.
To disable auto-download and install manually:
```bash
brew install juce
# or download from https://github.com/juce/juce/releases
```

### "codesign: No signing certificate"
You need a Developer ID certificate:
```bash
# Generate certificate
security create-cert -k ~/Library/Keychains/login.keychain -n "Developer ID Application: Your Name"
# Sign in Xcode: System Preferences > Security > Code Signing
```

### Plugin not showing in DAW
1. Check plugin is in correct directory (`~/Library/Audio/Plug-Ins/VST3/`)
2. Restart DAW
3. Refresh plugin list in DAW preferences
