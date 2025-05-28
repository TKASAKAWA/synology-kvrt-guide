# How to Use Kaspersky Virus Removal Tool (KVRT) for Linux on Synology DSM

This guide explains how to install and configure Kaspersky Virus Removal Tool (KVRT) for Linux to run automatically on your Synology DiskStation Manager (DSM) in a Command Line Interface (CUI) environment. This allows you to schedule regular virus scans for your NAS without a graphical user interface.

## 1. Obtaining and Preparing KVRT

KVRT for Linux needs to be downloaded from the official Kaspersky website. Typically, this is done via a web browser by clicking a download button. The downloaded file will usually be named `kvrt.run`.

**Note:** Direct `wget` links might not be available on the download page. You may need to use a web browser to download the file.

### Placing the KVRT File and Granting Execute Permissions

After downloading, transfer the `kvrt.run` file to your Synology NAS and grant it execute permissions. A recommended location is `/opt/kvrt/` as it's accessible but less prone to accidental modification by ordinary users.

1.  **Log in to your NAS via SSH:**
    From your PC's terminal, use the following command:
    ```bash
    ssh your_dsm_username@your_dsm_ip_address
    ```

2.  **Elevate to root privileges:**
    All subsequent operations require root privileges.
    ```bash
    sudo -i
    ```

3.  **Create a directory for KVRT:**
    ```bash
    mkdir -p /opt/kvrt
    ```

4.  **Transfer the KVRT file to your NAS:**
    Use tools like SCP to transfer `kvrt.run` to `/opt/kvrt/` on your NAS.
    ```bash
    # Run this from your PC's terminal (you might need to exit SSH session temporarily)
    scp /path/to/local/kvrt.run your_dsm_username@your_dsm_ip_address:/opt/kvrt/
    ```
    *After transfer, log in via SSH again and elevate to root if you exited.*

5.  **Grant execute permissions:**
    ```bash
    chmod +x /opt/kvrt/kvrt.run
    ```

6.  **Create a data directory for KVRT:**
    Create a directory under `/var/log/` where KVRT will store its logs, reports, and quarantined files.
    ```bash
    mkdir -p /var/log/kvrt_data
    chmod 700 /var/log/kvrt_data # Set appropriate permissions
    ```

## 2. KVRT Command-Line Execution Options

These commands are intended to be used within the DSM Task Scheduler, executed as the `root` user.

### Important Considerations:

* **Temporary Directory Cleanup:** Both commands include a line to delete the temporary KVRT directory (`/opt/kvrt/kvrt_temp`) before execution. This prevents the "Target directory already exists" error.
* **Log Output:** Scan logs (summary) will be redirected to `/var/log/kvrt.log`. Detailed reports (encrypted) will be outputted to `/var/log/kvrt_data/Reports/`.
* **Scan Scope:** To scan "all directories," `-custom /` is used, which scans the entire system including DSM's `/volume1` etc.
* **Root Execution:** The provided commands assume direct execution as the `root` user (e.g., after `sudo -i` or within the DSM Task Scheduler configured for the `root` user). `sudo` is not included in the command lines themselves.

### A. Detect Only (No Deletion/Quarantine): Recommended for Safety

This command scans the entire system. If threats are detected, **no automatic disinfection or quarantine actions will be performed.** Scan results will only be logged and reported. This is the safest method to prevent accidental data loss due to false positives.

```bash
# Clean up KVRT's temporary directory
rm -rf /opt/kvrt/kvrt_temp

# Execute KVRT to scan the entire system (detect only, no action)
# Console output is redirected to /var/log/kvrt.log
/opt/kvrt/kvrt.run --target /opt/kvrt/kvrt_temp -- -accepteula -silent -custom / -d /var/log/kvrt_data > /var/log/kvrt.log 2>&1
