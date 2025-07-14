# System_Audit_Repo
Bash Scripting Project
cat > system_audit_repo/system_audit.sh <<- "EOF"
#!/bin/bash

# Define log file
AUDIT_LOG="system_audit.log"

# Create a separator with date
echo "===== System Audit Report - $(date) =====" | tee -a $AUDIT_LOG

# Check failed services
echo -e "\n[+] Checking Failed Services:" | tee -a $AUDIT_LOG
systemctl --failed | tee -a $AUDIT_LOG

# Check critical system logs for errors
echo -e "\n[+] Checking Critical System Logs (Priority 3+):" | tee -a $AUDIT_LOG
journalctl -b -p 3 --no-pager | tail -20 | tee -a $AUDIT_LOG

# Check system uptime
echo -e "\n[+] System Uptime:" | tee -a $AUDIT_LOG
uptime -p | tee -a $AUDIT_LOG

# Check disk space
echo -e "\n[+] Disk Usage:" | tee -a $AUDIT_LOG
df -h | tee -a $AUDIT_LOG

# Check memory usage
echo -e "\n[+] Memory Usage:" | tee -a $AUDIT_LOG
free -h | tee -a $AUDIT_LOG

# Check CPU load and top processes
echo -e "\n[+] CPU Load and Top Processes:" | tee -a $AUDIT_LOG
top -b -n1 | head -15 | tee -a $AUDIT_LOG

# Check active system services
echo -e "\n[+] Active System Services:" | tee -a $AUDIT_LOG
systemctl list-units --type=service --state=running | tee -a $AUDIT_LOG

# Check network status and open ports
echo -e "\n[+] Network Status and Open Ports:" | tee -a $AUDIT_LOG
ip a | tee -a $AUDIT_LOG
ss -tuln | tee -a $AUDIT_LOG # Shows listening TCP and UDP sockets

# Check for security updates (Debian/Ubuntu)
if [ -x "$(command -v apt)" ]; then
    echo -e "\n[+] Checking Available Security Updates (Debian/Ubuntu):" | tee -a $AUDIT_LOG
    apt list --upgradable 2>/dev/null | tee -a $AUDIT_LOG
fi

# Check for security updates (RHEL/CentOS)
if [ -x "$(command -v yum)" ]; then
    echo -e "\n[+] Checking Available Security Updates (RHEL/CentOS):" | tee -a $AUDIT_LOG
    yum check-update --security | tee -a $AUDIT_LOG
fi

echo "Audit completed. Results saved to $AUDIT_LOG" | tee -a $AUDIT_LOG

EOF
