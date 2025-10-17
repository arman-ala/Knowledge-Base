# fdisk command

2025-10-11 23:08
Status: #DONE 
Tags: [[Linux]]

---
# Hey, Let's Master Disk Partitioning with fdisk: Your No-Nonsense Guide

Hey folks! If you're diving into Linux storage management, `fdisk` is that trusty old tool that's been around since the 1980s—kinda like the Swiss Army knife of partitioning. It's a command-line wizard that lets you create, tweak, and delete partitions on your disks interactively. These days, it handles everything from the classic MBR (Master Boot Record) to the modern GPT (GUID Partition Table), plus some legacy stuff like Sun, SGI, and BSD tables. Why bother with partitioning? Well, imagine slicing up a big pizza (your hard drive) into neat slices (partitions) so you can run multiple OSes on one disk, keep your system files separate from your memes for better security, or just organize storage without everything being one giant mess. It can boost performance too if you plan it right. But heads up: fdisk is powerful, so one wrong move could wipe your data. Always back up first!

In this guide, I'll walk you through it step by step, like we're chatting over coffee. We'll cover the basics, options, interactive mode, real-world ops, advanced tricks, and best practices. I'll throw in tons of examples with sample outputs and deep dives into what they mean—because seeing it in action makes all the difference.

## 1. Getting Started with fdisk and Why Partitioning Matters

Partitioning is basically carving up your physical disk into logical chunks. It's essential for:
- **Making room for new setups**: Like adding a fresh partition for a dual-boot Windows/Linux rig.
- **Multi-OS vibes**: Install Ubuntu next to Fedora without them stepping on each other's toes.
- **Security and organization**: Keep /home (your files) isolated from the root filesystem so if something goes wrong, it's not a total disaster.
- **Performance tweaks**: Align partitions for SSDs to avoid slowdowns.

Fdisk gives you hands-on control, but remember, it's not forgiving—changes are permanent once saved. Let's jump into the syntax.

## 2. How to Use fdisk: Syntax and Those Handy Options

### 2.1 The Basic Setup

At its core, fdisk works like this:
```bash
fdisk [options] device
```
Or for just listing stuff:
```bash
fdisk -l [device...]
```
You usually need `sudo` because messing with disks is root-level stuff. The "device" is something like `/dev/sda` (your main drive—be careful!).

### 2.2 All the Command-Line Options You Need to Know

Fdisk has a bunch of flags to customize its behavior. I'll explain each one, then give you an example with sample output and a deep breakdown. These are super useful for non-interactive tasks or tweaking how fdisk sees your hardware.

| Option | Description | Values |
|--------|-------------|--------|
| `-l` | Lists partition tables for all devices or just the ones you specify. Great for a quick inventory without changing anything. | N/A |
| `-b sectorsize` | Forces fdisk to assume a specific sector size for the disk. Modern drives might have 4K sectors, but you can override for compatibility. | 512, 1024, 2048, 4096 |
| `-c` | Sets compatibility mode: 'dos' mimics old DOS behavior (e.g., for legacy bootloaders), 'nondos' is the default for modern setups. | `dos`, `nondos` |
| `-u` | Changes how sizes are displayed in listings: 'sectors' for raw sector counts, 'cylinders' for old-school geometry (rarely used now). | `sectors`, `cylinders` |
| `-s partition` | Prints the size of a specific partition in 512-byte sectors. Quick way to check space without full listings. | Partition identifier |
| `-t type` | Limits fdisk to only support a specific partition table type, like 'dos' for MBR or 'gpt'. Useful for forcing a format. | Type identifier |
| `-w` | Controls wiping of disk signatures (old filesystem remnants) before starting: 'auto' wipes if needed, 'never' skips, 'always' forces it. | `auto`, `never`, `always` |
| `-W` | Like -w, but for wiping signatures on new partitions you create during the session. | `auto`, `never`, `always` |
| `-C cylinders` | Manually sets the number of cylinders (old CHS geometry). Deprecated—modern fdisk auto-detects, but handy for ancient hardware emulation. | Number |
| `-H heads` | Sets the number of heads per cylinder (again, CHS stuff). Deprecated for the same reason. | Number |
| `-S sectors` | Sets sectors per track (CHS). Deprecated, but can fix quirky old disks. | Number |

**Quick Tip**: Those last three (-C, -H, -S) are from the floppy disk era—don't sweat them for SSDs or HDDs today; fdisk handles topology automatically for better alignment and performance.

Now, let's see these in action with examples. I'll expand on the basics here.

### 2.3 Real-World Examples for Command-Line Options

**Listing All Disk Partitions (-l):**
```bash
sudo fdisk -l
```
**Sample Output:**
```
Disk /dev/sda: 30 GiB, 32212254720 bytes, 62914560 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x184931d5

Device     Boot   Start      End  Sectors Size Id Type
/dev/sda1  *       2048  2099199  2097152   1G 83 Linux
/dev/sda2       2099200 62914559 60815360  29G 8e Linux LVM

Disk /dev/sdb: 10 GiB, 10737418240 bytes, 20971520 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX

Device     Start      End  Sectors Size Type
/dev/sdb1   2048   1048575  1046528 511M Linux filesystem
/dev/sdb2 1048576  20971519 19922944 9.5G Linux LVM
```
**Deep Explanation:** This is your go-to for scouting all disks. It spits out details like total size, sector info (logical vs. physical—important for alignment), partition table type (dos/MBR here for sda, gpt for sdb), and per-partition breakdowns: boot flag (* for bootable on MBR), start/end sectors (for seeing gaps), total sectors (raw count), human-readable size, ID (hex code for type), and description. In this case, sda has a bootable 1G Linux partition and a big LVM one; sdb's GPT setup has a small filesystem and LVM chunk. Use this before any changes to avoid surprises—no writing happens, so it's safe.

**Specifying Sector Size (-b):**
```bash
sudo fdisk -b 4096 -l /dev/sde
```
**Sample Output:**
```
Disk /dev/sde: 10 GiB, 10737418240 bytes, 2621440 sectors
Units: sectors of 1 * 4096 = 4096 bytes
Sector size (logical/physical): 4096 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
Disklabel type: dos
Disk identifier: 0x3a7b2c9d

Device     Boot   Start      End  Sectors Size Id Type
/dev/sde1        1       256     256  1M 83 Linux
/dev/sde2       257    262143  261887 1G 83 Linux
```
**Deep Explanation:** Here, we're forcing a 4K (4096-byte) sector size on a drive that might default to 512 bytes logically. The output adjusts units to 4096-byte sectors, recalculates totals (e.g., fewer sectors overall), and shows I/O sizes matching the physical drive. This is crucial for advanced drives (like some enterprise SSDs) where mismatch causes inefficiency—misaligned reads/writes waste bandwidth. In this example, partitions are tiny for demo; notice how sizes scale down in sectors. Use -b when fdisk misdetects or for testing compatibility with tools expecting specific block sizes. Without it, you might get alignment warnings.

**Compatibility Mode (-c):**
```bash
sudo fdisk -c dos /dev/sda
```
**Sample Output (after entering interactive mode and 'p'):**
```
Disk /dev/sda: 30 GiB, 32212254720 bytes, 62914560 sectors
Units: cylinders of 16065 * 512 = 8225280 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x184931d5

Device     Boot Start       End   Blocks   Id  System
/dev/sda1  *    1         130    1044224+  83  Linux
/dev/sda2       131      3906  30407680   8e  Linux LVM
```
**Deep Explanation:** The -c dos flag switches to DOS compatibility, using cylinders (old geometry) instead of sectors for display. Output shows cylinder-based units (8225280 bytes per cylinder—legacy math from heads/sectors/track). Partitions list start/end in cylinders, blocks in KB. This mimics ancient tools for bootloader compatibility (e.g., old GRUB). In this run, we entered interactive mode to print ('p'); notice the + on blocks for partial cylinders. Use nondos (default) for modern accuracy—dos can confuse large drives >8GB. Deep down, it affects how fdisk interprets CHS limits, preventing overflow errors on legacy systems.

**Display Units (-u):**
```bash
sudo fdisk -u sectors -l /dev/sda
```
**Sample Output:**
```
Disk /dev/sda: 30 GiB, 32212254720 bytes, 62914560 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x184931d5

Device     Boot   Start      End  Sectors Size Id Type
/dev/sda1  *       2048  2099199  2097152   1G 83 Linux
/dev/sda2       2099200 62914559 60815360  29G 8e Linux LVM
```
**Deep Explanation:** -u sectors (default now) shows everything in sectors, making it precise for scripting or alignment checks. Compared to cylinders (via -u cylinders, which would use old units), this avoids geometry pitfalls on big drives. Here, start/end are raw sectors—e.g., sda1 from 2048 (aligned start) to 2099199. Sizes calculate as (end-start+1)*512 bytes. Use cylinders for legacy docs, but sectors for truth—it's why modern fdisk prefers it, reducing errors on >2TB GPT disks.

**Partition Size Report (-s):**
```bash
sudo fdisk -s /dev/sda1
```
**Sample Output:**
```
2097152
```
**Deep Explanation:** This spits out just the sector count (2097152) for /dev/sda1—no frills. Multiply by 512 for bytes: ~1GB. It's for scripts (e.g., if [ $(fdisk -s /dev/sda1) -gt 1000000 ]; then echo "Big enough"; fi). Deeply, it reads the partition table without loading the whole disk, fast for monitoring. If the partition doesn't exist, it errors—always verify with -l first.

**Limiting to Partition Type (-t):**
```bash
sudo fdisk -t dos /dev/sdc
```
**Sample Output (interactive 'p' after):**
```
Disk /dev/sdc: 20 GiB, 21474836480 bytes, 41943040 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x569c5370

Device     Boot Start      End  Sectors Size Id Type
```
**Deep Explanation:** -t dos forces MBR mode, rejecting GPT if present (errors if mismatch). Useful for ensuring compatibility—e.g., on a blank disk, it initializes as DOS. In interactive mode, 'o' creates DOS table. Output shows empty MBR setup. For GPT, use -t gpt. This prevents cross-format accidents, like trying GPT on old BIOS.

**Wiping Signatures (-w):**
```bash
sudo fdisk -w always /dev/sdd
```
**Sample Output (after 'p' in interactive):**
```
The device /dev/sdd contains Linux filesystem signatures. These are wiped before creating a new partition table.
Disk /dev/sdd: 5 GiB, 5368709120 bytes, 10485760 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x12345678

Device     Boot Start End Sectors Size Id Type
```
**Deep Explanation:** -w always aggressively wipes old signatures (e.g., ext4 headers) to avoid confusion. 'Auto' wipes only if needed; 'never' risks leftovers causing mount issues. Here, it cleared a Linux filesystem sig before new table. Deeply, signatures are magic bytes at sector 0—wiping ensures clean slate for repartitioning, preventing "partition contains signature" warnings. Use 'auto' for safety.

**Wiping New Partitions (-W):**
```bash
sudo fdisk -W auto /dev/sde  # Then create partition interactively with 'n'
```
**Sample Output (during creation):**
```
Created a new partition 1 of type 'Linux' and of size 500 MiB.
Partition #1 contains a vfat signature.
Do you want to remove the signature? [Y]es/[N]o?  # Auto would prompt or wipe based on flag
```
**Deep Explanation:** Similar to -w, but for fresh partitions. 'Auto' detects and offers to wipe (as shown); 'always' zaps without asking. In this case, it found VFAT (FAT32) remnants on new space and prompts. This avoids inheriting old data that could confuse mkfs later. Deeply, it's about zeroing superblocks—crucial for SSDs to trim unused space, improving write performance.

**Setting Cylinders (-C):**
```bash
sudo fdisk -C 1000 /dev/sdf -l
```
**Sample Output:**
```
Disk /dev/sdf: 500 MiB, 524288000 bytes, 1024000 sectors
Units: cylinders of 1000 * 512 = 512000 bytes  # Adjusted
Sector size (logical/physical): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xabcdef01

Device     Boot Start End Blocks Id System
/dev/sdf1        1   2   512 83 Linux  # Scaled to cylinders
```
**Deep Explanation:** Despite being deprecated, -C 1000 forces 1000 cylinders, tweaking units to 512000 bytes/cylinder. Output recalculates blocks accordingly—useful for emulating old hardware where BIOS limits cylinders. Here, a small disk shows partitions in this view. Modern drives ignore it for LBA (logical block addressing), but it helps debug legacy boot issues. Don't overuse; auto-detection is smarter for alignment.

**Setting Heads (-H):**
```bash
sudo fdisk -H 128 /dev/sdf -l
```
**Sample Output:**
```
Disk /dev/sdf: 500 MiB, 524288000 bytes, 1024000 sectors
Units: cylinders of 128 * 63 * 512 = 4128768 bytes  # With default S=63
Sector size (logical/physical): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xabcdef01

Device     Boot Start End Blocks Id System
/dev/sdf1        1   1   2064384 83 Linux  # Adjusted geometry
```
**Deep Explanation:** -H 128 sets heads to 128, combining with defaults (S=63 sectors/track) for cylinder size ~4MB. Output reflects this in blocks. Deprecated because drives report their own via ATA, but for virtual machines or old disks, it simulates limits (e.g., avoiding >1024-cylinder boot fails). In example, partition sizing shifts—use with -C/-S for full CHS control, but test thoroughly as it can misalign modern I/O.

**Setting Sectors per Track (-S):**
```bash
sudo fdisk -S 32 /dev/sdf -l
```
**Sample Output:**
```
Disk /dev/sdf: 500 MiB, 524288000 bytes, 1024000 sectors
Units: cylinders of 255 * 32 * 512 = 4177920 bytes  # With defaults H=255
Sector size (logical/physical): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xabcdef01

Device     Boot Start End Blocks Id System
/dev/sdf1        1   1   2088960 83 Linux  # Geometry tweak
```
**Deep Explanation:** -S 32 overrides sectors/track to 32, altering cylinder calc with defaults (H=255). Units become ~4MB/cylinder; blocks adjust. Like the others, it's for legacy CHS emulation—e.g., some old installers expect it. In this tiny disk example, it slightly changes block counts. Deeply, CHS was how BIOS addressed drives pre-LBA; now it's cosmetic, but vital for compatibility testing. Pair with -H/-C, but remember: misalignment from wrong values tanks SSD performance.

**Listing a Specific Disk (-l with device):**
```bash
sudo fdisk -l /dev/sda
```
**Sample Output:**
```
Disk /dev/sda: 30 GiB, 32212254720 bytes, 62914560 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x184931d5

Device     Boot   Start      End  Sectors Size Id Type
/dev/sda1  *       2048  2099199  2097152   1G 83 Linux
/dev/sda2       2099200 62914559 60815360  29G 8e Linux LVM
```
**Deep Explanation:** Zooms in on /dev/sda, same details as full -l but no clutter. Shows 30GB MBR disk with bootable 1G Linux (probably /boot) and 29G LVM (for root). Start at 2048 avoids old MBR areas. Use when multi-disk systems overwhelm you—quick for targeting one drive.

## 3. Diving into Interactive Mode: The Heart of fdisk

### 3.1 How to Jump In

Fire it up with:
```bash
sudo fdisk /dev/sda
```
You'll get a prompt like "Command (m for help):". Changes stay in RAM until you 'w' (write)—safe for experimenting. Exit with 'q' to bail without saving.

### 3.2 Your Interactive Command Cheat Sheet

Here's the full list, with descriptions and contexts. I'll add examples where needed in the practical section below.

| Command | Description | Usage Context |
|---------|-------------|---------------|
| `n` | Adds a new partition—primary, extended, or logical. | For fresh space allocation. |
| `d` | Deletes a partition by number. | Cleaning up unused space. |
| `p` | Prints the current table. | Always verify before/after changes. |
| `t` | Changes a partition's type (e.g., to swap or LVM). | Post-creation tweaks. |
| `a` | Toggles bootable flag (MBR only). | For boot partitions. |
| `u` | Switches display units (sectors/cylinders). | Viewing preferences. |
| `m` | Shows the help menu. | Newbies or reminders. |
| `w` | Writes changes and quits. | Final step—dangerous! |
| `q` | Quits without saving. | Abort mission. |
| `o` | Makes a blank MBR table. | Starting over with DOS. |
| `g` | Makes a blank GPT table. | Modern large-disk setup. |
| `l` | Lists partition types. | When picking 't' codes. |
| `v` | Verifies table for errors (e.g., overlaps). | Pre-write check. |

### 3.3 Handy Interactive Workflows

Start with 'm' for help—it's your lifeline. 'P' is for peeking. Examples coming up!

## 4. Hands-On Partition Management: Creating, Typing, Deleting

### 4.1 Making New Partitions

In interactive mode:
```bash
sudo fdisk /dev/sdb
Command (m for help): n
```
Pick primary/extended, number, first/last sector (use +size for ease, like +500M). Defaults align nicely.

**Best Practices:** Stick to defaults for starts (e.g., 2048 for alignment). Use {K,M,G} suffixes. Don't overlap!

### 4.2 Handling Partition Types

After 'n', use 't' to set type: 83 for Linux, 82 for swap, etc. 'L' lists all.

### 4.3 Deleting Safely

'd' then number. But whoa—data's still there until overwritten. Backup city!

## 5. Advanced Stuff: GPT, MBR, and Scripting

### 5.1 GPT vs. MBR: The Showdown

GPT: Unlimited partitions (up to 128), >2TB support, redundant tables—future-proof. MBR: 4 primaries max, 2TB limit, boot flags.

Create GPT: 'g'. MBR: 'o'. GPT hides boot flags since EFI handles booting.

### 5.2 Automating with Scripts

Echo commands: `echo -e "o\nw" | sudo fdisk /dev/sdb`. But for real automation, grab sfdisk or parted—they're script-friendly.

**Backup/Restore with sfdisk:**
```bash
sudo sfdisk -d /dev/sda > backup.dump
sudo sfdisk /dev/sda < backup.dump
```
Saves/restores layout—no data.

## 6. Best Practices and Fixing Goofs

### 6.1 Pro Tips

1. Backup with sfdisk -d always.
2. Triple-check device (lsblk helps).
3. 'Q' if unsure; 'w' commits.
4. partprobe or reboot post-change.
5. Let fdisk auto-align—it's smart.

### 6.2 Common Headaches

Busy device? partprobe. No boot flags on GPT? Normal. Signatures? Use -w.

## 7. Other Tools and Wrapping Up

### 7.1 Alternatives

- sfdisk: Scripting pro.
- parted: Resizing whiz.
- gdisk: GPT specialist.

### 7.2 Final Thoughts

Fdisk is a staple for Linux admins—consistent across distros, but wield it wisely. Pair with mkfs for filesystems. Practice on a USB!

Now, let's get practical with more examples...

# Practical Examples and Explanations (Expanded for Depth)

I've beefed this up to cover interactive commands too, with deep dives.

## Basic Disk Information Examples

**1. Listing All Disk Partitions** (Covered in 2.3 -l)

**2. Examining a Specific Disk** (Covered in 2.3 -l with device)

## Interactive Partition Management Examples

**3. Viewing Help Menu (m):**
```bash
sudo fdisk /dev/sdb
Command (m for help): m
```
**Sample Output:**
```
Command action
   a   toggle a bootable flag
   b   edit bsd disklabel
   c   toggle the dos compatibility flag
   d   delete a partition
   f   list free unpartitioned space
   g   create a new empty GPT partition table
   l   list known partition types
   m   print this menu
   n   add a new partition
   o   create a new empty DOS partition table
   p   print the partition table
   q   quit without saving changes
   s   create a new empty Sun disklabel
   t   change a partition type
   v   verify the partition table
   w   write table to disk and exit
   x   extra functionality (experts only)
```
**Deep Explanation:** 'M' is your starting point—lists all commands with short descriptions. No changes happen; it's read-only. In this output, you see basics like 'n' for new, 'p' for print, plus advanced like 'x' (for experts: set geometry, etc.). Use it every session to jog memory, especially on different fdisk versions (util-linux varies). It's why fdisk feels user-friendly despite being interactive—self-documenting!

**4. Printing Partition Table (p):** (Partial in others; full here)
```bash
sudo fdisk /dev/sda
Command (m for help): p
```
**Sample Output:** (As in example 1, but interactively—same details.)
**Deep Explanation:** Inside session, 'p' shows in-memory table (pre-write). Same format as -l: disk specs, then partitions. Deeply, it calculates sizes on-the-fly from sectors. Use after 'n' or 'd' to preview—e.g., spot overlaps before 'w'. If errors, 'v' complements it.

**5. Creating a New Partition (n):** (As in original 3)

**6. Changing Partition Type (t):** (As in original 4, with 'L' for list)
For 'l' (list types):
```bash
Command (m for help): l
```
**Sample Output:**
```
 0  Empty
 1  FAT12
 7  HPFS/NTFS/exFAT
 82 Linux swap / Solaris
 83 Linux
 8e Linux LVM
 ef EFI (FAT-12/16/32)
```
**Deep Explanation:** 'L' dumps hex codes and names—scrollable in terminal. Essential for 't': pick 82 for swap (enables mkswap). Deeply, types are 1-byte IDs in table; wrong one confuses kernel (e.g., mounting LVM as ext4 fails). Use for specialized setups like EFI (ef for /boot/efi).

**7. Deleting a Partition (d):** (As in original 5)

**8. Toggling Bootable Flag (a):**
```bash
sudo fdisk /dev/sda
Command (m for help): a
Partition number (1-4): 1
The bootable flag on partition 1 is enabled now.

Command (m for help): p
Device     Boot   Start      End  Sectors Size Id Type
/dev/sda1  *       2048  2099199  2097152   1G 83 Linux
```
**Deep Explanation:** 'A' flips the boot flag (* in 'p' output) for MBR—tells BIOS to boot from it. Only for primaries; GPT uses EFI. Here, we set it on sda1 (common for /boot). Deeply, it's bit 7 in the partition entry; without it, GRUB might not chainload. Toggle off for non-boot (e.g., data partitions). MBR-only—GPT ignores.

**9. Changing Display Units (u):**
```bash
sudo fdisk /dev/sda
Command (m for help): u
Changing display/entry units to sectors

Command (m for help): p
```
**Sample Output:** (Sectors view, as in -u example.)
**Deep Explanation:** 'U' toggles between sectors (default, precise) and cylinders (legacy). Post-change, 'p' refreshes. Deeply, cylinders use CHS calc (cylinders = sectors / (heads * sectors/track)); sectors avoid rounding errors on large disks. Use for consistency with scripts or old docs.

**10. Verify Table (v):**
```bash
sudo fdisk /dev/sdb  # After some changes
Command (m for help): v
```
**Sample Output:**
```
No errors detected.
Header version: 1.0
No warnings detected.
```
**Deep Explanation:** 'V' scans for issues: overlaps, bad sizes, unaligned starts. If errors: "Partition 1 overlaps..."—fix with 'd'/'n'. Deeply, it checks table integrity (e.g., sum of partitions < disk sectors, GPT checksums). Run before 'w' to catch mistakes; silent success means good to go. Great for complex multi-partition setups.

**11. Quit Without Saving (q):**
```bash
Command (m for help): q
```
**Sample Output:**
```
A write command is pending. Are you sure you want to quit? [Y]es/[N]o: y
```
**Deep Explanation:** 'Q' aborts if changes pending—prompts to confirm. No disk touch; memory clears. Deeply, it's a safety net—prevents accidental loss. Always use over Ctrl+C; explains why (e.g., "write pending").

**12. Write and Exit (w):**
```bash
Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```
**Deep Explanation:** 'W' commits to disk, syncs kernel (ioctl updates /proc/partitions). If busy, warns (use partprobe). Deeply, it writes MBR/GPT headers, updates kernel tables—partitions appear instantly if unmounted. Irreversible; post-w, run mkfs on new ones.

## Advanced Partitioning Examples

**13. Creating MBR Table (o):**
```bash
sudo fdisk /dev/sdc
Command (m for help): o
Created a new DOS disklabel with disk identifier 0x569c5370.
```
**Deep Explanation:** 'O' blanks MBR (DOS label), sets random ID. Erases old table—use on wiped disks. Deeply, writes 512-byte MBR with partition entries (4 slots) and boot code space. ID is random for uniqueness; GPT uses GUID. Follow with 'n' for partitions.

**14. Creating GPT Table (g):**

```bash
sudo fdisk /dev/sdc
```

**Interactive Session:**
```bash
Command (m for help): g
Created a new GPT disklabel (GUID: 8A6F3E8C-9D4F-4C7A-8B2A-7E3D1C9F5A2B)

Command (m for help): p
Disk /dev/sdc: 20 GiB, 21474836480 bytes, 41943040 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 8A6F3E8C-9D4F-4C7A-8B2A-7E3D1C9F5A2B
```

**Explanation:** This example demonstrates creating a new GPT (GUID Partition Table) partition table on a disk. The process involves:

1. Starting fdisk on the target device (/dev/sdc)
2. Using the 'g' command to create a new GPT partition table
3. Verifying the creation with the 'p' command The output shows that a new GPT disklabel has been created with a unique GUID (Globally Unique Identifier). GPT is the modern partitioning scheme that offers several advantages over the older MBR format:

- Support for disks larger than 2TB
- Support for up to 128 partitions by default
- Better redundancy with backup partition tables at both the beginning and end of the disk
- Unique GUIDs for each partition and disk
- No need for primary, extended, or logical partition distinctions This example shows a completely empty disk with just the GPT header information. After creating the partition table, you would proceed to create partitions as needed using the 'n' command.

**15. Creating Multiple Partitions (n multiple times):**
```bash
sudo fdisk /dev/sdc
```

**Interactive Session:**
```bash
Command (m for help): n
Partition number (1-128, default 1): 
First sector (2048-41943039, default 2048): 
Last sector, +sectors or +size{K,M,G,T,P} (2048-41943039, default 41943039): +1G

Created a new partition 1 of type 'Linux filesystem' and of size 1 GiB.

Command (m for help): t
Partition number (1-128): 1
Partition type (type L to list all types): 8200
Changed type of partition 'Linux filesystem' to 'Linux swap'.

Command (m for help): n
Partition number (2-128, default 2): 
First sector (2099200-41943039, default 2099200): 
Last sector, +sectors or +size{K,M,G,T,P} (2099200-41943039, default 41943039): +5G

Created a new partition 2 of type 'Linux filesystem' and of size 5 GiB.

Command (m for help): n
Partition number (3-128, default 3): 
First sector (12584960-41943039, default 12584960): 
Last sector, +sectors or +size{K,M,G,T,P} (12584960-41943039, default 41943039): 

Created a new partition 3 of type 'Linux filesystem' and of size 13.9 GiB.

Command (m for help): p
Disk /dev/sdc: 20 GiB, 21474836480 bytes, 41943040 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 8A6F3E8C-9D4F-4C7A-8B2A-7E3D1C9F5A2B

Device     Start      End  Sectors Size Type
/dev/sdc1   2048   2099199  2097152   1G Linux swap
/dev/sdc2 2099200  12584959 10485760   5G Linux filesystem
/dev/sdc3 12584960 41943039 29358080  14G Linux filesystem
```

**Explanation:** This example demonstrates creating multiple partitions with different types on a GPT-partitioned disk. The process includes:

1. Creating the first partition (/dev/sdc1) with a size of 1GB
2. Changing its type to "Linux swap" (type code 8200)
3. Creating a second partition (/dev/sdc2) with a size of 5GB
4. Creating a third partition (/dev/sdc3) that uses the remaining space (approximately 14GB)
5. Verifying the final partition layout with the 'p' command The output shows three partitions:

- /dev/sdc1: A 1GB Linux swap partition, typically used for virtual memory
- /dev/sdc2: A 5GB Linux filesystem partition, which could be used for /boot or other system directories
- /dev/sdc3: A 14GB Linux filesystem partition using the remaining space, suitable for the root filesystem or user data This example illustrates a typical partitioning scheme for a Linux system, with separate partitions for swap, system files, and user data. The GPT partitioning allows for up to 128 partitions, providing flexibility for complex partitioning schemes.
## Scripting and Automation Examples

**16. Basic Scripted Creation:** note limitations like no interactive Y/N)

```bash
echo -e "g\nn\n\n\n+5G\nt\n\n1\nw" | sudo fdisk /dev/sdd
```

**Sample Output:**
```text
Welcome to fdisk (util-linux 2.34).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table.
Created a new GPT disklabel.
Disk identifier: XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX

Created a new partition 1 of type 'Linux filesystem' and of size 5 GiB.
Partition #1 contains a ext4 signature.

Do you want to remove the signature? [Y]es/[N]o: n
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```

**Explanation:** This example demonstrates basic scripting of fdisk operations by piping commands to the interactive mode. The command sequence:

1. g - Creates a new GPT partition table
2. n - Starts creating a new partition
3. \n\n - Accepts default partition number and first sector
4. +5G - Sets the partition size to 5GB
5. t - Changes the partition type
6. \n\n1 - Selects the partition and sets type to 1 (EFI System)
7. w - Writes changes to disk and exits This approach allows for automation of partitioning tasks, but has limitations:

- It cannot handle interactive prompts that require specific responses (like the signature removal prompt in the output)
- It doesn't provide error handling
- It can be difficult to read and maintain For more complex scripting scenarios, consider using sfdisk or parted which are designed specifically for non-interactive partition management.

**17. Partition Backup/Restore:**

```bash
# Backup partition table
sudo sfdisk -d /dev/sda > sda_partition_table.dump
```

**Sample Output (in sda_partition_table.dump file):**
```text
# partition table of /dev/sda
label: dos
label-id: 0x184931d5
device: /dev/sda
unit: sectors

/dev/sda1 : start=2048, size=2097152, type=83, bootable
/dev/sda2 : start=2099200, size=60815360, type=8e
```

```bash
# Restore partition table
sudo sfdisk /dev/sda < sda_partition_table.dump
```

**Sample Output:**
```text
Checking that no-one is using this disk right now...
OK

Disk /dev/sda: 30 GiB, 32212254720 bytes, 62914560 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

>>> Script header accepted.
>>> Script header accepted.
>>> Script header accepted.
>>> Script header accepted.
>>> Created a new DOS disklabel with disk identifier 0x184931d5.
/dev/sda1: Created a new partition 1 of type 'Linux' and of size 1 GiB.
/dev/sda2: Created a new partition 2 of type 'Linux LVM' and of size 29 GiB.
/dev/sda2: Done.

New situation:
Disklabel type: dos
Disk identifier: 0x184931d5

Device     Boot   Start      End  Sectors Size Id Type
/dev/sda1  *       2048  2099199  2097152   1G 83 Linux
/dev/sda2       2099200 62914559 60815360  29G 8e Linux LVM

The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```
**Explanation:** This example demonstrates using sfdisk (a companion utility to fdisk) to backup and restore partition tables. The process involves:

1. Using sfdisk -d to dump the partition table of /dev/sda to a file
2. The dump file contains a human-readable representation of the partition table, including:    - Partition table type (label: dos)    - Disk identifier    - Unit of measurement (sectors)    - Details for each partition (start, size, type, and bootable flag)
3. Using sfdisk with input redirection to restore the partition table from the dump file
4. The restoration process recreates the partitions exactly as they were in the backup This technique is invaluable for:

- System recovery scenarios
- Cloning partition layouts to multiple similar systems
- Saving partition schemes before making experimental changes
- Documenting system configurations Note that this method only backs up the partition table structure, not the actual data within the partitions.
## Troubleshooting Examples

**18. Device Busy:**
```bash
sudo fdisk /dev/sda
```

**Interactive Session:**
```bash
Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read the partition table: WARNING: Re-reading the partition table failed with error 16: Device or resource busy.
The kernel still uses the old table. The new table will be used at the next reboot or after you run partprobe(8) or kpartx(8).
Syncing disks.
```

**Resolution:**
```bash
sudo partprobe
```

**Sample Output:**
```text
# No output indicates successful operation
```

**Verification:**
```bash
sudo fdisk -l /dev/sda
```

**Sample Output:**
```text
Disk /dev/sda: 30 GiB, 32212254720 bytes, 62914560 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x184931d5

Device     Boot   Start      End  Sectors Size Id Type
/dev/sda1  *       2048  2099199  2097152   1G 83 Linux
/dev/sda2       2099200 62914559 60815360  29G 8e Linux LVM
```
**Explanation:** This example demonstrates a common issue when modifying partition tables on disks with mounted filesystems. The error occurs because:

1. The kernel has active references to the old partition table
2. System services or mounted filesystems are using the device
3. The kernel cannot safely reload the partition table while it's in use

The solution is to use the partprobe command, which notifies the operating system kernel of partition table changes. This forces the kernel to reread the partition table without requiring a reboot. After running partprobe, the verification step with fdisk -l confirms that the system is now using the updated partition table.

This issue commonly occurs when:

- Modifying partitions on the system disk
- Working with disks that have mounted filesystems
- Changing partitions that are in use by LVM or other storage management systems

In production environments, it's often safer to schedule partition changes for maintenance windows when the system can be rebooted to ensure all services properly recognize the new partition layout.

**19. Fixing Alignment:**
```bash
sudo fdisk -l /dev/sde
```

**Sample Output (showing misalignment):**
```text
Disk /dev/sde: 10 GiB, 10737418240 bytes, 20971520 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
Disklabel type: dos
Disk identifier: 0x3a7b2c9d

Device     Boot   Start      End  Sectors Size Id Type
/dev/sde1        1024   1026047  1025024 501M 83 Linux
/dev/sde2     1026048  20971519 19945472 9.5G 83 Linux

Partition 1 does not start on physical sector boundary.
```

**Resolution:**
```bash
sudo fdisk /dev/sde
```

**Interactive Session:**
```bash
Command (m for help): d
Partition number (1-2): 1

Command (m for help): n
Partition type
   p   primary (1 primary, 0 extended, 3 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (1-4, default 1): 1
First sector (2048-20971519, default 2048): 
Last sector, +sectors or +size{K,M,G,T,P} (2048-20971519, default 20971519): +500M

Created a new partition 1 of type 'Linux' and of size 500 MiB.

Command (m for help): p
Disk /dev/sde: 10 GiB, 10737418240 bytes, 20971520 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
Disklabel type: dos
Disk identifier: 0x3a7b2c9d

Device     Boot   Start      End  Sectors Size Id Type
/dev/sde1        2048   1026047  1024000 500M 83 Linux
/dev/sde2     1026048  20971519 19945472 9.5G 83 Linux

Command (m for help): w
```
**Explanation:** This example demonstrates fixing a partition alignment issue. The problem is evident in the initial output:

- The physical sector size is 4096 bytes (4K), which is common for modern drives
- Partition 1 starts at sector 1024, which is not aligned to the 4096-byte physical sector boundary
- fdisk helpfully warns: "Partition 1 does not start on physical sector boundary"

Misaligned partitions can cause performance issues because:

- A single I/O operation might span multiple physical sectors
- Read/write operations may require additional overhead
- SSD wear can be unevenly distributed

The resolution involves:

1. Deleting the misaligned partition (/dev/sde1)
2. Creating a new partition with proper alignment
3. Accepting the default first sector (2048), which is aligned to the physical sector boundary
4. Specifying a similar size to the original partition
5. Verifying that the new partition starts at sector 2048, which is properly aligned

The final output shows that /dev/sde1 now starts at sector 2048, which is properly aligned to the 4096-byte physical sector boundary (2048 × 512 bytes = 1,048,576 bytes, which is divisible by 4096). Modern versions of fdisk automatically provide aligned defaults, but this example shows how to correct issues created by older tools or manual partitioning.