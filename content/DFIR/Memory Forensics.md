---
tags:
  - dfir
  - memory-forensics
  - volatility
date: 2025-02-25
---

Memory forensics differ from disk forensics analysis since it not only provides information about what resides on the target computer but also provides us with information about the processes or applications that were running at a particular time and detailed information on the execution flow on a system that may not be present in regular storage units or application logs.

Two main phases: Memory Acquisition and Memory Analysis.
## Memory Acquisition (Imaging) Tools:

| OS        | Tool                                                                                                                         |
| --------- | ---------------------------------------------------------------------------------------------------------------------------- |
| Windows   | [FTK imager](https://www.exterro.com/digital-forensics-software/ftk-imager), [WinPmem](https://github.com/Velocidex/WinPmem) |
| **Linux** | [LIME](https://github.com/504ensicsLabs/LiME)                                                                                |
| **macOS** | [osxpmem](https://code.google.com/archive/p/pmem/wikis/OSXPmem.wiki)                                                         |

## Memory Analysis:

Volatility3 is an open-source memory forensics framework. It can analyze Windows, Linux or macOS systems volatile memories.

Some artifacts that can be extracted from the memory dump:
- **System Information** → `windows.info`
- **Running processes** → `windows.pslist`, `windows.pstree`, `windows.psscan`
	Attackers typically disguise malwares with legit file names. To understand which processes are dangerous, check their parent process id (PPID) 
	To dump a process: `vol.py -f <memory_image> -o . windows.memmap --dump --pid <PID>`
- **Handles**  → `windows.handles`
	Handles plugin shows all objects a process has open. Files, registry keys and more.
	Usage: `vol.py -f <memory_image> windows.handles ‑‑pid <PID>`
- **Open network connections** → `windows.netscan`
	Look for unusual network connections
- **Recently accessed/open files** → `windows.filescan`
	To dump a file: `vol.py -f <memory_image> -o <output_dir> windows.dumpfiles ‑‑physaddr <offset>`
- **Attached device info** → `windows.devicetree`
- **Loaded DLLs** → `windows.dlllist`
- **Active user sessions** → `windows.getsids`, `windows.sessioninfo`
- **Command-line history** → `windows.cmdline`, `windows.consoles`
- **Malware detection** → `windows.malfind`, `windows.ssdt`
	Malfind checks for suspicious memory regions such as memory regions with non-standard memory protections (RWX, RX without a mapped file), process hollowing and DLL injection.

**Tip:** Even if the partition being fully encrypted, once it is mounted, any files accessed on the volume become cached by the [Windows Cache Manager](http://volatility-labs.blogspot.com/2012/10/movp-44-cache-rules-everything-around.html) per normal -- which means the `dumpfiles` plugin can help you recover them in plain text.



## Resources
https://volatility3.readthedocs.io/en/stable/index.html
https://blog.onfvp.com/post/volatility-cheatsheet/