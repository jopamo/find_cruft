# find_cruft

This script scans a given set of directories to find files that are *not* owned by Gentoo’s package manager (Portage). Optionally, it can delete these “unowned” (or “cruft”) files. Use this tool cautiously, as removing unowned files can sometimes break manually installed applications or custom system configurations.

---

## Table of Contents

1. [Overview](#overview)  
2. [Usage](#usage)  
3. [Options](#options)  
4. [Examples](#examples)  
5. [How It Works](#how-it-works)  
6. [Important Notes](#important-notes)  

---

## Overview

- **Purpose:**  
  - Identify files or directories on a Gentoo system that *are not* tracked by Portage.
  - Optionally remove them if they are indeed deemed unnecessary.

- **Key Features:**  
  - **Root Path Support**: You can specify a different root (useful for chroot setups).  
  - **Ignore List**: Certain system-critical files (like `/etc/passwd`) are always ignored.  
  - **Deletion**: A `--delete` flag that physically removes unowned files and directories.

- **Portage Integration:**  
  - The script looks into `/var/db/pkg` (or an alternate root’s `var/db/pkg`) to determine what files your system package manager already “knows.” Anything else may be considered “cruft.”

---

## Usage

```bash
find_cruft [options] [paths...]
```

1. Specify one or more directories to scan.
2. If you do not provide directories, the script shows usage info.
3. When run without the `--delete` option, it only **lists** potential cruft.

---

## Options

| Option              | Description                                                                                       |
|---------------------|---------------------------------------------------------------------------------------------------|
| `-h, --help`        | Display brief help text and exit.                                                                 |
| `-?, --man`         | Show the full manual page (extended documentation).                                              |
| `-V, --version`     | Print the current version number (`1.0`) and exit.                                               |
| `-v, --verbose`     | Increase verbosity. Can be used multiple times (e.g., `-vv`) for more detailed output.           |
| `-d, --delete`      | Physically remove discovered cruft files/directories.                                            |
| `-o, --output FILE` | Append output to `FILE`. Creates `FILE` if it doesn’t exist.                                     |
| `-O, --Output FILE` | Overwrite `FILE` instead of appending.                                                           |
| `-r, --root DIR`    | Specify a custom root directory (useful for scanning chroot environments).                       |

---

## Examples

1. **List unowned files in `/home` and `/usr/local`:**  
   ```bash
   ./find_cruft /home /usr/local
   ```
   This shows a list of files that do not belong to any installed package but **does not** delete anything.

2. **Delete any discovered cruft in `/srv`:**  
   ```bash
   ./find_cruft --delete /srv
   ```
   Files and directories that aren’t recognized by Portage get removed from `/srv` (recursively, if it’s a directory).

3. **Write output to a file instead of the screen:**  
   ```bash
   ./find_cruft -o unowned_files.log /opt
   ```
   This appends all unowned file paths under `/opt` to `unowned_files.log`.

4. **Overwrite an existing file with the script output:**  
   ```bash
   ./find_cruft -O new_cruft_list.log /usr/local
   ```
   This overwrites `new_cruft_list.log` with any cruft discovered in `/usr/local`.

5. **Scan a chroot environment at `/mnt/gentoo`:**  
   ```bash
   ./find_cruft -r /mnt/gentoo /etc /var
   ```
   - The script interprets `/etc` and `/var` as `/mnt/gentoo/etc` and `/mnt/gentoo/var`.  
   - Any “unowned” files it finds will be relative to that chroot system’s package database.

---

## How It Works

1. **Gather Portage-owned paths:**  
   - The script reads each package’s `CONTENTS` file in `/var/db/pkg` (or `$root/var/db/pkg`) to build an in-memory list of known/owned files.

2. **Traverse the specified directories:**  
   - It uses `File::Find` to walk through every file, directory, and symlink under the specified paths.

3. **Check against ignore list and known paths:**  
   - Certain high-level system paths (e.g. `/etc/passwd`) are always skipped.
   - If the path is found in Portage’s known list, it’s skipped as well.

4. **Decide what to do with unowned paths:**  
   - If `--delete` is *not* specified, the script just lists them.
   - If `--delete` *is* specified, the script attempts to remove them.  
     - Files and symlinks are unlinked.
     - Directories are removed with `remove_tree`, which deletes contents recursively.

---

## Important Notes

- **System Safety:**  
  Deleting files that are not Portage-owned does not guarantee they are unnecessary. Some software or manual customizations are intentionally installed outside of Portage. Review carefully before using `--delete`.
  
- **Verbosity:**  
  - `-v` (once) shows minimal extra info.  
  - `-vv` or more can be used for progressively more detailed logging.

- **Root vs. Absolute Paths:**  
  If you specify `--root /some/path`, an absolute directory like `/etc` in your arguments becomes `/some/path/etc` internally. This helps avoid using chroot directly, but be mindful that absolute symlinks inside that filesystem might still point outside it.

- **Permissions:**  
  If you run the script as a non-root user, certain directories will be skipped due to lack of read or write permissions.

- **Logs vs. Stderr:**  
  If an output file is specified, discovered cruft is printed there. Errors and warnings generally still appear on `stderr`.

---

## Acknowledgments

**Original Concept & Inspiration:** This script is inspired by prior work by Martin VE<auml>th and other Gentoo contributors who provided the original script.
