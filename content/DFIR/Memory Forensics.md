---
tags:
  - dfir
  - memory-forensics
  - volatility
---

Memory forensics differs from disk forensics analysis since it not only provides information about what resides on the target computer but also provides us with information about the processes or applications that were running at a particular time and detailed information on the execution flow on a system that may not be present in regular storage units or application logs.

Two main phases: Memory Acquisition and Memory Analysis.
#### Memory Acquisition (Imaging) Tools:

| OS        | Tool                                                                                                                         |
| --------- | ---------------------------------------------------------------------------------------------------------------------------- |
| Windows   | [FTK imager](https://www.exterro.com/digital-forensics-software/ftk-imager), [WinPmem](https://github.com/Velocidex/WinPmem) |
| **Linux** | [LIME](https://github.com/504ensicsLabs/LiME)                                                                                |
| **macOS** | [osxpmem](https://code.google.com/archive/p/pmem/wikis/OSXPmem.wiki)                                                         |

#### Memory Analysis:

Volatility3 is an open-source memory forensics framework. It can analyze Windows, Linux or macOS systems volatile memories.

Some artifacts that can be extracted from the memory dump:
- **Running processes** → `windows.pslist`, `windows.pstree`, `windows.psscan`
	Attackers typically disguise malwares with legit file names. To understand which processes are dangerous, check their parent process id (PPID) 
- **Open network connections** → `windows.netscan`
	Look for unusual network connections
- **Recently accessed/open files** → `windows.filescan`
	To dump a process: `vol -f <memory_image> -o . windows.memmap --dump --pid <PID>`
- **Loaded DLLs** → `windows.dlllist`
- **Active user sessions** → `windows.getsids`, `windows.sessioninfo`
- **Clipboard contents** → `windows.clipboard`
- **Command-line history** → `windows.cmdline`, `windows.cmdline`, `windows.consoles`
- **Malware detection** → `windows.malfind`, `windows.ssdt`
	Malfind checks for suspicious memory regions such as memory regions with non-standard memory protections (RWX, RX without a mapped file), process hollowing and DLL injection.