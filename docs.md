---
layout: page
title: Docs
permalink: /docs/
---
# Recording mobile games

For this we use an old Motorola G4 with Lineage OS installed on it. This version of Lineage comes with a recorder. However, it will only record the device's microphone. So we use the phone's recorder to capture the game's video, and us talking. We use obs to record ourselves talking and also the game sound using the line-in port on the PC. This creates a lot of noise, which we remove using the same Noise Suppression (-30db) settings as with the vox.

Because we have all the ingredients out of sync, we can sync up the video from the device, and the audio from obs via the two recordings of our vox, one in the video from the device, and one in obs where the game sound also is.

# Transcoding Captured Footage

Transcoding takes an existing video file and makes another video file from it. We usually do this to make a file that seeks better and repair any problems caused by the original encoding.

For this we use ffv1 which is an ffmpeg native lossless encoder. It will generally not use more than 4 cores, so you can divide your cores by four and that's how many files are coded in parallel, this is what -j 25% does. Check out the GNU Parallel docs for more on the options used.

For 1080p files, this will generally produce a smaller file than h264, and at a faster speed. Context 1 produces slightly smaller files. The mapping is for putting the VOX as a separate file.

{% highlight shell %}
parallel --nice 19 --filter-hosts -j 25% --bar -S Soma,Stardew,Portal ffmpeg -i {} -vcodec ffv1 -context 1 -acodec flac /mnt/LPWorking/Source/{/} -map 0:2 /mnt/LPWorking/Source/{/.}.flac ::: /mnt/LPWorking/Source/ORIG/*.mkv
{% endhighlight %}

There are some special cases:

For <1080p, we'll just use h264 to get a smaller file; no need for job specification as h264 uses more cores:

{% highlight shell %}
parallel ffmpeg -i {} -acodec flac -qp 0 -tune fastdecode /mnt/LPWorking/Source/{/} ::: /mnt/LPWorking/Source/ORIG/*.mkv
{% endhighlight %}

When transcoding obs captured 60000/1001 footage, due to a bug in that software not writing the framerate into the video stream, we can specify it manually with the rate option placed before the input.

{% highlight shell %}
ffmpeg -r 60000/1001 -i input.mkv
{% endhighlight %}

# Multiplayer Game Editing

Lessons learned from Sea of Thieves

If you can embed character portrait stuff into the source videos via transcoding without blocking anything, do so, it's always good to have who is playing at any point on screen.

Set up tracks that have the effects pre-prepared, like left vertisplit, right vertisplit, that way you can move track segments onto the relevant track rather than adding or removing chains of filters to / from clips.

Keep the sound mono unless completely isolated.

# Encoding from Kdenlive

We need a few different encoding profiles. The main one is for actually producing episodes, for this we use VP9. It's slow, but makes very small files. These must be small as we'll be archiving them forever, and they must be high enough quality to not take a big hit when youtube transcodes them.

{% highlight shell %}
Profile Name: LP
Extension: webm
Parameters: glsl.=1 row-mt=1 cpu-used=1 crf=23 vb=0 ab=256k threads=8
{% endhighlight %}

Lossless encoding is great for in-project usage to make layers easier to manipulate.

{% highlight shell %}
Profile name: Transparent
Extension: mkv
Parameters: glsl.=1 properties=lossless/FFV1 
pix_fmt=bgra
{% endhighlight %}

Audio-only encoding is another part of the prcess which means no quality is lost when exporting bits and pieces, like edited sound to be mixed together outside of kdenlive.

{% highlight shell %}
Profile name: Transparent
Extension: mkv
Parameters: glsl.=1 properties=lossless/FFV1 
pix_fmt=bgra
{% endhighlight %}

The glsl.=1 bit is to make sure the composition is rendered with movit rather than plain ffmpeg as we find that a bug tends to make an overlaid layer tint green. Movit is a little slow and not very stable for editing however, so we just want it at the end of the process.

# Kdenlive Dos and Don'ts

Switch off Track compositing, it gives strange results, better off with affines or composites.

If you use a blank png as the base track, you can use the transparent rendering profile to make trans movies for use in projects for simplification.

Don't use fade to / from black, use dissolve instead (maybe path this out of local versions?)

# Audio Mastering

Recording is done at 70 +30db

There's a bug in mlt from .12 onwards that makes the audio embedded in video be flaky. Workarounds include dropping back to an earlier melt to export the sound from the project, or separating out your video's audio and video with ffmpeg as separate files and grouping them in kdenlive. You do want to do final render with a higher version as the green compositing bug was fixed.

Keep a log of anything you do outside of this in the Channel folder named after the game, i.e. Soul Reaver 2 recipe.

Note that the idea isn't to get an exact overall i of -23 as that's actually fairly quiet, the idea is to get a repeatable process that produces stuff at the same consistent volumes across LPs, but if r128 compliance is then later forced on the finished product, it should adjust to the new volume easily as it will just need to be lowered in volume, but the game track on its own, is.

1. Render audio only using the (au if using .10) flac profile, selecting full project and stem audio, which will automatically make an audio file for each separate track.
2. You can downmix the vox here with sox (sox input.wav output.wav channels 1) to speed up the process, or do it in audacity next.
3. Import each track into audacity for final audio mix.
4. Make the vox mono (tracks, mix, Mix Stereo down to mono)
5. Compress the vox (threshold -40, noisefloor -30, ratio 10:1, attack time 0.10, release time 1.0, make-up gain unticked, compress based on peaks untick)
6. Export just the vox track from audacity as 24bit flac
7. ffmpeg -i voxnom.flac -af ebur128=dualmono=1 -f null /dev/null
8. Work out the difference between I and -27 (i - 27)
9. Amplify vox by this.
10. Use the limiter (Soft, 10db gain, limit to -3db, hold 10ms, no make-up gain) on the vox to top up the volume.
11. Now do something similar for the game audio (again you can mix things with sox -m -v1 input -v1 input output) volumes. ffmpeg -i voxgme.flac -af ebur128 -f null /dev/null
12. Work out the difference between I and -23 (i - 23)
13. Amplify game to -23 so the vox is clear over the game when it is autoducked, but fairly loud when there is no talking. If the game is very quiet, and you can't amplify up, use the limiter to get to -23. This will need to done separately on video tracks from sources outside the game. Very rarely, you might need to adjust the video sound *before* you normalise the volumes, because one bit is an anomaly, or because intro videos are too loud. 14. Turning up or down individual sections or even soft limiting the entire video.
14. Duplicate the game sound so we have two of them, place it under the vox, so we have gme, vox, gme.
15. Autoduck the game sound, with the voice underneath, at -12 duck, with 1s pause, and outer fades of 0.5. Threshold should be -30db
16. Export the audio as selections to flac so you end up with the first two tracks as a mix, and the last track separate.