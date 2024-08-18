# Nginx Debugging Project

This project contains a Bash script to fix an Nginx configuration issue and a postmortem report detailing a related outage incident.

## Table of Contents
1. [Nginx Fix Script](#nginx-fix-script)
2. [Postmortem: The Great Nginx Port 80 Fiasco](#postmortem-the-great-nginx-port-80-fiasco)
   - [Issue Summary](#issue-summary)
   - [Timeline](#timeline)
   - [Root Cause and Resolution](#root-cause-and-resolution)
   - [Corrective and Preventative Measures](#corrective-and-preventative-measures)

## Nginx Fix Script

This Bash script automates the process of ensuring Nginx is installed, configured to listen on port 80, and running correctly.

```bash
#!/bin/bash

# Check if Nginx is installed
if ! command -v nginx &> /dev/null; then
    echo "Nginx is not installed. Installing..."
    apt-get update
    apt-get install -y nginx
fi

# Ensure Nginx configuration listens on port 80
sed -i 's/listen 80 default_server;/listen 80;/g' /etc/nginx/sites-available/default

# Restart Nginx service
systemctl restart nginx

# Verify Nginx is listening on port 80
if netstat -tuln | grep :80 > /dev/null; then
    echo "Nginx is now listening on port 80"
else
    echo "Failed to configure Nginx to listen on port 80"
    exit 1
fi
```

### Usage

1. Save the script to a file (e.g., `fix_nginx.sh`)
2. Make it executable: `chmod +x fix_nginx.sh`
3. Run the script with root privileges: `sudo ./fix_nginx.sh`

## Postmortem: The Great Nginx Port 80 Fiasco

### Issue Summary
- **Duration**: 2 hours and 37 minutes, from 14:23 to 17:00 UTC on August 18, 2024
- **Impact**: Our main website was inaccessible to 100% of users, resulting in a complete service outage
- **Root Cause**: Misconfiguration in Nginx, preventing it from listening on port 80

### Timeline
- 14:23 UTC - Issue detected when an engineer noticed the website was down during a routine check
- 14:25 UTC - Initial investigation began, focusing on potential DDoS attacks or server hardware issues
- 14:45 UTC - Server logs showed no signs of attacks or hardware problems, investigation shifted to web server configuration
- 15:10 UTC - Discovered Nginx was running but not listening on port 80
- 15:30 UTC - Attempted to modify Nginx configuration manually, but changes didn't persist after service restart
- 15:45 UTC - Incident escalated to the senior DevOps team
- 16:15 UTC - Root cause identified: a recent automated update had modified the Nginx configuration
- 16:45 UTC - Fix implemented and tested on a staging server
- 17:00 UTC - Fix deployed to production, service restored

### Root Cause and Resolution
The root cause was traced to an automated update script that had inadvertently changed the Nginx configuration, replacing `listen 80;` with `listen 80 default_server;` in the default site configuration. This subtle change prevented Nginx from binding to all IPv4 addresses on port 80.

The issue was resolved by creating a Bash script to revert this change and ensure Nginx listens on port 80 for all IPv4 addresses. The script also added checks to verify Nginx installation and proper configuration.

### Corrective and Preventative Measures
To prevent similar issues in the future, we will implement the following measures:

1. Improve our automated update scripts to preserve critical configurations
2. Implement more comprehensive monitoring for web server status, including port listening checks
3. Create and maintain a runbook for quick diagnosis and resolution of common web server issues
4. Conduct regular audits of server configurations to catch discrepancies early

#### TODO List
1. [ ] Review and update all automated server management scripts
2. [ ] Set up automated alerts for Nginx configuration changes
3. [ ] Implement port listening checks in our monitoring system
4. [ ] Create a detailed runbook for web server troubleshooting
5. [ ] Schedule monthly configuration audits
6. [ ] Conduct a team post-mortem to share learnings and improve response procedures

### The Lighter Side: A Port-80 Pun-demonium
While our website was down, our error pages were up... and they were *error*-sistible! We're considering adding a "404: Port Not Found" joke to lighten the mood during future outages. Remember, in the world of web servers, it's important to keep your ports open and your sense of humor even more open!

![Nginx Configuration Humor](https://api.placeholder.com/300x200?text=Nginx+Configuration+Humor)

In conclusion, while this outage was a serious issue, it has provided valuable insights into our systems and processes. By implementing the corrective measures outlined above, we aim to significantly reduce the likelihood of similar incidents in the future. Let's raise a toast to resilient systems and may our ports always be open (except for the ones that shouldn't be, of course)!
