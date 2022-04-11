# HOWTO capture VHS video using Pinnacle Dazzle DVC100 on Linux

## Software and hardware preparations

Install `alsa-utils`, `v4l-utils` and `ffmpeg` packages which contain tools to
find and setup Linux video and audio devices. I can also recommend install
`vlc` video player for checking results but you can use your favorite one. To
cut a random noise between scenes and split the resulting video interactively I
would recommend using `avidemux`.

Pinnacle Dazzle DVC100 on Linux is supported by `em28xx` kernel module. I have
a device with id `2304:021a`. You can check if the device is visible using
`lsusb` command after connecting the device to the USB port:
```
$ lsusb | fgrep DVC100
Bus 001 Device 025: ID 2304:021a Pinnacle Systems, Inc. Dazzle DVC100 Audio Device
```

Use `udevadm info` command from `udev` package to check whether V4L subsystem
properly configured the device. The following command will find the name of the
device created by V4L:
```
udevadm info  /dev/video* | egrep 'DEVNAME|ID_V4L_PRODUCT' | fgrep -B 1 'Dazzle'
```

Use following command to find proper audio device:
```
$ cat /proc/asound/cards | fgrep DVC100
 1 [DVC100         ]: USB-Audio - DVC100
                      Pinnacle Systems GmbH DVC100 at usb-0000:05:00.4-1, high speed
```
First column contains the number of ALSA card. It should be combined
with `hw:` prefix to form the name of ALSA device. For the example above the
name of the device is `hw:1`.

For a sake of simplicity export names of the video and audio devices as
environment variables:
```
export VIDEO=/dev/video0
export AUDIO=hw:1
```

Set PAL as a video standard, proper frame size and unmute an audio capture
device using `v4l2-ctl`:
```
v4l2-ctl --device $VIDEO \
	--set-standard 5 --set-fmt-video=width=720,height=576 \
	--set-ctrl mute=0 --set-ctrl volume=31
```

Check V4L settings are applied:
```
v4l2-ctl --device $VIDEO --get-standard --get-fmt-video --get-ctrl mute --get-ctrl volume
```

Use ALSA mixer to increase volume of the audio. Min volume is 0, max volume is
16. Default volume is 8 which corresponds to the 50% volume. The following
command increases it to 13 which corresponds to the 81% volume:
```
amixer -D $AUDIO sset Line 13
```

Check that video is captured properly using `ffplay`:
```
ffmpeg -ar 44100 -thread_queue_size 1024 -f alsa -i $AUDIO \
	-r 25 -thread_queue_size 1024 -i $VIDEO \
	-codec copy -f matroska  - | ffplay -
```

On this stage video may have:
- a blurred stripe at the bottom of the frame;
- black columns at the right and left sides of the frame;
- sound and video not synchonized properly;
- some small shift between odd and even lines of the frame especially on
  dynamic scenes.

Such inconsistencies depend on the video recorder/player/capturer in use and
can easily be removed during and after capturing. But the center of the
frame should contain a color picture, you should hear the sound and it should
not be too noisy.

Typical issues that should be fixed on this stage are using incorrect color
coding and incorrect sound rate. If picture is black and white or too noisy
then incorrect color coding system is used probably. There are three main
standards PAL, NTSC and SECAM. If it is unclear which one was used for the
record you can try switching between them:

```
v4l2-ctl --device $VIDEO --list-standards
v4l2-ctl --device $VIDEO --set-standard <standard>
```

Second typical issue is an incorrect sound capturing rate. If sound is to noisy
or FFmpeg complains about audio synchronization it may mean sound capturing
rate is incorrect. It is specified by `-ar` option of FFmpeg.  Typical values
are 44100 Hz and 48000 Hz. 44100 worked to me so it is used in FFmpeg
examples provided.

## Capturing video

To keep as much quality as possible it is better to split a capturing process
on two stages: fast lossless capturing and compressing.  The lossless capturing
is saving content using a lossless compression algorithm. Following command use
`ffv1` codec for video and `pcm_s16le` for audio. Couple of fast filters are
applied before saving in order to simplify postprocessing.

I have only single audio channel on my camera and two channels on Dazzle
capturing device. I connected camera audio output to the left channel but
capture two channels, so I used `channelmap` FFmpeg filter to leave only left
channel.

Filter `crop` is applied to remove blurred and black stripes from the video
frame. Four parameters are used:
- output frame width and height (`w` and `h`);
- horizontal and vertical position of the output frame inside input frame (`x`
  and `y`).

To determine the values capture one frame from the input video using VLC or
other video player and measure the width of stripes using picture viewer or
editor. In my case resulting values are: `crop=w=700:h=556:x=8:y=0`.

Check the overall flow using ffplay before capturing:
```
ffmpeg -ar 44100 -thread_queue_size 1024 -f alsa -i $AUDIO \
	-itsoffset -0.5 -r 25 -thread_queue_size 1024 -i $VIDEO \
	-filter_complex 'channelmap=FL-0' -filter_complex 'crop=w=700:h=556:x=8:y=0' \
	-map 1:v  -map 0:a -codec:v ffv1 -codec:a pcm_s16le \
	-f matroska  - | ffplay -
```

At this stage the picture and sound should be correct. Audio and video can
still be not synchronized but it is difficult to fix on this stage, so we will
fix it after capturing.

Last thing is to set priority of the capturing stream to maximum to allow
capturing with near realtime speed. It is not necessary if our system is not
loaded and have disk which is fast enough, but I used HDD on my home server to
capture so I used `ionice` and `nice` to prioritize the process over other
userspace processes on ther system.

Final command for capturing:
```
sudo ionice -c 1 -n 0 nice -n -20 ffmpeg \
	-ar 44100 -thread_queue_size 1024 -f alsa -i $AUDIO \
	-r 25 -thread_queue_size 1024 -i $VIDEO \
	-filter_complex 'channelmap=FL-0' -filter_complex 'crop=w=700:h=556:x=8:y=0' \
	-map 1:v  -map 0:a -codec:v ffv1 -codec:a pcm_s16le \
	raw.mkv
```

After capturing is done use `q` button to stop FFmpeg gracefully.

Watch the result of the capturing using video player and find the shift between
audio and video. It can be done with VLC using "Audio delay" feature. Use keys
`j` and `k` to slightly change audio delay while it is totaly synchronized. Use
some scene with talking people to do that. Then use the delay in seconds to
shift the audio track in the resulting video. For example if delay is -500ms
(audio is later than video) then use the following command:
```
ffmpeg -i raw.mkv -itsoffset -0.5 -i raw.mkv -map 0:v -map 1:a -codec copy shifted.mkv
```

I found that delay can be different for different capture sessions. So be
careful and alwasy check the synchronization after capturing.

You may watch the resulting video trim and split is on several fragments using
FFmpeg command like this:
```
ffmpeg -i raw.mkv -ss 00:00:00 -t 00:14:00 -codec copy trimmed.mkv
```

## Compression and post processing

Before final compression select the codecs and find out parameters which suits
your case best. I used `h264` and `mp3`. FFmpeg wiki has a nice guide on
selecting the parameters, see [H.264 Video Encoding
Guide](https://trac.ffmpeg.org/wiki/Encode/H.264) and [MP3 Encoding
Guide](https://trac.ffmpeg.org/wiki/Encode/MP3).

You need applying a filter which removes interlaced artifacts before
compression. I used `yadif`, see some related notes
[here](https://github.com/kfrn/ffmpeg-things/blob/master/deinterlacing.md).

`-pix_fmt yuv420p` is used in command below to make result compatible with
older players.

I also found a recommendation to use `-flags +ilme+ildct` in the FFmpeg FAQ.
But using these flags led to shaking video effects in dynamic scenes, so I
totally removed these flags.

The final command for compression:
```
ffmpeg -i shifted.mkv -filter:v yadif \
    -codec:a libmp3lame -q:a 5 \
    -codec:v libx264 -crf 26 -preset slow \
    -tune film -pix_fmt yuv420p compressed.mkv
```

Avidemux is a perfect tool for post compression cutting and splitting. It
allows interactively moving between key frames and momentarily remove pieces of
video before key frame without recompression.

## Links:
- [FFmpeg wiki](https://trac.ffmpeg.org/)
- [FFmpeg documentation](https://ffmpeg.org/ffmpeg.html)
- [H.264 Video Encoding Guide](https://trac.ffmpeg.org/wiki/Encode/H.264)
- [MP3 Encoding Guide](https://trac.ffmpeg.org/wiki/Encode/MP3)
- [FFmpeg FAQ](https://ffmpeg.org/faq.html)
- [VLC player](https://www.videolan.org/vlc/)
- [Avidemux video editor](http://avidemux.sourceforge.net/)
