# Data Forensics Study Guide: Systems, Disks, and Advanced Forensics

Based on **Module 3: Systems, Disks, and Media / Boot Processes** and **Module 4: Advanced Forensics**. Repeated slides have been combined. The focus is what each concept means, why it matters in an investigation, and what evidence to look for.

## 1. The big picture

Digital forensics is the disciplined process of **identifying, preserving, acquiring, examining, interpreting, and reporting digital evidence**. A finding is useful only if you can explain where it came from and show that your handling did not invalidate it.

The same core process applies to a disk, a running computer, a router, a database, a phone, or an email:

1. **Identify and document** the device, state, time, people, and scope of the investigation.
2. **Preserve what could disappear**: on a running system, capture relevant volatile information before shutdown if appropriate.
3. **Acquire a defensible copy**: image storage when possible; record the method, tool, settings, and any limitations.
4. **Verify and protect**: hash the acquired data, restrict access, and maintain a chain of custody.
5. **Examine and correlate**: inspect files, metadata, deleted space, logs, memory, and independent sources.
6. **Report** what was observed, how it was tested, and what cannot be concluded.

> **Two meanings of “image”:** A *forensic disk image* is a copy of storage for analysis. An *image file* is a picture such as a JPEG. The slides discuss both.

### Volatility and integrity

- **Volatile evidence** changes or vanishes quickly: CPU registers/cache, running processes, network connections, routing/ARP tables, RAM. Files on disk usually persist longer. RFC 3227's practical idea is **collect the most volatile relevant data first**; adapt the order to the actual incident.
- **Live acquisition** can capture RAM and current connections, but the collection itself changes the running system. Document those changes. **Dead analysis** examines an acquired image in a controlled lab and is generally easier to repeat.
- A **hash** is a compact fingerprint of data. Matching hashes support that two copies have the same content; a hash does not tell you whether the content was true or trustworthy when first acquired. **SHA-256** is a sound default for integrity checking. The slides discuss **MD5** (128 bits, 32 hexadecimal digits), but MD5 has known collisions and is unsuitable where an attacker could deliberately manipulate content. Never use MD5 for passwords or signatures.
- **Chain of custody** records who handled an item, when, why, where it was stored, and each transfer. Hashes and custody records answer different questions; keep both.

## 2. How storage is organized

### Physical disk → partition → file system → file

| Layer | Plain meaning | Forensic relevance |
| --- | --- | --- |
| **Hard disk hardware** | Magnetic platters turn under read/write heads. A **track** is a circular path; a **sector** is an addressable piece of storage. | Raw sectors may contain data outside visible files. A damaged sector can prevent a read. |
| **Partition** | A division of a device that may contain a file system. | Hidden, unused, or old partitions can hold evidence. |
| **File system** | Structures that map file names and metadata to stored content and track free space. | Lets you locate files, deleted entries, timestamps, permissions, and unused space. |
| **Cluster / allocation unit** | The group of sectors allocated to a file as a unit. | The last cluster may contain unused **file slack**. |
| **File** | Content plus metadata such as name, size, times, and attributes. | A visible file is only one part of the available evidence. |

**Example:** A 1,500-byte file on a volume with 4,096-byte clusters needs one cluster. Its remaining **2,596 bytes** are file slack. Slack *may* contain remnants; it does not always contain recoverable prior data. **Unallocated space** consists of clusters not currently assigned to a live file; deleted content may remain there. A **lost cluster** is marked allocated but not referenced by a file and can also be investigated. SSD behavior, overwriting, encryption, and TRIM can limit recovery.

**Bad sectors** are unreadable or unreliable areas, not necessarily a failed whole disk. Note acquisition errors; tools such as CHKDSK or `fsck` can change a volume, so avoid repair operations on original evidence. Traditional hard-disk interfaces in the slides include IDE/ATA and SCSI; USB is common for external devices, and Fibre Channel is used in storage networks. Magnetic tape is sequential backup media; optical discs and flash/USB media are other evidence sources.

Common logical sector sizes are **512 bytes** and **4,096 bytes (4 KiB)**. The sector size and cluster size are separate properties.

### Capacity calculations (the exam pattern)

For the simplified disk geometry used in the slides:

\[
\text{Capacity in bytes} = (\text{bytes/sector})(\text{sectors/track})(\text{tracks/surface})(\text{recording surfaces}).
\]

**Example:** `512 × 50 × 2,000 × 4 = 204,800,000 bytes`, or `204,800,000 ÷ 1,024² ≈ 195.31 MiB`. If a question asks for tracks per surface, divide capacity by the product of the **other three** terms. Doubling surfaces doubles capacity if all other factors stay fixed. The slides write “MB/GB” while using powers of 1,024; the technically precise labels are **MiB/GiB**. Modern disks use logical block addressing, so this tracks/sectors calculation is principally a teaching model.

### File systems you need to distinguish

| Family | Essence | Useful evidence or feature |
| --- | --- | --- |
| **FAT12/16/32** | Older Microsoft file systems; FAT32 has a larger addressing range than FAT16. VFAT added long-name support. | Allocation tables, directory entries, deleted files, slack. |
| **NTFS** | Windows file system with a **Master File Table (MFT)**; files are described by attributes. | `$Mft` records, file names, times, `$LogFile`, `$Bitmap`, `$BadClus`, and **alternate data streams (ADS)**. Small *resident* content may live in an MFT record; larger *nonresident* content resides in clusters elsewhere. |
| **ext2 / ext3 / ext4** | Common Linux family using **inodes** for file metadata; ext3/4 add journaling. | Inodes, directories, allocation and journal information. |
| **HFS / HFS+** | Mac file systems discussed in these historically focused slides. | A file can have a **data fork** (ordinary content) and **resource fork** (additional resources). |
| **ZFS** | Storage/file-system design using pools and virtual devices. | Storage configuration, metadata, snapshots when present. |
| **Network / optical** | SMB/CIFS and NFS access remote files; ISO 9660 and UDF describe optical media. | Evidence may be on a server or disc, rather than the local disk alone. |

**NTFS extras:** EFS encrypts files using file encryption keys protected through certificates and keys; it is distinct from whole-disk encryption. Compression reduces stored size. An **ADS** is an additional named stream attached to an NTFS file and can conceal content from a casual directory listing; its presence alone does not prove wrongdoing. The special NTFS files are themselves evidence about the volume.

## 3. Boot processes: what starts before the OS?

The reason investigators study booting is that boot code, configuration, and early drivers can affect what loads and what evidence survives.

**Legacy BIOS/MBR path:** power → firmware **POST** (basic hardware checks) → select boot device → read its **MBR** → run boot code → locate active partition/volume boot code → load the OS bootloader → kernel, drivers, and services.

- A legacy **MBR** is the first 512-byte sector: boot code, a four-entry partition table, and signature `0x55AA`. An extended partition can hold additional logical partitions. Boot-sector or MBR tampering may execute before the OS starts.
- In the **Windows XP example**, `NTLDR` reads `boot.ini`, calls `NTDETECT.COM` for hardware information, and loads `ntoskrnl.exe` and needed drivers; `hiberfil.sys` relates to resuming hibernation. These names describe the slide's XP-era path, not every Windows version.
- **Linux** commonly uses GRUB to select and load a kernel and initial RAM disk (`initrd`/`initramfs`); LILO is an older loader.
- The slides describe **older Macs**: PowerPC systems used Open Firmware; Intel Mac OS X used EFI and `boot.efi` to start the XNU kernel (drawing from Mach and BSD). The data/resource fork model and tools such as BlackBag or MacDrive likewise reflect particular older Mac workflows.

**Exam distinction:** BIOS + MBR is a **legacy** boot path. UEFI with GPT follows a different layout, so do not apply the four-primary-partition rule or XP boot filenames to every current machine. Cloning/backup software such as Carbon Copy Cloner may omit content; check its settings before treating a backup as a complete forensic image.

## 4. Picture files as evidence

| Concept | What to remember |
| --- | --- |
| **Raster vs vector** | Raster = pixels, so enlarging can reveal pixels; vector = shapes/math, so it scales. Metafiles can mix both. |
| **BMP, PNG, GIF** | BMP is commonly large and minimally compressed; PNG supports lossless compression and transparency; GIF uses a limited palette and can animate. |
| **JPEG, TIFF** | JPEG normally compresses photos **lossily**; TIFF is a flexible high-quality container used for scans and publishing. Exact TIFF compression varies. |
| **Lossy vs lossless** | Lossy discards information to shrink a file; lossless can reproduce the original bytes after decompression. Neither label alone proves authenticity. |

**Recovery logic:** search live files and then unallocated/slack space in a forensic image; identify a file by its **internal signature/header**, not only its extension; carve likely fragments; repair a damaged header only on a **working copy** and document the intervention. Fragmentation or missing data may make full recovery impossible. Look at **EXIF** for camera and timestamp clues, but metadata is editable, and viewer software can alter a working file or its access time. **Steganography** hides information inside a carrier file; a watermark is another form of embedded information, often for ownership. These are possibilities to investigate, not conclusions from the file type alone.

## 5. Operating-system forensics

### Windows: where to look

| Evidence source | Possible finding |
| --- | --- |
| Files, metadata, MFT, slack, unallocated space, partitions | Activity times, deleted or concealed content, fragments. |
| RAM, process lists, open ports, DNS cache | Running programs and current communications that may disappear on shutdown. |
| Page file, memory dumps | Some memory contents, process/driver state, crash context. |
| Registry and system state backups | Configuration and user/system traces; backups may include the registry, boot files, AD/SYSVOL, and service data. |
| Browser history, cache, cookies, downloads | Web activity; storage location and format depend on browser and version. |
| NTFS alternate data streams | Extra named content attached to a file. |

**Tool examples by job:** FTK Imager, EnCase, X-Ways, or WinHex for acquisition/inspection; Magnet RAM Capture or similar tools for RAM; `tasklist`/Process Explorer/PsList for processes; `netstat -ano` or older `fport` for ports and owning processes; RegScanner for Registry searches; `vssadmin list shadows` to discover shadow copies. Helix, WFT, BartPE, old Internet Explorer paths, and some Mac utilities in the slides are **historical tool examples**. Remember what each tool *does* rather than assuming it works on a current OS.

### Linux: preserve, image, analyze

Linux devices and paths are clues. `/dev/...` represents devices; modern disk names often look like `/dev/sda` or `/dev/nvme0n1` (the slides' `/dev/hda` is older IDE naming). `/boot` holds boot material, `/etc` configuration, `/home` user files, `/var` logs and changing data, `/tmp` temporary files, and `/proc` a live view of processes and the kernel.

1. **Identify the correct device and layout** (`lsblk`, `fdisk -l`, or TSK's `mmls`). Mistaking the source for the destination can destroy evidence.
2. **Image it** with a verified forensic tool or a raw copying utility such as `dd`/`dcfldd`, writing the result to separate storage. Record read errors and tool settings.
3. **Hash the image** (prefer SHA-256), keep a protected master, and analyze a working copy.
4. **Mount only when needed, read-only**, or inspect the image through forensic tools. The slides illustrate `mount -o ro,noexec,nodev`; some journaling file systems need extra care to prevent replay or other writes. A hardware write blocker can add protection.
5. **Search and interpret**: `file` for actual file type, `xxd` for hexadecimal bytes, `strings` for readable fragments, `grep` for patterns, `awk` for fields, and `sed` for text transformations. These commands help examine data; they do not establish authenticity on their own.

**The Sleuth Kit (TSK)** works at different layers: `mmls` maps partitions, `fsstat` describes a file system, `fls` lists names including deleted entries, `istat` examines metadata, `icat` extracts content, and `mactime` builds a timeline. `dls` addresses unallocated units and `jls`/`jcat` inspect journal information. **Autopsy** provides a case-based graphical workflow around many of these tasks: keyword searches, file signatures, hashes, timelines, notes, and reports. Choose a tool by the question you are asking, then corroborate its output.

## 6. Network and router forensics

**Network forensics** captures and examines traffic and device records to reconstruct communications. Routers forward traffic between networks at the network layer; their routing tables choose paths. **Static routes** are configured by an administrator; **dynamic routes** update according to a protocol and its metrics. RIP uses hop count; OSPF uses link state and shortest-path calculations. Relevant metrics can include cost, delay, bandwidth, load, and reliability.

**Investigation questions:** Did someone alter the route or access controls? Is a traffic spike ordinary load or an attack? Which systems communicated and when? Preserve packet captures, router configuration, logs, firewall/IDS records, and synchronized timestamps. On a router, examine the current and saved configuration, routing table, ACLs, clock, users, and software version; **do not reset it before collecting volatile state**. Current configuration may be in RAM while a saved copy persists in nonvolatile storage. Cross-check with other devices and document every command run.

**Attack ideas:** reconnaissance → scanning → access → privilege escalation → persistence → attempts to conceal activity is a useful investigative sequence, not a guarantee that every attack follows six neat stages. Routing-table poisoning changes where packets go. A **DoS** attack overwhelms or disables a service; **DDoS** uses multiple attacking systems, often controlled through other compromised machines. A **SYN flood** consumes resources with many incomplete TCP handshakes; broadcast/reflection or amplification attacks turn traffic toward a victim. The slides also name older examples such as Smurf, Ping of Death, Teardrop, Trinoo, and TFN; know their broad mechanism rather than memorizing obsolete tool ports. Look for traffic spikes, repeated connection attempts, unusual routes, and coordination among hosts.

**Incident response:** triage and identify → contain/analyze/track → recover and prevent recurrence. A report should connect claims to preserved evidence, logs, timestamps, hashes, and documented handling.

## 7. Database forensics

Database investigations ask **who changed or accessed which records, when, from where, and with what effect**. Authorized insiders can misuse privileges, so a legitimate login does not settle whether an action was legitimate.

| Approach | Timing | Main idea | Limitation / advantage |
| --- | --- | --- | --- |
| **Reactive** | After a breach. | Reconstruct actions/state from database files, logs, backups, and other evidence. | If logging was absent or an attacker changed the data, reconstruction may be incomplete. |
| **Proactive** | Before and during an incident. | Design audit trails, monitoring, integrity checks, and provenance into the system. | Produces a stronger history for later investigation if configured and preserved properly. |

**Provenance** means tracing a record or change back to its source: who or what produced it, and when/how. Correlate database audit data with application, server, and network evidence. The slides list *cloud forensics* in the module outline but do not provide a separate cloud-forensics lesson.

## 8. Mobile forensics

Phones can hold calls, messages, contacts, photos, app data, browsing, location, system logs, and sometimes deleted traces. Evidence can live in RAM, internal flash, an SD card, a **SIM/UICC**, and backups. A SIM may hold identifiers such as IMSI/ICCID and sometimes contacts or messages; it is not a complete copy of phone activity. Android and iOS have different security models, and device state can change quickly.

**Workflow:** document the scene and device state → prevent unintended network changes or remote wipe when appropriate → choose an acquisition method → examine and correlate artifacts → validate and report. Isolation may involve airplane mode or a radio-shielding container, but changing settings itself changes the device; record the choice and consider the risk of locking or losing access.

**Acquisition methods, roughly from less to more invasive:** manual viewing; logical extraction through OS interfaces; lower-level file-system/physical methods; hardware techniques such as JTAG or chip-off; backup/cloud acquisition. What each method can recover depends on the model, OS, encryption, permissions, and authorization. Rooting, jailbreaking, or exploiting a bootloader can alter evidence. Preserve originals where possible, hash what can be hashed, log tool versions and limitations, and respect the relevant legal scope.

## 9. Malware forensics

Malware includes viruses, worms, trojans, ransomware, spyware, backdoors, rootkits, botnets, and fileless techniques. Symptoms such as unknown processes, new files, odd network traffic, or disabled security tools are **leads**, not proof by themselves.

**Questions to answer:** What entered the system, how, and when? What executed or changed? Did it persist, spread, or exfiltrate data? What are the indicators of compromise (IOCs), and which actions form the attacker's tactics, techniques, and procedures (TTPs)? Preserve binaries, scripts, memory, registry/configuration traces, logs, and network evidence.

- **Static analysis:** inspect a sample without running it (hash, `strings`, file/PE headers, disassembly, rules such as YARA). Safer and quicker, but packed or obfuscated malware can hide behavior.
- **Dynamic analysis:** execute a sample in an isolated, controlled environment and watch processes, file/registry changes, and network calls. Reveals behavior but requires containment and may miss behavior triggered only under other conditions.
- Example tools in the slides: Ghidra/IDA, Procmon, Wireshark, Volatility, and sandbox platforms. **Malware forensics** adds preserved evidence, a timeline, corroboration, and reporting to the technical analysis of the program.

## 10. Email and web attacks

### Email investigations

An email has a **body/attachments** and **headers** that record identifiers, dates, and parts of its delivery route. SMTP transports mail; mail clients and server-side storage can retain additional copies. Common cases include phishing, spoofing, spam, mail bombing, fraud, threats, and harassment.

**Practical sequence:** preserve the original message with full headers and attachments → make an analysis copy → examine the attachment safely → read `Received:` lines starting from a server you trust → compare timestamps, message IDs, and addresses → verify the route against mail-server, router, or firewall logs → document what is inferred. Displayed sender addresses and many header fields can be forged; an IP address alone does not identify the person who sent a message. A screenshot or printed email may omit essential headers.

In Outlook-oriented investigations, **PST** files store local mail/archival data, while **OST** files cache synchronized mailbox data for offline use. Search the relevant user profile, server-side mailbox, and backups as authorized; older paths in the slides are version specific.

### Web attacks and evidence

| Attack | Core mechanism | First evidence to correlate |
| --- | --- | --- |
| **XSS** | Untrusted content executes as script in a user's browser: stored, reflected, or DOM-based. | Requests, affected page/content, browser behavior, server logs. |
| **CSRF** | A user's browser is induced to send an unwanted authenticated request. | Request/session context and state-changing action. |
| **SQL/code injection** | Untrusted input changes a query or command's meaning. | Input, server/application logs, database changes. |
| **Parameter/cookie tampering** | A client alters values that the application trusts. | Captured requests, session records, server decisions. |
| **DoS/DDoS** | Excess traffic or resource exhaustion reduces availability. | Traffic capture, firewall/router logs, resource graphs, affected service timeline. |

Use packet captures (for example Wireshark or `tcpdump`), web/mail/server logs, and IDS alerts as **separate pieces of evidence**. Align their clocks and validate one source against another.

## 11. Rapid revision: the distinctions that matter

| Do not confuse | Difference |
| --- | --- |
| **Sector / cluster** | Sector is an addressed storage unit; cluster is a file-system allocation unit, often containing multiple sectors. |
| **Slack / unallocated** | Slack is unused space *inside an allocated file's final cluster*; unallocated clusters belong to no current file. |
| **Physical disk image / logical extraction** | A disk image can include raw areas outside live files; a logical extraction requests data through a file system or device interface. Actual coverage depends on the tool and device. |
| **Live / dead analysis** | Live captures changing state; dead analyzes a stopped system or preserved image. |
| **Hash / chain of custody** | Hash checks content equality; custody documents handling and access. |
| **Backup / forensic image** | A backup can omit deleted space, slack, metadata, or excluded files; verify its coverage before calling it a forensic image. |
| **Reactive / proactive database forensics** | Reconstruct after an incident / prepare trustworthy logging and provenance before one. |
| **Static / dynamic malware analysis** | Inspect without execution / observe execution in containment. |
| **DoS / DDoS** | Overload from an attack source / coordinated overload from multiple systems. |

**If you can explain the path from a raw device or event to a supported conclusion, while accounting for volatility, changes made by tools, and gaps in the evidence, you understand the central lesson of both modules.**
