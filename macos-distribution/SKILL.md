---
name: macos-distribution
description: macOS app distribution covering code signing (Developer ID, App Store certificates), notarization (notarytool), DMG/pkg creation, App Store submission workflow, sandboxing entitlements, StoreKit for in-app purchases and subscriptions (SubscriptionOfferView, appTransactionID, Transaction.currentEntitlements, subscriptionStatusTask), StoreKit testing with local configuration files, PrivacyInfo.xcprivacy manifest, and dual distribution strategies (App Store + direct). Use when preparing an app for distribution, implementing purchases/subscriptions, configuring signing, or troubleshooting App Store rejection.
---

# macOS Distribution & StoreKit

## Critical Constraints

- ❌ DO NOT distribute without code signing → ✅ Always sign with Developer ID (direct) or Apple Distribution (App Store)
- ❌ DO NOT skip notarization for direct distribution → ✅ Required since macOS 10.15 for Gatekeeper
- ❌ DO NOT use `Transaction.currentEntitlement(for:)` → ✅ Use `Transaction.currentEntitlements(for:)` (plural, returns sequence)
- ❌ DO NOT forget PrivacyInfo.xcprivacy → ✅ Required for App Store submission

## Distribution Decision Tree
```
App Store distribution?
├── YES → Apple Distribution certificate + sandbox + review
│   ├── Basic version (sandbox-safe features)
│   └── Pro features via IAP/subscription
└── Direct distribution?
    ├── Developer ID certificate + notarization
    ├── DMG or pkg installer
    └── Full system access (no sandbox required)

Dual strategy?
├── App Store: Basic/free version, sandbox-safe
└── Direct: Pro version, full Accessibility/CGEvent access
```

## Code Signing & Notarization (Direct)
```bash
# Sign with Developer ID
codesign --deep --force --verify --verbose \
    --sign "Developer ID Application: Your Name (TEAMID)" \
    --options runtime \
    MyApp.app

# Create DMG
hdiutil create -volname "MyApp" -srcfolder MyApp.app \
    -ov -format UDZO MyApp.dmg

# Notarize
xcrun notarytool submit MyApp.dmg \
    --apple-id "you@email.com" \
    --team-id "TEAMID" \
    --password "app-specific-password" \
    --wait

# Staple (attach notarization ticket)
xcrun stapler staple MyApp.dmg
```

## Sandboxing Entitlements (App Store)
```xml
<?xml version="1.0" encoding="UTF-8"?>
<plist version="1.0">
<dict>
    <key>com.apple.security.app-sandbox</key><true/>
    <key>com.apple.security.network.client</key><true/>
    <key>com.apple.security.files.user-selected.read-write</key><true/>
    <key>com.apple.developer.icloud-container-identifiers</key>
    <array><string>iCloud.com.company.app</string></array>
    <key>com.apple.security.application-groups</key>
    <array><string>group.com.company.app</string></array>
</dict>
</plist>
```

## StoreKit — SubscriptionOfferView
```swift
import StoreKit

// Basic subscription offer
SubscriptionOfferView(productID: "com.app.pro.monthly")
    .prefersPromotionalIcon(true)

// With custom icon
SubscriptionOfferView(productID: "com.app.pro.monthly") {
    Image("pro_icon").resizable().frame(width: 40, height: 40)
}

// Show upgrade/downgrade based on customer status
SubscriptionOfferView(groupID: "com.app.subscriptions", visibleRelationship: .upgrade)

// Detail action
SubscriptionOfferView(productID: "com.app.pro.monthly")
    .subscriptionOfferViewDetailAction { isShowingStore = true }
```

## StoreKit — Transaction Handling
```swift
// Check entitlements (new plural API)
for await result in Transaction.currentEntitlements(for: "com.app.pro") {
    if case .verified(let transaction) = result {
        let appTxID = transaction.appTransactionID
        if let offerPeriod = transaction.offerPeriod {
            // Handle offer period
        }
    }
}

// Track subscription status
.subscriptionStatusTask(for: "com.app.subscriptions") { statuses in
    if statuses.contains(where: { $0.state == .subscribed }) {
        customerStatus = .subscribed
    } else {
        customerStatus = .notSubscribed
    }
}

// AppTransaction — unique download identifier
let appTransaction = try await AppTransaction.shared
let transactionID = appTransaction.appTransactionID
let platform = appTransaction.originalPlatform  // .iOS, .macOS, .tvOS, .visionOS
```

## StoreKit Testing in Xcode
1. File → New → File From Template → search "StoreKit Configuration File"
2. Define products in the `.storekit` file
3. Edit scheme → Options → StoreKit Configuration → select your file
4. Use Transaction Manager window to test purchase scenarios

## Launch at Login
```swift
import ServiceManagement

func setLaunchAtLogin(_ enabled: Bool) {
    do {
        if enabled { try SMAppService.mainApp.register() }
        else { try SMAppService.mainApp.unregister() }
    } catch { print("Launch at login error: \(error)") }
}

func isLaunchAtLoginEnabled() -> Bool {
    SMAppService.mainApp.status == .enabled
}
```

## Common Mistakes & Fixes

| Mistake | Fix |
|---------|-----|
| "App is damaged" error | Notarize and staple before distribution |
| App Store rejection for entitlements | Justify each entitlement in review notes |
| `currentEntitlement(for:)` deprecated | Use `currentEntitlements(for:)` (plural) |
| Missing privacy manifest | Add PrivacyInfo.xcprivacy to app target |
| StoreKit products not loading in dev | Set StoreKit Configuration in scheme options |

## References

- [Notarizing macOS software](https://developer.apple.com/documentation/security/notarizing_macos_software_before_distribution)
- [App Sandbox Design Guide](https://developer.apple.com/documentation/security/app_sandbox)
- [StoreKit Documentation](https://developer.apple.com/documentation/storekit)
- [WWDC 2025: What's new in StoreKit](https://developer.apple.com/videos/play/wwdc2025/241/)
