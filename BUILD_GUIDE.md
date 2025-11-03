# AeroSpace Build Guide

## Building AeroSpace with Custom Swift Toolchain

This guide covers how to build AeroSpace using a custom Swift toolchain, which is useful when the project requires a specific Swift version different from the system default.

### Prerequisites

- macOS 13.0+ (as required by the project)
- Xcode installed
- Command Line Tools for Xcode

## Step 1: Install Custom Swift Toolchain

1. **Download Swift Package Installer**
   - Visit: https://www.swift.org/install/macos/package_installer/
   - Download the Package Installer for the desired Swift version (e.g., Swift 6.2)

2. **Install Swift Toolchain**
   - Run the downloaded `.pkg` installer
   - Follow the installation prompts
   - The toolchain will be installed to `/Library/Developer/Toolchains/`

3. **Configure Xcode Toolchain**
   - Open Xcode
   - Go to menu bar: **Xcode > Toolchains**
   - Select the newly installed Swift toolchain (e.g., "Swift 6.2")

## Step 2: Build Commands

### Debug Build
```bash
xcrun --toolchain swift swift build -c debug
```

### Release Build Options

#### Option 1: Simple Release (CLI only)
```bash
xcrun --toolchain swift swift build -c release
```

#### Option 2: Universal Binary Release (Recommended for distribution)
```bash
xcrun --toolchain swift swift build -c release --arch arm64 --arch x86_64 --product aerospace
```

#### Option 3: Full Release Build (CLI + GUI App)

**Option 3A: Using the project's release script (modified)**
```bash
# First, modify the release script to use custom toolchain:
# Edit line 26 in build-release.sh:
# Change: swift build -c release --arch arm64 --arch x86_64 --product aerospace -Xswiftc -warnings-as-errors
# To: xcrun --toolchain swift swift build -c release --arch arm64 --arch x86_64 --product aerospace -Xswiftc -warnings-as-errors

# Then run the script:
./build-release.sh --build-version "1.0.0"
```

**Option 3B: Manual full release process**
```bash
# Generate required files
./generate.sh --build-version "1.0.0" --generate-git-hash

# Build CLI with custom toolchain
xcrun --toolchain swift swift build -c release --arch arm64 --arch x86_64 --product aerospace -Xswiftc -warnings-as-errors

# Build GUI app with Xcode (will use selected toolchain in Xcode)
xcodebuild -scheme AeroSpace -configuration Release -derivedDataPath .xcode-build
```

## Step 3: Build Outputs

### Output Locations
- **Debug Build**: `.build/debug/`
- **Release Build**: `.build/release/`
- **Xcode Build**: `.xcode-build/Build/Products/Release/`

### Generated Artifacts
- **CLI Binary**: `aerospace` - Command-line interface tool
- **GUI App**: `AeroSpaceApp` - Main application with GUI
- **Debug Symbols**: `.dSYM` files for debugging

## Step 4: Verification

### Check Toolchain Version
```bash
xcrun --toolchain swift swift --version
```

### Verify Binary Architecture
```bash
# Check if binary exists and get info
file .build/release/aerospace

# For universal binaries, check included architectures
lipo -info .build/release/aerospace
```

### Test CLI Functionality
```bash
# Run the CLI to verify it works
.build/debug/aerospace --help
```

## Project-Specific Details

### Swift Version Requirements
- **Project Requirement**: Swift 6.2.0 (as specified in `.swift-version`)
- **System Default**: May vary (e.g., 6.0.3)
- **Solution**: Use custom toolchain with `xcrun --toolchain swift`

### Build Dependencies
The project automatically builds these dependencies:
- Antlr4 (4.13.1)
- Swift Collections (1.1.4)
- TOMLKit (0.5.5)
- BlueSocket (2.0.4)
- ISSoundAdditions (2.0.1)
- HotKey (0.2.1)

### Build Scripts
- `build-debug.sh` - Simple debug build
- `build-release.sh` - Comprehensive release build with packaging
- `build-docs.sh` - Generate documentation
- `build-shell-completion.sh` - Generate shell completions

## Troubleshooting

### Common Issues

1. **Toolchain not found**
   ```
   xcrun: error: unable to find utility "swift", not a developer tool
   ```
   - Solution: Ensure Xcode Command Line Tools are installed and the custom toolchain is properly installed.

2. **Swift version mismatch**
   ```
   error: package is using Swift tools version X but the installed version is Y
   ```
   - Solution: Use `xcrun --toolchain swift` prefix for all Swift commands.

3. **Build fails with architecture errors**
   - Solution: Ensure you're building for the correct architecture (`arm64` for Apple Silicon, `x86_64` for Intel)

### Clean Build
If you encounter build issues, try a clean build:
```bash
# Clean Swift Package Manager build
rm -rf .build

# Clean Xcode build (if using xcodebuild)
rm -rf .xcode-build

# Rebuild
xcrun --toolchain swift swift build -c debug
```

## Advanced Usage

### Building with Custom Flags
```bash
# Enable warnings as errors (like in release script)
xcrun --toolchain swift swift build -c release -Xswiftc -warnings-as-errors

# Build specific products
xcrun --toolchain swift swift build -c release --product aerospace
xcrun --toolchain swift swift build -c release --product AeroSpaceApp
```

### Cross-Architecture Builds
```bash
# Build for both Apple Silicon and Intel (universal binary)
xcrun --toolchain swift swift build -c release --arch arm64 --arch x86_64
```

## Development Workflow

1. **Initial Setup**: Install custom Swift toolchain and configure Xcode
2. **Development**: Use debug builds for rapid iteration
3. **Testing**: Build and test both CLI and GUI components
4. **Release**: Use release builds with proper versioning and code signing

## Notes

- The project uses Swift 6.2 which may not be the system default
- Always use `xcrun --toolchain swift` prefix to ensure the correct toolchain is used
- The release script (`build-release.sh`) provides comprehensive packaging for distribution
- Universal binaries support both Apple Silicon (arm64) and Intel (x86_64) Macs