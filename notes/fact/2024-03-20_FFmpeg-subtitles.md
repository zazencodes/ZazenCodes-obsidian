---
date: 2024-03-20
tags:
    - fact
hubs:
    - "[[ffmpeg]]"
urls:
    - https://www.bannerbear.com/blog/how-to-add-subtitles-to-a-video-with-ffmpeg-5-different-styles/
    - http://www.tcax.org/docs/ass-specs.htm
    - https://www.reddit.com/r/ffmpeg/comments/vit59g/ffmpeg_hardsubbing_using_force_style_fontname/
    - https://fontdrop.info/
    - https://stackoverflow.com/questions/57869367/ffmpeg-subtitles-alignment-and-position
---

# FFmpeg subtitles

## Styles

Convert from SRT to ASS
```bash
ffmpeg -i subtitle.srt subtitle.ass
```

YouTube style (with background)
```bash
ffmpeg -i input.mp4 -vf "subtitles=subtitle.srt:force_style='Fontname=Roboto,OutlineColour=&H40000000,BorderStyle=3'" output.mp4
```

Netflix style (with shadow)
```bash
ffmpeg -i input.mp4 -vf "subtitles=subtitle.srt:force_style='Fontname=Consolas,BackColour=&H80000000,Spacing=0.2,Outline=0,Shadow=0.75'" output.mp4
```
## Fonts

[Use custom fonts](https://www.reddit.com/r/ffmpeg/comments/vit59g/ffmpeg_hardsubbing_using_force_style_fontname/)

## Positioning

[Subtitle positions](https://stackoverflow.com/questions/57869367/ffmpeg-subtitles-alignment-and-position)

e.g.
```python
# Middle
SUBTITLE_OPTIONS = f"Alignment=10"  # Netflix style

# Bottom align with padding
SUBTITLE_OPTIONS = f"Alignment=2,MarginV=50"  # Netflix style
```

## Text fade in / out


e.g. ASS file, with 1s fade in/out
```
...

[Events]
Format: Layer, Start, End, Style, Name, MarginL, MarginR, MarginV, Effect, Text
Dialogue: 0,0:00:00.00,0:00:05.00,Default,,0000,0000,0000,,{\fade(1000,1000)}Example Subtitle Text
...
```
