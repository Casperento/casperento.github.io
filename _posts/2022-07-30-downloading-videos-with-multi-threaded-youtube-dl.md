---
layout: post
title: Downloading videos with multi-threaded youtube-dl
date: 2022-07-30 15:47 -0300
categories:
- Utils
tags:
- utils
---
Today I'd like to share a trick I've learned to download videos faster using [_youtube-dl_](https://youtube-dl.org/). The _youtube-dl_ tool is a command-line utility to download videos from YouTube. But sometimes, when you try to download a large video (something like >=200MB of size), you might face a download time of 40 minutes or more, even when your common download speed is high (around 10mb/s). So, after researching some solution for this, I've found out that the best option is to use multiple threads to download the video content.

First you need to have _youtube-dl_ installed on your machine. Follow the [official installation guide](https://github.com/ytdl-org/youtube-dl#installation) and you're done.

After that, you're gonna need to download an external downloader to call from _youtube-dl_'s CLI. We're gonna use [_aria2_](https://aria2.github.io/). Just grab [a compiled version](https://github.com/aria2/aria2/releases) for your system, and add it to yout PATH variable.

Finally, type the following command in your preferred shell to download a large video faster:

```bash
$ youtube-dl --external-downloader aria2c --external-downloader-args "-s 16 -x 16 -k 1M" VIDEO-URL
```

Passing these arguments to the external downloader _aria2c_, we're gonna use 16 connections to download the video file (`-s`), in which each chunk of the file has 1MiB (`-k`), limited by 16 connections with the server providing the file.

Finally, you can check _aria2_'s [documentation](https://aria2.github.io/manual/en/html/aria2c.html#options) to personalize arguments passed to it's CLI.
