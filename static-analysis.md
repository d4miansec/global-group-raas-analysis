# Static Analysis

This report documents the static analysis of the “Global Group RaaS” ransomware sample.
The executable is a 32-bit C++ PE compiled for Windows Vista (Timestamp: July 2025) and exhibits typical RaaS behavior including network propagation, credential usage, and data encryption techniques.

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

| API                   | Purpose                    | MITRE ATT&CK                         |
| --------------------- | -------------------------- | ------------------------------------ |
| `DnsQuery_W`          | Internal network discovery | T1018 – Remote System Discovery      |
| `WNetAddConnection2W` | Connect to SMB shares      | T1021.002 – SMB/Windows Admin Shares |
| `ClearEventLogW`      | Anti-forensics             | T1070 – Indicator Removal            |

### 2. Strings

This section summarizes interesting or suspicious strings discovered during static analysis.
Strings can reveal configuration data, ransom note fragments, internal function names, debug output, and artifacts related to persistence or network activity.

These strings typically indicate file-handling behavior, enumeration, or encryption activity:

<img width="825" height="154" alt="image" src="https://github.com/user-attachments/assets/399fd6ad-155c-41c6-937b-930e517e3313" />

Strings referencing networking APIs, hostnames, or network enumeration:

<img width="1013" height="423" alt="image" src="https://github.com/user-attachments/assets/c912bdfe-1b67-4d4f-9f41-3f1cce98526d" />

These messages appear to be internal debug traces. They help infer execution flow and validate ransomware functions:

<img width="949" height="570" alt="image" src="https://github.com/user-attachments/assets/11b53afc-17b8-48b8-a2f7-adf8b1541e9c" />

Ransom Note Fragments:

<img width="1055" height="52" alt="image" src="https://github.com/user-attachments/assets/7bcfe22e-3f2e-42d0-8075-9570e259c51e" />











## 2. Ransom Note

The content of the ransom note is stored as an encoded string in the executable. During runtime it is decoded and written to each affected directory.

<img width="1041" height="848" alt="1_BagCF6T5p-M6-AxrGh8KGg" src="https://github.com/user-attachments/assets/733bab2b-4809-4ffd-a80f-079806b19bcc" />

## 3. Initialization

Global Ransomware first sets the process’s priority class to **HIGH_PRIORITY_CLASS** using **SetPriorityClass**.
This ensures the ransomware maintains high CPU priority over other processes, increasing the speed of file enumeration and encryption.

<img width="808" height="169" alt="1_WHvIDFMsRAz9RPnnaaVehQ" src="https://github.com/user-attachments/assets/c8b3eb2c-69e6-416d-9417-2f5dfdc2c627" />

## 4. Command Line Arguments

The ransomware requires a password passed by the **-code** command line argument in order to start sucessfully.
Below is the list of all command-line arguments it accepts:

| Argument        | Description                                                                                   |
| --------------- | --------------------------------------------------------------------------------------------- |
| **-log**        | Enables console logging for debugging or verbose output.                                      |
| **-force**      | Allows multiple instances of the ransomware to run simultaneously, bypassing the mutex check. |
| **-detached**   | Starts encryption in a detached child process, improving stealth and resilience.              |
| **-code**       | Required password used to unlock and start the ransomware.                                    |
| **-path**       | Specifies a custom directory path to target for encryption.                                   |
| **-threads**    | Sets the number of worker threads used during encryption operations.                          |
| **-delay**      | Applies a delay (in seconds) before encryption begins.                                        |
| **-time**       | Delays execution until a specific HH:MM time is reached.                                      |
| **-host**       | Targets a specific remote host for encryption or propagation.                                 |
| **-skip-local** | Avoids encrypting local drives, focusing exclusively on network shares.                       |
| **-skip-net**   | Skips network scanning and network-share encryption, targeting only local drives.             |
| **-ldap**       | Enables LDAP-based domain spreading for lateral movement.                                     |
| **-u**          | Specifies the username used for LDAP authentication during network spreading.                 |
| **-p**          | Specifies the password used for LDAP authentication during network spreading.                 |

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


