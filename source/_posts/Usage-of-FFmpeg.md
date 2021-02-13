---
title: Usage of FFmpeg
date: 2021-02-13 20:35:35
tags:
- FFmpeg
---

# 转换GIF

```
$ ffmpeg -ss {start_time} -to {end_time} -i {input_file} -r {frame_rate} -vf scale=-2:{height} {output_name}.gif
```

For example:

```
$ ffmpeg -ss 01:30:02 -to 01:30:12 -i '227 ELEVEN.mp4' -r 15 -vf scale=-2:240 cut.gif
```

# 截取视频

```
$ ffmpeg -ss {start_time} -to {end_time} -i {input_file} -preset {preset} -c:v {encoding} {output_file}
```

For example:

```
$ ffmpeg -ss 00:30:01 -to 00:30:11 -i '227 ELEVEN.mp4' -preset veryslow -c:v libx264 cut.mkv
```
