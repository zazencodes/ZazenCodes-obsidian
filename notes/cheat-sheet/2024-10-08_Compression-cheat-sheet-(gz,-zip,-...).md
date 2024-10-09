---
date: 2024-10-08
tags:
    - cheat-sheet
hubs:
    - "[[bash]]"
---

# Compression cheat sheet (gz, zip, ...)

## Gzip and Tar
- **Compress a file or directory**: `tar [--exclude='./dir_name/venv' --exclude='./dir_name/logs' ...] -czf archive.tar.gz dir_name
- **Decompress a file or directory**: `tar -xzf archive.tar.gz`
- **View compressed file contents**: `tar -tzf archive.tar.gz`
- **Compress multiple files**: `tar -czf archive.tar.gz file1 file2`
- **Keep original files after compression**: `tar -czf archive.tar.gz --keep-old-files filename`

## Zip
- **Create a zip of a file**: `zip archive.zip file1 file2 ...`
- **Create a zip of a directory**: `zip -r archive.zip directory1`
- **Add files to an existing zip**: `zip archive.zip newfile`
- **Extract a zip file**: `unzip archive.zip`
- **List contents of a zip file**: `unzip -l archive.zip`
- **Update files in a zip**: `zip -u archive.zip updatedfile`

