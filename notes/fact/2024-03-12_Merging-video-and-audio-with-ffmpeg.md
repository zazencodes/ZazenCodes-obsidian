---
date: 2024-03-12
tags:
    - fact
hubs:
    - "[[ffmpeg]]"
urls:
    - https://superuser.com/questions/277642/how-to-merge-audio-and-video-file-in-ffmpeg/277667#277667
---

# Merging video and audio with ffmpeg

## Layering audio and video

Merge video that has no audio stream with audio. ffmpeg will stop encoding once one file ends. Transcribe audio (since MP4s cannot carry PCM audio streams).
```
ffmpeg -i video.mp4 -i audio.wav -c:v copy -c:a aac -shortest output.mp4
```

Copy the audio without re-encoding using MKV output.
```
ffmpeg -i video.mp4 -i audio.wav -c copy output.mkv
```

If your input video already contains audio, and you want to replace it, you need to tell ffmpeg which audio stream to take. The -map option makes ffmpeg only use the first video stream from the first input and the first audio stream from the second input for the output file.
```
ffmpeg -i video.mp4 -i audio.wav -c:v copy -c:a aac -map 0:v:0 -map 1:a:0 output.mp4
```

## Concatenating videos

Use -c:a copy: This option tells FFmpeg to copy the audio stream(s) without re-encoding, preserving the original quality.
```
ffmpeg -i input1.mp4 -i input2.mp4 -filter_complex "[0:v][1:v]concat=n=2:v=1:a=0[v]" -map "[v]" -map 0:a -c:a copy output.mp4
```


## Adding subtitles

```
ffmpeg -i input.mp4 -filter_complex "[0:v]subtitles=subtitles.srt[v]" -map "[v]" -map 0:a -c:a copy output.mp4
```


