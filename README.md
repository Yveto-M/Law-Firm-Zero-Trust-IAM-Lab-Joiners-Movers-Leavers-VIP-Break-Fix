# Law-Firm Zero Trust IAM Lab: Joiners, Movers, Leavers & VIP Break-Fix

This project simulates a mid-size enterprise law firm implementing Zero Trust architecture for **Joiner-Mover-Leaver (JML)** lifecycles, Information Barriers (Ethical Walls), and a traveling VIP partner MFA break-fix scenario. 

The environment is built on hybrid identity infrastructure utilizing Active Directory, a Windows file server, Microsoft Entra ID, Conditional Access, and Temporary Access Pass (TAP).

## Architecture Overview
* **On-Premise:** AD domain `UVBANK.com` with departmental OUs (Finance, Legal, HR, IT, Mergers & Acquisitions).
* **Storage:** Windows file server hosting departmental shares (`\\UVBANK-FS1\Finance`, `\\UVBANK-FS1\Legal`, `\\UVBANK-FS1\Mergers`) governed by strict NTFS/Share permissions.
* **Cloud Control Plane:** Microsoft Entra ID sync with P2 licensing enforcing Conditional Access and TAP.
* **Testing:** Dedicated lab "partner" and "associate" accounts used to validate location-based policies, RBAC, and MFA recovery.

**[Architecture overview]**<img width="808" height="377" alt="Screenshot 2026-03-05 171029" src="https://github.com/user-attachments/assets/3dd82b91-39fe-4fe4-9464-75bf5310a3e3" />



---

## Scenario 1: The Joiner (New M&A Associate)
**Goal:** Demonstrate automated provisioning where a new hire lands in the correct OU, inherits role-based groups, and receives strictly scoped least-privilege access on Day One.

### Execution & Validation
1. Created an AD security group `M&A_Associates` under the Departments OU to represent the specific legal role.
![Create M&A role group]<img width="422" height="335" alt="creating_group-3" src="https://github.com/user-attachments/assets/0e10bc01-3177-4625-ac97-5103e4032f58" />


2. Onboarded **Emma Williams** into the `Mergers & Acquisitions` OU and added her to the `M&A_Associates` group, ensuring access is driven dynamically by role, not individual user assignment.
[Emma in M&A OU]<img width="643" height="109" alt="M A-joiner-1" src="https://github.com/user-attachments/assets/2cdd44de-e511-4464-8ad9-a6aeb3e0b3fb" />

[Emma added to M&A_Associates]<img width="395" height="206" alt="MA-group-members-4" src="https://github.com/user-attachments/assets/96641950-cd61-41a2-8f7e-70530cfd0386" />


3. Validated the joiner experience by signing in as Emma and creating a deal folder (`E_williams_london_notes`) in the M&A share, proving she can work exclusively within her practice group's perimeter.
[Emma signs in]<img width="443" height="480" alt="willinams_login-5" src="https://github.com/user-attachments/assets/972cc4ab-f051-49c8-9bf0-714b0c7c726b" />

[Emma creates London notes folder]<img width="671" height="269" alt="williams_folder_edit-6" src="https://github.com/user-attachments/assets/cc92481d-7f9c-4376-afee-5a74e3f98ae1" />



> **The Business Value:** A new M&A associate is created in the correct OU, drops into the `M&A_Associates` role group, and on their first login can immediately work in the M&A share without having broad, unauthorized access to Finance or Legal content.

---

## Scenario 2: The Mover (Enforcing Ethical Walls)
**Goal:** Prove that when a user changes departments, legacy access is systematically revoked to prevent permission creep and Separation of Duties (SoD) violations.

### Execution & Validation
1. Configured file server share/NTFS permissions so `Finance_Share_RW` controls the Finance share and `Legal_Share_RW` controls the Legal share.
[Finance share mapped to Finance_Share_RW]<img width="332" height="210" alt="finanace-share-4" src="https://github.com/user-attachments/assets/6e534ba1-38c9-4636-b28b-28a57a9f0a18" />

[Legal share mapped to Legal_Share_RW]<img width="340" height="177" alt="Legal_Share-5" src="https://github.com/user-attachments/assets/801d84f3-9ec0-4aa8-8a44-302c8a510cf7" />


2. **Baseline:** Finance analyst **Akira Mori** is placed in the `Finance` OU and added to `Finance_Share_RW`. He successfully creates work in the Finance share.
[Akira signs in as Finance user]<img width="571" height="129" alt="akira_fi_user_login-6" src="https://github.com/user-attachments/assets/c68f64b5-2ed9-489b-ae1e-c0dd524507b4" />

[Akira in Finance_Share_RW]<img width="395" height="176" alt="fi-user-to-fiShares-1" src="https://github.com/user-attachments/assets/fe6161f0-7c01-4276-8f58-d7b68a78d686" />


3. **The Move:** Simulated a department transfer by moving Akira's account from the `Finance` OU to the `Legal` OU. Updated group memberships: Akira is removed from `Finance_Share_RW` and added to `Legal_Share_RW`.
[Move Akira to Legal OU]<img width="319" height="330" alt="Akira-move-to-legal-8" src="https://github.com/user-attachments/assets/62908d89-ee1b-48f0-8720-f89f6d0fedf6" />

[Akira now member of Legal_Share_RW only]<img width="408" height="216" alt="Akira-move-legal-memeber-9" src="https://github.com/user-attachments/assets/0945a629-22e4-49ad-a8d8-0d5a48d47658" />


4. **Validation:** Re-tested from Akira’s session. He can now access the Legal share, but is explicitly **denied** access to the Finance share.
[Legal share accessible for Akira]<img width="697" height="303" alt="bo-access-to-legal-for-fi-users-7" src="https://github.com/user-attachments/assets/41d4324d-b575-42f1-bf92-185ad56342be" />

[Finance share now denied for Akira]<img width="537" height="254" alt="akira-denied-finanace-10" src="https://github.com/user-attachments/assets/ef770afb-067d-4b8e-997b-e4375b08f34d" />



> **The Business Value:** When moving a user from Finance to Legal, swapping their role groups ensures they lose legacy access and gain new access cleanly. This actively prevents permission creep and enforces the firm's Ethical Walls.

---

## Scenario 3: VIP Break-Fix (Conditional Access & TAP)
**Goal:** Enforce a location-aware Conditional Access policy for a high-risk VIP, and safely execute MFA recovery when they lose their device abroad without weakening the global security posture.

### Execution & Validation
1. Assigned an Entra ID P2 license to the partner account to enable advanced Conditional Access and TAP.
[Partner has Entra ID P2 license]<img width="717" height="258" alt="user-p2-l-1" src="https://github.com/user-attachments/assets/9ccb6c0f-3883-4d4a-939d-88f0e779b1df" />


2. Created a **Block Non-US Sign-in** Conditional Access policy targeting the partner account, utilizing IP-based named locations.
[Location-based Conditional Access policy]<img width="652" height="131" alt="location-based-CA-2" src="https://github.com/user-attachments/assets/df43e030-bdda-4ffb-bdce-f72b7e5a296b" />


3. **The Block:** Connected the lab client to a Switzerland VPN. The sign-in succeeds at the credential layer but is immediately blocked by Conditional Access due to the geographic violation.
[VPN connected outside US]<img width="989" height="409" alt="vpn-use-3" src="https://github.com/user-attachments/assets/641676ce-88d6-4fca-8bf2-9440eeedfd1d" />

[Partner blocked outside US]<img width="435" height="403" alt="user-outside-US-blocked-4" src="https://github.com/user-attachments/assets/b58de2a7-ef44-40c7-aa61-b8138f705f45" />


4. **The Recovery (TAP):** To simulate a "lost phone while traveling" emergency, issued a short-lived **Temporary Access Pass** to the partner. 
[Create Temporary Access Pass]<img width="919" height="276" alt="creating-TAP" src="https://github.com/user-attachments/assets/2be677a3-958a-4f99-83ca-8485f1861cf0" />



5. Used the TAP to securely bypass the block, reach the Entra `Security info` page, and register a new Microsoft Authenticator token. Once complete, the TAP was deleted and the strict Conditional Access policy resumed normal enforcement.
![TAP sign-in to Security info]<img width="649" height="440" alt="tap-signi-in" src="https://github.com/user-attachments/assets/acb7fdba-e9e1-4d4b-beb4-ca58050ebab4" />

![Add Microsoft Authenticator using TAP]<img width="370" height="445" alt="user-require-mfa-for-successful-log-5" src="https://github.com/user-attachments/assets/5e5d91d1-b802-433b-a38b-2b8f8002b6ff" />


> **The Business Value:** A VIP account is protected by strict geo-blocking. During a lost-device scenario, issuing a time-bound Temporary Access Pass allows the partner to re-enroll securely. This recovers the VIP's productivity immediately without opening a permanent backdoor or weakening Conditional Access for the rest of the firm.
