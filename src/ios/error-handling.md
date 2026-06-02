# Error Handling

All errors surface through the `Result<String, MetricsError>` completion
passed to `captureDeviceFingerprint`.

```swift
public enum MetricsError: Error {
    case permissionDenied           // User denied ATT tracking
    case networkUnavailable         // No network connection
    case syncFailed(error: Error)   // Backend or serialization failure
    case regionRestricted           // EU/CA with disableInRestrictedRegions policy
    case unknown
}
```

| Error | Cause | Recommended action |
|-------|-------|--------------------|
| `permissionDenied` | User denied ATT or IDFA is unavailable | Continue without tracking, or prompt user to review settings |
| `networkUnavailable` | Device has no internet connection | No action needed — SDK retries automatically when online |
| `syncFailed` | Backend rejected the request or JSON failed to encode | Inspect the wrapped `Error`; SDK persists events locally for retry |
| `regionRestricted` | Device in EU/California with `disableInRestrictedRegions` | Switch to `.clampToTier1InRestrictedRegions`, or show a region-compliant UI |
| `unknown` | Unhandled internal state | File an issue at support@clickinsights.xyz |

### Example

```swift
await MetricsCollector.shared.captureDeviceFingerprint(
    versionNumber: "1.0.0",
    buildNumber: "100",
    appName: "MyApp"
) { result in
    switch result {
    case .success(let lid):
        print("Initialized — LID: \(lid)")

    case .failure(.permissionDenied):
        // Show a message or continue without analytics
        break

    case .failure(.regionRestricted):
        // Reinitialize with .clampToTier1InRestrictedRegions if appropriate
        break

    case .failure(.networkUnavailable):
        // No action; SDK will retry automatically
        break

    case .failure(.syncFailed(let underlying)):
        print("Sync error: \(underlying)")

    case .failure(.unknown):
        print("Unknown error")
    }
}
```
