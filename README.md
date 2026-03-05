# Law-Firm Zero Trust IAM Lab: Joiners, Movers, Leavers & VIP Break-Fix

This project simulates a mid-size enterprise law firm implementing Zero Trust architecture for **Joiner-Mover-Leaver (JML)** lifecycles, Information Barriers (Ethical Walls), and a traveling VIP partner MFA break-fix scenario. 

The environment is built on hybrid identity infrastructure utilizing Active Directory, a Windows file server, Microsoft Entra ID, Conditional Access, and Temporary Access Pass (TAP).

## Architecture Overview
* **On-Premise:** AD domain `UVBANK.com` with departmental OUs (Finance, Legal, HR, IT, Mergers & Acquisitions).
* **Storage:** Windows file server hosting departmental shares (`\\UVBANK-FS1\Finance`, `\\UVBANK-FS1\Legal`, `\\UVBANK-FS1\Mergers`) governed by strict NTFS/Share permissions.
* **Cloud Control Plane:** Microsoft Entra ID sync with P2 licensing enforcing Conditional Access and TAP.
* **Testing:** Dedicated lab "partner" and "associate" accounts used to validate location-based policies, RBAC, and MFA recovery.

![Architecture overview](./images/architecture-overview.png)


---

## Scenario 1: The Joiner (New M&A Associate)
**Goal:** Demonstrate automated provisioning where a new hire lands in the correct OU, inherits role-based groups, and receives strictly scoped least-privilege access on Day One.

### Execution & Validation
1. Created an AD security group `M&A_Associates` under the Departments OU to represent the specific legal role.
![Create M&A role group](./images/creating_group-3.jpg)

2. Onboarded **Emma Williams** into the `Mergers & Acquisitions` OU and added her to the `M&A_Associates` group, ensuring access is driven dynamically by role, not individual user assignment.
![Emma in M&A OU](./images/M-A-joiner-1-3.jpg)
![Emma added to M&A_Associates](./images/MA-group-members-4-4.jpg)

3. Validated the joiner experience by signing in as Emma and creating a deal folder (`E_williams_london_notes`) in the M&A share, proving she can work exclusively within her practice group's perimeter.
![Emma signs in](./images/willinams_login-5-7.jpg)
![Emma creates London notes folder](./images/williams_folder_edit-6-6.jpg)


> **The Business Value:** A new M&A associate is created in the correct OU, drops into the `M&A_Associates` role group, and on their first login can immediately work in the M&A share without having broad, unauthorized access to Finance or Legal content.

---

## Scenario 2: The Mover (Enforcing Ethical Walls)
**Goal:** Prove that when a user changes departments, legacy access is systematically revoked to prevent permission creep and Separation of Duties (SoD) violations.

### Execution & Validation
1. Configured file server share/NTFS permissions so `Finance_Share_RW` controls the Finance share and `Legal_Share_RW` controls the Legal share.
![Finance share mapped to Finance_Share_RW](./images/finanace-share-4-6.jpg)
![Legal share mapped to Legal_Share_RW](./images/Legal_Share-5-9.jpg)

2. **Baseline:** Finance analyst **Akira Mori** is placed in the `Finance` OU and added to `Finance_Share_RW`. He successfully creates work in the Finance share.
![Akira signs in as Finance user](./images/akira_fi_user_login-6.jpg)
![Akira in Finance_Share_RW](./images/fi-user-to-fiShares-1-7.jpg)

3. **The Move:** Simulated a department transfer by moving Akira's account from the `Finance` OU to the `Legal` OU. Updated group memberships: Akira is removed from `Finance_Share_RW` and added to `Legal_Share_RW`.
![Move Akira to Legal OU](./images/Akira-move-to-legal-8-4.jpg)
![Akira now member of Legal_Share_RW only](./images/Akira-move-legal-memeber-9-3.jpg)

4. **Validation:** Re-tested from Akira’s session. He can now access the Legal share, but is explicitly **denied** access to the Finance share.
![Legal share accessible for Akira](./images/bo-access-to-legal-for-fi-users-7-5.jpg)
![Finance share now denied for Akira](./images/akira-denied-finanace-10-2.jpg)


> **The Business Value:** When moving a user from Finance to Legal, swapping their role groups ensures they lose legacy access and gain new access cleanly. This actively prevents permission creep and enforces the firm's Ethical Walls.

---

## Scenario 3: VIP Break-Fix (Conditional Access & TAP)
**Goal:** Enforce a location-aware Conditional Access policy for a high-risk VIP, and safely execute MFA recovery when they lose their device abroad without weakening the global security posture.

### Execution & Validation
1. Assigned an Entra ID P2 license to the partner account to enable advanced Conditional Access and TAP.
![Partner has Entra ID P2 license](./images/user-p2-l-1-5.jpg)

2. Created a **Block Non-US Sign-in** Conditional Access policy targeting the partner account, utilizing IP-based named locations.
![Location-based Conditional Access policy](./images/location-based-CA-2-2.jpg)

3. **The Block:** Connected the lab client to a Switzerland VPN. The sign-in succeeds at the credential layer but is immediately blocked by Conditional Access due to the geographic violation.
![VPN connected outside US](./images/vpn-use-3-7.jpg)
![Partner blocked outside US](./images/user-outside-US-blocked-4-4.jpg)

4. **The Recovery (TAP):** To simulate a "lost phone while traveling" emergency, issued a short-lived **Temporary Access Pass** to the partner. 
![Create Temporary Access Pass](./images/creating-TAP.jpg)


5. Used the TAP to securely bypass the block, reach the Entra `Security info` page, and register a new Microsoft Authenticator token. Once complete, the TAP was deleted and the strict Conditional Access policy resumed normal enforcement.
![TAP sign-in to Security info](./images/tap-signi-in-3.jpg)
![Add Microsoft Authenticator using TAP](./images/user-require-mfa-for-successful-log-5-6.jpg)

> **The Business Value:** A VIP account is protected by strict geo-blocking. During a lost-device scenario, issuing a time-bound Temporary Access Pass allows the partner to re-enroll securely. This recovers the VIP's productivity immediately without opening a permanent backdoor or weakening Conditional Access for the rest of the firm.
