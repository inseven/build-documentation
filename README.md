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

### Inspecting Certificates

Unlike `.cer` files (which can be viewed using [Quick Look](https://support.apple.com/en-gb/guide/mac-help/mh14119/mac)), macOS doesn't make it particularly easy to work with `.p12` PCKS 12 files; only Keychain Access is able to open these files and they will be automatically added to your keychain. If you want to double-check what's in a PCKS 12 file before adding it to your GitHub secrets, you can do this using `openssl`:

```bash
openssl pkcs12 -info -nodes -in build_certificate.p12
```

### Updating Certificates

1. Create new certificates.
2. Re-generate profiles, download, and add to project--editing an expired profile will allow you to select a new certificate, save, and dowload.

## Builds

### GitHub Secrets

Builds use base64 encoded [PKCS 12](https://en.wikipedia.org/wiki/PKCS_12) certificate and private key containers specified in the `APPLE_DISTRIBUTION_CERTIFICATE_BASE64` and `MACOS_DEVELOPER_INSTALLER_CERTIFICATE` environment variables (with the password given in the `APPLE_DISTRIBUTION_CERTIFICATE_PASSWORD` and `MACOS_DEVELOPER_INSTALLER_CERTIFICATE_PASSWORD` environment variables respectively). This loosely follows the GitHub approach to [managing certificates](https://docs.github.com/en/actions/guides/installing-an-apple-certificate-on-macos-runners-for-xcode-development).

Keychain Access can be used to export your certificate and private key in the PKCS 12 format, and the base64 encoded version is generated as follows:

```bash
base64 build_certificate.p12 | pbcopy
```

This, along with the password used to protect the certificate, can then be added to the GitHub project secrets.

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

