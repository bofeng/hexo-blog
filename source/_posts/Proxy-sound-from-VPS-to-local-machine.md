---
title: Proxy sound from VPS to local machine
typora-root-url: ../../source
date: 2023-01-24 17:47:51
tags: [sound, linux]
---



## Problem

We want to play a sound on the remote Linux server, and use the web UI control / hear it



## Solution

We can achive this with following steps:

1. Create virtual sound card in VPS with `pulseaudio`
2. Use `ffmepg` to stream all data in the virtual sound card to a local UDP server. The data format can either be `pcm` or `mpegts`
3. Write a websocket server to proxy the UDP server data
4. In web side, we use javascript to connect to this websocket server, and use javascript to play the sound. If it is in PCM format, we can use this [pcm-player](https://github.com/samirkumardas/pcm-player), and if it is in MPEGTS format, we can use this [jsmpeg player](https://github.com/phoboslab/jsmpeg).



### Here are details:

1, Install `pulseaudio` and `ffmpeg`

```bash
$ apt install pulseaudio ffmpeg -y
```

2, Start `pulseaudio`, this will create a virtual sound card

```bash
$ pulseaudio --start --exit-idle-time=-1
```

Now if you use `pactl` to list current sound card, you should be able to see something like this:

```bash
$ pactl list short sources
0	auto_null.monitor	module-null-sink.c	s16le 2ch 44100Hz	IDLE
```

The above shows at index 0 we have a virtual sound card "auto_null".

3, Use `ffmpeg` to stream sound data to udp:

= 1) If use `pcm` format:

```bash
ffmpeg -f pulse -i 0 -f s16le -acodec pcm_s16le -ac 1 -ar 8000 udp://localhost:1234
```

this command will read data from sound card at index 0, encode data to pcm_s16le, encode with 1 channel (-ac), at sample rate 8000 (-ar) , then stream this data to local UDP server localhost:1234

= 2) If use `mpegts:mp2` format:

```bash
ffmpeg -f pulse -i 0 -f mpegts -codec:a mp2 -b:a 128k -muxdelay 0.001 udp://localhost:1234
```

this command will read data from sound card, encode to mpegts format with audio as mp2, then dump this data to local UDP server localhost:1234

**Note:** If you just want to debug if the virtual sound card works, you can stream the data from the card and save it to a mp3 file:

```bash
# in one terminal, start ffmpeg
$ ffmpeg -f pulse -i 0 -f mp3 out.mp3
# then in another terminal, play some sound
# apt install mpg123
$ mpg123 piano.mp3
# once the playing is done, go to the first terminal
# you will see the out.mp3 file, then download it to your local computer,
# play it to see if the sound is correct.
```

4, Run a UDP-to-Websocket server. This server will listen on two ports: a UDP port and a TCP port (for websocket). It will dump the data it received from the UDP port to all connected websocket clients.

```bash
$ ./udp2ws -wsaddr 0.0.0.0:6080 -udpaddr 127.0.0.1:1234 -data binary
```

The flow is, once it received data from ffmpeg on the UDP port (:1234 above), it will format the UDP binary data to websocket data frame with the "binary" data type, and broatcast the data to all connected websocket client connections.

You can find the [udp2ws source code here >>>](https://github.com/bofeng/udp2ws)

5, Use javascript in web to connect to this websocket server to play sound:

= 1) If use `pcm` format:

Download the player javascript file from https://github.com/samirkumardas/pcm-player/blob/master/pcm-player.min.js to static folder, then in html add code:

```html
<div>
    <h1>Click here to enable sound</h1>
</div>
<script type="text/javascript" src="/static/pcm-player.min.js"></script>
<script>
 window.onload = function() {
   var socketURL = 'ws://ws-server-ip:6080/';
   var player = new PCMPlayer({
        encoding: '16bitInt',
        channels: 1,
        sampleRate: 8000
   });

   var ws = new WebSocket(socketURL);
   ws.binaryType = 'arraybuffer';
   ws.addEventListener('message',function(event) {
        var data = new Uint8Array(event.data);
        player.feed(data);
   });
 }
</script>
```

= 2) If use `mpegts:mp2` format:

Download the player javascript file from https://raw.githubusercontent.com/phoboslab/jsmpeg/master/jsmpeg.min.js to static folder, then in html add code:

```html
<div>
    <h1>Click here to enable sound</h1>
    <canvas id="video-canvas"></canvas>
</div>
<script type="text/javascript" src="/static/jsmpeg.min.js"></script>
<script type="text/javascript">
    var canvas = document.getElementById('video-canvas');
    var url = 'ws://ws-server-ip:6080/';
    var player = new JSMpeg.Player(url, {
        canvas: canvas,
        video: false,
        maxAudioLag: 2,
        pauseWhenHidden: false
    });
</script>
```

Now if you play some sound in the server side, like this:
```bash
$ apt install mpg123
$ mpg123 piano.mp3
```

Then in the browser side you should be able to hear the sound. Due to the internet connection, there might be several seconds delay.



## Reference

* https://github.com/novnc/noVNC/issues/302#issuecomment-864419873
* https://github.com/novnc/noVNC/issues/302#issuecomment-1225449899
* https://github.com/phoboslab/jsmpeg
* https://github.com/samirkumardas/pcm-player
* https://www.golinuxcloud.com/golang-udp-server-client/
