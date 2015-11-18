## Sources
Original idea:
https://blog.jessfraz.com/post/docker-containers-on-the-desktop/

Base dockerfile:
https://github.com/jfrazelle/dockerfiles/tree/master/spotify

## Starting 
```
docker run -itd \
    -v /tmp/.X11-unix:/tmp/.X11-unix \
    -e DISPLAY=unix$DISPLAY \
    --device /dev/snd \
    --name spotify spotify
```
