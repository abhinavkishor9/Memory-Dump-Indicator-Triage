# Lab 26 – Detect Indicators in Memory Dumps

## Overview

This lab demonstrates basic memory-indicator triage using a controlled synthetic binary artefact. The objective was to extract readable strings and identify process names, command-line indicators, network-related strings, and persistence-related references.

The exercise introduced a practical starting point for memory forensics and demonstrated how command-line tools can help investigators identify potentially relevant artefacts within binary data.

**Important:** The file used in this lab is a synthetic memory-like sample created for testing. It is not a genuine RAM acquisition or a forensic memory dump.

## Lab Environment

| Component | Details |
|---|---|
| Operating System | Ubuntu 24.04 WSL |
| Python | 3.12.3 |
| GNU Binutils | 2.42 |
| GNU grep | 3.11 |
| Analysis Tools | strings, grep, xxd, sha256sum |
| Sample Type | Synthetic binary artefact |
| Analysis Method | Static indicator extraction |

## Investigation Scenario

A controlled binary artefact was created to simulate selected strings that an investigator might encounter during memory analysis.

The sample contained process names, a PowerShell command-line option, a URL-like string, an IP address, and a Windows Registry persistence reference.

The investigation focused on extracting and categorizing these strings without executing the sample or making assumptions about actual system activity.

## Objectives

- Understand the role of memory dumps in incident response.
- Extract readable strings from binary data.
- Identify process-related indicators.
- Search for suspicious command-line options.
- Identify URL and IP address strings.
- Detect persistence-related references.
- Record file integrity using SHA-256.
- Understand the limitations of string-based memory triage.

## Tools Used

| Tool | Purpose |
|---|---|
| `strings` | Extract printable strings from binary data |
| `grep` | Search extracted strings for selected indicators |
| `xxd` | Inspect the hexadecimal representation |
| `sha256sum` | Calculate the sample's SHA-256 hash |
| Python 3 | Generate the synthetic binary artefact |

## Sample Details

**File:** `samples/synthetic_memory.bin`

**File size:** 306 bytes

**SHA-256:**

```text
ee84e15030c509da3bfb5c374fd660c2e7532355d0cda38bd7f83c469adc81f3
```

The sample was generated using `tools/create_memory_sample.py`. Null bytes were used to separate the individual data regions.

## Key Findings

| Category | Observed Indicator |
|---|---|
| Process | `powershell.exe` |
| Process | `notepad.exe` |
| Command line | `-NoProfile -EncodedCommand` |
| URL-like string | `hxxp://update.example.invalid/payload` |
| IP address | `192.0.2.10` |
| Registry reference | `HKCU\Software\Microsoft\Windows\CurrentVersion\Run` |

### Process Indicators

The extracted strings included `powershell.exe` and `notepad.exe`.

The PowerShell reference was reviewed alongside its command-line text. The presence of these strings does not establish that either process was running.

### Command-Line Indicator

The following option was identified:

```text
-NoProfile -EncodedCommand
```

Encoded PowerShell commands can be used in legitimate administration as well as malicious activity. The command-line reference is therefore an investigation lead rather than proof of compromise.

### Network Indicators

The sample contained:

```text
hxxp://update.example.invalid/payload
192.0.2.10
```

The domain uses the reserved `.invalid` suffix, and the IP address belongs to a documentation range. These values were deliberately selected as non-operational test indicators.

No network activity was performed or established.

### Persistence Reference

The following Registry path was identified:

```text
HKCU\Software\Microsoft\Windows\CurrentVersion\Run
```

This is a commonly investigated Windows persistence location. Its presence as a string does not establish that a Registry value was created or modified.

## Evidence Collected

```text
evidence/
├── extracted-strings.txt
├── sample-hash.txt
├── memory-hex-dump.txt
└── indicator-triage-report.txt
```

## Investigation Conclusion

The synthetic binary artefact contained the predefined process, command-line, network, and persistence-related strings. These were successfully extracted and categorized using standard Linux command-line tools.

The exercise demonstrated how basic string extraction can support an initial memory-indicator triage workflow.

However, this was not a genuine memory acquisition. The findings do not establish process execution, network communication, persistence activity, or malicious behaviour.

For a real memory investigation, the extracted indicators would need to be correlated with structured memory-forensics evidence, such as process listings, command-line artefacts, loaded modules, and network connection information.

## Limitations

- The sample was synthetic and only 306 bytes in size.
- It did not represent an actual Windows RAM capture.
- No process execution or network activity was examined.
- No structured memory analysis was performed.
- String matches alone cannot establish malicious behaviour.

## Key Learning

Memory-indicator triage is an evidence-gathering activity. A string found in a binary artefact can help guide an investigation, but its meaning must be established through context and corroborating evidence.

**Investigation principle: An indicator is a lead, not a verdict.**
