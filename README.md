# Entra Hybrid Join Remediation

A set of PowerShell scripts for Microsoft Intune to detect and remediate devices with Entra hybrid join in a PENDING registration state.

## Problem Statement

When devices are hybrid joined to Entra ID (formerly Azure AD), they sometimes get stuck in a PENDING registration state, which can cause various authentication issues including:

- Device-based Conditional Access problems
- Single Sign-On failures
- Microsoft 365 application authentication issues
- Windows Hello for Business enrollment failures

This remediation package provides an automated solution to detect and fix affected devices.

## Solution

This repository contains two PowerShell scripts designed to be deployed as an Intune Remediation Script Package:

1. **Detection Script**: Identifies devices that are both Entra hybrid joined AND have a PENDING registration status
2. **Remediation Script**: Runs `dsregcmd /leave` with elevated permissions to clear the problematic join state

## Scripts Overview

### Detection Script (`Detect-EntraHybridJoinPending.ps1`)

- Checks if the device is both Entra hybrid joined and has a PENDING registration status
- Returns exit code 1 (non-compliant) if both conditions are true, triggering the remediation script
- Returns exit code 0 (compliant) if no remediation is needed
- Logs detailed information about the device's current join state

### Remediation Script (`Remediate-EntraHybridJoinPending.ps1`)

- Runs with administrator privileges to execute `dsregcmd /leave`
- Verifies the operation was successful by checking the device state after remediation
- Returns appropriate exit codes for Intune reporting
- Provides comprehensive logging for troubleshooting

## Deployment Instructions

### Prerequisites

- Microsoft Intune subscription
- Permissions to create and assign Remediation scripts (formerly Proactive Remediations)
- Target devices must be running Windows 10 or Windows 11

### Steps to Deploy in Intune

1. Sign in to the [Microsoft Intune admin center](https://endpoint.microsoft.com/)
2. Navigate to **Endpoint security** > **Remediation** > **Create script package**
3. Enter the following details:
   - **Name**: Entra Hybrid Join Pending Remediation
   - **Description**: Fixes devices with Entra hybrid join in PENDING registration status

4. On the **Detection script** tab:
   - Upload the `Detect-EntraHybridJoinPending.ps1` file
   - Configure these settings:
     - **Run this script using the logged-on credentials**: No
     - **Run script in 64-bit PowerShell**: Yes
     - **Enforce script signature check**: Optional (if you sign your scripts)

5. On the **Remediation script** tab:
   - Upload the `Remediate-EntraHybridJoinPending.ps1` file
   - Configure these settings:
     - **Run this script using the logged-on credentials**: No
     - **Run script in 64-bit PowerShell**: Yes
     - **Run script with administrator privileges**: Yes (crucial)
     - **Enforce script signature check**: Optional (if you sign your scripts)

6. On the **Scope tags** tab, add any relevant scope tags

7. On the **Assignments** tab:
   - Assign to the appropriate groups of devices
   - Set schedule and frequency as needed for your environment
   - Consider testing with a small group before wider deployment

8. Review your settings and click **Create**

## Logging

Both scripts log detailed information to help with troubleshooting:

- Detection script log: `C:\ProgramData\Microsoft\IntuneManagementExtension\Logs\EntraHybridJoinDetection.log`
- Remediation script log: `C:\ProgramData\Microsoft\IntuneManagementExtension\Logs\EntraHybridJoinRemediation.log`

The logs include:
- Timestamp for each action
- Device join status details
- Success/failure information
- Any errors encountered

## Troubleshooting

Common issues and solutions:

1. **Remediation not triggering**:
   - Check the detection script logs to verify correct detection
   - Ensure the detection script is returning exit code 1 when remediation is needed

2. **Remediation script fails**:
   - Verify the script is running with administrator privileges
   - Check the remediation logs for specific error messages
   - Test running `dsregcmd /leave` manually on an affected device

3. **Device still shows problems after remediation**:
   - Some devices may require a restart after the join state is cleared
   - Consider adding a scheduled task to restart the device after remediation

## Additional Notes

- After remediation, devices will need to re-register with Entra ID
- For domain-joined devices, this typically happens automatically at next user sign-in
- The remediation does not affect the domain join status, only the Entra (Azure AD) join

## License

[MIT License](LICENSE)

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.
