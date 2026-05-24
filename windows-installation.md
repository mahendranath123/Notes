# Windows Setup Offline Account Bypass Guide

## Step-by-Step Summary

### 1. Open Command Prompt During Setup

During the Windows installation or OOBE (Out-of-Box Experience) screen:

* Press **Shift + F10**
* On some laptops, use:

  **Shift + Fn + F10**

This opens the Command Prompt window.

---

### 2. Disable Internet Adapters

In Command Prompt, type:

```cmd
ncpa.cpl
```

This opens **Network Connections**.

* Right-click your active network adapter
* Select **Disable**

Disable:

* Wi-Fi adapter
* Ethernet adapter (if connected)

Close the window after disabling.

---

### 3. Enable Offline Account Option

Back in Command Prompt, run:

```cmd
oobe\bypassnro
```

The system will automatically restart.

This command enables the hidden offline setup option in Windows setup.

---

### 4. Create Local User Account

After restart:

* Continue Windows setup
* Choose:

```text
I don't have internet
```

Then select:

```text
Continue with limited setup
```

Now create:

* Local username
* Password (optional)

Finish setup normally.

---

## Notes

* Works on most Windows 11 installations.
* Useful when Microsoft account login is forced.
* Internet can be re-enabled after setup completes.

---

## Quick Commands Reference

```cmd
Shift + F10
ncpa.cpl
oobe\bypassnro
```
