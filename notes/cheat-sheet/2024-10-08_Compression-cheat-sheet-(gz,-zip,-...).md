---
date: 2024-10-08
tags:
    - cheat-sheet
hubs:
    - "[[bash]]"
---

# Compression cheat sheet (gz, zip, ...)

## Gzip and Tar
- **Compress a file or directory**: `tar [--exclude='./dir_name/venv' --exclude='./dir_name/logs' ...] -czf archive.tar.gz dir_name`
- **Decompress a file or directory**: `tar -xzf archive.tar.gz`
- **View compressed file contents**: `tar -tzf archive.tar.gz`
- **Compress multiple files**: `tar -czf archive.tar.gz file1 file2`
- **Keep original files after compression**: `tar -czf archive.tar.gz --keep-old-files filename`

### Common tar flags
| Flag | Description |
|------|-------------|
| c | Create a new archive |
| x | Extract files from archive |
| z | Filter archive through gzip |
| f | Use archive file |
| v | Verbose output |
| t | List contents of archive |
| r | Append files to archive |
| u | Update files in archive |
| k | Keep existing files |
| p | Preserve permissions |

## Zip
- **Create a zip of a file**: `zip archive.zip file1 file2 ...`
- **Create a zip of a directory**: `zip -r archive.zip directory1`
- **Add files to an existing zip**: `zip archive.zip newfile`
- **Extract a zip file**: `unzip archive.zip`
- **Update files in a zip**: `zip -u archive.zip updatedfile`

### Preview without unzipping
- **List contents of a zip file**: `unzip -l archive.zip`
- **Head** `unzip -p yourfile.zip | head -10`
- **Get line count** `unzip -p yourfile.zip | wc -l`
