# AVLeak
### Turning Antivirus Emulators Inside Out

## Abstract

AVLeak is a tool for fingerprinting consumer antivirus emulators through automated black box testing. AVLeak can be used to extract information from AV emulators that may be used to detect their presence and evade detection, including environmental artifacts, OS API behavioral inconsistencies, emulation of network connectivity, timing inconsistencies, process introspection, and CPU emulator “red pills”.

These artifacts of emulation may be discovered through painstaking, time consuming binary reverse engineering, or through black box testing with malware that conditionally chooses to unpack or not unpack based on its emulated environment. The current state of the art in black box AV emulator fingerprinting is a lot like handwriting SQL injection queries with a web browser, while AVLeak is like using using SQLmap.

## Content

### Background

Static signature-based malware detection is useless in the face of close to one million new samples per day. [^1] An essential response to this onslaught, automated dynamic analysis systems run unknown binaries in an isolate virtualized environment and observe their behavior, looking for known malware signatures or heuristically malicious behavior.

Malware can detect virtualized automated dynamic analysis systems and behave benignly. While attacks on high end virtualization systems such as VMWare, VirtualBox, QEMU, etc, have been extensively documented [^2], the emulators used in consumer antivirus products have not been studied as well. Consumer AVs have a number of limitations that make them prone to detection attacks: they must run without too much overhead on consumer PCs, they cannot redistribute full copies of Windows, and the AV industry is generally known for poor software engineering practices. Rather than virtualizing a full operating system as VMWare et al do, consumer AVs combine an x86 CPU emulator with emulation of usermode WinAPI functions, and a fake PC environment.

Reversing AV emulators is an incredibly difficult task, requiring expert knowledge of reverse engineering, x86 instruction set architecture, Windows internals, and malware behavior.

In response to the difficulty of REing AV emulators, researchers have turned to black box analysis, creating programs which take some reading within the emulator and decide to drop or not drop malware in response.[^3] [^4] [^5] When the AV emulator returns a decision of malware/not malware after analyzing the test program, the decision may be used to exfiltrate one bit of information from inside the emulator.

```
if ( WinAPI_Function_X() != EXPECTED_RETURN_VALUE ){
		drop_malware();	//AV detects malware -> WinAPI_Function_X is not emulated correctly
	}
	else{
		exit();	//No malware detected -> WinAPI_Function_X returned the correct value
	}
```

### AVLeak
AVLeak's innovation is to expand the one-bit channel of prior work to an arbitrarily large channel by creating a mapping of specific malware samples onto bytes.

Create a mapping of bytes to packed malware samples.

    0x00 -> ILOVEYOU
    0x01 -> Code Red
    0x02 -> Zeus
    ...
    0xFF -> Brain
    
The samples are packed and dropped at runtime after taking a reading inside the emulator:
```
GetUserName()     // returns "AVEMU"
DropMalware(0x41) // A
DropMalware(0x56) // V
...				  // E, M
DropMalware(0x55) // U
```
When the AV comes back with a list of malware detections, map the dropped malware names back across the table, extracting the fact that inside the emulator `GetUserName` returns "AVEMU".

	Scan results for binary "test.exe":
	Dropped: "Slammer" -> "A" inside the emulator
	Dropped: "Morris"  -> "V" inside the emulator
	...				   -> "E" / "M"
	Dropped: "Storm"   -> "U" inside the emulator

AVLeak is a tool built around this technique. Test cases to be run inside AV emulators are written in C, and Python scripts handle compilation, AV scanning, and result correlation. C test cases are simple to write, letting reverse engineers simply call a `printf`-like function to exfiltrate data from the emulators. AVLeak presents a Python API allowing for the creation of scriptable testing routines and integration with other applications, such as fuzzers, reversing tools, web frameworks, etc. New AVs can be integrated into AVLeak in a matter of hours, most of which is run time for automated scripts, and C test cases and Python testing scripts are write-once-run-against-any-AV.

### Results

AVLeak was used on the emulators in four commercial AV products: Kapersky, AVG, Bitdefender (licensed out to 20+ other AVs), and VBA. Artifacts of emulation were analyzed across six categories. Readers seeking more in-depth discussion of these artifacts are referred to the video of the talk, which also includes a live demo of AVLeak.

* Environmental artifacts
    * Hardcoded program names, computer names, product IDs, MACs, etc
    * Fake processes "running" and open windows
    * Recursively dumped file systems and registries within AVs, and found strange artifacts within them, including many which seem to be "easter eggs" left in by programmers, such as the `C:\\BATMAN` directory found in Bitdefender
* OS API inconsistency
    * Found functions that failed erroneously, returned unique incorrect values, caused analysis to stop.
    * File system permissions were not properly enforced.
    * Interaction with GUI components was not well emulated.
    * Fake library code
        * Found fake library code in all four AVs by dumping code at pointers returned by calls to `GetProcAddress`. Dumped every single exported function for multiple essential system DLLs and found common patterns; the AVs often used faulting or obscure instructions in these libraries as means of triggering emulation of specific functions (when the instruction is seen by the CPU emulator, it calls the appropriate WinAPI emulation function).
* Network emulation
    * Kaspersky, AVG, and Bitdefender emulated network connectivity and responsed to HTTP requests with generic `200 OK` status codes, and provided PE executables for download, presumably as means of catching malware which downloads and executes in stages. Extracting and analyzing these files provided more artifacts with which the emulators may be detected.
* Timing
    * Using a variety of timing APIs I measured timings within the emulators, and found that sophisticated attacks using subtle timing skews as documented in academic literature were unnecessary. 
    * All of the AVs used static values for the starting point of date and time, providing yet another environmental artifact.
    * Times were not consistent across various APIs for time measurement
    * In AVs that tried to do timing emulation properly "hyperreality" was problematic (ie: a call to `Sleep(1)` to sleep for 1 millisecond will have some multi-millisecond overhead, but this was not accounted for - measuring time before and after the call showed a difference of exactly 1 millisecond only)
* Process introspection
    * Process introspection artifacts include heap metadata and addresses, contents of uninitialized memory, runtime data structures (PEB, TEB, etc), process size, data left on stack after function calls, DLLs in memory after `LoadLibrary`. 
* CPU "red pills" [^6]
    * CPU red pills are instructions which behave differently on a CPU emulator than on a real CPU.
    * Red pill testing requires a lot of testing infrastructure, so support for it in AVLeak is currently limited. 
    * Handcrafted test cases for unique odd instructions like `cpuid` or `rdtscp` have lead to the discovery of CPU emulator inconsistencies.

### Malware Discovery

Googling for unique strings found within the AV emulators under test lead me to find hundreds of automated malware analysis reports publicly accessible online where binaries contained strings related to specific AV emulators. 

More notable than simple packed malware samples which check for emulation before executing, I also found that "EvilBunny", a highly advanced piece of "APT" malware [^7], exhibits anti-Bitdefender behavior, checking if is running as "TESTAPP", the name given to binaries under analysis in Bitdefender's emulator.

### Future Work

Future directions for work include integrating more AVs, building more test cases, and leveraging AVLeak for vulnerability research against AV emulators. Vulnerability research is particularly promising in light of research from Joxean Koret of COSEINC [^8] and Tavis Ormandy of Google Project Zero [^9].

### Conclusion
AVLeak has taken a problem which was previously extremely time consuming, required expensive RE tools and expert knowledge of multiple disciplines, and made it very fast and tractable to anyone who can write C code and use the Windows API.



## Thank You

* Jeremy Blackthorne for sponsoring me in conducting this research and giving me the original idea.[^10]
* Dr. Bülent Yener for his help as my advisor in this research.
* Patrick Biernat & Andrew Fasano for their help in building a prototype implementation of AVLeak.

## Further Reading
Readers interested in learning more about antivirus internals are referred to Joxean Koret and Elias Bachaalany's "The Antivirus Hacker's Handbook".

Additional resources which may be of interest:

Chen, Xu, Jon Andersen, Z. Morley Mao, Michael Bailey, and Jose Nazario. 2008. “Towards an understanding of anti-virtualization and anti-debugging behavior in modern malware.” In 2008 IEEE International Conference on Dependable Systems and Networks With FTCS and DCC (DSN), 177–186. IEEE. http://mdbailey.ece.illinois.edu/publications/dsn08_final.pdf

Jung, Paul. 2014. “Bypassing Sanboxes for Fun!”. BotConf: The Botnet Fighting Conference 2014, Nancy, France. https://www.botconf.eu/wp-content/uploads/2014/12/2014-2.7-Bypassing-Sandboxes-for-Fun.pdf

Kaspersky, Eugene. 2012. “Emulation: A Headache To Develop – But Oh-So Worth It”. https://eugene.kaspersky.com/2012/03/07/emulation-a-headache-to-develop-but-oh-so-worth-it/

Kaspersky, Eugene. 2013. “Emulate To Exterminate”. https://eugene.kaspersky.com/2013/07/02/emulate-to-exterminate/.

Rolles, Rolf. 2013. “Detecting an emulator using the windows api.”  http://reverseengineering.stackexchange.com/a/2806

Rolles, Rolf. 2015. "Memory Lane: Hacking Renovo". Möbius Strip Reverse Engineering. http://www.msreverseengineering.com/blog/2015/7/16/hacking-renovo

Singh, Abhishek, and Zheng Bu. 2013. “Hot Knives Through Butter: Evading File-based Sandboxes.” FireEye. https://www.fireeye.com/content/dam/fireeye-www/global/en/current-threats/pdfs/fireeye-hot-knives-through-butter.pdf

Wicherski, Georg. 2010. “dirtbox, an x86/Windows Emulator”.  Black Hat 2010, Las Vegas, Nevada. https://www.youtube.com/watch?v=P-mSjAnhbw8

Yoshioka, Katsunari, Yoshihiko Hosobuchi, Tatsunori Orii, and Tsutomu Matsumoto. 2011. “Your Sandbox is Blinded: Impact of Decoy Injection to Public Malware Analysis Systems.” Journal of Information Processing 19:153–168. https://ipsj.ixsq.nii.ac.jp/ej/?action=repository_uri&item_id=73611&file_id=1&file_no=1

## References

[^1] Haley, Kevin. 2015. "2015 Internet Security Threat Report: Attackers are bigger, bolder, and faster". Symantec Official Blog. http://www.symantec.com/connect/blogs/2015-internet-security-threat-report-attackers-are-bigger-bolder-and-faster

[^2] Ferrie, Peter. 2007. "Attacks on More Virtual Machine Emulators". Symantec Advanced Threat Research. http://pferrie.tripod.com/papers/attacks2.pdf

[^3] Swinnen, Arne, and Alaeddine Mesbahi. 2014. “One packer to rule them all: Empiri- cal identification, comparison and circumvention of current Antivirus detection techniques.” Black Hat 2014, Las Vegas, Nevada. https://www.blackhat.com/docs/us-14/materials/us-14-Mesbahi-One-Packer-To-Rule-Them-All-WP.pdf https://www.youtube.com/watch?v=gtLMXxZErWE

[^4] Adams, Kyle. 2014. “Evading code emulation: Writing ridiculously obvious malware that bypasses AV.” BSides Las Vegas 2014, Las Vegas, Nevada. https://www.youtube.com/watch?v=tkOtBkvS9xY

[^5] Nasi, Emeric. 2014. "Bypass Antivirus Dynamic Analysis: Limitations of the AV model and how to exploit them". Self-published. http://packetstorm.foofus.com/papers/virus/BypassAVDynamics.pdf

[^6] Paleari, Roberto, Lorenzo Martignoni, Giampaolo Fresi Roglia, and Danilo Bruschi. 2009. “A fistful of red-pills: How to automatically generate procedures to detect CPU emulators.” In WOOT’09 Proceedings of the 3rd USENIX conference on Offensive technologies.
http://static.usenix.org/event/woot09/tech/full_papers/paleari.pdf

[^7]Marschalek, Marion. 2014. "EvilBunny: Malware Instrumented By Lua”.  http://www.cyphort.com/evilbunny-malware-instrumented-lua/.

[^8] Koret, Joxean. 2014. “Breaking Antivirus Software.” SYSCAN 2014, Singapore, Singapore. http://mincore.c9x.org/breaking_av_software.pdf

[^9] Ormandy, Tavis. 2015. "Analysis and Exploitation of an ESET Vulnerability". Google Project Zero. http://googleprojectzero.blogspot.com/2015/06/analysis-and-exploitation-of-eset.html

[^10] Blackthorne, Jeremy, Bülent Yener. 2015. "Reverse Engineering Anti-Virus Emulators through Black-box Analysis". Computer Science Department, Rensselaer Polytechnic Institute. Technical report. http://cs.rpi.edu/research/pdf/15-02.pdf

#### Metadata

Tags: antivirus, malware, emulation

**Primary Author Name**: Alexei Bulazel

**Primary Author Affiliation**: Rensselaer Polytechnic Institute

