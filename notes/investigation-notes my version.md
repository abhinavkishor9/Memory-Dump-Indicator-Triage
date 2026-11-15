# Investigation Notes 
## 1. Investigation Overview

The objective of this investigation was to identify selected indicators inside a controlled synthetic memory-like binary artefact.

The analysis focused on extracting readable strings and searching for process names, command-line options, network-related values, and persistence-related references.

The sample was not executed or modified during the investigation.

## 2. Sample Identification

| Attribute | Value |
|---|---|
| Filename | `synthetic_memory.bin` |
| Location | `samples/synthetic_memory.bin` |
| File size | 306 bytes |
| File identification | Generic data |
| SHA-256 | `ee84e15030c509da3bfb5c374fd660c2e7532355d0cda38bd7f83c469adc81f3` |
| Sample type | Synthetic binary artefact |

The sample was generated using a Python script. Individual data regions were separated using null bytes to provide a simple binary structure for string extraction.

## 3. String Extraction

The following command was used:

```bash
strings -a samples/synthetic_memory.bin
```

The extracted strings included:

```text
LAB26_SYNTHETIC_MEMORY_IMAGE
PROCESS: powershell.exe
COMMAND_LINE: powershell.exe -NoProfile -EncodedCommand VABlAHMAdAA=
NETWORK_URL: hxxp://update.example.invalid/payload
REMOTE_IP: 192.0.2.10
REGISTRY_REFERENCE: HKCU\Software\Microsoft\Windows\CurrentVersion\Run
PROCESS: notepad.exe
MEMORY_ANALYSIS_END
```

The output confirmed that the predefined strings were present in the synthetic artefact.

## 4. Process Indicator Analysis

The following search was performed:

```bash
grep -Ein 'powershell|cmd\.exe|rundll32|regsvr32|wscript|cscript|notepad' \
evidence/extracted-strings.txt
```

Observed results:

```text
2:PROCESS: powershell.exe
3:COMMAND_LINE: powershell.exe -NoProfile -EncodedCommand VABlAHMAdAA=
7:PROCESS: notepad.exe
```

### Interpretation

- `powershell.exe` was identified in the extracted strings.
- `notepad.exe` was included as a benign comparison.
- The PowerShell reference was also present in the command-line string.

These findings establish the presence of process-name strings only. They do not establish that the processes were running.

## 5. Command-Line Indicator Analysis

The following command was used:

```bash
grep -Ein 'EncodedCommand|ExecutionPolicy|NoProfile|WindowStyle|DownloadString|FromBase64String' \
evidence/extracted-strings.txt
```

Observed result:

```text
3:COMMAND_LINE: powershell.exe -NoProfile -EncodedCommand VABlAHMAdAA=
```

### Interpretation

The command-line string contained:

- `-NoProfile`
- `-EncodedCommand`

The encoded-command option can be used to pass encoded PowerShell instructions. It is relevant during an investigation because it may conceal the readable form of a command.

However, the presence of this option does not establish malicious execution. The encoded content was not decoded or independently validated as executed during this exercise.

## 6. Network Indicator Analysis

### URL Search

Command:

```bash
grep -Ein 'https?://|hxxps?://' evidence/extracted-strings.txt
```

Observed result:

```text
4:NETWORK_URL: hxxp://update.example.invalid/payload
```

### IP Address Search

Command:

```bash
grep -Eo '([0-9]{1,3}\.){3}[0-9]{1,3}' \
evidence/extracted-strings.txt
```

Observed result:

```text
192.0.2.10
```

### Interpretation

The artefact contained a URL-like string and an IP address.

The `.invalid` domain is reserved for invalid example names. The IP address `192.0.2.10` is from a documentation range.

These values were deliberately selected for controlled testing. No DNS lookup, connection attempt, or network communication was performed.

The findings establish only that the strings were present in the sample.

## 7. Persistence Indicator Analysis

Command:

```bash
grep -Ein 'CurrentVersion\\Run|CurrentVersion\\RunOnce|ScheduledTask|Services\\' \
evidence/extracted-strings.txt
```

Observed result:

```text
6:REGISTRY_REFERENCE: HKCU\Software\Microsoft\Windows\CurrentVersion\Run
```

### Interpretation

The sample contained a reference to the Windows `CurrentVersion\Run` Registry location.

This location is commonly examined during persistence investigations because applications can configure entries to launch during user logon.

However, the synthetic artefact contains only a textual reference. It does not establish that the Registry key or a value was modified.

## 8. Hexadecimal Inspection

The following command was used:

```bash
xxd samples/synthetic_memory.bin | head -n 20
```

The hexadecimal output showed the underlying byte representation of the sample.

The null bytes separating the data regions were visible in the output. The extracted strings could therefore be correlated with their corresponding byte sequences.

The complete hexadecimal output was saved to:

```text
evidence/memory-hex-dump.txt
```

## 9. Evidence Integrity

The SHA-256 hash was calculated using:

```bash
sha256sum samples/synthetic_memory.bin
```

Observed hash:

```text
ee84e15030c509da3bfb5c374fd660c2e7532355d0cda38bd7f83c469adc81f3
```

The hash was recorded in `evidence/sample-hash.txt`.

## 10. Findings Summary

| Finding | Evidence | Assessment |
|---|---|---|
| PowerShell process-name string | Extracted strings | Confirmed string presence |
| Notepad process-name string | Extracted strings | Confirmed string presence |
| Encoded PowerShell option | Command-line string | Confirmed string presence |
| URL-like value | Extracted strings | Confirmed string presence |
| Documentation IP | Extracted strings | Confirmed string presence |
| Registry persistence reference | Extracted strings | Confirmed string presence |

## 11. Confirmed, Plausible, and Unknown

### Confirmed

- The synthetic artefact was successfully created.
- Its size was 306 bytes.
- Its SHA-256 hash was recorded.
- Readable strings were extracted.
- The predefined process, command-line, network, and Registry-related strings were identified.
- The hexadecimal representation was inspected.

### Plausible Investigation Leads

In a genuine memory investigation, the following observations could warrant additional examination:

- Encoded PowerShell command-line arguments.
- Unusual process execution context.
- Network indicators associated with suspicious processes.
- Registry references associated with persistence.

These are investigation leads, not confirmed malicious activities in this exercise.

### Unknown / Not Established

- Whether PowerShell was actually executed.
- Whether a process established a network connection.
- Whether the URL was accessed.
- Whether the Registry path was modified.
- Whether any code injection or malicious activity occurred.
- Whether the artefact represents any actual system memory state.

