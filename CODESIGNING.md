# Code Signing for DGGRID Portable Binaries

The portable DGGRID binaries are built via GitHub Actions and distributed as GitHub Release assets. They are currently **unsigned**. This document describes the implications per platform and the path to proper signing.

---

## macOS

macOS Gatekeeper blocks unsigned binaries downloaded from the internet.

### Current workaround for users

After downloading, remove the quarantine attribute before running:

```bash
xattr -dr com.apple.quarantine ./dggrid
```

### Levels of signing

| Level | Cost | Satisfies Gatekeeper | Notes |
|---|---|---|---|
| Ad-hoc (`codesign --sign -`) | Free | No | Removes "app is damaged" error but Gatekeeper still blocks downloaded binaries |
| Developer ID signing | $99/year (Apple Developer account) | Yes | Certificate stored as GitHub secret, applied in CI |
| Notarization | Included with Developer ID | Yes (required on modern macOS) | Binary submitted to Apple for scanning via `xcrun notarytool`, adds ~2 min to build |

### Path to proper signing

1. Enrol in the [Apple Developer Program](https://developer.apple.com/programs/)
2. Create a **Developer ID Application** certificate in Xcode / Apple Developer portal
3. Export as `.p12` and store as GitHub repository secrets:
   - `MACOS_CERTIFICATE` — base64-encoded `.p12` file
   - `MACOS_CERTIFICATE_PWD` — certificate password
   - `APPLE_ID`, `APPLE_ID_PASSWORD`, `APPLE_TEAM_ID` — for notarization
4. Add signing and notarization steps to the macOS jobs in `portables.yml`

---

## Windows

Windows SmartScreen displays a warning for unsigned executables downloaded from the internet. Users can bypass it by clicking "More info → Run anyway", but this is not ideal for wider distribution.

### Signing options

| Option | Cost | Notes |
|---|---|---|
| OV code signing certificate | ~$300+/year | From CAs such as DigiCert, Sectigo. Applied with `signtool.exe` in CI. |
| EV code signing certificate | ~$500+/year | Stored on a hardware token — harder to use in CI without a dedicated signing service |
| [SignPath.io](https://signpath.io) | **Free for open source** | Manages the certificate and signing workflow; integrates with GitHub Actions |

SignPath.io is the recommended path for open source projects — it provides proper OV/EV signing without the certificate cost.

---

## Linux

There is no user-facing equivalent to Gatekeeper or SmartScreen on Linux. However, supply chain integrity can be provided via:

- **GitHub artifact attestations** (SLSA provenance) — free, generated automatically by `actions/attest-build-provenance` in the workflow. Allows users to verify a binary was built by this repository's Actions workflow.
- **GPG signing** of release assets — a project GPG key signs a checksum file (`SHA256SUMS.gpg`) attached to the release.

---

## Current status

| Platform | Signing | Gatekeeper / SmartScreen |
|---|---|---|
| Linux | None (attestation planned) | N/A |
| Windows | None | Warning shown |
| macOS x64 | None | Blocked — use `xattr` workaround |
| macOS ARM64 | None | Blocked — use `xattr` workaround |

---

## References

- [Apple Notarizing macOS Software](https://developer.apple.com/documentation/security/notarizing_macos_software_before_distribution)
- [GitHub artifact attestations](https://docs.github.com/en/actions/security-for-github-actions/using-artifact-attestations/using-artifact-attestations-to-establish-provenance-for-builds)
- [SignPath open source program](https://signpath.io/product/open-source)
