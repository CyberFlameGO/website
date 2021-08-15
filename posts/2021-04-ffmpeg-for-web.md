---
title: The correct configuration options to compress videos for the web with ffmpeg
description: Why isn't my encoded video playing in modern browsers?
date: 2021-04-11
tags:
    - reference
layout: layouts/post.njk
---

The motivation of this post is to document the command-line options that worked for me to reduce the size of .mp4 files prior to publishing them on the web. Here they are!
```bash
ffmpeg -i input.mp4 -vcodec h264 -acodec aac -strict -2 -crf 30 out.mp4
```
This output of this command is compatible with all modern browsers (Chrome, Firefox, Safari) and even works in IE11! The `-crf 30` flag tells ffmpeg to compress the output as much as possible, you can reduce the number for higher quality.