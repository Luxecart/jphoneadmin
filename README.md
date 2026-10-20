# JPhone Web Admin Panel

> **Update:** Panel ab Android app ke Firestore collections (users, premium_plans, subscriptions, admin_notifications, app_config, admin_audit_logs) use karta hai, aur login (Google + admin check) lazmi hai. Hosting steps: `DEPLOY_GUIDE.md`.

A complete, production-ready, security-hardened Web Administration Panel for the **JPhone Android Application** ecosystem, built with React, TypeScript, Tailwind CSS, and Cloud Firestore.

---

## 1. Project Architecture & Preservation

- **Android App Isolation**: The Android module and source structure (`com.aistudio.jphone.qkzvtb`) remain completely untouched and independent.
- **Web Admin Module**: Structured cleanly in the web application directory with dedicated services, components, security rules, and diagnostic suites.
- **Unified Firebase Backend**: Directly connects to the existing JPhone Firebase project, sharing Cloud Firestore and Firebase Authentication with the Android client.
- **Strict Role-Based Access Control (RBAC)**: Enforces server-side authorization against `/admins/{uid}` with Attribute-Based Access Control (ABAC). Ordinary users cannot escalate privileges or alter subscription tiers.

---

## 2. Core Functional Modules

1. **Executive Dashboard**:
   - Total registered users, verified accounts, active vs suspended users.
   - Authoritative subscription tier distribution (FREE vs PREMIUM).
   - Clear indicators distinguishing verified records from pending payment gateway reconciliations.
   - Quick launch for Android diagnostics.

2. **User Management**:
   - Searchable by Name, Email, Phone Number, or Firebase UID.
   - Filter by Tier (FREE/PREMIUM), Account Status (Active/Suspended), and Verification status.
   - User detail drawer with confidential administrative notes editor.
   - Sensitive operations (suspension, reactivation, complimentary grants) require confirmation dialogs and write immutable audit logs.
   - User badges:
     - **FREE**: Green badge with explicit text `FREE`.
     - **PREMIUM**: Blue badge with explicit text `PREMIUM`.

3. **Free & Premium Membership Management**:
   - Plan definitions (Monthly Pro, Annual VIP, Lifetime Pass).
   - SKU, duration in days, pricing, and feature toggle switches.
   - Grant or revoke complimentary Premium access with administrative reasons and custom duration.

4. **Diagnostics & `CONFIGURATION_NOT_FOUND` Investigator**:
   - Deep inspection matrix for the Android error `com.google.firebase.FirebaseException: [CONFIGURATION_NOT_FOUND]`.
   - Complete resolution guide covering Phone Auth provider activation, SHA-1/SHA-256 certificate fingerprints, Play Integrity API, and test phone numbers.
   - Live Phone Auth diagnostic simulator testing gateway status without fabricating OTPs.

5. **Remote App Settings (Consumed by Android)**:
   - App branding, support email, maintenance mode with emergency bypass key and custom messages.
   - Live export of `remote_config.json` with 1-click clipboard copy for Android Studio.

6. **WhatsApp Support Integration**:
   - International E.164 phone number validation.
   - Pre-filled customer inquiry template with URI encoding.
   - Interactive `wa.me` preview and testing button.

7. **Advertising Management**:
   - Placements for Google AdMob (App ID, Banner, Interstitial, Rewarded), AppLovin MAX, and Unity Ads.
   - Master switch to disable ads and Test Mode to prevent policy strikes during QA.

8. **Push Notifications (FCM)**:
   - Broadcast composer with audience targeting (All, Free users, Premium subscribers, Topics).
   - Live Android system notification card mockup with real-time preview.
   - Dispatch confirmation dialogs and broadcast history.

9. **App Update Management**:
   - Version matrix (Min supported version vs Latest version).
   - Play Store download URL and Force Update enforcement.

10. **Immutable Audit Logs**:
    - Append-only security ledger recording administrator UID, action type, target reference, timestamp, and metadata diff.

---

## 3. Getting Started & Local Development

### 1. Install Dependencies
```bash
npm install
```

### 2. Configure Environment Variables
Copy `.env.example` to `.env`:
```bash
cp .env.example .env
```
Fill in your public Firebase Web App credentials (from Firebase Console > Project Settings > General > Your apps > Web app):
```env
VITE_FIREBASE_API_KEY="AIzaSyYourApiKey"
VITE_FIREBASE_PROJECT_ID="your-jphone-project"
VITE_FIREBASE_AUTH_DOMAIN="your-jphone-project.firebaseapp.com"
```

*Note: You can also configure or test credentials live directly in the browser by clicking the environment pill in the header.*

### 3. Run Development Server
```bash
npm run dev
```
Open `http://localhost:3000` in your browser.

---

## 4. Secure First Administrator Provisioning

**CRITICAL SECURITY RULE:** Users never become administrators merely by signing up. Client claims are not trusted.

To grant the first Super Administrator access to `attamuhammad6008@gmail.com`:

1. Download your Service Account Key from **Firebase Console** > **Project Settings** > **Service accounts** > **Generate new private key**.
2. Save it securely as `service-account.json` (do not commit to Git).
3. Obtain your Firebase Auth UID from **Authentication** > **Users**.
4. Run the provisioning script:
```bash
node scripts/provision-first-admin.js <FIREBASE_AUTH_UID> attamuhammad6008@gmail.com
```
This creates the authoritative record in `/admins/{uid}` required by `firestore.rules`.

---

## 5. Deploying Firestore Security Rules

Deploy the hardened ABAC security rules:
```bash
firebase deploy --only firestore:rules
```

---

## 6. Production Build & Firebase Hosting

```bash
# Build the production application
npm run build

# Deploy to Firebase Hosting
firebase deploy --only hosting
```

### Authorized Domains Configuration
In **Firebase Console** > **Authentication** > **Settings** > **Authorized domains**, ensure your hosting domain (e.g. `your-jphone-admin.web.app` or custom domain) is added to allow Google Auth popups.
