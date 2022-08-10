# Build Documentation

This describes the conventions used for versioning, building, and signing apps published by InSeven.

## Versions

### Build Numbers

The iOS and macOS apps use auto-generated build numbers that attempt to encode the build timestamp, along with some details of the commit used. They follow the format:

```
YYmmddHHMMxxxxxxxx
```

- `YY` -- two-digit year
- `mm` -- month
- `dd` -- day
- `HH` -- hours (24h)
- `MM` -- minutes
- `xxxxxxxx` -- zero-padded integer representation of a 6-character commit SHA

These can be quickly decoded using the `build-tools` script:

```
% scripts/build-tools/build-tools parse-build-number 210727192100869578
2021-07-27 19:21:00 (UTC)
0d44ca
```

## Certificates

### Certificate Requirements

It seems like macOS app (Catalist apps at the very least) require a separate Mac Installer certificate to sign the app on export meaning that apps targeting both platforms require the following certificate types (names take from the developer portal):

- Apple Distribution
- Mac Installer Distribution

Specifically, I _think_ macOS apps can be signed with the 'Apple Distribution' certificate but are submitted to the App Store by means of an installer which needs to be signed independently.

### Updating Certificates

1. Create new certificates.
2. Re-generate profiles, download, and add to project--editing an expired profile will allow you to select a new certificate, save, and dowload.

## Builds

## GitHub Secrets

- `APPLE_API_KEY`

- `APPLE_API_KEY_ID`

- `APPLE_API_KEY_ISSUER_ID`

- `APPLE_DISTRIBUTION_CERTIFICATE_BASE64`

  Create this by base64 encoding the distribution certificate, as exported from Keychain Access:

  ```
  base64 -i "Apple Distribution InSeven Limited.p12" | pbcopy
  ```
  
- `APPLE_DISTRIBUTION_CERTIFICATE_PASSWORD`

  The password used when exporting the distribution certificate.

