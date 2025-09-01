# EC2-Auto-Recovery-with-CloudWatch-Alarms
This project demonstrates how to create a CloudWatch alarm that automatically recovers an EC2 instance when it becomes impaired due to a system-level failure (e.g., hardware issues, underlying hypervisor problems).  This is crucial for DevOps, SysOps, and FinOps workflows that require maximum uptime and self-healing systems.

# Learning Objectives
	•	Understand CloudWatch metrics related to EC2 health.
	•	Use the AWS Console and CLI to set up auto-recovery alarms.
	•	Familiarize yourself with high-availability strategies for mission-critical workloads.
	•	Practice Infrastructure-as-Code mindset via terminal output.

 # Prerequisites
	•	AWS CLI configured (aws configure)
	•	IAM user or role with:
	•	cloudwatch:PutMetricAlarm
	•	ec2:DescribeInstances
	•	ec2:RecoverInstances
	•	An EC2 instance in the running state
	•	Billing alerts enabled in your AWS account (optional for CloudWatch practice)

# Step 1: Identify Your EC2 Instance
 aws ec2 describe-instances \
  --filters Name=instance-state-name,Values=running \
  --query "Reservations[].Instances[].[InstanceId, State.Name, AvailabilityZone]" \
  --output table

# Step 2: Create the CloudWatch Alarm
aws cloudwatch put-metric-alarm \
  --alarm-name "EC2-AutoRecovery-MyInstance" \
  --alarm-description "Auto-recovery alarm for EC2 instance health" \
  --metric-name StatusCheckFailed_System \
  --namespace AWS/EC2 \
  --statistic Minimum \
  --period 60 \
  --threshold 1 \
  --comparison-operator GreaterThanOrEqualToThreshold \
  --dimensions Name=InstanceId,Value=i-0123456789abcdef0 \
  --evaluation-periods 2 \
  --alarm-actions arn:aws:automate:us-east-1:ec2:recover \
  --unit Count

  # Step 3: Confirm Your Alarm Exists
  aws cloudwatch describe-alarms \
  --alarm-names "EC2-AutoRecovery-MyInstance" \
  --output table

This alarm will now monitor the StatusCheckFailed_System metric. If the value is 1 for two consecutive minutes, AWS will attempt to recover the instance.

# Optional Testing (with caution)
To simulate a failure, you would need to stop the instance abruptly (not gracefully). This can be done using fault injection tools like AWS Fault Injection Simulator or by contacting AWS Support to simulate underlying failures.

# Outputs
ALARM CREATED:
Name: EC2-AutoRecovery-MyInstance
Status: OK
Recovery Action: Automated via CloudWatch Alarm
Target Instance: i-0123456789abcdef0
Region: us-east-1

# Concepts and Best Practices
	•	StatusCheckFailed_System triggers when there’s an issue not related to your code or OS, such as hardware or hypervisor failure.
	•	Auto recovery will launch your instance on new hardware while preserving your instance metadata (ID, EBS volumes, IP, etc).
	•	This works only for EBS-backed instances in a single AZ.
	•	Combine with application-level health checks (like Route53 health checks or ECS self-healing) for full resiliency.


# Estimated Time
	•	Reading / Theory: 20 minutes
	•	Terminal Walkthrough: 10–15 minutes
	•	Dashboard Confirmation & Notes: 10 minutes

Total Estimated Time: ~45 minutes

# References & Resources
	•	AWS Docs - Recover Your Instance: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-recover.html
	•	AWS CLI - CloudWatch Alarms: https://docs.aws.amazon.com/cli/latest/reference/cloudwatch/put-metric-alarm.html
	•	CloudWatch Metrics Reference: https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/aws-services-cloudwatch-metrics.html
 
