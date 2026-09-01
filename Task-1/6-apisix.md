## Title

Default Credentials Lead to Administrative Access on Publicly Accessible Apache APISIX Dashboard.

## Affected URL

http://106.75.161.77:9000/

## Description

A publicly accessible Apache APISIX Dashboard was found to be using default administrative credentials (admin / admin). Successful authentication grants administrative privileges within the APISIX Dashboard, allowing access to route configurations, upstream services, plugin management, consumer/API key management, and other administrative functionality.

After authentication, the account was confirmed to have administrative privileges. Management functionality such as route/upstream configuration, plugin management, and consumer management was accessible, indicating elevated privileges.

This exposure creates a significant security risk, as APISIX acts as an API gateway and reverse proxy, meaning administrative access can be leveraged to intercept, redirect, or manipulate traffic to backend services.

## Steps to Reproduce

1. Navigate to the affected URL: http://106.75.161.77:9000/
2. Open the Apache APISIX Dashboard login page.
3. Authenticate using default administrative credentials (admin / admin).
4. Observe that login is successful.
5. Navigate through the administration interface to verify administrative privileges.
6. Access route, upstream, and plugin configuration settings and observe the extent of accessible functionality.

## POC


## Impact

1. Unauthorized access to the Apache APISIX administration dashboard.
2. Exposure of route configurations, upstream service details, and API consumer/key information.
3. Modification or deletion of routes, upstreams, plugins, and consumers, enabling traffic interception or redirection.
4. Potential exposure of sensitive backend service details and internal network topology.
5. Potential to leverage administrative access to inject malicious plugins or reroute traffic to attacker-controlled endpoints.
6. Potential to disrupt API gateway operations for all services routed through the instance, leading to denial of service.

## Remediation

1. Immediately change the default administrative credentials.
2. Enforce strong, unique passwords for all APISIX Dashboard accounts.
3. Restrict access to the APISIX Dashboard using network-based controls such as VPN, IP allowlists, or firewall rules.
4. Ensure Apache APISIX and its Dashboard are updated to the latest stable versions with security patches applied.
5. Review and audit existing routes, upstreams, and plugins for unauthorized changes.
6. Disable or remove unused accounts and enforce least-privilege access for administrative users.
