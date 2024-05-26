---
date: 2024-03-16
tags:
    - code-snip
hubs:
    - "[[python]]"
---

# Convert clip art image png to color

Replaces all non-zero colors in image with a single color
```python
from PIL import Image
import sys
from pathlib import Path


# NEW_COLOR = (230, 0, 0)  # This corresponds to #e60000
NEW_COLOR = (255, 255, 255)


def convert_image_to_red(input_path):
    output_path = f"{Path(input_path).stem}_recolor.png"

    # Load the image
    image = Image.open(input_path).convert("RGBA")
    width, height = image.size

    # Prepare the red color with the desired hex code, including a dummy alpha
    # Process every pixel
    for x in range(width):
        for y in range(height):
            r, g, b, alpha = image.getpixel((x, y))
            # If the pixel is not fully transparent, change its color to red while preserving its alpha value
            if alpha != 0:
                image.putpixel((x, y), NEW_COLOR + (alpha,))

    # Save the modified image
    image.save(output_path)
    print("Conversion complete. The output image is saved at:", output_path)


convert_image_to_red(sys.argv[1])

```
