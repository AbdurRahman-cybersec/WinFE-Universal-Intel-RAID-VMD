<img alt="Static Badge" src="https://img.shields.io/badge/license-education-blue?logo=github"> <img alt="Static Badge" src="https://img.shields.io/badge/In_Progress-WinFE-orange">

##  Repository Name
**WinFE-Universal-Intel-RAID-VMD**


##  Objective
Build a **fully functional Windows Forensic Environment (WinFE)** USB capable of detecting **internal NVMe and RAID volumes** on modern Intel-based systems (8th–15th Gen) while **preserving forensic integrity**.

This WinFE build ensures:
- No automatic disk mounting
- No write operations to target drives
- Reliable detection of RAID/VMD-managed storage


##  Device & Platform Compatibility

### Supported Systems
- Intel-based systems from **8th to 15th Generation**
- Intel Rapid Storage Technology (**RST**)
- Intel Volume Management Device (**VMD**)
- Storage modes: **AHCI**, **RAID**, **VMD (NVMe)**

### Typical Hardware Profiles
- Intel Core **i5 / i7 / i9** (8th–15th Gen)
- NVMe SSDs under RAID or VMD
- Intel chipsets:
  - 300-series → 800-series
  - Examples: `Z390`, `Z490`, `Z590`, `Z690`, `Z790`

---

##  Requirements

### 1️⃣ Windows ADK + WinPE Add-on
- Install the **Windows Assessment and Deployment Kit (ADK)**
- Install the **WinPE Add-on** matching the ADK version

### 2️⃣ Intel WinFE Toolkit
- Extracted locally, for example:
  ```text
  C:\IntelWinFE
  ```

### 3️⃣ Intel Rapid Storage Technology (RST) Drivers
- Version: **v20.x** (12th–15th Gen compatible)
- Download from Intel official website: https://www.intel.com/content/www/us/en/search.html#q=Intel%C2%AE%20Rapid%20Storage%20Technology%20Driver&t=All
- File:
  ```text
  SetupRST.exe
  ```

### 4️⃣ Administrator Access
- All commands must be run from an **elevated Command Prompt**


##  Step 1: Build the WinFE Base

1. Open **Command Prompt** as Administrator
2. Navigate to the Intel WinFE directory:
   ```cmd
   cd C:\IntelWinFE
   ```
3. Run the build script:
   ```cmd
   MakeWinFEx64-x86.bat
   ```
4. Wait for completion

📁 Output file:
```text
USB\x86-x64\x64\sources\boot.wim
```

---

##  Step 2: Extract Intel RAID / VMD Drivers

1. Extract the Intel RST drivers:
   ```cmd
   SetupRST.exe -extractdrivers "C:\RST_VMD"
   ```

2. Verify driver directory:
   ```text
   C:\RST_VMD\production\Windows10-x64\19041\Drivers\VMD
   ```

3. Confirm required files:
- `iaStorVD.inf`
- `iaStorVD.sys`
- `iaStorVD.cat`

---

##  Step 3: Inject Intel VMD Driver into WinFE

### Mount the WinFE Image
```cmd
mkdir C:\WinFE_Mount
Dism /Mount-Wim /WimFile:"D:\sources\boot.wim" /Index:1 /MountDir:"C:\WinFE_Mount"
```
> Replace `D:` with your actual WinFE USB drive letter

### Inject the Driver
```cmd
Dism /Image:"C:\WinFE_Mount" /Add-Driver /Driver:"C:\RST_VMD\production\Windows10-x64\19041\Drivers\VMD" /Recurse
```

### Commit and Unmount
```cmd
Dism /Unmount-Wim /MountDir:"C:\WinFE_Mount" /Commit
```

---

##  Step 4: Verify Driver Injection

1. Mount image for verification:
```cmd
mkdir C:\CheckMount
Dism /Mount-Wim /WimFile:"D:\sources\boot.wim" /Index:1 /MountDir:"C:\CheckMount"
```

2. Confirm driver presence:
```cmd
Dism /Get-Drivers /Image:"C:\CheckMount" | find "iaStorVD"
```

3. Unmount without saving:
```cmd
Dism /Unmount-Wim /MountDir:"C:\CheckMount" /Discard
```

 Expected output:
```text
iaStorVD.inf
```

---

##  Step 5: Test on Target System

1. Boot the target system from the WinFE USB with RAID/VMD enabled in BIOS
2. Open Command Prompt:
```cmd
diskpart
list disk
```

### Expected Result
- `Disk 0` → Internal NVMe / RAID volume
- `Disk 1` → WinFE USB

3. Verify driver loaded:
```cmd
driverquery | find "iaStor"
```

 Expected:
```text
iaStorVD.sys
```

---

##  Step 6 (Optional): Add Universal NVMe Drivers

To maximize compatibility, integrate vendor NVMe drivers:

### Supported Vendors
- Samsung NVMe
- Western Digital / SanDisk
- Micron / Crucial

Inject drivers using:
```cmd
Dism /Image:"C:\WinFE_Mount" /Add-Driver /Driver:"<NVMe_Driver_Folder>" /Recurse
```

---

## 🏁 Outcome

Your WinFE USB now:
- Boots on modern Intel systems (8th–15th Gen)
- Detects RAID, VMD, and NVMe storage
- Preserves forensic integrity
- Supports extensible driver integration

---

##  Next Recommendation

Place forensic tools in:
```text
X:\Tools
```

Recommended tools:
- Autopsy
- FTK Imager
- Magnet Acquire (portable)

---

## 📜 License & Disclaimer

This project is intended for **authorized forensic and DFIR use only**.
Ensure proper legal authority before examining any system.

