# Microsoft Intune — Hands-On Learning Notes

> Part of my **IT Learning Journal** — documenting hands-on lab practice and self-study as I build skills toward a DevOps/Cloud/Linux/Security-adjacent role.
>
> Everything here reflects **personal lab practice** — using a free Microsoft 365 developer tenant to explore Intune — not professional/employer experience. I'm noting that clearly so it's never misread as job history.
>
> ⚠️ **Please edit the specific exercises below to match exactly what you did.**

---

## 📌 What is Microsoft Intune?

**Microsoft Intune** is a cloud-based **endpoint management (MDM/MAM)** platform that lets organizations manage and secure devices — Windows PCs, phones, tablets — without needing them to be physically connected to a corporate network. It's part of Microsoft's broader "Microsoft Endpoint Manager" ecosystem, alongside Configuration Manager (SCCM).

In simple terms: Intune lets IT set rules for company devices — like requiring a passcode, enforcing encryption, or installing required apps — and enforce those rules automatically, from anywhere, over the internet.

---

## 🏢 Why Companies Use Intune

| Reason | Explanation |
|--------|-------------|
| **Remote device management** | Manage devices anywhere, without needing them on the corporate network |
| **BYOD support** | Securely manage company data on personal ("Bring Your Own Device") phones/laptops |
| **Compliance enforcement** | Ensures devices meet security requirements (encryption, passcode, OS version) before accessing company resources |
| **App deployment** | Push required apps to devices automatically |
| **Conditional Access integration** | Blocks access to company email/apps from non-compliant devices |
| **Cloud-native, no on-prem servers needed** | Unlike SCCM, doesn't require on-premises infrastructure |

This matters because with more remote/hybrid work and employees using personal devices, companies need a way to enforce security policies without physical access to every device — Intune solves that by managing everything through the cloud.

---

## ⚙️ How It Works (Simple Version)

1. A device is **enrolled** into Intune (company-owned devices get full management; personal devices often get lighter "app protection" management).
2. Administrators create **configuration profiles** (settings) and **compliance policies** (requirements) in the Intune portal.
3. These policies are pushed down to enrolled devices automatically.
4. Intune continuously checks whether devices remain compliant, and can restrict access to company resources if they fall out of compliance.

---

## 🏗️ Basic Architecture

```
           Administrator (Me)
                   │
                   ▼
        Microsoft Intune Admin Center
                   │
                   ▼
        Intune Cloud Service
     (Policies, Profiles, App Deployment)
                   │
     ┌─────────────┼─────────────┐
     ▼             ▼             ▼
 Windows PC     Mobile Phone   Tablet
 (Enrolled)     (Enrolled)   (Enrolled)
                   │
                   ▼
     Conditional Access (via Entra ID)
   (Blocks access if device is non-compliant)
```

**Explanation of each part:**
- **Intune Admin Center** — the web portal used to configure policies and monitor devices.
- **Intune Cloud Service** — where policies, profiles, and app assignments actually live and get pushed from.
- **Enrolled Devices** — Windows, iOS, Android, or macOS devices registered with Intune.
- **Conditional Access** — works together with Microsoft Entra ID to block sign-in/access from devices that don't meet compliance requirements.

---

## 💻 Hands-On Practice (What I Did)

- Signed up for a **free Microsoft 365 Developer tenant** (which includes Intune licensing for lab/learning purposes)
- Enrolled a **Windows 10/11 VM** into Intune using the "Windows Enrollment" method
- Created a basic **device configuration profile** (e.g., requiring a PIN, setting a lock screen timeout)
- Created a **compliance policy** (e.g., requiring BitLocker encryption and a minimum OS version)
- Reviewed how a device shows as "Compliant" or "Not Compliant" in the Intune console after policy evaluation
- Explored **app deployment** by assigning a simple app to the test device group
- Reviewed how **Conditional Access** (via Microsoft Entra ID) can be configured to block access to company resources from non-compliant devices (conceptual review, since this required tenant-level Conditional Access setup)

*(Adjust this list to match exactly what you tested in your own developer tenant.)*

---

## 🔧 Core Concepts I Studied

| Concept | What It Means |
|---------|----------------|
| **Enrollment** | The process of registering a device with Intune so it can be managed |
| **Configuration Profiles** | Settings pushed to devices (e.g., Wi-Fi settings, PIN requirements, restrictions) |
| **Compliance Policies** | Rules that define what makes a device "compliant" (e.g., encryption enabled, minimum OS version) |
| **App Deployment** | Assigning and pushing required or available apps to enrolled devices |
| **Conditional Access** | Entra ID feature that restricts access to company resources based on device compliance/identity signals |
| **MDM vs. MAM** | MDM (Mobile Device Management) manages the whole device; MAM (Mobile Application Management) manages just company apps/data, often used for personal (BYOD) devices |
| **Device Groups** | Collections of devices used to target policies and app assignments |

---

## ⚠️ Common Configuration Mistakes (Things I Ran Into / Learned About)

| Problem | Possible Cause | How to Fix | Verification |
|---------|-----------------|------------|----------------|
| Device stuck in "Pending" enrollment | Enrollment restrictions not configured to allow the device/user | Review enrollment restriction settings in Intune | Device shows "Enrolled"/"Active" status |
| Policy not applying to device | Device not in the correct assigned group, or sync hasn't occurred yet | Confirm group assignment; manually trigger device sync | Policy shows "Succeeded" under device policy status |
| Device shows "Not Compliant" unexpectedly | Device genuinely doesn't meet a requirement (e.g., encryption not enabled) | Check specific failed compliance rule in the console | Device status changes to "Compliant" after issue is fixed and re-synced |
| App deployment fails | App not assigned to the correct group, or device platform mismatch | Verify app assignment group and supported platform | App shows "Installed" status on the device |
| Conditional Access blocking legitimate access unexpectedly | Policy too broad, or device incorrectly marked non-compliant | Review Conditional Access policy scope and device compliance status | User can access resource after issue resolved |

---

## 🔄 Typical Device Management Workflow

```
Device Enrollment
        ↓
Device Added to Group
        ↓
Configuration Profile Assigned
        ↓
Compliance Policy Assigned
        ↓
Device Checks In and Applies Settings
        ↓
Compliance Status Evaluated
        ↓
(If Compliant) Access to Company Resources Allowed
        ↓
(If Not Compliant) Conditional Access Blocks/Restricts Access
```

---

## ✅ Best Practices (What I Learned)

**Start with Pilot Groups**
- Test configuration profiles and compliance policies on a small pilot device group before rolling out organization-wide.

**Keep Policies Clear and Minimal**
- Overly complex or conflicting profiles can cause unpredictable device behavior — simpler, well-documented policies are easier to troubleshoot.

**Pair Compliance with Conditional Access**
- Compliance policies are most effective when tied to Conditional Access — otherwise "non-compliant" is just a label with no real enforcement.

**Understand MDM vs. MAM**
- Use full MDM for company-owned devices, and lighter MAM (app protection) for personal BYOD devices, to avoid over-managing employees' personal phones.

**Monitor Regularly**
- Check the compliance dashboard regularly rather than assuming policies are working correctly after initial setup.

---

## 🎤 Interview Questions (Beginner Level)

**1. What is Microsoft Intune?**
A cloud-based endpoint management platform used to manage and secure devices like Windows PCs, phones, and tablets, without needing on-premises infrastructure.

**2. What's the difference between Intune and SCCM?**
Intune is cloud-native and manages devices over the internet; SCCM is traditionally on-premises and often used for devices within a corporate network. Many organizations use both together (co-management).

**3. What is device enrollment?**
The process of registering a device with Intune so it can receive policies and be managed.

**4. What is a compliance policy?**
A set of rules that define what makes a device "compliant" — for example, requiring encryption or a minimum OS version.

**5. What is a configuration profile?**
A set of settings pushed to a device, such as PIN requirements, Wi-Fi configuration, or restrictions.

**6. What is Conditional Access, and how does it relate to Intune?**
A Microsoft Entra ID feature that can block or allow access to company resources based on signals like device compliance status reported by Intune.

**7. What's the difference between MDM and MAM?**
MDM (Mobile Device Management) manages the entire device; MAM (Mobile Application Management) manages only company apps/data, commonly used for personal BYOD devices.

**8. Why would a company use MAM instead of MDM for personal devices?**
To protect company data within specific apps without taking full control over an employee's personal device.

**9. What might cause a device to show as "Not Compliant"?**
It's genuinely failing a required policy check — for example, missing encryption, an outdated OS version, or a required security setting not enabled.

**10. What's a device group used for in Intune?**
Grouping devices together so policies and app deployments can be targeted to specific sets of devices rather than applied one-by-one.

**11. What would you check if a policy isn't applying to a device?**
Whether the device is in the correct assigned group and whether it has recently synced with Intune.

**12. Why is it recommended to test policies on a pilot group first?**
To catch configuration issues on a small number of devices before they affect the whole organization.

**13. What happens if Conditional Access is configured but compliance policies aren't?**
There's no real basis to evaluate "compliant vs. non-compliant," so Conditional Access can't meaningfully enforce device security requirements.

**14. Can Intune deploy applications to devices?**
Yes — administrators can assign required or optional apps to device or user groups for automatic installation.

**15. What is co-management in the context of Intune and SCCM?**
Managing the same devices with both SCCM and Intune simultaneously, often used during a gradual migration from on-premises to cloud-based management.

**16. Why might a personal (BYOD) device not be a good fit for full MDM enrollment?**
Full MDM gives the organization significant control over the entire device, which can raise privacy concerns for employees on personal devices — MAM is often a better fit.

**17. What's one benefit of Intune being cloud-based compared to traditional on-prem management tools?**
Devices can be managed and secured anywhere with an internet connection, without needing to be on the corporate network.

**18. What role does Microsoft Entra ID play alongside Intune?**
It handles identity and authentication, and works with Intune (via Conditional Access) to control access to resources based on both user identity and device compliance.

**19. What's a practical first step if a user reports they can't access company email from their phone?**
Check the device's enrollment and compliance status in Intune, and review any Conditional Access policy that might be blocking access based on that status.

**20. Why is monitoring the compliance dashboard an ongoing task, not a one-time setup step?**
Device compliance can change over time (updates missed, settings changed), so ongoing monitoring ensures policies are actually being enforced, not just configured once and forgotten.

---

## 📝 My Learning Summary

Working hands-on with Intune in a free developer tenant gave me a practical understanding of cloud-based device management — enrolling a device, building configuration and compliance policies, and seeing how compliance status directly ties into access control through Conditional Access.

It also helped me understand the reasoning behind MDM vs. MAM, and why companies don't manage every device the same way — a company-owned laptop and an employee's personal phone need very different levels of control.

This reflects self-directed lab practice in a personal developer tenant, not professional job experience managing Intune across a real company's device fleet.
