# -Two-Objectives-Using-Bash-or-Python
System Health Monitoring Script (Bash): This script checks system health, including CPU usage, memory, and disk space, and sends alerts if thresholds are exceeded.
 Bash Script:
 #!/bin/bash

# Define thresholds
CPU_THRESHOLD=80
MEM_THRESHOLD=80
DISK_THRESHOLD=90

# Check CPU usage
CPU_USAGE=$(top -bn1 | grep "Cpu(s)" | sed "s/.*, *\([0-9.]*\)%* id.*/\1/" | awk '{print 100 - $1}')
if (( $(echo "$CPU_USAGE > $CPU_THRESHOLD" | bc -l) )); then
  echo "CPU usage is above $CPU_THRESHOLD%. Current: $CPU_USAGE%" >> /var/log/system_health.log
fi

# Check Memory usage
MEM_USAGE=$(free | grep Mem | awk '{print $3/$2 * 100.0}')
if (( $(echo "$MEM_USAGE > $MEM_THRESHOLD" | bc -l) )); then
  echo "Memory usage is above $MEM_THRESHOLD%. Current: $MEM_USAGE%" >> /var/log/system_health.log
fi

# Check Disk usage
DISK_USAGE=$(df / | grep / | awk '{ print $5 }' | sed 's/%//g')
if [ $DISK_USAGE -gt $DISK_THRESHOLD ]; then
  echo "Disk usage is above $DISK_THRESHOLD%. Current: $DISK_USAGE%" >> /var/log/system_health.log
fi
Application Health Checker (Python): This script checks if a web application is up and responds correctly by verifying HTTP status codes.
 Python Script:
 import requests

# Define the application URL
url = "http://your-app-url.com"

try:
    response = requests.get(url)
    if response.status_code == 200:
        print(f"Application is UP (Status Code: {response.status_code})")
    else:
        print(f"Application is DOWN (Status Code: {response.status_code})")
except requests.exceptions.RequestException as e:
    print(f"Error: Unable to reach the application. Exception: {e}")
import requests

# Define the application URL
url = "http://your-app-url.com"

try:
    response = requests.get(url)
    if response.status_code == 200:
        print(f"Application is UP (Status Code: {response.status_code})")
    else:
        print(f"Application is DOWN (Status Code: {response.status_code})")
except requests.exceptions.RequestException as e:
    print(f"Error: Unable to reach the application. Exception: {e}")
