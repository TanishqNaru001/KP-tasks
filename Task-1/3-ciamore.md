## Title

Default Credentials Lead to Administrative Access on Publicly Accessible CIAmore Device Interface.

## Affected URL

http://218.62.44.110:9999/

## Description

A publicly accessible CIAmore management interface was found to be accessible using default credentials (admin / admin). Successful authentication grants administrative privileges within the application, allowing access to system configurations, management functionality, and administrative settings.

After authentication, the account was confirmed to have administrative privileges. Management functionality such as system configuration, user management, and administrative settings was accessible, indicating elevated privileges.

This exposure creates a significant security risk, as the exposed interface could be reconfigured, monitored, or compromised by an unauthorized party.

## Steps to Reproduce

1. Navigate to the affected URL: http://218.62.44.110:9999/
2. Open the CIAmore login page.
3. Authenticate using default administrative credentials (admin / admin).
4. Observe that login is successful.
5. Navigate through the management interface to verify administrative privileges.
6. Access configuration settings and observe the extent of accessible functionality.

## POC


## Impact

1. Unauthorized access to the CIAmore management interface.
2. Exposure of system configuration and administrative details.
3. Modification of application settings and configurations.
4. Potential to disrupt operations or cause denial of service.
5. Access to sensitive information stored within or managed by the application.
6. Potential to use the exposed system as a pivot point for further compromise.

## Remediation

1. Immediately disable all default credentials.
2. Reset administrator passwords to strong, unique values.
3. Restrict access to the management interface using network-based controls such as VPN, IP allowlists, or firewall rules.
4. Ensure the application/firmware is up to date with security patches.
5. Implement management interface access controls if supported.
6. Consider using dedicated management network segmentation.
