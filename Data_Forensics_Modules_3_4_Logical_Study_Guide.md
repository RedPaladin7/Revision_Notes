# Data Forensics, Modules 3 and 4: A Guide That Builds One Step at a Time

**Based on:** *Systems, Disks, Media and Boot Process* (Module 3) and *Advanced Forensics* (Module 4).

## The thread we will follow

Imagine that a company's customer records have changed without permission. The investigator needs to answer: **What changed, who or what did it, how did the data move, and what evidence supports that account?** The answer might involve a computer's disk, its running memory, a network router, a database, an email, and a phone. We will learn those sources in the order in which they become useful.

The path through this guide is:

1. Understand and preserve evidence.
2. Understand how a device stores bytes and files.
3. Understand how a computer starts and leaves traces.
4. Make a reliable copy and recover files, including pictures.
5. Examine Windows, Linux, and older Mac systems.
6. Follow the activity across networks, databases, phones, malware, email, and the web.
7. Put independent observations together into a defensible conclusion.

**Two different “images”:** A **disk image** is a copy of a drive or partition for examination. A **picture image** is a photograph or graphic such as a JPEG. Both appear in the slides; context determines which one is meant.

---

## 1. Before touching a device: what counts as evidence?

Digital forensics means finding relevant digital data, preserving it, examining it, and explaining what it does and does not show. A file's presence is an observation. Claiming that a particular person created or sent it is an **interpretation** that needs supporting evidence.

### 1.1 The basic investigation sequence

1. **Identify and document:** What devices and accounts are involved? Record their condition, time settings, connections, and who found them.
2. **Preserve:** Decide what could disappear if a device is shut down or disconnected. Record any action that might change it.
3. **Acquire:** Make the most complete justified copy available. Record the tool, settings, errors, and what the method could not copy.
4. **Verify and protect:** Compute a hash of acquired data, preserve the original or master copy, and work on a separate copy.
5. **Examine and correlate:** Look for files, metadata, deleted material, logs, memory, and independent records that describe the same event.
6. **Report:** State what was observed, how it was obtained, how the conclusion follows, and what remains uncertain.

This same sequence applies to a disk, router, phone, email account, or database. The details of acquisition vary.

### 1.2 Why the order matters: volatility

A running computer is changing constantly. **RAM** holds current processes and sometimes keys or messages; the **process list**, **open connections**, **ARP cache**, and **routing table** can change or disappear. Disk files and backups generally last longer. RFC 3227 presents a broad **order of volatility**: CPU registers/cache; routing, ARP, process and kernel state; RAM; temporary file systems; disk; remote logs; physical configuration and topology; archival media. Apply this as a principle, not a command to collect irrelevant data. A router's running configuration may be in RAM while its saved startup configuration persists in nonvolatile memory.

**Live examination** collects information from a running device. It can capture fleeting evidence, but every command may change state. **Dead examination** analyzes a powered-off device or, preferably, a preserved image on a trusted analysis machine; it cannot recover RAM that was never captured. If a phone might lock when powered off, that changes the decision. Document the tradeoff.

### 1.3 Hash and chain of custody answer different questions

A **cryptographic hash** turns input bytes into a fixed-length digest. If a disk image has the same hash before and after transfer, that strongly supports that its bytes did not change. It does **not** establish that the original bytes were truthful or identify who created them.

- **MD5** produces 128 bits, usually shown as 32 hexadecimal characters. The slides show its padding, 512-bit blocks, and processing rounds. Its main lesson here is that a small input change changes the digest, **but attackers can produce collisions**. Do not depend on MD5 alone for adversarial integrity or use it for passwords or signatures.
- **SHA-256** is a better default for forensic copy verification. Store the hash, the exact item hashed, and the calculation time with the case notes. For passwords, use a purpose-built password hashing method such as Argon2, bcrypt, or scrypt instead.
- **Chain of custody** is the human and procedural record: who collected an item, when, where, why, how it was stored, and each subsequent transfer. It complements the hash.

**Running example:** If the customer-record export appears in an acquired drive image, its hash and custody history support that the *copy* is unchanged. A database audit log and network records are still needed to establish what happened to the export.

---

## 2. Follow the bytes: from a storage device to a file

To recover evidence that a normal file browser does not show, start at the bottom and build upward:

**storage device → addressed sectors/blocks → partition → file system → allocated clusters and metadata → visible files**.

### 2.1 The physical media and how a hard disk reads data

A traditional **hard disk drive (HDD)** keeps magnetic patterns on rigid, spinning **platters**, so it retains data without power. A **read/write head** accesses a platter surface. A **track** is a circular path on a surface; a **sector** is a piece of that path used for addressing. A track contains sectors; a disk can have several recording surfaces. Hard disks are usually fixed inside a computer but may also be external/removable. Common historical form factors include 3.5-inch desktop and 2.5-inch laptop drives; older 5.25-inch models also existed.

Common logical sector sizes are **512 bytes** and **4,096 bytes** (4 KiB, often called Advanced Format). A sector is **not** the same thing as a file. Modern drives typically expose **logical block addresses (LBA)**; the simple tracks-and-sectors geometry in the slides is mainly a useful teaching model for capacity problems.

**Bad sectors** cannot be read reliably. Imaging software should record read errors; sometimes specialized recovery or repeated reads can recover some data, but missing bytes must be acknowledged. `chkdsk`, `fsck`, ScanDisk, and `badblocks` are associated with finding or handling bad areas. Repair operations can change a volume, so examine a protected copy, not the original evidence.

The **interface** connects a drive or storage system to its host. The slides name IDE/EIDE/ATA (older PCs, including master/slave jumper settings), SCSI (multiple addressed devices), USB (portable external devices), and Fibre Channel (storage networks). These identify how to access a source; they do not by themselves say what files it contains.

Other media matter for the same reason: **magnetic tape** is often a sequential backup source; **floppy disks** are older magnetic media; **USB drives and flash cards** (SD, CF, MMC, Memory Stick, and others) are portable solid-state sources; **CD, DVD, and Blu-ray** are optical discs. Optical discs record data along a spiral rather than HDD tracks. CD-ROM is read-only, CD-R is generally recordable once, and CD-RW is rewritable; DVDs have several recordable/rewritable variants. Typical historical capacities in the slides are about 700 MB for CD, roughly 4.7 GB for a common single-layer DVD, and 25/50 GB for single/dual-layer Blu-ray. Check the actual medium and format instead of assuming a nominal capacity.

### 2.2 Capacity arithmetic from the disk diagrams

The slides' simplified model is:

```text
capacity in bytes = bytes per sector × sectors per track
                    × tracks per recording surface × number of recording surfaces
```

**Forward example:** `512 × 50 × 2,000 × 4 = 204,800,000 bytes`. Dividing by `1,024²` gives about **195.31 MiB**. The slides call this “MB,” but because they divide by powers of 1,024, **MiB** is the precise label. Similarly, divide by `1,024³` for GiB or `1,024⁴` for TiB. Decimal drive marketing often uses MB = 1,000,000 bytes instead.

**Another example with 4 KiB sectors:** `4,096 × 128 × 200,000 × 8 = 838,860,800,000 bytes = 781.25 GiB`.

**Inverse questions:** If capacity and three factors are known, divide by their product. For example, with a capacity of `256 × 1,024²` bytes, 512 bytes/sector, 4,096 tracks/surface and four surfaces, `sectors/track = 268,435,456 ÷ (512 × 4,096 × 4) = 32`. The same rearrangement finds tracks or surfaces. If only the number of surfaces doubles, capacity doubles. This covers the seven capacity exercises without learning seven unrelated formulas.

### 2.3 Partitions: dividing the device

A **partition** is a defined region of a disk. A partition may contain an operating system and a file system, and one device may hold several partitions. An examiner checks **every partition**, including hidden, unused, or apparently empty ones.

In **legacy MBR partitioning**, the first sector contains boot code and a table with **four entries**. An **extended partition** can be one of those entries and contain more logical partitions. This four-entry rule belongs to MBR, not to all modern disks: GPT uses a different partition layout.

### 2.4 File systems: turning locations into names

A **file system** maps names such as `report.csv` to the storage used by that file. It records directories, file sizes, allocation and free space, and often timestamps, ownership, or permissions. The hierarchy starts at a **root** and branches into directories and files. Older systems often restricted names to **8.3** characters; later systems support long names.

The file system allocates space in **clusters** (allocation units): groups of one or more sectors. A file that needs any part of a cluster normally occupies the whole cluster. This explains three different places an investigator may find data:

| Place | What it means | Example of possible evidence |
| --- | --- | --- |
| **Live file** | File system currently points to its allocated content. | Current export and metadata. |
| **File slack** | Bytes after a file's logical end in its last allocated cluster. | Possible remnants; exact contents depend on file system and how storage was written. |
| **Unallocated space** | Clusters not currently assigned to a live file. | Deleted-file content may remain until reused. |
| **Lost cluster** | Marked allocated but not referenced by a normal file entry. | Orphaned fragments from an error or interrupted operation. |

**Calculation:** With 4,096-byte clusters, a 1,500-byte file uses one cluster. The remaining `4,096 − 1,500 = 2,596` bytes form its last-cluster slack. A rough textbook estimate of average unused space is **half a cluster per file**, but it is a model, not a measured amount. Some older systems also distinguish **RAM slack** (padding sourced from memory) from the remainder of a sector or cluster; do not assume modern systems expose readable RAM there. SSD TRIM, encryption, overwrites, and file-system behavior can limit any recovery.

### 2.5 Recognize the file-system families

| Family | How it organizes data | What to inspect / remember |
| --- | --- | --- |
| **FAT12, FAT16, FAT32** | A File Allocation Table follows chains of clusters. FAT12/16 served smaller media; FAT32 supports larger ones; **VFAT** adds long filename support. | Directory entries, cluster chains, deleted entries, free space. Slide capacity limits are historical and depend on implementation and cluster size. |
| **NTFS** | The **Master File Table (MFT)** holds records describing files and directories with named **attributes**. | `$Mft`, `$MftMirr`, `$LogFile`, file times, names, data, permissions, and **alternate data streams (ADS)**. Small attributes can be **resident** in an MFT record; larger ones are **nonresident** in external clusters. `$MftMirr` mirrors only critical MFT records, not the entire table. |
| **ext2, ext3, ext4** | Linux file systems use **inodes** for metadata and directories to associate names with inodes. ext3/4 add journaling. | Inodes, directories, block allocation, journal and timestamps. Linux's VFS gives programs a common interface to multiple file systems. |
| **MFS, HFS, HFS+** | Older Apple file systems covered in the slides. | Metadata and the **data fork** and **resource fork** of older Mac files; do not treat the resource fork as invisible or expendable. |
| **ZFS** | Pools storage using **zpools** and **virtual devices (vdevs)**, with features such as compression. | Pool configuration and metadata; if available, snapshots. |
| **SMB/CIFS, NFS, NetWare protocols** | File access over a network, rather than a local file-system format. Samba implements SMB sharing on Unix-like systems; NFS is common in Unix/Linux. | Client and server copies, share settings, logs; do not assume the only copy is on the examined workstation. |
| **ISO 9660, UDF** | Formats used on optical discs. | Disc contents and format-specific filenames. Joliet, Rock Ridge, and Apple extensions address compatibility/names; **El Torito** supports bootable CDs. |

**NTFS features in context:** **Compression** changes how content is stored without changing the logical file. **EFS** encrypts individual files using file keys protected via user keys/certificates; it is not the same as full-disk encryption. NTFS can remap data away from bad clusters. An **ADS** adds a named stream to a file, such as `ordinary.txt:extra`; normal directory views can miss the extra stream, but ADS also has legitimate uses. These features affect how evidence is found and read.

---

## 3. How the computer starts, and why boot records matter

Now we know where partitions and files live. On startup, firmware must find a bootloader before the operating system can display a desktop. Tampering with early boot code can run before normal OS protections.

### 3.1 The legacy BIOS/MBR route

1. Power becomes stable; the CPU starts firmware.
2. **BIOS** runs **POST** (Power-On Self-Test) to check and initialize hardware; a cold start usually performs more checks than a restart. Beeps can signal hardware problems.
3. BIOS chooses a boot device and reads its first sector, the **MBR**. A traditional MBR is 512 bytes and contains startup code, a partition table, and the ending signature **`0x55AA`**.
4. The boot code locates the active partition and its volume boot code, which finds an **OS bootloader**.
5. The bootloader loads the kernel and the system starts services and applications.

**Windows XP example from the slides:** `NTLDR` reads `boot.ini` for boot choices, uses `NTDETECT.COM` to gather hardware information, and loads the kernel (`ntoskrnl.exe`) and required drivers. `hiberfil.sys` is involved in hibernation/resume; `pagefile.sys` is for paging. Other Windows files named in the slides, such as `hal.dll`, `win32k.sys`, `ntdll.dll`, `kernel32.dll`, `advapi32.dll`, `user32.dll`, and `gdi32.dll`, serve different kernel or user-space roles; memorizing that list is less useful than understanding the sequence. These filenames and steps describe **XP-era** booting. Modern Windows commonly uses UEFI/GPT and different boot components.

**Linux example:** firmware/boot code invokes a loader such as **GRUB** (or the older **LILO**). GRUB selects a kernel, often loads an initial RAM file system (`initrd`/`initramfs`), and passes control to the kernel; boot configuration may be under `/boot/grub`. GRUB can offer multiple kernels or operating systems.

**Historical Mac examples from the slides:** Older PowerPC Macs used **Open Firmware**; Intel Macs of the covered era used **EFI**, `boot.efi`, and the OS X **XNU** kernel, which combines Mach, BSD, and I/O Kit elements. The slides also discuss older Mac OS X boot discs and recovery utilities. These are specific to those machines, not a universal modern Mac boot sequence.

**Forensic question:** Was a strange program launched as an ordinary application, by an OS startup setting, or by modified boot code? Inspect the correct boot scheme for the device and work from a preserved copy. A backup utility such as **Carbon Copy Cloner** can omit files or areas; check its settings before equating a backup with a complete sector-level forensic image.

---

## 4. Make a copy, then ask what it contains

### 4.1 Physical image versus logical collection

A **bitstream/physical image** copies readable sectors or blocks, potentially including allocated files, file-system metadata, unallocated space, and slack. A **logical collection** requests particular files or records through an OS, file system, or device API; it may miss deleted bytes and hidden areas. A **backup** is designed for restoration and can omit both. Coverage always depends on the tool, permissions, encryption, read errors, and device.

Use a suitable **write blocker** or documented read-only approach where possible. Log the source/destination, tool version, settings, skipped sectors, hash, and custody; analyze a working copy. `dd`/`dcfldd`, FTK Imager, EnCase, X-Ways, and WinHex are examples in the slides of copying or examining different kinds of evidence. **Evidor** illustrates keyword searching across files and slack. `dd` is a raw copy utility, not a substitute for correct source selection and validation. **Helix** and **WFT** illustrate older live acquisition suites that record system state and create reports. Treat old version lists as historical.

**Example chain:** acquire disk → verify SHA-256 → map partitions → identify file system → read file records → search deleted and unallocated regions → compare with RAM and logs. Each step answers a narrower question; raw bytes only become meaningful after you locate and interpret their context.

### 4.2 Picture images: an example of file recovery

An exported screenshot or photo could be key to our running case. A **raster** picture is a grid of pixels; enlarging it beyond its resolution shows pixel limits. A **vector** picture stores shapes/paths and scales cleanly. A **metafile** may mix raster and vector elements (examples in the slides include EPS, CGM, WMF, and EMF). **Resolution** describes pixel dimensions; **color depth** affects possible colors.

| Format | The useful distinction |
| --- | --- |
| **BMP** | Often large because common variants store pixels with little or no compression. |
| **GIF** | Up to 256 colors per palette; supports simple animation and transparency. Good for simple graphics. |
| **PNG** | Usually lossless; supports transparency and interlacing. Animated PNG extensions also exist. |
| **JPEG / JPEG 2000** | JPEG is commonly lossy for photos; JPEG 2000 is a different standard with more encoding options. Ordinary JPEG does not support transparency. |
| **TIFF** | Flexible container often used for scans/publishing; compression, color depth, and file size vary. |

**Lossless** compression reconstructs the original data exactly; **lossy** compression throws some information away to reduce size. Techniques named in the slides include run-length, dictionary/LZW, Deflate, and Huffman/entropy coding (lossless), and transform coding, chroma subsampling, and quantization (lossy). A format label alone does not guarantee authenticity or a particular compression level.

**Recovery sequence:**

1. Search normal files, including hidden directories, attachments, removable media, and backups.
2. Search unallocated space and relevant slack in a **disk image** for plausible picture **headers/signatures**. A filename extension can be changed; internal bytes are stronger evidence of type.
3. **Carve** files from raw bytes. If a file is fragmented, recovering its header and first fragment may not recover the rest.
4. If a header is damaged, test any repair using a separate working copy and document exactly what bytes changed. A plausible display after repair does not prove the original image was intact.
5. View a derived copy and inspect metadata such as **EXIF** (camera details, exposure, timestamps). Metadata can be edited; corroborate its time and origin with other records.

The slides mention **GFE Stealth**, **P2 eXplorer**, **ILook**, EnCase, WinHex, and ordinary picture viewers such as XnView and IrfanView. Their roles are acquisition, opening forensic containers, extraction/hex inspection, and convenient viewing. Some tools and supported formats are historical. Viewers may edit an image or change file access metadata: give stakeholders a documented **copy**, not the protected master. Write-once media such as CD-R/DVD-R are historical preservation examples.

**Steganography** hides data inside an apparently ordinary carrier, which can be a picture, audio file, document, or executable. The hidden content might also be encrypted. A **watermark** embeds an ownership or provenance marker. Neither the mere presence of a picture nor a suspicious file size proves hidden data. The slides also raise image copyright: ownership and permission are separate questions from whether the file is evidence.

---

## 5. Examine an operating system: where does activity leave traces?

The disk gives persistent traces. A running OS also gives processes, memory, open connections, configuration, and usage records. Choose what to collect based on the question: did a user export a file, did a program run, or did the machine send data?

### 5.1 Windows: start with the evidence question

| Question | Places to check | Example tools or commands from the slides |
| --- | --- | --- |
| What is running now? | RAM, processes, logged-on users, network connections, DNS cache, system time. | FTK Imager/Magnet RAM Capture/DumpIt for acquisition; `tasklist`, Process Explorer, PsList; `netstat -ano`, `ipconfig /displaydns`, PsLoggedOn. |
| What files existed or changed? | File timestamps/attributes, NTFS MFT and ADS, slack, unallocated space, unused/hidden partitions. | FTK/EnCase/X-Ways/Autopsy; `dir /O:d` sorts listings by date; `attrib /s` finds attributes; `Sigverif` checks file signatures in older Windows workflows. |
| What settings or memory artifacts remain? | Registry, memory dumps, page file (virtual-memory pages), shadow copies, system-state backup. | RegScanner; memory tools; `vssadmin list shadows` for available copies. |
| What web or mail activity occurred? | History, cache, cookies, typed URLs, downloads, browser databases; PST/OST. | Browser-history viewers, Autopsy, mail examiners; correlate with server logs. |

**Virtual memory** gives processes an address space larger or different from physical RAM; the OS may page some memory to disk. Consequently the **page file** may retain fragments of activity after RAM changes. A **crash dump** may contain stop information, drivers, kernel/process/thread state, and call stacks. A dump is a snapshot with limits, not a complete history.

The **Registry** holds Windows and application configuration. **System-state backups** on relevant server versions can include Registry, boot files, Active Directory data, SYSVOL, COM+ registrations, or IIS configuration depending on roles. **Volume Shadow Copies** may preserve earlier versions; presence and coverage vary. **NTFS ADS** should be explicitly checked when a visible file seems innocuous.

Older examples such as **Helix, WFT, BartPE, fport, PsList, Rootkit Revealer, Mail PassView, PMDump, `index.dat`, and old Internet Explorer directories** illustrate *types of inquiry*: live collection, port-to-process mapping, memory dumping, browser traces, and bootable inspection. Their historical versions and paths do not automatically apply to a present-day Windows machine. Live tools may alter the system; log their use.

### 5.2 Linux: know the tree, then select the tool

Linux represents devices under `/dev`. The slides use older IDE names such as `/dev/hda` and partitions `/dev/hda1`; other drives may be `/dev/sda` or `/dev/nvme0n1`. **Never infer the evidence device from its name alone**: check model, size, partitions, and connections with tools such as `lsblk` and `fdisk -l`.

| Path | Plain-language role and likely evidence |
| --- | --- |
| `/` | Root of the directory tree. |
| `/boot` | Kernel/bootloader files; `/etc` system and service settings. |
| `/home`, `/root` | Ordinary users' homes; root administrator's home. |
| `/var` | Changing data such as logs and spools; `/tmp` temporary files. |
| `/proc` | Live process/kernel information; much of it is generated on demand rather than stored as ordinary disk files. |
| `/dev` | Device entries; `/mnt` often used as a mount point. |
| `/bin`, `/sbin`, `/lib`, `/usr`, `/opt` | Commands, system programs, libraries, general software, and add-on software. Exact directory arrangements vary by distribution. |

**A safe conceptual workflow:** identify the source → write a raw image to a *different* destination (`dd` or `dcfldd` can do this) → record errors and hash the image → keep a protected master → examine its partition table and file systems. If mounting an image, use a carefully verified **read-only** mount, with `noexec,nodev` when suitable; journaling and other file-system features may require extra options or a forensic viewer to avoid replay/writes. `-t` chooses a file-system type, `-o ro` requests read-only, and a loop device can expose an image. `rw` would allow writes; avoid it on evidence. Commands such as `chmod 440` may protect a working file but are not a substitute for an immutable master or write blocker.

Simple helpers: `file` tests file type, `xxd` shows hexadecimal bytes, `strings` extracts readable text, `grep` finds patterns, `awk` selects/processes fields, and `sed` transforms text. Use them on appropriate copies and verify that their output represents the right source and encoding.

**The Sleuth Kit (TSK) follows the same layers we learned in section 2:**

| Layer / question | Representative TSK commands |
| --- | --- |
| What does the medium/image contain? | `img_stat` describes image format; `img_cat` accesses raw image content; `mmls` maps partition layout. |
| What file system is this? | `fsstat` describes it. |
| Which names and metadata exist or were deleted? | `fls` lists names; `ffind` resolves names from metadata; `istat` describes metadata; `ils` lists metadata; `ifind` finds metadata associated with a data unit; `icat` extracts file content. |
| What is in a raw or unallocated data unit? | `dcat`, `dstat`, `dls`, `dcalc` inspect/extract or map data units. |
| What is in the file-system journal? | `jls` lists and `jcat` displays journal material. |
| Can we search or make a timeline? | `sigfind` searches signatures, `hfind` checks hash databases, `sorter` groups file types, `mactime` builds a file-activity timeline. |

**Autopsy** provides a case-oriented graphical workflow for many of these jobs: file listing, file signatures, keyword searching, hashes, timelines, metadata, notes, time zones, and reports. A **hash database** can help separate known files from suspicious ones; it is not a verdict. Older descriptions call Autopsy an HTML front end on Unix; its interface and platform support have changed. A reproducible case records which image, timezone, offsets, tool versions, and searches produced each observation.

### 5.3 Older Mac systems: preserve both kinds of file content

On HFS-era Macs, a file can have a **data fork** (ordinary content) and a **resource fork** (additional program resources/metadata). A tool that copies only one fork may omit relevant evidence. The slides' **BlackBag** suite, **FileSpy**, **HeaderBuilder**, **Comment Hunter**, **File Searcher**, **GraphicView**, **Breakup**, **LockMaster**, and a Mac `dd` assistant are examples of viewing forks, searching metadata, repairing headers on copies, previewing media, splitting large exports, and controlling file handling. **MacDrive** describes older Windows-based access to Mac volumes. A password-reset utility or repair option can modify evidence; document any use and prioritize methods that preserve the original.

---

## 6. Follow the data off the computer: network and routers

If the export left the machine, disk evidence alone cannot tell us its route. **Network forensics** uses captured packets, connection logs, and network-device configuration to reconstruct communication.

A **router** moves packets between networks using IP addressing (OSI layer 3). A **routing table** tells it where to send traffic. A **static route** is configured by an administrator; a **dynamic route** is learned and updated by a routing protocol. **RIP** uses hop count (a distance-vector approach); **OSPF** uses link-state information and shortest-path calculations. Route selection can consider cost, delay, bandwidth, load, reliability, or hop count depending on the protocol. A change in route may be ordinary configuration or an attack; compare configuration history and independent logs.

**Collect in this order when relevant:** record router time and current user sessions; capture running configuration and routes (volatile); then saved startup configuration, ACLs, logs, and version; obtain packet captures, firewalls/IDS records, and neighboring device logs. On Cisco-like equipment, examples include `show clock`, `show users`, `show ip route`, `show access-lists`, `show running-config`, `show startup-config`, and `show version`. Document all commands and transfer methods. Do not reset a router before preserving its live state. Synchronize time zones and clock offsets before comparing records.

### 6.1 What attack patterns might explain the traffic?

The slides present a possible intrusion sequence: **reconnaissance → scanning/enumeration → access → privilege escalation → persistence → covering tracks**. Use it as a set of investigative questions, not a rule that every attacker follows exactly six steps. Search normal, hidden, and deleted files, slack, logs, and configuration changes. A Windows hidden attribute can conceal a file from ordinary views; it does not remove its bytes.

**Routing-table poisoning** introduces false route information; this can divert or disrupt traffic. **Denial of service (DoS)** makes a service unavailable by exhausting bandwidth, CPU/memory/connection capacity, or triggering a failure. **Distributed DoS (DDoS)** uses many sending systems, sometimes coordinated through controllers and compromised “zombie” devices. A **SYN flood** sends many TCP connection starts without completing them; **Smurf/Fraggle** are older broadcast/reflection examples; **amplification** exploits a small request that provokes a larger response. **Ping of Death**, **Teardrop**, **Land**, **Trinoo**, and **TFN** are historical examples in the slides. Learn the failure mode rather than outdated port numbers. A traffic spike is a lead; distinguish attack from legitimate load using logs, packet patterns, and affected service times.

**Response and reporting:** triage/identify → contain and investigate → recover and prevent recurrence. Record actions and links from network evidence to host evidence. An IP address identifies a network endpoint at a time, not automatically a human actor.

---

## 7. The database: what happened to the records?

Now ask a narrower question: **which rows changed, who accessed them, when, and through what application or account?** A valid login does not prove an authorized action; insiders can misuse access. Preserve database files, transaction/audit logs, backups, application logs, and server/network records within the investigation's scope.

| Approach | When it acts | What it contributes | Main limitation |
| --- | --- | --- | --- |
| **Reactive forensics** | After an incident. | Reconstructs prior data and transactions from available files, logs, and backups. | Missing logs, altered data, and DBMS-specific formats may prevent complete reconstruction. |
| **Proactive forensics** | Designed before an incident, operates during it. | Audit logging, monitoring, integrity checks, and protected evidence collection make later reconstruction faster and more reliable. | Requires correct configuration and protected retention ahead of time. |

**Data provenance** traces a record or change to its source, time, location, and sequence of transformations. For example, a database audit entry attributes an update to an account; an application request and network session may show how that account was used. **Non-repudiation** aims to prevent an actor from plausibly denying a recorded action, but no single log guarantees a person's identity. An architecture can combine database, server, and network records, with integrity protection and custody from collection through reporting. The slides discuss both external attacks and serious insider risks; neither category should be assumed without evidence.

**Cloud forensics appears in Module 4's outline, but the supplied slides contain no developed cloud-forensics section.** The transferable idea here is to identify who controls the data and logs, preserve them within the authorized scope, and account for missing sources; there is no separate cloud procedure to memorize from these decks.

---

## 8. The phone: another copy of the same story

A phone can contain messages, calls, contacts, photos, browser/app activity, location data, and backups relevant to the export. Its evidence may reside in **RAM**, internal **flash**, removable **SD**, a **SIM/UICC**, or an associated backup. The SIM can contain **IMSI** (subscriber identity), **ICCID** (card identity), and sometimes contacts/messages or network information; PIN/PUK relate to access. The SIM is **not** a copy of everything on the phone. Encryption and security hardware can make raw stored bytes unreadable without keys.

**First preserve the state:** photograph and note device model, whether it is on/unlocked, battery level, network connections, displayed notifications, and time. Decide how to reduce remote changes or wiping while avoiding accidental lockout. Airplane mode changes the device; a Faraday bag or shielded enclosure can isolate radio signals but may have practical consequences. Record the choice and its effect. Use appropriate packaging and maintain custody.

Then select the least disruptive method that answers the question and is available on that model:

| Method | Typical result and limitation |
| --- | --- |
| **Manual view** | What the screen displays; narrow scope and actions may change state. |
| **Logical extraction** | Data exposed through OS interfaces, backups, or APIs; may omit deleted and protected areas. |
| **File-system or lower-level extraction** | More files/metadata when access is possible; exact coverage depends on permissions, device, and encryption. |
| **JTAG, chip-off, micro-read** | Hardware-level access; invasive, technically difficult, and encrypted bytes may still be inaccessible. |
| **Cloud or backup acquisition** | A related copy from another source, with its own times, coverage, and authorization. |

Bootloader exploits, rooting, and jailbreaking may enable more access but also change the device. Tools named in the slides include **Cellebrite UFED, Oxygen Forensics, XRY, MOBILedit**, and mobile-capable analysis software. Validate outputs, log versions and errors, hash what can be hashed, and correlate a phone message with server-side and recipient-side records. Consent, warrants, privacy, jurisdiction, and defensibility define the allowed scope; the slides refer to **NIST SP 800-101** as a guidance example.

---

## 9. Malware: could a program have done it?

Suppose the export occurred when nobody admits being at the computer. A malicious program is one possible explanation. **Malware** includes viruses (attach to other files), worms (spread themselves), trojans (masquerade as useful software), spyware, ransomware, backdoors, rootkits, botnet components, and fileless techniques. Symptoms such as unknown processes, new files/accounts, changed security settings, encrypted files, crashes, or odd connections are leads, not proof.

Follow an evidence-driven sequence: **entry point → execution → changes/persistence → network activity → affected data**. Collect candidate binaries/scripts, hashes, file/Registry traces, memory, logs, and traffic. **Indicators of compromise (IOCs)** are observable clues such as a hash, path, or domain; **TTPs** describe the behavior and method. Finding one IOC can link incidents but does not prove an actor's identity.

**Static analysis** inspects a sample without running it: hashes, `strings`, headers such as Windows PE, signatures/YARA, disassembly (IDA/Ghidra). It is generally safer, but packing and obfuscation hide behavior. **Dynamic analysis** runs a sample in a carefully isolated sandbox and observes processes, files, Registry, and network calls (for example Procmon, Wireshark, Cuckoo); behavior can vary with environment and triggers. **Volatility** can inspect a captured memory image. Malware *forensics* adds preservation, custody, timeline, correlation, and reporting to understanding the program. If a sample sent data, compare that observation with actual network and database records before concluding it exfiltrated this case's export.

---

## 10. Email and the web: common routes for the incident

### 10.1 Email: preserve the message, then trace it

An email has **headers** (routing and identifiers) and a **body** (message and attachments). Mail clients and servers may each keep different copies. SMTP carries messages between mail systems; the slides cite port 25 as a common server-to-server example. Webmail can bypass some local corporate mail-client records. Typical cases include spam, phishing, spoofing, mail bombing, threats, and fraud. A **mail storm** can also create large volumes of automated mail without a deliberate mail bomb.

**Investigation sequence:** preserve the original message with complete headers and attachments → work on a copy → examine attachments safely → inspect `Received:` lines from a **trusted receiving server** backward → compare message ID, addresses, IPs, and times → corroborate with mail-server, firewall, router, and account logs. A printed page or screenshot lacks much of the header detail. Many header fields and the visible sender can be forged; even a plausible originating IP does not by itself identify the person. A receiving server's own recorded handoff is more dependable than arbitrary upstream claims.

Outlook **PST** files can store local mail/archives; **OST** files are offline caches of a synchronized mailbox. Relevant folders may include Inbox, Drafts, Sent Items, and Calendar. Examine client, server, and backup copies as authorized; the slides' `Local Settings` path applies to older Windows versions. Email examiners, deleted-mail recovery tools, IP registration lookup, and header analyzers are aids, not independent proof of sender identity.

### 10.2 Web attacks: connect request to effect

| Attack | What the attacker changes | Evidence to connect |
| --- | --- | --- |
| **Stored/reflected/DOM-based XSS** | Gets untrusted script to run in a visitor's browser, via saved content, a response, or client-side page logic. | Request/page content, browser behavior, web logs. |
| **CSRF** | Tricks an authenticated user's browser into submitting an unwanted action. | User session, request context, changed state. |
| **SQL injection / code injection** | Makes input change the meaning of a database query or program command. | Submitted input, application/server logs, query or resulting DB changes. |
| **Parameter tampering / cookie poisoning** | Alters URL, hidden-form, or cookie values that an application trusts. | Original and modified requests, session data, application decisions. |
| **DoS / DDoS** | Exhausts a service or supporting network resources. | Traffic and timing, router/firewall logs, server metrics, failure timeline. |

Wireshark/`tcpdump` can inspect packets; **Snort** can produce detection alerts; web proxies can show HTTP transactions; server, router, firewall, and system logs show other parts of the story. An alert is a starting point. Align clocks, look for the actual changed records or service failure, and corroborate independent sources.

---

## 11. Finish the running investigation

Suppose the database audit log shows an export at 10:03; the Windows machine contains a matching CSV; its browser history records a file upload at 10:07; the router logs show an outbound connection; and a phone contains a message about that export. A careful report separates three levels:

1. **Observed:** the precise records, hashes, file contents, timestamps, and where each was obtained.
2. **Supported interpretation:** several sources agree that this machine or account exported and transmitted a file during that time window.
3. **Remaining uncertainty:** clocks may differ; a shared account may not identify its user; a connection alone does not prove what bytes crossed it; a restored file may be incomplete.

Check the database and transmitted-file contents if available. Reconcile time zones and clock drift. Preserve alternate explanations. The end goal is a conclusion another investigator can retrace from documented evidence, rather than a story built around one suggestive artifact.

### Quick check: explain these without looking back

1. How can the same deleted picture appear in unallocated space yet not in a file browser?
2. Why is file slack different from unallocated space, and how do you calculate its maximum size for a given file?
3. What does SHA-256 verify, and what does chain of custody add?
4. Why collect live RAM and router state before shutting down a system when they matter?
5. What changes between BIOS/MBR booting and a modern UEFI/GPT system, and why is `NTLDR` an XP-era example?
6. How do FAT cluster chains, an NTFS MFT record, and a Linux inode help you locate content?
7. When does a logical phone extraction miss material that a more complete collection might reach?
8. How would you connect a web request, a database edit, an email, and a network connection without claiming that one log alone identifies a person?

If you can answer these in order, you have the underlying line of thought for both modules: **where bytes live → what can disappear → how to preserve them → how to interpret each layer → how to corroborate a conclusion**.
