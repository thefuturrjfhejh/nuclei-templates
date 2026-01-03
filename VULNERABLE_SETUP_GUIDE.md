# Vulnerable Environment Setup Guide for CVE-2018-8581

This guide explains how to set up a valid (non-simulated) Microsoft Exchange Server environment to verify the Nuclei template for CVE-2018-8581. This is required because the maintainers have rejected simulated environments (like Docker containers that just mock the response).

## 1. Requirements

*   **Virtual Machine Software:** VMware Workstation (Player or Pro), VirtualBox, or Hyper-V.
*   **Operating System ISO:** Windows Server 2012 R2 or Windows Server 2016 (Evaluation versions are free from Microsoft).
*   **Exchange Server ISO:** Exchange Server 2016 Cumulative Update 3 (CU3) or earlier.
    *   *Note:* Newer versions may have patched this specific vulnerability. Exchange 2016 CU3 is known to be vulnerable.

## 2. Installation Steps

### Step A: Install Windows Server
1.  Create a new VM and install Windows Server 2016.
2.  Assign a static IP address to the VM.
3.  Rename the computer to something simple (e.g., `EXCHANGE01`).

### Step B: Install Active Directory Domain Services (AD DS)
Exchange Server requires a domain controller.
1.  Open **Server Manager**.
2.  Click **Add roles and features**.
3.  Select **Active Directory Domain Services** and install.
4.  After installation, click the flag icon in Server Manager and select **Promote this server to a domain controller**.
5.  Select **Add a new forest** and enter a root domain name (e.g., `vuln.local`).
6.  Complete the wizard and reboot.

### Step C: Install Prerequisites for Exchange 2016
Open PowerShell as Administrator and run:
```powershell
Install-WindowsFeature AS-HTTP-Activation, Desktop-Experience, NET-Framework-45-Features, RPC-over-HTTP-proxy, RSAT-Clustering, RSAT-Clustering-CmdInterface, RSAT-Clustering-Mgmt, RSAT-Clustering-PowerShell, Web-Mgmt-Console, WAS-Process-Model, Web-Asp-Net45, Web-Basic-Auth, Web-Client-Auth, Web-Digest-Auth, Web-Dir-Browsing, Web-Dyn-Compression, Web-Http-Errors, Web-Http-Logging, Web-Http-Redirect, Web-Http-Tracing, Web-ISAPI-Ext, Web-ISAPI-Filter, Web-Lgcy-Mgmt-Console, Web-Metabase, Web-Mgmt-Service, Web-Net-Ext45, Web-Request-Monitor, Web-Server, Web-Stat-Compression, Web-Static-Content, Web-Windows-Auth, Web-WMI, Windows-Identity-Foundation, RSAT-ADDS
```
(You may need to install .NET Framework 4.8 and Unified Communications Managed API 4.0 Runtime manually if the ISO check asks for them).

### Step D: Install Exchange Server 2016
1.  Mount the Exchange Server ISO.
2.  Run `Setup.exe`.
3.  Select **Mailbox role**.
4.  Proceed through the installation. This can take 30-60 minutes.

### Step E: Create a Low-Privilege User
1.  Open **Exchange Admin Center** (https://localhost/ecp) or **Active Directory Users and Computers**.
2.  Create a new user (e.g., `user1` with password `Password123!`).
3.  Ensure this user has a mailbox.

## 3. Verifying with Nuclei

### Option 1: Copy the Template (Easiest)
You do not need to clone the whole repository if you don't want to.
1.  On your laptop (attacker machine), create a file named `cve-2018-8581.yaml`.
2.  Paste the content of the template created in this pull request into that file.
3.  Run the command:
    ```bash
    nuclei -t cve-2018-8581.yaml -u https://<IP_OF_EXCHANGE_VM>/ews/exchange.asmx -u user1:Password123!
    ```

### Option 2: Clone the Repo
1.  Clone your fork: `git clone https://github.com/<YOUR_USERNAME>/nuclei-templates.git`
2.  Checkout this branch.
3.  Run:
    ```bash
    nuclei -t http/cves/2018/CVE-2018-8581.yaml -u https://<IP_OF_EXCHANGE_VM>/ews/exchange.asmx -u user1:Password123!
    ```

## 4. Expected Result
If the server is vulnerable:
1.  Nuclei will send a SOAP request subscribing to push notifications.
2.  The Exchange Server will send a notification to the Interactsh server (OOB interaction).
3.  Nuclei will report `[CVE-2018-8581]` as a finding.
