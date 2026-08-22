# Timeline — Lab 58 In-Memory Execution Investigation

## Timeline Purpose

This timeline documents the creation, correction, execution, and investigation of the controlled PowerShell script, together with the observed Windows telemetry.

Actual timestamps from the endpoint should be used when finalizing the investigation.

---

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

---

# Phase 1 — Environment Preparation

## 06:14 — Investigation Directory Created

The directory:

    C:\InMemoryExecutionLab

was created successfully.

The directory initially contained no payload files.

---

# Phase 2 — Initial Script Creation

## 06:21:16 — Script Created

The controlled script:

    memory-test.ps1

was created.

Initial metadata included:

    CreationTime: 22-08-2026 06:21:16

---

# Phase 3 — Initial Script Failure

## Before Successful Execution

The original script contained:

    $Data=[System.Text.Encoding]::UTF8.GetBytes($Data)

This caused `$Data` to be null when passed to `GetBytes()`.

The resulting error was:

    Array cannot be null.
    Parameter name: chars

A second error occurred when `GetString()` received null bytes.

This represented a controlled scripting error, not malicious behavior.

---

# Phase 4 — Script Correction

## 06:30:57 — Script Modified

The script was corrected by initializing `$Data` before converting it to bytes.

The final file length was:

    218 bytes

The LastWriteTime became:

    22-08-2026 06:30:57

---

# Phase 5 — Telemetry Before Successful Execution

## 06:24:36 — Sysmon Event ID 1

A matching Sysmon Event ID 1 event was observed.

## 06:24:36 — PowerShell Event ID 4104

A matching PowerShell Script Block Logging event was observed.

## 06:26:23 — PowerShell Event ID 4104

Another matching 4104 event was observed.

These events established earlier PowerShell activity around the investigation period.

---

# Phase 6 — Successful Execution

## 06:32:08 — Sysmon Event ID 1

A matching Sysmon Event ID 1 event was observed.

This aligns closely with the successful execution window.

## 06:32:08 — PowerShell Event ID 4104

A matching Script Block Logging event was observed.

This provided script-level telemetry.

## 06:33:08 — Corrected Script Executed

The command:

    powershell.exe -ExecutionPolicy Bypass -File "C:\InMemoryExecutionLab\memory-test.ps1"

executed successfully.

The output was:

    Controlled in-memory processing completed
    LAB58-CONTROLLED_DATA

The execution time was recorded as:

    22 August 2026 06:33:08

---

# Phase 7 — Network Investigation

## After Successful Execution

Sysmon Event ID 3 was reviewed.

The endpoint contained multiple network events from approximately:

    06:13
    through
    06:45

The telemetry demonstrated that Sysmon network monitoring was active.

No specific suspicious network connection associated with the controlled in-memory processing was established.

---

# Phase 8 — Memory-Related Telemetry Checks

## Event ID 7

Sysmon Event ID 7 was queried.

Result:

    No matching events

Therefore, Image Load telemetry was unavailable.

## Event ID 10

Sysmon Event ID 10 was queried.

Result:

    No matching events

Therefore, Process Access telemetry was unavailable.

These limitations reduced the visibility available for investigating genuine memory-based attacks.

---

# Phase 9 — Security Process Telemetry

## Security Event ID 4688

Security Event ID 4688 was investigated.

No relevant event was available.

This created a process-creation telemetry gap.

The investigation relied on Sysmon Event ID 1 and PowerShell Event ID 4104 instead.

---

# Phase 10 — Filesystem Investigation

## Final Script State

The lab directory contained:

    C:\InMemoryExecutionLab\memory-test.ps1

No conventional executable payload was found in the lab directory.

## Temp Directory

A check for `.exe` files in the user's Temp directory returned no relevant results.

This supported the controlled nature of the simulation.

---

# Phase 11 — Process Enumeration

A recent-process query was attempted.

The command encountered:

    Exception getting "StartTime": "Access is denied"

The query nevertheless returned information for numerous processes.

The access-denied behavior was documented as a process-enumeration limitation.

---

# Phase 12 — Hash Comparison

## Initial Script Hash

Before correction:

    1FC341260A636B2D40595AEF4FF2762269B363C97A8A22E5B213DC4F103F011B

## Final Script Hash

After correction:

    2AE98EF3347C83F2CEB37DE23A1C4E37DE29038285110AAFE885A49DA2C97850

The hash changed because the script contents were legitimately modified.

---

# Final Evidence Summary

| Evidence | Result |
|---|---|
| PowerShell Execution | Confirmed |
| Sysmon Event ID 1 | Observed |
| PowerShell Event ID 4104 | Observed |
| Sysmon Event ID 3 | Available |
| Sysmon Event ID 7 | Unavailable |
| Sysmon Event ID 10 | Unavailable |
| Security Event ID 4688 | Unavailable |
| Conventional Payload | Not found |
| Process Enumeration | Partially limited by access denied |
| In-Memory Processing | Controlled behavior demonstrated |
| Malicious In-Memory Execution | Not confirmed |

---

# Final Assessment

The timeline establishes that controlled PowerShell processing occurred and that corresponding Sysmon and PowerShell telemetry was available.

However, the investigation did not establish malicious in-memory code execution.

The absence of Event IDs 7 and 10 and Security Event ID 4688 significantly limits the ability to make a stronger memory-execution determination.

---

# Investigation Conclusion

The evidence supports:

    PowerShell Execution
          +
    Controlled In-Memory Data Processing
          +
    Script Block Telemetry
          +
    Process Creation Telemetry

The evidence does not support:

    Confirmed Shellcode Execution
    Confirmed Process Injection
    Confirmed Reflective DLL Loading
    Confirmed Malicious In-Memory Execution

The final DFIR conclusion is therefore:

> Controlled in-memory processing was demonstrated, but malicious in-memory execution was not confirmed from the available endpoint evidence.
