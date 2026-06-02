# Initialization

Call `captureDeviceFingerprint` early in your app lifecycle. It requests ATT
permission, applies region and compliance policy, and registers the device
with the Lucia backend. The returned `lid` (Lucia ID) uniquely identifies
the device across sessions.

### UIKit (AppDelegate)

```swift
import LuciaMetricsSDK

func application(
    _ application: UIApplication,
    didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
) -> Bool {
    Task { @MainActor in
        await MetricsCollector.shared.captureDeviceFingerprint(
            versionNumber: "1.0.0",
            buildNumber: "100",
            appName: "MyApp",
            userName: "user@example.com", // optional
            environment: .prod,
            tier: .tier1Metrics,
            regionPolicy: .disableInRestrictedRegions
        ) { result in
            switch result {
            case .success(let lid): print("LID: \(lid)")
            case .failure(let error): print("Error: \(error)")
            }
        }
    }
    return true
}
```

### SwiftUI

```swift
import SwiftUI
import LuciaMetricsSDK

@main
struct MyApp: App {
    init() {
        Task { @MainActor in
            await MetricsCollector.shared.captureDeviceFingerprint(
                versionNumber: Bundle.main.infoDictionary?["CFBundleShortVersionString"] as? String ?? "1.0",
                buildNumber: Bundle.main.infoDictionary?["CFBundleVersion"] as? String ?? "1",
                appName: Bundle.main.infoDictionary?["CFBundleName"] as? String ?? "App",
                environment: .prod
            ) { _ in }
        }
    }

    var body: some Scene {
        WindowGroup { ContentView() }
    }
}
```

### Environments

| Case | Backend |
|------|---------|
| `.prod` | `api.clickinsights.xyz` |
| `.staging` | `staging.api.clickinsights.xyz` |
| `.test` | `test.api.clickinsights.xyz` (overridable via `LuciaSDKTestBaseURL` in Info.plist) |
| `.develop(url:)` | Custom URL for local development |
