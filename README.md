# Automated-Threat-Detection-with-Wazuh-Shuffle-and-TheHive

# SOC Home Project: Automated Threat Detection and Response with Wazuh, Shuffle, and TheHive

## Project Overview
This project showcases an automated threat detection and response system using Wazuh, Shuffle, and TheHive. 

### Project Flow:
1. **Data Collection**: Wazuh agents on Windows collect telemetry.
2. **Rule Matching**: Wazuh Manager matches data against rules.
3. **Enrichment**: Adds threat intelligence data.
4. **Alert Creation**: Alerts generated in Shuffle and sent to TheHive.
5. **SOC Analyst Review**: Incidents are managed in TheHive.

## Installation

### VMware Setup:
1. Download [VMware Workstation](https://www.vmware.com).
2. Install and activate.

### Windows 10 VM Setup:
1. Download the Windows ISO from [Microsoft](https://www.microsoft.com).
2. Install Windows 10 on a new VM.

### Sysmon Installation:
1. Download Sysmon from [Sysinternals](https://docs.microsoft.com/sysinternals).
2. Install using:

    ```bash
    .\Sysmon64.exe -i sysmonconfig.xml
    ```

### Wazuh Setup on DigitalOcean:
1. Create a new Ubuntu droplet.
2. Update and install Wazuh:

    ```bash
    sudo apt update && sudo apt upgrade
    curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh && sudo bash ./wazuh-install.sh -a
    ```

### TheHive Installation:
1. Create a new droplet and install dependencies:

    - Java:
        ```bash
        sudo apt install java-common java-11-amazon-corretto-jdk
        ```
    - Cassandra:
        ```bash
        sudo apt install cassandra
        ```
    - Elasticsearch:
        ```bash
        sudo apt install elasticsearch
        ```
    - TheHive:
        ```bash
        sudo apt-get install -y thehive
        ```

2. Configure services and access TheHive at `http://<your_TheHive_IP_Address>:9000`.

# Wazuh Configuration

## Wazuh Agent Installation
To collect and analyze events from endpoints, install the Wazuh Agent:

1. **Add Agent**: Navigate to the Wazuh dashboard and click "Add Agent".
2. **Server Address**: Enter the Wazuh server IP address.
3. **Install Agent**: Execute the command provided by Wazuh on the Windows 10 endpoint.
4. **Check Service**: Ensure the Wazuh service is running on the endpoint.
5. **Agent Activation**: Refresh the Wazuh dashboard to confirm the agent is active.

## Sysmon Log Forwarding Configuration
1. Open the `ossec.conf` file located at:
    ```bash
    C:\Program Files (x86)\ossec-agent\ossec.conf
    ```
2. As an Administrator, add the following to send Sysmon logs to Wazuh:

    ```xml
    <localfile>
      <location>Microsoft-Windows-Sysmon/Operational</location>
      <log_format>eventchannel</log_format>
    </localfile>
    ```

3. Optionally, for PowerShell logs, replace the `location` value:

    ```xml
    <localfile>
      <location>Microsoft-Windows-PowerShell/Operational</location>
      <log_format>eventchannel</log_format>
    </localfile>
    ```

4. Save and restart the Wazuh service.

## Mimikatz Detection
Mimikatz is used for credential dumping. Here's how to detect it:

1. **Download Mimikatz**: Disable Windows Defender, then download `mimikatz_trunk.zip`.
2. **Run Mimikatz**: Extract the zip and execute `mimikatz.exe` using PowerShell as Administrator.
3. **Check Logs**: Open Wazuh and monitor Sysmon logs. If Mimikatz isn't detected, edit the Wazuh configuration:
    - Set `<logall>` and `<logall_json>` to `yes` in `/var/ossec/etc/ossec.conf`.
    - Restart Wazuh Manager with:
      ```bash
      sudo systemctl restart wazuh-manager.service
      ```

## Creating a Custom Rule for Mimikatz
1. Navigate to the **Rules** section in Wazuh.
2. Add a custom rule in `local_rules.xml`:
    ```xml
    <rule id="100002" level="15">
      <if_group>sysmon_event1</if_group>
      <field name="win.eventdata.originalFileName" type="pcre2">(?i)mimikatz\.exe</field>
      <description>Malicious Activity : Mimikatz Detected!</description>
      <mitre>
        <id>T1003</id>
      </mitre>
    </rule>
    ```
3. **Test the Rule**: Run Mimikatz on the Windows machine again and check if the alert is triggered.

4. **Test with Renamed File**: Rename `mimikatz.exe` to another name (e.g., `svchost.exe`) and verify the rule still triggers based on the `originalFileName`.

This configuration allows real-time monitoring and detection of Mimikatz on Windows systems using Wazuh, ensuring that even if an attacker renames the file, the detection will still work.


# Shuffle SOAR Configuration

## Overview
This guide walks through setting up Shuffle, an open-source Security Orchestration, Automation, and Response (SOAR) platform, with Wazuh, VirusTotal, and TheHive for automated threat detection and response workflows.

## Workflow Creation
1. **Sign up for Shuffle** and navigate to **Workflows -> New Workflow**.
2. Provide a name and description for the project, then click **Done**.

## Webhook Setup
1. **Drag Webhook** into the workflow, rename it, and set the call section to `$exec` by selecting **Execution Argument**.
2. Connect Wazuh with Shuffle by adding the following to the `ossec.conf`:
    ```xml
    <integration>
      <name>Shuffle</name>
      <hook_url>YOUR_WEBHOOK_URL_HERE</hook_url>
      <alert_format>json</alert_format>
      <rule_id>100002</rule_id>
    </integration>
    ```
   - **Rule ID 100002** sends Mimikatz detections to Shuffle.

Restart Wazuh with:
```bash
systemctl restart wazuh-manager.service
systemctl status wazuh-manager.service
```

## Testing Workflow

1. **Run Mimikatz** on the Windows machine (`svchost.exe` if renamed).
2. **Test Workflow**: Click **Test Workflow** in Shuffle and review the output via the webhook icon.

## Extracting Hash with Regex

1. **Regex Setup**: Select **Regex capture group** in Shuffle.
2. **Input Data**: Use `$exec.text.win.eventdata.hashes` with the following regex:
   ```regex
   MD5=([0–9A-Fa-f]{32})
   ```
## VirusTotal Integration

1. **Sign up for VirusTotal** and retrieve your API key.
2. **Add VirusTotal** to the workflow in Shuffle and authenticate using the API key.
3. **Action**: Set to **Get a hash report** and connect it to the MD5 extraction step.

## TheHive Integration

1. **Add TheHive** to the workflow and authenticate with your API key.
2. **Set the alert fields**:
   - **Title**: `$exec.title`
   - **Tags**: `["T1003"]` (OS credential dumping technique)
   - **Severity**: 2
   - **Source**: Wazuh
   - **Description**: "Mimikatz Detected"
   - **Host**: `$exec.text.win.system.computer`
   - **User**: `$exec.text.win.eventdata.user`

## Sending Email Alerts

1. **Add Email to Workflow**: Input your email in the recipient field.
2. **Customize the body** with details like:
   ```bash
   $exec.text.win.eventdata.utcTime
   Host: $exec.text.win.system.computer
   User: $exec.text.win.eventdata.user
   CommandLine: $exec.text.win.eventdata.commandLine
   ```

   Final Testing

   Run Mimikatz: Execute the Mimikatz binary on the Windows machine.
   Verify: Check for alerts in TheHive and email notifications.
