# Build Documentation

## Certificate Requirements

It seems like macOS app (Catalist apps at the very least) require a separate Mac Installer certificate to sign the app on export meaning that apps targeting both platforms require the following certificate types (names take from the developer portal):

- Apple Distribution
- Mac Installer Distribution

## Updating Certificates

1. Create new certificates.
2. Re-generate profiles, download, and add to project--editing an expired profile will allow you to select a new certificate, save, and dowload.
