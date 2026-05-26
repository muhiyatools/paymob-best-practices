# Mobile SDKs

All mobile SDKs share the same backend flow: **your backend creates the Intention** → **mobile app passes client_secret to the SDK** → **SDK handles checkout** → **callbacks fire on result**.

---

## Common Architecture

```
Your Backend                    Mobile App                     Paymob
    │                              │                             │
    │── POST /v1/intention/ ──────│─────────────────────────────│
    │← { client_secret, order.id }│─────────────────────────────│
    │                              │                             │
    │                              │── Init SDK ────────────────│
    │                              │   (csk + publicKey)         │
    │                              │                             │
    │                              │◄── SDK shows UI ───────────│
    │                              │                             │
    │◄── POST callback ───────────│─────────────────────────────│
    │   (HMAC-verified)           │                             │
    │                              │                             │
    │                              │◄── SDK callback ───────────│
    │                              │   (success/rejected/pending)│
```

---

## iOS SDK

### Installation

**CocoaPods** (recommended):
```ruby
pod 'Paymob'
```

**Manual**: Download `PaymobSDK.xcframework` → drag to Xcode → Embed & Sign.

### Usage

```swift
import PaymobSDK

class CheckoutViewController: UIViewController, PaymobSDKDelegate {
    func startPayment() {
        PaymobSDK.initiatePayment(
            controller: self,
            delegate: self,
            publicKey: "egy_pk_live_XXXX",
            clientSecret: "egy_csk_live_XXXX"
        )
    }

    // MARK: Delegate callbacks
    func transactionAccepted(transactionDetails: [String: Any]) { /* success */ }
    func transactionRejected(message: String) { /* declined */ }
    func transactionPending() { /* pending */ }
}
```

### Response Callback URL

Configure the integration ID's Response Callback URL for the SDK callbacks to fire. Region-specific URLs:

| Region | Response Callback URL |
|---|---|
| Egypt | `https://accept.paymob.com/api/acceptance/post_pay` |
| KSA | `https://ksa.accept.paymob.com/api/acceptance/post_pay` |
| UAE | `https://uae.accept.paymob.com/api/acceptance/post_pay` |
| Oman | `https://oman.accept.paymob.com/api/acceptance/post_pay` |

Required for the SDK delegate callbacks (`transactionAccepted`, `transactionRejected`, `transactionPending`) to fire correctly.

---

## Android SDK

### Installation

1. Download SDK `.aar` from Paymob
2. Place in `app/libs/`
3. Add to `settings.gradle.kts`:
```kotlin
repositories {
    maven { url = rootProject.projectDir.toURI().resolve("libs") }
    maven { url = uri("https://jitpack.io") }
}
```
4. Add dependency in `app/build.gradle.kts`:
```kotlin
implementation("com.paymob.sdk:Paymob-SDK:{{latest_version}}")
android { buildFeatures { dataBinding = true } }
```

### Usage

```kotlin
import com.paymob.paymob_sdk.PaymobSdk
import com.paymob.paymob_sdk.ui.PaymobSdkListener

class MainActivity : AppCompatActivity(), PaymobSdkListener {
    fun startPayment() {
        PaymobSdk.getInstance()
            .setPublicKey("egy_pk_live_XXXX")
            .setClientSecret("egy_csk_live_XXXX")
            .start(this@MainActivity, this@MainActivity)
    }

    override fun onSuccess() { /* success */ }
    override fun onFailure() { /* declined */ }
    override fun onPending() { /* pending */ }
}
```

Same Response Callback URL requirement as iOS.

---

## Flutter SDK

Bridge connecting Flutter to native iOS/Android SDKs.

### Supported Methods

Cards, Wallets, Apple Pay, Google Pay, Bank Installments, Valu, Souhoola, Forsa, Premium, Aman Installments.

### Requirements

- Min Android SDK: 23
- Compile SDK: 33

### Usage

```dart
import 'package:flutter/services.dart';

static const methodChannel = MethodChannel('paymob_sdk_flutter');

Future<void> payWithPaymob(String pk, String csk) async {
  try {
    final String result = await methodChannel.invokeMethod('payWithPaymob', {
      "publicKey": pk,
      "clientSecret": csk,
      "appName": "My App",
      "buttonBackgroundColor": 0xFF3A57E8,
      "buttonTextColor": 0xFFFFFFFF,
      "saveCardDefault": true,
      "showSaveCard": true,
    });

    switch (result) {
      case 'Successfull': /* success */ break;
      case 'Rejected': /* declined */ break;
      case 'Pending': /* pending */ break;
    }
  } on PlatformException catch (e) {
    print("Payment error: ${e.message}");
  }
}
```

---

## React Native SDK

### Installation

```bash
yarn add paymob-reactnative@https://github.com/PaymobAccept/paymob-reactnative-sdk.git
```

Android: enable data binding in `app/build.gradle`:
```java
android {
    buildFeatures { dataBinding = true }
}
```

### Usage

```javascript
import Paymob, { PaymentResult } from 'paymob-reactnative';

// Customize (MUST be done before presentPayVC)
Paymob.setAppIcon(base64Image);
Paymob.setAppName('My App');
Paymob.setButtonTextColor('#FFFFFF');
Paymob.setButtonBackgroundColor('#3A57E8');
Paymob.setShowSaveCard(true);
Paymob.setSaveCardDefault(true);

// Present payment screen
Paymob.presentPayVC(publicKey, clientSecret);

// Listen for result
const listener = Paymob.addPaymentResultListener((result) => {
  switch (result) {
    case PaymentResult.SUCCESSFUL: /* success */ break;
    case PaymentResult.REJECTED: /* declined */ break;
    case PaymentResult.PENDING: /* pending */ break;
  }
});
```

**Important**: All customization must be done **before** calling `presentPayVC`. Changes after won't apply.

---

## Supported Payment Methods by SDK

| Method | iOS | Android | Flutter | React Native |
|---|---|---|---|---|
| Cards | Yes | Yes | Yes | Yes |
| Wallets | Yes | Yes | Yes | Yes |
| Apple Pay | Yes | — | Yes | Yes |
| Google Pay | Yes | Yes | Yes | Yes |
| Bank Installments | Yes | Yes | Yes | Yes |
| Valu | Yes | Yes | Yes | Yes |
| Souhoola | Yes | Yes | Yes | Yes |
| Forsa | Yes | Yes | Yes | Yes |

---

## Anti-Patterns

- **Do not use WebView for Apple Pay** — Apple Pay is not supported in WebViews. Use native SDK.
- **Do not create the Intention on the mobile device** — always create it from your backend to avoid exposing your secret key.
- **Do not call `presentPayVC` before customization** (React Native) — customization won't take effect.
- **Do not skip the Response Callback URL configuration** — SDK callbacks won't fire without it.
