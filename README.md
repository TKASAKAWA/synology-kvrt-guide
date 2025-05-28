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
1.1 Log in to your NAS via SSH
From your PC's terminal, use the following command:

text

コピー
ssh your_dsm_username@your_dsm_ip_address
1.2 Elevate to root privileges
All subsequent operations require root privileges.

text

コピー
sudo -i
1.3 Create a directory for KVRT
