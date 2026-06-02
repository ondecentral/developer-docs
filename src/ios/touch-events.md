# Touch Event Tracking

Touch tracking requires **iOS 17+** and **Tier 2 or higher**. Call
`recordTouches()` after a successful `captureDeviceFingerprint`:

```swift
await MetricsCollector.shared.captureDeviceFingerprint(...) { result in
    if case .success = result {
        UIApplication.shared.recordTouches()
    }
}
```

This attaches gesture recognizers to the app's key window and captures the
following event types:

| Event type | Captured fields |
|------------|-----------------|
| `tap` | x, y, pressure, size |
| `fling` (pan/drag) | x, y, pressure, size, velocityX, velocityY |
| `scroll/drag` | x, y, pressure, size, distanceX, distanceY |

### Batching & Persistence

Events are not sent immediately. The SDK accumulates them in memory and
flushes when either condition is met:

- Batch reaches **10 events** (default)
- **10 seconds** have elapsed since the first event in the batch (default)

Before upload, events are written to disk encrypted with AES-256-GCM (see
[Security](./security.md)). Unsent events survive app restarts and are
retried when the network becomes available.

The SDK also flushes automatically on `UIApplication.willResignActive` and
saves remaining events on `UIApplication.willTerminate`.

### Rate Limiting

Touch events are subject to the active `RateLimitPolicy`. Events exceeding
the per-session or per-day cap are silently dropped. See
[Compliance & Privacy](./compliance.md) for configuration.
