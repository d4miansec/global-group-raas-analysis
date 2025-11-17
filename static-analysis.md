# Static Analysis

This sample is a C++ 32 bit portable executable file and was compiled for Windows Vista  on July 2025

### SHA256: 680c35d2b2343c0ad8d51accea696388bfba036476e6cf10cda597b5169061c9 ###
### MD5: 556baf26333a392f3377f41c26189d3c ###

<img width="723" height="447" alt="1_1GT87NcqWRI_QXbZ6EN-qw" src="https://github.com/user-attachments/assets/02cafc92-a13a-4cb8-b905-a2173b4baaa3" />

## 1. Imports & API Calls

### 4.1 High-level API categories
- **Crypto:** `CryptGenRandom` etc.
- **File I/O:** `CreateFileW`, `WriteFile`, `ReadFile`, `FindFirstFileExW`, `FindNextFileW`
- **Process & registry:** `RegSetValueExW`, `RegCreateKeyExW`, `CreateProcessW`, `TerminateProcess`
- **System controls:** `OpenSCManagerW`, `ControlService`, `TerminateProcess`, `DuplicateTokenExW`
- **Networking:** `WNetAddConnection2W`, `NetShareEnum`, `IcmpSendEcho`, `DnsQuery_W`
- **Anti-analysis:** `IsDebuggerPresent`
