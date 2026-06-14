# Authentication Methods & Justifications

This document provides a detailed overview of the authentication methods enabled in the Microsoft Entra ID tenant. It evaluates each method against the criteria of **Security** (resilience to attacks) and **Usability** (friction, onboarding ease), justifying their role in our Zero Trust identity architecture.

---

## 📊 Summary Matrix

| Authentication Method | Security Level | Usability Level | Primary Attack Vector Vulnerability | Best Used For |
| :--- | :--- | :--- | :--- | :--- |
| **Microsoft Authenticator (Push + Number Match)** | High | High | Device compromise, session hijacking | General employees, standard operations |
| **Passkey (FIDO2) / Security Keys** | Critical | Medium-Low | Physical key theft | Privileged accounts, system administrators |
| **Temporary Access Pass (TAP)** | High | Medium | Passcode theft / social engineering | User onboarding, passwordless registration |
| **Software OATH Tokens** | Medium | High | Phishing, device compromise | Third-party authenticators, backup access |
| **SMS Verification** | Low-Medium | High | SIM swapping, phishing, interception | Secondary backup, remote fallback |
| **Email OTP** | Low | High | Mailbox compromise, phishing | External collaboration, guest user fallback |

---

## 📱 1. Microsoft Authenticator App (Push Notifications & OTP)

The Microsoft Authenticator App is the recommended primary authentication method for general users.

### Security Assessment:
*   **MFA Fatigue Mitigation**: Enforces **number-matching** (verified during setup with code **64**). This requires the user to type a 2-digit code shown on the login screen into their phone, preventing "push bombing" where users blindly click "Approve" due to receiving multiple notifications.
*   **Contextual Awareness**: Displays the application name and geographic location of the login request, allowing users to recognize unauthorized login attempts immediately.
*   **Phishing Resistance**: Highly resistant to standard credential phishing, though still vulnerable to advanced proxy-based phishing (e.g., AitM - Adversary-in-the-Middle) unless configured with device-binding.

### Usability Assessment:
*   **Minimal Friction**: Approval requires a simple push notification response, reducing manual verification code typing.
*   **Offline Functionality**: If mobile data is unavailable, the app generates a time-based One-Time Passcode (TOTP) every 30 seconds that can be typed in manually.
*   **App Dependency**: Requires users to install a corporate application on their mobile devices, which can sometimes raise personal privacy concerns for employees.

---

## 🔑 2. Passkey (FIDO2 Security Keys)

FIDO2 Security keys (such as YubiKeys) and platform passkeys (Windows Hello, Apple FaceID/TouchID) utilize cryptography to provide passwordless, phishing-resistant authentication.

### Security Assessment:
*   **Phishing Resistant**: FIDO2 authentication is cryptographically bound to the specific domain (e.g., `login.microsoftonline.com`). If a user is tricked into visiting a fake login page, the hardware key or passkey will refuse to authenticate because the domain name does not match, completely neutralizing Adversary-in-the-Middle (AitM) phishing attacks.
*   **No Shared Secret**: Cryptographic keys remain stored on the physical device or platform secure enclave, preventing server-side credential theft.
*   **Zero Trust Alignment**: Represents the gold standard of the Zero Trust model (Verify Explicitly) by validating hardware possession.

### Usability Assessment:
*   **Hardware / Platform Overhead**: Requires distributing physical keys or ensuring that users are on compatible devices with local biometrics.
*   **Friction**: High initial enrollment setup friction, but very low day-to-day login friction (simple tap of the physical key or biometric verification).

---

## 🎫 3. Temporary Access Pass (TAP)

A Temporary Access Pass is a time-limited passcode configured by an administrator that acts as a strong single-use credential.

### Security Assessment:
*   **Onboarding Security**: Replaces the insecure practice of sending temporary clear-text passwords to new employees.
*   **Bootstrap Factor**: Allows new users to complete their initial passwordless registration (such as registering FIDO2 keys or Microsoft Authenticator) without ever creating or typing a traditional password.

### Usability Assessment:
*   **Limited Lifetime**: Automatically expires after a configured duration (e.g., 1 hour to 24 hours), reducing the risk window if intercepted.

---

## 💬 4. SMS Verification (Short Message Service)

SMS sends a temporary 6-digit passcode to the user's registered cellular phone number.

### Security Assessment:
*   **Low Security (Weakest Factor)**: Vulnerable to **SIM Swapping** attacks, where attackers impersonate the victim to transfer their cell service to a hacker-controlled SIM card.
*   **Interception Risk**: SMS messages travel over unencrypted cellular networks, making them vulnerable to SS7 interception and base station spoofing.
*   **Phishing Susceptibility**: Users are highly accustomed to typing 6-digit codes into forms, making them easy targets for basic phishing pages that harvest and replay the codes.

### Usability Assessment:
*   **Ubiquity**: Requires no application installs or special hardware; works on any mobile phone globally.
*   **Fallback Justification**: Despite security flaws, SMS is enabled solely as a secondary fallback method to prevent lockouts during device transitions or travel where internet connectivity is limited.

## 🔑 5. Backup Codes (Recovery Codes) & Recovery Mechanisms

To ensure operational resilience and prevent permanent account lockouts when a user loses their primary authentication device (e.g., cell phone or physical security key), a robust recovery mechanism must be designed, documented, and secured.

### 5.1 Microsoft Authenticator Cloud Backup
For standard users, Microsoft Authenticator supports cloud backup of account credentials, allowing them to restore their MFA accounts on a new device without administrative intervention.
*   **How it Works**: The app encrypts the accounts and backs them up to iCloud (iOS) or Google Drive (Android) using the user's personal Microsoft Account (MSA) or Apple ID.
*   **Security Best Practice**: The backup is encrypted using a recovery passphrase that the user must generate. Users must write down this recovery passphrase and store it in a secure password vault or physical safe. Without this passphrase, the backup cannot be decrypted.

### 5.2 Personal Microsoft Account (MSA) Recovery Codes
For personal Microsoft accounts, Microsoft allows the generation of a unique 25-character **Recovery Code** (e.g., `XXXXX-XXXXX-XXXXX-XXXXX-XXXXX`).
*   **Process**: Generated via Security settings -> Advanced security options.
*   **Usage**: Can bypass all other MFA prompts and passwords. It is a one-time use code that must be regenerated immediately after use.
*   **Secure Storage Policy**: Because this code provides absolute control over the account, it must **never** be stored in plain text on cloud drives (OneDrive, Google Drive), email drafts, or local computer text files. It must be stored in:
    1.  An encrypted password manager (e.g., Bitwarden, 1Password) with Master MFA enabled.
    2.  An offline physical copy printed or written and placed in a secure, fireproof safe.

### 5.3 Enterprise Recovery: Temporary Access Pass (TAP)
In an enterprise Microsoft Entra ID tenant (like `olamc.onmicrosoft.com`), self-service "personal recovery codes" are generally disabled for security. Instead, account recovery is managed through **Temporary Access Passes (TAPs)**.
*   **Process**: An administrator navigates to the user's authentication methods and generates a TAP.
*   **Configuration**: The admin sets a specific start time, duration (e.g., 1 hour), and specifies whether it is one-time use.
*   **Usage**: The user signs in using the TAP, bypassing standard MFA. This allows them to securely access `mysignins.microsoft.com/security-info` to register their new phone or Authenticator app.

### 5.4 Recovery Code Generation & Storage Checklist
Below is the mandatory security checklist for generating, securing, and managing static recovery codes:

#### 📋 Generation Steps
- [ ] **Navigate to Security Settings**: Access your Microsoft Account Advanced Security settings portal.
- [ ] **Generate Recovery Code**: Scroll to the "Recovery code" section and select **Generate a new code**. *(See `screenshots/recovery_code_generation.png`)*.
- [ ] **Acknowledge Invalidation**: Confirm that generating a new code automatically invalidates any previously issued recovery codes.

#### 🔒 Secure Storage Strategy
- [ ] **No Cleartext Cloud Storage**: Never store the recovery code in cleartext files (e.g., `.txt`, `.docx`, `.xlsx`) in cloud storage (OneDrive, Google Drive) or email drafts.
- [ ] **Encrypted Password Vault**: Store a digital copy inside an encrypted password manager (e.g., Bitwarden, 1Password) protected by Multi-Factor Authentication (MFA) on the master account.
- [ ] **Offline Physical Storage**: Print or write the 25-character code on paper and store it in a physical fireproof/waterproof home safe or safety deposit box.

#### 🔄 Lifecycle & Recovery Management
- [ ] **Single-Use Policy**: Treat the recovery code as a one-time credential. If used to recover the account, immediately log back in and generate a new code.
- [ ] **Annual Rotation**: Review and rotate the recovery code annually to ensure key integrity.

---

## 🎯 Architectural Justification

1.  **Defense in Depth**: We enforce a multi-tier authentication model. While standard business users utilize the Authenticator App, all **Global Administrators** and high-risk accounts are restricted to phishing-resistant **FIDO2 security keys**.
2.  **Zero Trust Framework**: Every login attempt is verified explicitly using real-time policy evaluation. We mandate Microsoft Authenticator because it feeds device compliance and location signals directly into Entra ID's risk-detection engine.
3.  **MFA Fatigue Controls**: Enforcing number-matching prevents the most common social engineering bypasses, ensuring that authentication remains active and deliberate.

