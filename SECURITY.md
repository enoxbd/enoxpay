# Security & Release Verification

## Official Release Verification

All official Enox Pay APKs are signed with our release key (`v2` APK Signature Scheme) and distributed exclusively via [GitHub Releases](https://github.com/enoxbd/enoxpay/releases).

Before installing any APK, verify both the file hash and the APK signing certificate:

### 1. Check APK SHA-256 (`v1.0.0`)

```bash
sha256sum EnoxPay-v1.0.0-release.apk
```

Expected digest:

```text
dd6b6696dbd267ef8e33e149fb8a2571b53fdf80cb520de4bb3c3f2669cb3168  EnoxPay-v1.0.0-release.apk
```

### 2. Verify Android Signing Certificate (`apksigner`)

```bash
apksigner verify --print-certs EnoxPay-v1.0.0-release.apk
```

Expected V2 Signer certificate SHA-256 digest:

```text
8ea1a2d2f9079bd22c1816f8a343d424f9350357c0854bd9a5c789d4913105f8
```

---

## Reporting Security Issues

If you discover a vulnerability or suspicious modified build, please contact [@enoxbd](https://t.me/enoxbd) directly rather than posting exploits publicly.
