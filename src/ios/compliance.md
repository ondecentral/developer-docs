# Compliance & Privacy

The SDK enforces GDPR Art. 5(1)(c) data minimization through a tiered
collection model, configurable region gating, per-field overrides, on-device
consent logging, and automatic data retention purging.

### Data Collection Tiers

Pass `tier:` to `captureDeviceFingerprint` to control what is collected.

| Tier | What is collected |
|------|------------------|
| `.tier1Metrics` (default) | Device ID (IDFA), IDFV, device model, OS version |
| `.tier2MetricsAndTouch` | Tier 1 + touch event telemetry |
| `.tier3Full` | Tier 2 + IP address + full device attribute payload |

Tier 3 requires a signed Data Processing Agreement (DPA) and explicit user
consent before use in production.

### Region Policy

Pass `regionPolicy:` to gate collection based on the user's detected jurisdiction.
Detection is entirely on-device (Locale + TimeZone) — no network call is made
before consent.

| Policy | Behavior in EU / California |
|--------|-----------------------------|
| `.disableInRestrictedRegions` (default) | SDK disabled; `regionRestricted` error returned |
| `.clampToTier1InRestrictedRegions` | Forced to Tier 1 regardless of requested tier |
| `.allowAll` | No region gating — requires your own DPA/DPIA compliance |

### Per-Field Overrides

Exclude specific fields within a tier using `DataMinimizationOverrides`. Useful
when a client DPA or legal review prohibits specific attributes.

```swift
let overrides = DataMinimizationOverrides(excludedFields: [
    DataMinimizationOverrides.Field.ipAddress,
    DataMinimizationOverrides.Field.timezone
])

await MetricsCollector.shared.captureDeviceFingerprint(
    ...,
    overrides: overrides
) { _ in }
```

Available field keys:

| Key | Tier required |
|-----|--------------|
| `ip_address` | Tier 3 |
| `cpu_cores` | Tier 3 |
| `memory` | Tier 3 |
| `device_pixel_ratio` | Tier 3 |
| `color_depth` | Tier 3 |
| `color_gamut` | Tier 3 |
| `timezone` | Tier 3 |
| `language` | Tier 3 |
| `screen_dimensions` | Tier 3 |
| `orientation` | Tier 3 |

### Rate Limiting

Default: 10,000 events/session and 100,000 events/day. Override via
`rateLimitPolicy:`:

```swift
let ratePolicy = RateLimitPolicy(
    maxEventsPerSession: 5_000,
    maxEventsPerDay: 50_000
)

await MetricsCollector.shared.captureDeviceFingerprint(
    ...,
    rateLimitPolicy: ratePolicy
) { _ in }
```

### Retention Policy

Touch events are purged from disk after 7 days; consent receipts after 2 years.
Override via `retentionPolicy:`:

```swift
let retention = RetentionPolicy(
    touchEventTTL: 3 * 24 * 60 * 60,           // 3 days
    consentReceiptTTL: 365 * 24 * 60 * 60       // 1 year
)
```

Purging runs automatically on app launch and before any batch upload.

### Consent Receipts

Every ATT authorization decision is logged as a `ConsentReceipt` containing
the timestamp, active tier, ATT status, and data scope. Receipts provide
demonstrable proof of consent for GDPR audits.

```swift
let receipts = ComplianceManager.shared.loadConsentLog()
```

Receipts are stored in `UserDefaults` and pruned automatically per the
configured retention TTL.
