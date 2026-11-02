# Cisco Packet Tracer on Arch Linux

A comprehensive, step-by-step installation and troubleshooting guide for running **Cisco Packet Tracer 9.0.1** on **Arch Linux** using the AUR package and AppImage, including the critical fix for KDE Plasma / Qt browser authentication failures.

---

## 📑 Table of Contents

- [1. Tested Environment](#1-tested-environment)
- [2. Downloading Cisco Packet Tracer](#2-downloading-cisco-packet-tracer)
- [3. Installing Prerequisites & Building from AUR](#3-installing-prerequisites--building-from-aur)
- [4. Locating the Installed AppImage](#4-locating-the-installed-appimage)
- [5. Installing the AppImage in ~/Applications](#5-installing-the-appimage-in-applications)
- [6. First Launch & Authentication Behavior](#6-first-launch--authentication-behavior)
- [7. Installing Supporting Utilities](#7-installing-supporting-utilities)
- [8. Configuring Firefox as Default Browser](#8-configuring-firefox-as-default-browser)
- [9. Fixing the KDE and Qt Compatibility Conflict](#9-fixing-the-kde-and-qt-compatibility-conflict)
- [10. Configuring User PATH](#10-configuring-user-path)
- [11. Creating the Permanent Packet Tracer Command](#11-creating-the-permanent-packet-tracer-command)
- [12. Final Directory Structure](#12-final-directory-structure)
- [13. Troubleshooting Reference Table](#13-troubleshooting-reference-table)
- [14. Uninstallation](#14-uninstallation)
- [15. Security and Licensing Disclaimer](#15-security-and-licensing-disclaimer)

---

## 1. Tested Environment

The procedures and fixes in this guide were verified in the following environment:

| Component | Specification |
| :--- | :--- |
| **Operating System** | Arch Linux (x86_64, Linux Kernel 6.x) |
| **Desktop Environment** | KDE Plasma (Wayland / X11) |
| **Default Web Browser** | Mozilla Firefox |
| **User Shell** | GNU Bash |
| **AUR Helper** | `yay` |
| **Cisco Packet Tracer Version** | 9.0.1 |
| **AUR Package** | `packettracer 9.0.1-1` |
| **Runtime Packaging** | AppImage (`packettracer.AppImage`) |

> [!NOTE]
> This represents the exact tested configuration. While these instructions are broadly applicable across Arch-based distributions (EndeavourOS, Manjaro, etc.), specific package versions and desktop environments may require minor adjustments.

---

## 2. Downloading Cisco Packet Tracer

Cisco Packet Tracer is proprietary software and cannot be redistributed directly by Arch Linux or the Arch User Repository (AUR). Users must obtain the installer through Cisco's official portal.

1. Navigate to the official Cisco Networking Academy download portal:  
   👉 **[https://www.netacad.com/resources/lab-downloads](https://www.netacad.com/resources/lab-downloads)**
2. Sign in with your registered Cisco NetAcad account.
3. Locate the **Ubuntu 64-bit** installer for version 9.0.1.
4. Download the package to your local downloads directory.

**Tested Installer Filename:**
```text
CiscoPacketTracer_901_Ubuntu_64bit.deb
```

**Initial File Location:**
```text
~/Downloads/CiscoPacketTracer_901_Ubuntu_64bit.deb
```

---

## 3. Installing Prerequisites & Building from AUR

### Step 3.1: System Update & Development Tools
Ensure your system is fully synchronized and that standard build tools are present:

```bash
sudo pacman -Syu
sudo pacman -S git base-devel
```

### Step 3.2: Search for the AUR Package
Verify package availability in the AUR:

```bash
yay -Ss packettracer
```

### Step 3.3: Initial Build Attempt & Missing Source Error
Run the installation through `yay`:

```bash
yay -S packettracer
```

During this step, the build will stop with the following expected error:

```text
==> ERROR: CiscoPacketTracer_901_Ubuntu_64bit.deb was not found in the build directory and is not a URL.
 -> error making: packettracer-exit status 1
```

**Why this occurs:** The PKGBUILD script requires the proprietary `.deb` package to construct the AppImage, but cannot legally download it on your behalf.

### Step 3.4: Resolve the Missing Source File
Copy your downloaded `.deb` installer into yay's cache build directory for `packettracer`:

```bash
cp ~/Downloads/CiscoPacketTracer_901_Ubuntu_64bit.deb ~/.cache/yay/packettracer/
```

### Step 3.5: Re-run the Build
Now trigger the build again:

```bash
yay -S packettracer
```

The build will detect the `.deb` archive, pass SHA512 checksum verification, extract the files, and successfully generate the `packettracer.AppImage`.

---

## 4. Locating the Installed AppImage

After installation, running `packettracer` directly in your shell may result in:

```text
bash: packettracer: command not found
```

To locate where the package files were placed, query `pacman`:

```bash
pacman -Ql packettracer
```

The critical binary is installed to:

```text
/usr/lib/packettracer/packettracer.AppImage
```

---

## 5. Installing the AppImage in ~/Applications

To preserve the package-managed original while ensuring your user account has clean execution permissions, install the AppImage into your personal `~/Applications` directory.

### Step 5.1: Create the User Applications Directory
```bash
mkdir -p ~/Applications
```

### Step 5.2: Safely Copy and Set File Ownership
```bash
sudo install -Dm755 /usr/lib/packettracer/packettracer.AppImage "$HOME/Applications/packettracer.AppImage"
sudo chown "$USER:$USER" "$HOME/Applications/packettracer.AppImage"
```

> [!WARNING]
> If `sudo mv` was previously used, the file may become root-owned (`-rwxr-xr-x 1 root root`), causing subsequent user modifications or launches to report `Operation not permitted`. Always verify ownership with `ls -l ~/Applications/packettracer.AppImage`.

The designated personal application location is:
```text
~/Applications/packettracer.AppImage
```

---

## 6. First Launch & Authentication Behavior

Launch Packet Tracer from the terminal:

```bash
~/Applications/packettracer.AppImage
```

Upon first launch:
1. The Cisco Packet Tracer GUI opens.
2. The End User License Agreement (EULA) prompt appears and must be accepted.
3. The Cisco login screen displays, offering to keep the user session authenticated for 3 months.
4. **The Problem:** Clicking the **Cisco Login** button fails to launch the external web browser to complete the OAuth authentication flow.

---

## 7. Installing Supporting Utilities

Inspect the terminal output during startup. You may observe warnings regarding missing utilities:

```bash
sudo pacman -S less qt6-tools xdg-utils
```

- **`less`**: Required by some internal launcher scripts for text/license paging; resolves `less: command not found`.
- **`qtpaths`**: Packet Tracer may emit warnings regarding `qtpaths: command not found`. In the tested configuration, installing `qt6-tools` does not fully eliminate internal AppImage warnings, but ensures system-level Qt discovery utilities are available.
- **Clarification:** These missing utilities are not the root cause of the browser authentication failure.

---

## 8. Configuring Firefox as Default Browser

Verify that the system's XDG default browser handler is explicitly configured for Firefox:

### Set Default Browser:
```bash
xdg-settings set default-web-browser firefox.desktop
```

### Verify Configuration:
```bash
xdg-settings get default-web-browser
```
*Expected Output:*
```text
firefox.desktop
```

### Test Browser Invocation:
```bash
xdg-open https://www.cisco.com
```
Firefox will successfully open the test page. However, clicking the login button inside Packet Tracer **still fails** to launch the browser due to an internal Qt library conflict.

---

## 9. Fixing the KDE and Qt Compatibility Conflict

> [!IMPORTANT]
> **This is the core resolution for Cisco authentication on Arch Linux with KDE Plasma.**

### The Root Cause
When launching from Packet Tracer, inspecting stderr reveals:

```text
kde-open: .../libQt6Core.so.6: version `Qt_6.12' not found
```

Packet Tracer's AppImage bundles its own internal Qt libraries. When Packet Tracer attempts to open an external authentication link, it invokes KDE Plasma's `kde-open`. Because the AppImage's internal library path takes precedence in the runtime environment, `kde-open` attempts to link against the AppImage's bundled Qt libraries rather than the host system's Qt libraries. This results in missing symbol errors and prevents the browser from ever opening.

### The Solution: Personal `kde-open` Wrapper
Create a personal wrapper script in `~/bin/kde-open` that directly delegates URL requests to Firefox without invoking KDE's conflicting binary:

```bash
# 1. Create personal binary directory
mkdir -p ~/bin

# 2. Create the wrapper script
cat > ~/bin/kde-open <<'EOF'
#!/bin/sh
exec /usr/bin/firefox "$@"
EOF

# 3. Make the wrapper executable
chmod +x ~/bin/kde-open
```

---

## 10. Configuring User PATH

Ensure your personal `~/bin` directory precedes system directories in your `PATH` so Packet Tracer invokes your custom wrapper:

```bash
echo 'export PATH="$HOME/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

### Verify Wrapper Resolution:
```bash
which kde-open
```
*Expected Output:*
```text
/home/USERNAME/bin/kde-open
```

> [!NOTE]
> This wrapper intercepts requests made by Packet Tracer and forwards URLs directly to `/usr/bin/firefox`. It does not delete, overwrite, or alter the system-wide `/usr/bin/kde-open` binary.

---

## 11. Creating the Permanent Packet Tracer Command

To ensure Packet Tracer always launches with the correct `PATH` hierarchy containing your browser wrapper, register an alias in `~/.bashrc`:

```bash
echo 'alias packettracer='\''PATH="$HOME/bin:$PATH" "$HOME/Applications/packettracer.AppImage"'\''' >> ~/.bashrc
source ~/.bashrc
```

### Final Registered Alias:
```bash
alias packettracer='PATH="$HOME/bin:$PATH" "$HOME/Applications/packettracer.AppImage"'
```

### Launch Packet Tracer:
```bash
packettracer
```

### Verification:
1. Packet Tracer launches smoothly.
2. Click **Cisco Login**.
3. Firefox immediately opens with the Cisco NetAcad authentication page.
4. Complete your login credentials in Firefox.
5. Packet Tracer authenticates, stores your 3-month session, and presents the workspace!

---

## 12. Final Directory Structure

The complete filesystem layout for this setup:

```text
/home/USERNAME/
│
├── Applications/
│   └── packettracer.AppImage                     # Executable user AppImage (755)
│
├── bin/
│   └── kde-open                                  # Browser redirection wrapper (755)
│
├── Downloads/
│   └── CiscoPacketTracer_901_Ubuntu_64bit.deb    # Original downloaded installer
│
├── .bashrc                                       # Stores PATH export and packettracer alias
│
└── .cache/
    └── yay/
        └── packettracer/
            └── CiscoPacketTracer_901_Ubuntu_64bit.deb  # AUR build directory
```

**System-Managed Reference Paths:**
- Package-managed AppImage: `/usr/lib/packettracer/packettracer.AppImage`
- AUR cache directory: `~/.cache/yay/packettracer/`

---

## 13. Troubleshooting Reference Table

| Symptom / Error | Root Cause | Solution |
| :--- | :--- | :--- |
| `packettracer: command not found` | The binary is an AppImage located in `/usr/lib/packettracer/`. | Copy the AppImage to `~/Applications/` and create the `packettracer` alias in `~/.bashrc`. |
| `ERROR: CiscoPacketTracer_901_...deb was not found` | Proprietary Cisco `.deb` cannot be downloaded automatically by yay. | Copy `CiscoPacketTracer_901_Ubuntu_64bit.deb` into `~/.cache/yay/packettracer/` and re-run `yay -S packettracer`. |
| `Operation not permitted` | File was moved or modified via `sudo`, resulting in `root` ownership. | Run `sudo chown "$USER:$USER" "$HOME/Applications/packettracer.AppImage"`. |
| `less: command not found` | Text pager utility is not installed on minimal Arch installations. | Run `sudo pacman -S less`. |
| `qtpaths: command not found` | Packet Tracer internal scripts search for Qt discovery tools. | Informational warning; installing `qt6-tools` provides system binaries, though warning is non-fatal. |
| `kde-open: .../libQt6Core.so.6: version Qt_6.12 not found` | Packet Tracer's bundled Qt libraries conflict with KDE's system `kde-open`. | Create the personal `~/bin/kde-open` script that redirects directly to `/usr/bin/firefox`. |
| `xdg-open` works in shell, but login fails in Packet Tracer | Packet Tracer specifically delegates browser calls through KDE handlers with overridden library paths. | Prepend `PATH="$HOME/bin:$PATH"` when launching the AppImage alias. |
| Blocked Google / LinkedIn tracking requests in console | Normal Content Security Policy (CSP) / telemetry blocking by network filters. | Non-fatal; safe to ignore. |
| `index.theme: File exists` | AppImage desktop integration icon cache notification. | Non-fatal; safe to ignore. |

---

## 14. Uninstallation

To completely remove Cisco Packet Tracer and all associated custom configurations:

### Step 1: Remove the AUR Package
```bash
yay -Rns packettracer
```

### Step 2: Remove the User AppImage
```bash
rm ~/Applications/packettracer.AppImage
```

### Step 3: Remove the Custom Browser Wrapper
```bash
rm ~/bin/kde-open
```

> [!CAUTION]
> Do **not** delete the entire `~/Applications` or `~/bin` directories, as they may contain other custom applications or scripts. Only remove the specific files mentioned above.

### Step 4: Clean Up `.bashrc`
Open `~/.bashrc` in your preferred text editor:
```bash
nano ~/.bashrc
```
Remove the following lines:
```bash
export PATH="$HOME/bin:$PATH"
alias packettracer='PATH="$HOME/bin:$PATH" "$HOME/Applications/packettracer.AppImage"'
```

### Step 5: Reload Shell
```bash
source ~/.bashrc
```

---

## 15. Security and Licensing Disclaimer

- **Proprietary Software:** Cisco Packet Tracer is the intellectual property of Cisco Systems, Inc. All rights reserved.
- **Authorized Acquisition:** Users must download the installation archive directly through Cisco Networking Academy ([netacad.com](https://www.netacad.com/)) using an authorized account.
- **Redistribution Notice:** This repository does **not** host, store, mirror, or distribute the `CiscoPacketTracer_901_Ubuntu_64bit.deb` archive, the `packettracer.AppImage` binary, or any proprietary Cisco binaries or assets.
- **Credentials & Privacy:** Never commit Cisco NetAcad account credentials, session tokens, or personal licensing data to version control.
- **Independent Community Resource:** This repository is an independent technical guide created for educational purposes and is neither affiliated with, sponsored by, nor endorsed by Cisco Systems, Inc.
