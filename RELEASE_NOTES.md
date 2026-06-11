# Release Notes - Gameball Android SDK

This file contains detailed release notes for the latest version. For complete version history, see [CHANGELOG.md](CHANGELOG.md).

---

## Latest Release: v3.2.0

**Release Date**: 2026-06-11
**Version**: 3.2.0
**Type**: Minor Release

---

## 🎉 What's New

v3.2.0 introduces a **widget event channel** so your app can react to what customers do inside the widget, **dismissal controls** for both the widget and the host app, and **external-link handling**. All v3.1.x code continues to work without modification — every addition is backward compatible.

### Widget Event Channel

The widget can now post events (e.g. game completion, reward redemption) back to your app. Register `widgetEventCallback` and each event arrives as a `Map<String, Object>` with a top-level `type` and a nested `metadata`:

```kotlin
val request = ShowProfileRequest.builder()
    .customerId("customer_123")
    .widgetEventCallback(object : Callback<Map<String, Any?>> {
        override fun onSuccess(event: Map<String, Any?>) {
            val type = event["type"] as? String                       // e.g. "gameCompleted"
            val metadata = event["metadata"] as? Map<*, *>

            when (type) {
                "gameCompleted" -> {
                    val hasWon = metadata?.get("hasWon") as? Boolean ?: false
                    val campaignId = metadata?.get("campaignId") as? Long   // 90340
                    if (hasWon) refreshBalance()
                }
            }
        }
        override fun onError(e: Throwable) { Log.e("Gameball", "callback error", e) }
    })
    .build()

GameballApp.getInstance(this).showProfile(this, request)
```

> Numbers arrive as `Long` (e.g. `campaignId = 90340`). Cast with `?.toInt()` if you need an `Int`.

### Web-Initiated Close

The widget can dismiss its own webview by calling `window.GameballWidget.closeWidget()` — no host code required.

### Host-Initiated Dismiss

Dismiss the widget programmatically from your app (e.g. on logout or a deep link):

```kotlin
GameballApp.getInstance(context).hideProfile()   // no-op when nothing is shown
```

### External-Link Handling

Links the widget flags with `gbExternalBrowser=true` open in the system browser. Optionally intercept them with `externalLinkCallback`:

```kotlin
val request = ShowProfileRequest.builder()
    .customerId("customer_123")
    .externalLinkCallback(object : Callback<String> {
        override fun onSuccess(url: String) { /* open `url` your own way */ }
        override fun onError(e: Throwable) { }
    })
    .build()
```

---

## 🔄 Changes

- Added `ShowProfileRequest.widgetEventCallback: Callback<Map<String, Object>>?`
- Added `ShowProfileRequest.externalLinkCallback: Callback<String>?`
- Added `GameballApp.hideProfile()`
- Exposed `window.GameballWidget.closeWidget()` to the widget webview

---

## Usage Examples

**React to a reward and refresh the wallet:**
```kotlin
val request = ShowProfileRequest.builder()
    .customerId("customer_123")
    .widgetEventCallback(object : Callback<Map<String, Any?>> {
        override fun onSuccess(event: Map<String, Any?>) {
            val metadata = event["metadata"] as? Map<*, *> ?: return
            if (metadata["hasWon"] as? Boolean == true) {
                showWinAnimation(metadata["rewardName"] as? String ?: "")
                refreshBalance()
            }
        }
        override fun onError(e: Throwable) { }
    })
    .build()

GameballApp.getInstance(this).showProfile(this, request)
```

**Dismiss on logout:**
```kotlin
fun logout() {
    GameballApp.getInstance(this).hideProfile()
    clearSession()
}
```

---

## Requirements

- Android API 21+
- Kotlin 2.0.0+
- AndroidX

---

## Migration

No changes required — all v3.1.x and v3.0.0 code works without modification. The new callbacks and `hideProfile()` are additive.

See [MIGRATION.md](MIGRATION.md) for details.

---

## Installation

```kotlin
dependencies {
    implementation 'com.github.gameballers:gb-mobile-android:3.2.0'
}
```

---

## Support

- 📧 Email: support@gameball.co
- 📖 Documentation: https://developer.gameball.co/
- 🐛 Issues: https://github.com/gameballers/gameball-android/issues

---

## Previous Release: v3.1.1

**Release Date**: 2025-12-15
**Type**: Patch Release

Guest mode support — the profile widget can be shown without customer authentication, and `ShowProfileRequest` no longer requires a customer ID. See [CHANGELOG.md](CHANGELOG.md) for the full history.
