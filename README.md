Overview
This project designs and implements an AWS Lambda function that automatically deletes EC2 snapshots older than one year. The infrastructure is defined using Terraform, ensuring reproducibility and Infrastructure as Code (IaC).
****************************************************
Architecture Components:

1. VPC and Subnet
•	A custom VPC is created with at least one private subnet.
•	The Lambda function is configured to run inside this VPC.
•	Security groups and subnet IDs are passed to the Lambda configuration.

2. IAM Role and Policies
•	IAM role: lambda_snapshot_cleanup_role.
•	Trust policy allows Lambda service to assume the role.
•	Attached policies include:
o	SnapshotCleanup: ec2:DescribeSnapshots, ec2:DeleteSnapshot.
o	Logs: logs:CreateLogGroup, logs:CreateLogStream, logs:PutLogEvents.
o	LambdaVPCAccess: permissions for managing ENIs (ec2:CreateNetworkInterface, etc.).

4. Lambda Function;
•	Written in Python.
•	Uses boto3 to connect to EC2.
•	Logic:
o	Retrieve all snapshots owned by the account.
o	Calculate cutoff date (365 days ago).
o	Identify snapshots older than cutoff.
o	Attempt deletion and log results.
o	Handle errors gracefully.

6. CloudWatch EventBridge Rule
•	Scheduled rule (e.g., rate(1 day)) triggers the Lambda daily.
•	Target: the snapshot cleanup Lambda function.

7. Monitoring;
   
•	CloudWatch Logs capture Lambda execution logs.
•	CloudWatch Metrics track invocations, errors, and duration.
•	Optional alarms can be configured for failures.
********************************************************************************
Deployment;

Packaging Lambda Code;

•	Lambda requires a .zip deployment package.
•	Terraform archive_file data source can package lambda_function.py automatically:
•	data "archive_file" "lambda_zip" {
•	  type        = "zip"
•	  source_file = "${path.module}/lambda/lambda_function.py"
•	  output_path = "${path.module}/lambda.zip"
}
Terraform Apply
•	Initialize and apply:
•	terraform init
terraform apply
•	This provisions VPC, IAM role, Lambda, and EventBridge rule.
******************************************************************************
Tool Choice
•	Terraform chosen for modularity, state management, and reproducibility.

Execution Steps
1.	Clone repository.
2.	Run terraform init.
3.	Run terraform apply.

Deployment Steps;

•	Terraform packages and deploys Lambda automatically.
•	Lambda runs inside VPC using subnet and security group IDs.

Assumptions;
•	AWS region: us-east-1 (can be modified).
•	Snapshots owned by the same account.

Monitoring;
•	Use CloudWatch Logs to view Lambda execution.
•	Use CloudWatch Metrics for performance and error tracking.
****************************************************************

Key Takeaways
- Automated cleanup of old EC2 snapshots saves costs and improves hygiene.
- Terraform ensures reproducible infrastructure.
- IAM role provides least privilege access.
- CloudWatch EventBridge enables scheduled automation.
- Monitoring ensures visibility and reliability.











