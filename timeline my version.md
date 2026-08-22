# Timeline 

# Investigation Timeline

| Order | Time | Source | Activity | Result |
|---:|---|---|---|---|
| 1 | 06:14 | File System | Investigation directory created | `C:\InMemoryExecutionLab` established |
| 2 | 06:21:16 | File System | Script created | `memory-test.ps1` created |
| 3 | Before 06:30 | PowerShell | Initial script version created | Variable-order error present |
| 4 | Before successful execution | PowerShell | First execution attempt | `GetBytes()` / `GetString()` errors |
| 5 | 06:30:57 | File System | Script corrected | LastWriteTime updated |
| 6 | 06:32:08 | Sysmon | Event ID 1 observed | PowerShell process creation |
| 7 | 06:32:08 | PowerShell | Event ID 4104 observed | Script Block Logging |
| 8 | 06:33:08 | PowerShell | Corrected script executed | Controlled output produced |
| 9 | 06:33:08 | PowerShell | Execution timestamp recorded | Successful execution reference |
| 10 | 06:33+ | Sysmon | Event ID 3 reviewed | Network telemetry available |
| 11 | 06:33+ | Sysmon | Event ID 7 checked | No events available |
| 12 | 06:33+ | Sysmon | Event ID 10 checked | No events available |
| 13 | 06:33+ | Security | Event ID 4688 checked | Relevant event unavailable |
| 14 | 06:33+ | Process | Running processes examined | Access-denied limitation encountered |
| 15 | 06:33+ | File System | Lab directory reviewed | No conventional payload found |
| 16 | 06:33+ | Temp | `.exe` files checked | No relevant results |
| 17 | Final | DFIR Analysis | Evidence correlated | Malicious in-memory execution not confirmed |

