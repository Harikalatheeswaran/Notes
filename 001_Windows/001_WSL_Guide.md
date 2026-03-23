# **Windows Subsystem for Linux (WSL) Guide**

## What is WSL and What Does It Do?
The Windows Subsystem for Linux (WSL) is a **compatibility layer** built directly into Windows 10 (1903+) and Windows 11. It lets you run a full Linux environment **natively** on Windows — no traditional virtual machine, no dual-boot, no extra hardware.

**Key benefits**:
- Run real Linux tools (`bash`, `apt`, `git`, `python`, `nmap`, etc.) side-by-side with Windows apps.
- Seamless file access: Linux sees your Windows drives at `/mnt/c/`, Windows sees Linux files in Explorer.
- Multiple distros (Ubuntu, Kali, Debian, etc.) running at the same time.
- Perfect for developers, pentesters, students, and anyone who needs Linux but loves Windows.

**WSL 1 vs WSL 2** (you will almost always use WSL 2):
- **WSL 1**: Translation layer (no VM). Lightweight but limited compatibility.
- **WSL 2**: Real lightweight VM + Microsoft’s custom Linux kernel. Full system-call compatibility, much faster I/O, Docker support, GUI apps. **This is the default and recommended version**.

## Does WSL Install a Linux Distro as a VM?
- WSL 2 uses a **lightweight VM** (managed by Microsoft, not VirtualBox/Hyper-V directly). You never see or manage the VM yourself.
- The actual Linux filesystem is stored in a single `.vhdx` virtual hard disk file that grows dynamically.
- It feels exactly like native Linux — super fast startup, low memory overhead (~300–600 MB when idle).

## Does WSL Allow Linux Commands Inside Windows CMD?
Yes, but **not directly**.  
You run Linux commands in a **WSL terminal** (bash/zsh).  
From PowerShell, CMD, or Windows Terminal you can do:
```powershell
wsl ls -la                  # run one command
wsl -d Ubuntu ls -la        # specify distro
wsl -d kali-linux nmap -sV  # run inside Kali
```
**Windows Terminal** (free from Microsoft Store) is the best — it shows Ubuntu and Kali as separate tabs.

## WSL Command Parameters Explained (Very Important)
Here are the flags you will see:

- `--install` → Enables WSL + Virtual Machine Platform + installs **default Ubuntu** + sets WSL 2 as default.
- `-d` or `--distribution` → Specifies which distro to use (e.g., `Ubuntu`, `kali-linux`, `Debian`).
- `--set-default-version 2` → Makes WSL 2 the default for all future distros.
- `--list` or `-l` → Shows installed distros.
  - `--verbose` or `-v` → Shows version (1 or 2) and state.
  - `--online` or `-o` → Shows **all available** distros you can install (very useful!).
- `-u` or `--user` → Runs as specific user (e.g., `root`).
- `--unregister` → Completely deletes a distro and its data.
- `--uninstall` → Removes **entire WSL** from Windows.
- `--update` → Updates WSL kernel and components.
- `--shutdown` → Shuts down all WSL VMs (frees RAM).

## Steps to Install Default Ubuntu & Kali Linux

### Prerequisites (same as your notes)
- Windows 10 1903+ or Windows 11
- Admin rights
- Internet connection

### Step 1: Enable WSL + Install Default Ubuntu (One Command!)
Open **PowerShell as Administrator** and run:
```powershell
wsl --install
```
**What happens**:
- Enables WSL + Virtual Machine Platform
- Downloads latest WSL kernel
- Installs **Ubuntu** (latest LTS — usually 24.04 or 22.04)
- Sets WSL 2 as default

Restart if prompted.

**To install Kali instead of (or in addition to) Ubuntu**:
```powershell
wsl --install -d kali-linux
```
(This is the official 2025+ method from Kali docs — no Store needed.)

**Alternative (Microsoft Store method)** — same result:
1. Open Microsoft Store
2. Search “Ubuntu” → Install (or “Kali Linux”)
3. Launch from Start menu

**Verify everything**:
```powershell
wsl --list --verbose
```
You should see:
```
  NAME            STATE           VERSION
* Ubuntu          Stopped         2
  kali-linux      Stopped         2
```

## Setting Up Username & Password (Critical First-Run Step)

**There is NO default username or password.**

When you launch the distro for the **very first time**, it automatically runs the setup wizard:

1. Launch Ubuntu or Kali from Start menu (or `wsl -d Ubuntu`)
2. It will say:
   ```
   Installing, this may take a few minutes...
   Please create a default UNIX user account.
   Enter new UNIX username:
   ```
3. Type your desired username (e.g., `buddy` or `kaliuser`) → Enter
4. Type a strong password → Enter → Confirm it

**Done!**  
This user automatically gets **sudo** rights. This is now your permanent default user.

### How to Change Password Later
Inside the distro:
```bash
passwd
```
Enter current password → new password → confirm.

### If You Forget Your Password (Reset Method)
From **PowerShell** (no need to open the distro):
```powershell
wsl -u root
```
(Or `wsl -d kali-linux -u root` if multiple distros)

Now you are root. Reset your user’s password:
```bash
passwd yourusername
```
Type new password twice.

Exit with `exit`.  
Next time you launch, it will use your normal user again.

**To change username** (advanced, not usually needed):
Create new user with `sudo adduser newname`, give sudo rights, then set it as default with:
```powershell
<distro>.exe config --default-user newname
```
(Replace `<distro>` with `ubuntu.exe` or `kali.exe`)

## Disk Space Usage (Clear Breakdown – Terminal Only vs GUI)

WSL stores everything in a dynamic `.vhdx` file (grows as you use it, max 256 GB by default).

**Base WSL + Kernel**: ~150–300 MB

**Terminal-only (CLI) usage**:
- **Ubuntu base** (fresh install, no extra packages): **~600–900 MB**
- After `sudo apt update && upgrade` + basic dev tools: **1–2 GB**
- **Kali base** (fresh): **1.2–2 GB**
- After `sudo apt install kali-linux-default` (most common tools): **4–7 GB**

**With GUI**:
- **Ubuntu GUI** (install `ubuntu-desktop` or just use WSLg apps): +1.5–3 GB extra (total ~3–5 GB)
- **Kali + Win-KeX GUI** (kali-win-kex + desktop): +3–5 GB extra
- Full Kali with `kali-linux-large` + Win-KeX + tools: **10–20+ GB** easily

**Real-world examples** (after 1 month normal use):
- Ubuntu CLI only (coding, git, python): 2–4 GB
- Kali CLI only (pentesting tools): 5–8 GB
- Kali GUI (Win-KeX windowed desktop): 12–18 GB

You can check exact usage:
```powershell
wsl --list --verbose
```
Or from Linux: `df -h`

### Requirements for GUI
```
WSL2 (not WSL1).
Windows 11 or Windows 10 (version 21H2 or higher) for wslg.
~2-3 GB additional disk space for Win-KeX and GUI tools.
RDP client (included in Windows) for Win-KeX’s default mode.
```

## Disk Space Usage
WSL consumes disk space on the host Windows system:

**WSL2 VM**: ~1-2 GB for the Linux kernel and virtual disk (VHDX).
**Ubuntu**:
Base install: ~600 MB.
With packages (e.g., development tools): ~2-5 GB.

**Kali Linux**:
Base install: ~1-2 GB.
With kali-linux-default: ~5-10 GB.
With kali-linux-large or GUI tools (Win-KeX): ~10-20 GB+.

**Win-KeX (GUI)**: ~2-3 GB for desktop environment and dependencies.
**VHDX Expansion**: WSL2’s virtual disk grows dynamically (up to 250 GB default) as you add files/packages.
**Location**:
VHDX: %userprofile%\AppData\Local\Docker\wsl\data.
Distro files: %userprofile%\AppData\Local\Packages.

**Management**:
Check usage: wsl --list --verbose.
Reclaim space: wsl --manage <distro> --set-sparse true.
Remove distro: wsl --unregister <distro>.

## Enabling GUI Support for Kali (Win-KeX) – Updated Steps
(Your notes are still correct — just added examples)

Inside Kali:
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y kali-win-kex
kex --win          # windowed mode (recommended)
# or
kex --esm -s       # Enhanced Session Mode (best clipboard/audio)
```

## Enabling GUI Support for Kali Linux
WSL2 supports GUI apps via `wslg` (WSL GUI). For Kali, use **Win-KeX** (Kali’s Windows-Kex) for a seamless desktop experience.

### Steps to Enable GUI for Kali
1. Update Kali:
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```
2. Install Win-KeX:
   ```bash
   sudo apt install -y kali-win-kex
   ```
3. Start Win-KeX in windowed mode:
   ```bash
   kex --win
   ```
   - This launches a Kali desktop via RDP in a window.
   - Alternatives: `kex --sl` (seamless mode) or `kex --esm` (enhanced session mode).
4. (Optional) Install additional Kali tools for GUI apps:
   ```bash
   sudo apt install -y kali-linux-default
   ```
5. If GUI fails, ensure WSL2 and `wslg` are active:
   ```powershell
   wsl --update
   ```
6. Access the desktop via Windows RDP client or directly in the Win-KeX window.

## Additional Notes (Expanded)
- **Performance**: WSL 2 is faster than most VMs. Idle RAM ~500 MB.
- **Networking**: Kali tools work perfectly (including Metasploit, Wireshark).
- **Updates**: Always run `wsl --update` in PowerShell + `sudo apt update && sudo apt full-upgrade` inside Linux.

## Steps to Delete the WSL Environment (Complete Cleanup)

### Option 1: Delete One Distro Only (Recommended)
```powershell
wsl --unregister Ubuntu
wsl --unregister kali-linux
```
This deletes **everything** for that distro (files, packages, settings). VHDX is gone. Space freed immediately.

### Option 2: Delete Entire WSL (Everything)
```powershell
wsl --uninstall
```
This removes:
- WSL feature
- Virtual Machine Platform
- Kernel
- All distros

Then restart and optionally disable the features in “Turn Windows features on or off” → “Windows Subsystem for Linux” and “Virtual Machine Platform”.

**Verify deletion**:
```powershell
wsl --list
```
Should say “The operation completed successfully. No installed distributions.”

---

### Storage optimization.

### Expected Disk Usage (WSL2 + Basic Dev Tools)

| Setup                              | Fresh Install | After `apt update && upgrade` + **basic dev tools** | After 1–2 months of normal use (git, Python, Node.js, VS Code Remote, Docker, etc.) |
|------------------------------------|---------------|-----------------------------------------------------|-------------------------------------------------------------------------------------|
| **Ubuntu only** (recommended)      | 650–900 MB    | **1.5 – 2.5 GB**                                    | **2.5 – 4 GB**                                                                      |
| **Kali only** (light)              | 1.2–1.8 GB    | **2.5 – 4 GB**                                      | **4 – 7 GB**                                                                        |
| **Both Ubuntu + Kali** (light)     | —             | **4 – 6.5 GB**                                      | **7 – 11 GB**                                                                       |

- **“Basic dev tools”** = `build-essential`, `git`, `python3-pip`, `nodejs`, `curl`, `wget`, `vim`, `gcc`, `make`, `docker`, etc.  
 - That’s it — nothing heavy like full Kali-large or GUI desktop.

- Even in the **worst case** (both distros + some extra packages after months), you will use **under 12 GB total**.  


### Why You Don’t Need to Worry
- WSL2 uses a **dynamic VHDX file** — it only grows when you actually add files/packages. It does **not** pre-allocate 256 GB.
- Idle overhead of WSL2 itself: ~200–400 MB.
- You can run just **one** distro (Ubuntu is lighter and perfect for dev). No need for both unless you specifically want Kali.

### Storage-Conscious Tips (Do These Once)

1. **Check real usage anytime** (PowerShell):
   ```powershell
   wsl --list --verbose
   ```
   Or inside Linux:
   ```bash
   du -sh /  # total used
   df -h     # free space
   ```

2. **Make the disk sparse** (saves 20–50% space automatically):
   ```powershell
   wsl --manage Ubuntu --set-sparse true
   ```
   (Do the same for Kali if installed)

3. **Compact the VHDX** when you delete big files:
   ```powershell
   wsl --shutdown
   optimize-vhd -Path "$env:USERPROFILE\AppData\Local\Packages\CanonicalGroupLimited.Ubuntu_79rhkp1fndgsc\LocalState\ext4.vhdx" -Mode Full
   ```
   (Replace path if needed — or just use the command below for any distro)

4. **Want to move everything off C: later?** (Super easy, keeps C: clean)
   - Export the distro to D: drive
   - Import it back to any folder on D:
   ```powershell
   wsl --export Ubuntu C:\temp\ubuntu.tar
   wsl --import Ubuntu D:\WSL\Ubuntu C:\temp\ubuntu.tar --version 2
   ```


