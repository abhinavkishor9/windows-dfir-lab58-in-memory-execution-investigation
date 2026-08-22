# Troubleshooting Notes — Lab 58 In-Memory Execution Investigation

## 1. Initial Script Produced GetBytes() Error

### Problem

The first version of the script attempted:

    $Data=[System.Text.Encoding]::UTF8.GetBytes($Data)

The `$Data` variable had not yet been initialized.

### Result

PowerShell returned:

    Array cannot be null.
    Parameter name: chars

### Cause

`GetBytes()` received a null value.

---

## 2. Second Error From GetString()

### Problem

The script then attempted:

    $Decoded=[System.Text.Encoding]::UTF8.GetString($Bytes)

Because `$Bytes` was never successfully created, the second operation also failed.

### Result

PowerShell returned:

    Array cannot be null.
    Parameter name: bytes

### Resolution

The script was reordered:

    $Data
       ↓
    $Bytes
       ↓
    $Decoded

---

## 3. Corrected Script

The final script used:

    $Data = "LAB58-CONTROLLED_DATA"

    $Bytes = [System.Text.Encoding]::UTF8.GetBytes($Data)

    $Decoded = [System.Text.Encoding]::UTF8.GetString($Bytes)

This executed successfully.

---

## 4. Execution Policy

The controlled script required:

    -ExecutionPolicy Bypass

The final command was:

    powershell.exe -ExecutionPolicy Bypass -File "C:\InMemoryExecutionLab\memory-test.ps1"

The system-wide Execution Policy was not permanently changed.

### DFIR Note

The use of `-ExecutionPolicy Bypass` can itself become useful evidence when command-line telemetry is available.

However, the parameter alone does not prove malicious activity.

---

## 5. Sysmon Event ID 7 Was Unavailable

### Problem

The following query returned no events:

    Get-WinEvent -FilterHashTable @{
        LogName="Microsoft-Windows-Sysmon/Operational"
        Id=7
    } -MaxEvents 20

### Interpretation

Image Load telemetry was unavailable on this endpoint.

### Resolution

The investigation continued using:

- Sysmon Event ID 1
- Sysmon Event ID 3
- PowerShell Event ID 4104
- Filesystem evidence

---

## 6. Sysmon Event ID 10 Was Unavailable

### Problem

Event ID 10 returned no events.

### Interpretation

Process Access telemetry was unavailable.

### DFIR Impact

This reduced visibility into process-memory access behavior.

### Important

We did not attempt to generate process injection merely to produce Event ID 10.

---

## 7. Security Event ID 4688 Was Unavailable

### Problem

No relevant Security Event ID 4688 was available.

### Resolution

The process investigation relied on:

    Sysmon Event ID 1
    +
    PowerShell Event ID 4104

### DFIR Note

Missing 4688 does not prove that PowerShell was not executed.

---

## 8. PowerShell Event ID 4104 Was Available

### Observation

Three relevant events were identified:

    22-08-2026 06:24:36
    22-08-2026 06:26:23
    22-08-2026 06:32:08

### Significance

These events provided script-level evidence surrounding the controlled activity.

---

## 9. Sysmon Event ID 3 Contained Many Events

### Observation

Sysmon Event ID 3 returned numerous network connection events.

### Interpretation

This demonstrated that network telemetry was operational.

However, general network activity does not prove that the controlled in-memory processing produced malicious network traffic.

### DFIR Principle

Network telemetry must be correlated with:

- Process
- Timestamp
- Destination
- Port
- Application

---

## 10. Process Enumeration Encountered Access Denied

### Problem

This query:

    Get-Process |
    Sort-Object StartTime -Descending |
    Select-Object -First 20 ProcessName, Id, StartTime, Path

returned an error:

    Exception getting "StartTime": "Access is denied"

### Explanation

Some Windows processes restrict access to their metadata.

### Resolution

The command still returned information for many processes.

A more robust approach is to query specific processes rather than relying on every process exposing `StartTime`.

---

## 11. Script Hash Changed

### Observation

Initial hash:

    1FC341260A636B2D40595AEF4FF2762269B363C97A8A22E5B213DC4F103F011B

Final hash:

    2AE98EF3347C83F2CEB37DE23A1C4E37DE29038285110AAFE885A49DA2C97850

### Explanation

The script contents were modified to correct the variable-order error.

Therefore, the hash changed legitimately.

### DFIR Lesson

Hash changes must always be interpreted in context.

---

## 12. No Conventional Payload Found

### Observation

The lab directory contained:

    memory-test.ps1

No conventional `.exe` or other payload was found in the checked lab directory.

The checked Temp `.exe` search also returned no results.

### Interpretation

This is consistent with the controlled nature of the lab.

It does not by itself prove memory-only execution.

---

# Troubleshooting Summary

| Issue | Resolution |
|---|---|
| `$Data` was null | Initialized `$Data` before `GetBytes()` |
| `$Bytes` was null | Corrected variable order |
| Script execution blocked | Used process-level `-ExecutionPolicy Bypass` |
| Event ID 7 unavailable | Documented telemetry gap |
| Event ID 10 unavailable | Documented telemetry gap |
| Event ID 4688 unavailable | Relied on Sysmon 1 and 4104 |
| Process enumeration hit access denied | Treated as permission limitation |
| Script SHA256 changed | Documented legitimate script correction |
| No conventional payload found | Recorded filesystem finding |
| Sysmon 3 contained general traffic | Correlated timestamps and processes |

---

# Final Troubleshooting Lesson

The key troubleshooting lesson from Lab 58 is that the investigation must adapt to the telemetry and execution conditions actually present on the endpoint.

The corrected script generated useful PowerShell and process telemetry, but missing Event IDs 7 and 10 and Security Event ID 4688 limited the ability to directly investigate memory-specific behavior.

Therefore, the correct DFIR approach is:

    Validate the environment
          ↓
    Fix legitimate lab errors
          ↓
    Collect available telemetry
          ↓
    Document missing telemetry
          ↓
    Avoid unsupported conclusions

This is especially important for in-memory execution investigations because the absence of an obvious payload file does not automatically prove that code executed from memory.
