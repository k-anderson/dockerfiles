FROM base/archlinux
MAINTAINER K Anderson <bitbashing@gmail.com>

RUN  pacman-key --refresh-keys \
        && pacman --needed --noprogressbar --noconfirm -Suy

ENV HOME /home/spotify
RUN useradd --create-home --home-dir $HOME spotify \
	&& gpasswd -a spotify audio \
	&& chown -R spotify:spotify $HOME \
        && pacman-db-upgrade \
        && pacman -S --needed --noprogressbar --noconfirm alsa-lib gconf gtk2 nss libxtst libx11 libxss git ttf-dejavu \
        && pacman -S --needed --noprogressbar --noconfirm binutils make gcc fakeroot

WORKDIR $HOME
USER spotify
RUN git clone https://aur.archlinux.org/libcurl-compat.git \
        && cd libcurl-compat \
        && makepkg

USER root
RUN pacman -U --needed --noprogressbar --noconfirm $HOME/libcurl-compat/*.pkg.tar.xz

USER spotify
RUN git clone https://aur.archlinux.org/spotify.git \
        && cd spotify \
        && makepkg

USER root
RUN pacman -U --needed --noprogressbar --noconfirm $HOME/spotify/*.pkg.tar.xz \
        && pacman -Rs --noprogressbar --noconfirm binutils make gcc fakeroot

USER spotify
# make search bar text better
RUN echo "QLineEdit { color: #000 }" > /home/spotify/spotify-override.css
ENTRYPOINT	[ "spotify" ]
CMD [ "-stylesheet=/home/spotify/spotify-override.css" ]
