# Security

### Certificate Pinning

The SDK supports SPKI SHA-256 certificate pinning to protect against MITM
attacks. Add one or more pin hashes to your app's `Info.plist`:

```xml
<key>LuciaSDKPinnedCertHashes</key>
<array>
  <string>sha256/YOUR_SPKI_HASH_BASE64=</string>
</array>
```

Pinning is enforced for the following hosts:

- `api.clickinsights.xyz`
- `staging.api.clickinsights.xyz`
- `test.api.clickinsights.xyz`

The SDK validates the leaf certificate's Subject Public Key Info (SPKI) after
the system's standard chain validation passes — pinning is an addition to PKI,
not a replacement.

If no hashes are declared, the SDK falls back to the system's default trust
evaluation. Existing integrations are unaffected until you add pins.

### At-Rest Encryption

Touch events are encrypted with **AES-256-GCM** before being written to the
app's Documents directory (`TouchEvents/`). Each file is a sealed box
(nonce + ciphertext + authentication tag).

The encryption key is generated once per install and stored in the Keychain
with `kSecAttrAccessibleAfterFirstUnlockThisDeviceOnly`. The data cannot be
read off a device without access to that device's Keychain.

Legacy plaintext `.json` files written by earlier SDK versions are
transparently re-encrypted to `.evt` on the next read.
