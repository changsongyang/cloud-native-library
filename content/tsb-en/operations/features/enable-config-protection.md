---
title: Config Protection
description: How to configure config protection for TSB created resources
---

Config protection is a feature that helps protect your Istio configuration from accidental changes.
This allows you to configure the users who are allowed to make changes to the Istio configuration generated
by TSB, protecting your Istio configuration from unintended changes. By default, a user with Kube namespace privileges
would be able to create new Istio configurations or edit TSB created configurations. While user managed configurations
are not altered, TSB managed configurations when changed, would be overwritten by TSB during the next sync cycle.

This feature is available in two variants:
- `enableAuthorizedUpdateDeleteOnXcpConfigs`: This allows the user to create and manage Istio configurations that are not managed by TSB.
You can add a specific set of users to the authorizedUsers list to give those users the privilege to edit or
delete TSB managed configurations.
- `enableAuthorizedCreateUpdateDeleteOnXcpConfigs`: This prevents any unauthorized user from creating, updating
or deleting any Istio configuration in the cluster.

You can configure the users who are allowed to make changes to the Istio configuration generated by TSB through your
`ControlPlane` CR.

By default, config protection is disabled for all Istio resources created by TSB.

You can enable this feature by adding the following to `ControlPlane` CR in XCP component:
```yaml
 configProtection:
   enableAuthorizedUpdateDeleteOnXcpConfigs: true
   enableAuthorizedCreateUpdateDeleteOnXcpConfigs: true
   authorizedUsers:
     - user1
     - system:serviceaccount:ns1:serviceaccount-1
```

For more details about ConfigProtection configuration, please refer to the following section
[Config Protection](../../refs/install/common/common_config#configprotection)