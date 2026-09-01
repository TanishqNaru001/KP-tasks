## Title

Default Credentials Lead to Administrative Access on Publicly Accessible Bonita BPM Portal.

## Affected URL

http://3.249.149.67/bonita/login.jsp?_l=en&redirectUrl=apps%2FappDirectoryBonita

## Description

A publicly accessible Bonita BPM portal was found to be using default/well-known credentials (walter.bates / bpm). Successful authentication grants access within the Bonita web application, allowing access to business process applications, case management, user/organization data, and other functionality depending on the account's assigned privileges.

After authentication, the account was confirmed to have elevated privileges within the platform. Functionality such as application access, process/case management, and organization data was accessible, indicating meaningful access to the environment.

This exposure creates a significant security risk, as Bonita BPM is used to run business process applications that may process sensitive organizational data, meaning unauthorized access can be leveraged to view or manipulate business workflows and associated data.

## Steps to Reproduce

1. Navigate to the affected URL: http://3.249.149.67/bonita/login.jsp?_l=en&redirectUrl=apps%2FappDirectoryBonita
2. Open the Bonita BPM portal login page.
3. Authenticate using default/well-known credentials (walter.bates / bpm).
4. Observe that login is successful.
5. Navigate through the portal to verify the level of access granted.
6. Access available applications, cases, and organization data and observe the extent of accessible functionality.

## POC


## Impact

1. Unauthorized access to the Bonita BPM portal and hosted applications.
2. Exposure of business process applications, case data, and organizational information.
3. Ability to interact with, modify, or progress business process cases depending on account permissions.
4. Potential exposure of sensitive data processed or stored within business applications.
5. Potential to disrupt business workflows relying on the platform.
6. Potential to use exposed access as a foothold for further compromise, particularly if administrative accounts are also using default credentials.

## Remediation

1. Immediately change all default/well-known account credentials.
2. Enforce strong, unique passwords for all Bonita BPM accounts.
3. Restrict access to the Bonita portal using network-based controls such as VPN, IP allowlists, or firewall rules.
4. Ensure Bonita BPM is updated to the latest stable version with security patches applied.
5. Review and audit user accounts, roles, and process/case data for unauthorized access or changes.
6. Disable or remove unused/default accounts and enforce least-privilege access for all users.
