## Title

Publicly Accessible Jenkins Dashboard Exposed Without Authentication.

## Affected URL

http://66.179.9.19/

## Description

A publicly accessible Jenkins instance was found to be exposing its dashboard without requiring authentication. Anonymous/unauthenticated access allows visibility into the Jenkins environment, including jobs, build history, and other information typically restricted to authenticated users.

Unrestricted access to the dashboard indicates that anonymous access controls are either disabled or misconfigured, potentially exposing build jobs, project names, plugin details, and other internal information to any unauthenticated visitor.

This exposure creates a significant security risk, as Jenkins instances often contain sensitive CI/CD data, credentials, and build artifacts, and depending on configured permissions may allow further unauthorized actions.

## Steps to Reproduce

1. Navigate to the affected URL: http://66.179.9.19/
2. Observe that the Jenkins dashboard loads without requiring authentication.
3. Browse through the exposed dashboard to view accessible jobs, builds, and configuration details.
4. Note the extent of information and functionality accessible without valid credentials.

## POC


## Impact

1. Unauthorized access to the Jenkins dashboard and CI/CD environment.
2. Exposure of job names, build history, and project structure.
3. Potential exposure of sensitive information such as environment variables, build logs, or plugin configurations.
4. Potential to escalate access further if additional misconfigurations (e.g., script console access, missing authorization matrix) exist.
5. Risk of unauthorized modification or triggering of build jobs if write access is also exposed.
6. Potential to use exposed information as a foothold for further compromise of connected systems.

## Remediation

1. Enable authentication and enforce login for all Jenkins users.
2. Configure an authorization strategy (e.g., Matrix-based security) to restrict anonymous and authenticated user permissions appropriately.
3. Restrict access to the Jenkins interface using network-based controls such as VPN, IP allowlists, or firewall rules.
4. Ensure Jenkins core and plugins are updated to the latest stable versions with security patches applied.
5. Review job configurations and build logs for exposure of sensitive credentials or data.
6. Disable the script console for non-administrative users and audit existing user permissions.
