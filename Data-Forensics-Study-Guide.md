# Data Forensics Study Guide

Course: Data Forensics (DSE 4443), Dr. Sindhura D N, School of Computer Engineering, MIT Manipal.

This guide rebuilds the three lecture modules into one reading order. Topics that the slides repeat (chain of custody, warning banners, volatile data, imaging) are explained once, in the place where they become useful. Lists of product names are grouped by what the tool is for, because the names matter less than the job they do.

The legal examples in the lectures are mostly United States and Canadian rules (Fourth Amendment, ECPA, Canada’s Charter). They are included because that is what the course teaches, even though the class is taught in India.

---

## How to use this guide

Read it in order the first time. Later, use the checklists at the end of each part and the quick reference in Part 13.

Five ideas show up everywhere. If you can explain these in your own words, most of the course follows from them:

1. **Work on a copy.** The original evidence is examined as little as possible.
2. **Prove the copy is exact.** A hash (MD5, SHA-1, or a checksum such as CRC) shows the duplicate matches the original.
3. **Write down who had it.** Chain of custody is the record that evidence was not swapped or altered.
4. **Collect what disappears first.** Volatile data is taken before data that sits safely on disk.
5. **Deletion is usually not destruction.** Removing a file removes its directory entry. The bytes often remain until something else overwrites them.

---

## 1. What digital forensics is

**Digital forensics** applies computer science and investigative procedure to digital evidence from computers, phones, and other storage. The purpose is to collect, analyze, and present that data so it can be used in a criminal, civil, or administrative case.

NIST’s description of a sound forensic process includes all of the following:

- lawful search authority
- a chain of custody
- validation with mathematical hash functions
- validated tools
- repeatability (another competent examiner can reach the same result)
- a detailed report
- expert testimony when the case requires it

**Computer forensics**, as used in these modules, is the same discipline applied to computers and their storage: identify, preserve, analyze, and present digital evidence in a legally acceptable way. The word *forensics* comes from Latin and refers to a forum where legal disputes are decided. The point of the work is a decision-maker, not a repaired hard drive.

### Forensics is not data recovery

| | Data recovery | Digital forensics |
|---|---|---|
| Goal | Get the user’s files back | Produce evidence that can be relied on in a case |
| Typical cause | Accidental delete, power loss, hardware failure | Hidden, deleted, or deliberately concealed data, plus ordinary files that matter to a case |
| Constraint | Restore the data | Do it without changing the original, and be able to prove that |

Evidence can cut either way. **Inculpatory** evidence suggests guilt or liability. **Exculpatory** evidence supports innocence. An examiner looks for both. Cherry-picking only the damaging files is not a defensible examination.

### What an examiner actually does

Four activities cover the job:

1. **Collect** data so the original stays unaltered.
2. **Examine** it: file types, dates created and modified, and whether the content is relevant.
3. **Present** findings clearly, with documentation a court can follow.
4. **Stay inside the law** that governs privacy, search, and digital rights.

Examiners usually start knowing very little about what is on a device. They search methodically with forensic software across the drive, memory dumps, and deleted space. Chip-off work and electron microscopes exist for damaged media, but they are rare. Most cases are software examinations of a verified image.

Two qualities of evidence are tested constantly:

- **Authenticity** — this item really came from the source you claim.
- **Reliability** — the methods and tools are trustworthy, so the result can be believed.

### Where the field is still weak

The lectures are frank about this. Digital evidence does not yet have the long laboratory tradition of fingerprints or DNA. There are fewer shared theories, fewer testing standards, and uneven training. The work is sometimes described as more of an art than a settled science. That is exactly why procedure matters: if the science is still maturing, the only thing that keeps a case standing is careful, repeatable, documented method.

### A short history, only as context

You do not need the timeline in detail. The useful point is that tools grew up because crime moved onto computers, and early tools were narrow.

- **1970s.** Financial fraud on mainframes (the classic “one-half cent” insider fraud) created a need for electronic crime investigation. FLETC began training law enforcement on digital evidence.
- **1980s.** Home computers and early operating systems (CP/M, DOS) multiplied file systems. Tools were mostly proprietary and limited to agencies such as the IRS and the RCMP.
- **Late 1980s–1990s.** Xtree Gold and Norton DiskEdit were early recovery utilities. ASR’s Expert Witness could recover deleted files and became the basis of **EnCase**. An old 8 GB disk-size limit forced tools to be rewritten.
- **Current standard names in the course:** AccessData **FTK**, **EnCase**, and **ILook**, plus **Autopsy**. They handle large sets of email, images, and logs, and they keep being updated for new hardware and encryption.

Staying current (publications, certifications, case law) is part of the job because both tools and statutes move faster than a single textbook.

---

## 2. Who investigates, and what kinds of cases exist

### The digital investigation triad

The lectures describe three groups that work together. Think of them as prevention, detection, and proof.

| Group | Job |
|---|---|
| Vulnerability assessment | Test physical systems, operating systems, and applications. Simulated attacks find weaknesses so they can be patched first. Often staffed by experienced penetration testers and system administrators. |
| Network intrusion detection | Watch traffic and logs for anomalies. Use firewall logs, alerts, and user behavior to find unauthorized users, stop further access, and hand evidence to investigators. |
| Digital investigations | Image disks, carve files, build timelines, and examine artifacts. Conclusions have to be evidence-based and ready for court. |

**Network forensics** is the traffic-and-logs side of this picture. It reconstructs an incident: how the attacker got in, which credentials were used, when it happened, and sometimes where the traffic appeared to come from. It can also show which files were opened, copied out, or changed. It is not a full substitute for a disk examination. Network evidence shows movement; disk evidence shows what was stored.

### Public-sector and private-sector investigations

This distinction controls almost every legal rule later in the course.

**Public-sector** investigations are run by police and other government agencies (local, state or national, including bodies such as the FBI). They operate under constitutional limits.

- In the United States, the **Fourth Amendment** bars unreasonable searches and requires due process and judicial oversight.
- In Canada, **Article 8 of the Charter of Rights** protects privacy and requires judicial authorization for digital searches.

**Private-sector** investigations are run by companies, law firms, or consultants. Typical aims are policy violations (email misuse, unauthorized access, data leaks), insider threats, trade-secret theft, and preparation for civil litigation. The result is often discipline or a lawsuit, not a criminal charge. A private case becomes a criminal case the moment investigators find something like child exploitation or drug trafficking. At that point they must notify law enforcement, and from then on they are bound by the stricter public-sector rules.

| | Public sector | Private sector |
|---|---|---|
| Who | Law enforcement and government | Company, counsel, consultants |
| Authority | Statute, warrant, constitution | Company policy, consent, ownership of the system |
| Typical outcome | Prosecution | Discipline, termination, civil suit |
| Evidence standard | Must survive criminal court | Should still be handled as if it might, because it often does |

A criminal case, in outline, moves from **complaint** (a victim or witness reports it), to **investigation** (specialists gather evidence), to **prosecution** (the evidence is put before a court). Police reports and blotters are common triggers. **First responders** secure and preserve the scene. **Digital evidence specialists** examine the systems.

### What the crimes look like

Common computer-related offenses in the lectures:

- unauthorized access and system penetration
- data theft and theft of intellectual property (patents, secrets, customer data)
- fraud, phishing, and fraudulent transactions
- threatening or harassing email, cyberstalking, discrimination
- hacking and spreading viruses or worms
- damage to services (denial of service, Trojans)
- the serious offenses that turn a workplace case into a police case: child exploitation, drug trafficking

**Insider attacks** are breaches of trust by someone already inside the organization. **External attacks** come from outside, sometimes hired by an insider or by a competitor trying to damage reputation.

Motives the lectures list, more than once under slightly different names, reduce to: money, ideology, revenge, thrill or attention, and paid espionage. Attackers are often organized and sometimes more technical than the agency responding. **Industrial espionage** — selling secrets to a rival — is treated as a serious crime. The course example is the 2006 Coca-Cola employee who tried to sell trade secrets to Pepsi and was prosecuted.

A useful way to think about an attack, from the lectures, is tools, targets, and materials that were misused. You do not need a taxonomy beyond that.

### Company policy is the private-sector “law”

In a company, the investigation often starts from policy rather than a penal statute. Clear policies reduce lawsuits and make the process fair.

An **acceptable use policy (AUP)** states which sites, software, and network uses are allowed, how information may leave the company, and what is prohibited (offensive content, unrestricted downloads). It is there to limit the company’s own liability as well as to control staff.

**Warning banners** shown at login do several jobs at once:

- they say the system is for official use
- they say use is monitored
- they say there is no expectation of privacy
- clicking through is treated as consent

Banners are easier to use in a dispute than a long policy manual nobody read. Government and private networks both use them, including on routers and firewalls. If a banner was accepted, a private employer often does not need a court order to look at that system.

Only **authorized requesters** may start an internal investigation: typically corporate security, the ethics office, or legal. That limit exists so one employee cannot weaponize an investigation against another.

An **incident response policy** names the team, who is notified, and which department investigates, repairs, and restores. After an incident, staff are expected to report it, leave the technology untouched, and list who was present and what happened, in order.

Not every policy breach is a crime. A hostile workplace (harassment, bullying, offensive content) can be handled internally, and company rules can reach personal social-media conduct when it affects the workplace. Document every disciplinary step, because the case may later be escalated.

**BYOD** (bring your own device) blurs ownership. A personal tablet on company Wi-Fi can mix personal files with company files. Connecting to the company network can bring the device under company rules, but the policy has to say so in advance. Examiners must not assume that “it was on our Wi-Fi” automatically means “we own every file on it.”

### Document classification

When the lectures talk about confidentiality labels, they mean how widely a document may be shared:

| Label | Meaning in the course |
|---|---|
| Top secret | Disclosure would severely harm the organization |
| Secret | Only a small set of people (for example, security blueprints) |
| Confidential / internal | Law enforcement or the company only (for example, payroll) |
| Restricted | Only people explicitly authorized |
| Unclassified / public | Anyone |

### Professional conduct

Examiners represent their organization. They stay objective, do not editorialize about illegal material, show compassion for victims, and do not discuss the case with anyone who is not authorized. The course’s cautionary example is an officer removed from a case for mocking the contents of seized evidence. Curiosity about the material is not part of the role.

**Check yourself**

- Explain the difference between recovery and forensics using authenticity and chain of custody, not slogans.
- Say who may search a computer in a company, and what changes if the content is a serious crime.
- Name the three groups in the investigation triad and one sentence on each.

---

## 3. How disks and media store data

Forensics keeps returning to the same fact: a computer does not store “files” as whole objects on the metal. It stores magnetic or optical patterns, and a file system is a map. If you understand the map, you understand why deleted files come back, why images must be bit-for-bit, and why slack space matters.

### Hard disks

A hard disk is **non-volatile**. It keeps data with the power off. It is the main permanent store in PCs, laptops, cameras, and DVRs, and it is faster to access than most removable media. Data is written as magnetic patterns on rigid platters.

**Form factors** in the lectures: 5.25 inch (obsolete), 3.5 inch (desktops), 2.5 inch (laptops). The size decides which bay and which power connector fit. Most drives are fixed inside the machine; some are removable. A metal shell keeps dust out.

On older parallel-ATA drives, **jumpers** set master or slave (primary or secondary). The drive also has an interface connector and a power connector.

**Inside the drive**

- **Platters** are flat disks coated with magnetic material. Larger platters hold more, but the layout also affects how fast data can be reached.
- **Read/write heads** fly less than 0.1 microns above the platter. They do not touch it during normal operation.
- Platters spin at thousands of revolutions per minute.
- A **track** is one concentric circle. Tracks are numbered from the outer edge inward. More tracks means higher density and more capacity.
- A **sector** is a slice of a track and is the smallest physical unit the drive addresses. Traditional sectors are **512 bytes**. Newer Advanced Format drives use **4096-byte** sectors because larger sectors waste less space on error correction.
- Tracks and sectors are laid out by **low-level formatting** at the factory. Similar track-and-sector ideas appear on older media such as floppies.

**Bad sectors** are damaged spots. Only those sectors are unusable, not the whole disk. ScanDisk, CHKDSK (Windows), and `badblocks` (Linux) mark them so the operating system will not reuse them. Data already trapped in a bad sector is generally gone. Bad sectors accumulate over the life of the drive. NTFS can **hot-fix** by moving data off a failing sector automatically.

### Capacity

The course formula for a simple disk is:

**Capacity = (bytes per sector) × (sectors per track) × (tracks per surface) × (number of surfaces)**

Units, as the lectures use them, are powers of 1024, not 1000:

| Unit | Size |
|---|---|
| 1 KB | 1024 bytes |
| 1 MB | 1024² = 1,048,576 bytes |
| 1 GB | 1024³ = 1,073,741,824 bytes |
| 1 TB | 1024⁴ bytes |

The ladder continues PB, EB, ZB, YB, but the problems stop at TB.

Work in bytes first, then divide. Do not convert each factor separately or rounding errors will creep in.

**Problem 1.** 512 bytes/sector, 50 sectors/track, 2,000 tracks/surface, 4 surfaces.

512 × 50 × 2,000 × 4 = 204,800,000 bytes = 204,800,000 / 1024² ≈ **195.31 MB**.

**Problem 2.** 512 × 63 × 10,000 × 4 = 1,290,240,000 bytes ≈ **1.20 GB**. (63 sectors per track is the old IDE CHS geometry.)

**Problem 3.** 4,096 × 128 × 200,000 × 8 = 838,860,800,000 bytes = **781.25 GB**. This is an Advanced Format, 4K-sector disk.

**Problem 4.** Capacity is 500 GB. Sector 1,024 bytes, 256 sectors/track, 4 surfaces. Find tracks per surface.

500 × 1024³ = 536,870,912,000 bytes.  
Tracks/surface = 536,870,912,000 / (1,024 × 256 × 4) = **512,000**.

**Problem 5.** Capacity 2 TB, 4,096-byte sectors, 256 sectors/track, 262,144 tracks/surface. Find surfaces.

2 × 1024⁴ = 2,199,023,255,552 bytes.  
Surfaces = 2,199,023,255,552 / (4,096 × 256 × 262,144) = **8**.

**Problem 6.** Capacity 256 MB, 512-byte sectors, 4,096 tracks/surface, 4 surfaces. Find sectors per track.

256 × 1024² = 268,435,456 bytes.  
Sectors/track = 268,435,456 / (512 × 4,096 × 4) = **32**.

**Problem 7.** Disk A: 512 × 64 × 10,000 × 4 = 1,310,720,000 bytes ≈ 1.22 GB. Disk B is identical but has 8 surfaces, so ≈ 2.44 GB. **Doubling the surfaces doubles the capacity**, because surfaces are a direct multiplier.

### Interfaces

The interface is how the drive talks to the computer. It affects speed and how many devices you can attach.

| Interface | What to remember |
|---|---|
| IDE / EIDE / ATA | Controller integrated on the drive. The common older PC interface. Master/slave jumpers belong here. |
| SCSI | Faster than early ATA, devices chained and identified by SCSI ID. |
| USB | Plug-and-play external disks. Up to 127 devices. |
| Fibre Channel | Used in storage area networks. Optical fiber, high transfer rates. |

Motherboards often support more than one of these.

### File systems

A **file system** is the set of rules for naming files, arranging them in directories, recording which sectors belong to which file, and tracking free space. It also stores **metadata**: created time, owner, permissions.

Most file systems are a tree. There is a root, then folders, then files. Older DOS names were **8.3** (eight characters, a dot, three-character extension). Modern systems allow long names, up to about 255 characters.

**Microsoft family**

| System | What it is for |
|---|---|
| FAT12 | Small disks and floppies. 12-bit entries. About 4,084 clusters. At 512 bytes/cluster that is roughly 2 MB; at 4 KB/cluster the practical ceiling cited in the lectures is about 16 MB. |
| FAT16 | Early hard disks, roughly 2–4 GB. Wasteful on large volumes because clusters get huge. |
| VFAT | FAT16 extended with long filenames, running in protected mode. |
| FAT32 | 32-bit table. Handles large disks better. Partition size can reach 2 TB, but Windows itself will only *create* FAT32 volumes up to 32 GB. |
| NTFS | The modern Windows file system in this course. Large volumes, security, compression, rich metadata, and hot-fix of bad sectors. |

**NTFS in more detail**

The **Master File Table (MFT)** is a database of every file and folder on the volume. NTFS keeps special metadata files such as `$Mft` (the table itself), `$MftMirr` (a partial mirror, so a damaged MFT can be repaired), and `$LogFile` (a journal used in recovery).

Each file is a set of **attributes**. If an attribute is small enough to sit inside its MFT record, it is **resident**. If it is too big, it is **nonresident** and the MFT record points at clusters elsewhere. Typical attributes include the file name, the data, a security descriptor, an object ID, and timestamps. Attribute lists can be extended, which is one reason NTFS aged better than FAT.

NTFS can compress files, folders, or a whole volume; compression happens when the file is saved or closed. **EFS** (Encrypting File System) encrypts files with public-key cryptography. Each file gets its own file-encryption key, and the user’s private key unwraps it. Access is tied to a digital certificate on the user account. Encryption is transparent while that user is logged in, which matters in forensics: imaging a disk does not decrypt EFS files by itself.

**Other systems named in the course**

- **Linux.** ext, then ext2 (inodes), ext3 (adds journaling), ext4 (very large volumes, up to about 1 EB). The **virtual file system (VFS)** layer lets the kernel support many file systems behind one interface.
- **Classic Mac.** MFS, then HFS, then HFS Plus (32-bit addressing, 255-character names). HFS stores a **data fork** (the content) and a **resource fork** (metadata and Mac-specific resources).
- **Solaris.** ZFS pools storage into zpools made of vdevs, uses large blocks, and has built-in compression.

**Network and optical file systems**

- **SMB** is the Windows network file-sharing protocol. **Samba** implements it on Unix. **CIFS** is the open, internet-oriented SMB variant over TCP/IP.
- **NetWare Core Protocol** provided file, print, and related services on older Novell networks.
- **NFS** is the Unix/Linux remote file system, client/server, built on RPC.
- Optical discs use **ISO 9660** and **UDF**. ISO 9660 extensions you should be able to name: **Joliet** (long names), **Rock Ridge** (Unix permissions and names), Apple ISO, and **El Torito** (booting from a CD).

**Check yourself**

- Compute a capacity from the four factors, in bytes, then in MB or GB using 1024.
- Explain resident versus nonresident NTFS attributes.
- Give one reason FAT16 wastes space on a large disk, and one reason NTFS is preferred.

---

## 4. Partitions, clusters, slack, and booting

### Partitions

A **partition** is a logical division of a disk. Different partitions can hold different operating systems and file systems, which isolates damage and makes recovery easier.

- A disk can have up to **four primary** partitions.
- One of those slots can be an **extended** partition, which is then split into logical drives.
- The **boot partition** holds the files needed to start the operating system.
- The **Master Boot Record (MBR)** lives at **sector 0**. It contains the boot code and the **partition table**. If the partition table entry is deleted, the partition “disappears” even though its clusters are still full of data.
- NTFS stores a **second copy of the boot sector** near the logical middle of the volume so a damaged boot sector can be rebuilt.

Common reasons a partition vanishes: malware, power loss, a damaged MBR, or misuse of a format or partitioning command.

Tools that create or delete partitions, as named in the course: Windows Disk Management (the GUI will refuse to delete the active system volume or an extended volume that still has logical drives), **FDISK** (older DOS tool that can rewrite the MBR), and **DISKPART**. Recovery tools that try to rebuild the partition table include Active@ Partition Recovery, DiskInternals Partition Recovery, GetDataBack, Acronis Recovery Expert, and TestDisk.

### Clusters and slack space

The file system does not hand programs one sector at a time. It allocates a **cluster** (also called an allocation unit): a fixed group of sectors. The cluster is the smallest unit a file can occupy. Cluster size depends on the file system and the volume size. Smaller clusters waste less space and cost more bookkeeping.

A 100-byte file still consumes a whole cluster. The unused bytes at the end of that cluster are **slack space** (file slack). Slack often still holds whatever was on those sectors before, including pieces of deleted files. That is why forensic tools search slack, and why ordinary backups that only copy live files miss it.

A rough estimate of wasted space, from the lectures:

**(cluster size / 2) × (number of files)**

The reasoning is statistical: the last cluster of a file is on average half empty.

**Lost clusters** are marked allocated in the file system but no file points at them. They show up after crashes and interrupted writes, and they can hold fragments of deleted or damaged files. CHKDSK, ScanDisk, and Linux `fsck` find them and can turn them into recoverable files.

### The Windows XP boot sequence

The course teaches the classic BIOS boot, not modern UEFI. Learn this sequence as given.

**Order:** POST → BIOS → MBR → boot sector → `ntldr` loads the kernel and drivers.

**Startup files** the lectures name: `ntldr`, `boot.ini`, `ntdetect.com`, `pagefile.sys`, `ntbootdd.sys`.

**Core system files** once the kernel is coming up: `ntoskrnl.exe` and `ntkrnlpa.exe` (kernel), `hal.dll` (hardware abstraction), `win32k.sys`, `ntdll.dll`, `kernel32.dll`, `advapi32.dll`, `user32.dll`, `gdi32.dll`.

These files normally live in the root of the system drive and in `System32`. If they are missing or out of order, the machine does not boot. A boot disk (or a forensic boot CD) can start a machine whose own operating system should not be allowed to run.

Why this belongs in forensics: booting the suspect operating system changes files, timestamps, and logs. Examiners boot from their own trusted media, or they do not boot the suspect disk at all.

**Check yourself**

- Where is the MBR, and what two things does it hold?
- Why can slack space contain another file’s data?
- Recite the XP boot order from power-on to `ntldr`.

---

## 5. What deletion actually does

This is one of the highest-value topics in the course. The slides scatter it across commands, recycle bins, and tool catalogs. The mechanism is simple.

### The mechanism

Files are located through an index:

- **FAT** uses the File Allocation Table.
- **NTFS** uses the Master File Table.

**Deleting a file removes or marks the index entry. It does not, by itself, erase the clusters.** Those clusters become free space and will be reused later. Until they are overwritten, the bytes are still there, and a recovery tool can rebuild the file if it can find the start and the chain of clusters.

**Deleting a partition** removes the partition-table entry. The file system underneath is still on the disk until something overwrites it.

**Formatting** usually writes a new file-system structure. It does not reliably wipe every sector. Treat “I formatted it” as “the map was replaced,” not “the data is gone.”

**Moving a file** depends on whether the destination is the same partition. On the same partition, the system mostly updates pointers. Across partitions, it copies the file and then deletes the original, so a deleted copy can remain on the source.

### Windows deletion commands

`DEL` and `ERASE` at the command prompt remove the directory entry. They do not scrub the clusters. Wildcards delete many files at once.

| Switch | Effect |
|---|---|
| `/p` | Ask before each file |
| `/f` | Delete even if read-only |
| `/s` | Include subdirectories |
| `/q` | Quiet; no prompts |
| `/a:` | Filter by attribute, for example `/a:h` for hidden, `/a:-r` for not read-only |
| `/?` | Help |

**Disk Cleanup** (Start → Programs → Accessories → System Tools → Disk Cleanup) removes categories such as temporary setup files, downloaded program files, temporary internet files, old CHKDSK files, Recycle Bin contents, temporary files unused for more than a week, offline files, and catalog files. It is a housekeeping tool. It does not guarantee every temporary file is gone, and files it deletes are often still recoverable until overwritten.

### The Recycle Bin

The Recycle Bin (Windows) and Trash (Mac) are a delay, not a wipe.

- Files deleted from Explorer usually go to the bin. **Shift+Delete**, command-line deletion, very large files, and files on network or removable drives often **bypass** the bin.
- Windows can restore a binned file to its original path. The lectures note that Mac Trash does not offer the same individual-and-bulk flexibility.
- Each partition has its own bin folder. Names you may see: `\RECYCLED`, `\RECYCLER`, `$Recycle.Bin`.
- Default size is about **10%** of the disk. When the bin is full, the oldest items are removed first (FIFO).
- On NTFS, each user has a separate bin identified by the account **SID**.
- **INFO** or **INFO2** files store the original path so a restore knows where the file belonged. If those files are damaged, one repair path in the lectures is to delete them and restart Windows so they are recreated.
- Replacements such as Diskeeper’s Recovery Bin or Sysinternals Fundelete exist because the normal bin misses Shift+Delete. You only need to know that the stock bin is incomplete.

### Linux deletion

`rm` removes the directory entry. There is no confirmation unless you ask for it. Data remains until overwritten. Desktop environments add their own trash folder; `rm` does not use it.

| `rm` switch | Effect |
|---|---|
| `-f` | Force, ignore missing files, no prompt |
| `-i` | Ask before each deletion |
| `-r` | Recursive, a directory and its contents |
| `-v` | Print each removal |

`shred` overwrites the file’s bytes, so ordinary undelete will not work. The lectures treat a shredded file as unrecoverable.

| `shred` switch | Effect |
|---|---|
| `-f` | Change permissions if needed so the overwrite can proceed |
| `-n` | Number of passes (the lecture default is 25) |
| `-s` | Limit how many bytes are overwritten |
| `-u` | Truncate and unlink the file after shredding |
| `-v` | Show each pass |
| `-x` | Do not round up to a whole block, so slack beyond the file size is left alone |
| `-z` | Final pass of zeros, which hides the fact that random overwrites happened |

### Actually destroying data

Recovery fails only when the clusters are overwritten or the media is physically ruined.

- **Wiping software** overwrites every sector, often with random data.
- A **degausser** destroys data on magnetic media with a strong magnetic field. It does nothing useful to optical discs or to flash in the way it does to a hard disk.
- **Physical destruction** (shredding, scoring the platters) is for the most sensitive material.
- Formatting is not enough.

### Recovery versus forensic recovery

Data recovery puts the file back for the user. Forensic recovery does that without writing onto the evidence disk, and it keeps the chain of custody.

Practical rules from the lectures:

- Act before the free space is reused.
- Install recovery software on a **different** drive. Installing it on the evidence drive can overwrite the very clusters you hope to read.
- Local deletes are often still in the Recycle Bin. Network deletes are often easier because servers have backups.
- Deleted files on a machine you still have are a common, ordinary IT problem. The forensic difference is that you image first and recover from the image.

Tools fall into tiers. Learn the tiers, not the full catalog.

| Tier | Examples from the lectures | Used for |
|---|---|---|
| Free / consumer | Recuva, PhotoRec, TestDisk | Accidental deletion at home |
| Commercial | EaseUS, Stellar, R-Studio, R-Undelete, GetDataBack | Broader format support, deeper scans |
| Forensic-grade | EnCase, FTK, Autopsy / The Sleuth Kit | Court use: write-blocking, carving, reporting, chain of custody |

Older names you may be asked to recognize: DOS `UNDELETE` (MS-DOS 5 through 6.22 only; unsafe on later Windows), Active@ UNDELETE and UNERASER, FTK Imager. Specialized repair utilities exist for Office files, zip archives, scratched CDs, and camera RAW images. The principle is always the same: the directory entry is gone or the header is damaged, and the tool hunts the remaining bytes.

**Check yourself**

- Why can `DEL` be undone, and why can `shred -u` not?
- Name two situations where a file never enters the Recycle Bin.
- What is the forensic reason to install a recovery tool on another disk?

---

## 6. Digital evidence and the order of volatility

**Digital evidence** is information stored or transmitted in digital form that can be used in court. It comes from computers, phones, servers, or networks.

The property that decides *when* you collect it is **volatility**: how long it survives a change in power or system state.

| | Volatile | Non-volatile |
|---|---|---|
| Persistence | Lost when power is removed | Remains after shutdown until overwritten |
| Examples | RAM, running processes, open network sockets, login sessions, clipboard, cache | Files on disks and SSDs, USB drives, optical discs, flash and EEPROM, registry hives on disk, logs, backups |
| How it is collected | **Live acquisition** while the system stays up | **Static acquisition**; the system can be powered off |
| If you wait | It can be gone in milliseconds to minutes, and it cannot be recreated | Lower immediate risk, but files can still be overwritten later |
| Repeatability | One-shot. You cannot go back and capture the same RAM | The disk can be re-imaged and the hashes compared |

### Order of volatility

Collect from most volatile to least volatile. Two versions appear in the lectures. They agree; the IEEE list is finer.

**Practical order used on scene**

1. RAM and running processes
2. Network connections and open ports
3. System cache and temporary data
4. Files on disk

**IEEE order quoted in the module**

1. Registers and cache
2. Routing tables, ARP cache, process tables, kernel statistics
3. System memory (RAM)
4. Temporary file systems
5. Data on disk

Capturing volatile data **changes the machine**, because a tool has to run. Document every command. Run those tools from a forensic CD or USB, not from the suspect’s hard disk, so you do not overwrite free space and you do not trust binaries the suspect may have replaced.

Commands named in the course:

- **Windows:** `netstat`, `nbtstat`, `ipconfig`, `pslist`, Task Manager
- **Unix:** `netstat`, `arp`, `ifconfig`, `ps`, and `dd` for a memory dump

### What “securing evidence” means

Preservation means protecting data and devices from alteration, loss, and damage. Retention continues after the trial if counsel says so; do not delete or return evidence because the hearing ended.

At the scene, the first responder secures the area, logs everyone who enters, and preserves both the screen (volatile) and the disks (non-volatile). Do not casually power a machine on or off. Some systems are booby-trapped to wipe themselves on shutdown or on boot. The safe pattern is: record what is on screen, capture volatile data if the machine is already on, image the disk, then shut down by a deliberate procedure.

When evidence is not being examined it stays in a locked place with limited access. Images live on restricted systems. An access log records name, date, time, and purpose. Tampering, even accidental, can make the evidence unusable in court.

**Check yourself**

- Sort these from first to last: disk files, ARP cache, RAM, registers.
- Why is a RAM capture not reproducible tomorrow?
- Why are live-response tools launched from your own media?

---

## 7. Law, authority, and admissibility

A technically perfect image is useless if the search was unlawful or the custody record is broken.

### Rules that keep a case defensible

- Examine the **duplicate**, not the original, whenever you can.
- Do not tamper with evidence. If anything changes, document it.
- Keep a chain of custody and handle items carefully.
- Do not go beyond your own knowledge. Say what you did not determine.
- Stay inside the scope of the warrant or the policy authority you were given. Evidence outside that scope can be excluded.

### Chain of custody

**Chain of custody** (the lectures also say chain of evidence) is the continuous record of who had the evidence, when, and for how long, from the scene to the courtroom.

A broken chain lets the other side argue that the item was swapped, altered, or contaminated. Courts can exclude it.

The handling pattern in the module:

1. Bag and seal it in a tamper-evident evidence bag.
2. Tag it with case number, date and time, and the collector’s name or badge.
3. Log it on the chain-of-evidence form: description, serial number, and other identifiers.
4. Sign the item in and out at every transfer.

Notes are taken in **ink**, not pencil, so they cannot be quietly rewritten. The evidence log and the final report are part of the same record.

### Search authority

**Before** anyone examines a computer they need a lawful basis. Skipping this wastes the technical work, because the result may be inadmissible.

A **search warrant** is a judge’s permission for police to search and seize. It requires:

- **probable cause** — facts that would make a reasonable person believe evidence of a crime will be found
- an **affidavit**, a sworn statement of those facts, notarized
- limits on **what** may be searched and **when**

Once signed, a digital-evidence first responder collects what the warrant covers.

**How to write the statement that supports a warrant**, from the lectures:

- number pages and lines so a passage can be cited
- date it
- identify yourself: name, role, employer
- add credentials if they show you are qualified
- tell the events in time order, with dates and times
- stick to facts; attach logs; do not speculate
- explain technical terms in plain language
- sign it

**Warrants are not the only lawful path.**

| Situation | Why a warrant may be unnecessary |
|---|---|
| Plain view | An officer who is lawfully present sees the evidence and may seize it. |
| Consent | The owner, or someone with real authority (an IT manager, a senior staff member, a parent), agrees to the search. |
| Private actor | Stricter constitutional rules bind the government. Evidence a private person collects is often still admissible, but only if it was preserved properly. |

Consent is only as good as the person’s authority. A roommate or a student generally cannot consent to a search of someone else’s computer. Users are not required to disclose passcodes unless a specific law compels them.

Private-sector searches still need forensic procedure. “We own the laptop” is authority to look. It is not permission to alter the disk and then call the result evidence.

### Statutes named in the course

Know what each one is *about*. The slides do not ask you to recite sections.

- **Privacy Protection Act (PPA)** — limits government searches of publishers and journalists’ materials.
- **Electronic Communications Privacy Act (ECPA)** — regulates access to electronic communications, including stored email and interception.
- **USA PATRIOT Act** — expanded investigative authorities after 2001; the course lists it among the laws that define whether a search is allowed.

Case law matters because older statutes get applied to new technology through court decisions. A search that exceeds the warrant can lose the evidence even if the files are genuine.

**Check yourself**

- Define probable cause and say what an affidavit is for.
- Give one warrant exception and the limit on who may consent.
- List the four physical steps of bagging and logging an exhibit.

---

## 8. The investigation from start to finish

The slides offer several numbered lists. They are not competing versions of one procedure. They describe different layers. Keep them separate.

### Layer A — Incident response (the organization)

This is the six-step lifecycle for handling an incident inside an organization.

1. **Preparation.** Roles, phone numbers, passwords, and tools are ready before anything happens. Policies are distributed. Alerts (SMS, email) work. Backups and restore steps are tested. Logging is already on. A **baseline** of normal behavior exists so later logs have something to be compared with.
2. **Detection.** Someone reviews logs and alerts: system, firewall, antivirus, and intrusion-detection logs. Early detection limits damage.
3. **Containment.** Isolate the affected systems (pull the network cable or otherwise cut access), limit who can reach them, and stop further damage and evidence loss. Shutting down is sometimes right and sometimes destroys volatile evidence. The decision depends on whether live data still matters.
4. **Eradication.** Remove the malware, the rogue account, or the person. Find the root cause. Reconfigure what was changed. For a policy violation this may be a disciplinary action rather than a technical cleanup.
5. **Recovery.** Restore data, software, and configuration from backups. Check that the threat is actually gone. Retest. The goal is a working business, not only a clean forensic image.
6. **Follow-up.** What happened, why, whether the response worked, what it cost, whether insurance applies, and what procedure changes. New prevention comes out of this step.

### Layer B — Forensic method (the evidence)

This is the scientific core, stated in the lectures as three stages:

1. **Acquisition.** Take the evidence and make an exact image without altering the original. The computer as a metal box is less important than the data on it.
2. **Authentication.** Show the image is a true copy, using hash values computed by the forensic tool.
3. **Analysis.** Examine files, logs, and deleted space for what the case needs. Do not open original files in ordinary applications. Opening a file can change the last-access time and other metadata.

Filtering is part of analysis. Most of a disk is irrelevant. The examiner is looking for the files that connect to the allegation.

### Layer C — People at the scene

Three roles, in the order control passes:

| Role | Duty |
|---|---|
| First responder | First qualified person on scene (officer, IT, security). Decide how big the scene is, set a perimeter, keep people out, list who is present, do not touch more than necessary, photograph what is on screen before it disappears. |
| Investigator | Takes the briefing and the evidence, sets the chain of command, decides whether the perimeter grows or shrinks, and owns the chain of custody. |
| Crime scene technician | The forensic specialist. Captures volatile data, images disks, shuts systems down correctly, seals and tags items, and stores them under lock until examination. |

### Layer D — A running computer, step by step

When the machine is on and will be examined, the lectures give this order:

1. Photograph every monitor, showing what is on screen.
2. Preserve volatile data.
3. Create a disk image before shutdown.
4. Verify the image with a hash or checksum.
5. Shut down using a deliberate, safe procedure.
6. Photograph the setup: front, back, and every cable.
7. Unplug, and label every cable and peripheral.
8. Use an antistatic wrist strap before handling drives.
9. Bag components in antistatic bags, away from heat and magnets.

### Layer E — A private company’s path through a case

Separate again from the technical steps, this is the organizational sequence in the first module:

1. Staff call counsel.
2. The investigator prepares a first-response procedure.
3. Evidence is seized and moved to the lab.
4. Bit-stream images are made and hashed (the lectures say MD5).
5. The image is examined and a report is written.
6. The client decides whether to press charges or proceed internally.
7. Sensitive client data is destroyed when retention is over.

The **Enron** example in the second module is there to illustrate the same point: once paper was shredded, the digital record and the care taken with it became the case. The **9/11** example is about retention: evidence can be kept and re-examined for years.

### Before you leave for the scene

Preparation is a requirement, not a courtesy. Trained people, a stocked lab, field kits, and reference material (manuals, known-good tool documentation) prevent improvisation. Documents that should already exist: chain-of-custody forms, property forms, and a contact list for counsel and specialists.

Interview complainants and witnesses. Their account decides where to look.

On site, assess the room (office, lab, public place), how long the work will disrupt the business, whether you need more people, and whether there is a biological or physical hazard as well as a digital one. Transport in antistatic packaging, keep magnetic media away from strong fields, and if a machine must stay on to preserve volatile data, plan the power.

**Check yourself**

- Place “hash the image” in Layer B, and “pull the network cable” in Layer A. Say why they are different acts.
- List the first responder’s jobs without mixing in the examiner’s analysis.
- Why is the photograph of the screen taken before shutdown?

---

## 9. Acquiring and duplicating evidence

Electronic evidence is fragile because **booting changes it**. Acquisition is designed around that fact.

### Bit-stream image versus backup

A **bit-stream image** (a forensic image) is a sector-by-sector copy of the storage, including deleted space, slack, and unallocated clusters.

A normal **backup** copies files the file system knows about. It skips slack, deleted files, and often some system areas. Backups are for restoring a business. They are not a forensic image.

Images are written to **sterile media**: storage that has been wiped, so you cannot mix remnants of an old case into this one.

### Write blockers

A **write blocker** sits between the evidence disk and the examiner’s computer and refuses write commands. Hardware blockers (the lectures mention FastBloc, and IDE/SCSI/SATA blockers generally) are preferred. The point is simple: analysis must not modify the original.

### A practical acquisition sequence

1. Record the machine: serial numbers, configuration, what is connected.
2. Photograph, then remove drives using antistatic precautions.
3. Boot the *examiner’s* environment (forensic boot disk), not the suspect operating system.
4. Image through a write blocker onto sterile media.
5. Validate with a hash or CRC.
6. Store the image on a restricted system and log the transfer.

If the hash of the image does not match the hash of the source, you do not analyze that image. You find out why and acquire again.

### Tools, by job

You will see long product lists in the slides. Group them:

**Software imaging and preview**

- **dd** — byte-for-byte copy on Linux and Unix, also ported to Windows. You choose input, block size, and how far to skip or seek. It can copy a whole disk, one partition, or only the MBR sector.
- **dcfldd / DC3DD** — forensic-oriented variants of `dd` named in the first module (the slides say DC3DD) that add hashing and logging.
- **FTK Imager** — preview, export, several image formats.
- **SafeBack** — DOS-based, sector by sector, historically accepted by the FBI.
- **DriveSpy, SnapBack DatArrest** — older imaging utilities cited in the module.
- **Mount Image Pro** — mounts EnCase and `dd` images so they can be examined as if they were disks.
- **EnCase** — full acquisition and analysis, treated as an industry standard in the course.

**Copying an image across a network**

**Netcat** moves a byte stream over the network in a simple client/server setup (TCP or UDP). Paired with `dd`, it is how an examiner images a machine onto a collection system without pulling the disk first. The lectures also note that netcat is powerful enough to be abused as a general remote-access tool, so its presence on a suspect machine can itself be a finding.

**Hardware imaging**

Portable labs exist for field work. Names in the course: ImageMASSter Solo-3 (the slide claims on the order of 4 GB per minute, with USB, IDE, SATA, and SCSI), RoadMASSter-3 (field imaging with a live view), and Disk Jockey IT (a very small write blocker). You do not need their marketing specs. You need to know that hardware imagers exist so a disk can be duplicated without a full lab bench.

**Ordinary duplication is a different task**

Tools such as R-Drive Image, Save-N-Sync, or a tape copier make operational backups and standard images for disaster recovery and mass deployment. Useful to a business, not a substitute for a hashed forensic image.

**Check yourself**

- Give one kind of data a backup misses and an image keeps.
- What does a write blocker refuse to do, and why?
- What do you do if the verification hash does not match?

---

## 10. Examination, analysis, and the report

Analysis happens on the authenticated image. The original stays sealed.

### What examiners look through

- **Logical layer** — files and folders as the file system presents them.
- **Physical layer** — sectors, unallocated space, slack, and deleted entries the directory no longer lists.
- **Keyword search** across both layers.
- **File carving** — recovering a file by its header and structure when the directory entry is gone. The course example is carving out deleted mail.
- **Timelines** — linking files and logs to when something happened.
- **Application and file-type analysis** — narrowing a huge disk to the programs that matter.
- **Ownership and possession** — who the account was, and whether the artifact shows the person knew the file was there.

File-system parsers, registry viewers, and cache viewers exist because Windows leaves a large amount of structured metadata outside ordinary documents. Data-carving tools and disk editors (WinHex is the one the course highlights, including a write-protected mode) are for the physical layer. **Evidor** is named as a keyword searcher that includes slack and writes an HTML or text report.

### The report

Document every step as you go, not at the end from memory.

- evidence log: identifier, time, description, custody changes
- results of each analysis step
- a final report a non-specialist can follow, with excerpts, logs, and findings
- retention or disposal only as policy or the court requires
- follow-up if there is an appeal or new evidence

The report is also where expert-witness standards show up: clear language, no claims beyond the examination, and enough detail that another examiner could repeat the work.

**Check yourself**

- Contrast a logical file listing with a carve from unallocated space.
- Name two metadata fields you can destroy by double-clicking a file on the original disk.

---

## 11. The lab and the toolkit

### Why a lab exists

A lab is a controlled place to recover, examine, and store electronic evidence. Demand grew with cybercrime. A lab that cannot prove its own procedures will undermine the cases it supports.

Planning, in the order the lectures care about:

- a defined mission and set of services
- alignment with legal rules and the organization’s needs
- room to grow as storage sizes and tools change

**Spaces:** an administrative area, an examination room, a network room, and a separate evidence store.

**Four “modes”** the slides name — business, technology, scientific, and artistic — mean only that a lab is at once a funded operation, a technical workshop, a methodical science bench, and a place where unfamiliar problems need judgment. Do not treat that list as four procedures.

### Conditions that protect evidence

| Concern | What the course expects |
|---|---|
| Fire | Wet-pipe, dry-pipe, pre-action, and gaseous suppression. The fires that matter here are Class A (ordinary combustibles) and Class C (electrical). Evidence storage should survive water. |
| Power | Forensic machines draw a lot of power. Examination benches get dedicated circuits. A UPS and backup power keep a live acquisition from dying mid-capture. |
| Network | An isolated forensic LAN. Physical separation is preferred to a VLAN, because a virtual boundary is easier to cross by mistake. Storage is redundant. |
| Environment | Extra cooling for dense equipment, humidity control, limits on electromagnetic interference, antistatic measures, and reasonable noise levels. |
| Security | Two-factor entry, cameras, access logs, and stricter rules for the evidence room than for the rest of the lab. |

### What goes in the kit

**Hardware:** target drives to receive images, write blockers, data cables, a forensic duplicator.

**Software:** EnCase, FTK, Autopsy; `dd` and DC3DD; MD5 and SHA-1 tools; carving tools; file-system and registry viewers; log collectors and report templates.

**Integrity tools do three related jobs:** hash calculators show a copy matches, bit-stream imagers make the copy, and file-integrity checkers show a file has not changed since it was hashed.

**Scene kit:** sterile media, labels, chain-of-custody forms, anti-static bags and mats, screwdrivers and Allen keys (including Torx), a camera, pens and markers, bootable USBs or CDs, an incident checklist, and write-protect devices.

**First-response media** are bootable USB or CD environments so the suspect operating system never starts, plus a checklist so the first person on scene does not invent a procedure.

### Tool hygiene

Keep tools current, test them before you depend on them, and record which tool and which version touched each item. A hash match produced by an untested, unknown build is a weak hash match.

**Check yourself**

- Why is the forensic LAN physically separate?
- Name three items you would be wrong to omit from a scene kit, and why.

---

## 12. Removable media, image files, and hidden data

### Removable and optical media

| Medium | Forensic point |
|---|---|
| Magnetic tape | Cheap sequential backup. You cannot jump to a file the way you can on a disk; you read through the tape. |
| Floppy disks | Obsolete, but still appear in old cases. Sizes were 8, 5.25, and 3.5 inch. |
| CD, DVD, Blu-ray | Optical. Not ruined by magnets, MRI, X-ray, or EMP. Data sits in a spiral read from the inside outward, unlike concentric hard-disk tracks. |
| Flash cards | SD, CompactFlash, Memory Stick, MMC, xD, SmartMedia. |
| USB flash drives | Rewritable, portable, sometimes disguised as ordinary objects. |

**Capacities named in the lectures**

- CD up to about 700 MB. CD-ROM is read-only, CD-R writes once, CD-RW rewrites.
- DVD roughly 4.5 to 17 GB depending on layers and sides. Formats include DVD-R, DVD+R, DVD-RW, DVD-RAM. They are not all interchangeable.
- Blu-ray about 25 GB single-layer and 50 GB dual-layer. HD-DVD is the failed rival format; you only need to recognize the name.

A scratch in the polycarbonate is often harmless. Damage to the thin lacquer layer on the label side can destroy the data. Optical recovery tools cited for damaged discs include CDRoller, IsoBuster, and similar utilities. Again, the category matters more than the brand list.

### Image files as evidence

Pictures are a large share of real examinations: photographs, scans, and graphics. Many are irrelevant. The work is separating the ones that matter.

**Kinds of image**

- **Raster** — a grid of pixels (a bitmap). Resolution is fixed. Scaling up does not add detail.
- **Vector** — shapes described mathematically. They scale cleanly. Logos and line drawings are typical.
- **Metafile** — both, in one file. Examples: EPS, CGM, WMF, EMF.

**Formats**

| Format | Remember |
|---|---|
| BMP | Windows bitmap, often 24-bit and uncompressed, so files are large. A 1280×960 photo can be about 3.5 MB as BMP and under 0.5 MB as JPEG. |
| GIF | 256 colors (8-bit), animation and transparency, suited to logos and flat graphics, not photographs. |
| PNG | 24-bit, lossless, transparency, interlacing, no animation. Open format. |
| JPEG | 24-bit, lossy. JPEG 2000 adds more modern compression and can support transparency. |
| TIFF | High quality, common in scanning and print, 8 to 32 bits, large files. |

**Compression**

- **Lossy** throws away data the eye tends not to miss. The file cannot be restored to the exact original. JPEG is the example. Techniques named: chroma subsampling, reducing colors, fractal compression, transform coding, vector quantization. Size cuts are often well past 50%.
- **Lossless** shrinks the file and can recreate every original byte. PNG, GIF, and TIFF commonly work this way. Techniques named: LZW (adaptive dictionary), deflate, Huffman / entropy coding, run-length encoding. Savings are often around half.

The forensic consequence: a lossy file is not a perfect record of the camera’s original samples, and a carved fragment must be matched to the right format or it will not open.

### Finding and repairing images

Images turn up on disks, in mail, and on optical media. Examiners often have to extract them from a full-disk image before anyone can view them.

A **header** stores width, height, color depth, and the format marker. If the header is missing, viewers call the file corrupt even when the picture data is intact. A hex editor such as WinHex can show the header. Replacing it with a header copied from a similar file sometimes repairs documents; with images, a wrong header can distort or still fail to open the picture.

Fragments in unallocated space and slack can be reassembled. Text fragments are easier to interpret than image fragments. Unknown extensions can be identified with a reference such as FILExt, then confirmed against the header bytes, which are harder to fake than the extension.

**Forensic image tools named in the course**

- **EnCase** shows graphics inside a bit-stream image, including deleted pictures, without changing the original.
- **GFE Stealth** works with SafeBack images, used by the FBI and IRS in the lectures’ account. It copies without compression and keeps an audit trail.
- **P2 eXplorer** views images from SafeBack, EnCase, and similar tools, and shows how the image was acquired.
- **ILook** provides a file-browser style view, hex view, extraction, and reports.

Lawyers and investigators often cannot be given a full forensic suite. Ordinary viewers (IrfanView, ACDSee, ThumbsPlus, XnView, and others) are used on **copies**. XnView can show **EXIF**: camera model, exposure, and the date the picture was taken, which is useful context and also something a later edit may change.

Viewers can alter evidence by updating access times or by editing. Copies that will be handed around should sit on write-once media (CD-R, DVD-R), so nobody saves over them.

### Steganography and copyright

**Steganography** hides a message in the unused or least noticeable bits of a carrier file (a document, image, audio file, or even an executable). The carrier looks normal. Reading the hidden data takes special software, and the payload may also be encrypted. A **watermark** is a related idea: a mark embedded in a file to show origin or ownership.

**Copyright** in the lectures is the creator’s right against unauthorized copying. In the United States it exists when the work is created; registration is what makes that right easier to prove in court. This is context for image cases, not a separate procedure.

**Check yourself**

- Why is a JPEG a weaker “exact copy of the scene” than a lossless scan?
- What does a broken header do, and what does it not do, to the picture data?
- Why are review copies of images put on CD-R rather than a shared folder?

---

## 13. Quick reference

### The five rules

1. Image first, analyze the image.
2. Hash the image and keep the hash.
3. Chain of custody for every transfer.
4. Most volatile evidence first.
5. Deletion removes the pointer, not the bytes, until something overwrites them.

### Order of volatility (IEEE)

1. Registers and cache
2. Routing tables, ARP cache, process table, kernel stats
3. RAM
4. Temporary file systems
5. Disk

### Incident response

Preparation → Detection → Containment → Eradication → Recovery → Follow-up

### Forensic method

Acquisition → Authentication → Analysis → Report

### On a live machine

Photograph screens → volatile data → image → verify hash → shut down → photograph cables → label → antistatic bag

### Disk capacity

Capacity = bytes/sector × sectors/track × tracks/surface × surfaces

1 KB = 1024 bytes, 1 MB = 1024², 1 GB = 1024³, 1 TB = 1024⁴

Average wasted space ≈ (cluster size / 2) × number of files

### Boot (Windows XP, as taught)

POST → BIOS → MBR (sector 0) → boot sector → `ntldr`

Startup files: `ntldr`, `boot.ini`, `ntdetect.com`, `pagefile.sys`, `ntbootdd.sys`

### File systems at a glance

FAT12 small floppies → FAT16 early disks → VFAT long names → FAT32 large disks (Windows creates only up to 32 GB) → NTFS (MFT, EFS, compression, hot-fix)

Linux: ext2 inodes, ext3 journal, ext4 large volumes. Mac: HFS Plus. Solaris: ZFS.

### Hashes and copies

| Tool or idea | Role |
|---|---|
| MD5, SHA-1 | Show an image matches the source |
| CRC | A simpler checksum used the same way in several slides |
| Write blocker | Stops writes to the original |
| `dd` | Byte-for-byte copy |
| Netcat + `dd` | Byte-for-byte copy over the network |
| Backup software | Not a forensic image |

### Deletion

| Action | Recoverable by undelete? |
|---|---|
| `DEL`, Explorer delete, `rm` | Usually, until overwritten |
| Recycle Bin / Trash | Yes, by design, until emptied |
| Shift+Delete, command line, network, removable | Skips the bin; data may still be in free space |
| Format | Often, because the bytes remain |
| `shred`, full-disk overwrite | No, for practical purposes |
| Degauss or shred the platter | No |

---

## 14. A short self-test

Answer without looking. If you can do these, you have the course rather than the slide order.

1. A colleague says, “We took a backup, so we have a forensic image.” What is missing from that backup?
2. Put these in collection order: a Word file on the desktop, the ARP cache, CPU registers, the contents of RAM.
3. A file is deleted with `DEL` and the machine stays on, writing new files. What are you racing against?
4. Why does NTFS slack space sometimes contain an older file?
5. Compute the capacity of a disk with 512-byte sectors, 64 sectors per track, 8,000 tracks per surface, and 2 surfaces. Give bytes and MB.
6. Who may consent to a search of an employee’s company laptop, and who may not consent to a search of a roommate’s personal laptop?
7. Name the three forensic stages and the one number you compare at the end of authentication.
8. An internal harassment case turns up files that look like child sexual abuse material. What changes about the investigation?
9. Why do you photograph the monitor before you pull power?
10. A JPEG and a PNG of the same picture are both on the disk. Which one can be restored to its exact stored bytes after compression, and why?

**Answers**

1. Deleted files, slack, and unallocated space. Also, typically, a hash taken against the original at the time of acquisition.
2. Registers, then ARP cache, then RAM, then the Word file.
3. Reuse of the freed clusters. New files can overwrite them.
4. The cluster is larger than the file. Bytes after the file’s real end were not cleared, so they still hold whatever used that cluster before.
5. 512 × 64 × 8,000 × 2 = 524,288,000 bytes = 524,288,000 / 1024² = 500 MB exactly.
6. The company, through its policy, banner consent, and authorized staff, may examine the company laptop. A roommate does not have authority over a computer they do not own.
7. Acquisition, authentication, analysis. You compare the hash (or checksum) of the image to the hash of the source.
8. It is now a criminal matter. Notify law enforcement. From that point, public-sector search rules and a formal chain of custody apply.
9. The screen is volatile. Power loss destroys whatever the display and RAM currently show.
10. The PNG. PNG compression is lossless. JPEG compression has already discarded data; decompressing it does not bring those samples back.
