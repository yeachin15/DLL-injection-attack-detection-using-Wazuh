# 🛡️ Detecting Process Injection Attacks with Wazuh & Sysmon

<p align="center">
<img src="https://img.shields.io/badge/Wazuh-SIEM-blue?style=for-the-badge">
<img src="https://img.shields.io/badge/Sysmon-Telemetry-green?style=for-the-badge">
<img src="https://img.shields.io/badge/Windows-10%2F11-blue?style=for-the-badge">
<img src="https://img.shields.io/badge/MITRE-T1055.001-red?style=for-the-badge">
</p>

---

## 📖 Overview

This project demonstrates how to build a Security Operations Center (SOC) detection lab capable of identifying **Process Injection Attacks** using **Wazuh SIEM** and **Microsoft Sysmon**.

Process Injection is a common **Defense Evasion** technique where malicious code is injected into legitimate Windows processes to evade detection. To safely emulate this behavior without using malware, this lab leverages **InjectProc**, an open-source process injection simulation tool.

The objective is to generate process injection telemetry on a Windows endpoint, collect it through Sysmon, forward it to Wazuh, and create custom detection rules that generate high-fidelity alerts while reducing false positives.

---

## 🎯 Project Objectives

### 🔥 Attack Simulation

Emulate common process injection techniques using **InjectProc** on a Windows endpoint.

### 📡 Telemetry Collection

Configure **Sysmon** to capture low-level Windows events, including:

* Event ID 8 – CreateRemoteThread
* Process Creation Events
* DLL Load Events

### 🚨 SIEM Detection

Develop custom **Wazuh rules** that:

* Detect suspicious injection activity
* Map detections to MITRE ATT&CK
* Suppress common false positives
* Generate analyst-friendly alerts

---

## 🏗️ Lab Architecture

```mermaid
flowchart LR

A[InjectProc Attack Simulation]
--> B[Windows Endpoint]

B --> C[Sysmon]

C --> D[Wazuh Agent]

D --> E[Wazuh Manager]

E --> F[Security Alert]
```

---

# ⚙️ Lab Setup

The following components were used to build the detection lab.

---

## 💻 Windows Endpoint

### Operating System

* Windows 10 / 11 (64-bit)

### Required Runtime

* Microsoft Visual C++ Redistributable (x64)
[Visual C++ Installed](https://www.microsoft.com/en-us/download/details.aspx?id=53840)


<img width="1398" height="692" alt="Capture2" src="https://github.com/user-attachments/assets/57157716-bbd4-4b8e-a4ec-e9adb654bf93" />

<img width="1398" height="730" alt="Capture1" src="https://github.com/user-attachments/assets/fb90d2bf-241e-4b22-95fc-f38e70aacd3b" />


---

## 🧪 Attack Simulation Tools

### InjectProc

InjectProc is used to emulate process injection techniques without deploying malware.

#### Installation Steps

1. Download [**InjectProc.exe**](https://github.com/secrary/InjectProc/releases)
2. Navigate to the **Assets** section of the release page
3. Download the executable
4. If Microsoft Defender blocks the file:

   * Click **(...)**
   * Click **Keep**
   * Click **Delete Drop Down**
   * Select **Keep Anyway**

<img width="1398" height="692" alt="Capture3" src="https://github.com/user-attachments/assets/5ca20f03-b6ad-445b-a22e-3258cb590180" />

<img width="563" height="341" alt="Screenshot 2026-06-18 150211" src="https://github.com/user-attachments/assets/5855757a-92e7-446e-a57d-45208275ab36" />

<img width="454" height="503" alt="Screenshot 2026-06-18 150249" src="https://github.com/user-attachments/assets/90f5cb01-d832-4e7d-bb50-cabf0885f0d5" />

---

### Test DLL

A benign DLL is used as the injection payload.

**File:** `hello-world-x64.dll`

Download the DLL from the Assets section of the release page.

[DLL Download](https://github.com/carterjones/hello-world-dll/releases)


<img width="1398" height="692" alt="Capture5" src="https://github.com/user-attachments/assets/05f6752c-d496-4575-8538-c4457255fb1d" />

<img width="546" height="318" alt="Screenshot 2026-06-18 151330" src="https://github.com/user-attachments/assets/c7d9bd48-9da9-4dd7-85ad-3efe68a1720d" />

---

## 🧰 Additional Software

The following applications were installed to provide legitimate target processes during testing:

* [Google Chrome](https://www.google.com/intl/en_uk/chrome/)
* [WinRAR](https://www.win-rar.com/download.html?&L=0)



---

## 🛡️ Wazuh Infrastructure

### Wazuh Manager

A dedicated [Wazuh server](https://github.com/malwarekid/SOAR-Flow) was deployed to collect and analyze endpoint telemetry.

<img width="802" height="42" alt="Screenshot 2026-06-18 152411" src="https://github.com/user-attachments/assets/1c0855be-94a8-4ca2-b00b-305c5ded176f" />

---

### Wazuh Agent

A [Wazuh agent](https://documentation.wazuh.com/current/installation-guide/wazuh-agent/index.html) was installed on the Windows endpoint to forward Sysmon events.

<img width="328" height="293" alt="Screenshot 2026-06-18 152821" src="https://github.com/user-attachments/assets/da2497ab-c861-444e-b543-2d71e003ef2e" />


---

## 🔍 Sysmon Deployment

Sysmon was installed to provide enhanced Windows telemetry.

### Download Components

* [Sysmon64.exe](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon)
* [sysmonconfig.xml](sysmonconfig.xml)

<img width="1398" height="692" alt="Capture6" src="https://github.com/user-attachments/assets/11e56845-6976-4d6d-b9eb-5ef5da33c0d0" />

<img width="1125" height="593" alt="Capture7" src="https://github.com/user-attachments/assets/292407df-2cb7-429b-bcb2-7f8bbe8be980" />

### install sysmon

Install Sysmon using an elevated PowerShell(Run as administrator) session:

Go to Sysmon folder an run this command.

```powershell
.\sysmon64.exe -accepteula -i .\sysmonconfig.xml
```

<img width="1115" height="628" alt="Capture8" src="https://github.com/user-attachments/assets/1eac7461-0362-4ef5-925e-a7a538e98358" />

---

## ✅ Environment Summary

| Component       | Purpose              |
| --------------- | -------------------- |
| Windows 10/11   | Test Endpoint        |
| Sysmon          | Telemetry Collection |
| Wazuh Agent     | Log Forwarding       |
| Wazuh Manager   | SIEM Platform        |
| InjectProc      | Attack Emulation     |
| Hello World DLL | Injection Payload    |
| Chrome / WinRAR | Target Processes     |


## 📝 Description

This project demonstrates how to set up a Security Operations Center (SOC) lab to detect **Process Injection Attacks** using **Wazuh (SIEM)** and Windows **Sysmon**. 

Process injection is a defense evasion technique where an attacker hides malicious code inside a legitimate Windows process. To safely simulate this behavior without using actual malware, this project utilizes **InjectProc**—an open-source emulation tool. Since InjectProc interacts with low-level Windows APIs and doesn't support all Windows versions, a specific compatible lab environment was engineered to capture the attack lifecycle.

**Attack Simulation: Emulating two common process injection techniques using InjectProc on a Windows endpoint.

**Telemetry Collection: Configuring Sysmon to capture kernel-level events (like Event ID 8: CreateRemoteThread).

**SIEM Detection: Writing custom Wazuh rules to trigger high-fidelity alerts and filter out false positives from legitimate applications like Google Chrome.
so we set up our lab environment using the following:

## Lab Setup

Windows 10/11 64-bit, Microsoft Visual C++ installed ([vc_redist.x64.exe](https://www.microsoft.com/en-us/download/details.aspx?id=53840)).


<img width="1398" height="692" alt="Capture2" src="https://github.com/user-attachments/assets/57157716-bbd4-4b8e-a4ec-e9adb654bf93" />


<img width="1398" height="730" alt="Capture1" src="https://github.com/user-attachments/assets/fb90d2bf-241e-4b22-95fc-f38e70aacd3b" />




[InjectProc](https://github.com/secrary/InjectProc/releases) downloaded and installed on the Windows endpoint.

go GitHub repository> Assets> InjectProc.exe

click block file (...) than keep> Delete  to select "keep anyway". 


<img width="1398" height="692" alt="Capture3" src="https://github.com/user-attachments/assets/5ca20f03-b6ad-445b-a22e-3258cb590180" />


<img width="1398" height="692" alt="Capture4" src="https://github.com/user-attachments/assets/44c2f5ea-72b2-464a-9e51-52fcc5613698" />


<img width="563" height="341" alt="Screenshot 2026-06-18 150211" src="https://github.com/user-attachments/assets/dc4c6bb9-f617-415a-86a2-6f86444c97d0" />


<img width="454" height="503" alt="Screenshot 2026-06-18 150249" src="https://github.com/user-attachments/assets/fb99d38a-50e3-4cb0-83cc-f5f02c7e26a1" />


[hello-world-x64.dll](https://github.com/carterjones/hello-world-dll/releases) downloaded on the Windows endpoint.


go GitHub repository> Assets> hello-world-x64.dll



<img width="1398" height="692" alt="Capture5" src="https://github.com/user-attachments/assets/2220035e-303c-4528-ba93-49f0e9c49978" />


<img width="546" height="318" alt="Screenshot 2026-06-18 151330" src="https://github.com/user-attachments/assets/360a315f-f2c9-48b2-a937-1fdfc17ab5be" />


[Google Chrome](https://www.google.com/intl/en_uk/chrome/) and [WinRAR](https://www.win-rar.com/download.html?&L=0) installed on the Windows endpoint.

An installed [Wazuh server](https://github.com/malwarekid/SOAR-Flow) running version.


<img width="802" height="42" alt="Screenshot 2026-06-18 152411" src="https://github.com/user-attachments/assets/1c0855be-94a8-4ca2-b00b-305c5ded176f" />


An installed Wazuh agent on the Windows endpoint


<img width="328" height="293" alt="Screenshot 2026-06-18 152821" src="https://github.com/user-attachments/assets/da2497ab-c861-444e-b543-2d71e003ef2e" />


[Sysmon](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon) installed on the Windows endpoint.

an download the Sysmon configuration file: [sysmonconfig.xml](sysmonconfig.xml).


<img width="1398" height="692" alt="Capture6" src="https://github.com/user-attachments/assets/11e56845-6976-4d6d-b9eb-5ef5da33c0d0" />


<img width="1125" height="593" alt="Capture7" src="https://github.com/user-attachments/assets/292407df-2cb7-429b-bcb2-7f8bbe8be980" />


using PowerShell (Run as administrator) and go sysmon folder run this command 
```PowerShell
.\sysmon64.exe -accepteula -i .\sysmonconfig.xml
```

<img width="1115" height="628" alt="Capture8" src="https://github.com/user-attachments/assets/1eac7461-0362-4ef5-925e-a7a538e98358" />


Wazuh agent mechin to open this file "ossec.conf", C:\Program Files (x86)\ossec-agent and edit to collect sysmon logs
this Rules add tha currect place.
```Rules
<localfile>
    <location>Microsoft-Windows-Sysmon/Operational</location>
    <log_format>eventchannel</log_format>
</localfile>
```


<img width="1162" height="611" alt="Capture9" src="https://github.com/user-attachments/assets/68b07a97-1f65-4da9-82f3-b26d21caa20b" />


Restart the Wazuh agent using PowerShell (Run as administrator) 
```PowerShell
Restart-Service -Name wazuh
```

or manager apps to restart
windows star to search "manage" open tha apps than manage to restart


<img width="328" height="293" alt="Screenshot 2026-06-18 152821" src="https://github.com/user-attachments/assets/9a823deb-d830-46bc-9308-794161ae0479" />


Wazuh server mechin to open this file "local_rules.xml",
```bash
sudo nano /var/ossec/etc/rules/local_rules.xml
```

an add this Rules
```` Rules
<group name="windows,sysmon">
  <rule id="100200" level="12">
    <if_sid>61610</if_sid>
    <description>Possible process injection activity detected from "$(win.eventdata.sourceImage)" on "$(win.eventdata.targetImage)"</description>
    <mitre>
      <id>T1055.001</id>
    </mitre>
  </rule>
 
  <rule id="100100" level="0">
    <if_sid>100200</if_sid>
    <field name="win.eventdata.sourceImage" type="pcre2">(C:\\\\Windows\\\\system32)|chrome.exe</field>
    <description>Ignore Windows binaries and Chrome</description>
  </rule>
</group>
````


<img width="1115" height="628" alt="Screenshot 2026-06-18 155658" src="https://github.com/user-attachments/assets/88cf547a-92eb-4aef-bef2-782416384beb" />

