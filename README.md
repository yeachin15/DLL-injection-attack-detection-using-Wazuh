
## 📝 Description

This project demonstrates how to set up a Security Operations Center (SOC) lab to detect **Process Injection Attacks** using **Wazuh (SIEM)** and Windows **Sysmon**. 

Process injection is a defense evasion technique where an attacker hides malicious code inside a legitimate Windows process. To safely simulate this behavior without using actual malware, this project utilizes **InjectProc**—an open-source emulation tool. Since InjectProc interacts with low-level Windows APIs and doesn't support all Windows versions, a specific compatible lab environment was engineered to capture the attack lifecycle.

**Attack Simulation: Emulating two common process injection techniques using InjectProc on a Windows endpoint.

**Telemetry Collection: Configuring Sysmon to capture kernel-level events (like Event ID 8: CreateRemoteThread).

**SIEM Detection: Writing custom Wazuh rules to trigger high-fidelity alerts and filter out false positives from legitimate applications like Google Chrome.

so we set up our lab environment using the following:

Windows 10/11 64-bit, Microsoft Visual C++ installed ([vc_redist.x64.exe]((https://www.microsoft.com/en-us/download/details.aspx?id=53840)).
<img width="1398" height="730" alt="Capture1" src="https://github.com/user-attachments/assets/fb90d2bf-241e-4b22-95fc-f38e70aacd3b" />





InjectProc installed on the Windows endpoint. This is downloaded from their GitHub repository.






hello-world-x64.dll downloaded on the Windows endpoint from here. This DLL will help us confirm that the process injection attacks are successful. It displays a “Hello world!” message whenever it is run.








Google Chrome and WinRAR installed on the Windows endpoint.

An installed Wazuh server running version 4.3.6.




An installed and enrolled Wazuh agent on the Windows endpoint running version 4.3.6.



Sysmon installed on the Windows endpoint.





sudo nano /var/ossec/etc/rules/local_rules.xml













Technique 1: DLL injection T1055.001
Malware authors often use this technique to inject their code into another process. The malware writes the path to a malicious dynamic-link library (DLL) in the virtual address space of another process. It then ensures that the target process loads the DLL by creating a remote thread. To demonstrate this technique using InjectProc, we do the following:

Launch a Command Prompt as an administrator and navigate to the InjectProc directory.
Run the command:
InjectProc.exe dll_inj hello-world-x64.dll cmd.exe
This command injects the file hello-world-x64.dll into a cmd.exe process and runs the DLL code under the context of the target process. Any non-critical running process can be used but in our example, we use cmd.exe. After running the command, we get an “Injected Successfully” message in the Command Prompt and a “Hello world!” message.

Detection with Wazuh
To detect the process injection techniques described in this post, we need to install Sysmon on the Windows endpoint and configure the Wazuh agent to collect Sysmon logs using the following steps:

Download Sysmon from the Microsoft Sysinternals page.
Download the Sysmon configuration file: sysmonconfig.xml.
Install Sysmon with the downloaded configuration file using PowerShell (as administrator):
.\sysmon64.exe -accepteula -i .\sysmonconfig.xml
Edit the Wazuh agent ossec.conf file to specify the location to collect Sysmon logs:
<localfile>
    <location>Microsoft-Windows-Sysmon/Operational</location>
    <log_format>eventchannel</log_format>
</localfile>
5. Restart the Wazuh agent to apply the changes by running the following PowerShell command as an administrator:

Restart-Service -Name wazuh
Detection rules
When we run the InjectProc command and look at the Sysmon logs. We see that it tries to create a remote thread in cmd.exe using the CreateRemoteThread function. The CreateRemoteThread function is used by applications to create a thread that runs in the virtual address space of another process. The sysmon event can be seen below:

EventID: 8
CreateRemoteThread detected:
SourceProcessGuid: {58b1d23b-d824-6299-bb06-000000000400}
SourceProcessId: 4284
SourceImage: C:\Users\win10\Downloads\InjectProc.exe
TargetProcessGuid: {58b1d23b-d645-6299-6706-000000000400}
TargetProcessId: 3340
TargetImage: C:\Windows\System32\cmd.exe
NewThreadId: 6716
We add the following detection rules to the /var/ossec/etc/rules/local_rules.xml file on the Wazuh manager:

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
The rule 100100 is used to ignore alerts created by legitimate applications like Microsoft Windows utilities and Google Chrome. Users can edit this rule to include applications and paths specific to their environment.

Technique 2: Process hollowing (process replacement) T1055.012
In process hollowing, the malware does not inject code into a host program, instead, it unmaps (hollows out) the legitimate code from the memory of the target process, and overwrites the memory space with a malicious executable. An example of malware that uses this technique is Lokibot, a widely distributed information stealer that uses process hollowing to inject itself into a legitimate Windows process. To demonstrate this technique using InjectProc, we do the following:

1. Launch a Command Prompt terminal as an administrator and switch to the InjectProc directory.

2. Run the command:

InjectProc.exe proc_rpl "C:\Program Files\Google\Chrome\Application\chrome.exe" "C:\Program Files\WinRAR\WinRAR.exe"
This command attempts to inject the WinRAR executable file into Google Chrome. If the command runs successfully, a WinRAR window opens under the context of Google Chrome. When we take a look at the Windows Task Manager, we see a running WinRAR process under the Google Chrome process tree:


Detection rule
From the Sysmon logs, we see an event generated showing that our target image (chrome.exe) has been tampered with:

EventID: 25
Process Tampering:
RuleName: -
ProcessGuid: {58b1d23b-da26-6299-c606-000000000400}
ProcessId: 8188
Image: C:\Program Files\Google\Chrome\Application\chrome.exe
Type: Image is replaced    
We can go ahead to create a rule for this event. To do this, we add the following rule to the /var/ossec/etc/rules/local_rules.xml file on the Wazuh manager:

<group name="windows,sysmon">
  <rule id="100201" level="12">
    <if_sid>61600</if_sid>
    <description>Process injection activity detected: "$(win.eventdata.Image)" has been tampered with</description>
    <mitre>
      <id>T1055.012</id>
    </mitre>
  </rule>
</group>
Once the rules have been added, we restart the manager to apply the changes using the command below:

systemctl restart wazuh-manager
Detection results
To confirm the rules work, we run the InjectProc commands again in a new administrator command prompt window. The rules we have created detect each process injection attempt as seen below:


Conclusion
Process Injection is a technique widely used by malicious actors. Therefore, Organizations should have mechanisms to detect this activity within their network. This post has shown how to detect process injection techniques in order to protect critical infrastructure using Wazuh.

References
Process Injection, Technique T1055 – Enterprise | MITRE ATT&CK
