# Mr Womens Billing App — Setup, Testing & Capacitor APK Guide

**Brand:** Mr Womens — *Shine Everyday ✦ Stay Beautiful*  
**Address:** Tirupur Road, Kangayam  
**Phone:** 9600670996  
**GSTIN:** 33GHWPS6754J1ZA  
**Target Printer:** POSiFLOW PSF20 58mm Bluetooth / USB Thermal Printer

---

## 1. Project File Structure

```text
mr-womens-billing-app/
├── capacitor.config.json       # Capacitor Android configuration
├── package.json                # Capacitor & build scripts
├── README.md                   # Setup, testing & native printer documentation
└── www/                        # Web assets directory packaged into Android APK
    ├── index.html              # Semantic HTML5 mobile-first shell
    ├── styles.css              # Boutique blush-pink & magenta theme + 58mm @media print CSS
    ├── db.js                   # IndexedDB persistent database & atomic bill transactions
    ├── receipt.js              # 58mm thermal receipt & ESC/POS byte stream generator
    ├── app.js                  # Main UI state, billing math, calculator, history & reports
    ├── sw.js                   # Offline-first Service Worker asset caching
    ├── manifest.json           # Installable PWA Manifest
    └── assets/
        └── mr-womens-logo.jpg  # Original Mr Womens cartoon girl logo & wordmark
```

---

## 2. Local Web Testing Instructions

1. Extract this ZIP archive to a folder (e.g., `mr-womens-billing-app/`).
2. Serve the `www/` directory over HTTP (required for Service Worker & IndexedDB):
   ```bash
   npx serve www -l 3000
   ```
3. Open `http://localhost:3000` in Chrome (use Chrome DevTools Device Toolbar set to an Android viewport such as **Pixel 7 / Galaxy S23** at `390x844`).
4. **Verify Offline Operation:**
   - Load the app once so `sw.js` caches all assets and IndexedDB initializes.
   - In Chrome DevTools -> **Network**, toggle **Offline**.
   - Refresh the page and create a bill: all products, categories, GST math, and bill saves continue working 100% offline.
5. **Verify GST Calculation:**
   - Tap **New Bill** -> Add `Earrings` (`₹250`, `3% GST`).
   - Verify: `Subtotal: ₹250.00`, `GST (3%): ₹7.50`, `Grand Total: ₹257.50`.
   - Or load the 3-item reference invoice (`2× Hair Clip Set ₹100 + 1× Earrings ₹250 + 1× Makeup Sponge ₹80`) with `12% GST` -> `Subtotal: ₹530.00`, `GST: ₹63.60`, `Grand Total: ₹593.60`.

---

## 3. Packaging as an Android APK with Capacitor

```bash
# 1. Install Capacitor dependencies
npm install

# 2. Add the Android native platform
npx cap add android

# 3. Copy web assets (www/) into the Android project
npx cap sync android

# 4. Open in Android Studio to build APK or run on your phone
npx cap open android
```

In Android Studio:
- Go to **Build -> Build Bundle(s) / APK(s) -> Build APK(s)** to generate the installable `.apk` file for your Android phone.

---

## 4. POSiFLOW PSF20 58mm Printer Integration Guide

### A. Browser / System Print Fallback (Implemented & Working Now)
- Tapping **Print** on the New Bill or Bill Preview screen formats the receipt strictly for **58mm paper width** (`48mm` printable area) using `@media print { @page { size: 58mm auto; margin: 0mm; } }`.
- On Android, if you have a USB-OTG connection or an Android Print Service plugin (such as **RawBT Print Service** or **NokoPrint** installed from Google Play), `window.print()` routes directly to the POSiFLOW PSF20 58mm printer.

### B. Native Bluetooth ESC/POS Integration in Capacitor (Separate Native Step)
> **Truthful Architecture Note:** Standard mobile web browsers cannot send raw Classic Bluetooth SPP (Serial Port Profile) ESC/POS bytes without a native bridge or companion print service.

To enable direct silent one-tap Bluetooth ESC/POS printing inside the Capacitor Android APK:

1. Install the Capacitor Bluetooth Serial / Thermal Printer plugin:
   ```bash
   npm install @capacitor-community/bluetooth-le
   # Or for Classic Bluetooth SPP (used by POSiFLOW PSF20):
   npm install cordova-plugin-bluetooth-serial
   npx cap sync android
   ```
2. Add Bluetooth permissions in `android/app/src/main/AndroidManifest.xml`:
   ```xml
   <uses-permission android:name="android.permission.BLUETOOTH" />
   <uses-permission android:name="android.permission.BLUETOOTH_ADMIN" />
   <uses-permission android:name="android.permission.BLUETOOTH_CONNECT" />
   <uses-permission android:name="android.permission.BLUETOOTH_SCAN" />
   ```
3. Pair the **POSiFLOW PSF20** in Android Settings -> Connected Devices (Default PIN: `0000` or `1234`).
4. Pass the `Uint8Array` returned by `buildEscPos58mmPayload(bill, settings)` (already implemented in `receipt.js`) to `bluetoothSerial.write(payload)`.
