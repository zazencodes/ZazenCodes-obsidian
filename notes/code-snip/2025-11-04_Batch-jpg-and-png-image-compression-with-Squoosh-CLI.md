---
date: 2025-11-04
tags:
    - code-snip
hubs:
    - "[[bash]]"
---

# Batch jpg and png image compression with Squoosh CLI

Convert all images in `img` dir to web format with max size 1600px (height/width) and output to `img-web`.

```bash
#!/usr/bin/env bash
# Compress & resize iPhone JPEG/PNG images for web using Squoosh CLI.
# - Caps BOTH width & height at 1600px without upscaling (preserves aspect ratio)
# - Retains file type (JPEG -> mozjpeg, PNG -> oxipng)
# - Alphabetical processing, verbose logs, fail-fast on any error

set -euo pipefail

# Ensure we're running under bash even if invoked via sh/zsh
if [[ -z "${BASH_VERSION:-}" ]]; then
  exec /usr/bin/env bash "$0" "$@"
fi

INPUT_DIR="img"
OUTPUT_DIR="img-web"
MAX=1600

# --- helpers ---------------------------------------------------------------

fail() { echo "ERROR: $*" >&2; exit 1; }

need() {
  command -v "$1" >/dev/null 2>&1 || fail "Missing dependency: $1"
}

dim() {
  # Echo "width height" for a given file using identify (ImageMagick) or sips (macOS)
  local f="$1"
  if command -v identify >/dev/null 2>&1; then
    identify -format "%w %h" "$f"
  elif command -v sips >/dev/null 2>&1; then
    # sips output has labels; extract numbers
    local w h
    w=$(sips -g pixelWidth  "$f" 2>/dev/null | awk '/pixelWidth/  {print $2}')
    h=$(sips -g pixelHeight "$f" 2>/dev/null | awk '/pixelHeight/ {print $2}')
    echo "$w $h"
  else
    fail "Need either 'identify' (ImageMagick) or 'sips' (macOS) to read image dimensions."
  fi
}

# --- preflight -------------------------------------------------------------

[[ -d "$INPUT_DIR"  ]] || fail "Missing input dir: $INPUT_DIR"
[[ -d "$OUTPUT_DIR" ]] || fail "Missing output dir: $OUTPUT_DIR"

#!/usr/bin/env bash
need rsync
# Fast-fail if neither identify nor sips is present for dimension reads
if ! command -v identify >/dev/null 2>&1 && ! command -v sips >/dev/null 2>&1; then
  fail "Need either 'identify' (ImageMagick) or 'sips' (macOS) to read image dimensions."
fi

# Prefer ImageMagick for processing; fallback to sips if not available
HAS_MAGICK=false
if command -v magick >/dev/null 2>&1; then
  HAS_MAGICK=true
elif command -v convert >/dev/null 2>&1; then
  # Older IM installs expose 'convert'; we can call via 'magick' alias
  HAS_MAGICK=true
fi

echo "⇒ Starting compression"
echo "   Input : $INPUT_DIR"
echo "   Output: $OUTPUT_DIR"
echo "   Max side: ${MAX}px"
echo

# Build a sorted list (alphabetical) of JPEG/PNG files (flat folder)
files=()
# Collect with NUL delim to preserve spaces and avoid silent exits
while IFS= read -r -d '' f; do
  files+=("$f")
done < <(
  find "$INPUT_DIR" -maxdepth 1 -type f \
    \( -iname '*.jpg' -o -iname '*.jpeg' -o -iname '*.png' \) -print0
)

# Sort the list for stable processing order
if [[ ${#files[@]} -gt 0 ]]; then
  sorted=()
  while IFS= read -r line; do
    sorted+=("$line")
  done < <(printf '%s\n' "${files[@]}" | LC_ALL=C sort)
  files=("${sorted[@]}")
fi

echo "Found ${#files[@]} file(s) to process."

if [[ ${#files[@]} -eq 0 ]]; then
  echo "No JPEG/PNG files found in $INPUT_DIR; nothing to do."
  exit 0
fi

if [[ "$HAS_MAGICK" == false ]]; then
  echo "ImageMagick not found; will use sips for resizing. For best compression, install ImageMagick (and oxipng for PNG)."
fi

# --- main loop -------------------------------------------------------------

for f in "${files[@]}"; do
  # figure out extension (case-insensitive)
  ext="${f##*.}"
  ext_lc=$(printf '%s' "$ext" | tr '[:upper:]' '[:lower:]')

  # pick type to RETAIN the original type
  case "$ext_lc" in
    jpg|jpeg) img_type="jpeg" ;;
    png)      img_type="png" ;;
    *)        continue ;;
  esac

  base="$(basename "$f")"
  echo "→ Probing: $base"

  # get dimensions (robust to failures; do not exit the whole script)
  dim_out=""
  if ! dim_out=$(dim "$f" 2>/dev/null); then
    echo "! Skipping unreadable image (dim fail): $base"
    continue
  fi
  w="${dim_out%% *}"
  h="${dim_out#* }"
  if [[ -z "$w" || -z "$h" || ! "$w" =~ ^[0-9]+$ || ! "$h" =~ ^[0-9]+$ ]]; then
    echo "! Skipping image with invalid dimensions: $base"
    continue
  fi

  resize_arg=""
  if (( w > MAX || h > MAX )); then
    if (( w >= h )); then
      # landscape or square: cap width
      resize_arg='{"width":'"$MAX"',"method":"lanczos3","premultiply":true,"linearRGB":true}'
      resize_note="width→${MAX}"
    else
      # portrait: cap height
      resize_arg='{"height":'"$MAX"',"method":"lanczos3","premultiply":true,"linearRGB":true}'
      resize_note="height→${MAX}"
    fi
  else
    resize_note="no-resize"
  fi

  echo "• Processing: $base  (${w}×${h}, ${resize_note})"

  out="$OUTPUT_DIR/$base"

  # Compute resize geometry for IM/sips
  resize_geom=""
  if (( w > MAX || h > MAX )); then
    if (( w >= h )); then
      resize_geom="${MAX}x"
    else
      resize_geom="x${MAX}"
    fi
  fi

  if [[ "$HAS_MAGICK" == true ]]; then
    # Use ImageMagick for best quality/size
    if [[ "$img_type" == "jpeg" ]]; then
      if [[ -n "$resize_geom" ]]; then
        if command -v magick >/dev/null 2>&1; then
          magick "$f" -auto-orient -strip -colorspace sRGB -filter Lanczos -resize "$resize_geom" -sampling-factor 4:2:0 -interlace Plane -quality 82 "$out"
        else
          convert "$f" -auto-orient -strip -colorspace sRGB -filter Lanczos -resize "$resize_geom" -sampling-factor 4:2:0 -interlace Plane -quality 82 "$out"
        fi
      else
        if command -v magick >/dev/null 2>&1; then
          magick "$f" -auto-orient -strip -colorspace sRGB -sampling-factor 4:2:0 -interlace Plane -quality 82 "$out"
        else
          convert "$f" -auto-orient -strip -colorspace sRGB -sampling-factor 4:2:0 -interlace Plane -quality 82 "$out"
        fi
      fi
    else
      # PNG path: resize via IM, then optionally run oxipng for lossless optimization
      tmp="$OUTPUT_DIR/.tmp_$base"
      if [[ -n "$resize_geom" ]]; then
        if command -v magick >/dev/null 2>&1; then
          magick "$f" -auto-orient -strip -colorspace sRGB -filter Lanczos -resize "$resize_geom" PNG24:"$tmp"
        else
          convert "$f" -auto-orient -strip -colorspace sRGB -filter Lanczos -resize "$resize_geom" PNG24:"$tmp"
        fi
      else
        if command -v magick >/dev/null 2>&1; then
          magick "$f" -auto-orient -strip -colorspace sRGB PNG24:"$tmp"
        else
          convert "$f" -auto-orient -strip -colorspace sRGB PNG24:"$tmp"
        fi
      fi
      if command -v oxipng >/dev/null 2>&1; then
        oxipng -o4 --strip all -q -out "$out" "$tmp" && rm -f "$tmp"
      else
        mv -f "$tmp" "$out"
      fi
    fi
  else
    # Fallback to sips (macOS built-in). Limited compression; focuses on resizing.
    if [[ -n "$resize_geom" ]]; then
      if (( w >= h )); then
        sips --setProperty formatOptions high --resampleWidth "$MAX" "$f" --out "$out" >/dev/null
      else
        sips --setProperty formatOptions high --resampleHeight "$MAX" "$f" --out "$out" >/dev/null
      fi
    else
      # No resize needed; copy file as-is to output
      rsync -a "$f" "$out"
    fi
    # If PNG and oxipng is available, try to optimize
    if [[ "$img_type" == "png" ]] && command -v oxipng >/dev/null 2>&1; then
      oxipng -o4 --strip all -q "$out" >/dev/null || true
    fi
  fi
done

echo
echo "✓ All done. Output in: $OUTPUT_DIR"

```
