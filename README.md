# Enox Pay

**Automated Mobile Banking SMS & Notification Payment Gateway for Android**

Enox Pay captures incoming Mobile Financial Service (MFS) payment notifications and SMS transactions on Android, extracts structured transaction metadata locally, and forwards signed real-time webhooks to your merchant backend with retry queues, Telegram alerts, CSV exports, and analytics.

[![Download v1.0.0 APK](https://img.shields.io/badge/Download-Enox%20Pay%20v1.0.0-6366F1?style=for-the-badge&logo=android&logoColor=white)](https://github.com/enoxbd/enoxpay/releases/latest)
[![Platform](https://img.shields.io/badge/Android-7.0%2B%20(API%2024--36)-10B981?style=for-the-badge)](https://github.com/enoxbd/enoxpay/releases)

---

## Download & Verify (`v1.0.0`)

1. Open [**Releases → v1.0.0**](https://github.com/enoxbd/enoxpay/releases/tag/v1.0.0) and download **`EnoxPay-v1.0.0-release.apk`**.
2. Install the APK on your Android device (**Android 7.0+** / API 24–36).
3. Verify the authenticity of your download against the official checksums:

| Property | Verified Value |
| --- | --- |
| **Package ID** | `com.enox.enoxpay.bmdxyz` |
| **Version** | `1.0.0` (`versionCode: 1`) |
| **APK SHA-256** | `dd6b6696dbd267ef8e33e149fb8a2571b53fdf80cb520de4bb3c3f2669cb3168` |
| **APK Signer V2 Cert SHA-256** | `8ea1a2d2f9079bd22c1816f8a343d424f9350357c0854bd9a5c789d4913105f8` |

> See [`SECURITY.md`](SECURITY.md) for verification commands with `sha256sum` or `apksigner`.

---

## Key Capabilities

- **Instant SMS & Notification Capture**: Parses incoming MFS alerts (bKash, Nagad, Rocket, Upay, and custom regex profiles) with dual-SIM slot tracking.
- **Reliable Webhook Delivery**: Customizable JSON payload templates, Bearer/API-Key/custom headers, HMAC-SHA256 request signing (`X-Enox-Signature`), automatic WorkManager retries, and live test pings.
- **Persistent Background Gateway**: Foreground service with battery-optimization diagnostics, watchdog checks, and instant boot recovery.
- **Analytics & Transaction Logs**: Real-time volume/success metrics, status filtering (`SUCCESS`, `PENDING`, `FAILED`), search, trash recovery, and one-tap CSV export.
- **Telegram Bot Alerts**: Optional capture notifications, webhook delivery reports, and scheduled gateway heartbeats.
- **App Lock & Privacy**: Optional biometric / PIN app lock and encrypted local backup/restore.

---

## Quick Setup Guide

1. **Install & Grant Permissions**: Open **Permissions** inside Enox Pay and grant SMS access, Notification Listener (if capturing app notifications), Draw Over Other Apps (optional keep-alive stability), and disable battery optimization for uninterrupted background delivery.
2. **Configure Webhook**: Open the **Config** tab, enter your HTTPS webhook endpoint URL, configure auth headers and optional **Webhook Secret** for HMAC-SHA256 signing, then tap **Test Ping**.
3. **Start Gateway**: Return to **Home** and toggle **Gateway Active**.

---

## Webhook Payload & Signature Verification

Default JSON payload sent via `POST` (`Content-Type: application/json; charset=utf-8`):

```json
{
  "platform": "bKash",
  "amount": "1500.00",
  "transaction_id": "CL48X9P2KQ",
  "sender": "017XXXXXXXX",
  "balance": "12450.00",
  "fee": "0.00",
  "sim_slot": "SIM 1",
  "source": "SMS",
  "sms": "You have received Tk 1,500.00 from 017XXXXXXXX...",
  "time": "1794486000000"
}
```

When a **Webhook Secret** is configured, Enox Pay includes:
- `X-Enox-Event`: `transaction.captured`
- `X-Enox-Timestamp`: `<unix_seconds>`
- `X-Enox-Signature`: `sha256=<hex_hmac_sha256(secret, "$timestamp.$rawBody")>`

For full headers, template placeholders, and server verification snippets (Node.js, PHP, Python), see [`docs/WEBHOOK_GUIDE.md`](docs/WEBHOOK_GUIDE.md).

---

## Support & Links

- **Official Support (Telegram):** [@enoxbd](https://t.me/enoxbd)
- **Live Release Notes:** [`CHANGELOG.md`](CHANGELOG.md)
- **Security & Checksums:** [`SECURITY.md`](SECURITY.md)
