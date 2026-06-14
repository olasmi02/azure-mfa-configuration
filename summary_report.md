# Azure MFA Configuration Summary Report

This report documents the step-by-step administrative actions taken within the Microsoft Entra admin center to configure, enforce, and verify Multi-Factor Authentication (MFA) settings for the tenant `olamc.onmicrosoft.com`.

---

## ⚙️ Step 1: Navigating to the Entra ID Interface
1.  Open a browser and navigate to the **Microsoft Entra admin center** (`https://entra.microsoft.com`).
2.  Sign in using the tenant administrator account **OlamileyeDuduyemi@olamc.onmicrosoft.com**.
3.  On the left navigation pane, expand the **Identity** menu to expose user management, groups, and security settings.

---

## 🔒 Step 2: Selecting and Enabling Authentication Methods
To enforce strong security, we configured the global authentication methods policy:

1.  Navigate to **Identity** > **Protection** > **Authentication methods** > **Policies**.
2.  In our tenant, the following methods are enabled:
    *   **Microsoft Authenticator**: Enabled for **All users** (configured with push notifications and number-matching enabled).
    *   **Passkey (FIDO2)**: Enabled for **All users** (provides passwordless security key options).
    *   **SMS**: Enabled for **All users** (configured as a secondary verification option).
    *   **Temporary Access Pass**: Enabled for **All users** (facilitates recovery and initial passwordless setup).
    *   **Software OATH tokens**: Enabled for **All users** (supports standard authenticator app codes).
    *   **Email OTP**: Enabled for **All users** (one-time passcode fallback sent to registered email addresses).
3.  Click **Save** at the bottom of the portal.

---

## 👥 Step 3: Creating the Test User Account
To verify the registration and login flows, a dedicated test user account was created:

1.  Navigate to **Identity** > **Users** > **All users** > **New user** > **Create new user**.
2.  Configure the following attributes:
    *   **User principal name**: `testusermfa@olamc.onmicrosoft.com`
    *   **Display name**: test userMFA
    *   **Password**: Auto-generated (copied for initial login).
    *   **Usage location**: United States (critical for license and policy assignment).
3.  Assign a **Microsoft Entra ID P2** license to the user (necessary for Conditional Access features).
4.  Click **Create**.

---

## 🛡️ Step 4: Configuring the Conditional Access Policy
Rather than relying on legacy per-user MFA (which is static and disruptive), we implemented a modern, signal-based Conditional Access policy:

1.  Navigate to **Identity** > **Protection** > **Conditional Access** > **Policies**.
2.  Click **New policy** and name it `CA001: Enforce MFA`.
3.  Under **Assignments**:
    *   **Users or agents**: Click *Specific users included* and select our test user `testusermfa@olamc.onmicrosoft.com`.
    *   **Target resources**: Select **All resources (formerly 'All cloud apps')**.
4.  Under **Access controls** > **Grant**:
    *   Select **Grant access**.
    *   Check **Require multi-factor authentication**.
5.  Under **Enable policy**: Set to **On** (in blue).
6.  Click **Save** / **Create**.

---

## 📝 Step 5: Testing User Onboarding & Multiple Methods Enrollment
1.  Open an Incognito browser session and navigate to the security registration page (`https://mysignins.microsoft.com/security-info`).
2.  Log in as `testusermfa@olamc.onmicrosoft.com` using the temporary password.
3.  The portal prompts the user with: **"More information required. Your organization needs more information to keep your account secure."**
4.  Click **Next**. The browser redirects to the Microsoft Authenticator setup wizard:
    *   Select **Microsoft Authenticator** as the primary method.
    *   Install the **Microsoft Authenticator app** on a mobile device (in this setup, an **iPhone 12 Pro**).
    *   Scan the provided QR code using the mobile device camera.
    *   Verify the app setup by completing the push notification number-matching challenge (code **64**).
5.  After registering the Authenticator App, the wizard prompts the user to add secondary methods to prevent account lockouts:
    *   **Phone Method**: Input the cell phone number (e.g., `+1 555-0199`) and select **SMS**. Entra ID sends a verification code via text message. Typing the code registers the phone factor.
    *   **Email Method**: Under the security info dashboard, select **Add sign-in method** > **Email**. Input the personal recovery email (`testusermfa@gmail.com`). Entra ID sends a 6-digit code via email. Entering this code registers the email factor.
6.  The **Security info** dashboard (`mysignins.microsoft.com/security-info`) now displays all three factors registered simultaneously for the user `testusermfa@olamc.onmicrosoft.com`, confirming a high-availability fallback architecture. *(See `screenshots/user_security_info_all.png`)*.

---

## 🧪 Step 6: Verifying the MFA Login Flow (Primary and Fallbacks)
To verify the policy triggers and all methods are operational for the test user:

### 6.1 Primary Method Test: Microsoft Authenticator Push
1.  Open a private browser window and attempt to sign in to the Azure Portal (`https://portal.azure.com`) as `testusermfa@olamc.onmicrosoft.com`.
2.  Enter the password.
3.  The sign-in process halts and displays the **Approve sign-in request** challenge displaying the two-digit number.
4.  The test user's Authenticator app on the **iPhone 12 Pro** receives the push notification. Entering the matching number and authenticating approves the session.

### 6.2 Backup Method Test: SMS OTP Challenge
1.  Open a private browser window and sign in as the test user.
2.  At the MFA prompt, click **"Sign in another way"** or **"I can't use my Microsoft Authenticator app right now"**.
3.  Select **Text +X XXXXXXX99** from the options.
4.  The Entra ID sign-in screen displays the **SMS Verification Prompt** *(See `screenshots/sms_mfa_prompt.png`)*.
5.  An SMS containing a 6-digit verification code is delivered to the user's registered phone.
6.  Enter the 6-digit code in the browser and click **Verify**. The authentication succeeds and grants access.

### 6.3 Backup Method Test: Email OTP Challenge
1.  Open a private browser window and sign in as the test user.
2.  At the MFA prompt, select **"Sign in another way"**.
3.  Select **Email t*****@gmail.com** from the options.
4.  The Entra ID sign-in screen displays the **Email Verification Prompt** *(See `screenshots/email_mfa_prompt.png`)*.
5.  An email containing a 6-digit OTP code is sent to the user's registered fallback email address.
6.  Enter the 6-digit code in the browser and click **Verify**. The authentication succeeds and grants access.
### 6.4 Backup/Recovery Code Testing & Security Setup
1. Open a browser and navigate to the user's advanced security settings.
2. Select **Generate a new code** in the Recovery Code card.
3. The browser displays the 25-character recovery code in a redacted format for safety *(See `screenshots/recovery_code_generation.png`)*.
4. Copy the code and store it securely using the checklist detailed in the [Authentication Methods Report](./auth_methods_documentation.md#54-recovery-code-generation--storage-checklist).
5. Open a private browser window and sign in as the test user.
6. At the MFA prompt, select **"Sign in another way"** > **"Use a recovery code"**.
7. Enter the 25-character recovery code to bypass standard MFA challenges.
8. Verify that the login succeeds. Immediately log back in to generate a new recovery code, as the code used is now invalidated.
