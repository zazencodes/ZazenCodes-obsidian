---
date: 2024-07-14
tags:
    - code-snip
hubs:
    - "[[python]]"
---

# Sort iphone videos by date taken

requirements:

```
git+https://github.com/Zulko/moviepy
```

```python
from datetime import datetime
from pathlib import Path
import shutil
import moviepy.editor as mp

VALID_FILE_TYPES = ("mov", "mp4")

def get_capture_date(video_path):
    """
    Extracts the capture date from a video using MoviePy.

    Args:
        video_path: Path to the video file.

    Returns:
        A datetime object representing the capture date, or None if not found.
    """
    try:
        clip = mp.VideoFileClip(video_path)
        # Access metadata through clip.reader.info dictionary
        metadata = clip.reader.infos["metadata"]
        if 'creation_time' in metadata:
            return datetime.strptime(metadata['creation_time'], '%Y-%m-%dT%H:%M:%S.%fZ')
    except (IOError, KeyError):
        # Handle errors like file not found or missing metadata key
        pass
    return None


def move_and_rename(source_folder, target_folder):
    """
    Moves MOV files with extracted dates to a new folder, renaming with a sequence number.

    Args:
      source_folder: Path to the folder containing files.
      target_folder: Path to the target folder for sorted and renamed files.
    """
    os.makedirs(target_folder, exist_ok=True)  # Create target folder if it doesn't exist

    for filename in os.listdir(source_folder):
        if not any((filename.lower().endswith(ext) for ext in VALID_FILE_TYPES)):
            continue
        video_path = os.path.join(source_folder, filename)
        capture_date = get_capture_date(video_path)
        if capture_date:
            # Format date string (adjust format if needed)
            date_str = str(capture_date)
            # Create formatted filename with sequence number
            new_filename = f"{date_str}_{filename}"
            target_path = os.path.join(target_folder, new_filename)
            # Move and rename file
            print(f"mv {video_path} {target_path}")
            shutil.move(video_path, target_path)


def main(folder_path):
    source_folder = folder_path
    target_folder = Path(folder_path).with_stem("sorted")
    if target_folder.is_dir():
        raise ValueError(f"Target dir exists: {target_folder}. Exiting.")

    move_and_rename(source_folder, target_folder)


if __name__ == "__main__":
    import os
    import argparse

    parser = argparse.ArgumentParser()
    parser.add_argument("--folder-path", help="Folder to organize")
    args = parser.parse_args()

    main(args.folder_path)
```

