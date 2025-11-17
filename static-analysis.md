# Static Analysis

This sample is a C++ 32 bit portable executable file and was compiled for Windows Vista  on July 2025

**SHA-256:** 680c35d2b2343c0ad8d51accea696388bfba036476e6cf10cda597b5169061c9  
**MD5:**     556baf26333a392f3377f41c26189d3c

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

Global Ransomware first sets the process's priority class to **HIGH_PRIORITY_CLASS** using the **SetPriorityClass** API function.
This strategic action ensures that the ransomware operates with enhanced efficiency and responsiveness, allowing it to execute its malicious tasks more effectively.

<img width="808" height="169" alt="1_WHvIDFMsRAz9RPnnaaVehQ" src="https://github.com/user-attachments/assets/c8b3eb2c-69e6-416d-9417-2f5dfdc2c627" />

## 4. Command Line Arguments

The ransomware requires a password passed by the **-code** command line argument in order to start sucessfully.
Below is the list of all command-line arguments it accepts:

- **-log :** Enables console logging
- **-force :** Enables multi-instance running
- **-detached :** Start encryption in child process
- **-code :** Password for starting the file
- **-path :** Specify path for encryption
- **-threads :** Sets the number of threads to use for encryption operations
- **-delay :** Specifies a delay in seconds before starting the encryption process
- **-time :** Sets a specific time (in HH:MM format) to wait until before starting encryption
- **-host :** Targets a specific host for encryption or spreading
- **-skip-local :** Skips encryption of local drives, focusing only on network shares
- **-skip-net :** Skips network scanning and encryption, focusing only on local drives
- **-ldap :** Enables LDAP-based domain spreading for network propagation
- **-u :** Specifies the username for LDAP authentication in network spreading
- **-p :** Specifies the password for LDAP authentication in network spreading

<img width="645" height="542" alt="1_6XMa0lFBzoCJXtEdVhgZVg" src="https://github.com/user-attachments/assets/a43cabb9-fcae-478a-8630-56e3c48d32f4" />

## 5. Run-Once Mutex

The ransomware checks for another instance of itself by looking for a specific mutex using **CreateMutexW**.
If the mutex already exists, it cleans up and exits instantly. However, this check can be bypassed by using the **-force** command line option, allowing multiple instances to run on the same system.

<img width="627" height="417" alt="1_Kby5q6gHQLSpZMlcEEnmEg" src="https://github.com/user-attachments/assets/d374131d-7ad1-4a35-b220-f3d6555524e9" />

## 6. Deleting Event Logs and Shadow Copies

To hide its tracks, the malware deletes the Event Log.
the function sets up an array of event log types and goes through each one. It tries to open each log using **OpenEventLogW**.
If it successfully opens a log, it clears it with **ClearEventLogW**. If clearing fails, it backs up the log to **NUL**, which means it discards the log.

<img width="506" height="505" alt="1_w0rP7NVQN8HvcBBN6xWUoA" src="https://github.com/user-attachments/assets/6f379e5d-449c-4ec5-afa4-160571c62d76" />

For deleting shadow copies, it prepares the CMD command and executes it using **CreateProcessW**.

<img width="572" height="362" alt="1_oWpWCIlHQrsSiLmfLpiHWQ" src="https://github.com/user-attachments/assets/6e9d09a5-08bf-488a-a4b9-1c53e620244c" />

### cmd.exe /c vssadmin delete shadows /all /quiet ###

<img width="653" height="291" alt="1_RfF3Q6hJxKudk7B3h1kpnw" src="https://github.com/user-attachments/assets/c94ed752-67fd-4d4c-8df9-78145080c7b1" />


