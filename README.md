AWS IAM User Migration & Security Hardening
📋 Overview
This project automates the migration of 100+ users to AWS IAM using a CSV file and Bash scripting, while enforcing security best practices like MFA and password policies.

🛠️ Prerequisites
AWS CLI configured with admin permissions.

CSV file formatted as user,group,password (see users.csv).

Bash environment (AWS CloudShell, GitBash, or Linux terminal).

dos2unix tool installed.

🚀 Migration Steps
1. Prepare the CSV File
Template: IT_Team_ABC-Company.xlsx (provided).

Modifications:

Rename columns to user, group, and password.

Remove @abc-company.com from email addresses.

Add temporary passwords (e.g., ChangeMe123456!).

Save as users.csv (UTF-8 encoding).
