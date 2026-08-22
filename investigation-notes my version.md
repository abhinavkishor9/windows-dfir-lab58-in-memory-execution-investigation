# Investigation Notes 

# Investigation Directory

The controlled directory was:

`C:\InMemoryExecutionLab`

The directory was created successfully and initially contained no payload files.

---

# Initial Filesystem State

The initial directory check produced an empty directory.

This established the starting condition:

    C:\InMemoryExecutionLab
        |
        +-- No payload files

The controlled script was created afterward.

---

# Initial Script

The first version of `memory-test.ps1` contained:

    $Data=[System.Text.Encoding]::UTF8.GetBytes($Data)
    $Decoded=[System.Text.Encoding]::UTF8.GetString($Bytes)

This contained a variable-order problem.

`$Data` was used as input to `GetBytes()` before it had been assigned a string.

`$Bytes` was then unavailable when `GetString()` was called.

---

# First Execution Attempt

The first execution produced:

    Array cannot be null.
    Parameter name: chars

and subsequently:

    Array cannot be null.
    Parameter name: bytes

The script still printed:

    Controlled in-memory processing completed

This demonstrated an important investigation point: visible output at the end of a script does not prove that every earlier operation succeeded.

---

# Script Correction

The script was corrected to initialize `$Data` first:

    $Data = "LAB58-CONTROLLED_DATA"

    $Bytes = [System.Text.Encoding]::UTF8.GetBytes($Data)

    $Decoded = [System.Text.Encoding]::UTF8.GetString($Bytes)

    Write-Output "Controlled in-memory processing completed"
    Write-Output $Decoded

The corrected file length became:

    218 bytes

---

# Script Metadata

The final script metadata was:

    Name: memory-test.ps1
    Length: 218
    CreationTime: 22-08-2026 06:21:16
    LastWriteTime: 22-08-2026 06:30:57

The LastWriteTime reflects the script correction.

---

# SHA-256 Comparison

An earlier hash was:

    1FC341260A636B2D40595AEF4FF2762269B363C97A8A22E5B213DC4F103F011B

After the script was corrected, the hash became:

    2AE98EF3347C83F2CEB37DE23A1C4E37DE29038285110AAFE885A49DA2C97850

The hash difference is expected because the file contents changed during correction.

This provides an important forensic lesson:

    File edited
        |
        v
    Contents changed
        |
        v
    SHA-256 changed

The hash change was not evidence of malicious modification.

---

# Successful Execution

The corrected script was executed using:

    powershell.exe -ExecutionPolicy Bypass -File "C:\InMemoryExecutionLab\memory-test.ps1"

Output:

    Controlled in-memory processing completed
    LAB58-CONTROLLED_DATA

The execution timestamp was recorded as:

    22 August 2026 06:33:08

---

# Sysmon Event ID 1

The following events were identified:

    22-08-2026 06:13:37
    22-08-2026 06:24:36
    22-08-2026 06:32:08

The 06:32:08 event aligns with the successful execution timeframe.

The event was used to establish PowerShell process creation evidence.

---

# PowerShell Event ID 4104

Three relevant Script Block Logging events were identified:

    22-08-2026 06:24:36
    22-08-2026 06:26:23
    22-08-2026 06:32:08

The search terms included:

    LAB58-CONTROLLED_DATA
    memory-test
    GetBytes
    GetString

This provided useful script-level evidence.

---

# Security Event ID 4688

Security Event ID 4688 was investigated but no relevant event was available.

This created a process-creation telemetry gap.

The investigation therefore relied on Sysmon Event ID 1 and PowerShell Event ID 4104 for process/script evidence.

---

# Sysmon Event ID 3

Sysmon Event ID 3 was queried and returned many network connection events.

Examples covered the period between approximately:

    06:13
    and
    06:45

The events demonstrated that network telemetry was functioning.

No specific malicious network connection associated with the controlled in-memory processing was established.

---

# Sysmon Event ID 7

Event ID 7 was queried.

No matching events were returned.

Therefore:

    Sysmon Event ID 7
        =
    Not available

This limited the ability to use Image Load telemetry as supporting evidence.

---

# Sysmon Event ID 10

Event ID 10 was queried.

No matching events were returned.

Therefore:

    Sysmon Event ID 10
        =
    Not available

This limited the ability to use process-access telemetry to investigate memory-related behavior.

---

# Process Enumeration

The following command was attempted:

    Get-Process |
    Sort-Object StartTime -Descending |
    Select-Object -First 20 ProcessName, Id, StartTime, Path

The command encountered:

    Exception getting "StartTime": "Access is denied"

The command nevertheless returned information for multiple processes.

Relevant entries included:

    powershell
    msedgewebview2
    OneDrive
    conhost
    SearchApp
    RuntimeBroker

The error demonstrated that process enumeration can be affected by permission restrictions.

---

# Payload Search

The investigation directory contained only:

    C:\InMemoryExecutionLab\memory-test.ps1

A search of the lab directory did not identify a conventional executable payload.

A check of the Temp directory for `.exe` files did not return results.

This supported the controlled nature of the exercise.

---

# Evidence Correlation

The activity was correlated as:

    PowerShell Process
          |
          v
    Sysmon Event ID 1
          |
          v
    PowerShell Event ID 4104
          |
          v
    Controlled Memory Processing
          |
          +---- Sysmon Event ID 3 Review
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

# Findings

- PowerShell execution was confirmed.
- The initial script contained a variable initialization error.
- The corrected script executed successfully.
- Script Block Logging provided evidence of the controlled activity.
- Sysmon Event ID 1 provided process creation evidence.
- Sysmon Event ID 3 was available.
- Sysmon Event ID 7 was unavailable.
- Sysmon Event ID 10 was unavailable.
- Security Event ID 4688 was unavailable.
- No conventional payload executable was found in the checked locations.
- The script hash changed because the script itself was corrected.

---

