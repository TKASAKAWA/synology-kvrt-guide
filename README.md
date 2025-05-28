# How to Use Kaspersky Virus Removal Tool (KVRT) for Linux on Synology DSM

This guide explains how to install and configure Kaspersky Virus Removal Tool (KVRT) for Linux to run automatically on your Synology DiskStation Manager (DSM) in a Command Line Interface (CUI) environment. This allows you to schedule regular virus scans for your NAS without a graphical user interface.

## 1. Obtaining and Preparing KVRT

KVRT for Linux needs to be downloaded from the official Kaspersky website. Typically, this is done via a web browser by clicking a download button. The downloaded file will usually be named `kvrt.run`.

**Note:** Direct `wget` links might not be available on the download page. You may need to use a web browser to download the file.

Download Kaspersky Virus Removal Tool application
@@@@Linux version is hidden below.@@@@
https://www.kaspersky.com/downloads/free-virus-removal-tool
Kaspersky Virus Removal Tool
for Linux®
Virus removal helps clean your Linux PC of malware if it has been infected.

Steps to Prepare KVRT

1.1 Log in to your NAS via SSH:
From your PC's terminal, use the following command:
'''
ssh your_dsm_username@your_dsm_ip_address
'''

1.2 Elevate to root privileges:
All subsequent operations require root privileges.
'''
sudo -i
'''

1.3 Create a directory for KVRT:
'''
mkdir -p /opt/kvrt
'''

1.4 Transfer the KVRT file to your NAS:
Use SCP to transfer kvrt.run to /opt/kvrt/ on your NAS. Run this from your PC's terminal:
'''
scp /path/to/local/kvrt.run your_dsm_username@your_dsm_ip_address:/opt/kvrt/
'''
After the transfer, log in via SSH again and elevate to root if you exited.

1.5 Grant execute permissions:
'''
chmod +x /opt/kvrt/kvrt.run
'''

1.6 Create a data directory for KVRT:
Create a directory under /var/log/ where KVRT will store its logs, reports, and quarantined files.
'''
mkdir -p /var/log/kvrt_data
chmod 700 /var/log/kvrt_data
'''

KVRT Command-Line Execution Options
These commands are intended to be used within the DSM Task Scheduler, executed as the root user.

Important Considerations:

Temporary Directory Cleanup: The commands below include a line to delete the temporary KVRT directory (/opt/kvrt/kvrt_temp) before execution. This prevents the "Target directory already exists" error.
Log Output: Scan logs (summary) will be redirected to /var/log/kvrt.log. Detailed reports (encrypted) will be outputted to /var/log/kvrt_data/Reports/.
Scan Scope: To scan "all directories," -custom / is used, which scans the entire system including DSM's /volume1, etc.
Root Execution: The provided commands assume direct execution as the root user (e.g., after sudo -i or within the DSM Task Scheduler configured for the root user). sudo is not included in the command lines themselves.
2.1 Detect Only (No Deletion/Quarantine): Recommended for Safety

This command scans the entire system. If threats are detected, no automatic disinfection or quarantine actions will be performed. Scan results will only be logged and reported. This is the safest method to prevent accidental data loss due to false positives.

'''
rm -rf /opt/kvrt/kvrt_temp
/opt/kvrt/kvrt.run --target /opt/kvrt/kvrt_temp -- -accepteula -silent -custom / -d /var/log/kvrt_data > /var/log/kvrt.log 2>&1
'''

2.2 Detect and Quarantine (Move to Quarantine): Use with Caution

This command scans the entire system. If threats are detected, they will be automatically moved to quarantine. Quarantined files become inoperable and are moved to /var/log/kvrt_data/Quarantine/ in an encrypted form. Deletion is not performed directly, but be aware that quarantining critical system files due to false positives can affect DSM's functionality. Use this option with extreme caution.

'''
rm -rf /opt/kvrt/kvrt_temp
/opt/kvrt/kvrt.run --target /opt/kvrt/kvrt_temp -- -accepteula -silent -processlevel 3 -custom / -d /var/log/kvrt_data > /var/log/kvrt.log 2>&1
'''

Note on -processlevel 3: The KVRT documentation states "Neutralization involves applying actions in the following order: Disinfection. If the object cannot be disinfected, the application attempts to restore the object from backup. If the object cannot be restored, the application deletes the object." While Kaspersky products typically prioritize quarantine, this description indicates a potential for ultimate deletion. If you absolutely want to avoid deletion, stick to option 2.1 (Detect Only) and manually handle threats after reviewing the report.

Important: -custom / scans the entire system from the root directory. If you only want to scan shared folders, specify the volume path, e.g., -custom /volume1 (or -custom /volume1 -custom /volume2 for multiple volumes).

Configuring DSM Task Scheduler
Set up the automated scan task from your Synology DSM's web interface.

Log in to DSM with an administrator account and go to Control Panel > Task Scheduler.
Click Create > Scheduled Task > User-defined script.

"General" Tab:
Task name: e.g., Kaspersky_Daily_Scan
User: Select root from the dropdown menu.
Event: Select "Boot-up" (if you want it to run on every boot) or "Schedule" (for regular daily/weekly runs).
Status: Ensure "Enabled" is checked.

"Schedule" Tab:
Date: Select "Daily" and set the desired execution time (e.g., 3:00 AM, during low NAS usage hours).

"Task Settings" Tab:
User-defined script: Paste the entire chosen command (from 2.1 or 2.2) from "2. KVRT Command-Line Execution Options" here.
Send run details by email: (Optional) Check this box and configure email notifications if you want to receive the content of /var/log/kvrt.log via email upon task completion.

Click "OK" to save the task. If a warning appears, click "Yes" to proceed with root execution.

Verification and Log Checking
After creating the task, it is highly recommended to perform a manual test run. Go back to Task Scheduler, select your newly created task, and click the "Run" button.

Execution Log Check: After execution, connect to your NAS via SSH and check the /var/log/kvrt.log file for the summary scan results.
'''
cat /var/log/kvrt.log
'''
This log will show basic information like scan start/finish, processed files, detected threats, and errors. A "Detected: 0" indicates no threats were found.

Detailed Report Check: Encrypted detailed reports are saved in the /var/log/kvrt_data/Reports/ directory. These files cannot be read directly in CUI. To view their contents, you must launch the KVRT GUI using SSH X-forwarding from your PC and load the report.
'''
ssh -X your_dsm_username@your_dsm_ip_address
sudo /opt/kvrt/kvrt.run
'''

Operational Notes
System Load: The CPU usage will spike during the scan. Schedule the task during off-peak hours to minimize impact on NAS performance.
Definition File Updates: KVRT will attempt to automatically download and update its definition files upon launch if an internet connection is available. Ensure your DSM has internet access.
Disk Space: Ensure you have sufficient disk space for temporary files, reports, and quarantined items.

Open-source project for installing Synology DSM on self-built NAS:
https://github.com/RROrg/rr
