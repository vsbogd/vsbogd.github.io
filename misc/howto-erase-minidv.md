# HOWTO erase miniDV tape on Linux using FireWire IEEE 1394 port

## Preparations

```
ffmpeg --version
```

Get the frame size

## Prepare black and silent video file

```
ffmpeg -t 600 -s 720x576 -f rawvideo -pix_fmt yuv420p -r 25 -i /dev/zero empty.dv
```

```
ffmpeg \
    -f lavfi -i color=size=720x576:rate=25:color=black \
    -f lavfi -i anullsrc=channel_layout=stereo:sample_rate=44100 \
    -s 720x576 -pix_fmt yuv420p -r 25 \
    -t 600 empty.dv
```

## Write video by camera

```
cat empty.dv > /dev/dv1394
```

## Links

https://www.ffmpeg.org/doxygen/1.0/dv1394_8h-source.html
