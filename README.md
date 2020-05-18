This PowerShell Module queries Microsoft Graph, and allows for cross-tenant Backup & Restore actions of your Intune Configuration.

Intune Configuration is backed up as (json) files in a given directory.

## Installing IntuneBackupAndRestore

```powershell
# Install IntuneBackupAndRestore from the PowerShell Gallery
Install-Module -Name IntuneBackupAndRestore
```

## Prerequisites
- Requires AzureAD PowerShell Module (`Install-Module -Name AzureAD`)  
- Requires [MSGraphFunctions] (`Install-Module -Name MSGraphFunctions`)
- Connect to Microsoft Graph using the `Connect-Graph` PSCmdlet first.

## Installing MSGraphFunctions

```powershell
# Install MSGraphFunctions from the PowerShell Gallery
Install-Module -Name MSGraphFunctions
```

## Prerequisites
- Requires Azure AD Module installed.

### Authenticate to Microsoft Graph
```powershell
Import-Module MSGraphFunctions

# Authenticate for the first time and grant permissions for the "Microsoft Intune PowerShell" Enterprise Application. (Interactive Authentication (Supports MFA))
Connect-Graph -AdminConsent $true

# Interactive Authentication (Supports MFA)
Connect-Graph

# Non Interactive Authentication (Supports Automation Goals)
$Credential = Get-Credential
Connect-Graph -Credential $Credential
```

## Examples

### Example 01 - List all Intune Device Compliance Policies
```powershell
$compliancePolicies = Get-GraphDeviceCompliancePolicy
$compliancePolicies
```

### Example 02 - Duplicate an Intune Device Configuration Policy
```powershell
$deviceConfiguration = Get-GraphDeviceConfiguration -id xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
$requestBody = $deviceConfiguration | ConvertTo-Json

New-GraphDeviceConfiguration -requestBody $requestBody
```

### Example 03 - Retrieve PowerShell Script Content
```powershell
# Content of a Single Script
Get-GraphDeviceManagementScript -Id xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx | Get-GraphDeviceManagementScriptContent

# Or
Get-GraphDeviceManagementScriptContent -Id xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx

# Output PowerShell Script Content of all Uploaded Scripts
Get-GraphDeviceManagementScript | Get-GraphDeviceManagementScriptContent
```

### Example 04 - Delete an Intune Device Compliance Policy
```powershell
Get-GraphDeviceComplaincePolicy -Id xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx | Remove-GraphDeviceComplaincePolicy
```

## Features

### Backup actions
- Administrative Templates (Device Configurations)
- Administrative Template Assignments
- Client Apps
- Client App Assignments
- Device Compliance Policies
- Device Compliance Policy Assignments
- Device Configurations
- Device Configuration Assignments
- Device Management Scripts (Device Configuration -> PowerShell Scripts)
- Device Management Script Assignments
- Software Update Rings
- Software Update Ring Assignments
- Endpoint Security Configurations
  - Security Baselines
    - Windows 10 Security Baselines
    - Microsoft Defender ATP Baselines
    - Microsoft Edge Baseline
  - Antivirus
  - Disk encryption
  - Firewall
  - Endpoint detection and response
  - Attack surface reduction
  - Account protection
  - Device compliance

### Restore actions
- Administrative Templates (Device Configurations)
- Administrative Template Assignments
- Client App Assignments
- Device Compliance Policies
- Device Compliance Policy Assignments
- Device Configurations
- Device Configuration Assignments
- Device Management Scripts (Device Configuration -> PowerShell Scripts)
- Device Management Script Assignments
- Software Update Rings
- Software Update Ring Assignments
- Endpoint Security Configurations
  - Security Baselines
    - Windows 10 Security Baselines
    - Microsoft Defender ATP Baselines
    - Microsoft Edge Baseline
  - Antivirus
  - Disk encryption
  - Firewall
  - Endpoint detection and response
  - Attack surface reduction
  - Account protection
  - Device compliance

> Please note that some Client App settings can be backed up, for instance the retrieval of Win32 (un)install cmdlets, requirements, etcetera. The Client App itself is not backed up and this module does not support restoring Client Apps at this time.

## Examples

### Example 01 - Full Intune Backup
```powershell
Start-IntuneBackup -Path C:\temp\IntuneBackup
```

### Example 02 - Full Intune Restore
```powershell
Start-IntuneRestoreConfig -Path C:\temp\IntuneBackup
Start-IntuneRestoreAssignments -Path C:\temp\IntuneBackup
```

### Example 03 - Restore Intune Assignments 
If configurations have been restored:
```powershell
Start-IntuneRestoreAssignments -Path C:\temp\IntuneBackup
```

If reassigning assignments to existing (non-restored) configurations. In this case the assignments match the configuration id to restore to.  
This allows for restoring if display names have changed.
```powershell
Start-IntuneRestoreAssignments -Path C:\temp\IntuneBackup -RestoreById $true
```

### Example 04 - Restore only Intune Compliance Policies

```powershell
Invoke-IntuneRestoreDeviceCompliancePolicy -Path C:\temp\IntuneBackup
```

```powershell
Invoke-IntuneRestoreDeviceCompliancePolicyAssignments -Path C:\temp\IntuneBackup
```

### Example 05 - Restore Only Intune Device Configurations
```powershell
Invoke-IntuneRestoreDeviceConfiguration -Path C:\temp\IntuneBackup
```

```powershell
Invoke-IntuneRestoreDeviceConfigurationAssignments -Path C:\temp\IntuneBackup
```

### Example 06 - Backup Only Intune Endpoint Security Configurations
```powershell
Invoke-IntuneBackupDeviceManagementIntent -Path C:\temp\IntuneBackup
```

### Example 07 - Restore Only Intune Endpoint Security Configurations
```powershell
Invoke-IntuneRestoreDeviceManagementIntent -Path C:\temp\IntuneBackup
```

### Example 08 - Compare two Backup Files for changes
```powershell
# The DifferenceFilePath should point to the latest Intune Backup file, as it might contain new properties.
Compare-IntuneBackupFile -ReferenceFilePath 'C:\temp\IntuneBackup\Device Configurations\Windows - Endpoint Protection.json' -DifferenceFilePath 'C:\temp\IntuneBackupLatest\Device Configurations\Windows - Endpoint Protection.json'
```

### Example 09 - Compare all files in two Backup Directories for changes
```powershell
# The DifferenceFilePath should point to the latest Intune Backup file, as it might contain new properties.
Compare-IntuneBackupDirectories -ReferenceDirectory 'C:\temp\IntuneBackup' -DifferenceDirectory 'C:\temp\IntuneBackup2'
```

## Known Issues
- Does not support backing up Intune configuration items with duplicate Display Names. Files may be overwritten.
- Unable to restore Client App Assignments for Windows Line-of-Business Apps (MSI)
