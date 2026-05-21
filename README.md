# Automatically Disable Active Directory Users on GuardDuty Findings

This repo hosts the CloudFormation template accompanying the AWS Security Blog post "[Automating identity lifecycle and security with AWS Directory Service APIs](https://aws.amazon.com/blogs/security/automating-identity-lifecycle-and-security-with-aws-directory-service-apis/)" published on the [AWS Security Blog](https://aws.amazon.com/blogs/security/) channel.

> TODO: Replace the post link above with the direct blog post URL once it is published.

## Overview
This solution automatically disables an [AWS Managed Microsoft AD](https://docs.aws.amazon.com/directoryservice/latest/admin-guide/directory_microsoft_ad.html) user account when [Amazon GuardDuty](https://aws.amazon.com/guardduty/) detects suspicious runtime activity on an EC2 instance joined to the directory. It reduces the time between detection and containment by removing the human step of mapping a Linux process owner back to a directory identity. The solution helps you:
- **Contain threats faster**: Disable the impacted user in Active Directory within seconds of a matching GuardDuty finding, without waiting for an on-call responder to investigate.
- **Map process activity to a directory identity**: A Systems Manager Run Command document resolves the Linux effective user ID from the finding to the underlying AD `sAMAccountName`.
- **Keep responders in the loop**: An email notification is sent through Amazon SNS whenever a user is disabled, including the username, source IP, and the full CloudTrail event for auditing.

![Architectural Diagram](/Images/ArchitecturalDiagram.png)

The solution uses [Amazon EventBridge](https://aws.amazon.com/eventbridge/) to match GuardDuty findings, [AWS Step Functions](https://aws.amazon.com/step-functions/) to orchestrate the response, [AWS Systems Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/what-is-systems-manager.html) Run Command and Automation to resolve the username and call the [AWS Directory Service Data API](https://docs.aws.amazon.com/directoryservicedata/latest/DirectoryServiceDataAPIReference/Welcome.html) `DisableUser` action, and [Amazon SNS](https://aws.amazon.com/sns/) with an [AWS KMS](https://aws.amazon.com/kms/) customer managed key for encrypted notifications.

## Deployment
### CloudFormation
### Prerequisites
To deploy the solution you need the following:
* An active AWS account with permissions to deploy CloudFormation stacks
* [Amazon GuardDuty](https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_settingup.html) enabled in the target Region, with [Runtime Monitoring](https://docs.aws.amazon.com/guardduty/latest/ug/runtime-monitoring.html) enabled for EC2
* An [AWS Managed Microsoft AD](https://docs.aws.amazon.com/directoryservice/latest/admin-guide/directory_microsoft_ad.html) directory containing the user accounts you want to protect
* One or more Linux EC2 instances joined to that directory and managed by [AWS Systems Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-setting-up.html) (SSM Agent installed and reporting)
* An email address that can receive SNS notifications

### Getting Started
1. Download the deployment template. For this solution we use the [guardduty-ad-user-auto-disable.yaml](/Templates/CloudFormation/guardduty-ad-user-auto-disable.yaml) CloudFormation template.
2. [Create a stack from the AWS CloudFormation Console](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-console-create-stack.html). The CloudFormation template will create all the resources described in the blog post.
3. Confirm the SNS subscription email sent to the address you provided so you receive notifications when a user is disabled.

### Template Parameters
The stack template includes the following parameters:

| Parameter | Required | Description |
| --- | --- | --- |
| DirectoryID | Required | The AWS Managed Microsoft AD directory ID where users will be disabled. Must match the pattern `d-xxxxxxxxxx`. |
| NotificationEmail | Required | Email address that will receive an SNS notification each time a user is disabled in Active Directory. |

### Cleanup
To clean up resources:
1. Open the AWS CloudFormation console.
2. Select the stack you created.
3. Choose **Delete stack**.
4. Confirm the deletion.

Alternatively, use the AWS CLI:
```bash
aws cloudformation delete-stack --stack-name <your-stack-name>
```

Any AD users that were disabled by the automation will remain disabled after the stack is deleted. Re-enable them from the directory if needed.

## Security

See [CONTRIBUTING](CONTRIBUTING.md#security-issue-notifications) for more information.

## License

This library is licensed under the MIT-0 License. See the LICENSE file.
