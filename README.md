# Windows DFIR Lab 58 — In-Memory Execution Investigation

## Overview

This lab investigates the forensic challenges associated with possible in-memory execution on a Windows endpoint. The investigation was designed as a safe simulation rather than a reproduction of real shellcode execution, reflective DLL injection, process injection, or other malicious memory-execution techniques.

A controlled PowerShell script was used to perform harmless in-memory data processing. The investigation then examined process creation, PowerShell Script Block Logging, network telemetry, filesystem state, and the availability of memory-related Sysmon events.

The lab demonstrated an important DFIR problem: suspicious memory-oriented behavior may not leave a conventional executable payload on disk, and no single Windows event necessarily proves that code executed from memory.

---

# Investigation Objectives

- Establish a filesystem baseline before controlled execution.
- Investigate PowerShell activity associated with controlled in-memory processing.
- Correlate process execution with PowerShell Script Block Logging.
- Examine available network telemetry around the execution timeframe.
- Determine whether a conventional payload executable was written to disk.
- Check whether Sysmon Image Load telemetry is available.
- Check whether Sysmon Process Access telemetry is available.
- Investigate Windows Security Event ID 4688 as an independent process source.
- Document execution errors and corrections encountered during the lab.
- Distinguish controlled memory processing from confirmed malicious in-memory execution.
- Identify telemetry gaps that would require memory or EDR analysis in a real incident.

---

# Investigation Scenario

A SOC analyst investigates suspicious PowerShell activity on a Windows workstation. The process appears to manipulate data in memory, but no obvious executable payload is found in the investigation directory or the checked temporary location.

The analyst must determine:

- What PowerShell process performed the activity.
- What script content was executed.
- Whether a conventional payload existed on disk.
- Whether suspicious child processes were created.
- Whether network communication followed the activity.
- Whether Image Load or Process Access telemetry is available.
- Whether the available evidence is sufficient to support an in-memory execution hypothesis.

The investigation is performed using harmless laboratory data.

---

# Lab Environment

| Component | Details |
|---|---|
| Operating System | Windows |
| Investigation Type | Host-Based DFIR |
| Investigation Directory | `C:\InMemoryExecutionLab` |
| Primary Process | PowerShell |
| Sysmon Event ID 1 | Observed |
| Sysmon Event ID 3 | Observed |
| Sysmon Event ID 7 | Not available |
| Sysmon Event ID 10 | Not available |
| PowerShell Event ID 4104 | Observed |
| Windows Security Event ID 4688 | Not available |

---

# Safety Boundaries

This lab does not execute:

- Shellcode
- Reflective DLL injection
- Process injection
- Credential theft
- Malware
- Arbitrary machine code
- Real malicious payloads

The controlled script only converts a harmless string to UTF-8 bytes and back to text.

---

# Investigation Workflow

1. Create the investigation directory.
2. Establish an empty filesystem baseline.
3. Create a controlled PowerShell script.
4. Record the initial script hash.
5. Attempt script execution.
6. Investigate and correct the script error.
7. Execute the corrected script with a process-level Execution Policy bypass.
8. Record the execution timestamp.
9. Investigate Sysmon Event ID 1.
10. Investigate PowerShell Event ID 4104.
11. Investigate Windows Security Event ID 4688.
12. Investigate Sysmon Event ID 3.
13. Check Sysmon Event ID 7.
14. Check Sysmon Event ID 10.
15. Review running processes.
16. Search for conventional payload files.
17. Recalculate the corrected script hash.
18. Correlate the evidence.
19. Document telemetry gaps.
20. Determine whether in-memory execution can actually be confirmed.

---

# Controlled Script

The final corrected script performed:

    $Data = "LAB58-CONTROLLED_DATA"

    $Bytes = [System.Text.Encoding]::UTF8.GetBytes($Data)

    $Decoded = [System.Text.Encoding]::UTF8.GetString($Bytes)

    Write-Output "Controlled in-memory processing completed"
    Write-Output $Decoded

This script performs harmless data transformation in memory.

It does not execute arbitrary memory, inject into another process, load a malicious DLL, or execute shellcode.

---

# Initial Script Error

The first script version contained an ordering error:

    $Data=[System.Text.Encoding]::UTF8.GetBytes($Data)

The variable `$Data` had not been initialized before `GetBytes()` was called.

This produced an `ArgumentNullException`.

The next operation also failed because `$Bytes` was null.

The script nevertheless reached its final output statements, which demonstrated why output alone should not be treated as proof that every preceding operation succeeded.

The script was subsequently corrected before the successful execution.

---

# Successful Execution

The corrected script was executed with:

    powershell.exe -ExecutionPolicy Bypass -File "C:\InMemoryExecutionLab\memory-test.ps1"

The successful output was:

    Controlled in-memory processing completed
    LAB58-CONTROLLED_DATA

The execution time was recorded as approximately:

    22 August 2026 06:33:08

The process-level Execution Policy bypass was used without changing the system-wide Execution Policy.

---

# Sysmon Event ID 1

Sysmon Event ID 1 was observed.

The investigation searched for events associated with:

    memory-test
    powershell.exe

Three relevant events were returned at:

    22-08-2026 06:13:37
    22-08-2026 06:24:36
    22-08-2026 06:32:08

The 06:32:08 event aligns with the successful execution window.

The event was used to establish process creation evidence.

---

# PowerShell Event ID 4104

PowerShell Event ID 4104 was available.

Three matching events were identified:

    22-08-2026 06:24:36
    22-08-2026 06:26:23
    22-08-2026 06:32:08

The search included terms related to:

    LAB58-CONTROLLED_DATA
    memory-test
    GetBytes
    GetString

This provided script-level evidence of the controlled PowerShell activity.

---

# Windows Security Event ID 4688

Security Event ID 4688 was investigated but was not available for the relevant activity.

This created a process-creation telemetry gap.

The investigation therefore relied primarily on:

- Sysmon Event ID 1
- PowerShell Event ID 4104
- Direct process information
- Filesystem evidence

The absence of 4688 was not interpreted as evidence that PowerShell did not execute.

---

# Sysmon Event ID 3

Sysmon Event ID 3 was available.

The investigation returned multiple network events, including events throughout the 06:13–06:45 timeframe.

The available results demonstrated that Sysmon network telemetry was functioning.

However, the reviewed evidence did not establish a specific suspicious network connection caused by the controlled in-memory processing.

Therefore, the investigation did not classify the observed Event ID 3 activity as malicious.

---

# Sysmon Event ID 7

Sysmon Event ID 7 was queried:

    Get-WinEvent -FilterHashTable @{
        LogName="Microsoft-Windows-Sysmon/Operational"
        Id=7
    } -MaxEvents 20

No events were returned.

Therefore, Image Load telemetry was not available for this investigation.

This is significant because Image Load telemetry could potentially provide additional context in investigations involving DLL-based memory execution.

---

# Sysmon Event ID 10

Sysmon Event ID 10 was also queried.

No matching events were returned.

Therefore, Process Access telemetry was not available for this investigation.

This limited the ability to investigate memory-access relationships between processes.

---

# Filesystem Investigation

The investigation directory contained only:

    C:\InMemoryExecutionLab\memory-test.ps1

No conventional executable payload was found in the lab directory.

The checked Temp directory also did not return the expected `.exe` payload artifacts.

This supported the controlled nature of the scenario.

The absence of a payload file did not itself prove memory-only execution.

---

# File Metadata

The final script metadata included:

    Name: memory-test.ps1
    Length: 218 bytes
    CreationTime: 22-08-2026 06:21:16
    LastWriteTime: 22-08-2026 06:30:57

The LastWriteTime reflects the script correction that occurred before successful execution.

---

# SHA-256

An earlier hash was recorded before the script was corrected:

    1FC341260A636B2D40595AEF4FF2762269B363C97A8A22E5B213DC4F103F011B

After the script was corrected, the hash became:

    2AE98EF3347C83F2CEB37DE23A1C4E37DE29038285110AAFE885A49DA2C97850

The hash changed because the contents of the script were edited.

This is an important forensic observation and should not be interpreted as malicious file modification.

---

# Process Investigation

A process enumeration command was used:

    Get-Process |
    Sort-Object StartTime -Descending |
    Select-Object -First 20 ProcessName, Id, StartTime, Path

The command encountered an access-denied error while evaluating the `StartTime` property of at least one process.

Despite that error, process information was returned for multiple processes.

Relevant entries included PowerShell processes and normal Windows applications.

This demonstrated that process enumeration itself can encounter permission-related limitations.

---

# Evidence Correlation

The investigation followed this model:

    PowerShell Process
          |
          v
    Sysmon Event ID 1
          |
          v
    PowerShell Event ID 4104
          |
          v
    Controlled In-Memory Processing
          |
          +---- Filesystem Review
          |
          +---- Sysmon Event ID 3
          |
          +---- Sysmon Event ID 7 Check
          |
          +---- Sysmon Event ID 10 Check
          |
          +---- Security Event ID 4688 Check
          |
          v
    Final Assessment

---

# Key Findings

- The initial script contained a variable initialization error.
- The first execution attempt generated `ArgumentNullException` errors.
- The script was corrected before the successful execution.
- The corrected script executed successfully using `-ExecutionPolicy Bypass`.
- Sysmon Event ID 1 was observed.
- PowerShell Event ID 4104 was observed at multiple timestamps.
- Sysmon Event ID 3 was available.
- Sysmon Event ID 7 was unavailable.
- Sysmon Event ID 10 was unavailable.
- Security Event ID 4688 was unavailable.
- No conventional executable payload was found in the checked lab or Temp locations.
- The final script hash differed from the initial hash because the script was corrected.
- The controlled scenario did not establish confirmed malicious in-memory execution.

---

# DFIR Interpretation

The investigation provides evidence of PowerShell execution and controlled in-memory data processing.

However, the evidence does not establish that malicious code executed directly from memory.

A real in-memory execution case would normally require stronger evidence, potentially including:

- Memory capture
- EDR telemetry
- Process injection telemetry
- Memory-region analysis
- Process dumps
- DLL loading evidence
- Suspicious process access
- Network correlation
- Malware analysis

The absence of Sysmon Event IDs 7 and 10 significantly limits the available endpoint evidence.

---

# MITRE ATT&CK Relevance

Potentially relevant techniques include:

**T1059.001 — PowerShell**

PowerShell was used to execute the controlled script.

Memory-oriented techniques could become relevant in a real incident, but the controlled lab does not establish a specific memory-injection technique.

The ATT&CK technique should therefore be mapped conservatively based on observed evidence.

---

# Evidence Limitations

The investigation had several important limitations:

- Security Event ID 4688 was unavailable.
- Sysmon Event ID 7 was unavailable.
- Sysmon Event ID 10 was unavailable.
- The PowerShell script had to be corrected before successful execution.
- One process enumeration query encountered access-denied behavior.
- Sysmon Event ID 3 contained general network activity, but no specific malicious connection was established.
- No memory image was collected.

These limitations mean that the investigation can demonstrate suspicious-style memory processing concepts but cannot confirm malicious in-memory execution.

---

# Conclusion

This investigation demonstrated the challenge of investigating memory-oriented execution without relying solely on filesystem artifacts.

PowerShell Event ID 4104 and Sysmon Event ID 1 provided useful execution evidence. Sysmon Event ID 3 provided network telemetry. However, the absence of Sysmon Event IDs 7 and 10 and Security Event ID 4688 reduced visibility into memory-related and process-access behavior.

The controlled script performed harmless data transformation in memory and did not represent actual malware.

The final DFIR conclusion is therefore:

> The evidence confirms controlled PowerShell execution and in-memory data processing, but does not confirm malicious in-memory code execution.

---

# Skills Demonstrated

- Windows DFIR
- In-Memory Execution Investigation
- PowerShell Investigation
- PowerShell Event ID 4104
- Sysmon Event ID 1
- Sysmon Event ID 3
- Sysmon Event ID 7 Availability Analysis
- Sysmon Event ID 10 Availability Analysis
- Process Investigation
- Filesystem Investigation
- SHA-256 Hash Analysis
- Telemetry Gap Analysis
- Event Viewer Investigation
- Evidence Correlation
- DFIR Timeline Construction

---

# Disclaimer

This lab used harmless controlled PowerShell code. It did not execute shellcode, perform process injection, load malicious DLLs, or deploy malware.
