---
layout: post
title: "Locking Down an Azure Environment Using Tags, Azure Policy and Exemptions"
description: "A practical Azure governance pattern for stopping new resource deployment in a legacy or frozen environment while retaining controlled management of existing resources."
date: 2026-08-04
author: The Cloud Captain
categories:
  - Azure
  - Governance
tags:
  - Azure Policy
  - Azure Governance
  - Resource Tags
  - Policy Exemptions
  - Azure RBAC
  - Resource Locks
  - Legacy Environments
permalink: /articles/locking-down-an-azure-environment/
---

> **Design objective**
>
> Prevent new Azure resources from being deployed into a legacy or frozen environment, while allowing approved teams to continue operating and updating resources that already exist.

> **Important limitation**
>
> Azure Policy evaluates both resource creation and resource update requests. It does not provide a universal `create-only` condition that works consistently across every Azure resource provider. The solution in this article therefore uses a tag to identify the frozen environment, a deny policy to block requests, and resource-level policy exemptions to preserve approved existing resources.

| Document item | Value |
| --- | --- |
| Audience | Cloud architects, platform engineers, governance teams, and subscription owners |
| Primary services | Azure Policy, Azure resource tags, policy exemptions, Azure RBAC, and Azure Activity Log |
| Main use case | Freeze a legacy resource group without making every existing resource read-only |
| Control tag | `EnvironmentState = Frozen` |
| Recommended rollout | Audit and inventory first, exemptions second, enforcement last |

## Contents

1. [The requirement](#1-the-requirement)
2. [Why Azure resource locks are not enough](#2-why-azure-resource-locks-are-not-enough)
3. [Why a simple tag-based deny policy is not enough](#3-why-a-simple-tag-based-deny-policy-is-not-enough)
4. [Recommended governance pattern](#4-recommended-governance-pattern)
5. [Control design](#5-control-design)
6. [Create the main freeze policy](#6-create-the-main-freeze-policy)
7. [Create the tag protection policy](#7-create-the-tag-protection-policy)
8. [Assign the main policy safely](#8-assign-the-main-policy-safely)
9. [Inventory and exempt existing resources](#9-inventory-and-exempt-existing-resources)
10. [Apply the frozen tag](#10-apply-the-frozen-tag)
11. [Validation tests](#11-validation-tests)
12. [RBAC and operational protection](#12-rbac-and-operational-protection)
13. [Break-glass and controlled change](#13-break-glass-and-controlled-change)
14. [Important limitations and edge cases](#14-important-limitations-and-edge-cases)
15. [Recommended production rollout](#15-recommended-production-rollout)
16. [Reference links](#16-reference-links)

## 1. The requirement

Organisations often reach a point where an Azure environment must remain operational but should no longer grow.

Common examples include:

- A legacy application that is being retired.
- A subscription that is being replaced by a new landing zone.
- A resource group retained for regulatory, audit, or migration reasons.
- An environment where only break-fix changes should be permitted.
- A platform that must remain available while new deployments move elsewhere.

The desired outcome normally sounds simple:

> Allow teams to manage the resources that already exist, but prevent them from creating anything new.

The difficulty is that Azure Resource Manager and Azure Policy do not expose a universal policy condition that reliably distinguishes a create request from an update request across every resource type.

A standard deny policy normally evaluates the requested final state. If the request matches the deny condition, Azure blocks it whether it represents a new resource or a change to an existing resource.

## 2. Why Azure resource locks are not enough

Azure resource locks provide two useful controls:

- `CanNotDelete` prevents deletion but still allows modification.
- `ReadOnly` prevents both modification and deletion.

Neither lock directly meets the requirement:

| Control | Existing resource updates | Existing resource deletion | New resource creation |
| --- | ---: | ---: | ---: |
| No lock | Allowed | Allowed | Allowed |
| `CanNotDelete` | Allowed | Blocked | Generally allowed |
| `ReadOnly` | Blocked | Blocked | May disrupt management operations but is not a purpose-built create-only control |

A delete lock is useful as an additional safeguard for critical resources, but it does not stop someone from deploying another resource into the same resource group.

A read-only lock is usually too restrictive because it prevents operational changes to existing resources. It can also interfere with service-specific operations that use Azure Resource Manager writes behind the scenes.

For this reason, resource locks should be treated as a complementary protection rather than the primary environment-freeze mechanism.

## 3. Why a simple tag-based deny policy is not enough

A first attempt might be to deny any resource carrying this tag:

```text
EnvironmentState = Frozen
```

That introduces two problems.

### 3.1 Existing resources would also be blocked

If the policy denies requests for resources in the frozen state, an update to an existing resource is also evaluated. The update can therefore be denied even though the resource already existed before the environment was frozen.

### 3.2 A new resource could copy the tag

If the policy logic is designed incorrectly, a user might add the same legacy or frozen tag to a new resource and accidentally satisfy an allow condition.

The control tag should classify the **resource group or environment**, not act as proof that an individual resource existed before the freeze date.

## 4. Recommended governance pattern

The recommended pattern contains four layers:

1. **Environment classification**  
   Apply `EnvironmentState = Frozen` to the resource group.

2. **Deny policy**  
   Deny resource requests inside resource groups whose state is `Frozen`.

3. **Existing-resource exemptions**  
   Create policy exemptions at the lowest practical scope for resources that existed before the freeze.

4. **Administrative protection**  
   Restrict who can change the control tag, policy assignment, or exemptions through Azure RBAC and Privileged Identity Management.

The effective behaviour is:

| Request | Policy result |
| --- | --- |
| Update an exempt existing resource | Allowed |
| Delete an exempt existing resource | Allowed unless a delete lock or separate policy blocks it |
| Create a new top-level resource | Denied |
| Update a non-exempt resource | Denied |
| Create a resource in a non-frozen resource group | Not affected by this policy |

This is not an Azure-native historical record of the environment. The approved resource inventory and its exemptions become the allow-list representing what existed at the time of the freeze.

## 5. Control design

Use a resource-group tag rather than requiring every resource type to support tags.

Recommended tag:

```text
EnvironmentState = Frozen
```

Suggested state values for a broader lifecycle model:

| Value | Meaning |
| --- | --- |
| `Active` | Normal deployment and operational changes are allowed |
| `Restricted` | New deployments require approval |
| `Frozen` | New resource creation is denied; existing approved resources are exempt |
| `Retired` | Environment is scheduled for removal or archival |

The policy in this article uses `Frozen` as the enforcement trigger.

## 6. Create the main freeze policy

Create a custom Azure Policy definition at management group or subscription scope.

The policy checks the tag on the parent resource group by using the `resourceGroup()` policy function. It excludes resource-group objects and selected governance/deployment resource types so that the control can still be administered.

```json
{
  "properties": {
    "displayName": "Deny resource changes in frozen resource groups",
    "policyType": "Custom",
    "mode": "All",
    "description": "Denies resource create and update requests in resource groups tagged as Frozen. Existing approved resources must be covered by policy exemptions.",
    "metadata": {
      "category": "Governance",
      "version": "1.0.0"
    },
    "parameters": {
      "stateTagName": {
        "type": "String",
        "metadata": {
          "displayName": "Environment state tag name"
        },
        "defaultValue": "EnvironmentState"
      },
      "frozenTagValue": {
        "type": "String",
        "metadata": {
          "displayName": "Frozen tag value"
        },
        "defaultValue": "Frozen"
      },
      "effect": {
        "type": "String",
        "metadata": {
          "displayName": "Effect"
        },
        "allowedValues": [
          "Audit",
          "Deny",
          "Disabled"
        ],
        "defaultValue": "Audit"
      }
    },
    "policyRule": {
      "if": {
        "allOf": [
          {
            "value": "[if(contains(toLower(field('id')), '/resourcegroups/'), resourceGroup().tags[parameters('stateTagName')], '')]",
            "equals": "[parameters('frozenTagValue')]"
          },
          {
            "field": "type",
            "notIn": [
              "Microsoft.Resources/subscriptions/resourceGroups",
              "Microsoft.Resources/deployments",
              "Microsoft.Authorization/policyAssignments",
              "Microsoft.Authorization/policyExemptions",
              "Microsoft.Authorization/locks"
            ]
          }
        ]
      },
      "then": {
        "effect": "[parameters('effect')]"
      }
    }
  }
}
```

### Why deployment resources are excluded

ARM, Bicep, and Terraform operations can create deployment-history resources even when their actual purpose is to update an existing resource. Blocking `Microsoft.Resources/deployments` can stop the deployment before Azure evaluates the individual exempt resource.

Excluding the deployment record does not automatically permit the resources within the deployment. Each child resource request is still subject to applicable Azure Policy evaluation.

### Why policy and lock resources are excluded

The governance team needs a controlled way to create, update, and remove policy assignments, exemptions, and locks. These operations should be protected with RBAC rather than made impossible through the freeze policy itself.

## 7. Create the tag protection policy

The frozen tag is the switch that activates enforcement. It must not be removable by normal environment contributors.

Azure Policy does not maintain a historical memory that says a resource group was previously frozen. A user removing the tag changes the final state being evaluated. The safest approach is therefore to maintain an explicit list of protected resource group IDs.

The following policy denies removal or alteration of the required frozen tag for resource groups listed in the assignment parameters.

```json
{
  "properties": {
    "displayName": "Protect the frozen state of selected resource groups",
    "policyType": "Custom",
    "mode": "All",
    "description": "Requires selected resource groups to retain the configured frozen environment tag and value.",
    "metadata": {
      "category": "Governance",
      "version": "1.0.0"
    },
    "parameters": {
      "protectedResourceGroupIds": {
        "type": "Array",
        "metadata": {
          "displayName": "Protected resource group IDs"
        }
      },
      "stateTagName": {
        "type": "String",
        "defaultValue": "EnvironmentState"
      },
      "frozenTagValue": {
        "type": "String",
        "defaultValue": "Frozen"
      },
      "effect": {
        "type": "String",
        "allowedValues": [
          "Audit",
          "Deny",
          "Disabled"
        ],
        "defaultValue": "Audit"
      }
    },
    "policyRule": {
      "if": {
        "allOf": [
          {
            "field": "type",
            "equals": "Microsoft.Resources/subscriptions/resourceGroups"
          },
          {
            "field": "id",
            "in": "[parameters('protectedResourceGroupIds')]"
          },
          {
            "anyOf": [
              {
                "field": "[concat('tags[', parameters('stateTagName'), ']')]",
                "exists": "false"
              },
              {
                "field": "[concat('tags[', parameters('stateTagName'), ']')]",
                "notEquals": "[parameters('frozenTagValue')]"
              }
            ]
          }
        ]
      },
      "then": {
        "effect": "[parameters('effect')]"
      }
    }
  }
}
```

Assign this definition at subscription or management-group scope and include only the resource groups that have completed the freeze process.

To unfreeze an environment, the governance team should first remove its resource group ID from the protected list or use a formally approved exemption.

## 8. Assign the main policy safely

Do not begin with `Deny` in a production subscription.

Recommended sequence:

1. Create the policy definition.
2. Assign it with the `Audit` effect.
3. Identify the resource groups that will be frozen.
4. Inventory all existing resources and dependencies.
5. Create resource-level exemptions.
6. Test representative update and deployment paths.
7. Change the assignment effect to `Deny`.
8. Apply the frozen tag.

Example Azure CLI variables:

```bash
subscriptionId="<SUBSCRIPTION_ID>"
resourceGroupName="<RESOURCE_GROUP_NAME>"
policyDefinitionName="deny-resource-changes-in-frozen-rgs"
policyAssignmentName="deny-resource-changes-in-frozen-rgs"

subscriptionScope="/subscriptions/${subscriptionId}"
resourceGroupScope="${subscriptionScope}/resourceGroups/${resourceGroupName}"

az account set --subscription "${subscriptionId}"
```

The simplest method is to create the definition in the Azure portal and paste the complete policy JSON shown earlier.

For Azure CLI, save only the contents of the `policyRule` object as `freeze-policy-rule.json`, and save only the contents of the `parameters` object as `freeze-policy-parameters.json`. Then run:

```bash
az policy definition create \
  --name "${policyDefinitionName}" \
  --display-name "Deny resource changes in frozen resource groups" \
  --description "Denies resource requests in frozen resource groups unless an exemption applies." \
  --mode All \
  --subscription "${subscriptionId}" \
  --rules @freeze-policy-rule.json \
  --params @freeze-policy-parameters.json
```

Assign the definition with the `Audit` parameter value before changing the effect to `Deny`.

## 9. Inventory and exempt existing resources

Policy exemptions are the critical part of this pattern. An exemption tells Azure Policy not to apply a particular assignment to the exemption scope.

### 9.1 Create an inventory

Use Azure Resource Graph, the Azure portal, or PowerShell to produce an approved inventory before enabling enforcement.

```powershell
Connect-AzAccount
Set-AzContext -SubscriptionId "<SUBSCRIPTION_ID>"

$ResourceGroupName = "<RESOURCE_GROUP_NAME>"
$InventoryPath = ".\${ResourceGroupName}-resource-inventory.csv"

$Resources = Get-AzResource -ResourceGroupName $ResourceGroupName |
    Sort-Object ResourceType, Name

$Resources |
    Select-Object Name, ResourceType, Location, ResourceId |
    Export-Csv -Path $InventoryPath -NoTypeInformation

$Resources |
    Select-Object Name, ResourceType, Location, ResourceId |
    Format-Table -AutoSize
```

Review the inventory with the application and platform owners. Remove resources that should not be retained rather than automatically exempting everything without review.

### 9.2 Retrieve the policy assignment

```powershell
$SubscriptionId = "<SUBSCRIPTION_ID>"
$PolicyAssignmentName = "deny-resource-changes-in-frozen-rgs"
$SubscriptionScope = "/subscriptions/$SubscriptionId"

$Assignment = Get-AzPolicyAssignment `
    -Name $PolicyAssignmentName `
    -Scope $SubscriptionScope
```

### 9.3 Create one exemption for each approved resource

```powershell
$ResourceGroupName = "<RESOURCE_GROUP_NAME>"
$Resources = Get-AzResource -ResourceGroupName $ResourceGroupName

foreach ($Resource in $Resources) {
    $InputBytes = [System.Text.Encoding]::UTF8.GetBytes($Resource.ResourceId)
    $Sha256 = [System.Security.Cryptography.SHA256]::Create()

    try {
        $HashBytes = $Sha256.ComputeHash($InputBytes)
    }
    finally {
        $Sha256.Dispose()
    }

    $ShortHash = (($HashBytes | ForEach-Object { $_.ToString("x2") }) -join "").Substring(0, 16)
    $ExemptionName = "existing-$ShortHash"

    New-AzPolicyExemption `
        -Name $ExemptionName `
        -DisplayName "Existing resource: $($Resource.Name)" `
        -Description "Approved existing resource retained when the environment was frozen." `
        -PolicyAssignment $Assignment `
        -Scope $Resource.ResourceId `
        -ExemptionCategory Mitigated
}
```

The hash-based exemption name avoids invalid characters and reduces the chance of duplicate exemption names.

### 9.4 Confirm the exemptions

```powershell
Get-AzPolicyExemption `
    -PolicyAssignmentIdFilter $Assignment.Id |
    Select-Object Name, DisplayName, Scope, ExemptionCategory |
    Sort-Object Scope |
    Format-Table -AutoSize
```

Export the final exemption list as evidence:

```powershell
Get-AzPolicyExemption `
    -PolicyAssignmentIdFilter $Assignment.Id |
    Select-Object Name, DisplayName, Scope, ExemptionCategory, ExpiresOn |
    Export-Csv ".\policy-exemptions.csv" -NoTypeInformation
```

### 9.5 Use the lowest practical exemption scope

Do not create an exemption at the resource-group scope. A resource-group exemption would exempt the entire group and allow new resources to be created, defeating the objective.

A resource-level exemption can also apply to resources below its scope. This matters for services with child resources, such as:

- Storage accounts and file shares.
- SQL servers and databases.
- API Management services and APIs.
- Key Vaults and child configuration objects.
- Network resources and nested configuration.

Exempt at the lowest manageable scope and test whether the service can create new child resources beneath an exempt parent. Where child-resource creation must also be prevented, separate child-resource governance or service-specific controls may be required.

## 10. Apply the frozen tag

After the inventory, exemptions, and validation are complete, apply the control tag to the resource group.

Azure CLI:

```bash
az tag update \
  --resource-id "${resourceGroupScope}" \
  --operation Merge \
  --tags EnvironmentState=Frozen
```

PowerShell:

```powershell
$ResourceGroup = Get-AzResourceGroup -Name "<RESOURCE_GROUP_NAME>"
$Tags = @{} + $ResourceGroup.Tags
$Tags["EnvironmentState"] = "Frozen"

Set-AzResourceGroup `
    -Name $ResourceGroup.ResourceGroupName `
    -Tag $Tags
```

Do not replace the complete tag collection accidentally. Preserve existing ownership, cost, application, and compliance tags when adding the frozen-state tag.

## 11. Validation tests

Perform both positive and negative tests.

### Test 1: Update an exempt existing resource

Make a harmless approved configuration change to an existing resource and then reverse it.

Expected result:

```text
Allowed because the resource is covered by a policy exemption.
```

### Test 2: Create a new resource

Attempt to deploy a low-cost test resource that is covered by the policy, such as a new virtual network or storage account.

Expected result:

```text
Denied by Azure Policy.
```

Remove any partially created deployment records after testing.

### Test 3: Deploy through ARM, Bicep, or Terraform

Run a deployment that changes only an exempt existing resource.

Expected result:

```text
The deployment record is allowed and the exempt resource update succeeds.
```

Then add a new resource to the same template or Terraform plan.

Expected result:

```text
The new resource request is denied.
```

### Test 4: Remove or change the frozen tag

Attempt to change:

```text
EnvironmentState = Frozen
```

to another value from a non-governance identity.

Expected result:

```text
Denied by the tag protection policy or blocked by the identity's RBAC permissions.
```

### Test 5: Create a child resource under an exempt parent

Test representative services that contain child resources.

Expected result:

```text
The result depends on the exemption scope and resource provider. Record the tested behaviour and add service-specific controls where required.
```

## 12. RBAC and operational protection

Azure Policy enforcement is only as strong as the permissions protecting its assignments and exemptions.

Normal application teams should not have permissions to:

- Delete or modify the freeze policy assignment.
- Create policy exemptions.
- Change the protected resource-group list.
- Remove the frozen state tag.
- Assign themselves broader access.

Recommended separation:

| Role | Responsibility |
| --- | --- |
| Application team | Operate approved existing resources |
| Platform team | Maintain landing-zone and shared platform controls |
| Governance team | Manage policy definitions, assignments, exemptions, and environment lifecycle state |
| Security operations | Monitor policy changes and unusual administrative activity |
| Break-glass administrator | Perform exceptional recovery under documented approval |

Use Azure Privileged Identity Management for high-impact roles and activate them only when required.

Remember that a `NotActions` entry in a custom role is not an explicit deny. If the same identity receives the excluded permission through another role assignment, the permission can still be available. Review all effective role assignments at management group, subscription, resource group, and resource scope.

## 13. Break-glass and controlled change

A frozen environment may still need an emergency repair, version upgrade, certificate replacement, or provider-generated child resource.

Use a documented process:

1. Record the business and technical reason.
2. Identify the exact resource or resource type affected.
3. Create a time-limited policy exemption at the narrowest scope.
4. Activate the required privileged role through PIM.
5. Perform and validate the change.
6. Remove or allow the exemption to expire.
7. Review the Activity Log and deployment evidence.
8. Update the approved inventory when the permanent resource baseline changes.

Example time-limited exemption:

```powershell
$Expiry = (Get-Date).ToUniversalTime().AddHours(4)

New-AzPolicyExemption `
    -Name "emergency-change-<CHANGE_ID>" `
    -DisplayName "Emergency change <CHANGE_ID>" `
    -Description "Temporary exemption approved for emergency remediation." `
    -PolicyAssignment $Assignment `
    -Scope "<RESOURCE_ID>" `
    -ExemptionCategory Waiver `
    -ExpiresOn $Expiry
```

Use `Mitigated` when another control addresses the policy objective. Use `Waiver` when the organisation has formally accepted a temporary or permanent exception.

## 14. Important limitations and edge cases

### 14.1 Azure Policy does not provide universal create-only detection

This design uses exemptions as the existing-resource allow-list. It should not be described as a native historical create-versus-update feature.

### 14.2 Exemptions are inherited below their scope

An exemption on a parent resource can affect child resources. This may permit new child-resource creation beneath an exempt parent. Test each important resource provider.

### 14.3 Some resource operations use additional resources

An update can create extension resources, deployment records, private endpoints, role assignments, diagnostic settings, managed identities, or provider-specific child objects. These may require separate approval and exemptions.

### 14.4 Data-plane operations are different from management-plane operations

Azure Policy primarily governs Azure Resource Manager requests. Application data-plane actions, such as writing blobs, querying a database, or updating content inside a service, are governed by that service's authentication, network, and authorization controls.

### 14.5 Existing automation may fail

Deployment pipelines, backup services, monitoring platforms, Defender plans, patching tools, and managed services may create or update resources automatically. Review service identities and expected write operations before enforcement.

### 14.6 Resource locks can still be useful

Add `CanNotDelete` locks to critical retained resources where accidental deletion is also a concern. Test service-specific behaviour before applying a `ReadOnly` lock.

### 14.7 Exemptions require lifecycle management

Every exemption should have:

- A named owner.
- A clear reason.
- An approval reference.
- A creation date.
- An expiry date where possible.
- A periodic review.

## 15. Recommended production rollout

| Phase | Action | Exit criteria |
| --- | --- | --- |
| 1. Discover | Identify candidate environments and business owners | Approved freeze scope |
| 2. Inventory | Export resources, child resources, identities, pipelines, and dependencies | Signed-off baseline |
| 3. Audit | Assign the policy with `Audit` | No unexplained findings |
| 4. Exempt | Create narrow exemptions for retained resources | Existing operations validated |
| 5. Protect | Configure tag protection and RBAC/PIM | Normal users cannot bypass the control |
| 6. Enforce | Change the main policy effect to `Deny` | New-resource tests are blocked |
| 7. Observe | Monitor Activity Log and Policy events | No critical service disruption |
| 8. Review | Revalidate exemptions and environment purpose | Regular governance review completed |

### Final design principles

- Use the tag to identify the environment state, not to prove resource age.
- Use policy exemptions as the approved existing-resource allow-list.
- Never exempt the complete resource group when the goal is to stop new resource creation.
- Protect assignments, exemptions, tags, and elevated roles through RBAC and PIM.
- Start in audit mode and test deployment pipelines before enforcement.
- Treat child resources and extension resources as separate design considerations.
- Maintain a break-glass process rather than disabling governance controls informally.

## 16. Reference links

- [Azure Policy overview](https://learn.microsoft.com/azure/governance/policy/overview)
- [Azure Policy deny effect](https://learn.microsoft.com/azure/governance/policy/concepts/effect-deny)
- [Evaluate the impact of a new Azure Policy definition](https://learn.microsoft.com/azure/governance/policy/concepts/evaluate-impact)
- [Azure Policy tag patterns](https://learn.microsoft.com/azure/governance/policy/samples/pattern-tags)
- [Azure Policy exemptions](https://learn.microsoft.com/azure/governance/policy/concepts/exemption-structure)
- [Azure CLI policy exemption commands](https://learn.microsoft.com/cli/azure/policy/exemption)
- [Lock Azure resources](https://learn.microsoft.com/azure/azure-resource-manager/management/lock-resources)
- [Azure RBAC role assignments](https://learn.microsoft.com/azure/role-based-access-control/role-assignments)

> **End state**
>
> The resource group is explicitly classified as frozen, existing approved resources remain manageable through narrow policy exemptions, and new resource deployment is denied unless the governance team follows the controlled exception process.
