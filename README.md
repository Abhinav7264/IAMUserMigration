
![PORTFOLIO PROJECTS_AWS - MODULE 2_ARCHITECTURE](https://github.com/user-attachments/assets/40c8b0f1-947c-4d75-a53d-86a96bfc4107)

# 🔐 AWS IAM User Migration & Security Hardening  


## 📋 Overview  
This project automates the migration of **100+ users** to AWS IAM using a CSV file and Bash scripting, while enforcing security best practices like **MFA** and **password policies**.  

---

## 🚀 Quick Start  

### 📂 Create a Folder (`aws-mod2`) and Download Files  
**Required Files**:  

- 📄 `IT_Team_ABC-Company.xlsx`  
  - **Purpose**: Source spreadsheet for user data.  
  - **Note**: Edit this file to match CSV requirements.  

- 🔒 `force_mfa_policy.txt`  
  - **Purpose**: IAM policy enforcing MFA for all users.  

- 📊 `users.csv`  
  - **Purpose**: Processed CSV input for the automation script.  
  - **Format**: `user,group,password`.  

- 🛠️ `aws-iam-create-user.sh`  
  - **Purpose**: Bash script to automate user creation.  
---

### Prerequisites  
- [x] **AWS CLI** configured with admin permissions.  
- [x] **CSV file** formatted as `user,group,password` (see [`users.csv`]).  
- [x] **Bash environment** (AWS CloudShell, GitBash, or Linux terminal).  
- [x] `dos2unix` tool installed.  

## 🛠️ Step-by-Step Guide  

### 1. Prepare the CSV File  
**Template**: `IT_Team_ABC-Company.xlsx`  
**Modifications**:  
- Rename columns to `user`, `group`, and `password`.  
- Remove `@abc-company.com` from email addresses.  
- Add temporary passwords (e.g., `ChangeMe123456!`).
- change the groups according to what already exist in IAM.
- Save as `users.csv` (UTF-8 encoding).  

**Example CSV Format**:  

| user                  | group         | password          |  
|-----------------------|---------------|-------------------|  
| barbara.brown         | CloudAdmin    | ChangeMe123456!   |  
| michael.scott         | NetworkAdmin  | ChangeMe123456!   |  
| brian.garcia          | LinuxAdmin    | ChangeMe123456!   |  


# AWS Cloud Shell  [ It already comes with AWS CLI installed! ]

1. Access **AWS Cloud Shell**
2. Install '**dos2unix**' 

```bash
sudo yum install dos2unix -y
```
3. Script Download

```bash
wget https://tcb-bootcamps.s3.amazonaws.com/bootcamp-aws/en/aws-iam-create-user.sh
```

4. Change script’s permissions

```bash
ls -la
chmod +x aws-iam-create-user.sh
ls -la
```

5. Upload 'users2.csv' file

```bash
# validating file content

cat users.csv
```


# Run the Script
```bash
# Make the script executable
chmod +x aws-iam-create-user.sh

# Execute with CSV input
./aws-iam-create-user.sh users.csv
````
# :white_check_mark: **Verification**

````bash
# List all users
aws iam list-users
aws iam get-group --group-name CloudAdmin
````


## Attach Policy to allow users to change their passwords:
Step 1: List All IAM Groups
First, retrieve the names of all IAM user groups in your account:

````bash
aws iam list-groups --query "Groups[*].GroupName" --output text
````
Save the output to a file (e.g., groups.txt):

````bash
aws iam list-groups --query "Groups[*].GroupName" --output text | tr '\t' '\n' > groups.txt
````
#### Step 2: Attach the Policy to All Groups
Run this script to attach the policy to every group listed in groups.txt:

````bash
POLICY_ARN="arn:aws:iam::aws:policy/IAMUserChangePassword"

while read GROUP; do
  aws iam attach-group-policy \
    --group-name "$GROUP" \
    --policy-arn "$POLICY_ARN"
  echo "Attached IAMUserChangePassword to group: $GROUP"
done < groups.txt
````


### Let’s test user access

- **IAM** | **Dashboard** | **copy URL**
- Open an 'anonymous / private' tab in another browser and access it with the URL 
try logging in with any username and change password when it prompt from [ **ChangeMe123456!** to **ChangeMe123456!*@*** ]


#### Enable MFA on your root user
- **Google Authenticator**
- Generate token and validate access using a second authentication factor
**Activate MFA** | **Virtual MFA device**
Open the authenticator app, and add an 'account' | Read QRCode | Wait for two consecutive tokens to be generated.

#### Enable MFA for All Users
Step 1: Create and Attach MFA Enforcement Policy
#### Create Custom Policy in AWS IAM:
Navigate to IAM > Policies > Create Policy
Use the JSON content from force_mfa_policy.txt
Name the policy EnforceMFAPolicy

#### Attach Policy to Groups:

````bash
# Ensure groups.txt contains target IAM groups (one per line)
# Example groups.txt:
# CloudAdmin
# DBA
# NetworkAdmin

while read -r GROUP; do
  POLICY_ARN=$(aws iam list-policies --query "Policies[?PolicyName=='EnforceMFAPolicy'].Arn" --output text)
  aws iam attach-group-policy \
    --group-name "$GROUP" \
    --policy-arn "$POLICY_ARN"
  echo "✅ Attached EnforceMFAPolicy to group: $GROUP"
done < groups.txt
````

🛡️ What This Does
🔐 Denies access to all AWS services unless MFA is active

📌 Applies to all IAM groups specified in groups.txt

🔄 Automated attachment using AWS CLI


That's all folks

