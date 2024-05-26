---
date: 2024-03-22
tags:
    - code-snip
hubs:
    - "[[python]]"
---

# Convert PNG to transparent (with thresholds)

```python
from PIL import Image
import argparse
from pathlib import Path

"""
e.g.

> pyv convert_white_png_to_transparent.py ~/Downloads/excl.webp --threshold 200
"""


def convert_to_transparent(input_image_path, threshold):
    output_image_path = f"{Path(input_image_path).stem}_transparent.png"

    # Open the image
    image = Image.open(input_image_path)

    # Convert image to RGBA if it's not already
    image = image.convert("RGBA")

    # Iterate through each pixel
    data = image.getdata()
    new_data = []

    for item in data:
        # Check if pixel is close to white or a shade of grey
        if all(abs(channel - 255) <= threshold for channel in item[:3]):
            new_data.append((255, 255, 255, 0))  # Convert pixel to transparent
        else:
            new_data.append(item)

    # Update image with new pixel data
    image.putdata(new_data)

    # Save the image with transparent pixels
    image.save(output_image_path, format="PNG")

    print(f"Saved result: {output_image_path}")


def main():
    parser = argparse.ArgumentParser(
        description="Convert pixels close to white or a shade of grey to transparent"
    )
    parser.add_argument("input", help="Input image file path")
    parser.add_argument(
        "--threshold",
        type=int,
        default=10,
        help="Threshold value to determine how close a pixel is to white or grey (default: 100)",
    )
    args = parser.parse_args()

    convert_to_transparent(args.input, args.threshold)


if __name__ == "__main__":
    main()

```

