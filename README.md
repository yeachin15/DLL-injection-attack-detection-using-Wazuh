

## 📝 Description

This project demonstrates how to set up a Security Operations Center (SOC) lab to detect **Process Injection Attacks** using **Wazuh (SIEM)** and Windows **Sysmon**. 

Process injection is a defense evasion technique where an attacker hides malicious code inside a legitimate Windows process. To safely simulate this behavior without using actual malware, this project utilizes **InjectProc**—an open-source emulation tool. Since InjectProc interacts with low-level Windows APIs and doesn't support all Windows versions, a specific compatible lab environment was engineered to capture the attack lifecycle.

**Attack Simulation: Emulating two common process injection techniques using InjectProc on a Windows endpoint.

**Telemetry Collection: Configuring Sysmon to capture kernel-level events (like Event ID 8: CreateRemoteThread).

**SIEM Detection: Writing custom Wazuh rules to trigger high-fidelity alerts and filter out false positives from legitimate applications like Google Chrome.


## so we set up our lab environment using the following:

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

click  block file (...) than keep> Delete  to select "keep anyway".

<img width="1398" height="692" alt="Capture5" src="https://github.com/user-attachments/assets/2220035e-303c-4528-ba93-49f0e9c49978" />

<img width="546" height="318" alt="Screenshot 2026-06-18 151330" src="https://github.com/user-attachments/assets/360a315f-f2c9-48b2-a937-1fdfc17ab5be" />

Google Chrome and WinRAR installed on the Windows endpoint.

An installed [Wazuh server](https://github.com/malwarekid/SOAR-Flow) running version.

<img width="802" height="42" alt="Screenshot 2026-06-18 152411" src="https://github.com/user-attachments/assets/1c0855be-94a8-4ca2-b00b-305c5ded176f" />




An installed Wazuh agent on the Windows endpoint

<img width="328" height="293" alt="Screenshot 2026-06-18 152821" src="https://github.com/user-attachments/assets/da2497ab-c861-444e-b543-2d71e003ef2e" />

[Sysmon](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon) installed on the Windows endpoint.

an download the Sysmon configuration file: [sysmonconfig.xml](sysmonconfig.xml).

<img width="1398" height="692" alt="Capture6" src="https://github.com/user-attachments/assets/11e56845-6976-4d6d-b9eb-5ef5da33c0d0" />

<img width="1125" height="593" alt="Capture7" src="https://github.com/user-attachments/assets/292407df-2cb7-429b-bcb2-7f8bbe8be980" />

using PowerShell (Run as administrator) and go sysmon folder run this command 
```.\sysmon64.exe -accepteula -i .\sysmonconfig.xml```

<img width="1115" height="628" alt="Capture8" src="https://github.com/user-attachments/assets/1eac7461-0362-4ef5-925e-a7a538e98358" />


Wazuh agent mechin to open this file "ossec.conf", C:\Program Files (x86)\ossec-agent and edit to collect sysmon logs
this Rules add tha currect place.
```
<localfile>
    <location>Microsoft-Windows-Sysmon/Operational</location>
    <log_format>eventchannel</log_format>
</localfile>
```
<img width="1162" height="611" alt="Capture9" src="https://github.com/user-attachments/assets/68b07a97-1f65-4da9-82f3-b26d21caa20b" />


Restart the Wazuh agent using PowerShell (Run as administrator) 
Restart-Service -Name wazuh
or manager apps to restart
windows star to search "manage" open tha apps than manage to restart

<img width="328" height="293" alt="Screenshot 2026-06-18 152821" src="https://github.com/user-attachments/assets/9a823deb-d830-46bc-9308-794161ae0479" />

Wazuh server mechin to open this file "local_rules.xml",
```sudo nano /var/ossec/etc/rules/local_rules.xml```

an add this Rules
```
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
```
<img width="1115" height="628" alt="Screenshot 2026-06-18 155658" src="https://github.com/user-attachments/assets/88cf547a-92eb-4aef-bef2-782416384beb" />

