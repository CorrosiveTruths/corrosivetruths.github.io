---
layout: post
title:  "VMAF comparisons for 2160 upscaled Content on YouTube"
date:   2020-09-22 12:50:00 +0100
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
- 4k
- 2160
- 2160p
---
* TOC
{:toc}

# Introduction

The purpose of these tests was to disprove the notion that uploading higher than 1080 content would give better quality results for the 1080 version of an upload than just uploading at 1080 by showing the vp9 version where supported instead of the h264 version. Turned out it does actually work. The turning point for me was the observation that some channel's videos show you the vp9 version and some the h264, yet Youtube always encodes to both h264 and vp9 codecs. Since I did a bunch of vmaf comparisons of video for the previous article, [Encoding for Youtube]({% post_url 2020-03-08-encoding-for-youtube %}) - I thought I would do something similar for 2160 upscales of the same source material.

# Capture

Nine games were chosen, from slow-paced 30fps games with flat block-colour menus to a 60fps colourful racing game as well as some games I thought would be a challenge for both the encoders and the quality assessment.

* Death Stranding
* Detroit: Become Human
* Diablo 3
* Flower
* Forza Horizon 4
* Untitled Goose Game
* Persona 4
* Resident Evil 7
* Rogue Legacy

These representative 3-minute clips will be referred to by shorthand names: Death Detroit Diablo Flower Forza Goose Persona RE7 Rogue.

Capturing is further detailed in the [previous article]({% post_url 2020-03-08-encoding-for-youtube %}). Whilst transcoding is lossless, the initial captures are not true lossless for practicality's sake as they are limited range, yuv420p and were in 60000/1001fps.

# Quality

VMAF Blurb: VMAF (Video Multi-Method Assessment Fusion) is a perceptual video quality assessment algorithm developed by Netflix. VMAF Development Kit (VDK) is a software package that contains the VMAF algorithm implementation, as well as a set of tools that allows a user to train and test a custom VMAF model. Read [this](https://medium.com/netflix-techblog/toward-a-practical-perceptual-video-quality-metric-653f208b9652) techblog post for an overview, or [this](https://medium.com/netflix-techblog/vmaf-the-journey-continues-44b51ee9ed12) post for the latest updates and tips for best practices.

Basically, vmaf is a rating for comparing a distorted video to its original and telling you how close it is. It is an objective measure of how close the video matches, one that takes many methods into account  and is specifically for video. The scale itself is more subjective and based upon how real humans would rate the content.

# 2160 Transcoding

I thought it would be interesting to upscale with (ffmpeg default is bicubic) and without filters (nearest neighbour). The x265 encoder was used at the slow preset at crf 18 (constant rate factor, going from 0, lossless, to 51, awful). The last set of tests showed that using lossless doesn't really affect the final vmaf rating from Youtube, so this time only the crf version was used, and is more feasible to upload outside of testing. The slow preset is just so it encodes in a reasonable time across the network.

```bash
parallel --nice 20 --eta -S node1,node2,node3 -j1 ffmpeg -i {}_ll.mkv -crf 18 -c libx265 -preset slow -vf scale=3840x2160:flags=bicubic {}_265_s_18_4k.mkv ::: Death Detroit Diablo Flower Forza Goose Persona RE7 Rogue
parallel --nice 20 --eta -S node1,node2,node3 -j1 ffmpeg -i {}_ll.mkv -crf 18 -c libx265 -preset slow -vf scale=3840x2160:flags=neighbor {}_265_s_18_4kn.mkv ::: Death Detroit Diablo Flower Forza Goose Persona RE7 Rogue
```

The naming convention is just the game name, video type (265), crf value (18) and 4k for default upscaling (bicubic) and 4kn for upscaling with none or (nearest) neighbor.

Despite working with upscaled videos we can still perform vmaf comparisons by simply running the reference lossless videos through the same upscale filter when comparing. We even have a new comparison to make since vmaf provides a 4k (2160) model (vmaf_4k_v0.6.1) as well as the usual 1080 (vmaf_v0.6.1) one.

> Most recently, we added a new 4K VMAF model which predicts the subjective quality of video displayed on a 4K TV and viewed from a distance of 1.5H. A viewing distance of 1.5H is the maximum distance for the average viewer to appreciate the sharpness of 4K content. The 4K model is similar to the default model in the sense that both models capture quality at the critical angular frequency of 1/60 degree/pixel. However, the 4K model assumes a wider viewing angle, which affects the foveal vs peripheral vision that the subject uses.

*[VMAF: The Journey Continues](https://netflixtechblog.com/vmaf-the-journey-continues-44b51ee9ed12)*

To give a little context, H is height of device, and for the default model, it's a 1080 display viewed at 3H.

# How did 1080 go?

|Name|Transcode VMAF|Transcode Rate (kb/s)|YT.264 VMAF|YT.264 Rate (kb/s)|
|---|---|---|---|---|
|Death|99.023438|13962|57.284986|4242|
|Detroit|97.94216|6447|67.874461|3490|
|Diablo|96.761145|10799|64.274843|5477|
|Flower|96.238418|11387|61.241824|5118|
|Forza|97.810374|26326|52.031377|5401|
|Goose|96.301997|1695|75.384589|2565|
|Persona|97.644543|3768|84.632011|2517|
|RE7|96.152931|5012|64.376164|4227|
|Rogue|97.675072|3067|88.606968|5037|

![1080 Transcode & Upload]({{site.url}}/assets/images/1080Transcode_Upload.svg "1080 Transcode & Upload")  
_1080 Transcode & Upload_

These are the realistic results from last time, I do use this encode to produce video, although I don't start with lossless, but quicksync's highest quality. This is what happens when you transcode from lossless with x265, crf18, using the slow preset at 1080p and then upload to Youtube.

The videos you are presented with when you watch Youtube are in h264. The vp9 versions are generated, but not shown. As expected they are both lower bitrate and higher quality which is why vp9 is so desirable (and begs the question of why google doesn't always show the vp9 version of every video where supported).

# 2160 vmaf and bitrates

Let's see how the 2160 versions turned out.

Here's how they we got the vmaf numbers from the transcodes (4k & 4kn) and from the downloaded youtube files (using youtube-dl) respectively.

```bash
parallel --nice 20 --eta -j1 'ffmpeg -i /mnt/LPWorking/vmaf/{}_265_s_18_4k.mkv -i /mnt/LPWorking/vmaf/{}_ll.mkv -filter_complex "[1]scale=3840x2160:flags=bicubic[ref];[0][ref]libvmaf=model_path=/usr/share/model/vmaf_4k_v0.6.1.pkl" -f null -t 180 /dev/null' ::: Death Detroit Diablo Flower Forza Goose Persona RE7 Rogue &> /mnt/LPWorking/vmaf/vmafscores_2160-2160.txt
parallel --nice 20 --eta -j1 'ffmpeg -i /mnt/LPWorking/vmaf/{}_265_s_18_4kn.mkv -i /mnt/LPWorking/vmaf/{}_ll.mkv -filter_complex "[1]scale=3840x2160:flags=neighbor[ref];[0][ref]libvmaf=model_path=/usr/share/model/vmaf_4k_v0.6.1.pkl" -f null -t 180 /dev/null' ::: Death Detroit Diablo Flower Forza Goose Persona RE7 Rogue &> /mnt/LPWorking/vmaf/vmafscores_2160-2160n.txt
parallel --nice 20 --eta -j1 'ffmpeg -i /mnt/LPWorking/vmaf/yt/{}_265_s_18_4k-2160.webm -i /mnt/LPWorking/vmaf/{}_ll.mkv -filter_complex "[1]scale=3840x2160:flags=bicubic[ref];[0][ref]libvmaf=model_path=/usr/share/model/vmaf_4k_v0.6.1.pkl" -f null -t 180 /dev/null' ::: Death Detroit Diablo Flower Forza Goose Persona RE7 Rogue &> /mnt/LPWorking/vmaf/yt/vmafscores_4k_vp9-2160-2160.txt
parallel --nice 20 --eta -j1 'ffmpeg -i /mnt/LPWorking/vmaf/yt/{}_265_s_18_4kn-2160.webm -i /mnt/LPWorking/vmaf/{}_ll.mkv -filter_complex "[1]scale=3840x2160:flags=neighbor[ref];[0][ref]libvmaf=model_path=/usr/share/model/vmaf_4k_v0.6.1.pkl" -f null -t 180 /dev/null' ::: Death Detroit Diablo Flower Forza Goose Persona RE7 Rogue &> /mnt/LPWorking/vmaf/yt/vmafscores_4k_vp9-2160-2160n.txt
```

|Name|2160-2160 VMAF|Trans Rate (kb/s)|2160-2160 YT.vp9 VMAF|2160-2160 YT.vp9 Rate (kb/s)|
|---|---|---|---|---|
|Death_265_s_18_4k|99.631391|34757|88.30421|13366|
|Death_265_s_18_4kn|99.092667|40346|87.80756|13542|
|Detroit_265_s_18_4k|99.06393|28424|92.226379|9982|
|Detroit_265_s_18_4kn|98.51933|26449|91.717363|10317|
|Diablo_265_s_18_4k|99.396409|26308|93.658602|17674|
|Diablo_265_s_18_4kn|99.037009|30518|93.195276|18916|
|Flower_265_s_18_4k|97.879916|30788|92.673437|17500|
|Flower_265_s_18_4kn|96.749556|33642|91.738545|17982|
|Forza_265_s_18_4k|99.154993|61127|86.989702|22880|
|Forza_265_s_18_4kn|98.742877|69084|86.440331|23332|
|Goose_265_s_18_4k|98.723693|4099|93.148316|3544|
|Goose_265_s_18_4kn|98.46808|6880|93.093757|5599|
|Persona_265_s_18_4k|99.781146|9372|96.129109|5695|
|Persona_265_s_18_4kn|99.755002|11047|96.165361|6529|
|RE7_265_s_18_4k|98.350062|14046|90.456835|9482|
|RE7_265_s_18_4kn|97.564546|16557|89.861327|10323|
|Rogue_265_s_18_4k|99.816733|6971|99.145447|6291|
|Rogue_265_s_18_4kn|99.837137|11038|99.074572|8689|

![4k Transcode & Upload]({{site.url}}/assets/images/4kTranscode_Upload.svg "4k Transcode & Upload")  
_4k Transcode & Upload_

The transcodes this time have higher vmaf scores, with big increases in bitrate. The increase in bitrate I was expecting - we are storing quadruple resolution video after all, although I was surprised at the unfiltered upscaling, expecting it to make uniformly smaller files rather than usually bigger.

When you ignore the transcoded size and just look at the default 1080 versus the default 2160 and also add an average the difference in 1080 and upscaled 2160 on Youtube becomes more apparent.

|Game|4k2160 to vp9 vmaf|4k2160 to vp9 rate|1080 to 264 vmaf|1080 to 264 rate|
|---|---|---|---|---|
|Death|88.922144|13366|57.284986|4242|
|Detroit|91.718805|9982|67.874461|3490|
|Diablo|91.723392|17674|64.274843|5477|
|Flower|96.413496|17500|61.241824|5118|
|Forza|88.858607|22880|52.031377|5401|
|Goose|91.507133|3544|75.384589|2565|
|Persona|94.266461|5695|84.632011|2517|
|RE7|89.171265|9482|64.376164|4227|
|Rogue|97.913176|6291|88.606968|5037|
|Average|92.27716433|11824|68.41191367|4230|

![4k Transcode & 1080 Upload Comparison]({{site.url}}/assets/images/4kTranscode_1080Upload_Comparison.svg "4k Transcode & 1080 Upload Comparison")  
_4k Transcode & 1080 Upload Comparison_

2160 on YouTube doesn't just provide enough extra bitrate to provide a similar quality to 1080 at an increased resolution, it provides enough to bump up the average vmaf score from 68 to 92, to the point that you have to look closely and know where to look to distinguish the difference between Youtube and upload in some cases.

# YouTube downscaling

When you upload a video, it isn't available in just the resolution you upload, but also downscaled into different resolutions for viewing. To find out what kind of downscaling Youtube uses you can compare the 1080p versions of your 2160p uploads with an ffmpeg downscaled version of that same 2160p Youtube upload. No need for vmaf comparisons here, the obvious result was lanczos when placed one on top of the other. I could have used lanczos instead of the default, bicubic, for upscaling too, (in fact this has become my default approach).

This means when comparing 2160p content to 1080p, the comparison of vmaf numbers is a bit fairer by using the same 1080 model and emulating the same sort of downscale that Youtube itself would perform.

Luckily, the vmaf scores are comparable - a file upscaled from 1080 to 2160 will have a _very_ similar vmaf score (under 1.7% different) in the 4k model to that same file scaled back down to 1080 in the default 1080 model.

More usefully, downscaling to 1080 can also be used to test youtube's different resolution transcodes. So not only can we compare 2160 and 1080, but also 1440.

# 2160, 1440, 1080 vmaf

So here's how the youtube video's 1440 resolution videos were compared once downloaded via youtube-dl (other versions skipped for brevity).

```bash
parallel --nice 20 --eta -S Soma,Stardew,Portal -j1 'ffmpeg -i /mnt/LPWorking/vmaf/yt/{}_265_s_18_4k-1440.webm -i /mnt/LPWorking/vmaf/{}_ll.mkv -filter_complex "[0]scale=1920x1080:flags=lanczos[main];[1]scale=3840x2160:flags=bicubic[up];[up]scale=1920x1080:flags=lanczos[ref];[main][ref]libvmaf=model_path=/usr/share/model/vmaf_v0.6.1.pkl" -f null -t 180 /dev/null' ::: Death Detroit Diablo Flower Forza Goose Persona RE7 Rogue &> /mnt/LPWorking/vmaf/yt/vmafscores_4k_vp9-1440-1080_scaled.txt
parallel --nice 20 --eta -S Soma,Stardew,Portal -j1 'ffmpeg -i /mnt/LPWorking/vmaf/yt/{}_265_s_18_4kn-1440.webm -i /mnt/LPWorking/vmaf/{}_ll.mkv -filter_complex "[0]scale=1920x1080:flags=lanczos[main];[1]scale=3840x2160:flags=neighbor[up];[up]scale=1920x1080:flags=lanczos[ref];[main][ref]libvmaf=model_path=/usr/share/model/vmaf_v0.6.1.pkl" -f null -t 180 /dev/null' ::: Death Detroit Diablo Flower Forza Goose Persona RE7 Rogue &> /mnt/LPWorking/vmaf/yt/vmafscores_4k_vp9-1440-1080n_scaled.txt
```

|Name|"1080 YT.vp9 VMAF"|"1080 YT.vp9 Rate (kb/s)"|"1440-1080 YT.vp9 VMAF"|"1440 YT.vp9 Rate (kb/s)"|"2160-1080 YT.vp9 VMAF"|"2160 YT.vp9 Rate (kb/s)"|
|---|---|---|---|---|---|---|
|Death_265_s_18_4k|72.264176|2382|85.825568|7339|88.922144|13366|
|Death_265_s_18_4kn|71.148515|2363|85.365538|7274|88.329157|13542|
|Detroit_265_s_18_4k|86.224406|1549|91.167497|5199|91.718805|9982|
|Detroit_265_s_18_4kn|84.761498|1508|90.855967|5162|91.383948|10317|
|Diablo_265_s_18_4k|79.263833|3437|90.008917|10202|91.723392|17674|
|Diablo_265_s_18_4kn|77.8263|3411|89.448375|10274|91.204057|18916|
|Flower_265_s_18_4k|82.283623|3244|93.480704|9669|96.413496|17500|
|Flower_265_s_18_4kn|81.132132|3161|93.064839|9575|95.783452|17982|
|Forza_265_s_18_4k|70.939047|4176|83.72851|11200|88.858607|22880|
|Forza_265_s_18_4kn|69.784144|4134|83.251745|11210|88.021732|23332|
|Goose_265_s_18_4k|85.233304|650|90.33181|1626|91.507133|3544|
|Goose_265_s_18_4kn|84.380214|660|90.401845|1769|91.553551|5599|
|Persona_265_s_18_4k|90.565604|1228|93.979172|3427|94.266461|5695|
|Persona_265_s_18_4kn|89.361618|1246|93.895276|3600|94.260319|6529|
|RE7_265_s_18_4k|79.695647|1543|87.706611|5469|89.171265|9482|
|RE7_265_s_18_4kn|78.559084|1508|87.21628|5485|88.701008|10323|
|Rogue_265_s_18_4k|96.160856|1936|97.730732|3723|97.913176|6291|
|Rogue_265_s_18_4kn|95.042382|1957|97.703936|4152|97.911519|8689|

![4k Upload Allres Comparison]({{site.url}}/assets/images/4kUpload_Allres_Comparison.svg "4k Upload Allres Comparison")  
_4k Upload Allres Comparison_

As you can see, 1440p is a reasonable compromise, needing smaller files, but still producing significantly higher vmaf scores at 1080 and 1440 than 1080 on its own.

# Conclusion

For new Youtube uploaders, it's up to them to decide if the significant increase in upload size is worth the increase in fidelity at 1080p that you get from getting the vp9 to show by default, and even then, that vp9 version _may_ not look as good as an established channel's. Both new channels and established channels can benefit from the higher achievable fidelity at above 1080p resolutions. It isn't just people with 4k hardware that benefit though, it's trivial on PCs at least to select 2160p and have Youtube downscale the video to their display.

This will let you view higher quality 1080 on Youtube. It isn't just quantisation errors that disappear though - viewing a 2160p Youtube video downscaled to your 1080p display also lets you see a video that effectively has no chroma subsampling since the yuv420p that Youtube (and everywhere else) uses quarter-resolution colour, and 2160p to 1080p is a quarter of the resolution, you get full colour resolution in 1080p.

Saying that, image quality isn't everything, and it's strange seeing Youtube embrace the very lastest in video technology, crazy resolutions like 8k, mind-numbingly expensive to encode av1, yet not implementing decades old sound technology as whilst you can upload a 5.1 surround track, you can't listen to a video in it. The only way you can get 5.1 surround out of Youtube is using Dolby Pro Logic II for which there is no full-featured free encoder.

At least sound quality is a solved issue as opus@160kbps is pretty much transparent.

# More to do

Repeat but with lanczos filter.

It would also be nice to re-upload the test files to a larger channel to find out what differences there are other than default codec to show a viewer. I can't attribute the slightly higher vmaf and bitrate to the upscaling process unless I can compare to an upload that directly shows the vp9 version by default when a 1080 video is uploaded.

av1? 8k? sub 1080p?

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

# Appendix Z Notes

There is a channel dedicated to this article where you can find all the uploads featured within. <https://www.youtube.com/channel/UCwXGFTOhuMJ1jQzOXRx3Fbg>

Original videos are too large to share.

Our actual gaming channel is <https://www.youtube.com/c/AndSoBegins>
