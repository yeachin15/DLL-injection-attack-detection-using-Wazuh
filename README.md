

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



* Download the injection tool from here: [InjectProc Releases](https://github.com/secrary/InjectProc/releases)
* Download the sample 64-bit DLL for testing: [hello-world-x64.dll Releases](https://github.com/carterjones/hello-world-dll/releases)

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




