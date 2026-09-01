## Title

Default Credentials Lead to Administrative Access on Publicly Accessible Azkaban Workflow Manager.

## Affected URL

http://134.33.72.50:8081/index

## Description

A publicly accessible Azkaban workflow management interface was found to be using default credentials (azkaban / azkaban). Successful authentication grants access within the Azkaban web console, allowing access to project management, workflow scheduling, job configurations, and execution history.

After authentication, the account was confirmed to have administrative privileges. Management functionality such as project upload, workflow/job configuration, and execution scheduling was accessible, indicating elevated privileges.

This exposure creates a significant security risk, as Azkaban is used to orchestrate and execute batch workflows and jobs, meaning administrative access can be leveraged to execute arbitrary jobs and scripts within the environment it manages.

## Steps to Reproduce

1. Navigate to the affected URL: http://134.33.72.50:8081/index
2. Open the Azkaban login page.
3. Authenticate using default credentials (azkaban / azkaban).
4. Observe that login is successful.
5. Navigate through the web console to verify administrative privileges.
6. Access project, workflow, and job configuration settings and observe the extent of accessible functionality.

## POC


## Impact

1. Unauthorized access to the Azkaban workflow management console.
2. Exposure of project configurations, workflow/job definitions, and execution history/logs.
3. Ability to upload, modify, or schedule new projects and jobs, potentially leading to arbitrary command/script execution.
4. Potential exposure of sensitive information such as connection strings or credentials embedded within job configurations.
5. Potential to disrupt or compromise downstream systems executed or orchestrated by scheduled jobs.
6. Potential to use the execution environment as a pivot point for further network compromise.

## Remediation

1. Immediately change the default credentials.
2. Enforce strong, unique passwords for all Azkaban accounts.
3. Restrict access to the Azkaban web console using network-based controls such as VPN, IP allowlists, or firewall rules.
4. Ensure Azkaban is updated to the latest stable version with security patches applied.
5. Review and audit existing projects, jobs, and execution logs for unauthorized changes or activity.
6. Disable or remove unused accounts and enforce least-privilege access for administrative users.
