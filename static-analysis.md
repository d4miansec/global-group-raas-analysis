# Static Analysis

This sample is a C++ 32 bit portable executable file and was compiled for Windows Vista  on July 2025

### SHA256: 680c35d2b2343c0ad8d51accea696388bfba036476e6cf10cda597b5169061c9 ###
### MD5: 556baf26333a392f3377f41c26189d3c ###

<img width="723" height="447" alt="1_1GT87NcqWRI_QXbZ6EN-qw" src="https://github.com/user-attachments/assets/02cafc92-a13a-4cb8-b905-a2173b4baaa3" />

## 1. Imports & API Calls

### 1.1 High-level API categories
- **Crypto:** `CryptGenRandom` etc.
- **File I/O:** `CreateFileW`, `WriteFile`, `ReadFile`, `FindFirstFileExW`, `FindNextFileW`
- **Process & registry:** `RegSetValueExW`, `RegCreateKeyExW`, `CreateProcessW`, `TerminateProcess`
- **System controls:** `OpenSCManagerW`, `ControlService`, `TerminateProcess`, `DuplicateTokenExW`
- **Networking:** `WNetAddConnection2W`, `NetShareEnum`, `IcmpSendEcho`, `DnsQuery_W`
- **Anti-analysis:** `IsDebuggerPresent`

### 1.2 Potentially malicious imports

<img width="740" height="164" alt="image" src="https://github.com/user-attachments/assets/3016d699-c849-4e07-89ca-4512ba78fc58" />

dnsapi.dll -> DnsQuery_W
    Attackers can use DNS queries to discover valid domain names within a network. By querying for various records, they can enumerate active devices, services, and their configurations.
    
mpr.dll -> WNetAddConnection2W
    Ransomware might leverage WNetAddConnection2W to connect to shared drives where sensitive data is stored, thereby facilitating data encryption or exfiltration
    
advapi32.dll -> ClearEventLogW
    By clearing event logs, attackers can hide indicators of compromise (IoCs) and other suspicious behaviors, making detection less likely during investigations.


## 2. Ransom Note

The content of the ransom note is stored as an encoded string in the executable

<img width="1041" height="848" alt="1_BagCF6T5p-M6-AxrGh8KGg" src="https://github.com/user-attachments/assets/733bab2b-4809-4ffd-a80f-079806b19bcc" />

## 3. Initialization

Global Ransomware first sets the process's priority class to HIGH_PRIORITY_CLASS using the SetPriorityClass API function.
This strategic action ensures that the ransomware operates with enhanced efficiency and responsiveness, allowing it to execute its malicious tasks more effectively.

<img width="808" height="169" alt="1_WHvIDFMsRAz9RPnnaaVehQ" src="https://github.com/user-attachments/assets/c8b3eb2c-69e6-416d-9417-2f5dfdc2c627" />

## 4. Command Line Arguments

The ransomware requires a password passed by the -code command line argument in order to start sucessfully.
Below is the list of all command-line arguments it accepts:

- **-log :** Enables console logging


