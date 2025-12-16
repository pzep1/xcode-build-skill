# xcode-build-skill

A Claude Code plugin that teaches Claude to build and manage iOS/macOS projects using native Xcode CLI tools (`xcodebuild`, `xcrun simctl`) instead of MCP servers.

## Installation

### Option 1: Install from GitHub (Recommended)

```bash
# Add the marketplace
/plugin marketplace add YOUR_USERNAME/xcode-build-skill

# Install the plugin
/plugin install xcode-build-skill@xcode-build-marketplace
```

### Option 2: Install Directly from Git URL

```bash
/plugin install --git https://github.com/YOUR_USERNAME/xcode-build-skill
```

### Option 3: Local Installation (for development)

```bash
# Clone the repo
git clone https://github.com/YOUR_USERNAME/xcode-build-skill.git

# Add as local marketplace
/plugin marketplace add ./xcode-build-skill

# Install
/plugin install xcode-build-skill@xcode-build-marketplace
```

## What It Does

This plugin provides Claude with comprehensive guidance for:

- **Building iOS/macOS apps** with `xcodebuild`
- **Managing simulators** with `xcrun simctl` (boot, install, launch, logs)
- **Taking screenshots** and recording video
- **UI automation** via XCUITest framework (tap, type, gestures, element queries)

## Why Use This?

Instead of relying on external MCP servers like XcodeBuildMCP, this plugin teaches Claude to use Apple's native CLI tools directly.

| Aspect | MCP Approach | This Plugin |
|--------|--------------|-------------|
| Dependencies | External MCP server | None (native tools) |
| Flexibility | Limited to MCP tools | Full CLI capabilities |
| UI Automation | Coordinate-based | Semantic element targeting |
| Learning | Abstracts away details | Teaches actual commands |

## Usage

Once installed, the skill auto-activates when you ask Claude about:

- Building iOS/macOS apps
- Running simulators
- Managing Xcode projects
- UI testing and automation

**Example prompts:**
- "Build the app for iPhone simulator"
- "List available simulators"
- "Take a screenshot of the running app"
- "How do I tap a button in a UI test?"

## Plugin Structure

```
xcode-build-skill/
├── .claude-plugin/
│   ├── plugin.json           # Plugin manifest
│   └── marketplace.json      # Marketplace manifest
├── skills/
│   └── xcode-build/
│       ├── SKILL.md          # Main skill definition
│       ├── CLI_REFERENCE.md  # xcodebuild + simctl reference
│       └── XCUITEST_GUIDE.md # UI automation guide
├── README.md
└── LICENSE
```

## Quick Reference

### Build for Simulator

```bash
# Get simulator UUID
UDID=$(xcrun simctl list devices --json | jq -r '.devices | .[].[] | select(.name=="iPhone 16 Pro") | .udid' | head -1)

# Build
xcodebuild -workspace App.xcworkspace -scheme App \
  -destination "platform=iOS Simulator,id=$UDID" build
```

### Simulator Management

```bash
xcrun simctl list devices          # List simulators
xcrun simctl boot $UDID            # Boot simulator
xcrun simctl install $UDID App.app # Install app
xcrun simctl launch $UDID com.id   # Launch app
```

### Screenshots

```bash
xcrun simctl io $UDID screenshot /tmp/screenshot.png
```

### UI Automation (XCUITest)

```swift
let app = XCUIApplication()
app.launch()

app.textFields["email"].tap()
app.textFields["email"].typeText("user@example.com")
app.buttons["Login"].tap()

XCTAssertTrue(app.staticTexts["Welcome"].exists)
```

## Requirements

- macOS with Xcode installed
- Claude Code CLI
- `jq` (optional, for parsing JSON output)

## License

MIT License - see [LICENSE](LICENSE)

## Contributing

Contributions welcome! Please feel free to submit issues and pull requests.

---

**Note:** Replace `YOUR_USERNAME` with your actual GitHub username after pushing to GitHub.
