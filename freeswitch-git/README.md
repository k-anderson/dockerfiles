# FreeSWITCH from Source (git)

## Files

### Dockerfile
This image is derived from a docker file originally built to follow the guide at
https://freeswitch.org/confluence/display/FREESWITCH/FreeSWITCH+1.6+Video
by ianblenke.  For the fork source see
https://github.com/ianblenke/docker-freeswitch-video
I have since modified it to build slightly differently and extended it somewhat
to suit my needs.

### Makefile
A convent shortcut for docker commands that came with the fork.  Too bad 
coreos doesn't have make!

### The build.sh
That file was present when I forked this docker repo.  I have no need for it
but it is such a nice build script I don't want to remove it ;)

### README.md
See [readme](README.md)

### image-files/modules.conf.in
This is placed in the FreeSWITCH build directory as modules.conf shortly
before the build.  It can be used to change the modules built into the image.

### image-files/fs-systemctl-tweaks.conf
This is placed into /etc/systemctl.d/ and contains the recommended kernel
tweaks from FreeSWITCH.  I doubt it is very effective on a docker image
but I have been starting it with privileged=true, so maybe?

### image-files/conf
This directory replaces the installed FreeSWITCH configuration directory
completely.

### image-files/cert
This directory replaces the FreeSWITCH cert directory completely.  It is
expected to contain:
* agent.pem
* cafile.pem
* wss.pem

## Purpose
This purpose of this image is to be a simple configuration of FreeSWITCH from source 
so that I can play/poke/tweak.  This is not intended to be a production ready 
image.

## Command Reference

### Building the image
From within the folder this repo was checked out to:

       docker build -t k-anderson/freeswitch-git .

* -t k-anderson/freeswitch-git: Repository name (and optionally a tag) for the image
* *path to folder with docker file*

### Starting FreeSWITCH
At the moment I have been starting it using:

	docker run -d -t --name=fs-git --privileged=true --net="host" k-anderson/freeswitch-git

* -d: Detached mode. Run container in the background, print new container id
* -t: Allocate a pseudo-tty.  (I believe this keeps the FreeSWITCH console happy, if we started fs with -nc then the container dies after the fork)
* --name=fs-git: This is optional, just means later this container can be referenced via 'fs-git'
* --privileged=true: Give extended privileges to this container (I am not sure this is required, perhaps for timers?  Regardless I saw others doing it so I did too!)
* --net="host": With the networking mode set to host a container will share the hosts network stack and all interfaces from the host will be available to the container.  Compared to the default bridge mode, the host mode gives *significantly* better networking performance...
* *image name*

### Connecting to the fs_cli
There are a number of ways to connect to the FreeSWITCH CLI.  

Since docker 1.3 ther is a new command exec.  Using this we can execute fs_cli on the container running FreeSWITCH with an interactive TTY:

       docker exec -it fs-git fs_cli

* -it: Keep stdin open even if not attached and allocate a pseudo-tty
* *container name/id*
* *command*

This can also be used to enter the container and poke:

       docker exec -it fs-git bash

* -it: Keep stdin open even if not attached and allocate a pseudo-tty
* *container name/id*
* *command*

The other method I have used is to create a new temporary container running just the fs_cli and using the same network environment:

       docker run -it --rm --net="host" k-anderson/freeswitch-git fs_cli

* -it: Keep stdin open even if not attached and allocate a pseudo-tty
* --rm: Automatically remove the container when it exits
* --net="host": Same as above but we need the two containes (one running FS and one running fs_cli) to have the access to the same network environment.  If this was properly set up you could use --net="container:fs-git"
* *image name*
* *command to run instead of the default*

### Serving Verto
This is not great behavour, we are using a FreeSWITCH image to serve HTTP files but we want to take avantage of easily 
using the verto source in the image.  This will start a FreeSWITCH container (detached) but override the default command
to start apache in the foreground instead.

	docker run -d --name=fs-verto --net="host" k-anderson/freeswitch-git /usr/sbin/apache2ctl -D FOREGROUND

* -d: Detached mode. Run container in the background, print new container id
* --name: This is optional, just means later this container can be referenced via 'fs-verto'
* --net="host": With the networking mode set to host a container will share the hosts network stack and all interfaces from the host will be available to the container.  In this case we don't need the preformance but the image does not 'expose' any ports (can that be done dymaically at startup?)
* *image name*
* *command to run instead of the default*

## TODO
* Figure out the network stack
* Run as the freeswitch user
* Cleanup the verto stuff
* Read only? https://blog.docker.com/2015/02/docker-1-5-ipv6-support-read-only-containers-stats-named-dockerfiles-and-more/
