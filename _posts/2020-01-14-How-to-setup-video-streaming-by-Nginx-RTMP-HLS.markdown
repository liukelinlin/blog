---
layout: post
title:  "How to setup video streaming by Nginx, RTMP and HLS"
date:   2020-01-14 16:21:17 -0500
categories: Media
---
The tutorial will show you how to use Nginx、Nginx-RTMP-module、RTMP and HLS to set up a video streaming demo locally. 
We prefer to use Docker container technology to set up the environment and libs required quickly.

From [Apple official website](https://developer.apple.com/documentation/http_live_streaming/understanding_the_http_live_streaming_architecture), the architecture of HLS looks like as follow:

![HLS-arch](https://liukelinlin.github.io/images/hls_archicture.png)

It mainly includes the following steps:

- Video/Audio streaming acquisition
- AV Input based on RTMP
- Media encode and segment
- Distribution based on HLS
- Client app reassemble and present

The demo laptop:

```
MacBook Pro, Intel Core i5 2.3GHz x4, 16G RAM.
```

And `FFmpeg` installation is required, which is used to acquire streaming data from the camera and microphone.

The server container will be created by docker. Nginx config is customized (credit to TareqAlqutami): [nginx_no-ffmpeg.conf](https://github.com/TareqAlqutami/rtmp-hls-server/blob/master/conf/nginx_no-ffmpeg.conf),
and copy to "/tmp/nginx_no-ffmpeg.conf".

It is time to create the streaming server container by docker:

```
docker run -d -p 1935:1935 -p 8080:8080 -v /tmp/nginx_no-ffmpeg.conf:/etc/nginx/nginx.conf alqutami/rtmp-hls
```

After the container is up, we use FFmpeg to acquire video/audio streaming data and send to server RTMP port 1935.

```
ffmpeg -f avfoundation -framerate 30 -i "0" -c:v libx264 -an -f flv rtmp://127.0.0.1:1935/live/test
```

Now, you can use the host browser to open and watch the streaming: http://127.0.0.1:8080/players/hls.html

![Streaming-image](https://liukelinlin.github.io/images/me.jpeg)

and get `stats` by: http://127.0.0.1:8080/stat

![Streaming-stats](https://liukelinlin.github.io/images/hls_live_streaming_stats.jpg)
