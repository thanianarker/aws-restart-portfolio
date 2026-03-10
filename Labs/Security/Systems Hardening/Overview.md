# Systems Hardening with Patch Manager via AWS Systems Manager

## Overview

I completed a comprehensive hands-on lab on systems patching and vulnerability management using AWS Systems Manager's Patch Manager. Through this lab, I gained practical experience managing OS and application updates across heterogeneous infrastructure—patching both Linux and Windows instances at scale. I created custom patch baselines with approval rules, applied security policies to multiple instances using tag-based targeting, and verified compliance across a fleet. By the end of this lab, I demonstrated proficiency with enterprise-scale patch management, a critical component of security hardening and compliance in cloud infrastructure.

## Topics Covered

After completing this lab, I was able to:

- Use AWS Systems Manager Patch Manager for fleet-wide patch management
- Apply default patch baselines to Linux instances
- Create custom patch baselines with approval rules and severity filtering
- Configure auto-approval policies for patches based on classification and severity
- Use patch groups to organize instances by patch baselines
- Apply tags to EC2 instances for patch targeting
- Execute patch operations (scan and install) across multiple instances
- Configure reboot policies for patching operations
- Monitor patch operation progress and execution status
- Review patch compliance across fleets
- Verify which patches were applied and when
- Understand Systems Manager documents and Run Command integration
- Implement security hardening through systematic patch management

## Introducing the Technologies

### AWS Systems Manager and Patch Manager

AWS Systems Manager is a suite of tools for managing AWS resources and on-premises infrastructure. Patch Manager, a core capability, automates the patching process for OS and application updates across hundreds or thousands of instances.

| Systems Manager Component | Purpose | Use Case |
|---------------------------|---------|----------|
| **Fleet Manager** | Inventory and visibility of managed instances | View all instances under management |
| **Patch Manager** | Automated patching at scale | Apply security updates across fleets |
| **Run Command** | Execute commands on instances without SSH | Execute patch baselines on targets |
| **State Manager** | Maintain desired state on instances | Track patch operations and compliance |
| **Compliance** | Monitor configuration and patch compliance | Verify security posture across fleet |

### Patch Baseline Concepts

A patch baseline defines which patches are approved for deployment to instances. Baselines can be predefined (default) or custom.

| Baseline Type | Flexibility | Customization | Best For |
|---------------|-------------|---------------|----------|
| **Default Baseline** | Predefined, cannot modify | None—use as-is | Quick deployment, standard configurations |
| **Custom Baseline** | User-defined approval rules | Severity, classification, products | Specialized requirements, compliance policies |

### Patch Baseline Rules and Filtering

Custom baselines use approval rules to specify which patches to deploy based on multiple criteria:

| Rule Parameter | Purpose | Examples |
|----------------|---------|----------|
| **Products** | Operating systems/products to patch | Windows Server 2019, Amazon Linux 2 |
| **Severity** | Patch criticality level | Critical, Important, Medium, Low |
| **Classification** | Patch type/category | Security Updates, Bugfixes, Service Packs |
| **Auto-approval** | Days until patches auto-approve | 0 (immediate), 3, 7, 14, 30 days |
| **Compliance reporting** | Severity level for non-compliance | Critical, High, Medium, Low |

### Patch Groups and Instance Tagging

Patch groups associate instances with specific patch baselines through tags. This enables targeting instances for patch operations.

| Concept | Purpose | Implementation |
|---------|---------|-----------------|
| **Patch Group Tag** | Identifies which baseline applies to instance | EC2 tag: `Patch Group: LinuxProd` |
| **Tag Key** | Tag name | `Patch Group` |
| **Tag Value** | Baseline association | `LinuxProd`, `WindowsProd`, etc. |
| **Tag Targeting** | Patch operation selection method | Select instances via tags, not manually |

## Tasks

### Task 1: Patch Linux Instances Using Default Baselines

#### Step 1.1: Access Fleet Manager

I navigated to the AWS Management Console and searched for **Systems Manager**. I selected it to open the Systems Manager dashboard.

In the left navigation pane, I selected **Fleet Manager** under **Node Management** to view all managed instances.

I observed six pre-configured EC2 instances:
- **3 Linux instances** (Linux-1, Linux-2, Linux-3)
- **3 Windows instances** (Windows-1, Windows-2, Windows-3)

📝 **Note**: These instances were pre-configured with IAM roles that grant Systems Manager permissions, enabling them to be managed through Systems Manager without SSH or RDP access.

#### Step 1.2: View Instance Details

I selected the **Linux-1** instance and clicked **Node actions** → **View details** to examine its configuration.

The details page displayed:

| Detail | Value |
|--------|-------|
| Platform Type | EC2 |
| Node Type | Managed Node |
| OS Name | Amazon Linux 2 |
| IAM Role | Systems Manager role (pre-configured) |

I navigated back to the Systems Manager homepage.

#### Step 1.3: Access Patch Manager

In the left navigation pane, I selected **Patch Manager** under **Node Management** to access the patching interface.

#### Step 1.4: Patch Linux Instances

I clicked **Patch now** to begin patching the Linux instances.

I configured the patch operation:

| Setting | Value |
|---------|-------|
| Patching operation | Scan and install |
| Reboot option | Reboot if needed |
| Instances to patch | Patch only the target instances I specify |
| Target selection | Specify instance tags |
| Tag key | Patch Group |
| Tag value | LinuxProd |

💭 **Consider**: "Scan and install" performs a single operation that scans for available patches and immediately installs them. "Reboot if needed" ensures instances restart only if required by patches (e.g., kernel updates).

I clicked **Add** to add the tag, then **Patch now** to initiate the patching operation.

✅ **Task complete**: Patch operation initiated for Linux instances.

#### Step 1.5: Monitor Patch Progress

A status page appeared showing the patching operation progress. I observed:

- **AWS-PatchNowAssociation panel**: Shows the operation status and number of affected instances (3)
- **Scan/Install operation summary**: Displays visual progress for each instance

I monitored the page until all three Linux instances completed patching with status "Success."

⚠️ **Caution**: Patching operations can trigger reboots. In production, schedule patching during maintenance windows to minimize service disruption.

✅ **Task complete**: Linux instances successfully patched with default baseline.

### Task 2: Create Custom Patch Baseline for Windows Instances

#### Step 2.1: Access Patch Baselines

I navigated to Systems Manager and selected **Patch Manager** in the left navigation pane.

I clicked the **Patch baselines** tab to view all available baselines.

📝 **Note**: Default baselines for Amazon Linux, Ubuntu, Windows Server, and other OS types are pre-defined and read-only. Custom baselines provide the flexibility to implement organization-specific patch policies.

#### Step 2.2: Create New Custom Baseline

I clicked **Create patch baseline** to begin configuring a custom Windows patch baseline.

I configured the **Patch baseline details**:

| Setting | Value |
|---------|-------|
| Name | WindowsServerSecurityUpdates |
| Description | Windows security baseline patch |
| Operating System | Windows |
| Default patch baseline | Unchecked |

#### Step 2.3: Configure Approval Rules for Critical Patches

In the **Approval rules for operating systems** section, I added the first rule:

| Setting | Value |
|---------|-------|
| Products | Windows Server 2019 (deselect "All") |
| Severity | Critical |
| Classification | Security Updates |
| Auto-approval | 3 days |
| Compliance reporting | Critical |

💭 **Consider**: Setting auto-approval to 3 days allows testing time before patches are automatically approved. This balances security (critical patches applied quickly) with stability (time to verify patches don't break applications).

I clicked **Add rule** to add a second approval rule.

#### Step 2.4: Configure Approval Rules for Important Patches

I configured the second rule:

| Setting | Value |
|---------|-------|
| Products | Windows Server 2019 (deselect "All") |
| Severity | Important |
| Classification | Security Updates |
| Auto-approval | 3 days |
| Compliance reporting | High |

📝 **Note**: Separating rules by severity allows different approval timelines. Critical patches receive faster approval while Important patches undergo slightly longer testing.

I clicked **Create patch baseline** to finalize the baseline configuration.

✅ **Task complete**: Custom patch baseline created with security-focused approval rules.

#### Step 2.5: Create Patch Group Association

After creation, I selected the **WindowsServerSecurityUpdates** baseline from the list (it may appear on the second page).

I clicked **Actions** → **Modify patch groups** to associate the baseline with a patch group.

In the **Patch groups** field, I entered:

```
WindowsProd
```

I clicked **Add**, then **Close** to save the patch group association.

✅ **Task complete**: Patch group "WindowsProd" associated with the custom Windows baseline.

### Task 3: Patch Windows Instances

#### Step 3.1: Tag Windows Instances

I navigated to the **EC2** service and selected **Instances**.

I selected the **Windows-1** instance and clicked the **Tags** tab.

I clicked **Manage tags** → **Add new tag** and configured:

| Setting | Value |
|---------|-------|
| Key | Patch Group |
| Value | WindowsProd |

I clicked **Save** to apply the tag.

I repeated this process for **Windows-2** and **Windows-3** instances, applying the same tag to all three.

📝 **Note**: The Linux instances were pre-tagged with `Patch Group: LinuxProd` during lab setup. I tagged the Windows instances to enable patch group-based targeting.

✅ **Task complete**: All three Windows instances tagged with "Patch Group: WindowsProd".

#### Step 3.2: Patch Windows Instances

I navigated back to Systems Manager and selected **Patch Manager**.

I clicked **Patch now** to initiate Windows patching.

I configured the patch operation:

| Setting | Value |
|---------|-------|
| Patching operation | Scan and install |
| Reboot option | Reboot if needed |
| Instances to patch | Patch only the target instances I specify |
| Target selection | Specify instance tags |
| Tag key | Patch Group |
| Tag value | WindowsProd |

I clicked **Add**, then **Patch now** to start the patching operation.

#### Step 3.3: Monitor Patch Execution

A status page appeared showing patch operation progress. When the **Execution ID** link became available, I clicked it to view detailed execution information.

The page opened in the **State Manager** section, showing the execution status for each Windows instance.

I clicked **Output** for one of the instances to view detailed patching logs.

The **Run Command** output displayed patch operation details, including:

```
PatchGroup: WindowsProd
[Patch operation details and status messages]
[List of patches found and installed]
```

⚠️ **Caution**: Run Command output shows verbose operation details. Scrolling through confirms patches were evaluated and applied according to the WindowsServerSecurityUpdates baseline rules.

✅ **Task complete**: Windows instances patched using custom baseline.

### Task 4: Verify Patch Compliance

#### Step 4.1: Review Compliance Summary

I navigated to **Patch Manager** and clicked the **Dashboard** tab.

Under **Compliance summary**, I observed:

```
Compliant: 6
```

This confirmed all six instances (3 Linux + 3 Windows) were compliant with their respective patch baselines.

💭 **Consider**: Compliance status indicates instances match their baseline configuration. Non-compliant status would indicate missing patches that should be approved by the baseline rules.

#### Step 4.2: Review Compliance Details

I clicked the **Compliance reporting** tab to view detailed compliance information for each instance.

The page displayed a table with all instances:

| Column | Content |
|--------|---------|
| Node ID | Instance identifier |
| Compliance status | Compliant/Non-compliant |
| Platform | Linux/Windows |
| Baseline ID | Associated patch baseline |
| Last operation date | Timestamp of last patch operation |

I scrolled right to view additional columns:

| Column | Content |
|--------|---------|
| Critical noncompliant count | Unapplied critical patches |
| Security noncompliant count | Unapplied security patches |
| Other noncompliant count | Unapplied non-security patches |

All instances showed **0** noncompliant patches.

#### Step 4.3: View Applied Patches for Instance

I clicked on a **Windows instance** Node ID to view its details.

The node details page opened. I clicked the **Patch** tab to view the patching history.

The page displayed a list of patches applied to the instance:

| Patch Information | Details |
|------------------|---------|
| Patch ID | Microsoft patch identifier (e.g., KB5012345) |
| Title | Patch name/description |
| Classification | Type (Security Update, Bugfix, etc.) |
| Severity | Critical, Important, Medium, Low |
| Installed Time | Timestamp when patch was applied |
| State | Installed, Available, Failed |

✅ **Task complete**: Successfully verified compliance across all instances and reviewed applied patches.

📝 **Note**: The **Installed Time** column shows when patches were applied, providing audit trail information. This is critical for compliance documentation and security investigations.

## Patch Management Workflow Summary

The complete patch management process I executed:

1. **Define Baselines**: Created custom baselines with approval rules
2. **Assign Patch Groups**: Associated baselines with patch group tags
3. **Tag Instances**: Applied patch group tags to EC2 instances
4. **Execute Patching**: Ran patch operations targeting instances by tag
5. **Monitor Operations**: Tracked patch operation progress in real-time
6. **Verify Compliance**: Confirmed all instances matched baseline requirements
7. **Audit Results**: Reviewed which patches were applied and when

This workflow enables organizations to:
- **Scale operations** across hundreds of instances
- **Enforce policies** consistently across environments
- **Reduce risk** through systematic security updates
- **Maintain compliance** with regulatory requirements
- **Audit changes** for security investigations

## Conclusion

I successfully demonstrated comprehensive systems hardening through patch management:

- ✅ Patched Linux instances using AWS-provided default patch baselines
- ✅ Created a custom Windows patch baseline with security-focused approval rules
- ✅ Configured multiple approval rules based on severity and classification
- ✅ Associated patch baselines with patch groups
- ✅ Tagged EC2 instances for patch group targeting
- ✅ Executed patch operations (scan and install) across instances
- ✅ Configured automatic reboots where needed during patching
- ✅ Monitored patch operation progress and execution details
- ✅ Verified compliance across all instances (6/6 compliant)
- ✅ Reviewed detailed patch compliance reporting
- ✅ Inspected applied patches, classifications, and installation timestamps
- ✅ Understood Systems Manager integration with Run Command and State Manager

This lab demonstrated enterprise-scale patch management, a critical security control for maintaining infrastructure hardening, compliance, and vulnerability remediation across heterogeneous fleets.

## Additional Resources

- [AWS Systems Manager Documentation](https://docs.aws.amazon.com/systems-manager/)
- [Patch Manager User Guide](https://docs.aws.amazon.com/systems-manager/latest/userguide/patch-manager.html)
- [Creating Patch Baselines](https://docs.aws.amazon.com/systems-manager/latest/userguide/patch-manager-baselines.html)
- [Patch Groups Documentation](https://docs.aws.amazon.com/systems-manager/latest/userguide/patch-manager-patch-groups.html)
- [Patch Manager Best Practices](https://docs.aws.amazon.com/systems-manager/latest/userguide/patch-manager-best-practices.html)
- [Systems Manager Run Command](https://docs.aws.amazon.com/systems-manager/latest/userguide/run-command.html)
- [State Manager Documentation](https://docs.aws.amazon.com/systems-manager/latest/userguide/state-manager.html)
- [EC2 Instance Tagging Guide](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/Using_Tags.html)
- [AWS Systems Manager SSM Documents](https://docs.aws.amazon.com/systems-manager/latest/userguide/what-is-ssm-doc.html)
- [Compliance as Code with Systems Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/compliance.html)
