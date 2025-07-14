#!/bin/bash
# Define the report file name with a timestamp
TIMESTAMP=$(date +"%Y-%m-%d_%H-%M-%S")
REPORT_FILE="$HOME/Desktop/system_audit_report_$TIMESTAMP.txt"
USAGE=$()
# Start of the report
echo "=========================================" > "$REPORT_FILE"
echo "  System and Security Audit" >> "$REPORT_FILE"
echo "=========================================" >> "$REPORT_FILE"
echo "" >> "$REPORT_FILE"

echo "Report generated on: $(date)" >> "$REPORT_FILE"
echo "-----------------------------------------" >> "$REPORT_FILE"
echo "" >> "$REPORT_FILE"

echo "1. System Information" >> "$REPORT_FILE"
echo "---------------------" >> "$REPORT_FILE"
echo "Hostname: $(hostname)" >> "$REPORT_FILE"
echo "IP Address: $(hostname -I | awk '{print $1}')" >> "$REPORT_FILE" # Get the first IP address
echo "Uptime: $(uptime)" >> "$REPORT_FILE"
echo "Kernel Version: $(uname -r)" >> "$REPORT_FILE"
echo "" >> "$REPORT_FILE"
# Define the disk usage threshold (80%))
DISK_USAGE_THRESHOLD=80

echo "2. Disk Usage" >> "$REPORT_FILE"
echo "---------------" >> "$REPORT_FILE"

# Get disk usage 
df -h | grep -v "snap" | grep -v "tmpfs" | grep -v "udev" | grep -v "devtmpfs" | tail -n +2 | while read -r LINE; do
    USAGE=$(echo "$LINE" | awk '{print $5}' | sed 's/%//g')
    FILESYSTEM=$(echo "$LINE" | awk '{print $1}')
    MOUNTPOINT=$(echo "$LINE" | awk '{print $6}')

    echo "Filesystem: $FILESYSTEM, Mountpoint: $MOUNTPOINT, Usage: $USAGE%" >> "$REPORT_FILE"
    if ((USAGE >= DISK_USAGE_THRESHOLD )); then
        echo "  *** WARNING: Disk usage on $MOUNTPOINT exceeds ${DISK_USAGE_THRESHOLD}%! ***" >> "$REPORT_FILE"
    fi 
    done
echo "" >> "$REPORT_FILE"
echo "3. Logged-in Users and Potential Insecure Accounts" >> "$REPORT_FILE"
echo "----------------------------------------------------" >> "$REPORT_FILE"

echo "Currently logged-in users:" >> "$REPORT_FILE"
who >> "$REPORT_FILE"
echo "" >> "$REPORT_FILE"

echo "Checking for user accounts with empty passwords:" >> "$REPORT_FILE"
# It's important to use tools like `passwd -S` or analyze /etc/shadow securely
# Direct parsing of /etc/passwd or /etc/shadow for this purpose requires root privileges
# and could be prone to errors or security issues if not done carefully.
# For a basic audit, we'll indicate if it's possible to check.
if [ -f "/etc/shadow" ] && [ -r "/etc/shadow" ]; then
    EMPTY_PASSWORD_USERS=$(sudo awk -F: '($2 == "") {print $1}' /etc/shadow 2>/dev/null)
    if [ -n "$EMPTY_PASSWORD_USERS" ]; then
        echo "  WARNING: The following user(s) have empty passwords (security risk):" >> "$REPORT_FILE"
        echo "$EMPTY_PASSWORD_USERS" >> "$REPORT_FILE"
    else
        echo "  No user accounts with empty passwords found." >> "$REPORT_FILE"
    fi
else
    echo "  Cannot check for empty passwords (requires root privileges or /etc/shadow is not readable)." >> "$REPORT_FILE"
fi
echo "" >> "$REPORT_FILE"
echo "4. Top Memory-Consuming Processes (Top 5)" >> "$REPORT_FILE"
echo "--------------------------------------------" >> "$REPORT_FILE"
ps aux --sort=-%mem | head -n 5 >> "$REPORT_FILE" # Displaying header and top 5 processes
echo "" >> "$REPORT_FILE"
echo "5. Essential Service Status" >> "$REPORT_FILE"
echo "-----------------------------" >> "$REPORT_FILE"
SERVICES=("systemd" "auditd" "cron" "ufw")
for SERVICE in "${SERVICES[@]}"; do
    if systemctl -q is-active "$SERVICE"; then
        echo "$SERVICE: Running" >> "$REPORT_FILE"
    else
        echo "$SERVICE: Not Running" >> "$REPORT_FILE"
    fi
done
echo "" >> "$REPORT_FILE"
echo "6. Failed Login Attempts (Last 24 Hours)" >> "$REPORT_FILE"
echo "------------------------------------------" >> "$REPORT_FILE"

# /var/log/auth.log 
if [ -f "/var/log/auth.log" ] && [ -r "/var/log/auth.log" ]; then
    FAILED_ATTEMPTS=$(grep "Failed password" /var/log/auth.log | tail -n 10) # Check last 10 failed attempts
    if [ -n "$FAILED_ATTEMPTS" ]; then
        echo "  The following failed login attempts were detected:" >> "$REPORT_FILE"
        echo "$FAILED_ATTEMPTS" >> "$REPORT_FILE"
    else
        echo "  No failed login attempts found in recent logs." >> "$REPORT_FILE"
    fi
else
    echo "  Authentication log file (/var/log/auth.log) not found." >> "$REPORT_FILE"
fi
echo "" >> "$REPORT_FILE"
echo  "Done"
