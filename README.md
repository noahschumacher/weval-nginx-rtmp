# weval-nginx-rtmp Dockerized and Optimized Streaming Server
This repo is a small collection of files used to build and run a dockerized NGINX-RTMP streaming server. The `Dockerfile` contains all specifics for building the image and the `docker-compose.yml` is used to quickly build and or startup a running container of the image. The specifics of the nginx server are located in `nginx.conf`.

Out of the box this docker images comes with:
* Streamer Validation Capabilities
* Optimized Stream Encoding
* Adaptive Bitrate Streaming
* HTTP - HLS server

This was originally designed for a live audio-video music streaming website I was building called Muuse. At its peak we were able to handle 15+ concurrent video streamers with 1.5k+ concurrent viewers and sub 12second latency.

## Quick-Start
1. Download OBS
2. Open OBS and set the following in _____:
    * *Service*: Custom
    * *Server*: `rtmp://<ip-of-host>:1935/live`
    * *Stream Key*: [paste in your stream key] (use `test` if not checking)
3. Start the docker-container:
    * `cd` into the top level of `weval-nginx-rtmp`
    * Run and build image with `docker-compose.yml` with `docker-compose up`
      * If you want to be detatched from the running container of the build image add `-d` or `--detatch` to the above command
4. Press `Start Streaming` under the control panel on the right side. You should see a green or red square show up on the bottom right side of OBS followed by a kb/s number. This number informs you of your upload speed. If the square is green you are streaming with good quality. If the square is red there might be some buffering during specific segments.
5. Download and open a [VLC](http://www.videolan.org/vlc/index.html) player (it also works in Raspberry Pi using `omxplayer`)
   * Click in the "Media" menu
   * Click in "Open Network Stream"
   * Enter the URL from above as `rtmp://<ip_of_host>:8080/live/<key>` replacing `<ip_of_host>` with the IP of the host in which the container is running and `<key>` with the key you created in OBS Studio. For example: `rtmp://192.168.0.30:8080/live/test`
   * Click "Play"!

You should stream your live stream coming through with some latency!

*Note: If your computer is on the older side of the device you are running does not have a fairly powerful CPU it may struggle to handle encoding 3 different AV qualities. If this is the case there will be severe latency and choppyness.*


## `Dockerfile`
The docker file is based on a base ubuntu-18.04 version. At a high level it compeletes the following steps
1. Sets some env variables to use in the file.
2. Installs some packages that allow for the downloading, compiling and execution of nginx.
3. Downloads, unpacks and compiles nginx with specific configurations.
    - These configurations are essential for ensuring we are using the right executable and `nginx.conf` files as well.
4. Creates some symbolic links to forwards the logs through docker.
5. Copies the local nginx.conf file to the proper location.
6. Exposes ports and starts the server.

## `nginx.conf`
The nginx.conf files is already highly commented to be very clear as to what each section is doing as well as how it can be altered to meet your needs. **Note: this will most likely need some initial tweaking to meet your applications needs and specifications**. At a high level this configuration does the following.
* Specifies 4 workers each with 10000 worker connections
* `rtmp`:
  * The rtmp ingest server listens on port 1935
  * `application live`:
    * `on_publish` and `on_publish_done` specify the endpoint to send information about a new incoming stream and a ending stream. This is useful if server protection is required.
    * Encoding:
      * In this configuration we using ffmpeg to encode 3 different qualities.
      * There is a lot of subtleties in this ffmpeg command and if it is not setup correctly it can lead to large latencies. For instance setting `keyint=60:no-scenecut` was critical for me to achieve <12s latency.
      * The names of each encoding must align with the `hls_variants`
  * `application_show`:
    * This directive specifies output parametes such as the fragment and platlist length, path to the manifests, etc.
    * The `hls_variant` have a name and bandwidth. The bandwith is the minimum bitrate a client needs to view the variant. It enables the client player to utilize Adaptive Bitrate Streaming.
3. `http`:
  * This directive is where client players can obtain hls, dash, or more.
  * `server`:
    * In this configuration we are only enabling hls live streaming.


## Further OBS Optimizations
By aligning OBS settings and the desired encoding we can further reduce compute, and latency.


The broadcast software is what takes in audio and video and sends it your streaming server. The most common software for this is called <a target="_blank" rel="noreferrer" href="https://obsproject.com/">Open Broadcast Software (OBS)</a>. OBS is an incredibly powerful tool, and it’s **free**! <a target="_blank" rel="noreferrer" href="https://streamlabs.com/">Streamlabs OBS</a> is another free option that is basically OBS with a little larger feature sweet built out. There are some paid options such as Wirecast and VidBlasterX. Though, for almost all cases, OBS/StreamlabsOBS is all you will need. The rest of this tutorial assumes you are using OBS or Streamlabs OBS as they generally the same layout and setting structure.
* To download OBS follow the <a target="_blank" rel="noreferrer" href="https://obsproject.com/">guide</a> for your operating system.*
* To download Streamlabs OBS click <a target="_blank" rel="noreferrer" href="https://streamlabs.com/">here</a>.*


### Configuration - *these were the setting we recommended for our service; yours may differ*.
*Please read through each step. Pressing Apply in each step will save your changes but is not required.*
* **Open OBS**. If you have not downlaoded OBS or Streamlabs OBS please refer to the above section.
* **Cancel the Auto-Configuration Wizard**. If it is your first time opening OBS you will be prompted with the Auto-Configuration Wizard. Press cancel if you see this.
* **Navigate to the Settings**. You should see the settings button on the bottom right-hand side in the controls section.
* **Select the *Stream* section** on the left side menu panel. Set the following options:
    * *Service*: Custom
    * *Server*: `rtmp://<ip-of-host>:1935/live`
    * *Stream Key*: [paste in your stream key] (use `test` if not checking)
* **Select the *Output* section** on the left side menu.
* **Select *Output Mode*: Advanced**. Then set the following options in the *Streaming* tab.
  * *Audio Track*: 1
  * *Bitrate*: 3500. Generally this should be between 2500 - 6000 depending on your Resolution and Audio qualities.
  * *Keyframe Interval*: 2
  * **Audio tab**
    * *Audio Bitrate*: 192 or greater. If you are going to be streaming music we recommend 224+ to ensure high quality audio.
* **Select the *Audio* section** on the left side menu.
  * *Sample Rate*: 44.1kHz. Only select 48kHz if you know all source input audio is also at 48kHz
  * *Channels*: Stereo
* **Select the *Video* section** on the left side menu.
  * *Base Resolution*: 1280x720. This should match your output reslution. If it does match it can increase load on your computer.
  * *Output Resolution*: 1280x720. It can be either 1920x1080 or 1280x720 but we recommend 720p as is good quality and does not require nearly as much *bandwidth to support.
  * *FPS*: 30. We do not recommend 60fps as it requires extremely reliable and strong upload capabilities. Choosing 60fps can cause a portion of your audience *to be unable to view your stream and is not currently recommended.
* **Select the *Advanced* section** on the left side menu.
  * Scroll down until you see *Automatically Reconnect*. Set the following options
    * *Enable*: selected
    * *Retry Delay*: 2s
    * *Maximum Retries*: 15
* **Press okay** to save your changes.
* **Start the stream!** Just press *start streaming* under the control panel on the right side. You should see a green or red square show up on the bottom right side of OBS followed by a kb/s number. This number informs you of your upload speed. If the square is green you are streaming with good quality. If the square is red there might be some buffering during specific segments.

For a more complete intro to OBS this <a target="_blank" rel="noreferrer" href="https://www.youtube.com/watch?v=Muk9LfEWHeU&ab_channel=bai">video</a> is great.

## Viewing the stream with VLC
1. Open a [VLC](http://www.videolan.org/vlc/index.html) player (it also works in Raspberry Pi using `omxplayer`)
2. Click in the "Media" menu
3. Click in "Open Network Stream"
4. Enter the URL from above as `rtmp://<ip_of_host>:8080/live/<key>` replacing `<ip_of_host>` with the IP of the host in which the container is running and `<key>` with the key you created in OBS Studio. For example: `rtmp://192.168.0.30:8080/live/test`
5. Click "Play"