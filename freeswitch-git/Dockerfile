FROM debian:jessie
MAINTAINER K Anderson <bitbashing@gmail.com>

##
## Update the system (just to be sure this is the latest)
## Then install wget so it can be used to add the FreeSWITCH debian key
## and vim because when I am working on a container it is convient to have
##
RUN DEBIAN_FRONTEND=noninteractive APT_LISTCHANGES_FRONTEND=none apt-get update -y && \
    DEBIAN_FRONTEND=noninteractive APT_LISTCHANGES_FRONTEND=none apt-get dist-upgrade -y && \
    DEBIAN_FRONTEND=noninteractive APT_LISTCHANGES_FRONTEND=none apt-get install -y wget vim

##
## Install the FreeSWITCH debain key
##
RUN echo "deb http://files.freeswitch.org/repo/deb/debian/ jessie main" > /etc/apt/sources.list.d/99FreeSWITCH.list && \
    wget -O - http://files.freeswitch.org/repo/deb/debian/key.gpg |apt-key add - && \
    DEBIAN_FRONTEND=noninteractive APT_LISTCHANGES_FRONTEND=none apt-get update -y

##
## Install 'most' the dependencies to build FreeSWITCH 1.6
## then the  dependencies to build mod_amqp and mod_erlang_event/mod_kazoo
## and finally some dependencies to auto-tune channels (for fun)
##
RUN DEBIAN_FRONTEND=noninteractive APT_LISTCHANGES_FRONTEND=none apt-get install -y --force-yes freeswitch-video-deps-most && \
    DEBIAN_FRONTEND=noninteractive APT_LISTCHANGES_FRONTEND=none apt-get install -y erlang-nox librabbitmq-dev && \
    DEBIAN_FRONTEND=noninteractive APT_LISTCHANGES_FRONTEND=none apt-get install -y ladspa-sdk tap-plugins swh-plugins libgsm1 libfftw3-3 autotalent

##
## When the video branch was undergoing many rebases this setting pull.rebase
## simplified 'git pull'.  There is little reason to set this for the
## docker image, but if you intend to muck with source it could be
## helpful (I prefer to rebase anyway so)...
## Then clone the FreeSWITCH source to /usr/src/freeswitch.git
## (If we wanted to set a branch to build from this would
## be a good place to do it)
##
RUN git config --global pull.rebase true && \
    git clone https://freeswitch.org/stash/scm/fs/freeswitch.git /usr/src/freeswitch.git

##
## Switch to the source directory and copy our modules.conf 
## into the container.  This will keep the FreeSWITCH build 
## system from creating it and defer to ours.
##
WORKDIR /usr/src/freeswitch.git
ADD image-files/modules.conf.in modules.conf

##
## Build and install FreeSWITCH including
## download the sound files used for prompts
## and music-on-hold (both in CD quality)
##
RUN ./bootstrap.sh -j && \
    ./configure -C && \
    make && \
    make install && \
    make cd-sounds-install cd-moh-install

##
## Set some environment vars
##
ENV FREESWITCH_PATH /usr/local/freeswitch
ENV PATH /usr/local/freeswitch/bin:$PATH

##
## Switch to the directory in the image
## were FreeSWITCH is installed and replace
## the default configuration as well as certs
##
WORKDIR /usr/local/freeswitch
RUN rm -rf conf
ADD image-files/conf conf
ADD image-files/certs certs

##
## Some final install house-cleaning creating
## a FreeSWITCH user and taking care of some
## file permissions
##
RUN useradd --system --home-dir ${FREESWITCH_PATH} --comment "FreeSWITCH Voice Platform" --groups daemon freeswitch && \
    chown -R freeswitch:daemon ${FREESWITCH_PATH} && \
    chmod -R ug=rwX,o= ${FREESWITCH_PATH} && \
    chmod -R u=rwx,g=rx ${FREESWITCH_PATH}/bin/*

##
## FreeSWITCH wiki recommends some kernel tweaks so add
## those now.  Some investigation needs to be made to
## see how effective those are in a docker environment...
##
ADD image-files/fs-systemctl-tweaks.conf /etc/sysctl.d/fs-systemctl-tweaks.conf

## 
## The build for verto appears to be in-flux but I had issues... It didn't work until I installed nodejs-legacy instead.
## See http://stackoverflow.com/questions/21168141/can-not-install-packages-using-node-package-manager-in-ubuntu 
## I also think simply symlinking would have resolved it (ln -s /usr/bin/nodejs /usr/bin/node)
##
## Bower seems to be the only thing required to install the dependencies if you wanted to server it from apache.
## I am sure this is not the proper way of serving it from something other than grunt because it
## attempts developer tasks such as broswer-sync.  I think it is all ignorable....
##
WORKDIR /usr/src/freeswitch.git/html5/verto/verto_communicator
RUN DEBIAN_FRONTEND=noninteractive APT_LISTCHANGES_FRONTEND=none apt-get install -y npm nodejs-legacy && \
    npm install -g bower && \
    bower --config.interactive=false --allow-root install

##
## Temporary workaround that needs deeper investigation as to why my browser seems to have no matching resolutions...
##
RUN perl -i -pe 's|(this\.data\.vidQual = videoQuality\[videoQuality\.length - 1\]\.id;)|if(videoQuality.length) \1|g' js/verto-service.js

##
## Serve verto communicator from images made from this container via apache
## (mostly taken from the 'FreeSWITCH+1.6+Video wiki article)
##
RUN ln -s /usr/src/freeswitch.git/html5/verto/js /var/www/html && \
    ln -s /usr/src/freeswitch.git/html5/verto/verto_communicator /var/www/html && \
    echo '<html><body></body></html>' > /var/www/html/index.html && \
    cp -a /etc/apache2/sites-available/default-ssl.conf /etc/apache2/sites-available/verto-ssl.conf && \
    perl -i -pe 's|SSLCertificateFile.*|SSLCertificateFile /usr/local/freeswitch/certs/wss.pem|g' /etc/apache2/sites-available/verto-ssl.conf && \
    perl -i -pe 's|SSLCertificateKeyFile.*|SSLCertificateKeyFile /usr/local/freeswitch/certs/wss.pem|g' /etc/apache2/sites-available/verto-ssl.conf && \
    perl -i -pe 's|SSLCertificateChainFile.*|SSLCertificateChainFile /usr/local/freeswitch/certs/wss.pem|g' /etc/apache2/sites-available/verto-ssl.conf && \
    perl -i -pe 's|ServerSignature On|ServerSignature Off|g' /etc/apache2/conf-available/security.conf && \
    perl -i -pe 's|ServerTokens OS|ServerTokens Prod|g' /etc/apache2/conf-available/security.conf && \
    perl -i -pe 's|SSLProtocol(.*)|SSLProtocol All -SSLv2 -SSLv3|g' /etc/apache2/mods-available/ssl.conf && \
    perl -i -pe 's|SSLCipherSuite(.*)|SSLCipherSuite AES128+EECDH:AES128+EDH|g' /etc/apache2/mods-available/ssl.conf && \
    perl -i -pe 's|#SSLHonorCipherOrder on|SSLHonorCipherOrder on|g' /etc/apache2/mods-available/ssl.conf && \
    a2enmod ssl && \
    a2ensite verto-ssl

##
## By default when a container is created it should run FreeSWITCH!
##
CMD /usr/local/freeswitch/bin/freeswitch -c

