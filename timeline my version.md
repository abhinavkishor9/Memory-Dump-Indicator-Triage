# Investigation Timeline 
| Step | Activity | Result |
|---|---|---|
| 1 | Created the Lab 26 workspace | Directories created successfully |
| 2 | Verified Python and analysis tools | All required tools available |
| 3 | Created the synthetic memory-like sample | Binary artefact generated successfully |
| 4 | Identified the sample using `file` | Classified as generic data |
| 5 | Extracted printable strings | All predefined strings recovered |
| 6 | Searched for process indicators | `powershell.exe` and `notepad.exe` identified |
| 7 | Searched for command-line indicators | `-NoProfile` and `-EncodedCommand` identified |
| 8 | Searched for network indicators | URL-like string and IP address identified |
| 9 | Searched for persistence references | `CurrentVersion\Run` identified |
| 10 | Calculated SHA-256 | Sample hash recorded |
| 11 | Inspected hexadecimal representation | Binary structure reviewed |
| 12 | Generated consolidated triage report | Findings documented |

## Key Evidence

| Evidence | Location |
|---|---|
| Synthetic binary sample | `samples/synthetic_memory.bin` |
| Sample generator | `tools/create_memory_sample.py` |
| Extracted strings | `evidence/extracted-strings.txt` |
| SHA-256 hash | `evidence/sample-hash.txt` |
| Hexadecimal dump | `evidence/memory-hex-dump.txt` |
| Consolidated report | `evidence/indicator-triage-report.txt` |

