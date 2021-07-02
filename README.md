# Markut

Given the [VOD](https://help.twitch.tv/s/article/video-on-demand) of the stream and the [markers](https://help.twitch.tv/s/article/creating-highlights-and-stream-markers) that are exported as a [CSV](https://en.wikipedia.org/wiki/Comma-separated_values) file, generate a video using [ffmpeg](https://www.ffmpeg.org/) that cuts out part of the VOD according to the provided markers.

![thumbnail](https://i.imgur.com/shk7eqG.png)

## Quick Start

Install [Go](https://golang.org/) and [ffmpeg](https://www.ffmpeg.org/).

```console
$ go build markut.go
$ ./markut final -csv marks.csv -input vod.mp4 -delay 4
```

### Input arguments

1. `-csv marks.csv` - CSV file with the first field containing timestamp of the cut in seconds starting from the beginning of the video. The amount of marks must be even as they form video chunks. See the screenshot at the top.
2. `-input vod.mp4` - the input video file that is chopped into chunks.
3. `-delay 4` - the offset in seconds added to every mark in `marks.csv`. If the marks are imported from Twitch this might be very useful because Twitch marks have around 4 seconds lag.

### Output files

1. Sequence of chunks `chunk-00.mp4`, `chunk-01.mp4`, etc. The amount of chunks depends on the amount of marks in `marks.csv`
2. `ourlist.txt` - instruction file to concatenate all the chunks. https://trac.ffmpeg.org/wiki/Concatenate
3. `output.mp4` - final concatenation of the chunks.

## Adjusting the `-delay` with `markut chunk`

With `markut chunk` instead of `markut final` you can render only one chunk of the final video to see if the delay is aligned properly and adjust the delay as needed:

```console
$ ./markut chunk -csv marks.csv -input vod.mp4 -delay 4 -chunk 0
```

After the delay is chosen you can use `markut final` to render the final video:

```console
$ ./markut final -csv marks.csv -input vod.mp4 -delay 4
```

It will try to rerender some of the existing chunks generated by previous `markut chunk` calls making ffmpeg complain about already existing files. We intentionally don't pass `-y` to ffmpeg to let you answer `N` in case you are ok with using already existing chunks in the final video.

## Ignored Markers

If the second field of the CSV file with markers contains the string "ignore" the marker is not considered as part of the chunk end.
