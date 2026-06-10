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

---

## 🎯 Architectural Justification

1.  **Defense in Depth**: We enforce a multi-tier authentication model. While standard business users utilize the Authenticator App, all **Global Administrators** and high-risk accounts are restricted to phishing-resistant **FIDO2 security keys**.
2.  **Zero Trust Framework**: Every login attempt is verified explicitly using real-time policy evaluation. We mandate Microsoft Authenticator because it feeds device compliance and location signals directly into Entra ID's risk-detection engine.
3.  **MFA Fatigue Controls**: Enforcing number-matching prevents the most common social engineering bypasses, ensuring that authentication remains active and deliberate.

