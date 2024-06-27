## Overview

RapidRadar is an organization-wide Cloud Security and Cost management solution. It's designed to identify security threats for AWS Resources and automatically take action to remediate them, detect unused AWS resources and enables AWS Inspector and AWS GuardDuty across all accounts and regions of your AWS Organization. With real-time reporting capabilities, RapidRadar leverages popular Notification Channels such as **Slack**, **Microsoft Teams**, and **Google Chat** to provide engineers with immediate insights and feedback.

Furthermore, it offers integration with **Tailscale** to track known IPs of AWS Users and generates detailed cost reports on a daily, weekly, and monthly basis for all active resources attributed to AWS SSO Users. To bolster its capabilities, RapidRadar seamlessly shares findings with Azure Cloud for in-depth investigation and analysis, enhancing your organization's overall cloud security and cost management strategy.

As part of its commitment to incident response, RapidRadar also integrates with **PagerDuty**, automatically creating incidents for identified security findings. This feature ensures that your team is promptly alerted and can take swift action to address and resolve security incidents, further strengthening your organization's security posture.

---

## Functionalities

- [Configuration Management Database (CMDB)](#configuration-management-database-cmdb)
- [IAM KeyPair Access Tracker](#iam-keypair-access-tracker)
- [IP Tracker](#ip-tracker)
- [Post Deploy SSM Automation](#post-deploy-ssm-automation)

---

## Features

- [AWS Security Reference Architecture Integration](#aws-security-reference-architecture-integration)
- [Unused AWS Resources Detection](#unused-aws-resources-detection)
- [AWS Resources Report Generator for Deleted/Disabled SSO Users](#user-offboarding-workflow)
- [AWS Resources Auto-Tagger](#aws-resources-auto-tagger)
- [Policy-Driven Resource Creation Controls](#policy-driven-resource-creation-controls)
- [Seamless AWS Systems Manager Host Configuration Enabler](#seamless-aws-systems-manager-host-configuration-enabler)
- [VPC Flow Logs Tagging Automation](#vpc-flow-logs-tagging-automation)
- [Threat Detection](#threat-detection)
- [Auto Remediation of Threats](#auto-remediation-of-threats)
- [Real-time Notification Capability + PagerDuty Integration + Azure Cloud Support](#real-time-notification-capability--pagerduty-integration--azure-cloud-support)
- [Hourly Reminders](#hourly-reminders)
- [Alerts Suppression](#alerts-suppression)
- [Tailored Alerts for Unique Needs](#tailored-alerts-for-unique-needs)
- [Multi-Account and Organizational Unit Deployment](#multi-account-and-organizational-unit-deployment)

---

## How to Deploy

#### Pre-Requisites

- Install AWSCLI, yq, rain-cli and docker
- Configure AWS profiles along with regions for management and automation accounts
- Create SES Identites for Email Addresses that are to be used for `SenderEmailAddress` and `ReceiverEmailAddressess`
- Under **env/** folder, the yml files contain all the [environment variables](env/README.md#rapidradar-environment-variables). Set them according to your requirements.

#### Run Script

Run **rapidradar.sh** script to start deployment of RapidRadar Solution

```
./rapidradar.sh create
```

---

## Functionalities Details

### Configuration Management Database (CMDB)

The CMDB functionality within RapidRadar is a powerful tool designed to capture and manage metadata for all active AWS resources. When enabled (`AddSupportforCMDB: true`), it provides detailed insights into the infrastructure, aiding in resource tracking, analysis, and cost reporting. To leverage the CMDB functionality, make sure to enable CMDB by setting `AddSupportforCMDB` to `true` in the configuration **env/common.yml**

#### Supported AWS Resource Types:

- EC2 Instances
- EKS Clusters
- EFS Filesystems
- RDS Clusters and Instances

#### Captured Metadata:

- **Resource ID:** Uniquely identifies each AWS resource.
- **Resource Type:** Indicates the type of AWS resource (e.g., EC2 Instance, EKS cluster, EFS filesystem, RDS Cluster/Instance).
- **Resource State:** Specifies whether the resource is currently pending, running or stopped.
- **Public IP:** Captures Public IP of the resource if it has any.
- **Deploy Method:** Captures the deployment method through the **DeployMethod** tag, revealing how the resource was deployed.
- **Team:** Assigns a team identifier based on the **Team** tag, facilitating team-specific resource categorization.
- **Deployed By:** Identifies the creator of the resource, sourced from CloudTrail logs or the **DeployedBy** tag.
- **Creation Date and Time:** Records the timestamp of when the resource was created.
- **Launch Date and Time:** Records the timestamp of when the resource was started.
- **Platform:** Indicates the platform type for resource.
- **Is SSM Managed:** Specifies whether the resource (e.g. EC2 Instance) is SSM managed or not.
- **Is IMDSv2 Enabled:** Specifies whether IMDSv2 is enabled for the resource.
- **Tags:** Collects all tags associated with the resource, providing additional context.
- **Cost Type:** Identifies the cost type for the resource (on-demand, reserved, etc)
- **Daily, Monthly and Hourly Cost:** Forecasts daily, monthly and hourly cost of the resource.

#### Dynamically Updated DynamoDB Tables:

- Active Resources Table:<br>Contains metadata for all currently active AWS resources.
- Deleted Resources Table:<br>Stores metadata for resources that have been deleted.

#### Daily, Weekly, and Monthly Cost Reports:

The CMDB functionality also plays a crucial role in generating cost reports. These reports are sent to the specified email address on a daily, weekly, and monthly basis. Key features of the cost reports include:

- Cost Fluctuation Notification:<br>Indicates whether the overall cost has increased or decreased, along with the percentage change.

#### Auto-Tagging and SCP Integration:

- Auto-Tagging Requirements:<br>If CMDB is enabled, resources supported by CMDB must have the following tags if auto-tagging is enabled for that resource type:
  - DeployedBy
  - DeployMethod
  - Team
    These tags are crucial for proper resource categorization and alignment with organizational policies.
- SCP Integration:<br>If SCP is enabled for a specific resource type, resources of that type must have the following tags, or else the creation will be blocked:
  - DeployedBy
  - DeployMethod
  - Team

### IAM KeyPair Access Tracker

The IAM KeyPair Access Tracker functionality within RapidRadar is a comprehensive tool designed to monitor and manage secret-access keypairs associated with IAM users. When enabled (`AddSupportforIAMKeyPairAccessTracker: true`), it captures metadata, activities, and usage details for enhanced security and access management and ensures that any access from a new or unfamiliar IP address triggers an immediate security alert, swiftly notifying relevant users via the chosen Notification Channel. To leverage the IAM KeyPair Access Tracker functionality, make sure to enable it by setting `AddSupportforIAMKeyPairAccessTracker` to `true` in the configuration **env/common.yml**

#### Captured Metadata:

- **Access Key ID:** A unique identifier for each secret-access keypair.
- **IAM User Association:** Identifies the IAM user, the access keypair is attached to.
- **Status:** Indicates whether the access keypair is active or inactive.
- **Creation Date:** Records the timestamp of when the access keypair was created.
- **Last Used Date:** Captures the timestamp of the last usage of the access keypair.
- **Key Activity:** Captures IP addresses, user agents, and timestamps when keypairs are used.

#### Dynamically Updated DynamoDB Tables:

- IAM Keypair Access Tracker Table:<br>Contains metadata for all secret-access keypairs, including associations, status, creation date, and last usage. Stores key activities, capturing IP addresses, user agents, and timestamps when access keypairs are used.

#### Threat Detection and Alerting:

The IAM KeyPair Access Tracker functionality actively detects new access keypair activities. Upon detecting a new usage, it triggers an alert to notify users. The alert includes details such as IP address, user agent, and timestamp, empowering users to promptly address and validate the legitimacy of the keypair usage.

### IP Tracker

The IP Tracker functionality within RapidRadar is designed to monitor and correlate IP addresses associated with Single Sign-On (SSO) users. When enabled (`AddSupportforIPTracker: true`), it captures IP addresses, user details, and usage history for enhanced security and access monitoring. This historical IP data is invaluable when tracking login attempts, especially for the root user. By continuously tracking and preserving this IP address history, RapidRadar enhances your organization's security posture by swiftly detecting any new or unauthorized login attempts for the root user. Any deviation from established login patterns triggers immediate security alerts, ensuring that your AWS environment remains fortified against potential security breaches. To leverage the IP Tracker functionality, make sure to enable it by setting `AddSupportforIPTracker` to `true` in the configuration **env/common.yml**

#### Tailscale Support:

If Tailscale support is enabled, the machine IP address is also stored for each SSO user. To leverage this support, make sure to enable it by setting `TrackTailscaleIPs` to `true` in the configuration [env/common.yml](env/README.md#tailscale-configuration)

#### Captured Metadata:

- **SSO User ID:** A unique identifier for each SSO user.
- **SSO Username:** The username of the SSO user.
- **IP Address:** Captures the IP address when an SSO user logs in.
- **Last Login Date:** Records the timestamp of the last login for each SSO user.
- **TailScale IP:** Captures the machine IP address from tailscale for each SSO user.

#### Dynamically Updated DynamoDB Tables:

- IP Correlation Table:<br>Contains correlated data, including SSO user ID, username, IP address, and last login date along with tailscale machine IP address.
- SSO Users IP History Table:<br>Stores historical IP addresses usage for all SSO users.

#### Threat Detection and Alerting:

The IP Tracker functionality actively detects root logins and password change attempts/requests for SSO users. By referencing the IP history in the SSO Users IP History DynamoDB table, RapidRadar verifies the legitimacy of login attempts. If an attempt is made, an alert is triggered, notifying users to take notice and validate the attempt.

### Post Deploy SSM Automation

RapidRadar introduces this powerful feature designed to streamline the automation and management of EC2 instances. When this functionality is enabled, RapidRadar takes care of various tasks to ensure seamless integration with AWS Systems Manager (SSM). To leverage this feature, enable and configure it according to your requirements in [env/post-deploy-ssm-automation.yml](env/README.md#envpost-deploy-ssm-automationyml).

#### Key Features:

- IAM Role Management:
  - **Automated IAM Role Creation:** If no existing IAM role name is provided in `ExistingEC2SSMIAMRoleName`, RapidRadar creates an IAM role with the necessary policies for SSM and attaches it with EC2 instance if no IAM Role is already attached to the instance on launch.
  - **Missing Policy Attachment:** For instances with an existing IAM role, RapidRadar checks for required managed policies (**AmazonSSMManagedInstanceCore**, **AmazonSSMManagedEC2InstanceDefaultPolicy**) along with the user-defined additional policies (ARNs) in `NamesOfAdditionalPoliciesToAutoAttach`. These policies are automatically attached to the EC2 Instance's IAM role if missing.
- VPC Endpoint Management:
  - **Automated VPC Endpoint Creation:** RapidRadar ensures that the EC2 instances have the essential VPC endpoints for SSM functionality (`com.amazonaws.[region].ssm`, `com.amazonaws.[region].ssmmessages`, `com.amazonaws.[region].ec2messages`) along with the user-defined additional VPC endpoints in `VPCEndpointsList` patameter.
- IMDSv2 Enablement:
  - **Auto Enable IMDSv2:** If enabled by setting `AutoEnableEC2InstancesIMDSv2` to `true`, RapidRadar enables IMDSv2 on the EC2 instances using AWS Systems Manager State Manager after successful launch.

---

## Features Details

### AWS Security Reference Architecture Integration

RapidRadar is designed to leverage best practices outlined in the AWS Security Reference Architecture. By enabling AWS Inspector and GuardDuty, along with enforcing EBS default encryption, RapidRadar aligns with the principles of this reference architecture to enhance the security posture of your AWS environment.

#### AWS Security Reference Architecture Principles:

- Defense in Depth:<br>RapidRadar reinforces defense in depth by implementing AWS Inspector for security assessments, GuardDuty for continuous threat detection and ensuring EBS default encryption for enhanced data protection.
- Automated Security Best Practices:<br>RapidRadar automates the activation and configuration of AWS Inspector, GuardDuty and enables EBS default encryption across all AWS accounts and regions, ensuring consistent adherence to automated security best practices.
- Continuous Monitoring and Detection:<br>Leveraging GuardDuty, RapidRadar enables continuous monitoring and detection of security threats, aligning with the AWS Security Reference Architecture's emphasis on continuous security monitoring.
- Centralized Configuration Management:<br>Inspector, GuardDuty settings and EBS default encryption are centrally managed through RapidRadar, facilitating a centralized configuration approach recommended by the AWS Security Reference Architecture.

#### Enables AWS GuardDuty

AWS GuardDuty is an intelligent threat detection service that helps you protect your AWS environment from security threats. RapidRadar solution delegates the AWS Account you provide as `GuardDutyAdminAccountId` parameter in [env/sra.yml](env/sra.yml) as an administrator for AWS GuardDuty and enables it across all accounts and regions for comprehensive threat detection and security monitoring. If value for this parameter is not provided, then AWS GuardDuty is enabled in each target account separately. Enable AWS GuardDuty by setting `EnableGuardDuty` to `true` in the configuration [env/sra.yml](env/README.md#guardduty-configuration)

- Automated GuardDuty Activation:<br>RapidRadar ensures the automatic activation of GuardDuty, saving you the manual effort of enabling it individually across accounts and regions.
- Centralized Configuration:<br>GuardDuty settings are centrally managed through RapidRadar, simplifying configuration and ensuring consistent security policies.
- Continuous Threat Detection:<br>GuardDuty actively monitors your AWS environment for potential security threats.

#### Enables AWS Inspector

Amazon Inspector is a security assessment service provided by Amazon Web Services (AWS). It helps you discover and identify security vulnerabilities in your AWS resources and applications. RapidRadar solution delegates the AWS Account you provide as `InspectorAdminAccountId` parameter in [env/sra.yml](env/sra.yml) as an administrator for AWS Inspector and enables it across all accounts and regions for comprehensive security assessment coverage for your entire AWS environment. If value for this parameter is not provided, then AWS Inspector is enabled in each target account separately. Enable AWS Inspector by setting `EnableInspector2` to `true` in the configuration [env/sra.yml](env/README.md#inspector-configuration)

- Automated AWS Inspector Activation:<br>RapidRadar automates the activation of AWS Inspector across all AWS accounts and regions, ensuring comprehensive security assessments.
- Centralized Configuration:<br>AWS Inspector settings are centrally managed through RapidRadar, providing a single point of control for security assessments.
- Continuous Vulnerability Identification:<br>AWS Inspector continuously identifies and reports security vulnerabilities, enhancing your defense against potential threats.

#### Enables Default EBS Encryption

RapidRadar facilitates the automatic activation and configuration of default EBS encryption across all AWS accounts and regions, enforcing the encryption of the new EBS volumes and snapshot copies that you create. Enable this by setting `EnableEBSDefaultEncryption` to `true` in the configuration [env/sra.yml](env/README.md#ebs-default-encryption-configuration)

- Automated EBS Encryption Activation:<br>RapidRadar ensures the automatic activation of default EBS encryption for enhanced data protection.
- Centralized Configuration:<br>Centrally manages EBS default encryption settings through RapidRadar, maintaining a unified configuration approach.

### Unused AWS Resources Detection

RapidRadar is equipped with a comprehensive Unused AWS Resources Detection feature that actively monitors and identifies underutilized resources across all accounts and regions within your AWS Organization. This functionality aims to streamline resource management and optimize your AWS environment by detecting and prompting action on various unused resources.

#### Detected Resources and Actions:

- **Stopped EC2 Instances:**
  - RapidRadar identifies EC2 instances in a stopped state for extended periods.
  - Sends a series of action reminders:
    - 1st reminder after 5 days of detection.
    - 2nd reminder after 5 days of the 1st reminder if no action is taken.
    - 3rd reminder after 13 days of the 2nd reminder.
    - Terminates unused EC2 instances after creating an AMI and sends the AMI ID in the 4th alert, triggered 1 day after the 3rd reminder.
- **Over-Permissive IAM Role Policies for EC2 Instances:**
  - Detects and addresses IAM roles attached to EC2 instances with over-permissive policies (those containing '\*' in the Resource block).
  - Deletes or detaches over-permissive policies to enhance security.
- **Launch Wizard Security Groups:**
  - Identifies and removes launch-wizard security groups that are not attached to any resource, promoting clean and efficient security group management.
- **Available EBS Volumes:**
  - Notifies concerned users via email and Notification Channel about EBS volumes that are not attached to any resources, facilitating proactive resource cleanup.

#### Action Workflow:

- Reminder Notifications:<br>Sends action reminders with specified timelines to the concerned users, urging them to take necessary actions.
- Automated Cleanup:<br>Initiates resource cleanup if no action is taken within the designated timeline.
  For EC2 instances, creates an AMI before termination, providing users with a snapshot of the resource.

### User Offboarding Workflow

RapidRadar offers a powerful feature to streamline incident response in the event of the deletion or disabling of an AWS Single Sign-On (SSO) User. The AWS Resources Report Generator compiles a comprehensive report containing details of all active AWS Resources historically created by the affected AWS SSO User. This report is then seamlessly delivered via email and accompanied by a Notification Channel notification, ensuring that concerned users are promptly and effectively notified. To leverage the User Offboarding Workflow functionality, make sure to enable it by setting `AddSupportforUserOffboardingWorkflow` to `true` in the configuration **env/common.yml**

#### Key Features:

- **Comprehensive Report Compilation:** RapidRadar compiles a detailed report containing information on all active AWS Resources historically created by the deleted or disabled AWS SSO User.
- **Notification Channels Integration:** Delivers the compiled report through Notification Channels, ensuring instant and accessible notifications to relevant stakeholders.
- **Email Notification:** Sends the report via email, providing users with a convenient and easily accessible format for reviewing the information.

#### Incident Response Workflow:

- Deletion or Disabling of AWS SSO User:<br>RapidRadar initiates the incident response process upon detecting the deletion or disabling of an AWS SSO User.
- Report Compilation:<br>Generates a detailed report of all active AWS Resources associated with the affected AWS SSO User.
- Notification Delivery:<br>Sends the compiled report via email and Notification Channels, facilitating timely and efficient communication with concerned users.

### AWS Resources Auto-Tagger

RapidRadar introduces the powerful AWS Resources Auto-Tagger feature, providing users with the capability to automatically tag both existing and newly provisioned resources within your AWS infrastructure. This feature allows users to define and customize tags during the solution's deployment process, ensuring efficient resource categorization and management. To leverage the AWS Resources Auto-Tagger feature, enable and provide tags of your choosing in **env/auto-tagger.yml**

#### Key Features:

- **Automated Tagging Process:** RapidRadar automates the tagging process for various AWS Resources, eliminating the need for manual tagging efforts.
- **Tagging with SSM Templates for Terraform Deployments:** RapidRadar provides the capability to create tag templates in SSM Parameter Store for each resource type in all accounts and regions. Users can define tag keys for template generation using `*TagKeysForTagTemplateGeneration` for each of the resource type that RapidRadar supports if `Tag*UsingTagTemplateForTerraformDeployment` for that particular resource type is enable by setting it to `true`.
  - **NOTE: This key feature is applicable only for new resources and does not apply to existing ones.**
  - When a new resource is deployed using **Terraform** or has a **DeployMethod** tag with the value **Terraform**, RapidRadar retrieves tags from the SSM Parameter Store template.
  - The template is used only if it has been modified with user-specified values, ensuring dynamic and customizable tag generation.
  - Useful in scenarios where resources are deployed by multiple team members, allowing for consistent tagging practices across projects.
- **User-Defined Tags:** Empowers users to define and customize tags according to their organizational needs during the deployment of the RapidRadar solution.
- **Consistent Resource Categorization:** Ensures consistent and uniform tagging across AWS Resources, promoting efficient resource identification and management.
- **Resource Coverage:** Currently covers auto-tagging for the following AWS Resources:
  - [VPCs](env/README.md#auto-tagger-configuration-for-vpc)
  - [Subnets](env/README.md#auto-tagger-configuration-for-subnets)
  - [EBS Volumes](env/README.md#auto-tagger-configuration-for-ebs-volumes)
  - [Elastic IPs](env/README.md#auto-tagger-configuration-for-elastic-ips)
  - [EBS Snapshots](env/README.md#auto-tagger-configuration-for-ebs-snapshots)
  - [AMIs](env/README.md#auto-tagger-configuration-for-amis)
  - [EC2 Instances](env/README.md#auto-tagger-configuration-for-ec2-instances)
  - [EKS Clusters](env/README.md#auto-tagger-configuration-for-eks-clusters)
  - [RDS Clusters and Instances](env/README.md#auto-tagger-configuration-for-rds-clusters-and-instances)
  - [Elastic FileSystems](env/README.md#auto-tagger-configuration-for-elastic-filesystems)
  - [FSX FileSystems](env/README.md#auto-tagger-configuration-for-fsx-filesystems)
  - [SecretsManager Secrets](env/README.md#auto-tagger-configuration-for-secretsmanager-secrets)
  - [Backup Plans](env/README.md#auto-tagger-configuration-for-backup-plans)
  - [LoadBalancers](env/README.md#auto-tagger-configuration-for-loadbalancers)

#### Benefits:

- Organizational Alignment:<br>Facilitates alignment with organizational policies by consistently applying user-defined tags to resources, ensuring adherence to tagging standards.
- Greater Visibility and Control:<br>Enhances visibility and control over AWS resources by promoting a standardized approach to resource tagging.
- Compliance Assurance:<br>Promotes compliance with organizational tagging policies, contributing to a well-organized and compliant cloud environment.

#### Sending Missing Tags Notifications

RapidRadar also provides the capability to send notifications for missing tags on AWS Resources. Users can define the tags for each resource type using `TagsKeyValue*` if `SendMissingTagsNotification*` parameter is set to `true` in the `env/auto-tagger.yml` file.<br>
**Note: If the auto-tagger is enabled for a particular resource type, sending missing tags notifications cannot be enabled for that resource type.**

### Policy-Driven Resource Creation Controls

RapidRadar provides robust support for enforcing compliance and security measures by leveraging AWS Service Control Policies (SCPs). This feature ensures that AWS resources are created with adherence to specified conditions, and if conditions are not met, the creation is blocked. RapidRadar also sends alerts to Notification Channels and emails to the concerned user, providing clear insights into why the creation was blocked. To configure and leverage the SCP functionality in RapidRadar, specify the required tagging conditions in the **env/scp.yml** file.

#### Supported Scenarios:

- [EC2 Instance Tagging](env/README.md#block-ec2-launch-without-certain-tags): Blocks EC2 instance creation if launched without specified tags.
  - Configure using `BlockEC2LaunchWithoutCertainTags` set to `true` and specify comma-separated list of tag keys in `EC2InstanceLaunchSCPTagKeys`.
- [EKS Cluster Tagging](<(env/README.md#block-eks-cluster-creation-without-certain-tags)>): Blocks EKS cluster creation without specific tags.
  - Configure using `BlockEksClusterCreationWithoutCertainTags` set to `true` and specify comma-separated list of tag keys in `EksClusterCreationSCPTagKeys`.
- [RDS Cluster or Instance Tagging](env/README.md#block-rds-clusterinstance-creation-without-certain-tags): If an RDS cluster or instance is launched without prescribed tags, SCP blocks the creation.
  - Configure using `BlockRdsClusterInstanceCreationWithoutCertainTags` set to `true` and specify comma-separated list of tag keys in `RdsClusterInstanceCreationSCPTagKeys`.
- [EFS Filesystem Tagging](env/README.md#block-efs-filesystem-creation-without-certain-tags): Blocks EFS filesystem creation without specific tags.
  - Configure using `BlockEfsFileSystemCreationWithoutCertainTags` set to `true` and specify comma-separated list of tag keys in `EfsFileSystemCreationSCPTagKeys`.
- [IMDSv2 Enforcement](env/README.md#block-ec2-instance-launch-if-imdsv2-is-disabled): Blocks EC2 instance creation without IMDSv2 enabled.
  - Configure using `BlockEC2LaunchWithoutIMDSV2` set to `true` with the bypass tag key in `SCPBypassTagKeyEC2LaunchWithoutIMDSV2`. This bypass tag comes in play when there's a specific need to launch an EC2 Instance without enabling IMDSv2.
- [Public IP on EC2 Instances](env/README.md#block-ec2-instance-launch-with-public-ip): Blocks EC2 instance creation if launched with a public IP.
  - Configure using `BlockEC2LaunchWithPublicIP` set to `true` with the bypass tag key in `SCPBypassTagKeyEC2LaunchWithPublicIP`. This bypass tag comes in play when there's a specific need to launch an EC2 Instance with Public IP.
- [EBS Volume Encryption](env/README.md#block-unencrypted-ebs-volume-creation): Blocks EBS volume creation without encryption, including root volumes.
  - Configure using `BlockUnencryptedEBSVolumeCreation` set to `true` with the bypass tag key in `SCPBypassTagKeyEBSVolumeCreation`. This bypass tag comes in play when there's a specific need to create an unencrypted EBS Volume or launch an EC2 Instance with unencrypted Root EBS Volume.
- [Load Balancer Bypass Tag](env/README.md#block-loadbalancer-creation): Blocks load balancer creation without the required bypass tag.
  - Configure using `BlockLoadBalancerCreation` set to `true` with the bypass tag key in `SCPBypassTagKeyLoadBalancerCreation`. This bypass tag comes in play when there's a need to create a LoadBalancer.
- [Elastic IP Bypass Tag](env/README.md#block-elastic-ip-allocation): Blocks Elastic IP allocation without the necessary bypass tag.
  - Configure using `BlockEIPAllocation` set to `true` with the bypass tag key in `SCPBypassTagKeyEIPAllocation`. This bypass tag comes in play when there's a need to allocate an Elastic IP.
- [RDS Encryption Enforcement](env/README.md#block-rds-clusterinstance-creation-if-encryption-is-disabled): Blocks RDS cluster or instance creation without encryption.
  - Configure using `BlockUnencryptedRDSCreation` set to `true` with the bypass tag key in `SCPBypassTagKeyUnencryptedRDSCreation`. This bypass tag comes in play when there's a specific need to create RDS Cluster or Instance without encryption enabled.
- [IAM Users Bypass Tag](env/README.md#block-iam-user-creation): Blocks IAM user creation without the bypass tag.
  - Configure using `BlockIAMUsersCreation` set to `true` with the bypass tag key in `SCPBypassTagKeyIAMUsersCreation`. This bypass tag comes in play when there's a need to create IAM Users.
- [Public EBS Snapshots](env/README.md#block-making-ebs-snapshots-public): Blocks actions attempting to make EBS snapshots public.
  - Configure using `BlockMakeEBSSnapshotPublic` set to `true`.

#### AWS Service Actions Handling:

SCPs do not affect any service-linked role. Service-linked roles enable other AWS services to integrate with AWS Organizations and can't be restricted by SCPs. So, if an AWS service performs one of the mentioned actions, RapidRadar handles it based on the `AWSServiceDeploymentAction` parameter specified in **env/scp.yml**:

- **Delete:** Deletes the resource created by the AWS service.
- **Ignore:** Ignores the action, and no further action is taken.
  RapidRadar also sends alerts to chosen Notification Channel, notifying users when an AWS service has taken an action, providing a **10-minute** window to take corrective measures. If no action is taken within the specified timeframe, the resource is automatically deleted.

### Seamless AWS Systems Manager Host Configuration Enabler

RapidRadar automates the activation and configuration of Default Host Management Configuration across your AWS accounts and regions within your AWS Organization saving you the manual effort of enabling it individually in each. This feature empowers AWS Systems Manager to automatically manage your Amazon EC2 instances as managed instances which streamlines operational tasks, allowing seamless execution of commands, resource inventory tracking, and configuration management.

### VPC Flow Logs Tagging Automation

RapidRadar simplifies the process of tagging your Virtual Private Clouds (VPCs) for Flow Logs. This feature allows users to specify the desired tag key and value through the `VpcFlowLogsTagKeyValue` parameter in the **env/common.yml** file. RapidRadar ensures that both existing and new VPCs are appropriately tagged, streamlining the configuration needed for Flow Logs in your AWS environment.

### Threat Detection

RapidRadar's threat detection functionality is a proactive shield for your AWS infrastructure, systematically identifying and evaluating potential security vulnerabilities, misconfigurations, and anomalous activities. Here's how RapidRadar ensures the security of your AWS environment:

#### Key Features:

- **Comprehensive Security Assessment:**<br>RapidRadar conducts a thorough assessment of your AWS infrastructure, continuously scanning for potential security threats. This includes identifying vulnerabilities, misconfigurations, and any anomalous activities that may pose a risk to your environment.
- **Swift Action on Detected Threats:**<br>Upon detecting a potential threat, RapidRadar acts swiftly to notify users, ensuring that you are promptly informed about the nature of the threat. Notifications are sent through both email and your chosen Notification Channel (Slack, Microsoft Teams, Google Chat).
- **PagerDuty Integration for Incident Management:**<br>With PagerDuty integration enabled (`AddSupportforPagerDuty`: `true`), RapidRadar takes incident response to the next level. It creates incidents on PagerDuty, providing a centralized platform for managing and responding to security incidents.
- **Incorporating PagerDuty Incident Numbers in Alerts:**<br>Alerts sent out to Notification Channels and email include PagerDuty incident numbers along with clickable links to the corresponding incidents. This seamless integration enhances incident traceability and expedites incident resolution.
- **Hourly Reminder Alerts:**<br>RapidRadar offers a unique feature where users can receive hourly reminders for specific severity types. Users have the flexibility to specify the severity types for which they want hourly reminders (`SendHourlyAlertsForSeverityTypes`), including options such as `CRITICAL`, `HIGH`, `MEDIUM`, `LOW`, `ALL`, or a combination of multiple values. Supported Reminders:
  - Security Groups
  - IAM Users
  - S3 Buckets

#### Threat Detection Alerts and Severity Levels

This table provides a concise overview of the alerts RapidRadar supports, their associated severity levels, and the corresponding PagerDuty urgency for effective incident response and prioritization.
| Alert Type | Severity Level | PagerDuty Urgency |
|------------------------|------------------------------------------|----------------------------|
| Root User - Password Change Attempt (Failure/Success) | CRITICAL | HIGH |
| Root User - Successful Login | CRITICAL | HIGH |
| Root User - 3 Failed Login Attempts in 5 minutes (Brute Force Attack) | CRITICAL | HIGH |
| Security Group - All Traffic Ingress Port Open to 0.0.0.0/0 (Not Attached) | LOW | LOW |
| Security Group - Ingress Port (Remote) Open to 0.0.0.0/0 (Not Attached) | LOW | LOW |
| Security Group - Ingress Port (Traffic) Open to 0.0.0.0/0 (Not Attached) | LOW | LOW |
| Security Group - Ingress Port (Other) Open to 0.0.0.0/0 (Not Attached) | LOW | LOW |
| Security Group - All Traffic Ingress Port Open to 0.0.0.0/0 (Attached to Public EC2 Instance or internet-facing LoadBalancer) | CRITICAL | HIGH |
| Security Group - Ingress Port (Remote) Open to 0.0.0.0/0 (Attached to Public EC2 Instance or internet-facing LoadBalancer) | CRITICAL | HIGH |
| Security Group - Ingress Port (Traffic) Open to 0.0.0.0/0 (Attached to Public EC2 Instance or internet-facing LoadBalancer) | HIGH | LOW |
| Security Group - Ingress Port (Other) Open to 0.0.0.0/0 (Attached to Public EC2 Instance or internet-facing LoadBalancer) | LOW | LOW |
| Security Group - All Traffic Ingress Port Open to 0.0.0.0/0 (Attached to Other Resource) | MEDIUM | LOW |
| Security Group - Ingress Port (Remote) Open to 0.0.0.0/0 (Attached to Other Resource) | MEDIUM | LOW |
| Security Group - Ingress Port (Traffic) Open to 0.0.0.0/0 (Attached to Other Resource) | LOW | LOW |
| Security Group - Ingress Port (Other) Open to 0.0.0.0/0 (Attached to Other Resource) | LOW | LOW |
| Security Group - CRITICAL/HIGH Ingress Port Auto-Closed | INFO | N/A |
| Security Group - All Rules with Ingress Ports open to 0.0.0.0/0 closed/deleted (Finding Remediated) | INFO | N/A |
| EC2 Instance - Launch Blocked (Unencrypted Root Volume) | INFO | N/A |
| EC2 Instance - Launch Blocked (Disabled IMDSv2) | INFO | N/A |
| EC2 Instance - Launch Blocked (Public IP Associated) | INFO | N/A |
| EC2 Instance - Launched with disabled IMDSv2 without bypass tag | HIGH | LOW |
| EC2 Instance - Launched with Public IP without bypass tag | HIGH | LOW |
| EC2 Instance - Launched with unencrypted Root Volume | HIGH | LOW |
| EC2 Instance - IAM Role is disassociated with (`AutoAttachIAMRoleEC2`: `true`) (Finding Remediated) | INFO | N/A |
| EC2 Instance - Required policy is detached with (`AutoAttachMissingPolicies`: `true`) (Finding Remediated) | INFO | N/A |
| EC2 Instance - Attached IAM Role has an over-permissive policy and Auto-Remediation is enabled (Finding Remediated) | INFO | N/A |
| EC2 Instance - Attached IAM Role with bypass tag key has an over-permissive policy | LOW | LOW |
| EC2 Instance - Attached IAM Role has an over-permissive policy and Auto-Remediation is not enabled | CRITICAL | HIGH |
| EC2 Instance - Replaced Launch Wizard Security Group when Remediation is enabled | LOW | LOW |
| EC2 Instance - Launched without required tags | LOW | LOW |
| EC2 Instance - Launched by AWS Service (e.g. EKS Cluster) | CRITICAL | HIGH |
| AMI - Created without required tags | LOW | LOW |
| AMI - Made public and Auto-Remediation is enabled (Finding Remediated) | INFO | N/A |
| AMI - Made public and Auto-Remediation is not enabled | CRITICAL | HIGH |
| EBS Volume - Creation Blocked (Unencrypted Volume) | INFO | N/A |
| EBS Volume - Created without encryption enabled | HIGH | LOW |
| EBS Volume - Created without required tags | LOW | LOW |
| EBS Volume - Created by AWS Service | HIGH | LOW |
| EBS Snapshot - Created without required tags | LOW | LOW |
| EBS Snapshot - Making public Blocked | INFO | N/A |
| EBS Snapshot - Made public and Auto-Remediation is enabled (Finding Remediated) | INFO | N/A |
| EBS Snapshot - Made public and Auto-Remediation is not enabled | CRITICAL | HIGH |
| Elastic IP - Allocated without required tags | LOW | LOW |
| Elastic IP - Allocation Blocked (without bypass tag) | INFO | N/A |
| Elastic IP - Associated to a Resource which does not have `EIPAssociationOverrideTagKey` tag key present | LOW | LOW |
| Elastic IP - Created by AWS Service | HIGH | LOW |
| VPC - Created without required tags | LOW | LOW |
| Subnet - Created without required tags | LOW | LOW |
| IAM User - Password Change Attempt (Failure/Success) | CRITICAL | HIGH |
| IAM User - Creation Blocked | INFO | N/A |
| IAM User - Created without bypass tag | MEDIUM | LOW |
| IAM User - Deleted (Finding Remediated) | INFO | N/A |
| IAM User - Created Secret-Access Keypair | HIGH | LOW |
| IAM User - Created Login Profile | HIGH | LOW |
| IAM User - Made Active Secret-Access Keypair Inactive after `DeactivateUnusedSecretAccessKeypairInDays` number of days | INFO | N/A |
| IAM User - 3 Failed Login Attempts in 5 minutes (Brute Force Attack) | CRITICAL | HIGH |
| IAM User - Detection of Unknown Activity using Secret-Access KeyPair | HIGH | LOW |
| S3 Account Block Public Access Setting - If disabled and Auto-Remediation is enabled (Finding Remediated) | INFO | N/A |
| S3 Account Block Public Access Setting - If disabled and Auto-Remediation is not enabled | CRITICAL | HIGH |
| S3 Bucket - Made public using ACL or Bucket Policy | CRITICAL | HIGH |
| S3 Bucket - Objects made public using object ACL | CRITICAL | HIGH |
| S3 Bucket - Deleted (Finding Remediated) | INFO | N/A |
| S3 Bucket - Made private at Bucket and Object level (Finding Remediated) | INFO | N/A |
| LoadBalancer - Creation Blocked (without bypass tag) | INFO | N/A |
| LoadBalancer - Created without required tags | LOW | LOW |
| LoadBalancer - Created by AWS Service | HIGH | LOW |
| EKS - Created Cluster without required tags | LOW | LOW |
| RDS - Cluster or Instance Creation Blocked (Unencrypted Storage) | INFO | N/A |
| RDS - Created Cluster or Instance without encryption enabled | HIGH | LOW |
| RDS - Created Cluster or Instance without required tags | LOW | LOW |
| RDS - Snapshot made public and Auto-Remediation is enabled (Finding Remediated) | INFO | N/A |
| RDS - Snapshot made public and Auto-Remediation is not enabled | CRITICAL | HIGH |
| Backup - Created Plan without required tags | LOW | LOW |
| EFS - Created FileSystem without required tags | LOW | LOW |
| FSx - Created FileSystem without required tags | LOW | LOW |
| SecretsManager - Created Secret without required tags | LOW | LOW |

### Auto Remediation of Threats

In addition to its robust threat detection capabilities, RapidRadar elevates your cloud security strategy by offering automated remediation for identified security threats. When a potential security threat is detected within your AWS Resources, RapidRadar doesn't stop at merely alerting you. Instead, it takes decisive action to remediate the threat, restoring your security posture swiftly and effectively. Once the automated remediation has been successfully executed, RapidRadar ensures that you are promptly informed of the threat's detection and resolution. Notifications are sent through email and your chosen Notification Channels, offering peace of mind that the security threat has been proactively addressed. To configure and leverage this feature in RapidRadar, configure it in the **env/auto-remediation.yml** file. The following threats/findings are currently supported and automatically remediated if enabled:

- **Security Group Ingress Rules:**
  - All Traffic Port Open to 0.0.0.0/0:<br>If the All Traffic port is open to `0.0.0.0/0` in the ingress rules of a security group, and that security group is attached to either a public EC2 instance or an internet-facing load balancer, the security group rule is deleted.
    - Configure using `AllTrafficSGRulesRemediation` set to `true`.
  - [Remote Access Ports](#advanced-security-group-configuration) Open to 0.0.0.0/0:<br>If a remote access port is open to `0.0.0.0/0` in the ingress rules of a security group, and that security group is attached to either a public EC2 instance or an internet-facing load balancer, the security group rule is closed to the IP address of the user who created this rule. If no IP is found, the rule is deleted.
    - Configure using `RemoteAccessPortsSGRulesRemediation` set to `true`.
  - [Traffic Ports](#advanced-security-group-configuration) Open to 0.0.0.0/0:<br>If a traffic port is open to `0.0.0.0/0` in the ingress rules of a security group, and that security group is attached to either a public EC2 instance or an internet-facing load balancer, the security group rule is closed to the IP address of the user who created this rule. If no IP is found, the rule is deleted.
    - Configure using `TrafficPortsSGRulesRemediation` set to `true`.
- **Launch Wizard Security Group:**<br>If a launch wizard security group is created, it is automatically deleted. If it is attached to an EC2 instance or a load balancer without any other security groups attached, it is replaced with a blackhole security group (with no ingress) and then deleted. If there are other security groups attached, the launch wizard security group is deleted straightaway. If found to be attached to some other resource except for EC2 instance and load balancer, an alert is sent indicating detachment without deletion due to attachments to other resources.
  - Configure using `LaunchWizardSGRemediation` set to `true`.
- **S3 Account Public Access Block Disabled:**<br>If the S3 account public access block is disabled, it is automatically enabled again, and alerts are sent to chosen notification channels.
  - Configure using `S3AccountPublicBlockRemediation` set to `true`.
- **EBS Snapshot Made Public:**<br>If an EBS snapshot is made public, it is automatically made private, and notifications are sent.
  - Configure using `EBSPublicSnapshotsRemediation` set to `true`.
- **AMI Made Public:**<br>If an AMI is made public, it is automatically made private, and notifications are sent.
  - Configure using `PublicAMIsRemediation` set to `true`.
- **RDS Snapshot Made Public:**<br>If an RDS snapshot is made public, it is automatically made private, and notifications are sent.
  - Configure using `PublicRDSSnapshotsRemediation` set to `true`.
- **Overpermissive Policies Attached to IAM Role:**<br>If overpermissive policies are attached to an IAM role and that IAM role is attached to an EC2 instance, RapidRadar automatically detaches managed policies or deletes inline policies, sending notifications. Overpermissive is defined as the **AdministratorAccess** managed policy or if resource is '\*' in an inline policy.
  - Configure using `AttachedOverPermissiveRolesRemediation` set to `true`.
- **Inactive Secret Access Keypair:**<br>RapidRadar automatically makes an inactive secret access keypair if not used for a specified period.
  - Configured using `UnusedSecretAccessKeypairRemediation` set to `true` with number of days in `DeactivateUnusedSecretAccessKeypairInDays` parameter in the **env/auto-remediation.yml** file.

### Real-time Notification Capability + PagerDuty Integration + Azure Cloud Support

RapidRadar features an advanced real-time notification capability, ensuring users receive immediate alerts across various events and findings in their AWS environment. Whether it's security findings, cost management insights, or automated actions, RapidRadar ensures that users stay informed and can take immediate action when necessary. The solution supports three popular notification channels currently and this can be configured in **env/common.yml** under **Alerts Config** section:

- Slack
- Microsoft Teams
- Google Chat

#### Key Features:

- **Multi-Channel Support:** RapidRadar seamlessly integrates with Slack, Microsoft Teams, and Google Chat, providing users with the flexibility to choose their preferred notification platform.
- **Immediate Alerts:** Receive real-time alerts on critical events, such as security threats detected by AWS services, cost fluctuations, or automated remediation actions taken by RapidRadar.
- **Incident Reporting with PagerDuty:** In addition to standard notifications, RapidRadar supports PagerDuty integration. This includes the creation of incidents for further investigation and response in the case of security incidents or findings. This support can be leveraged by setting `AddSupportforPagerDuty` to `true` in **env/common.yml**
  - Users can specify the severity types for which PagerDuty incidents should be created using the `CreateIncidentForSeverityTypes` parameter. Supported values include `CRITICAL`, `HIGH`, `MEDIUM`, `LOW`, `ALL`, or a combination of multiple values. This customization allows users to focus PagerDuty incident creation on the severity levels most crucial to their operations.
- **Azure Cloud Log Analytics Integration:** RapidRadar sends every detection and finding to Azure Cloud Log Analytics, ensuring a comprehensive log of activities and events in your Azure environment for in-depth investigation and analysis. To leverage this support, make sure to enable it by setting `AddSupportforAzureCloud` to `true` in the configuration **env/common.yml**
- **Automated Remediation Actions:** RapidRadar not only alerts you to security threats but also automates remediation actions, ensuring a proactive defense posture in your AWS environment.

### Alerts Suppression

RapidRadar introduces the powerful Alerts Suppression feature, providing users with the ability to selectively silence alerts for specific resources within their AWS infrastructure. This feature enables users to define a tag key-value pair, specified in the `AlertSuppressionResourceTagKeyValue` parameter in the **env/common.yml** file, to indicate which resources should have alerts suppressed.

#### How it Works:

- **Tagging Resources:** Users can tag resources with a specific key-value pair. If this key-value pair matches the one specified in `AlertSuppressionResourceTagKeyValue`, alerts for that resource will be suppressed.
- **Permission Set Verification:** To ensure security and control, the alerts suppression feature works based on the user's permission set. If the user tagging the resource is logged in with a permission set that has the tag key-value specified in `AlertSuppressionPermissionSetTagKeyValue`, the suppression takes effect. Otherwise, alerts are not suppressed, and the standard alerting mechanism continues.

#### Benefits:

- Selective Suppression:<br>Silences alerts only for resources with the specified tag key-value pair, providing granular control over alerting.
- Enhanced Security:<br>Ensures that the user suppressing alerts has the necessary permission set, adding an extra layer of security and preventing misuse.
- Flexible Configuration:<br>Users can customize the tag key-value pair, allowing adaptability to various tagging strategies within their organization.

### Tailored Alerts for Unique Needs

RapidRadar recognizes that AWS environments are diverse, and users may have unique requirements for alerting configurations. To cater to these specific needs, RapidRadar introduces the "Tailored Alerts for Unique Needs" feature which allows users to personalize alert settings for various scenarios, providing flexibility and adaptability to diverse AWS architectures. These tailored configurations empower users to customize alerts according to their specific AWS architecture and security requirements. Adjust the provided parameters in the configuration file **env/alerts-customization** to match your unique needs and scenarios.

#### Advanced Security Group Configuration

Tailor your security group alerting based on specific port configurations:

- [EC2 Instance Security Groups](env/README.md#security-groups-for-ec2-instances):
  - Remote Access Ports:<br>Define remote access ports that, when open, trigger alerts. Remote access ports can be specified using `EC2SecurityGroupIngressRemoteAccessPorts`. These are typically ports used for remote management, such as SSH (22) or RDP (3389).
  - Traffic Ports:<br>Specify traffic ports that, when open, trigger alerts. Traffic ports can be configured using `EC2SecurityGroupIngressTrafficPorts` and usually refer to ports associated with application traffic.
  - Ignore Ports:<br>Designate ports to be ignored for alerting with `EC2SecurityGroupIngressIgnorePorts`. Ports specified here will not trigger alerts.
- [LoadBalancer Security Groups](env/README.md#security-groups-for-loadbalancers):
  - Remote Access Ports:<br>Set remote access ports for load balancer security groups using `LoadBalancerSecurityGroupIngressRemoteAccessPorts`. Similar to EC2 instances, these are ports used for remote management.
  - Traffic Ports:<br>Customize traffic ports with `LoadBalancerSecurityGroupIngressTrafficPorts`. These ports are associated with application traffic.
  - Ignore Ports:<br>Exclude specific ports from triggering alerts using `LoadBalancerSecurityGroupIngressIgnorePorts`.

#### Elastic IP Association Alerts

Control alerting for Elastic IP association based on specific tag requirements:

- [Tag Key for Alerting](env/README.md#elastic-ip-association): Specify the tag key that, if absent on the resource to which Elastic IP is being associated, triggers an alert using `EIPAssociationOverrideTagKey`.

#### EC2 Instance IAM Role Detection Bypass

Tailor IAM role alerting based on specific IAM role configurations:

- [Bypass Detection Tag Key](env/README.md#ec2-instance-iam-role-detection): Define a tag key that, when present on an IAM role attached to an EC2 instance, bypasses detection for over-permissive policy changes using `EC2InstanceIAMRoleDetectionBypassTagKey`.

### Multi-Account and Organizational Unit Deployment

RapidRadar offers robust support for deploying across multiple AWS accounts and Organizational Units (OUs), providing flexibility and scalability for various deployment scenarios. Whether you're deploying across multiple accounts or within specific OUs within your AWS Organization, RapidRadar streamlines the deployment process while ensuring consistency and adherence to organizational policies.

#### Deployment Options:

- **Deployment Targets:** Specify the deployment targets using the `DeploymentTargets` parameter in the **env/common.yml** file. You can specify comma-separated OU or account IDs, or simply designate the root OU.
- **Exclude Accounts:** Utilize the `ExcludeAccounts` parameter to exclude specific accounts when deploying at the OU level, whether as child or root OUs.

#### Key Features:

- **Multi-Account Deployment:** RapidRadar facilitates deployment across multiple AWS accounts, allowing organizations to manage resources efficiently across different environments and teams.
- **Organizational Unit Deployment:** With support for deployment within specific OUs, RapidRadar enables targeted deployments tailored to organizational structures, ensuring resources are provisioned in alignment with hierarchical requirements.
- **Root OU Deployment:** For organizations deploying resources at the root level of their AWS Organization, RapidRadar simplifies the process, providing a centralized approach to resource management and governance.

#### Benefits:

- **Scalability and Flexibility:** RapidRadar's multi-account and OU deployment capabilities scale to meet the needs of organizations of all sizes, offering flexibility in resource provisioning and management.
- **Consistency and Compliance:** By enforcing deployment standards across accounts and OUs, RapidRadar promotes consistency and compliance with organizational policies and security best practices.
- **Centralized Management:** Organizations can centrally manage deployments across multiple accounts and OUs, streamlining resource provisioning and governance processes.
