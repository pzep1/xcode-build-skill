# XCUITest UI Automation Guide

This guide covers UI automation using Apple's native XCUITest framework. XCUITest replaces MCP-based tools like `tap`, `type_text`, and `describe_ui` with more powerful, native capabilities.

## Why XCUITest?

| Feature | MCP Tools | XCUITest |
|---------|-----------|----------|
| Element discovery | `describe_ui()` | `app.debugDescription`, element queries |
| Tapping | `tap({x, y})` | `element.tap()`, semantic targeting |
| Typing | `type_text({text})` | `element.typeText("...")` |
| Waiting | None | `waitForExistence(timeout:)` |
| Assertions | None | Full XCTest assertions |
| Gestures | `gesture({preset})` | `swipeUp()`, `pinch()`, custom gestures |

XCUITest targets elements semantically (by accessibility label, identifier, type) rather than coordinates, making tests more reliable.

---

## Setup

### 1. Create UI Test Target

In Xcode:
1. File → New → Target
2. Select "UI Testing Bundle"
3. Name it `AppUITests`
4. Ensure it's added to your scheme's Test action

Or add to `project.pbxproj` manually.

### 2. Basic Test Structure

```swift
import XCTest

final class AppUITests: XCTestCase {

    var app: XCUIApplication!

    override func setUpWithError() throws {
        continueAfterFailure = false
        app = XCUIApplication()
        app.launch()
    }

    override func tearDownWithError() throws {
        app.terminate()
    }

    func testExample() throws {
        // Your test code here
    }
}
```

---

## Finding Elements (replaces `describe_ui`)

### Print Entire UI Hierarchy

```swift
func testPrintHierarchy() {
    print(app.debugDescription)
}
```

This outputs the full accessibility tree - equivalent to `describe_ui()`.

### Query Elements by Type

```swift
// Buttons
let button = app.buttons["Login"]
let allButtons = app.buttons

// Text fields
let emailField = app.textFields["email"]
let passwordField = app.secureTextFields["password"]

// Labels
let welcomeLabel = app.staticTexts["Welcome"]

// Images
let avatar = app.images["profileImage"]

// Switches
let toggle = app.switches["notifications"]

// Cells (in lists)
let cell = app.cells["userCell_123"]

// Navigation bars
let navBar = app.navigationBars["Settings"]

// Tab bars
let tabBar = app.tabBars.firstMatch
let tab = app.tabBars.buttons["Profile"]

// Alerts
let alert = app.alerts["Error"]
let alertButton = app.alerts.buttons["OK"]

// Sheets
let sheet = app.sheets.firstMatch
```

### Query by Accessibility Identifier

Best practice: Set `accessibilityIdentifier` in your SwiftUI/UIKit code:

```swift
// In your app code
Button("Submit") { ... }
    .accessibilityIdentifier("submitButton")

// In your test
let submitButton = app.buttons["submitButton"]
```

### Query by Label Text

```swift
// Exact match
let button = app.buttons["Sign In"]

// Partial match
let button = app.buttons.matching(NSPredicate(format: "label CONTAINS 'Sign'")).firstMatch
```

### Query by Index

```swift
let firstButton = app.buttons.element(boundBy: 0)
let thirdCell = app.cells.element(boundBy: 2)
```

### Check Element Exists

```swift
let button = app.buttons["Login"]
XCTAssertTrue(button.exists)
XCTAssertTrue(button.isHittable)  // visible and tappable
```

---

## Tapping Elements (replaces `tap`)

### Tap by Element

```swift
// Tap button
app.buttons["Login"].tap()

// Tap text field to focus
app.textFields["email"].tap()

// Tap cell in list
app.cells["item_123"].tap()

// Tap tab
app.tabBars.buttons["Profile"].tap()

// Double tap
app.images["zoomableImage"].doubleTap()

// Two-finger tap
app.maps.firstMatch.twoFingerTap()
```

### Tap by Coordinates (like MCP's tap)

```swift
// Tap at specific point
let coordinate = app.coordinate(withNormalizedOffset: CGVector(dx: 0.5, dy: 0.5))
coordinate.tap()

// Tap relative to element
let button = app.buttons["Login"]
let offset = button.coordinate(withNormalizedOffset: CGVector(dx: 0.5, dy: 0.5))
offset.tap()

// Tap at absolute coordinates
let point = app.windows.firstMatch.coordinate(withNormalizedOffset: .zero)
    .withOffset(CGVector(dx: 100, dy: 200))
point.tap()
```

### Long Press

```swift
// Long press for 2 seconds
app.cells["item"].press(forDuration: 2.0)

// Long press then drag
app.cells["item"].press(forDuration: 1.0, thenDragTo: app.cells["target"])
```

---

## Typing Text (replaces `type_text`)

### Type in Text Field

```swift
let emailField = app.textFields["email"]
emailField.tap()
emailField.typeText("user@example.com")

// Clear and type
emailField.tap()
emailField.clearAndEnterText("new@example.com")  // Custom extension needed
```

### Clear Text Field

```swift
extension XCUIElement {
    func clearAndEnterText(_ text: String) {
        guard let stringValue = self.value as? String else {
            self.tap()
            self.typeText(text)
            return
        }

        self.tap()

        // Select all and delete
        let deleteString = String(repeating: XCUIKeyboardKey.delete.rawValue, count: stringValue.count)
        self.typeText(deleteString)
        self.typeText(text)
    }
}
```

### Type in Secure Text Field

```swift
let passwordField = app.secureTextFields["password"]
passwordField.tap()
passwordField.typeText("secretPassword123")
```

### Special Keys

```swift
// Press return/enter
app.textFields["search"].typeText("query\n")

// Or use keyboard key
app.keyboards.buttons["Return"].tap()

// Tab key
app.typeText("\t")
```

---

## Gestures (replaces `gesture`)

### Swipe

```swift
// Swipe directions
app.swipeUp()
app.swipeDown()
app.swipeLeft()
app.swipeRight()

// Swipe on specific element
app.tables.firstMatch.swipeUp()

// Swipe with velocity
app.swipeUp(velocity: .fast)  // .slow, .default, .fast
```

### Scroll

```swift
// Scroll in table/collection
let table = app.tables.firstMatch
table.swipeUp()

// Scroll until element visible
while !app.cells["item_50"].isHittable {
    app.swipeUp()
}
app.cells["item_50"].tap()
```

### Pinch

```swift
// Pinch to zoom
app.maps.firstMatch.pinch(withScale: 2.0, velocity: 1.0)  // zoom in
app.maps.firstMatch.pinch(withScale: 0.5, velocity: -1.0)  // zoom out
```

### Rotate

```swift
app.images["rotatable"].rotate(CGFloat.pi / 4, withVelocity: 1.0)
```

### Drag

```swift
// Drag from one point to another
let start = app.coordinate(withNormalizedOffset: CGVector(dx: 0.5, dy: 0.8))
let end = app.coordinate(withNormalizedOffset: CGVector(dx: 0.5, dy: 0.2))
start.press(forDuration: 0.5, thenDragTo: end)

// Drag element to location
app.cells["draggable"].press(forDuration: 0.5, thenDragTo: app.cells["dropZone"])
```

---

## Waiting for Elements

### Wait for Existence

```swift
let button = app.buttons["Submit"]
let exists = button.waitForExistence(timeout: 5)
XCTAssertTrue(exists)
```

### Wait for Element to Disappear

```swift
let spinner = app.activityIndicators.firstMatch
let expectation = XCTNSPredicateExpectation(
    predicate: NSPredicate(format: "exists == false"),
    object: spinner
)
wait(for: [expectation], timeout: 10)
```

### Wait for Condition

```swift
let predicate = NSPredicate(format: "isHittable == true")
let expectation = XCTNSPredicateExpectation(predicate: predicate, object: app.buttons["Login"])
wait(for: [expectation], timeout: 5)
```

---

## Assertions

```swift
// Element exists
XCTAssertTrue(app.buttons["Login"].exists)

// Element doesn't exist
XCTAssertFalse(app.alerts["Error"].exists)

// Element is hittable (visible and tappable)
XCTAssertTrue(app.buttons["Submit"].isHittable)

// Element is enabled
XCTAssertTrue(app.buttons["Submit"].isEnabled)

// Label text
XCTAssertEqual(app.staticTexts["username"].label, "john_doe")

// Element count
XCTAssertEqual(app.cells.count, 10)

// Value (for text fields, sliders, etc.)
XCTAssertEqual(app.textFields["email"].value as? String, "user@example.com")
```

---

## Screenshots

```swift
func testTakeScreenshot() {
    // Take screenshot
    let screenshot = app.screenshot()

    // Attach to test results
    let attachment = XCTAttachment(screenshot: screenshot)
    attachment.name = "Main Screen"
    attachment.lifetime = .keepAlways
    add(attachment)
}

// Screenshot of specific element
let elementScreenshot = app.buttons["Login"].screenshot()
```

---

## Complete Test Examples

### Login Flow Test

```swift
func testLoginFlow() throws {
    // Navigate to login if needed
    if app.buttons["Sign In"].exists {
        app.buttons["Sign In"].tap()
    }

    // Wait for login screen
    let emailField = app.textFields["email"]
    XCTAssertTrue(emailField.waitForExistence(timeout: 5))

    // Enter credentials
    emailField.tap()
    emailField.typeText("test@example.com")

    let passwordField = app.secureTextFields["password"]
    passwordField.tap()
    passwordField.typeText("password123")

    // Tap login button
    app.buttons["Login"].tap()

    // Verify login success
    let welcomeText = app.staticTexts["Welcome"]
    XCTAssertTrue(welcomeText.waitForExistence(timeout: 10))
}
```

### Story Feed Swipe Test

```swift
func testStorySwipe() throws {
    // Wait for stories to load
    let storyCard = app.otherElements["storyCard"]
    XCTAssertTrue(storyCard.waitForExistence(timeout: 10))

    // Swipe right to like
    storyCard.swipeRight()

    // Verify match popup (if matched)
    if app.alerts["It's a Match!"].waitForExistence(timeout: 2) {
        app.alerts.buttons["Keep Swiping"].tap()
    }

    // Swipe left to pass
    let nextCard = app.otherElements["storyCard"]
    if nextCard.waitForExistence(timeout: 5) {
        nextCard.swipeLeft()
    }
}
```

### Profile Creation Flow

```swift
func testProfileCreation() throws {
    // Step 1: Name
    let nameField = app.textFields["firstName"]
    XCTAssertTrue(nameField.waitForExistence(timeout: 5))
    nameField.tap()
    nameField.typeText("John")
    app.buttons["Continue"].tap()

    // Step 2: Photos
    let addPhotoButton = app.buttons["addPhoto"]
    XCTAssertTrue(addPhotoButton.waitForExistence(timeout: 5))
    addPhotoButton.tap()

    // Handle photo picker
    if app.sheets.firstMatch.waitForExistence(timeout: 2) {
        app.sheets.buttons["Choose from Library"].tap()
    }

    // ... continue flow
}
```

---

## Running UI Tests

### From Command Line

```bash
# Run all UI tests
xcodebuild \
  -workspace /path/to/App.xcworkspace \
  -scheme AppUITests \
  -destination "platform=iOS Simulator,id=$UDID" \
  test

# Run specific test class
xcodebuild \
  -workspace /path/to/App.xcworkspace \
  -scheme AppUITests \
  -destination "platform=iOS Simulator,id=$UDID" \
  -only-testing "AppUITests/LoginTests" \
  test

# Run specific test method
xcodebuild \
  -workspace /path/to/App.xcworkspace \
  -scheme AppUITests \
  -destination "platform=iOS Simulator,id=$UDID" \
  -only-testing "AppUITests/LoginTests/testLoginFlow" \
  test
```

### View Test Results

```bash
# Test results are in the result bundle
open /path/to/TestResults.xcresult

# Or view in Xcode's test navigator
```

---

## Best Practices

### 1. Use Accessibility Identifiers

```swift
// In app code
Button("Submit") { }
    .accessibilityIdentifier("loginSubmitButton")

// In test
app.buttons["loginSubmitButton"].tap()
```

### 2. Create Page Objects

```swift
struct LoginPage {
    let app: XCUIApplication

    var emailField: XCUIElement { app.textFields["email"] }
    var passwordField: XCUIElement { app.secureTextFields["password"] }
    var loginButton: XCUIElement { app.buttons["Login"] }

    func login(email: String, password: String) {
        emailField.tap()
        emailField.typeText(email)
        passwordField.tap()
        passwordField.typeText(password)
        loginButton.tap()
    }
}

// Usage
func testLogin() {
    let loginPage = LoginPage(app: app)
    loginPage.login(email: "test@example.com", password: "password123")
}
```

### 3. Wait Instead of Sleep

```swift
// Bad
sleep(3)
app.buttons["Submit"].tap()

// Good
let button = app.buttons["Submit"]
XCTAssertTrue(button.waitForExistence(timeout: 5))
button.tap()
```

### 4. Handle Alerts and System Dialogs

```swift
// Add UI interruption handler in setUp
addUIInterruptionMonitor(withDescription: "System Alert") { alert in
    if alert.buttons["Allow"].exists {
        alert.buttons["Allow"].tap()
        return true
    }
    return false
}

// Trigger the alert by interacting with the app
app.tap()  // Sometimes needed to trigger the handler
```

---

## Migration from MCP Tools

| MCP Tool | XCUITest Equivalent |
|----------|---------------------|
| `describe_ui()` | `print(app.debugDescription)` |
| `tap({x: 100, y: 200})` | `app.coordinate(withNormalizedOffset:).tap()` |
| `type_text({text: "hello"})` | `element.typeText("hello")` |
| `gesture({preset: "scroll-down"})` | `app.swipeDown()` |
| `screenshot()` | `app.screenshot()` |

The main shift is from coordinate-based interaction to element-based queries, which makes tests more maintainable and reliable.
