# File Integrity Monitoring (FIM) & Who-Data Setup in Wazuh  
This project documents how to configure **FIM** and enable **Who-Data auditing** on Wazuh agents or the manager itself, enabling visibility into **who made changes to monitored files**, from what process, and when.

---

## 🎯 Objective  
To configure Wazuh's File Integrity Monitoring (FIM) module with `whodata` enabled on a Linux-based system. This allows security teams to detect file modifications and attribute them to specific users and processes for better auditability.

---

## 🔍 What is Who-Data in Wazuh?  
**Who-data** is an advanced feature of the FIM module that uses Linux auditd to record:
- The user who made the file change  
- The process or command that performed the action  
- The timestamp and action type (e.g., creation, deletion)  

This adds attribution to raw file integrity monitoring, transforming it from passive monitoring into an active forensic audit tool.

---

## 📚 Skills Learned  
- Installing and configuring `auditd` and `audispd-plugins`  
- Enabling FIM with real-time and whodata auditing  
- Troubleshooting directory permissions and symlinks  
- Testing FIM and validating audit rules  
- Editing and verifying Wazuh configuration files

---

## 🛠️ Tools Used  
<div>
  <img src="https://img.shields.io/badge/-Wazuh-0078D4?style=for-the-badge&logo=Wazuh&logoColor=white" />
  <img src="https://img.shields.io/badge/-auditd-000000?style=for-the-badge" />
  <img src="https://img.shields.io/badge/-Linux-333333?style=for-the-badge&logo=Linux&logoColor=white" />
</div>

---

## 📝 Deployment Steps

## Who-Data Install

### 1. Install Who-Data Dependencies
SSH into your Linux server (Wazuh manager or agent), then install `auditd` and `audispd-plugins`:
```bash
sudo yum install audit
sudo yum install audispd-plugins
```
### 2. Remove AuditD Rules That Break Who-Data
Check for this disabled task rule:
```bash
sudo auditctl -l | grep task
```
If output contains `-a never,task`, remove it from:
```bash
sudo nano /etc/audit/rules.d/audit.rules
```
Then:
```bash
sudo systemctl restart auditd
```
```bash
sudo systemctl restart wazuh-agent/manager
```

## FIM Configuration

### 1. Configure Wazuh FIM for Who-Data
Open the Wazuh config:
```bash
sudo nano /var/ossec/etc/ossec.conf
```
Under `<syscheck>` (File Integrity Monitoring), add or update the monitored directories:
```bash
<directories check_all="yes" report_changes="yes" realtime="yes" whodata="yes">/your/directory</directories>
```
Example:
```bash
<directories check_all="yes" report_changes="yes" realtime="yes" whodata="yes">/opt</directories>
<directories check_all="yes" report_changes="yes" realtime="yes" whodata="yes">/lib</directories>
<directories check_all="yes" report_changes="yes" realtime="yes" whodata="yes">/lib64</directories>
```
Restart agent:
```bash
sudo systemctl restart wazuh-agent
```
### 2. Test and Validate
Touch/Delete a File:
```bash
cd /your/directory
sudo touch test.txt
sudo rm test.txt
```
Check Wazuh WebUI → FIM dashboard for log entries.
Verify Audit Rules:
```bash
sudo auditctl -l | grep wazuh_fim
```
You should see entries like:
```bash
-w /opt -p wa -k wazuh_fim
-w /lib -p wa -k wazuh_fim
-w /lib64 -p wa -k wazuh_fim
```

### Troubleshooting
- If you're not receiving logs:
- Confirm the directory isn't a **symlink** using:
```bash
ls -ld /your/directory
```
- Add the **target path** of the symlink in `ossec.conf` in the same directories to check section, just like you did the rest.
- Ensure correct permissions exist on the directory for auditd to access logs.

---

## 👨‍💻 Author  
Mario Tagaras | Florida State University Alum  
