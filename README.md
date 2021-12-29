# weval-nginx-rtmp Dockerized and Optimized Streaming Server
This repo is a small collection of files used to build and run a dockerized NGINX-RTMP streaming server. The `Dockerfile` contains all specifics for building the image and the `docker-compose.yml` is used to quickly build and or startup a running container of the image.

## Building & Starting the Streaming Server Image
1. cd into the top level of `weval-nginx-rtmp`
2. Run and build image based off the `docker-compose.yml` fike run the following: `docker-compose up`
    * If you want to be detatched from the running container of the build image add `-d` or `--detatch` to the above command

**NOTE**
* If you make changes to anything in directory you can quickly rebuild an updated image with `docker-compose up --build`
* To quickl;y remove all the built of images run `docker system prune`


## Dockerfile
The docker file is based on a base ubuntu-18.04 version. At a high level it compeletes the following steps
1. Sets some env variables to use in the file.
2. Installs some packages that allow for the downloading, compiling and execution of nginx.
3. Downloads, unpacks and compiles nginx with specific configurations.
    - These configurations are essential for ensuring we are using the right executable and `nginx.conf` files as well.
4. Creates some symbolic links to forwards the logs through docker.
5. Copies the local nginx.conf file to the proper location.
6. Exposes ports and starts the server.

## nginx.conf
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
