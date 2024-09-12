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

## Conclusion
This project integrates Wazuh, Shuffle, and TheHive for real-time detection and response. By automating workflows, it streamlines SOC operations and reduces threat response time.

