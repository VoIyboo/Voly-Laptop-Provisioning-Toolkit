# Voly Laptop Provisioning Toolkit

> Work in progress
> This toolkit is currently being built and tested. It is not ready for production use yet.
>             
> This is currently an idea and planned works are on the way!

A small Windows laptop provisioning toolkit for repeatable new starter device setup.

The goal of this project is to reduce manual laptop setup time while avoiding unsafe practices such as storing administrator passwords inside scripts. The toolkit uses Offline Domain Join for domain joining, then runs a no-question laptop-side build script to handle the rest of the device preparation.

## Project status

This project is currently a W.I.P.

Current focus areas:

```text
Testing Offline Domain Join flow
Improving app install detection
Improving reporting output
Adding safer error handling
Cleaning up company profile support
Testing classic Outlook signature deployment
Reviewing Intune and GPO friendly options
```

Use this toolkit in a test environment only until it has been fully validated.

## What it does

```text
Creates an Offline Domain Join package from a domain-connected admin machine
Finds the next computer name from AD based on a naming pattern
Renames the laptop or desktop
Applies Offline Domain Join without storing admin credentials on the laptop
Installs or checks required apps from Apps.json
Adds managed Edge and Chrome bookmarks
Prepares a classic Outlook signature for each user at login
Creates Outlook, Teams and Chrome shortcuts
Checks TPM, Secure Boot, BitLocker, firewall and local admins
Starts a Windows Update scan
Creates a final build report and ticket note
```

## Why this exists

Manually setting up new starter laptops can be slow and inconsistent.

This toolkit is being designed to make laptop builds:

```text
Faster
Repeatable
Documented
Safer
Easier to hand over
Less dependent on memory or manual steps
```

The main security goal is to avoid storing privileged credentials on new laptops. Instead of putting an admin password inside the script, the toolkit uses Offline Domain Join.

## Folder layout

```text
Voly-Laptop-Provisioning-Toolkit
├─ Build-OfflineJoinBlob.ps1
├─ Start-LaptopBuild.ps1
├─ Config
│  ├─ Companies.json
│  ├─ Apps.json
│  └─ Bookmarks.json
├─ Templates
│  ├─ SignatureTemplate.html
│  └─ TaskbarLayout.xml
├─ BuildBlobs
└─ Reports
```

## How it works

The toolkit has two main stages.

## Stage 1. Admin machine

Run this from a domain-connected admin workstation or server.

```powershell
.\Build-OfflineJoinBlob.ps1 -CompanyKey Sumo -DeviceType Laptop
```

This creates an Offline Domain Join package for the next available computer name.

Example output folder:

```text
BuildBlobs\SA-LT-0086
```

Inside that folder:

```text
OfflineDomainJoin.blob
DeviceBuild.json
```

Copy the generated build folder to the new laptop or to a USB.

## Stage 2. New laptop

Run this from the new laptop using PowerShell as Administrator.

```powershell
.\Start-LaptopBuild.ps1 -BuildPackagePath ".\BuildBlobs\SA-LT-0086"
```

For testing without an automatic restart:

```powershell
.\Start-LaptopBuild.ps1 -BuildPackagePath ".\BuildBlobs\SA-LT-0086" -NoRestart
```

## Configuration

## Company profiles

Company settings are stored in:

```text
Config\Companies.json
```

Example settings:

```json
{
  "DomainDns": "sumopower.local",
  "DefaultOu": "OU=Laptops,OU=Computers,DC=sumopower,DC=local",
  "Pattern": "SA-LT-{0000}"
}
```

The `{0000}` section is the number section.

The script checks AD, finds the highest matching number, then uses the next available computer name.

Example:

```text
SA-LT-0085
SA-LT-0086
SA-LT-0087
```

For another company, a different pattern can be used.

Example:

```json
{
  "Pattern": "KANE{0000}"
}
```

## Apps

Apps are configured in:

```text
Config\Apps.json
```

Supported install methods:

```text
winget
exe
msi
url
checkonly
```

Example:

```json
{
  "Name": "Google Chrome",
  "CompanyKeys": ["All"],
  "InstallMethod": "winget",
  "Source": "Google.Chrome",
  "Arguments": "",
  "Detection": {
    "Paths": ["%ProgramFiles%\\Google\\Chrome\\Application\\chrome.exe"],
    "Services": [],
    "RegistryDisplayNames": ["Google Chrome"]
  }
}
```

Example local installer:

```json
{
  "Name": "Example App",
  "CompanyKeys": ["Sumo"],
  "InstallMethod": "exe",
  "Source": "\\\\server\\share\\Installers\\ExampleApp.exe",
  "Arguments": "/S",
  "Detection": {
    "Paths": ["%ProgramFiles%\\Example App\\Example.exe"],
    "Services": [],
    "RegistryDisplayNames": ["Example App"]
  }
}
```

## Bookmarks

Bookmarks are configured in:

```text
Config\Bookmarks.json
```

Replace placeholder links before testing:

```text
https://PUT-TICKET-PORTAL-HERE
https://PUT-GENESYS-LINK-HERE
https://PUT-SINGLEVIEW-LINK-HERE
```

The toolkit applies bookmarks as managed Edge and Chrome policies.

## Useful switches

```powershell
-SkipDomainJoin
-SkipApps
-SkipBookmarks
-SkipSignature
-SkipSecurityChecks
-NoRestart
```

Example app-only test run:

```powershell
.\Start-LaptopBuild.ps1 -CompanyKey Sumo -SkipDomainJoin -NoRestart
```

## Reports

Reports are saved to:

```text
C:\ProgramData\VolyLaptopPrep\Reports
```

The toolkit also copies these files to the public desktop:

```text
Laptop Build Report.txt
Laptop Build Ticket Note.txt
```

Example ticket note:

```text
New starter laptop setup completed.

Device name: SA-LT-0086
Serial number: ABC123456
Company: Sumo Energy
Domain joined before reboot: False
Restart required: True

Warnings or items to check:
Apps: ScreenConnect Client - Check only item. No installer configured.
Security: BitLocker C: - Protection: Off
```

## Important notes

```text
Do not store admin passwords in this toolkit.
The laptop-side script is designed to run without prompts.
Offline Domain Join is used to avoid storing privileged credentials on new laptops.
The Outlook signature works best with classic Outlook.
New Outlook signatures are cloud and roaming based, so local signature scripting may not apply cleanly.
Taskbar pinning is included as an XML template, but is better deployed through Intune or GPO.
BitLocker enablement is off by default.
Turn on BitLocker automation only after confirming recovery key backup works correctly.
Always test on one spare laptop before using this on real onboarding devices.
```

## Roadmap

Planned improvements:

```text
Better app retry handling
Cleaner build summary dashboard
More detailed error logging
Optional Intune sync trigger
Optional printer deployment
Optional VPN profile setup
Optional warranty and asset report
Improved BitLocker validation
Improved ScreenConnect detection
Improved classic Outlook signature template
```

## Disclaimer

This project is experimental and provided as a work in progress.

Review the scripts before running them. Test in a lab or on a spare device first. Do not use this on production onboarding devices until the workflow has been fully tested and approved for your environment.

## Licence

No licence has been selected yet.

This project is currently private testing material unless a licence is added later.
