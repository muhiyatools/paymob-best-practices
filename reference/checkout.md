# Checkout: Unified + Pixel

After creating an Intention, present the checkout to the customer.

---

## Unified Checkout (Redirect)

Fastest integration. Redirect the browser to a Paymob-hosted page.

```
https://accept.paymob.com/v1/unifiedcheckout/?clientSecret={client_secret}&publicKey={public_key}
```

### How It Works

1. Customer is redirected to Paymob's hosted checkout
2. All enabled payment methods are displayed
3. Customer enters details, completes 3DS auth
4. Paymob redirects back to your **Transaction Response Callback URL**
5. Paymob sends **Transaction Processed Callback** to your `notification_url`

### Customization

Via Paymob Dashboard → Checkout Customization:
- Brand colors, fonts, button styles (rounded/rectangular)
- Payment method display (tabs or list)
- Billing address fields (show/hide)
- Save Card option
- Split Payments card count
- Terms & Conditions link
- Payment retries (enable/disable)
- Post-payment redirect + thank-you message

### Mobile WebView

Works in mobile WebViews but **Apple Pay is not supported** in WebViews. Use native SDK for Apple Pay.

---

## Pixel (Embedded JS SDK)

Embed Paymob's checkout UI directly on your page. Payment data stays within Paymob's iframe — your site stays out of PCI scope.

### Setup

```html
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/paymob-pixel@latest/styles.css">
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/paymob-pixel@latest/main.css">
<script src="https://cdn.jsdelivr.net/npm/paymob-pixel@latest/main.js" type="module"></script>

<div id="paymob-elements"></div>
```

### Initialization

```javascript
new Pixel({
  publicKey: 'egy_pk_live_XXXX',
  clientSecret: 'egy_csk_live_XXXX',
  paymentMethods: ['card', 'wallet', 'apple-pay', 'google-pay'],
  elementId: 'paymob-elements',
  disablePay: false,
  showSaveCard: true,
  forceSaveCard: false,
  beforePaymentComplete: async (paymentMethod) => {
    // Validate cart, check inventory, etc.
    return true; // false to abort
  },
  afterPaymentComplete: async (response) => {
    // Payment done — redirect to success page
    // Do NOT rely on this for server-side confirmation
    window.location.href = '/order/success?id=' + response.id;
  },
  onPaymentCancel: () => {
    window.location.href = '/order/cancelled';
  },
  cardValidationChanged: (isValid) => {
    // Enable/disable your custom pay button
  },
  customStyle: {
    Font_Family: 'Inter, sans-serif',
    Font_Size_Label: '16',
    Font_Size_Input_Fields: '16',
    Font_Size_Payment_Button: '14',
    Font_Weight_Label: 400,
    Font_Weight_Input_Fields: 400,
    Font_Weight_Payment_Button: 500,
    Primary_Color: '#3A57E8',
    Button_Text_Color: '#FFFFFF',
    Border_Radius: '8'
  }
});
```

### Configuration Reference

| Option | Type | Description |
|---|---|---|
| `publicKey` | string | Required. Your Paymob public key |
| `clientSecret` | string | Required. From Create Intention response |
| `paymentMethods` | string[] | `card`, `wallet`, `apple-pay`, `google-pay`, `valu`, etc. |
| `elementId` | string | HTML container element ID |
| `disablePay` | bool | Disable Paymob's pay button (use your own) |
| `showSaveCard` | bool | Show save card checkbox |
| `forceSaveCard` | bool | Force save (no opt-out) |
| `beforePaymentComplete` | function(paymentMethod) → bool | Validation hook |
| `afterPaymentComplete` | function(response) | Post-payment callback |
| `onPaymentCancel` | function() | Cancel handler |
| `cardValidationChanged` | function(isValid) | Card form validity |
| `customStyle` | object | Style overrides (see example above) |

### Apple Pay Pixel Requirements

- Domain must be verified with Apple (see `reference/testing.md`)
- Apple Pay certificates must be created and uploaded to Paymob Dashboard

---

## Choosing the Right Checkout

| Consideration | Unified Checkout | Pixel |
|---|---|---|
| Implementation effort | Redirect URL | JS SDK integration |
| Brand control | Dashboard settings | Full custom CSS |
| PCI scope | Out of scope | Out of scope (iframe) |
| Apple Pay browser | Yes | Yes (with domain verification) |
| Apple Pay WebView | **No** | **No** — use native SDK |
| Google Pay | Yes | Yes |
| Best for | Quick launch, plugins | Branded checkout |

---

## Anti-Patterns

- **Do not use Pixel's `afterPaymentComplete` as payment confirmation** — it fires client-side and can be manipulated. Only the server-side Transaction Processed Callback is trustworthy.
- **Do not put Apple Pay in a WebView** — it won't work. Use native SDKs.
- **Do not load Pixel before Intention creation** — you need the `client_secret` first.
- **Do not cache Pixel CSS/JS in a way that blocks updates** — always use `@latest` version from CDN.
