# Sine Wave Generator VST3 Plugin

Plugin audio VST3 de synthèse sine wave avec visualizer oscilloscope.

## Fonctionnalités

- **Fréquence :** 1-4000 Hz (slider + display)
- **Amplitude :** 0-1 (control de volume)
- **Waveform :** Sine/Square/Saw/Triangle (switch)
- **Phase :** 0-2π (control de décalage)
- **Visualizer :** Oscilloscope en temps réel

## Stack technique

- **Framework :** JUCE 7.x
- **Build :** CMake
- **Plateformes :** macOS (priorité), Linux, Windows
- **Tests :** CppUnit + JUCE unit testing

## Installation

### Prérequis

**macOS :**
```bash
brew install cmake
# JUCE via Homebrew ou téléchargement
brew install juce
```

**Windows :**
```bash
# Visual Studio 2019/2022
# CMake 3.20+
# JUCE 7.x (télécharger depuis https://github.com/juce/juce)
```

### Build

```bash
mkdir build && cd build
cmake .. -DCMAKE_BUILD_TYPE=Release
cmake --build . --config Release
```

### Installation macOS

```bash
# Le plugin s'install automatiquement dans ~/Library/Audio/Plug-Ins/VST3
cp build/Vst3Sine.plugin/Contents/MacOS/Vst3Sine ~/Library/Audio/Plug-Ins/VST3/
```

## Utilisation

1. Ouvrez un DAW compatible VST3 (Logic Pro, Ableton, Cubase)
2. Ajoutez le plugin "Vst3Sine"
3. Ajustez les contrôles (frequency, amplitude, waveform)
4. Utilisez le visualizer pour suivre l'onde en temps réel

## Tests

```bash
cd build
ctest --output-on-failure
```

## Structure du projet

```
vst3-sine-plugin/
├── CMakeLists.txt          # Build configuration
├── src/
│   ├── SineSynth.cpp       # AudioProcessor principal
│   ├── SineSynth.h
│   ├── SineEditor.cpp      # UI avec oscilloscope
│   └── SineEditor.h
├── tests/
│   └── SineSynthTests.cpp  # Tests unitaires
├── docs/adr/
│   └── ADR-001.md          # Architecture Decision Record
└── README.md
```

## Licence

MIT License

## Contribuer

1. Fork le projet
2. Créer une branche `feature/nom-court`
3. Commit avec Conventional Commits (`feat: ...`, `test: ...`)
4. Push et créer une PR
