---
title: 'Astronomer Software v0.30 release notes'
sidebar_label: 'Astronomer Software'
id: release-notes
description: Astronomer Software release notes.
---

<!--- Version-specific -->

0.30 is the latest stable and long-term support (LTS) version of Astronomer Software. To upgrade to 0.30, see [Upgrade Astronomer](upgrade-astronomer.md). For more information about Software release channels, see [Release and lifecycle policies](release-lifecycle-policy.md). To read release notes specifically for the Astro CLI, see [Astro CLI release notes](cli-release-notes.md).

## 0.30.2

Release date: September 22, 2022

### Additional improvements

- You can now use the [Fluentd Helm chart](https://github.com/astronomer/astronomer/blob/master/charts/fluentd/values.yaml) to set a `securityContext` for Fluentd Pods and containers.
- Improved the startup time for the platform NATS server.
- You can now configure external containers in the `astronomer.houston.config` section of the Astronomer Helm chart.

### Bug fixes

- Fixed several CVEs as a result of updating images for system components. 

## 0.30.1

Release date: September 12, 2022

### Bug fixes

- Fixed the following vulnerabilities:
    - [CVE-2022-1996](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2022-1996)
    - [CVE-2022-21698](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2022-21698)
    - [CVE-2022-35949](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2022-35949)
    - [CVE-2022-35948](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2022-35948)
    - [CVE-2022-37434](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2022-37434)

## 0.30.0

Release date: August 29, 2022

:::danger Breaking Change for Azure Database for PostgreSQL

A change in 0.30 enabled the `trgm` extension for PostgreSQL. If you use Azure Database for PostgreSQL as your database backend, you need to enable the `pg_trgm` extension before upgrading to Software 0.30 using either Azure portal or the Azure CLI. See [Azure documentation](https://docs.microsoft.com/en-us/azure/postgresql/flexible-server/concepts-extensions) for configuration steps.

If you don't complete this setup before your upgrade, the upgrade will fail.

:::

### Improved token refreshing for IdP integrations

The Software UI now refreshes your JSON web token (JWT) based on the validity of your authentication token from your IdP. This means that as long as you stay logged in to your IdP, you no longer have to refresh the Software UI to continue accessing the Software UI, Astro CLI, and Houston API.

Additionally, if you change a user's access to Astronomer from your IdP, their permissions will be automatically updated in Astronomer after their current IdP token expires. If you remove a user completely from Astronomer, they are automatically logged out of the Software UI and CLI after their current IdP token expires.

As part of this change, you can now configure `jwt.authDuration` in your [Houston Helm configuration](https://github.com/astronomer/docs/blob/main/software_configs/0.30/default.yaml). If a user is logged on longer than `authDuration`, they will be immediately logged out regardless of the status of their JWT or authentication token.

### Additional improvements

- Workspace users are now paginated in the Software UI.
- You can now configure credentials for a private image registry by specifying a secret you create instead of a username and password. The secret is attached to any Pods that need to access the registry.
- You can now specify `authUrlParams` for your identity provider (IdP) in `config.yaml`.
- System Editors can no longer manage Teams or users in a Workspace. These permissions are now available only at the System Admin level.

### Bug fixes

- Fixed an issue where applying an IAM role to a Deployment would reset the Deployment's **Extra Capacity** setting back to the default of 0 AU.
- Fixed an issue where System Admins could receive an error when trying to view a Team imported from a different IdP than their current one.
- When a System Admin makes a change to a Team, that change now appears in the UI without needing to refresh the page.
- Configurations for disabling a specific executor type in `config.yaml` are now reflected in the Software UI.
- Fixed an issue where Workspace-level service accounts could view Deployment information from Deployments outside of their Workspace.
- Fixed an issue where updating the role of a user in a Team using the Astro CLI would not throw an error as expected.
- Fixed an issue where JSON web tokens persisted after a user logged out if `idpGroupsRefreshEnabled` was set to `false`.
- Users authenticating with Google Direct are no longer automatically logged out of Astronomer Software after 1 hour.
