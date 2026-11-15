# Memory-Dump-Indicator-Triage
## Overview
### 1. What Is a Memory Dump?

A memory dump is a snapshot of data stored in a computer's Random Access Memory (RAM) at a particular point in time.

It can preserve information about system activity that may not be available through disk-based investigation alone.

Memory analysis is an important part of Digital Forensics and Incident Response (DFIR), particularly when investigating suspicious process execution, fileless techniques, and activity that may leave limited disk evidence.

### 2. Why Is Memory Forensics Important?

RAM can contain information about:

- Running processes and process relationships.
- Command-line arguments.
- Loaded modules and DLLs.
- Network connections and sockets.
- Open handles.
- Memory regions and injected code.
- Credentials and other sensitive information.
- Execution-related artefacts.

Investigators can use this information to reconstruct aspects of system activity and identify potential evidence of compromise.

### 3. What Are Indicators in Memory?

Indicators are artefacts or values that may help an investigator identify suspicious activity.

Examples include:

| Indicator Category | Example |
|---|---|
| Process name | `powershell.exe` |
| Command-line option | `-EncodedCommand` |
| Network address | `192.0.2.10` |
| URL | `hxxp://example.invalid/resource` |
| Registry reference | `CurrentVersion\Run` |
| Executable path | `C:\Users\Public\...` |

These indicators must be interpreted in context. Their presence alone does not establish malicious behaviour.

### 4. String Extraction

Memory dumps and other binary files can contain readable text alongside non-printable bytes.

The Linux `strings` utility extracts sequences of printable characters from binary data.

Example:

```bash
strings -a memory.bin
```

The `-a` option scans the complete file.

String extraction is useful for initial triage because it can reveal process names, URLs, command-line arguments, Registry paths, and other readable artefacts.

However, it does not interpret the underlying memory structures or establish whether an identified process was active.

### 5. Searching with grep

The `grep` utility searches text for specified patterns.

Example:

```bash
grep -Ein 'powershell|cmd\.exe|rundll32' extracted-strings.txt
```

Useful options include:

- `-E`: Extended regular expressions.
- `-i`: Case-insensitive matching.
- `-n`: Display line numbers.

Investigators can use targeted searches to categorize extracted strings and reduce the amount of output that requires manual review.

### 6. Common Investigation Indicators

#### A. Process Indicators

Examples:

- `powershell.exe`
- `cmd.exe`
- `rundll32.exe`
- `regsvr32.exe`
- `wscript.exe`

These are legitimate Windows executables. Their presence should be evaluated using process context, parent-child relationships, execution paths, command lines, and other evidence.

#### B. Command-Line Indicators

Examples:

- `-EncodedCommand`
- `-NoProfile`
- `-ExecutionPolicy`
- `-WindowStyle`

Some command-line options can be relevant during an investigation, especially when combined with unusual process behaviour or suspicious arguments.

An encoded command is not automatically malicious.

#### C. Network Indicators

Examples:

- IP addresses.
- Domain names.
- URLs.
- Socket-related information.

A string containing an IP address or URL does not prove that a connection was established.

Network findings should be correlated with structured connection information, timestamps, process context, and network telemetry.

#### D. Persistence Indicators

Examples:

- `HKCU\Software\Microsoft\Windows\CurrentVersion\Run`
- Scheduled task references.
- Windows service references.
- Startup folder paths.

These references can help identify potential persistence mechanisms.

A path found as a string does not prove that a corresponding Registry value, task, service, or file exists.

### 7. Hexadecimal Inspection

Binary data can be examined using hexadecimal tools such as `xxd`.

Example:

```bash
xxd memory.bin
```

Hexadecimal inspection displays the underlying bytes alongside their readable character representation.

It helps investigators understand how extracted strings are represented inside a binary file and inspect non-printable data.

### 8. File Integrity with SHA-256

SHA-256 calculates a cryptographic hash of a file.

Example:

```bash
sha256sum memory.bin
```

A hash provides a way to identify and verify the sample being analysed.

For forensic workflows, recording the hash helps maintain evidence integrity and supports reproducibility.

### 9. Structured Memory Analysis

String extraction is only an initial triage technique.

A genuine memory investigation requires tools that understand the operating system's memory structures.

Volatility 3 is a memory-forensics framework that supports structured analysis of compatible memory images.

Examples of Windows plugins include:

| Plugin | Purpose |
|---|---|
| `windows.pslist` | Enumerates process information |
| `windows.psscan` | Scans for process structures |
| `windows.cmdline` | Extracts process command lines |
| `windows.netscan` | Examines network artefacts |
| `windows.dlllist` | Lists loaded DLLs |

Plugin availability and results depend on the memory image, operating system version, symbols, and acquisition quality.

### 10. Limitations of String-Based Analysis

String extraction has important limitations:

- It cannot reconstruct all memory structures.
- It may miss encoded, compressed, or non-printable data.
- It does not establish whether a process was active.
- It does not prove that a URL or IP was contacted.
- It cannot independently confirm Registry modifications.
- It may return unrelated strings that require contextual review.

A real memory investigation should combine string extraction with structured memory analysis and other available evidence.

### 11. Key Investigation Principle

An indicator is a lead, not a verdict.

A suspicious-looking process name, command-line option, network address, or Registry reference should trigger further examination rather than an immediate malware conclusion.

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

## Lab Scenario

### Scenario

A security investigation requires an analyst to become familiar with identifying potentially relevant artefacts within system memory.

During incident response, memory captures can contain process information, command-line arguments, network-related data, and references to operating-system configuration. Some of these artefacts may help investigators understand suspicious activity that is not immediately apparent from disk-based evidence.

For this exercise, a controlled synthetic binary file was prepared to represent selected memory-like data regions. It contains predefined process names, a PowerShell command-line string, a URL-like value, an IP address, and a Windows Registry reference.

The analyst is tasked with examining the file without executing it and identifying the predefined indicators.

### Investigation Tasks

The analyst must:

1. Verify the analysis environment and required tools.
2. Identify the synthetic binary artefact and record its SHA-256 hash.
3. Extract readable strings from the file.
4. Search for process names and command-line indicators.
5. Identify URL-like values and IP address strings.
6. Locate Registry references associated with potential persistence.
7. Inspect the underlying hexadecimal representation.
8. Organize the findings into an investigation report.

### Evidence Handling

The sample must remain unchanged during analysis.

The analyst should retain the original synthetic artefact, record its hash, and save extracted strings and search results as separate evidence files.

All observations must be based on the actual command output. The analyst must not interpret a string match as proof of execution or system activity.

### Expected Investigation Approach

The investigation should begin with basic file identification and string extraction, followed by targeted searches for relevant indicators.

Each result should be documented with its evidence source and a careful interpretation.

Where a string suggests suspicious activity, the analyst should identify what additional evidence would be required to validate the observation in a genuine memory investigation.

### Scope and Limitations

This scenario uses a synthetic binary artefact rather than an actual RAM acquisition.

It does not simulate a complete Windows memory image and does not establish that any process was running, any network connection was made, or any persistence mechanism was configured.

The exercise is intended to teach initial memory-indicator triage and evidence-based interpretation.

### Expected Outcome

The analyst should successfully extract and categorize the predefined indicators, preserve the sample's integrity, and produce a concise investigation record.

The final assessment should clearly separate confirmed string presence from activity that remains unknown or unverified.

## Lab Objectives – Lab 26: Detect Indicators in Memory Dumps

### Primary Objective

Develop an understanding of how investigators perform initial indicator triage on binary memory artefacts by extracting readable strings and searching for potentially relevant system and network references.

### Technical Objectives

By completing this lab, the learner should be able to:

1. Explain the purpose of memory acquisition and memory forensics in incident response.
2. Distinguish memory-based investigation from traditional disk-based analysis.
3. Generate a controlled synthetic binary artefact for analysis.
4. Extract printable strings from binary data using the `strings` utility.
5. Search extracted content using `grep` and regular expressions.
6. Identify process-name references and command-line options.
7. Locate URL-like strings and IPv4-shaped values.
8. Recognize Windows Registry paths associated with potential persistence.
9. Inspect binary data using hexadecimal representation.
10. Calculate and document a sample's SHA-256 hash.

### Investigation Objectives

The investigation should also demonstrate the ability to:

- Separate observed indicators from assumptions.
- Explain why suspicious-looking strings require contextual validation.
- Distinguish a process-name reference from confirmed process execution.
- Distinguish a network indicator from confirmed network communication.
- Recognize that a Registry reference does not prove persistence configuration.
- Identify the limitations of basic string extraction.
- Understand when structured memory-forensics tools are required.

### Expected Learning Outcome

At the end of the exercise, the learner should be able to conduct a basic static indicator-triage workflow and document its findings accurately.

The learner should also understand that string extraction is an initial investigative technique, not a substitute for structured memory analysis or a standalone method for confirming compromise.

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

