# 🔐 AWS IAM User Migration & Security Hardening

## 📋 Overview
This project automates the migration of **100+ users** to AWS IAM using a **CSV file** and **Bash scripting** while enforcing security best practices like **MFA** and **password policies**.

---

## 🚀 Quick Start

### 📂 Folder Structure (`aws-mod2`)
| File                  | Purpose                         |
|-----------------------|---------------------------------|
| `IT_Team_ABC-Company.xlsx` | Source spreadsheet for user data 📄 |
| `force_mfa_policy.json` | IAM policy to enforce MFA 🔒 |
| `users.csv`           | Processed CSV input 📊         |
| `aws-iam-create-user.sh` | Bash script to automate user creation 🛠️ |

---

## ✅ Prerequisites
- AWS CLI configured with **admin permissions** 🔑
- CSV file formatted as `user,group,password`
- Bash environment (**AWS CloudShell**, **GitBash**, or **Linux terminal**)
- `dos2unix` installed (for Windows compatibility)

---

## 🔑 CSV File Preparation

### Template: `IT_Team_ABC-Company.xlsx`
| Column    | Example           |
|-----------|------------------|
| user      | michael.scott    |
| group     | NetworkAdmin     |
| password  | ChangeMe123456!  |

- Remove **@abc-company.com** from emails.
- Add temporary passwords like **ChangeMe123456!**.
- Save as **`users.csv`** (UTF-8 encoding).

---

## ⚙️ AWS CloudShell Setup

1. Access **AWS CloudShell**.
2. Install `dos2unix`:

```bash
sudo yum install dos2unix -y
```

3. Download the Bash Script:

```bash
wget https://tcb-bootcamps.s3.amazonaws.com/bootcamp-aws/en/aws-iam-create-user.sh
```

4. Grant Execution Permissions:

```bash
chmod +x aws-iam-create-user.sh
```

5. Upload your `users.csv` file and validate:

```bash
cat users.csv
```

---

## ▶️ Run the Script

```bash
./aws-iam-create-user.sh users.csv
```

### ✅ Verification

```bash
aws iam list-users
aws iam get-group --group-name CloudAdmin
```

---

## 🔑 Allow Users to Change Passwords

### Attach `IAMUserChangePassword` Policy to All Groups

1. List all groups:

```bash
aws iam list-groups --query "Groups[*].GroupName" --output text | tr '\t' '\n' > groups.txt
```

2. Attach Policy:

```bash
POLICY_ARN="arn:aws:iam::aws:policy/IAMUserChangePassword"

while read GROUP; do
  aws iam attach-group-policy \
    --group-name "$GROUP" \
    --policy-arn "$POLICY_ARN"
  echo "🔑 Attached IAMUserChangePassword to group: $GROUP"
done < groups.txt
```

---

## 🔐 Enforce MFA
### Enable MFA for Root User
1.	Navigate to AWS IAM Console → Dashboard.
2.	Click on Activate MFA on your root account.
3.	Select Virtual MFA device and click Continue.
4.	Open Google Authenticator (or similar app) and scan the QR code.
5.	Enter the two consecutive MFA codes displayed on your device.
6.	Click Assign MFA to complete the process.

### Create Custom MFA Enforcement Policy

1. Navigate to **IAM → Policies → Create Policy**.
2. Use the content from **`force_mfa_policy.json`**.
3. Name the policy **EnforceMFAPolicy**.

### Attach MFA Policy to All Groups

```bash
POLICY_ARN=$(aws iam list-policies --query "Policies[?PolicyName=='EnforceMFAPolicy'].Arn" --output text)

while read -r GROUP; do
  aws iam attach-group-policy \
    --group-name "$GROUP" \
    --policy-arn "$POLICY_ARN"
  echo "✅ Attached EnforceMFAPolicy to group: $GROUP"
done < groups.txt
```

### 🔄 What This Does
| Feature        | Description                |
|---------------|---------------------------|
| 🔐 MFA        | Denies access unless MFA is active |
| 📋 Group-Based | Applies to all users in the specified groups |
| 🚀 Automated   | Uses AWS CLI to attach the policy |

---

### 🎯 Testing User Access
1. Login via IAM **Dashboard URL**.
2. Use temporary password.
3. Update password at first login.
4. Enable **Virtual MFA** using Google Authenticator.

---

## 🛡️ Security Recap
| Feature    | Status |
|------------|-------|
| User Migration | ✅ Completed |
| Password Policy | ✅ Enforced |
| MFA        | ✅ Mandatory |
| IAM User Permissions | 🔐 Restricted |

---

### 🎉 That's All, Folks! 🦸‍♀️🦸‍♂️
Automated. Secure. Scalable.


