# Turbowarp-Storage-System
# TurboWarp Virtual Filesystem

A fully functional 16MB FAT-style filesystem built entirely in TurboWarp Scratch. Stores, reads, deletes, renames, and defragments real plain text files — with authentic Latin-1 encoding and a working terminal UI.

> **Not a simulation.** Files that go in are real plain text bytes. Files that come out are identical, authentic plain text files openable by any text editor, interpreter, or OS.

---

## Features

- 16MB of byte-addressable virtual disk storage
- FAT-style file system with create, read, delete, rename, and defrag
- Authentic Latin-1 (extended ASCII) encoding — byte-for-byte identical to real plain text files
- Terminal UI with command parsing
- Upload real `.txt`, `.py`, `.js`, `.md`, `.json` (any plain text) files into the virtual disk
- Download files back out as real, working plain text files
- Case-sensitive filenames
- Free space tracking
- Defragmentation to reclaim deleted file space
- Format command with confirmation prompt
- Persistent storage — files survive project save/reload

---

## How It Works

### The Architecture

```
Terminal UI          ← what the user sees
File operations      ← FILE_READ, FILE_WRITE, FILE_DELETE, FILE_RENAME
FAT metadata         ← four parallel lists tracking file locations
Encode/Decode        ← ENCODE_STR, DECODE_BYTES
Read/Write Byte      ← the actual disk interface
PAD2                 ← zero-pads hex values to 2 digits
DISK                 ← one giant list, 131,072 items, 16MB total
```

### The Disk

The entire 16MB is stored in a single Scratch list called `DISK` with 131,072 items. Each item holds 128 bytes packed as a 256-character hex string:

```
item 1: "48656C6C6F..."  (128 bytes = 256 hex chars)
item 2: "0000000000..."
...
item 131072: "0000000000..."
```

Every character in every file lives somewhere in this list as a 2-digit hex value.

### The FAT (File Allocation Table)

Four parallel lists track every file:

| List | Contents |
|---|---|
| `FAT_NAME` | filename (e.g. `notes.txt`) |
| `FAT_START` | starting byte address on disk |
| `FAT_SIZE` | file size in bytes |
| `FAT_FLAGS` | `01` = active, `00` = deleted |

### Encoding

Every character is converted to its Latin-1 unicode value, then to a 2-digit hex string:

```
H → unicode 72 → hex 48
e → unicode 101 → hex 65
l → unicode 108 → hex 6C
...
```

Decoding reverses this exactly. The result is byte-for-byte identical to any real plain text file.

### Addressing

To find byte `B` on disk:

```
List item  = floor(B / 128) + 1
Char pos   = (B mod 128) × 2 + 1
Value      = letters charPos to charPos+1 of DISK[item]
```

---

## Terminal Commands

```
write [filename] [content]     Create a new file
read [filename]                Read a file's content
del [filename]                 Delete a file
rename [filename] [newname]    Rename a file
dir                            List all active files
free                           Show remaining bytes
defrag                         Reclaim deleted file space
download [filename]            Export file to your device
upload                         Import a plain text file from your device
format                         Wipe the entire disk (asks for confirmation)
help                           Show all commands
```

> **Note:** Content must be wrapped in square brackets:
> `write hello.txt [this is my content]`

---

## Supported File Types

Any plain text file works — the filesystem only cares about the bytes, not the extension:

```
.txt  .py  .js  .ts  .html  .css  .json
.xml  .md  .csv  .c  .cpp  .java  .rb
```

Binary files (`.png`, `.mp3`, `.zip`, etc.) are **not supported** — they contain headers and non-text bytes that will corrupt on encode/decode.

---

## Technical Specs

| Property | Value |
|---|---|
| Total capacity | 16,777,216 bytes (16MB) |
| List items | 131,072 |
| Bytes per item | 128 |
| Hex chars per item | 256 |
| Character encoding | Latin-1 (Unicode code points 00–FF) |
| FAT structure | 4 parallel lists |
| Deletion method | Flag flip (like real FAT) |
| Addressing | Byte-addressable |
| Capacity variable | `BYTES_PER_ITEM` (change to scale storage) |

### Scaling Storage

Changing `BYTES_PER_ITEM` scales the entire disk automatically:

| BYTES_PER_ITEM | Storage |
|---|---|
| 8 | 1MB |
| 16 | 2MB |
| 32 | 4MB |
| 64 | 8MB |
| 128 | 16MB (default) |
| 256 | 32MB |

---

## Required TurboWarp Extensions

| Extension | Author | Used For |
|---|---|---|
| Text | CST1229, BludIsAnLemon, Man-o-Valor | `unicode of`, `unicode as letter`, `letters X to Y`, `is identical to` |
| Base | TrueFantom | Hex/decimal/binary conversion |
| List Tools | LilyMakesThings | Bulk FAT list operations |
| Files | — | Upload/download real files |

---

## How To Run

1. Open [turbowarp.org](https://turbowarp.org)
2. Load the `.sb3` project file
3. Click the green flag
4. If first run, type `format` and confirm to initialize the disk
5. Type `help` to see all commands

---

## Limitations

- 16MB total combined storage for all files
- No folders or directories — flat filesystem only
- No file overwrite — delete then rewrite to update a file
- Deleted space not reclaimed until `defrag` is run
- Upload does not auto-detect filename — you will be prompted to enter one
- Binary files not supported
- Performance scales with file size due to Scratch's string operations

---

## Fun Facts

- The entire project file is only ~22KB — 750x smaller than its own storage capacity
- 16MB can hold approximately 329 Bee Movie scripts
- Deleted files are not wiped — just flagged, exactly like real FAT filesystems
- The disk is a single Scratch list pretending to be a storage medium
- Plain text files downloaded from this filesystem are indistinguishable from files created by any real text editor
