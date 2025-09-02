# 🛠️ EC2 Auto-Recovery with CloudWatch Alarms

**Author**: FrostyWorks
**Last Updated**: 2025-09-01  
**Tags**: AWS, EC2, CloudWatch, High Availability, Auto-Recovery, DevOps, SysOps

---

## 📘 Overview

This guide shows you how to set up a **CloudWatch Alarm** that automatically recovers an **Amazon EC2 instance** when a system-level issue occurs. This is a key high-availability strategy in AWS environments and a valuable hands-on skill for DevOps and SysOps engineers.

---

## 🎯 Learning Objectives

- Understand how EC2 system status checks work
- Use CloudWatch to monitor `StatusCheckFailed_System`
- Automatically trigger EC2 recovery using CloudWatch alarm actions
- Gain experience with AWS CLI and CloudWatch configurations
- Learn principles of fault-tolerant, self-healing architecture

---

## 🔧 Prerequisites

Make sure you have the following:

- AWS CLI installed and configured:
  ```bash
  aws configure
  ```
- IAM role or user permissions:
  - `ec2:DescribeInstances`
  - `cloudwatch:PutMetricAlarm`
  - `ec2:RecoverInstances`
- An existing **running EC2 instance**
- EC2 must be **EBS-backed** and in a **single AZ**

---

## 🚀 Setup Guide

### Step 1: Identify Your Instance

Run the following to get your instance ID:

```bash
aws ec2 describe-instances \
  --filters Name=instance-state-name,Values=running \
  --query "Reservations[].Instances[].[InstanceId, State.Name, Placement.AvailabilityZone]" \
  --output table
```

Copy the correct `InstanceId` for use in the next step.

---

### Step 2: Create the Auto-Recovery CloudWatch Alarm

Replace `i-0123456789abcdef0` with your actual instance ID:

```bash
aws cloudwatch put-metric-alarm \
  --alarm-name "EC2-AutoRecovery-MyInstance" \
  --alarm-description "Auto-recovery alarm for EC2 system status failure" \
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
```

Note: Change the region code in the ARN if your instance is not in `us-east-1`.

---

### Step 3: Confirm Alarm Was Created

```bash
aws cloudwatch describe-alarms \
  --alarm-names "EC2-AutoRecovery-MyInstance" \
  --output table
```

Expected output should show alarm state as `OK` and action as `ec2:recover`.

---

## 🧪 Optional: Simulate a Failure

To test this in a real environment:
- Use **AWS Fault Injection Simulator (FIS)**
- Force a system-level issue (advanced)
- Review recovery events in **EC2 > Status Checks**

---

## 🧠 Notes and Best Practices

- Auto recovery only works for system-level issues (hardware, host)
- Does *not* recover from application or software failures
- Ensure the EC2 instance is not in a **stopped** state
- Monitor with additional alarms (CPU, disk, etc.) for broader health coverage
- Use IAM roles instead of static credentials for production setups

---

## 🕒 Time Estimate

| Task                          | Time Estimate |
|-------------------------------|---------------|
| Identify EC2 & permissions    | 5 mins        |
| Create alarm                  | 5 mins        |
| Confirm + review              | 5–10 mins     |
| (Optional) simulate/test      | Varies        |
| **Total Estimated Time**      | 15–20 mins    |

---

## 📷 Screenshots to Include (Recommended)

- Screenshot of alarm listed in CloudWatch > Alarms
- EC2 > Monitoring tab with linked alarm
- EC2 > Status Checks tab (post-recovery if tested)

---

## 📤 Output Summary

| Alarm Name         | EC2-AutoRecovery-MyInstance     |
|--------------------|----------------------------------|
| Metric             | StatusCheckFailed_System         |
| Period             | 60 seconds                       |
| Evaluation Periods | 2                                |
| Threshold          | >= 1                             |
| Action             | `arn:aws:automate:<region>:ec2:recover` |

---

## 📦 Resources

- [AWS EC2 Auto Recovery Docs](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-recover.html)
- [CloudWatch CLI Reference](https://docs.aws.amazon.com/cli/latest/reference/cloudwatch/put-metric-alarm.html)
- [Status Check Metrics](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/monitoring-system-instance-status-check.html)

---

🧊 _Built with grit by FrostyWorks | GitHub: (https://github.com/frostyworks)
