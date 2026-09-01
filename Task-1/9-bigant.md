## Title

Default Credentials Lead to Administrative Access on Publicly Accessible BigAnt Server Admin Panel.

## Affected URL

http://152.207.22.94:8000/index.php/home/login/index/saas/default/to/admin.html

## Description

A publicly accessible BigAnt Server administration panel was found to be using default/weak credentials (admin / 123456). Successful authentication grants administrative privileges within the BigAnt admin console, allowing access to user management, system configuration, messaging/collaboration settings, and other administrative functionality.

After authentication, the account was confirmed to have administrative privileges. Management functionality such as user/organization management, system configuration, and service settings was accessible, indicating elevated privileges.

This exposure creates a significant security risk, as BigAnt Server hosts internal messaging and collaboration data, meaning administrative access can be leveraged to view, modify, or exfiltrate sensitive organizational communications and user data.

## Steps to Reproduce

1. Navigate to the affected URL: http://152.207.22.94:8000/index.php/home/login/index/saas/default/to/admin.html
2. Open the BigAnt Server admin login page.
3. Authenticate using default/weak credentials (admin / 123456).
4. Observe that login is successful.
5. Navigate through the admin console to verify administrative privileges.
6. Access user management, system configuration, and messaging settings and observe the extent of accessible functionality.

## POC


## Impact

1. Unauthorized access to the BigAnt Server admin console.
2. Exposure of user accounts, organizational structure, and system configuration details.
3. Modification of user permissions, system settings, and messaging/collaboration configurations.
4. Potential exposure of sensitive internal communications and user data.
5. Potential to create, modify, or delete user accounts, leading to further unauthorized access.
6. Potential to disrupt messaging/collaboration services or use the platform as a pivot point for further compromise.

## Remediation

1. Immediately change the default/weak administrative credentials.
2. Enforce strong, unique passwords for all BigAnt Server accounts.
3. Restrict access to the admin panel using network-based controls such as VPN, IP allowlists, or firewall rules.
4. Ensure BigAnt Server is updated to the latest stable version with security patches applied.
5. Review and audit existing user accounts and system configurations for unauthorized changes.
6. Disable or remove unused accounts and enforce least-privilege access for administrative users.
