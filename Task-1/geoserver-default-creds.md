## Title

Default Credentials Lead to Administrative Access on Publicly Accessible GeoServer Instance.

## Affected URL

https://www.eic.mn:8080

## Description

A publicly accessible GeoServer instance was found to be using default administrative credentials (admin / geoserver). Successful authentication grants administrative privileges within the GeoServer web administration interface, allowing access to data store configurations, layer management, workspace settings, and other administrative functionality.

After authentication, the account was confirmed to have administrative privileges. Management functionality such as data store configuration, layer/workspace management, and server settings was accessible, indicating elevated privileges.

This exposure creates a significant security risk, as GeoServer instances are frequently used to host and process geospatial data and can, in some configurations, be leveraged for further compromise (e.g., via known GeoServer RCE vectors if the version is outdated).

## Steps to Reproduce

1. Navigate to the affected URL: https://www.eic.mn:8080
2. Open the GeoServer web administration login page.
3. Authenticate using default administrative credentials (admin / geoserver).
4. Observe that login is successful.
5. Navigate through the administration interface to verify administrative privileges.
6. Access data store, workspace, and layer configuration settings and observe the extent of accessible functionality.

## POC


## Impact

1. Unauthorized access to the GeoServer administration interface.
2. Exposure of data store connection details, workspace configurations, and layer definitions.
3. Modification or deletion of published layers, workspaces, and data stores.
4. Potential exposure of sensitive geospatial or backend datastore credentials.
5. Potential to leverage administrative access as a foothold for further compromise, particularly if the GeoServer version is vulnerable to known remote code execution issues.
6. Reputational and operational impact if geospatial services are disrupted or defaced.

## Remediation

1. Immediately change the default administrative credentials.
2. Enforce strong, unique passwords for all GeoServer accounts.
3. Restrict access to the GeoServer admin interface using network-based controls such as VPN, IP allowlists, or firewall rules.
4. Ensure GeoServer is updated to the latest stable version with security patches applied.
5. Review and audit existing data stores and layers for unauthorized changes.
6. Disable or remove unused accounts and enforce least-privilege access for administrative users.
