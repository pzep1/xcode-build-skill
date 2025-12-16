# Xcode CLI Reference

Complete command reference for `xcodebuild` and `xcrun simctl`.

## xcodebuild Commands

### Project Discovery

```bash
# List all schemes in workspace
xcodebuild -workspace /path/to/App.xcworkspace -list

# List all schemes in project
xcodebuild -project /path/to/App.xcodeproj -list

# Show available SDKs
xcodebuild -showsdks

# Show available destinations for a scheme
xcodebuild -workspace /path/to/App.xcworkspace -scheme SchemeName -showDestinations

# Show all build settings
xcodebuild -workspace /path/to/App.xcworkspace -scheme SchemeName -showBuildSettings

# Get specific build setting
xcodebuild -workspace /path/to/App.xcworkspace -scheme SchemeName \
  -showBuildSettings | grep PRODUCT_BUNDLE_IDENTIFIER
```

### Building for iOS Simulator

```bash
# Basic build
xcodebuild \
  -workspace /path/to/App.xcworkspace \
  -scheme SchemeName \
  -destination "platform=iOS Simulator,name=iPhone 16 Pro" \
  build

# Build with specific simulator UUID
xcodebuild \
  -workspace /path/to/App.xcworkspace \
  -scheme SchemeName \
  -destination "platform=iOS Simulator,id=XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX" \
  -configuration Debug \
  build

# Build with custom derived data path (recommended)
xcodebuild \
  -workspace /path/to/App.xcworkspace \
  -scheme SchemeName \
  -destination "platform=iOS Simulator,id=$UDID" \
  -derivedDataPath /tmp/build \
  build

# Clean build
xcodebuild \
  -workspace /path/to/App.xcworkspace \
  -scheme SchemeName \
  -destination "platform=iOS Simulator,id=$UDID" \
  clean build

# Build with specific iOS version
xcodebuild \
  -workspace /path/to/App.xcworkspace \
  -scheme SchemeName \
  -destination "platform=iOS Simulator,name=iPhone 16 Pro,OS=18.0" \
  build
```

### Building for Device

```bash
# Build for generic iOS device (no signing)
xcodebuild \
  -workspace /path/to/App.xcworkspace \
  -scheme SchemeName \
  -destination "generic/platform=iOS" \
  -configuration Release \
  build

# Build for connected device
xcodebuild \
  -workspace /path/to/App.xcworkspace \
  -scheme SchemeName \
  -destination "platform=iOS,id=DEVICE_UDID" \
  build
```

### Building for macOS

```bash
xcodebuild \
  -workspace /path/to/App.xcworkspace \
  -scheme MacScheme \
  -destination "platform=macOS" \
  build
```

### Archives and Distribution

```bash
# Create archive
xcodebuild \
  -workspace /path/to/App.xcworkspace \
  -scheme SchemeName \
  -destination "generic/platform=iOS" \
  -archivePath /tmp/App.xcarchive \
  archive

# Export IPA from archive
xcodebuild \
  -exportArchive \
  -archivePath /tmp/App.xcarchive \
  -exportPath /tmp/export \
  -exportOptionsPlist /path/to/ExportOptions.plist
```

### Testing

```bash
# Run all tests
xcodebuild \
  -workspace /path/to/App.xcworkspace \
  -scheme SchemeName \
  -destination "platform=iOS Simulator,id=$UDID" \
  test

# Run specific test class
xcodebuild \
  -workspace /path/to/App.xcworkspace \
  -scheme SchemeName \
  -destination "platform=iOS Simulator,id=$UDID" \
  -only-testing "AppTests/UserServiceTests" \
  test

# Run specific test method
xcodebuild \
  -workspace /path/to/App.xcworkspace \
  -scheme SchemeName \
  -destination "platform=iOS Simulator,id=$UDID" \
  -only-testing "AppTests/UserServiceTests/testLoginSuccess" \
  test

# Skip specific tests
xcodebuild \
  -workspace /path/to/App.xcworkspace \
  -scheme SchemeName \
  -destination "platform=iOS Simulator,id=$UDID" \
  -skip-testing "AppTests/SlowTests" \
  test

# Test with code coverage
xcodebuild \
  -workspace /path/to/App.xcworkspace \
  -scheme SchemeName \
  -destination "platform=iOS Simulator,id=$UDID" \
  -enableCodeCoverage YES \
  test

# Save test results
xcodebuild \
  -workspace /path/to/App.xcworkspace \
  -scheme SchemeName \
  -destination "platform=iOS Simulator,id=$UDID" \
  -resultBundlePath /tmp/TestResults.xcresult \
  test
```

### Useful Flags

| Flag | Description |
|------|-------------|
| `-workspace <path>` | Path to .xcworkspace |
| `-project <path>` | Path to .xcodeproj |
| `-scheme <name>` | Build scheme |
| `-destination <spec>` | Target device/simulator |
| `-configuration <name>` | Debug or Release |
| `-derivedDataPath <path>` | Where to put build products |
| `-quiet` | Suppress xcodebuild output |
| `-parallelizeTargets` | Build targets in parallel |
| `-jobs <n>` | Number of concurrent build jobs |

---

## xcrun simctl Commands

### Listing Simulators

```bash
# List all simulators (human readable)
xcrun simctl list devices

# List as JSON (better for parsing)
xcrun simctl list devices --json

# List only available simulators
xcrun simctl list devices available

# List simulators for specific OS
xcrun simctl list devices "iOS 18"

# List device types
xcrun simctl list devicetypes

# List runtimes
xcrun simctl list runtimes
```

### Extracting UDIDs with jq

```bash
# Get UDID of specific simulator
xcrun simctl list devices --json | \
  jq -r '.devices | .[].[] | select(.name=="iPhone 16 Pro") | .udid' | head -1

# Get all booted simulators
xcrun simctl list devices --json | \
  jq -r '.devices | .[].[] | select(.state=="Booted") | .udid'

# Get available simulators
xcrun simctl list devices --json | \
  jq -r '.devices | .[].[] | select(.isAvailable==true) | {name, udid}'
```

### Simulator Lifecycle

```bash
# Boot simulator
xcrun simctl boot $UDID

# Shutdown simulator
xcrun simctl shutdown $UDID

# Shutdown all simulators
xcrun simctl shutdown all

# Erase simulator (reset to clean state)
xcrun simctl erase $UDID

# Delete simulator
xcrun simctl delete $UDID

# Create new simulator
xcrun simctl create "My iPhone" \
  "com.apple.CoreSimulator.SimDeviceType.iPhone-16-Pro" \
  "com.apple.CoreSimulator.SimRuntime.iOS-18-0"
```

### App Management

```bash
# Install app
xcrun simctl install $UDID /path/to/App.app

# Uninstall app
xcrun simctl uninstall $UDID com.bundle.identifier

# Launch app
xcrun simctl launch $UDID com.bundle.identifier

# Launch with console output
xcrun simctl launch --console $UDID com.bundle.identifier

# Launch with stdout/stderr redirect
xcrun simctl launch \
  --stdout=/tmp/stdout.log \
  --stderr=/tmp/stderr.log \
  $UDID com.bundle.identifier

# Launch and wait for debugger
xcrun simctl launch -w $UDID com.bundle.identifier

# Terminate app
xcrun simctl terminate $UDID com.bundle.identifier

# List installed apps
xcrun simctl listapps $UDID

# Get app info
xcrun simctl appinfo $UDID com.bundle.identifier

# Get app container path
xcrun simctl get_app_container $UDID com.bundle.identifier
```

### Screenshots and Video

```bash
# Take screenshot
xcrun simctl io $UDID screenshot /tmp/screenshot.png

# Screenshot as JPEG
xcrun simctl io $UDID screenshot --type=jpeg /tmp/screenshot.jpg

# Record video
xcrun simctl io $UDID recordVideo /tmp/recording.mp4

# Record with codec
xcrun simctl io $UDID recordVideo --codec=h264 /tmp/recording.mp4

# Stop recording: Press Ctrl+C in the terminal running recordVideo
```

### Location

```bash
# Set custom location
xcrun simctl location $UDID set 37.7749,-122.4194

# Set location by name
xcrun simctl location $UDID set "San Francisco, CA"

# Reset location
xcrun simctl location $UDID clear
```

### Status Bar Overrides

```bash
# Override time
xcrun simctl status_bar $UDID override --time "9:41"

# Override battery
xcrun simctl status_bar $UDID override --batteryLevel 100 --batteryState charged

# Override network
xcrun simctl status_bar $UDID override --dataNetwork wifi --wifiBars 3

# Clear all overrides
xcrun simctl status_bar $UDID clear
```

### Push Notifications

```bash
# Send push notification
xcrun simctl push $UDID com.bundle.identifier /path/to/payload.json

# Payload example (payload.json):
# {
#   "aps": {
#     "alert": {
#       "title": "Test",
#       "body": "Hello from simctl"
#     }
#   }
# }
```

### Privacy Permissions

```bash
# Grant permission
xcrun simctl privacy $UDID grant photos com.bundle.identifier
xcrun simctl privacy $UDID grant camera com.bundle.identifier
xcrun simctl privacy $UDID grant microphone com.bundle.identifier
xcrun simctl privacy $UDID grant location com.bundle.identifier

# Revoke permission
xcrun simctl privacy $UDID revoke photos com.bundle.identifier

# Reset all permissions
xcrun simctl privacy $UDID reset all com.bundle.identifier
```

### Pasteboard

```bash
# Get pasteboard contents
xcrun simctl pbinfo $UDID

# Copy text to pasteboard
echo "Hello" | xcrun simctl pbcopy $UDID

# Paste from pasteboard
xcrun simctl pbpaste $UDID
```

### URL Handling

```bash
# Open URL in simulator
xcrun simctl openurl $UDID "https://example.com"

# Open deep link
xcrun simctl openurl $UDID "myapp://path/to/screen"
```

### Keychain

```bash
# Add certificate to keychain
xcrun simctl keychain $UDID add-root-cert /path/to/cert.pem

# Add CA certificate
xcrun simctl keychain $UDID add-ca-cert /path/to/ca.pem
```

### Diagnostics

```bash
# Collect diagnostic info
xcrun simctl diagnose

# Verbose logging
xcrun simctl logverbose $UDID enable
# ... reproduce issue ...
xcrun simctl logverbose $UDID disable

# Spawn process in simulator
xcrun simctl spawn $UDID log stream --predicate 'processImagePath CONTAINS "App"'
```

---

## Logging with /usr/bin/log

```bash
# Stream logs for specific app
/usr/bin/log stream \
  --predicate 'processImagePath CONTAINS[cd] "AppName"' \
  --level debug

# Stream with JSON output
/usr/bin/log stream \
  --predicate 'processImagePath CONTAINS[cd] "AppName"' \
  --style json

# Stream with timeout
/usr/bin/log stream \
  --predicate 'processImagePath CONTAINS[cd] "AppName"' \
  --timeout 60s

# Filter by message content
/usr/bin/log stream \
  --predicate 'eventMessage CONTAINS[cd] "error"' \
  --level debug

# Save to file (background)
/usr/bin/log stream \
  --predicate 'processImagePath CONTAINS[cd] "AppName"' \
  --style json > /tmp/logs.json &
LOG_PID=$!

# Stop logging
kill $LOG_PID
```

### Common Predicates

| Predicate | Description |
|-----------|-------------|
| `processImagePath CONTAINS[cd] "App"` | Filter by app name |
| `eventMessage CONTAINS[cd] "error"` | Filter by message |
| `category == "network"` | Filter by category |
| `subsystem == "com.apple.xxx"` | Filter by subsystem |
| `messageType == error` | Only errors |

---

## Finding Built App Path

```bash
# If using -derivedDataPath
find /tmp/build -name "*.app" -type d | head -1

# Default derived data location
find ~/Library/Developer/Xcode/DerivedData -name "*.app" -path "*Debug-iphonesimulator*" | head -1

# Get from build settings
xcodebuild -workspace App.xcworkspace -scheme App -showBuildSettings | grep "BUILT_PRODUCTS_DIR"
```

---

## Complete Workflow Example

```bash
#!/bin/bash
set -e

# Configuration
WORKSPACE="/path/to/App.xcworkspace"
SCHEME="App"
BUNDLE_ID="com.example.app"
DERIVED_DATA="/tmp/build"

# 1. Find simulator
echo "Finding simulator..."
UDID=$(xcrun simctl list devices --json | \
  jq -r '.devices | .[].[] | select(.name=="iPhone 16 Pro" and .isAvailable==true) | .udid' | head -1)

if [ -z "$UDID" ]; then
  echo "Error: No simulator found"
  exit 1
fi
echo "Using simulator: $UDID"

# 2. Boot simulator
echo "Booting simulator..."
xcrun simctl boot "$UDID" 2>/dev/null || true
sleep 3

# 3. Build
echo "Building..."
xcodebuild \
  -workspace "$WORKSPACE" \
  -scheme "$SCHEME" \
  -destination "platform=iOS Simulator,id=$UDID" \
  -derivedDataPath "$DERIVED_DATA" \
  -configuration Debug \
  build

# 4. Find app
APP_PATH=$(find "$DERIVED_DATA" -name "*.app" -type d | head -1)
echo "Found app: $APP_PATH"

# 5. Install
echo "Installing..."
xcrun simctl install "$UDID" "$APP_PATH"

# 6. Launch with logging
echo "Launching..."
xcrun simctl launch --console "$UDID" "$BUNDLE_ID"
```
