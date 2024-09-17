# Build Documentation

This describes the conventions used for versioning, building, and signing apps published by InSeven.

## Versions

### Build Numbers

The iOS and macOS apps use auto-generated build numbers that attempt to encode the build timestamp, along with some details of the commit used. They follow the format:

```
YYmmddHHMMxxxxxxxx
```

- `YY` – two-digit year
- `mm` – month
- `dd` – day
- `HH` – hours (24h)
- `MM` – minutes
- `xxxxxxxx` – zero-padded integer representation of a 6-character commit SHA

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

### Environment Variables and GitHub Secrets

Builds use base64 encoded [PKCS 12](https://en.wikipedia.org/wiki/PKCS_12) certificate and private key containers specified in the `APPLE_DISTRIBUTION_CERTIFICATE_BASE64` and `MACOS_DEVELOPER_INSTALLER_CERTIFICATE` environment variables (with the password given in the `APPLE_DISTRIBUTION_CERTIFICATE_PASSWORD` and `MACOS_DEVELOPER_INSTALLER_CERTIFICATE_PASSWORD` environment variables respectively). This loosely follows the GitHub approach to [managing certificates](https://docs.github.com/en/actions/guides/installing-an-apple-certificate-on-macos-runners-for-xcode-development).

Keychain Access can be used to export your certificate and private key in the PKCS 12 format, and the base64 encoded version is generated as follows:

```bash
base64 -i "Apple Distribution InSeven Limited.p12" | pbcopy
```

This, along with the password used to protect the certificate, can then be added to the GitHub project secrets.

- `APPLE_API_KEY`

- `APPLE_API_KEY_ID`

- `APPLE_API_KEY_ISSUER_ID`

- `APPLE_DISTRIBUTION_CERTIFICATE_BASE64`

  Create this by base64 encoding the distribution certificate, as exported from Keychain Access:

  ```bash
  base64 -i "Apple Distribution InSeven Limited.p12" | pbcopy
  ```
  
- `APPLE_DISTRIBUTION_CERTIFICATE_PASSWORD`

  The password used for the distribution certificate.

- `MACOS_DEVELOPER_INSTALLER_CERTIFICATE_BASE64`

  This is known by a number of different names across Apple's tooling, including 'Mac Installer Certificate', and '3rd Party Mac Developer Installer'.

- `MACOS_DEVELOPER_INSTALLER_CERTIFICATE_PASSWORD`

  The password used for the developer installer certificate.

## Runners

### macOS

#### Software

Builds expect the following software to be installed:

- asdf
  - lua
  - ruby
- Git
- [GitHub CLI](https://cli.github.com) (via [Homebrew](https://brew.sh))
- Xcode

The GitHub Actions runner uses the path in '~/actions-runner/.path'. This needs to be updated to include Homebrew and asdf:

```
/Users/jbmorley/.asdf/shims:/opt/homebrew/bin:/opt/homebrew/sbin:/usr/local/bin:/System/Cryptexes/App/usr/bin:/usr/bin:/bin:/usr/sbin:/sbin:/var/run/com.apple.security.cryptexd/codex.system/bootstrap/usr/local/bin:/var/run/com.apple.security.cryptexd/codex.system/bootstrap/usr/bin:/var/run/com.apple.security.cryptexd/codex.system/bootstrap/usr/appleinternal/bin:/Library/Apple/usr/bin
```

#### Environment Variables

GitHub Actions runners are expected to define a number of environment variables:

- `IOS_XCODE_PATH`—path to the Xcode install to be used for iOS builds
- `MACOS_XCODE_PATH`—path to the Xcode install to be used for macOS builds
- `DEFAULT_IPHONE_DESTINATION`—details of the preferred iPhone simulator (e.g., 'iOS Simulator,name=iPhone 16 Pro')

These are set in '~/actions-runner/.env':

```
LANG=en_US.UTF-8
IOS_XCODE_PATH=/Applications/Xcode.app
MACOS_XCODE_PATH=/Applications/Xcode.app
```

### Raspbian

#### Software

Builds expect the following software to be installed:

- [GitHub CLI](https://cli.github.com); installed from the downloaded Debian package
- [jq](https://github.com/jqlang/jq); installed via Homebrew
