---
layout: post
title:  "Encoding for YouTube"
date:   2020-03-08 12:16:00 +0000
author: Peter
published: true
categories:
- article
tags:
- youtube
- encoding
- vmaf
- x264
- h264
- x265
- h265
- vp9
- transcoding
- lossless
- video
- let's play
---
* TOC
{:toc}

# Introduction

The purpose of these tests and this document is somewhat to justify my own settings and find out if I need to change them. However, the results are of general interest, so I thought I'd turn them into an article.

# Capture

Nine games were chosen for capture that I felt represented a variety of encoding challenges, from slow-paced 30fps games with flat block-colour menus to a 60fps colourful racing game as well as some games I thought would be a challenge for both the encoders and the quality assessment. Games were chosen because they represent things YouTube gamers would actually be uploading rather than using open source movies.

We all love [Big Buck Bunny](https://peach.blender.org/) and his wacky adventures, but it's almost, but not quite, entirely unlike game footage.

* Death Stranding
* Detroit: Become Human
* Diablo 3
* Flower
* Forza Horizon 4
* Untitled Goose Game
* Persona 4
* Resident Evil 7
* Rogue Legacy

These will be referred to by shorthand names: Death Detroit Diablo Flower Forza Goose Persona RE7 Rogue

The games were captured via OBS from a Black Magic Intensity Pro 4k device, all at 60000/1001 fps as that's what the PS4 outputs in and the PC captures were set to match. NVEnc was used in lossless mode, however, the capture was set to limited / movie colour range and used nv12 (yuv420) as the pixel format, so it wasn't true lossless, but matches the capabilities of the final destination of YouTube.

# Lossless Versions

After the initial capture, a three minute section that displayed the best cross-section of gameplay was chosen. These sections were stream-copied for a bitrate comparison to the original capture, and then transcoded losslessly to some working files at non-fractional framerates to make sure indexing works right and to not drop any more frames, these were then transcoded again losslessly (x264 veryslow qp 0) into three minute clips.

|Game|Capture rate* (kb/s)|Transcode rate (kb/s)|Final Fps|
|---|---|---|---|
|Death|307012|176507|30|
|Detroit|434886|217563|30|
|Diablo|281281|211467|60|
|Flower|317912|238585|60|
|Forza|500664|389333|60|
|Goose|241264|189136|60|
|Persona|81923|43857|30|
|RE7|298612|221580|60|
|Rogue|78416|33123|60|

![Capture rate* (kb/s) and Transcode rate (kb/s)]({{site.url}}/assets/images/Capture_rate_and_Transcode_rate.svg "Capture rate* kb/s and Transcode rate kb/s")  
_Capture rate* (kb/s) and Transcode rate (kb/s)_

*Stream-copying works on i-frame boundaries so these figures are close, but will have up to 3 seconds of extra footage, the rate takes these extra seconds into consideration.

Rates here are calculated from mkv file containing only the video steam, as embedded stream rates aren't always accurate. The downside is there'll be a touch of overhead.

Two interesting facts to note.

The capture and transcoded files have the same video frames in them, but very different bitrates. NVEnc was running in real-time and so needed to use more bits to store the information. Transcodes were not done in real-time at the 'veryslow' preset (the slowest reasonable preset, the only slower one is called placebo for good reason). All this work was done on a variety of commodity equipment in parallel, so no transcode times are available.

The variation between games is *enormous*. If you compare the smallest and largest transcoded bitrates, Forza is **more than 10 times** bigger than Rogue. Rogue Legacy is a four-way scrolling sprite-based game and was specifically chosen as easy to encode (along with Persona).

# Quality

I also want to get across early on that *bitrate does not equal quality*. Here we can see wildly different bitrates all representing lossless information and thus the same quality. Bitrate comparisons are often very misleading for this reason - this is why we're using VMAF and its 0-100 scale.

And you don't care about bitrate unless you're streaming or targeting a specific size of file.

You care about *Quality*, how good it looks in comparison to what you see on a screen. This is also a good demonstration though that one thing we do care about is overall filesize. If we didn't, we'd all happily stick to a completely lossless workflow and upload enormous files, but that would take forever, and we wouldn't be able to archive the videos. The best file is one that looks sufficiently good and is sufficiently small (and renders in a sufficiently prompt time). That will of course vary between people. In the case of this document, several different quality settings and a few recommend settings will be trialled.

VMAF Blurb: VMAF (Video Multi-Method Assessment Fusion) is a perceptual video quality assessment algorithm developed by Netflix. VMAF Development Kit (VDK) is a software package that contains the VMAF algorithm implementation, as well as a set of tools that allows a user to train and test a custom VMAF model. Read [this](https://medium.com/netflix-techblog/toward-a-practical-perceptual-video-quality-metric-653f208b9652) techblog post for an overview, or [this](https://medium.com/netflix-techblog/vmaf-the-journey-continues-44b51ee9ed12) post for the latest updates and tips for best practices.

Basically, vmaf is a rating for comparing a distorted video to its original and telling you how close it is. It is an objective measure of how close the video matches, one that takes many methods into account  and is specifically for video. The scale itself is more subjective and based upon how real humans would rate the content.

# YouTube Transcodes

YouTube transcodes your videos to a lower quality stream-friendly version. It does this to two codecs, h264 & vp9.

By uploading our lossless videos to YouTube, downloading the transcoded versions and using vmaf to compare the videos, we can get an idea of what sort of quality we can get out of YouTube and put a number to it.

Name|YT.264 VMAF|YT.264 Rate (kb/s)|YT.vp9 VMAF|YT.vp9 Rate (kb/s)
---|---|---|---|---
Forza_ll|54.373016|5654|56.147891|3285
Death_ll|57.606028|4257|65.894301|1961
Flower_ll|61.560809|5142|67.273151|2939
Diablo_ll|64.417506|5496|70.17594|3207
RE7_ll|69.8519|4034|77.311906|2287
Detroit_ll|68.334414|3536|79.690493|1716
Goose_ll|76.11744|2625|85.667461|1325
Persona_ll|85.245592|2607|89.414688|1473
Rogue_ll|96.449077|3751|93.799547|2202

![Lossless uploaded to YouTube]({{site.url}}/assets/images/Lossless_uploads_to_youtube.svg "Lossless uploaded to YouTube")  
*Lossless uploaded to YouTube*

vp9 gives better results than h264 *despite* the bitrate being lower.

The bitrate is not *consistent*. An assumption might be that YouTube has a certain bitrate limit and that, should you feed it lossless content, it will use say, up to 5mb/s, but that's not the case, YouTube like most modern encoders will encode targeting a quality - otherwise the bitrates would be flat.

You can see how YouTube will reflect hard-to-encode things and easy-to-encode things in much the same way as lossless. Coincidentally, difficult to encode things appear on the left, whereas the right-side games are easier to encode, requiring fewer bits for better quality. After a certain point, the video's quality really starts dropping, so it isn't a strict quality it targets, likely some form of bitrate constrained crf (constant rate factor, default quality targeting in most software encoders) is happening. It is also the case that YouTube segments its videos and encodes each segment separately these are the parts you need to download and assemble to create the finished YouTube video.

You may expect that lossless to YouTube would get you the best possible results from YouTube in terms of quality. In actuality, we can (and do) get higher vmaf ratings from lossy transcodes, but usually not by much and you can see a definite, but weak correlation between the vmaf rating of the Transcode and the vmaf rating of the YouTube version. A stronger correlation would likely be shown with lower quality trancodes. Local transcodes produce more predictable results.

# Producing the Videos

Here are how the rest of the videos were produced for testing.

Two codecs, h264 and h265 encoded with libx264 and libx265.

Two speeds, s: slow, vs: veryslow.

x264 and x265 both allow you to select a speed preset with one of the following values:

* placebo
* veryslow
* slower
* slow
* medium (default)
* fast
* faster
* veryfast
* superfast
* ultrafast

We're using veryslow for x264 as generally, this will produce the smallest files. Generally. For x265 we're using slow because that's what I actually use, but it is arbitrary, it represents what is fast enough. x264 is also fast enough at veryslow, but there is no slower speed (worth using). I can still use x265 after upgrading with a slower preset. Ultrafast is used just to make the point that even when following guidelines on how to encode, there is still room to tune for speed / efficiency within those parameters, but the results aren't that interesting - so you can see them in the Google Sheet, but I haven't used them in this write-up.

Just to quickly explain the quality settings, here are some terms.

[ffmpeg's h264 encoding guide](https://trac.ffmpeg.org/wiki/Encode/H.264) says of crf:

Constant Rate Factor (CRF)

"Use this rate control mode if you want to keep the best quality and care less about the file size. This is the recommended rate control mode for most uses." Note that it ranges from 0 (lossless) to 51 (hideous). Lower values mean higher quality. It will just use whatever bitrate gives you, perceptually, the required arbitrary quality. It is vbr (variable bitrate).

With cbr (constant bitrate), you assign a bitrate, and that's how much it uses. x264 doesn't actually have a cbr mode, but you can emulate it with x264 by specifying nal-hrd=cbr and the bitrate limitations. This is what OBS does too.

Several Quality settings, crf 15,18,23,28, strict YouTube recommended with bitrate recommendations (8Mbps for 30fps, and 12Mbps for 60fps), and YouTube recommended, but swapping in cbr instead of vbr. Since I'd also seen recommendations like record in cbr at 8 or 12Mbps, then encode in cbr at the same bitrate for YouTube, and thought that was terrible advice, that I'd do that too. The idea I'm guessing being that the encoder will preserve the same bits each pass.

Finally since part of what I want to be able to do is make a mistake and then re-render an episode once the source files are gone. So I wanted to see how "double-baked video" would do. So x265 was encoded at crf 18 and then that, in turn, was encoded at crf 18.

# Comparing quality targets versus bitrate targets

Turns out that YouTube's encoding process takes away so much quality that only the crf 28 encodes caused a particularly sharp decline in the final vmaf of the YouTube encodes, even double cbr, whilst consistently worse than crf, was using fairly high bitrates that mostly survived the process. So it seems like you can't go _too_ wrong at least.

A good demonstration of the difference between picking a quality setting and picking a bitrate can be shown by taking the same codec and using both modes and showing the vmaf versus the bitrate. So let's take x264, use the same speed preset of veryslow and encode crf 18 versus YouTube strict using 8 or 12Mbps. That gets us this.

Game|CRF18 VMAF|CRF18 Rate (kb/s)|YTRec VMAF|YTRec Rate (kb/s)
---|---|---|---|---
Death|99.179093|16859|69.120015|8386
Detroit|98.028856|8889|77.208784|8228
Diablo|96.894216|13035|80.557112|12282
Flower|96.282884|14280|94.192944|12412
Forza|98.319477|32144|68.470782|12083
Goose|96.063484|2618|82.46231|12218
Persona|97.764895|3826|89.441311|7988
RE7|96.013602|6867|76.679025|11783
Rogue|97.993974|3205|98.423263|12430
Average|97.39338678|11303|81.83950511|10868

![Quality versus Bitrate]({{site.url}}/assets/images/Quality_versus_bitrate.svg "Quality versus Bitrate")  
*Comparing setting quality versus selecting target bitrate for x264*

Instead of being fair, let's be realistic and use x265 instead and compare that to the YouTube recommendations. Saying that though, we'll still use the slow preset in x265 so it doesn't take forever.

Game|265_18 VMAF|265_18 Rate (kb/s)|YTRec VMAF|YTRec Rate (kb/s)
---|---|---|---|---
Death|99.023438|13962|69.120015|8386
Detroit|97.94216|6447|77.208784|8228
Diablo|96.761145|10799|80.557112|12282
Flower|96.238418|11387|94.192944|12412
Forza|97.810374|26326|68.470782|12083
Goose|96.301997|1695|82.46231|12218
Persona|97.644543|3768|89.441311|7988
RE7|96.152931|5012|76.679025|11783
Rogue|97.675072|3067|98.423263|12430
Average|97.283342|9163|81.83950511|10868

![Quality versus Bitrate]({{site.url}}/assets/images/Quality_265_versus_264_bitrate.svg "Quality versus Bitrate")  
*Comparing setting quality in x265 versus selecting target bitrate for x264*

*This* is why you don't select a bitrate when encoding for YouTube, it won't give consistent results, in fact it's almost the opposite as games will vary in quality massively and that is then passed on to YouTube to source its own video from, whereas selecting a quality will simply use more bits for more difficult content, and fewer bits when it isn't being challenged so much. It's also a vindication of both crf and vmaf as crf is supposed to target a perceptual quality and vmaf is supposed to measure it.

12Mbps just isn't enough to take on Forza, and 8Mbps isn't enough to take on Death Stranding, but 12Mbps is completely overkill for Rogue Legacy. The average is especially telling. x265 uses fewer bits overall, but maintains a much higher and consistent quality. Only Rogue Legacy will look better on transcode, and only by a sliver, and not remotely proportional to the four times the bits being used to produce it. There are also plenty of examples of games using a much lower bitrate in x264 *and* x265 and producing far better results than the much higher bitrates of the YouTube recommended bitrates in x264. Untitled Goose Game, Persona 4, and Resident Evil 7 show just how inefficient selecting a bitrate target can be. Please stop.

# Comparing Fixed-bitrate methods

Anyway, there were a bunch of different fixed bitrate transcodes, let's have a look at them all.

![Targetting Bitrate]({{site.url}}/assets/images/Bitrate_targets.svg "Targetting Bitrate")  
*Comparing selecting target bitrate for x264 with Strict YouTube versus cbr and cbr into cbr*

Unsurprisingly, CBR is always worse than VBR, and CBR into CBR is always worse than just CBR on its own.

Surprisingly, CBR wasn't _much_ worse than VBR, but then I suppose it isn't really CBR, but an emulation of it that makes it pad the bitrate over a certain framesize, whereas most slow spots or black screen are going to be shorter than that framesize.

The conclusion however is still avoid CBR when transcoding for YouTube, and actually, avoid it all together. Even for streaming you're better off using crf with a constrained bitrate limit.

# Comparing CRF methods

![Comparing x265 crf values]({{site.url}}/assets/images/crf_comparison.svg "Comparing x265 crf values")  
*Comparing targetting quality with x265*

My process has been to use crf 15 for 60fps and crf 18 for 30fps. Turns out, that's overkill and the quality gain from 18 to 15 isn't worth the rate increase, and 18 is fine for 30 or 60fps. So from now I will be using only crf 18.

# Double-baking crf

![Double-baking crf 18]({{site.url}}/assets/images/Double-baked_18_2.svg "Double-baking crf 18")  
*Double-baking crf 18*

I'm also quite happy that crf 18 can survive a further transcode of itself with an expected generational loss, but at least the bitrate also goes down. There's barely any effect on the YouTube encodes, here we're showing the better vp9 codec.

# What about 4k?

Next article, how 4k affects YouTube encoding.

![Uploading 2160p]({{site.url}}/assets/images/4k_preview.svg "Uploading 2160p")  
*Uploading 2160p*

Turns out the encoding done by YouTube when you upload 4k is so much better that even uploading upscaled 1080p content will result in significantly better YouTube vmaf scores for the same resolution. Here we show the realistic scenario. h264 version of the YouTube video shown by default for me versus the 2160p vp9 versions which are the default when uploading the upscaled to 2160p video and viewing in various resolution settings. Not shown, but comparing 1080 vp9 to 1080 vp9 versions on YouTube still shows a significant quality increase and 2160p viewed in its native resolution gives comparable results to 2160p viewed as 1080p.

So in actuality, all future videos will be quick rendered in 2160p with a lanzos upscale and then in 1080p crf 18 slow for archival. Because YouTube's encoder is better when handling 2160p content.

# Appendix A Tools

ffmpeg-4.1.3 for transcoding.

```
ffmpeg version 4.1.3 Copyright (c) 2000-2019 the FFmpeg developers
built with gcc 9.2.0 (Gentoo 9.2.0-r2 p3)
configuration: --prefix=/usr --libdir=/usr/lib64 --shlibdir=/usr/lib64 --docdir=/usr/share/doc/ffmpeg-4.1.3/html --mandir=/usr/share/man --enable-shared --cc=x86_64-pc-linux-gnu-gcc --cxx=x86_64-pc-linux-gnu-g++ --ar=x86_64-pc-linux-gnu-ar --optflags='-march=native -O2 -pipe' --disable-static --enable-avfilter --enable-avresample --enable-libvmaf --enable-version3 --disable-stripping --disable-optimizations --disable-libcelt --disable-indev=v4l2 --disable-outdev=v4l2 --disable-indev=oss --disable-indev=jack --disable-outdev=oss --enable-bzlib --disable-runtime-cpudetect --disable-debug --disable-gcrypt --disable-gnutls --disable-gmp --enable-gpl --enable-hardcoded-tables --enable-iconv --disable-libtls --disable-libxml2 --disable-lzma --enable-network --disable-opencl --disable-openssl --enable-postproc --disable-libsmbclient --enable-ffplay --enable-sdl2 --enable-vaapi --enable-vdpau --enable-xlib --enable-libxcb --enable-libxcb-shm --enable-libxcb-xfixes --enable-zlib --disable-libcdio --disable-libiec61883 --disable-libdc1394 --disable-libcaca --disable-openal --enable-opengl --disable-libv4l2 --enable-libpulse --disable-libdrm --disable-libjack --disable-libopencore-amrwb --disable-libopencore-amrnb --disable-libcodec2 --disable-libfdk-aac --disable-libopenjpeg --disable-libbluray --disable-libgme --disable-libgsm --disable-mmal --disable-libmodplug --enable-libopus --disable-libilbc --disable-librtmp --disable-libssh --disable-libspeex --disable-libsrt --enable-librsvg --disable-ffnvcodec --enable-libvorbis --disable-libvpx --disable-libzvbi --disable-appkit --disable-libbs2b --disable-chromaprint --disable-libflite --disable-frei0r --disable-libfribidi --disable-fontconfig --disable-ladspa --disable-libass --disable-lv2 --enable-libfreetype --disable-librubberband --disable-libzmq --disable-libzimg --disable-libsoxr --enable-pthreads --disable-libvo-amrwbenc --enable-libmp3lame --disable-libkvazaar --disable-libaom --disable-libopenh264 --disable-libsnappy --disable-libtheora --disable-libtwolame --enable-libwavpack --disable-libwebp --enable-libx264 --enable-libx265 --enable-libxvid --disable-armv5te --disable-armv6 --disable-armv6t2 --disable-neon --disable-vfp --disable-vfpv3 --disable-armv8 --disable-mipsdsp --disable-mipsdspr2 --disable-mipsfpu --disable-altivec --disable-amd3dnow --disable-amd3dnowext --disable-avx2 --cpu=host --disable-doc --disable-htmlpages --enable-manpages
libavutil      56. 22.100 / 56. 22.100
libavcodec     58. 35.100 / 58. 35.100
libavformat    58. 20.100 / 58. 20.100
libavdevice    58.  5.100 / 58.  5.100
libavfilter     7. 40.101 /  7. 40.101
libavresample   4.  0.  0 /  4.  0.  0
libswscale      5.  3.100 /  5.  3.100
libswresample   3.  3.100 /  3.  3.100
libpostproc    55.  3.100 / 55.  3.100
```

vmaf-1.3.15 for the vmaf library.

gnu parallel-20191022 for pushing jobs around the network.

obs-24.0.3 for capturing the video from the Black Magic card.

youtube-dl-2020.01.24 for downloading the videos from YouTube for comparison.

# Appendix B Creating Transcodes

All capturing was done via hmdi into a Black Magic Intensity Pro 4k in 1080p full rgb @60000/1001. OBS was used to record the stream in yuv420p limited colour.

Transcode to lossless just to smooth out any possible issues with indexing or the like and to strip the audio.

```bash
parallel --nice 20 --eta -S node1,node2,node3 -j1 ffmpeg -i {} -qp 0 -an {.}_ll_medium.mkv ::: /mnt/LPWorking/vmaf/ORIG/*.mkv
```

Framecopy out the 'br' versions, which are specifically to get an idea of the bitrate of the original captures for the same three minutes. These will be slightly longer than 180 seconds, but when we work out the bitrate, we take these extra seconds into consideration. This is why it's not 100% accurate.

```bash
parallel --eta -j1 ffmpeg -ss {1} -nostdin -i {2}.mkv -c copy -an -t 180 -y {2}_br.mkv ::: 237 141 90 16 5 7 210 7 14 :::+ Death Detroit Diablo Flower Forza Goose Persona RE7 Rogue
```

Get three minute clips from the transcodes of the original captures - also set the framerate to what the content should be in, rather than the obs one. Non-fractional framerates for simplicity's sake. These were used as reference videos for vmaf.

```bash
parallel --eta -S node1,node2,node3 -j1 ffmpeg -ss {1} -nostdin -i /mnt/LPWorking/vmaf/ORIG/{2}_ll_medium.mkv -qp 0 -preset veryslow -r {3} -t 180 -y /mnt/LPWorking/vmaf/{2}_ll.mkv ::: 237 141 90 16 5 7 210 7 14 :::+ Death Detroit Diablo Flower Forza Goose Persona RE7 Rogue :::+ 30 30 60 60 60 60 30 60 60
```

CRF versions were made like this.

```bash
parallel --eta -j1 -S node1,node2,node3 ffmpeg -i /mnt/LPWorking/vmaf/{1}_ll.mkv -preset veryslow -crf {2} /mnt/LPWorking/vmaf/{1}_264_vs_{2}.mkv ::: Death Detroit Diablo Flower Forza Goose Persona RE7 Rogue ::: 15 18 23 28
parallel --eta -j1 -S node1,node2,node3 ffmpeg -i /mnt/LPWorking/vmaf/{1}_ll.mkv -preset slow -c libx265 -crf {2} /mnt/LPWorking/vmaf/{1}_265_s_{2}.mkv ::: Death Detroit Diablo Flower Forza Goose Persona RE7 Rogue ::: 15 18 23 28
```

Strict YouTube versions.

```bash
parallel -j1 --eta -S node1,node2,node3 ffmpeg -i /mnt/LPWorking/vmaf/{1}_ll.mkv -g {3} -bf 2 -profile:v high -movflags faststart -coder 1 -preset veryslow -b:v {2}M -y /mnt/LPWorking/vmaf/{1}_264_vs_ytrec.mp4 ::: Death Detroit Diablo Flower Forza Goose Persona RE7 Rogue :::+ 8 8 12 12 12 12 8 12 12 :::+ 15 15 30 30 30 30 15 30 30
parallel -j1 --eta -S node1,node2,node3 ffmpeg -i /mnt/LPWorking/vmaf/{1}_ll.mkv -g {3} -bf 2 -profile:v high -movflags faststart -coder 1 -preset ultrafast -b:v {2}M -y /mnt/LPWorking/vmaf/{1}_264_uf_ytrec.mp4 ::: Death Detroit Diablo Flower Forza Goose Persona RE7 Rogue :::+ 8 8 12 12 12 12 8 12 12 :::+ 15 15 30 30 30 30 15 30 30
```

CBR versions.

```bash
parallel -j1 --eta -S node1,node2,node3 --nice 20 ffmpeg -i /mnt/LPWorking/vmaf/{1}_ll.mkv -g {3} -bf 2 -profile:v high -movflags faststart -coder 1 -preset ultrafast -x264-params "nal-hrd=cbr" -b:v {2}M -minrate {2}M -maxrate {2}M -bufsize 2M  -y /mnt/LPWorking/vmaf/{1}_264_uf_cbr.mp4 ::: Death Detroit Diablo Flower Forza Goose Persona RE7 Rogue :::+ 8 8 12 12 12 12 8 12 12 :::+ 15 15 30 30 30 30 15 30 30
parallel -j1 --eta -S node1,node2,node3 --nice 20 ffmpeg -i /mnt/LPWorking/vmaf/{1}_ll.mkv -g {3} -bf 2 -profile:v high -movflags faststart -coder 1 -preset veryslow -x264-params "nal-hrd=cbr" -b:v {2}M -minrate {2}M -maxrate {2}M -bufsize 2M  -y /mnt/LPWorking/vmaf/{1}_264_vs_cbr.mp4 ::: Death Detroit Diablo Flower Forza Goose Persona RE7 Rogue :::+ 8 8 12 12 12 12 8 12 12 :::+ 15 15 30 30 30 30 15 30 30
```

CBR to CBR versions.

```bash
parallel -j1 --eta -S node1,node2,node3 --nice 20 ffmpeg -i /mnt/LPWorking/vmaf/{1}_264_vs_cbr.mp4 -g {3} -bf 2 -profile:v high -movflags faststart -coder 1 -preset veryslow -x264-params "nal-hrd=cbr" -b:v {2}M -minrate {2}M -maxrate {2}M -bufsize 2M -y /mnt/LPWorking/vmaf/{1}_264_vs_cbrcbr.mp4 ::: Death Detroit Diablo Flower Forza Goose Persona RE7 Rogue :::+ 8 8 12 12 12 12 8 12 12 :::+ 15 15 30 30 30 30 15 30 30
parallel -j1 --eta -S node1,node2,node3 --nice 20 ffmpeg -i /mnt/LPWorking/vmaf/{1}_264_uf_cbr.mp4 -g {3} -bf 2 -profile:v high -movflags faststart -coder 1 -preset ultrafast -x264-params "nal-hrd=cbr" -b:v {2}M -minrate {2}M -maxrate {2}M -bufsize 2M -y /mnt/LPWorking/vmaf/{1}_264_uf_cbrcbr.mp4 ::: Death Detroit Diablo Flower Forza Goose Persona RE7 Rogue :::+ 8 8 12 12 12 12 8 12 12 :::+ 15 15 30 30 30 30 15 30 30
```

... and finally, Double-baked CRF.

```bash
parallel --eta -j1 --nice 20 -S node1,node2,node3 ffmpeg -i /mnt/LPWorking/vmaf/{}_265_s_18.mkv -preset slow -crf 18 /mnt/LPWorking/vmaf/{}_265_s_1818.mkv ::: Death Detroit Diablo Flower Forza Goose Persona RE7 Rogue
```

# Appendix C Transcoded VMAF comparisons

Self comparison for lossless

```bash
parallel --nice 20 --eta -j1 -S node1,node2,node3 ffmpeg -i /mnt/LPWorking/vmaf/{1}_ll.mkv -i /mnt/LPWorking/vmaf/{1}_ll.mkv -lavfi libvmaf="model_path=/usr/share/model/vmaf_v0.6.1.pkl" -f null /dev/null ::: Death Detroit Diablo Flower Forza Goose Persona RE7 Rogue &> /mnt/LPWorking/vmaf/vmafscores_self.txt
```

Comparisons for all the CRF files.

```bash
parallel --nice 20 --eta -j1 -S node1,node2,node3 ffmpeg -i /mnt/LPWorking/vmaf/{1}_{3}_{4}_{2}.mkv -i /mnt/LPWorking/vmaf/{1}_ll.mkv -lavfi libvmaf="model_path=/usr/share/model/vmaf_v0.6.1.pkl" -f null /dev/null ::: Death Detroit Diablo Flower Forza Goose Persona RE7 Rogue ::: 15 18 23 28 ::: 264 265 :::+ vs s &> /mnt/LPWorking/vmaf/vmafscores_crf.txt
```

Comparisons for all the bitrate targetted files.

```bash
parallel --nice 20 --eta -j1 -S node1,node2,node3 ffmpeg -i /mnt/LPWorking/vmaf/{1}_264_{2}_{3}.mp4 -i /mnt/LPWorking/vmaf/{1}_ll.mkv -lavfi libvmaf="model_path=/usr/share/model/vmaf_v0.6.1.pkl" -f null -t 180 /dev/null ::: Death Detroit Diablo Flower Forza Goose Persona RE7 Rogue ::: uf vs ::: ytrec cbr cbrcbr &> /mnt/LPWorking/vmaf/vmafscores_mp4.txt
```

Comparisons for Twice-baked CRF.

```bash
parallel --eta -j1 -S node1,node2,node3 ffmpeg -i /mnt/LPWorking/vmaf/{1}_265_s_1818.mkv -i /mnt/LPWorking/vmaf/{1}_ll.mkv -lavfi libvmaf="model_path=/usr/share/model/vmaf_v0.6.1.pkl" -f null -t 180 /dev/null ::: Death Detroit Diablo Flower Forza Goose Persona RE7 Rogue &> /mnt/LPWorking/vmaf/vmafscores_1818.txt
```

# Appendix D VMAF Comparisons from YouTube

Lossless upload.

```bash
parallel -j1 --eta -S node1,node2,node3 ffmpeg -i /mnt/LPWorking/vmaf/yt/{}_ll.mp4 -i /mnt/LPWorking/vmaf/{}_ll.mkv -lavfi libvmaf="model_path=/usr/share/model/vmaf_v0.6.1.pkl" -f null -t 180 /dev/null ::: Death Detroit Diablo Flower Forza Goose Persona RE7 Rogue &> vmafscores_ll_264.txt
parallel -j1 --eta -S node1,node2,node3 ffmpeg -i /mnt/LPWorking/vmaf/yt/{}_ll.webm -i /mnt/LPWorking/vmaf/{}_ll.mkv -lavfi libvmaf="model_path=/usr/share/model/vmaf_v0.6.1.pkl" -f null -t 180 /dev/null ::: Death Detroit Diablo Flower Forza Goose Persona RE7 Rogue &> vmafscores_ll_vp9.txt
```

CRF comparisons.

```bash
parallel --eta -j1 -S node1,node2,node3 ffmpeg -i /mnt/LPWorking/vmaf/yt/{1}_{3}_{4}_{2}.mp4 -i /mnt/LPWorking/vmaf/{1}_ll.mkv -lavfi libvmaf="model_path=/usr/share/model/vmaf_v0.6.1.pkl" -f null -t 180 /dev/null ::: Death Detroit Diablo Flower Forza Goose Persona RE7 Rogue ::: 15 18 23 28 ::: 264 265 :::+ vs s &> vmafscores_264.txt
parallel --eta -j1 -S node1,node2,node3 ffmpeg -i /mnt/LPWorking/vmaf/yt/{1}_{3}_{4}_{2}.webm -i /mnt/LPWorking/vmaf/{1}_ll.mkv -lavfi libvmaf="model_path=/usr/share/model/vmaf_v0.6.1.pkl" -f null -t 180 /dev/null ::: Death Detroit Diablo Flower Forza Goose Persona RE7 Rogue ::: 15 18 23 28 ::: 264 265 :::+ vs s &> vmafscores_vp9.txt
```

Comparisons for all the bitrate targetted files.

```bash
parallel --nice 20 --eta -j1 -S node1,node2,node3 ffmpeg -i /mnt/LPWorking/vmaf/yt/{1}_264_{2}_{3}.mp4 -i /mnt/LPWorking/vmaf/{1}_ll.mkv -lavfi libvmaf="model_path=/usr/share/model/vmaf_v0.6.1.pkl" -f null -t 180 /dev/null ::: Death Detroit Diablo Flower Forza Goose Persona RE7 Rogue ::: uf vs ::: ytrec cbr cbrcbr &> /mnt/LPWorking/vmaf/yt/vmafscores_rec_264.txt
parallel --nice 20 --eta -j1 -S node1,node2,node3 ffmpeg -i /mnt/LPWorking/vmaf/yt/{1}_264_{2}_{3}.webm -i /mnt/LPWorking/vmaf/{1}_ll.mkv -lavfi libvmaf="model_path=/usr/share/model/vmaf_v0.6.1.pkl" -f null -t 180 /dev/null ::: Death Detroit Diablo Flower Forza Goose Persona RE7 Rogue ::: uf vs ::: ytrec cbr cbrcbr &> /mnt/LPWorking/vmaf/yt/vmafscores_rec_vp9.txt
```

Comparisons for Twice-baked CRF.

```bash
parallel --nice 20 --eta -j1 -S node1,node2,node3 ffmpeg -i /mnt/LPWorking/vmaf/yt/{1}_265_s_1818.mp4 -i /mnt/LPWorking/vmaf/{1}_ll.mkv -lavfi libvmaf="model_path=/usr/share/model/vmaf_v0.6.1.pkl" -f null -t 180 /dev/null ::: Death Detroit Diablo Flower Forza Goose Persona RE7 Rogue &> /mnt/LPWorking/vmaf/yt/vmafscores_1818_264.txt
parallel --nice 20 --eta -j1 -S node1,node2,node3 ffmpeg -i /mnt/LPWorking/vmaf/yt/{1}_265_s_1818.webm -i /mnt/LPWorking/vmaf/{1}_ll.mkv -lavfi libvmaf="model_path=/usr/share/model/vmaf_v0.6.1.pkl" -f null -t 180 /dev/null ::: Death Detroit Diablo Flower Forza Goose Persona RE7 Rogue &> /mnt/LPWorking/vmaf/yt/vmafscores_1818_vp9.txt
```

# Appendix X Results Tables

Here are the tables of data for completeness.

If you just want to see the data for charting etc. Please use the Google Doc https://docs.google.com/spreadsheets/d/1D3000VzF7XZ1HKsLKYdK2CULOyII0hFwo257vxUzLnY/edit?usp=sharing

Game|Capture rate* (kb/s)|Transcode rate (kb/s)|Final Fps|Capture Size|Transcode Size
---|---|---|---|---|---
Death|307012|176507|30|7040946293|3971409494
Detroit|434886|217563|30|9970854162|4895175886
Diablo|281281|211467|60|6415678692|4758011796
Flower|317912|238585|60|7293305749|5368162899
Forza|500664|389333|60|11318766196|8759995647
Goose|241264|189136|60|5514996855|4255550684
Persona|81923|43857|30|1858737306|986790988
RE7|298612|221580|60|6825892227|4985544618
Rogue|78416|33123|60|1779155973|745274044
Average|282441|191239|50|6446481495|4302879562

Name|Transcode VMAF|Transcode Rate (kb/s)|YT.264 VMAF|YT.264 Rate (kb/s)|YT.vp9 VMAF|YT.vp9 Rate (kb/s)|Trancoded Size|YT.264 Size|YT.vp9 Size
---|---|---|---|---|---|---|---|---|---
Death_264_uf_cbr|67.170819|8002|56.039215|4438|59.186626|1927|180041664|99844109|43355383
Death_264_uf_cbrcbr|64.361487|8002|54.460046|4416|57.944661|1928|180039429|99363584|43384830
Death_264_uf_ytrec|67.914076|8310|56.244398|4423|59.311758|1932|186965479|99514490|43480780
Death_264_vs_15|99.694551|25197|57.55067|4256|65.716546|1959|566931155|95754503|44073333
Death_264_vs_18|99.179093|16859|57.404738|4242|65.547333|1957|379330750|95453399|44038592
Death_264_vs_23|95.273895|8251|57.720714|4427|60.503374|1930|185637158|99609686|43422512
Death_264_vs_28|84.922153|4016|50.188563|4186|58.311813|1964|90369193|94186195|44185395
Death_264_vs_cbr|68.247038|8004|57.331133|4445|60.304952|1934|180097959|100007254|43507926
Death_264_vs_cbrcbr|65.563274|8003|56.280751|4466|59.675803|1938|180058613|100481041|43607225
Death_264_vs_ytrec|69.120015|8386|57.491718|4433|60.3571|1936|188685066|99733146|43569036
Death_265_s_15|99.622277|21093|58.426482|4434|60.789081|1935|474599657|99758224|43548322
Death_265_s_18|99.023438|13962|57.284986|4242|65.486753|1955|314153911|95434306|43982566
Death_265_s_1818|97.502328|12807|57.852953|4427|60.483807|1932|288152248|99612477|43473458
Death_265_s_23|95.377011|6852|57.575012|4435|60.462077|1932|154161249|99784141|43468481
Death_265_s_28|86.784151|3263|50.371678|4194|58.649406|1960|73407268|94363557|44089549
Death_ll|99.937872|176507|57.606028|4257|65.894301|1961|3971409494|95774239|44121968
Detroit_264_uf_cbr|76.297497|8002|68.886341|3504|79.071504|1711|180039724|78842052|38504205
Detroit_264_uf_cbrcbr|75.42974|8002|68.311523|3510|78.361377|1715|180039500|78966842|38590540
Detroit_264_uf_ytrec|77.022013|8238|67.31789|3484|78.597533|1712|185351940|78398301|38516595
Detroit_264_vs_15|98.552945|19016|68.155937|3499|79.59872|1714|427869845|78730566|38560415
Detroit_264_vs_18|98.028856|8889|68.128538|3511|79.433685|1711|200009536|78999786|38497687
Detroit_264_vs_23|96.208758|3345|67.317815|3467|78.501758|1701|75273720|77997467|38269453
Detroit_264_vs_28|90.955378|1788|55.609121|3291|64.554201|1665|40221670|74043579|37458385
Detroit_264_vs_cbr|76.568403|8004|69.547736|3500|79.966464|1718|180094594|78753355|38656200
Detroit_264_vs_cbrcbr|75.886727|8005|69.196849|3494|79.506965|1717|180105807|78615884|38622669
Detroit_264_vs_ytrec|77.208784|8228|67.962808|3508|79.230978|1715|185140499|78919264|38587640
Detroit_265_s_15|98.438399|13375|67.999537|3504|79.347754|1705|300943133|78849315|38372501
Detroit_265_s_18|97.94216|6447|67.874461|3490|79.145717|1701|145062821|78529570|38278091
Detroit_265_s_1818|97.094961|5485|67.328119|3453|78.57657|1689|123405557|77691690|38002460
Detroit_265_s_23|96.48004|2552|67.211333|3462|78.484413|1693|57418441|77906170|38095093
Detroit_265_s_28|92.530513|1278|56.030748|3320|65.086951|1666|28756460|74709380|37483895
Detroit_ll|99.504689|217563|68.334414|3536|79.690493|1716|4895175886|79567396|38608798
Diablo_264_uf_cbr|77.996417|12004|63.051692|5496|68.32209|3195|270100246|123649023|71886724
Diablo_264_uf_cbrcbr|76.400558|12004|62.572641|5528|67.713572|3193|270099321|124371913|71842697
Diablo_264_uf_ytrec|79.272198|12361|69.590782|5550|76.076226|3163|278123593|124875197|71174661
Diablo_264_vs_15|98.106589|19346|64.318407|5482|70.04671|3203|435275210|123337158|72069890
Diablo_264_vs_18|96.894216|13035|64.269661|5498|69.925102|3196|293280333|123711748|71905850
Diablo_264_vs_23|92.804394|6737|63.775731|5450|69.352453|3185|151587428|122620534|71664092
Diablo_264_vs_28|84.837268|3476|62.118481|5433|67.233671|3151|78213679|122232518|70906575
Diablo_264_vs_cbr|79.399942|12004|64.161044|5519|69.760989|3201|270099790|124188605|72029145
Diablo_264_vs_cbrcbr|78.043105|12004|63.753272|5488|69.280496|3201|270099749|123469217|72016828
Diablo_264_vs_ytrec|80.557112|12282|64.132311|5465|69.837804|3204|276350877|122972209|72093012
Diablo_265_s_15|97.950499|16153|72.790804|5330|74.668467|3121|363439250|119935055|70224019
Diablo_265_s_18|96.761145|10799|64.274843|5477|70.00497|3189|242971955|123228196|71743989
Diablo_265_s_1818|95.238331|10344|64.005861|5456|69.729306|3174|232734805|122763424|71425082
Diablo_265_s_23|93.006921|5392|63.979798|5468|69.619533|3160|121330646|123020259|71104048
Diablo_265_s_28|85.919885|2622|62.563314|5427|68.010864|3110|58984142|122100782|69969486
Diablo_ll|99.495108|211467|64.417506|5496|70.17594|3207|4758011796|123665492|72164165
Flower_264_uf_cbr|90.820921|12003|60.289722|5112|65.51911|2905|270074770|115020107|65365803
Flower_264_uf_cbrcbr|87.23414|12003|59.52696|5136|64.528292|2897|270076071|115567271|65189777
Flower_264_uf_ytrec|92.20385|12482|60.31722|5103|65.603094|2912|280851844|114812752|65522093
Flower_264_vs_15|98.033976|21657|61.476664|5142|67.105189|2930|487284073|115699067|65929188
Flower_264_vs_18|96.282884|14280|78.859306|5196|70.496349|2925|321303667|116912183|65811665
Flower_264_vs_23|90.309735|7280|60.627877|5145|66.016645|2907|163791603|115753544|65397501
Flower_264_vs_28|79.213158|3784|57.865338|5072|62.783757|2877|85132045|114109923|64737343
Flower_264_vs_cbr|93.089044|12003|61.070225|5127|66.610977|2920|270076943|115348193|65701206
Flower_264_vs_cbrcbr|90.262987|12003|60.685497|5119|66.097981|2913|270077386|115169337|65544217
Flower_264_vs_ytrec|94.192944|12412|61.119528|5132|66.688747|2927|279263084|115470547|65864494
Flower_265_s_15|97.890013|17416|61.383645|5135|67.058359|2921|391866568|115542709|65729248
Flower_265_s_18|96.238418|11387|61.241824|5118|66.91126|2916|256212430|115157382|65599187
Flower_265_s_1818|93.792025|10549|77.822758|5187|69.782112|2898|237347369|116700825|65214516
Flower_265_s_23|91.079349|5639|60.673012|5115|66.230594|2899|126871689|115078549|65231405
Flower_265_s_28|82.067773|2775|58.792273|5092|64.040091|2872|62440546|114562620|64612975
Flower_ll|99.523266|238585|61.560809|5142|67.273151|2939|5368162899|115706202|66135515
Forza_264_uf_cbr|64.729647|12004|50.81258|5440|60.342175|3279|270100444|122407182|73780840
Forza_264_uf_cbrcbr|61.770599|12004|50.810904|5725|53.410165|3228|270099542|128812514|72636182
Forza_264_uf_ytrec|66.114957|12195|51.122722|5423|60.825468|3280|274381014|122022612|73803256
Forza_264_vs_15|99.29691|48754|54.167064|5639|56.009519|3280|1096954002|126867156|73803418
Forza_264_vs_18|98.319477|32144|52.112966|5423|63.959207|3404|723248160|122018809|76589940
Forza_264_vs_23|92.537892|15131|51.751005|5385|63.348598|3408|340451509|121152707|76668786
Forza_264_vs_28|80.466659|6753|52.019913|5628|54.765883|3277|151935408|126640535|73723302
Forza_264_vs_cbr|66.84449|12005|51.670847|5398|62.687511|3441|270103243|121462450|77421072
Forza_264_vs_cbrcbr|64.543753|12007|51.390652|5408|61.763106|3434|270165096|121678591|77265584
Forza_264_vs_ytrec|68.470782|12083|51.676026|5423|62.87408|3452|271868686|122015814|77674164
Forza_265_s_15|99.097435|40023|52.005312|5391|64.009681|3400|900506342|121299280|76505252
Forza_265_s_18|97.810374|26326|52.031377|5401|63.894881|3399|592338478|121519709|76481172
Forza_265_s_1818|95.234687|24286|53.618469|5620|55.63266|3265|546438965|126458972|73458497
Forza_265_s_23|92.052077|12271|51.672386|5402|63.338261|3392|276093174|121541573|76321839
Forza_265_s_28|81.483807|5268|51.731833|5656|54.616867|3253|118523102|127252331|73187594
Forza_ll|99.731102|389333|54.373016|5654|56.147891|3285|8759995647|127216993|73909804
Goose_264_uf_cbr|82.44302|12004|75.741353|2585|85.238116|1318|270100006|58166935|29648678
Goose_264_uf_cbrcbr|82.060309|12004|75.574072|2584|85.035081|1326|270100653|58142795|29835382
Goose_264_uf_ytrec|82.679568|12506|75.854697|2593|85.376269|1314|281391131|58345790|29565474
Goose_264_vs_15|96.859109|4094|75.811661|2564|85.318066|1300|92104733|57696445|29245506
Goose_264_vs_18|96.063484|2618|75.558384|2537|85.026407|1292|58913753|57091305|29064982
Goose_264_vs_23|93.689916|1530|63.995765|2668|70.673336|1433|34426215|60019415|32239266
Goose_264_vs_28|89.087909|1033|62.342069|2634|68.754366|1417|23248165|59270440|31878260
Goose_264_vs_cbr|82.210953|12005|78.012663|2328|84.943394|1329|270103419|52390287|29894900
Goose_264_vs_cbrcbr|81.879387|12008|75.672767|2558|85.171845|1305|270170378|57552529|29366799
Goose_264_vs_ytrec|82.46231|12218|75.882753|2583|85.412708|1310|274902604|58108981|29481612
Goose_265_s_15|96.970247|2604|75.593086|2575|85.068361|1292|58583974|57936822|29071221
Goose_265_s_18|96.301997|1695|75.384589|2565|84.802974|1284|38133039|57709265|28893136
Goose_265_s_1818|95.157821|1915|76.78748|2228|83.586637|1285|43095519|50132383|28902174
Goose_265_s_23|94.421579|931|63.916998|2694|70.601952|1441|20939142|60614632|32417698
Goose_265_s_28|90.766738|572|74.276496|2254|80.978117|1276|12862051|50716476|28703598
Goose_ll|98.769961|189136|76.11744|2625|85.667461|1325|4255550684|59058303|29818338
Persona_264_uf_cbr|87.87474|8002|83.464212|2780|87.824627|1527|180040874|62560046|34347722
Persona_264_uf_cbrcbr|87.050873|8002|82.802116|2823|87.204416|1555|180040618|63523033|34995319
Persona_264_uf_ytrec|89.250364|8150|84.758816|2583|88.954543|1476|183376795|58106608|33213109
Persona_264_vs_15|98.071494|5589|85.021831|2562|89.209192|1467|125755818|57634444|33004785
Persona_264_vs_18|97.764895|3826|84.822397|2545|89.023112|1462|86079699|57259079|32895970
Persona_264_vs_23|96.791966|2084|76.347832|2452|80.029167|1400|46897687|55178369|31500447
Persona_264_vs_28|94.271684|1189|75.307659|2415|78.92975|1387|26749320|54338683|31205736
Persona_264_vs_cbr|89.112521|8002|84.798437|2575|89.017621|1478|180038928|57941045|33249331
Persona_264_vs_cbrcbr|88.70434|8002|84.544804|2585|88.769201|1492|180039239|58165181|33559401
Persona_264_vs_ytrec|89.441311|7988|84.956704|2576|89.161317|1472|179727730|57956970|33122168
Persona_265_s_15|97.951096|5515|84.842391|2534|89.056717|1458|124084162|57023780|32806520
Persona_265_s_18|97.644543|3768|84.632011|2517|88.865603|1446|84771267|56623656|32523764
Persona_265_s_1818|97.157507|3560|85.743572|2466|87.748083|1409|80099914|55491841|31696960
Persona_265_s_23|96.762617|2030|76.251858|2452|79.936432|1393|45679867|55168405|31341544
Persona_265_s_28|94.731936|1118|75.446565|2399|79.116052|1377|25146715|53969481|30991059
Persona_ll|98.573126|43857|85.245592|2607|89.414688|1473|986790988|58665095|33149132
RE7_264_uf_cbr|75.888887|12004|64.227227|4214|74.419716|2213|270100090|94807880|49790816
RE7_264_uf_cbrcbr|74.956536|12005|63.790475|4212|73.843212|2208|270106343|94760349|49672323
RE7_264_uf_ytrec|76.594269|11797|64.487254|4238|74.829886|2220|265428700|95364168|49950325
RE7_264_vs_15|97.105435|10552|64.780621|4257|75.20065|2214|237430576|95790898|49823799
RE7_264_vs_18|96.013602|6867|64.542449|4232|74.924611|2204|154499455|95213185|49586747
RE7_264_vs_23|92.261366|3589|63.415114|4097|73.529831|2160|80760447|92189174|48598981
RE7_264_vs_28|84.265748|2050|50.471717|3472|57.670474|1940|46131371|78122796|43646861
RE7_264_vs_cbr|76.056284|12006|64.662748|4251|75.006354|2209|270139875|95648587|49702661
RE7_264_vs_cbrcbr|75.264467|12008|64.335408|4221|74.613173|2201|270188428|94973634|49530895
RE7_264_vs_ytrec|76.679025|11783|64.79921|4277|75.185211|2220|265114062|96242045|49945085
RE7_265_s_15|97.120023|7661|64.637117|4264|75.060625|2211|172362662|95930665|49751101
RE7_265_s_18|96.152931|5012|64.376164|4227|74.762484|2204|112763295|95112842|49584275
RE7_265_s_1818|94.367464|5074|67.990275|3946|75.536917|2226|114161155|88784903|50088347
RE7_265_s_23|93.104052|2546|63.48773|4186|73.661428|2169|57289274|94189994|48794086
RE7_265_s_28|86.943132|1335|51.266253|3575|58.684662|1964|30033787|80427882|44194339
RE7_ll|99.259192|221580|69.8519|4034|77.311906|2287|4985544618|90764454|51446797
Rogue_264_uf_cbr|96.925727|12003|86.99517|5302|90.175933|2689|270077750|119284444|60507087
Rogue_264_uf_cbrcbr|96.030996|12003|85.911095|5435|89.050071|2741|270078488|122278877|61683684
Rogue_264_uf_ytrec|98.027889|12711|88.544829|5153|91.696659|2575|285996470|115933980|57933800
Rogue_264_vs_15|98.269626|4315|89.015472|5132|92.124669|2541|97084851|115478779|57177302
Rogue_264_vs_18|97.993974|3205|88.866763|5122|91.956588|2538|72106078|115234619|57115175
Rogue_264_vs_23|97.065343|1984|77.474442|4933|80.302464|2626|44633382|110985633|59081494
Rogue_264_vs_28|94.769673|1293|92.951124|3607|90.688775|2209|29097238|81155353|49696367
Rogue_264_vs_cbr|98.042362|12003|88.87322|5173|91.995923|2572|270076402|116399110|57871595
Rogue_264_vs_cbrcbr|97.708131|12003|88.541975|5217|91.678204|2588|270077320|117373447|58240322
Rogue_264_vs_ytrec|98.423263|12430|89.049659|5187|92.17192|2557|279668682|116710275|57540070
Rogue_265_s_15|98.020741|4181|88.762654|5099|91.878941|2530|94080135|114718029|56930200
Rogue_265_s_18|97.675072|3067|88.606968|5037|91.652086|2514|69006708|113343466|56554417
Rogue_265_s_1818|97.155745|2910|95.185235|3670|92.564231|2160|65471586|82578171|48596652
Rogue_265_s_23|96.665464|1851|77.045894|4903|79.874038|2612|41654962|110318033|58771227
Rogue_265_s_28|94.167452|1127|92.415327|3659|90.186826|2134|25364215|82326855|48019095
Rogue_ll|98.665875|33123|96.449077|3751|93.799547|2202|745274044|84406256|49539400

# Appendix Z Notes

How everything was done.

I suspect if I tried big bunny, a lot more of these results would have been expected, because that's the sort of content these things are tested with, whereas, Detroit, Flower, and Untitled Goose Game are not like most animations or films that netflix or x264 testing will have come across. These more unusual visual experiences are part of the draw of modern games.

There is a channel dedicated to this article where you can find all the uploads featured within. https://www.youtube.com/channel/UCwXGFTOhuMJ1jQzOXRx3Fbg

Original videos are too large to share.

Our actual gaming channel is https://www.youtube.com/c/AndSoBegins
