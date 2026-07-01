# BitLocker — Hands-On Learning Notes

> Part of my **IT Learning Journal** — documenting hands-on lab practice and self-study as I build skills toward a DevOps/Cloud/Linux/Security-adjacent role.
>
> Everything here reflects **personal lab practice** — testing BitLocker on a Windows virtual machine — not professional/employer experience. I'm noting that clearly so it's never misread as job history.
>
> ⚠️ **Please edit the specific exercises below to match exactly what you did.**

---

## 📌 What is BitLocker?

**BitLocker** is a **disk encryption feature built into Windows** (Pro, Enterprise, and Education editions) that protects data by encrypting an entire drive. If a laptop is lost or stolen, BitLocker ensures that whoever has the physical device still can't read the data without the correct recovery key or password.

In simple terms: it's like putting the entire hard drive in a locked safe — even if someone removes the drive and plugs it into another computer, the data is unreadable without the key.

---

## 🏢 Why Companies Use BitLocker

| Reason | Explanation |
|--------|-------------|
| **Data protection on lost/stolen devices** | Encrypted drives can't be read even if physically removed |
| **Compliance requirements** | Many regulations (HIPAA, GDPR, PCI DSS) expect data-at-rest encryption |
| **Free with Windows Pro/Enterprise** | No extra licensing cost for basic full-disk encryption |
| **Centralized management** | Can be managed at scale via Intune or Active Directory Group Policy |
| **Protects against offline attacks** | Prevents someone from bypassing Windows login by booting from another OS to read the disk directly |

This matters because a stolen unencrypted laptop can expose company or customer data instantly — BitLocker turns that same stolen laptop into a device the thief can't extract any meaningful data from.

---

## ⚙️ How It Works (Simple Version)

1. BitLocker encrypts the entire drive using strong encryption (AES).
2. A **TPM (Trusted Platform Module)** chip, if present, stores the encryption key securely and verifies the system hasn't been tampered with before releasing it.
3. When the device boots normally, BitLocker unlocks automatically (with TPM) or after PIN/password entry.
4. If someone tries to access the drive outside of the normal boot process (e.g., removing the disk), the data remains encrypted and inaccessible without the recovery key.

---

## 🏗️ Basic Architecture / Flow

```
           Windows Device
                 │
                 ▼
       BitLocker Encryption Engine
                 │
        ┌────────┴────────┐
        ▼                 ▼
   TPM Chip           Recovery Key
 (stores/protects      (backup unlock
   the key)              method)
                 │
                 ▼
         Encrypted Drive
   (unreadable without key/TPM
       validation on boot)
```

**Explanation of each part:**
- **TPM (Trusted Platform Module)** — a small hardware chip that securely stores the encryption key and checks system integrity at boot.
- **Recovery Key** — a 48-digit backup key generated during setup, used if TPM validation fails or hardware changes are detected.
- **Encrypted Drive** — the actual disk, fully encrypted so its contents are unreadable without proper unlock.

---

## 💻 Hands-On Practice (What I Did)

- Enabled BitLocker on a **Windows 10/11 VM** with a virtual TPM enabled in the hypervisor settings
- Walked through the setup wizard: choosing to save the recovery key (saved it to a file for lab purposes, since there was no Microsoft/AD account tied to the lab VM)
- Confirmed drive encryption status using both the **Control Panel BitLocker interface** and the command line
- Practiced retrieving and using the **recovery key** to unlock the drive after simulating a TPM validation failure (by changing virtual hardware settings)
- Tested suspending and resuming BitLocker protection (useful before BIOS/firmware updates in real environments)
- Reviewed how BitLocker status and recovery keys can be centrally managed and reported through **Microsoft Intune** in an enterprise setting (conceptual review, not a live managed deployment)

*(Adjust this list to match exactly what you tested in your own lab.)*

---

## 🔧 Core Concepts I Studied

| Concept | What It Means |
|---------|----------------|
| **TPM (Trusted Platform Module)** | Hardware chip that securely stores encryption keys and verifies system integrity |
| **Recovery Key** | 48-digit backup key used to unlock the drive if normal unlock fails |
| **Encryption Modes** | AES-128 or AES-256, with XTS or CBC cipher modes depending on Windows version |
| **BitLocker To Go** | BitLocker protection extended to removable USB drives |
| **Suspend vs. Decrypt** | Suspending pauses protection temporarily (e.g., for updates) without fully decrypting the drive |
| **Group Policy / Intune Management** | Centralized ways to enforce and monitor BitLocker across many company devices |

---

## ⚠️ Common Configuration Mistakes (Things I Ran Into / Learned About)

| Problem | Possible Cause | How to Fix | Verification |
|---------|-----------------|------------|----------------|
| BitLocker won't enable | No TPM present or TPM disabled in BIOS/hypervisor | Enable virtual/physical TPM, or allow BitLocker without TPM via Group Policy (less secure, not recommended for production) | BitLocker setup wizard proceeds without error |
| Lost recovery key | Key wasn't saved/backed up during setup | Recover from Microsoft account, AD, or Intune if it was backed up there; otherwise data may be unrecoverable | Successfully unlock drive using retrieved key |
| Drive stuck asking for recovery key on every boot | Hardware/firmware change detected (e.g., BIOS update) TPM sees this as tampering | Suspend BitLocker before planned hardware/firmware changes | Device boots normally without prompting after resuming protection |
| Encryption taking a very long time | Full disk encryption of a large drive takes significant time, especially on slower hardware | Allow it to complete in the background; avoid interrupting the process | Status shows "100% encrypted" in BitLocker settings |
| Recovery key not backed up to AD/Intune in a managed environment | Group Policy/Intune policy not properly configured to require key backup | Configure policy to mandate automatic recovery key escrow | Key appears in AD/Intune console for the device |

---

## 🔄 Typical Setup Workflow

```
Confirm TPM is present/enabled
             ↓
Enable BitLocker on the drive
             ↓
Choose unlock method (TPM only / TPM + PIN / password)
             ↓
Save Recovery Key (file, print, AD, or Microsoft/Intune account)
             ↓
Choose encryption mode (new drive: used space only vs. full drive)
             ↓
Encryption Process Runs
             ↓
Verify Status (Control Panel or command line)
             ↓
(Enterprise) Confirm Recovery Key Escrowed to AD/Intune
```

---

## ✅ Best Practices (What I Learned)

**Recovery Key Management**
- Always back up the recovery key somewhere separate from the encrypted device itself — a key stored only on the encrypted drive is useless if that drive won't boot.

**Enterprise Deployment**
- Use Group Policy or Intune to enforce automatic recovery key backup to Active Directory or Microsoft Entra ID — relying on individual users to save their own key is risky at scale.

**Before Hardware/Firmware Changes**
- Suspend BitLocker before BIOS/firmware updates to avoid unnecessary recovery key prompts caused by TPM detecting a "change."

**TPM Requirement**
- Prefer TPM-backed BitLocker over TPM-less configurations — TPM provides much stronger protection against offline attacks.

**Testing**
- Test the recovery process at least once (in a lab, not production!) so you understand what a locked-out user will actually experience.

---

## 🎤 Interview Questions (Beginner Level)

**1. What is BitLocker?**
A Windows feature that encrypts an entire drive so its data can't be read without the correct key, even if the physical drive is removed.

**2. What is a TPM and why does BitLocker use it?**
A Trusted Platform Module is a hardware chip that securely stores the encryption key and verifies the system hasn't been tampered with before unlocking the drive.

**3. What is a BitLocker recovery key?**
A 48-digit backup key used to unlock a BitLocker-protected drive if the normal automatic/TPM unlock fails.

**4. What Windows editions support BitLocker?**
Windows Pro, Enterprise, and Education editions.

**5. What happens if you lose your BitLocker recovery key and can't unlock the drive?**
The data on the drive is effectively inaccessible — this is why backing up the recovery key is critical.

**6. What is BitLocker To Go?**
BitLocker protection extended to removable drives, such as USB flash drives.

**7. Why might a BitLocker-protected device suddenly prompt for the recovery key at boot?**
The TPM detected a change in the boot process or hardware/firmware (e.g., a BIOS update), which it treats as a potential tampering event.

**8. How can you avoid unnecessary recovery key prompts before a planned firmware update?**
Suspend BitLocker protection before the update, then resume it afterward.

**9. What encryption standard does BitLocker use?**
AES, typically in 128-bit or 256-bit strength, depending on configuration.

**10. Where can a company centrally store/manage BitLocker recovery keys for employee devices?**
In Active Directory (for domain-joined devices) or Microsoft Entra ID / Intune (for cloud-managed devices).

**11. Why is data-at-rest encryption important for compliance?**
Many regulations (HIPAA, GDPR, PCI DSS) require organizations to protect stored sensitive data, and disk encryption is a standard control for meeting that requirement.

**12. What's the difference between encrypting "used space only" vs. the "entire drive"?**
Used space only encrypts just the data currently on the drive (faster, good for new drives); entire drive encryption also encrypts empty space (slower, more thorough, better for drives that previously held unencrypted data).

**13. Can BitLocker protect against someone guessing your Windows login password?**
Not directly — BitLocker protects against offline attacks (removing the disk); a weak Windows password is a separate risk that BitLocker alone doesn't solve.

**14. Why would an organization enforce PIN + TPM instead of TPM-only unlock?**
Adding a PIN provides an extra authentication factor, protecting against certain attacks where TPM alone could be bypassed on physical access.

**15. What tool can you use on the command line to check BitLocker status?**
The `manage-bde -status` command.

**16. Why is BitLocker considered a "defense against offline attacks"?**
Because even if someone removes the drive and tries to read it on another machine, the data remains encrypted and unreadable without the key.

**17. What should happen to the recovery key in a properly managed enterprise environment?**
It should be automatically backed up/escrowed to Active Directory or Intune, not left solely with the end user.

**18. What's a real-world scenario where BitLocker recovery would be needed?**
A laptop's motherboard is replaced for repair — the hardware change can trigger BitLocker to require the recovery key on next boot.

**19. Does BitLocker protect data if a laptop is stolen while powered on and unlocked?**
No — BitLocker protects data at rest against offline access; if the device is already unlocked and running, other security controls (screen lock, account security) are what matter at that point.

**20. What's one advantage of BitLocker being built into Windows rather than a third-party tool?**
No additional licensing cost for basic encryption on supported Windows editions, and native integration with Windows and Microsoft management tools like Intune.

---

## 📝 My Learning Summary

Practicing with BitLocker in a lab VM helped me understand disk encryption beyond the "it's just a checkbox" level. Walking through TPM-based encryption, generating and testing a recovery key, and simulating a TPM validation failure gave me a much clearer picture of what actually happens — and what a user experiences — when a device asks for a recovery key.

Understanding the relationship between TPM, recovery keys, and centralized management (via AD or Intune) also helped me see why relying on individual users to safely store their own recovery key isn't a scalable or safe approach for a real organization.

This reflects self-directed lab practice, not professional job experience managing BitLocker across a fleet of company devices.
