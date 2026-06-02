# Installation

### Swift Package Manager

In Xcode, select **File > Add Package Dependencies** and enter the repository URL. Or add it to your `Package.swift`:

```swift
.package(url: "https://github.com/your-org/Lucia-iOS-SDK.git", from: "1.0.0")
```

### Info.plist Configuration

Add your API key:

```xml
<key>LuciaSDKKey</key>
<string>YOUR_API_KEY_HERE</string>
```

Add a tracking permission description (required for IDFA access):

```xml
<key>NSUserTrackingUsageDescription</key>
<string>Used to provide personalized analytics.</string>
```

Optional — enable certificate pinning (see [Security](./security.md)):

```xml
<key>LuciaSDKPinnedCertHashes</key>
<array>
  <string>sha256/YOUR_SPKI_HASH_BASE64=</string>
</array>
```
