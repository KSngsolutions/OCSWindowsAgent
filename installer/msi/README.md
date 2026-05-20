# OCS Inventory NG Agent MSI (WiX)

This directory adds an MSI installer flow for enterprise auto-deployment while preserving the existing NSIS installer in `NSIS_agent_setup/`.

## What this MSI does

- Supports silent deployment with `msiexec /qn`
- Installs OCS agent binaries to `C:\Program Files\OCS Inventory Agent`
- Generates `ocsinventory.ini` in `C:\ProgramData\OCS Inventory NG\Agent`
- Registers the Windows service by running `OcsService.exe -install`
- Unregisters/stops the Windows service on uninstall with `OcsService.exe -uninstall`
- Keeps service startup mode managed by the native service registration logic (`SERVICE_AUTO_START` in `OcsService.exe`)

## Prerequisites

1. Build the Windows agent binaries first (solution output in `Release\`).
2. Install WiX Toolset v3.11 or newer (with MSBuild integration).
3. Build from a Windows machine with Visual Studio build tools.

## Build MSI

From repository root:

```powershell
msbuild installer\msi\OCS-Windows-Agent.wixproj /p:Configuration=Release /p:Platform=x64
```

If your binaries are in a different output directory, override `ReleaseDir`:

```powershell
msbuild installer\msi\OCS-Windows-Agent.wixproj /p:Configuration=Release /p:Platform=x64 /p:ReleaseDir="C:\path\to\Release"
```

## Silent install

```powershell
msiexec /i OCS-Windows-Agent-x64.msi /qn SERVER_URL="https://ocs.example.local/ocsinventory" SERVER_SSL=1 DEBUG=1 PROLOG_FREQ=1 /l*v C:\Windows\Temp\OCSAgentInstall.log
```

## Supported MSI properties

- `SERVER_URL` (default: `http://localhost/ocsinventory`)
- `SERVER_SSL` (default: `0`)
- `SERVER_USER` (default: empty)
- `SERVER_PASSWORD` (default: empty)
- `SERVER_AUTH_REQUIRED` (default: `0`)
- `DEBUG` (default: `0`)
- `PROLOG_FREQ` (default: `1`)
- `INVENTORY_ON_STARTUP` (default: `1`)
- `PROXY_TYPE` (default: `0`)
- `PROXY` (default: empty)
- `PROXY_PORT` (default: empty)
- `PROXY_AUTH_REQUIRED` (default: `0`)
- `PROXY_USER` (default: empty)
- `PROXY_PASSWORD` (default: empty)

## Logging

### MSI logging (installer)

Use standard Windows Installer verbose logging:

```powershell
msiexec /i OCS-Windows-Agent-x64.msi /qn /l*v C:\Windows\Temp\OCSAgentInstall.log
```

### Agent logging

Set `DEBUG` MSI property (`0`, `1`, or `2`), which is written to:

- `[OCS Inventory Agent] Debug=` in `C:\ProgramData\OCS Inventory NG\Agent\ocsinventory.ini`

## Silent uninstall

```powershell
msiexec /x OCS-Windows-Agent-x64.msi /qn /l*v C:\Windows\Temp\OCSAgentUninstall.log
```

## Notes

- The MSI intentionally uses `OcsService.exe -install` / `-uninstall` rather than pure WiX `ServiceInstall` to preserve existing service registration behavior implemented by the project.
- The existing NSIS installer flow is unchanged.
