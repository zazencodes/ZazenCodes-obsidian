---
date: 2024-04-10
tags:
    - code-snip
hubs:
    - "[[bash]]"
---

# ffmpeg convert mp4 to mp3

```bash
#!/bin/bash
if [ $# -eq 0 ]; then
    echo "Usage: $0 <input_file.mp4>"
    exit 1
fi
input_file="$1"
output_file="${input_file%.mp4}.mp3"
echo "Converting $input_file to $output_file at 192kbps bitrate"
ffmpeg -i $input_file -b:a 192K -vn $output_file
```
