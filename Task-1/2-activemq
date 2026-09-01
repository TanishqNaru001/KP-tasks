## Title

Default Credentials Lead to Administrative Access on Publicly Accessible ActiveMQ Instance.

## Affected URL

http://18.185.36.91:1880

## Description

A publicly accessible ActiveMQ instance was found to be using default administrative credentials (admin / admin). Successful authentication grants administrative privileges within the ActiveMQ web console, allowing access to queue and topic management, message browsing, broker configuration, and other administrative functionality.

After authentication, the account was confirmed to have administrative privileges. Management functionality such as queue/topic management, message inspection, and broker configuration was accessible, indicating elevated privileges.

This exposure creates a significant security risk, as ActiveMQ instances handle application messaging and can, in some configurations, be leveraged for further compromise (e.g., via known ActiveMQ RCE vectors if the version is outdated).

## Steps to Reproduce

1. Navigate to the affected URL: http://18.185.36.91:1880
2. Open the ActiveMQ web console login page.
3. Authenticate using default administrative credentials (admin / admin).
4. Observe that login is successful.
5. Navigate through the administration interface to verify administrative privileges.
6. Access queue, topic, and broker configuration settings and observe the extent of accessible functionality.

## POC


## Impact

1. Unauthorized access to the ActiveMQ administration console.
2. Exposure of queue and topic configurations, and message contents.
3. Creation, deletion, or modification of queues, topics, and broker settings.
4. Potential exposure of sensitive data transiting messaging queues.
5. Potential to leverage administrative access as a foothold for further compromise, particularly if the ActiveMQ version is vulnerable to known remote code execution issues.
6. Potential to disrupt application functionality relying on the message broker, leading to denial of service.

## Remediation

1. Immediately change the default administrative credentials.
2. Enforce strong, unique passwords for all ActiveMQ accounts.
3. Restrict access to the ActiveMQ web console using network-based controls such as VPN, IP allowlists, or firewall rules.
4. Ensure ActiveMQ is updated to the latest stable version with security patches applied.
5. Review and audit existing queues and topics for unauthorized changes or message exposure.
6. Disable or remove unused accounts and enforce least-privilege access for administrative users.
