# HOWTO erase miniDV tape on Linux using FireWire IEEE 1394 port

## Preparations

Get the frame size:
```
export FRAME=720x576
```

## Prepare black and silent video file

```
ffmpeg \
    -f lavfi -i color=size=${FRAME}:rate=25:color=black \
    -f lavfi -i anullsrc=channel_layout=stereo:sample_rate=44100 \
    -s ${FRAME} -pix_fmt yuv420p -r 25 \
    -t 600 empty.dv
```

## Write video by camera

```
cat empty.dv > /dev/dv1394
```

## Links

- [FFMPEG headers](https://www.ffmpeg.org/doxygen/1.0/dv1394_8h-source.html)
