## Title

Default Credentials Lead to Administrative Access on Publicly Accessible Canon Device Interface.

## Affected URL

http://139.177.33.206:8000/

## Description

A publicly accessible Canon device management interface was found to be accessible using default/weak credentials (Administrator / 7654321). Successful authentication grants administrative privileges within the device management application, allowing access to device configurations, management functionality, and administrative settings.

After authentication, the account was confirmed to have administrative privileges. Management functionality such as device configuration, system settings, and network/device monitoring was accessible, indicating elevated privileges.

This exposure creates a significant security risk as the device can be reconfigured, monitored, or compromised, and may expose further information about the internal network it is connected to.

## Steps to Reproduce

1. Navigate to the affected URL: http://139.177.33.206:8000/
2. Open the Canon device login page.
3. Authenticate using the identified administrative credentials (Administrator / 7654321).
4. Observe that login is successful.
5. Navigate through the management interface to verify administrative privileges.
6. Access configuration settings and observe the extent of accessible functionality.

## POC


## Impact

1. Unauthorized access to the Canon device management interface.
2. Exposure of device configuration and network details.
3. Modification of device settings, network configuration, and security policies.
4. Potential to disrupt device operations or cause denial of service.
5. Access to sensitive information such as stored jobs, address books, or network scan data, depending on device type.
6. Potential to use the device as a pivot point for further network compromise.

## Remediation

1. Immediately change the default/weak administrative credentials.
2. Enforce strong, unique passwords for all device administrator accounts.
3. Restrict access to the device management interface using network-based controls such as VPN, IP allowlists, or firewall rules.
4. Ensure device firmware is up to date with security patches.
5. Implement management interface access controls if supported.
6. Consider using dedicated management network segmentation.
