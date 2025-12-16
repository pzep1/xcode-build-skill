# xcode-build-skill

A Claude Code skill that teaches Claude to build and manage iOS/macOS projects using native Xcode CLI tools (`xcodebuild`, `xcrun simctl`) instead of MCP servers.

## What It Does

This skill provides Claude with comprehensive guidance for:

- **Building iOS/macOS apps** with `xcodebuild`
- **Managing simulators** with `xcrun simctl` (boot, install, launch, logs)
- **Taking screenshots** and recording video
- **UI automation** via XCUITest framework (tap, type, gestures, element queries)

## Why Use This?

Instead of relying on external MCP servers like XcodeBuildMCP, this skill teaches Claude to use Apple's native CLI tools directly. Benefits:

| Aspect | MCP Approach | This Skill |
|--------|--------------|------------|
| Dependencies | External MCP server | None (native tools) |
| Flexibility | Limited to MCP tools | Full CLI capabilities |
| UI Automation | Coordinate-based | Semantic element targeting |
| Learning | Abstracts away details | Teaches actual commands |

## Installation

### Option 1: Clone to User Skills (Recommended)

```bash
# Clone directly to your Claude skills directory
git clone https://github.com/YOUR_USERNAME/xcode-build-skill.git ~/.claude/skills/xcode-build
```

### Option 2: Clone Anywhere + Symlink

```bash
# Clone to your projects
git clone https://github.com/YOUR_USERNAME/xcode-build-skill.git ~/Projects/xcode-build-skill

# Symlink to Claude skills
ln -s ~/Projects/xcode-build-skill ~/.claude/skills/xcode-build
```

### Option 3: Project-Level Skill

```bash
# Add to a specific project
git clone https://github.com/YOUR_USERNAME/xcode-build-skill.git .claude/skills/xcode-build
```

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

## Files

| File | Description |
|------|-------------|
| `SKILL.md` | Main skill definition with frontmatter |
| `CLI_REFERENCE.md` | Complete `xcodebuild` and `xcrun simctl` command reference |
| `XCUITEST_GUIDE.md` | XCUITest UI automation patterns |

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
