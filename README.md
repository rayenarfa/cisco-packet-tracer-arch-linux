# Cisco Packet Tracer on Arch Linux

A comprehensive, step-by-step installation and troubleshooting guide for running **Cisco Packet Tracer 9.0.1** on **Arch Linux** using the AUR package and AppImage, including critical fixes for KDE Plasma / Qt browser authentication failures and unreadable dark-theme UI color schemes.

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
- [9. Fixing the KDE and Qt Compatibility Conflict (Browser Authentication)](#9-fixing-the-kde-and-qt-compatibility-conflict-browser-authentication)
- [10. Configuring User PATH](#10-configuring-user-path)
- [11. Fixing KDE Plasma Theme & Color Scheme Incompatibilities (Unreadable UI Text)](#11-fixing-kde-plasma-theme--color-scheme-incompatibilities-unreadable-ui-text)
- [12. Creating the Permanent Packet Tracer Command](#12-creating-the-permanent-packet-tracer-command)
- [13. Creating the Desktop Application Launcher (.desktop Entry with Icon)](#13-creating-the-desktop-application-launcher-desktop-entry-with-icon)
- [14. Final Directory Structure](#14-final-directory-structure)
- [15. Troubleshooting Reference Table](#15-troubleshooting-reference-table)
- [16. Uninstallation](#16-uninstallation)
- [17. Security and Licensing Disclaimer](#17-security-and-licensing-disclaimer)

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

## 9. Fixing the KDE and Qt Compatibility Conflict (Browser Authentication)

> [!IMPORTANT]
> **This is the core resolution for Cisco browser authentication on Arch Linux with KDE Plasma.**

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

## 11. Fixing KDE Plasma Theme & Color Scheme Incompatibilities (Unreadable UI Text)

### The Problem
When running Packet Tracer 9.0.1 on KDE Plasma (particularly when using dark or custom color schemes), a severe Qt theme incompatibility occurs:
- Certain configuration tabs display dark input fields with nearly invisible black/dark text labels.
- Other dialogs render with mismatched, unreadable contrast.
- Affected dialogs commonly include:
  - **Router0 → Config → Routing → Static**
  - **Router0 → Config → Routing → RIP**
  - **Router0 → Config → Interface settings**

Rather than altering your system-wide KDE Plasma theme every time you study networking, you can force Packet Tracer to render using Qt's neutral **Fusion** style while masking KDE desktop environment variables.

### Step 11.1: Test the Environment Wrapper
Close Packet Tracer completely, then execute this test command in your terminal:

```bash
env \
  XDG_CURRENT_DESKTOP="" \
  KDE_FULL_SESSION="" \
  KDE_SESSION_VERSION="" \
  QT_QPA_PLATFORMTHEME="" \
  QT_STYLE_OVERRIDE="Fusion" \
  PATH="$HOME/bin:$PATH" \
  "$HOME/Applications/packettracer.AppImage"
```

#### What Each Variable Accomplishes:

| Variable | Purpose |
| :--- | :--- |
| `XDG_CURRENT_DESKTOP=""` | Prevents Packet Tracer from detecting KDE Plasma. |
| `KDE_FULL_SESSION=""` | Prevents KDE session-level styling integration. |
| `KDE_SESSION_VERSION=""` | Prevents KDE-specific Qt version integration hooks. |
| `QT_QPA_PLATFORMTHEME=""` | Disables the KDE Qt platform theme engine for this process. |
| `QT_STYLE_OVERRIDE="Fusion"` | Forces Qt to use the clean, neutral **Fusion** widget style. |
| `PATH="$HOME/bin:$PATH"` | Preserves the `~/bin/kde-open` Firefox authentication workaround. |

### Step 11.2: Check Affected Configuration Pages
With Packet Tracer open under this environment:
1. Place a router (e.g. `Router0`) on the workspace.
2. Click the router and navigate to:
   - **Config → Routing → Static**
   - **Config → Routing → RIP**
   - **Config → Interface**
3. Verify that all input boxes, table entries, and background text are crisp, clean, and completely readable.

---

## 12. Creating the Permanent Packet Tracer Command

Now that both the browser authentication fix and the Qt Fusion theme fix are verified, combine them into a permanent shell alias.

### Step 12.1: Replace Any Existing Alias in `~/.bashrc`
Remove any previous simple alias and register the complete environment alias:

```bash
sed -i '/^alias packettracer=/d' ~/.bashrc

cat >> ~/.bashrc <<'EOF'
alias packettracer='env XDG_CURRENT_DESKTOP= KDE_FULL_SESSION= KDE_SESSION_VERSION= QT_QPA_PLATFORMTHEME= QT_STYLE_OVERRIDE=Fusion PATH="$HOME/bin:$PATH" "$HOME/Applications/packettracer.AppImage"'
EOF

source ~/.bashrc
```

### Final Registered Alias:
```bash
alias packettracer='env XDG_CURRENT_DESKTOP= KDE_FULL_SESSION= KDE_SESSION_VERSION= QT_QPA_PLATFORMTHEME= QT_STYLE_OVERRIDE=Fusion PATH="$HOME/bin:$PATH" "$HOME/Applications/packettracer.AppImage"'
```

### Launch Packet Tracer:
```bash
packettracer
```

### Benefits of this Permanent Configuration:
1. **Your KDE Plasma theme is untouched**: Your desktop stays dark and styled however you like.
2. **Packet Tracer UI is 100% readable**: Rendered with clean Fusion controls on every dialog.
3. **Cisco NetAcad login works flawlessly**: Redirects cleanly to Firefox.

---

## 13. Creating the Desktop Application Launcher (.desktop Entry with Icon)

For the best user experience on KDE Plasma (or GNOME / XFCE), you can integrate Packet Tracer directly into your application launcher so it appears when pressing the **Super** (Windows) key, complete with the official Cisco Packet Tracer icon.

### Step 13.1: Create the Clean Binary Launcher
Instead of relying on a shell-only alias, create an executable wrapper in `~/bin/packettracer`:

```bash
cat > ~/bin/packettracer <<'EOF'
#!/bin/sh
exec env \
  XDG_CURRENT_DESKTOP="" \
  KDE_FULL_SESSION="" \
  KDE_SESSION_VERSION="" \
  QT_QPA_PLATFORMTHEME="" \
  QT_STYLE_OVERRIDE="Fusion" \
  PATH="$HOME/bin:$PATH" \
  "$HOME/Applications/packettracer.AppImage" "$@"
EOF

chmod +x ~/bin/packettracer
```

### Step 13.2: Extract the Official High-Resolution Icon
The official Packet Tracer icon is embedded inside the AppImage. Extract it using `7z`:

```bash
mkdir -p ~/.local/share/icons
7z e ~/Applications/packettracer.AppImage app.png -o"$HOME/.local/share/icons/" -y
mv ~/.local/share/icons/app.png ~/.local/share/icons/cisco-packet-tracer.png
```

### Step 13.3: Create the Desktop Entry
Create `~/.local/share/applications/cisco-packet-tracer.desktop`:

```ini
cat > ~/.local/share/applications/cisco-packet-tracer.desktop <<'EOF'
[Desktop Entry]
Type=Application
Name=Cisco Packet Tracer
GenericName=Network Simulator
Comment=Cisco Packet Tracer Networking Simulation Tool
Exec=/home/USERNAME/bin/packettracer %f
Icon=/home/USERNAME/.local/share/icons/cisco-packet-tracer.png
Terminal=false
Categories=Education;Science;Network;
MimeType=application/x-pkt;application/x-pka;application/x-pkz;application/x-pksz;
StartupWMClass=PacketTracer
Keywords=cisco;packet;tracer;network;simulation;
EOF
```
*(Replace `USERNAME` with your actual Linux username, or use `$USER`).*

### Step 13.4: Update KDE System Configuration Cache
Force KDE Plasma to index the new application immediately:

```bash
update-desktop-database ~/.local/share/applications
kbuildsycoca6
```

Now, pressing **Super** and typing `packet` shows **Cisco Packet Tracer** with its official icon and launches it with all browser and theme fixes applied!

---

## 14. Final Directory Structure

The complete filesystem layout for this setup:

```text
/home/USERNAME/
│
├── Applications/
│   └── packettracer.AppImage                     # Executable user AppImage (755)
│
├── bin/
│   ├── kde-open                                  # Browser redirection wrapper (755)
│   └── packettracer                              # Permanent executable launcher (755)
│
├── .local/share/
│   ├── applications/
│   │   └── cisco-packet-tracer.desktop           # System application launcher entry
│   └── icons/
│       └── cisco-packet-tracer.png               # Official Cisco icon
│
├── Downloads/
│   └── CiscoPacketTracer_901_Ubuntu_64bit.deb    # Original downloaded installer
│
├── .bashrc                                       # Stores PATH export
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

## 15. Troubleshooting Reference Table

| Symptom / Error | Root Cause | Solution |
| :--- | :--- | :--- |
| `packettracer: command not found` | The binary is an AppImage located in `/usr/lib/packettracer/`. | Copy the AppImage to `~/Applications/` and create the `packettracer` alias in `~/.bashrc`. |
| `ERROR: CiscoPacketTracer_901_...deb was not found` | Proprietary Cisco `.deb` cannot be downloaded automatically by yay. | Copy `CiscoPacketTracer_901_Ubuntu_64bit.deb` into `~/.cache/yay/packettracer/` and re-run `yay -S packettracer`. |
| `Operation not permitted` | File was moved or modified via `sudo`, resulting in `root` ownership. | Run `sudo chown "$USER:$USER" "$HOME/Applications/packettracer.AppImage"`. |
| `less: command not found` | Text pager utility is not installed on minimal Arch installations. | Run `sudo pacman -S less`. |
| `qtpaths: command not found` | Packet Tracer internal scripts search for Qt discovery tools. | Informational warning; installing `qt6-tools` provides system binaries, though warning is non-fatal. |
| `kde-open: .../libQt6Core.so.6: version Qt_6.12 not found` | Packet Tracer's bundled Qt libraries conflict with KDE's system `kde-open`. | Create the personal `~/bin/kde-open` script that redirects directly to `/usr/bin/firefox`. |
| `xdg-open` works in shell, but login fails in Packet Tracer | Packet Tracer specifically delegates browser calls through KDE handlers with overridden library paths. | Prepend `PATH="$HOME/bin:$PATH"` when launching the AppImage alias. |
| **Dark input fields, invisible text on Static Routing/RIP/Config** | Qt theme incompatibility between Packet Tracer 9.x and KDE Plasma color schemes. | Launch with `QT_STYLE_OVERRIDE=Fusion` and disabled KDE session variables in the alias. |
| Blocked Google / LinkedIn tracking requests in console | Normal Content Security Policy (CSP) / telemetry blocking by network filters. | Non-fatal; safe to ignore. |
| `index.theme: File exists` | AppImage desktop integration icon cache notification. | Non-fatal; safe to ignore. |

---

## 16. Uninstallation

To completely remove Cisco Packet Tracer and all associated custom configurations:

### Step 1: Remove the AUR Package
```bash
yay -Rns packettracer
```

### Step 2: Remove the User AppImage
```bash
rm ~/Applications/packettracer.AppImage
```

### Step 3: Remove Custom Wrappers and Desktop Entry
```bash
rm ~/bin/kde-open
rm ~/bin/packettracer
rm ~/.local/share/applications/cisco-packet-tracer.desktop
rm ~/.local/share/icons/cisco-packet-tracer.png
update-desktop-database ~/.local/share/applications
```

> [!CAUTION]
> Do **not** delete the entire `~/Applications`, `~/bin`, or `~/.local/share/` directories, as they contain other system applications and scripts. Only remove the specific files mentioned above.

### Step 4: Clean Up `.bashrc`
Open `~/.bashrc` in your preferred text editor:
```bash
nano ~/.bashrc
```
Remove the following lines:
```bash
export PATH="$HOME/bin:$PATH"
alias packettracer='env XDG_CURRENT_DESKTOP= KDE_FULL_SESSION= KDE_SESSION_VERSION= QT_QPA_PLATFORMTHEME= QT_STYLE_OVERRIDE=Fusion PATH="$HOME/bin:$PATH" "$HOME/Applications/packettracer.AppImage"'
```

### Step 5: Reload Shell
```bash
source ~/.bashrc
```

---

## 17. Security and Licensing Disclaimer

- **Proprietary Software:** Cisco Packet Tracer is the intellectual property of Cisco Systems, Inc. All rights reserved.
- **Authorized Acquisition:** Users must download the installation archive directly through Cisco Networking Academy ([netacad.com](https://www.netacad.com/)) using an authorized account.
- **Redistribution Notice:** This repository does **not** host, store, mirror, or distribute the `CiscoPacketTracer_901_Ubuntu_64bit.deb` archive, the `packettracer.AppImage` binary, or any proprietary Cisco binaries or assets.
- **Credentials & Privacy:** Never commit Cisco NetAcad account credentials, session tokens, or personal licensing data to version control.
- **Independent Community Resource:** This repository is an independent technical guide created for educational purposes and is neither affiliated with, sponsored by, nor endorsed by Cisco Systems, Inc.
