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

## 📝 Step 5: Testing User Onboarding & Enrollment
1.  Open an Incognito browser session and navigate to the security registration page (`https://mysignins.microsoft.com/security-info`).
2.  Log in as `testusermfa@olamc.onmicrosoft.com` using the temporary password.
3.  The portal prompts the user with: **"More information required. Your organization needs more information to keep your account secure."**
4.  Click **Next**. The browser redirects to the Microsoft Authenticator setup wizard:
    *   Select **Microsoft Authenticator** as the primary method.
    *   Install the **Microsoft Authenticator app** on a mobile device (in this setup, an **iPhone 12 Pro**).
    *   Scan the provided QR code using the mobile device camera.
5.  To verify the integration, the browser shows a verification screen titled **"Let's try it out"**, displaying a verification code (e.g., **64**).
6.  A push notification arrives on the user's **iPhone 12 Pro**. Entering **64** in the mobile app registers the method.
7.  The registered factors are successfully saved under the user's **Security info** dashboard.

---

## 🧪 Step 6: Verifying the MFA Login Flow
To verify the policy triggers correctly:

1.  Open a new private browser window and attempt to sign in to the Azure Portal (`https://portal.azure.com`) as `testusermfa@olamc.onmicrosoft.com`.
2.  Enter the user password.
3.  The sign-in process halts and displays the **Approve sign-in request** challenge displaying the two-digit number.
4.  The test user's Authenticator app on the **iPhone 12 Pro** receives the push notification. Entering the matching number and authenticating approves the session.
5.  The login succeeds, redirecting the user to the Azure Portal home dashboard.
