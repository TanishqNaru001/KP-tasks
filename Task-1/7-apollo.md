## Title

Default Credentials Lead to Administrative Access on Publicly Accessible Apollo Configuration Management Portal.

## Affected URL

http://47.84.131.67:8070/

## Description

A publicly accessible Apollo (configuration management) portal was found to be using default administrative credentials (apollo / admin). Successful authentication grants administrative privileges within the Apollo admin dashboard, allowing access to application configurations, namespace management, cluster settings, and other administrative functionality.

After authentication, the account was confirmed to have administrative privileges. Management functionality such as configuration/namespace editing, environment/cluster management, and release publishing was accessible, indicating elevated privileges.

This exposure creates a significant security risk, as Apollo is used to centrally manage application configuration across environments, meaning administrative access can be leveraged to alter configuration values consumed by connected applications and services.

## Steps to Reproduce

1. Navigate to the affected URL: http://47.84.131.67:8070/
2. Open the Apollo portal login page.
3. Authenticate using default administrative credentials (apollo / admin).
4. Observe that login is successful.
5. Navigate through the administration interface to verify administrative privileges.
6. Access configuration, namespace, and cluster settings and observe the extent of accessible functionality.

## POC


## Impact

1. Unauthorized access to the Apollo configuration management portal.
2. Exposure of application configuration values, namespaces, and cluster/environment details.
3. Modification or publishing of configuration changes affecting all applications consuming Apollo-managed configs.
4. Potential exposure of sensitive configuration data such as connection strings, credentials, or internal service endpoints.
5. Potential to disrupt or compromise downstream applications by pushing malicious or invalid configuration changes.
6. Potential to use exposed configuration data as a foothold for further compromise of connected systems.

## Remediation

1. Immediately change the default administrative credentials.
2. Enforce strong, unique passwords for all Apollo accounts.
3. Restrict access to the Apollo admin portal using network-based controls such as VPN, IP allowlists, or firewall rules.
4. Ensure Apollo is updated to the latest stable version with security patches applied.
5. Review and audit existing namespaces and configuration releases for unauthorized changes.
6. Disable or remove unused accounts and enforce least-privilege access for administrative users.
