FROM ubuntu:24.04

# See README.md for what this does and why

# This Dockerfile will fail if the repository is not used as the build context.
# do an early COPY step to ensure that build is being invoked correctly.

ENV DGL_CHROOT=""
ENV DGL_CONF_HOME=/home/crawl-dev/dgamelaunch-config
ENV SCRIPTS="$DGL_CONF_HOME/server/scripts"
ENV SRC_SCRIPTS=/usr/src/dgamelaunch-config/server/scripts
ARG DEBIAN_FRONTEND=noninteractive
ARG TERM=dumb
ARG BRANCH

# basic prerequisites
RUN apt-get update
RUN apt-get install -y locales locales-all
ENV LANG=en_US.UTF-8
RUN sed -i -e "s/# $LANG.*/$LANG.UTF-8 UTF-8/" /etc/locale.gen \
    && dpkg-reconfigure --frontend=noninteractive locales \
    && update-locale LANG=$LANG
RUN apt-get install -y --no-install-recommends sudo git gh bash build-essential autoconf automake bison libncursesw5-dev flex liblua5.1-0-dev libsqlite3-dev libz-dev locales vim nano tmux locales-all pkg-config python3 python3-pip python3-yaml ccache cpulimit libpng-dev sqlite3 libpcre3 libpcre3-dev advancecomp pngcrush openssh-server curl wget libfl-dev cron jq
RUN apt-get install -y apache2 libcgi-pm-perl libdbi-perl libdbd-sqlite3-perl cpanminus && cpanm Linux::Inotify2
RUN apt-get install -y nginx nginx-extras fcgiwrap spawn-fcgi && mkdir -p /var/run/fcgiwrap
RUN echo "deb [trusted=yes] http://archive.ubuntu.com/ubuntu/ bionic main universe" >> /etc/apt/sources.list && \
    echo "deb [trusted=yes] http://archive.ubuntu.com/ubuntu/ focal main universe" >> /etc/apt/sources.list && \
    apt-get update && \
    apt-get install -y gcc-6 g++-6 gcc-7 g++-7 gdb && \
    apt-get install -y libsdl1.2-dev libsdl-image1.2-dev

RUN pip3 install tornado jinja2 --break-system-packages \
&& rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* \
&& ln -s /usr/bin/python3 /usr/bin/python

RUN useradd -m crawl \
&& useradd -m crawl-dev \
&& usermod -G root -a crawl \
&& usermod -G root -a crawl-dev \
&& usermod -G www-data -a crawl \
&& usermod -G www-data -a crawl-dev \
&& usermod -G crawl -a root \
&& usermod -G crawl -a www-data \
&& usermod -G crawl-dev -a root \
&& usermod -G crawl-dev -a www-data

# CCache
# Update symlinks
RUN /usr/sbin/update-ccache-symlinks
ENV PATH="/usr/lib/ccache:$PATH"
RUN ccache -F 0
RUN ccache -M 0

RUN git clone ${BRANCH:+-b $BRANCH} https://github.com/refracta/dcss-server /usr/src/dgamelaunch-config
RUN git config --global --add safe.directory /usr/src/dgamelaunch-config
RUN chown -R crawl-dev:crawl-dev /usr/src/dgamelaunch-config
RUN git config --global --add safe.directory $DGL_CONF_HOME
ENV PATH="$DGL_CONF_HOME/bin:$PATH"

WORKDIR /home/crawl-dev

# allow crawl-dev to run various things with sudo without a password. These
# permissions may be too open for a production server.
COPY server/etc/dgl_sudoers /etc/sudoers.d/

# set up dgamelaunch
# TODO: what are these directories in ~crawl-dev used for? `run` doesn't exist
# TODO: Fix logs, announcements.log, run
USER crawl-dev
RUN mkdir logs && touch logs/announcements.log
RUN git clone https://github.com/crawl/dgamelaunch.git \
    && cd dgamelaunch && ./autogen.sh --enable-debugfile --enable-sqlite --enable-shmem && make

RUN git config --global url.https://github.com/.insteadOf git://github.com/

USER root
RUN cd dgamelaunch \
    && mkdir -p $DGL_CHROOT/bin \
    && make install && cp ee virus $DGL_CHROOT/bin \
    && touch $DGL_CHROOT/dgamelaunch

RUN set -e && \
    TTYD_CHECKSUM="8a217c968aba172e0dbf3f34447218dc015bc4d5e59bf51db2f2cd12b7be4f55" && \
    curl -L https://github.com/tsl0922/ttyd/releases/download/1.7.7/ttyd.x86_64 -o /usr/bin/ttyd && \
    echo "$TTYD_CHECKSUM  /usr/bin/ttyd" | sha256sum -c - && \
    chmod +x /usr/bin/ttyd

# this is taken from build-crawl-dirs to here because it's the only directory that's outside of volumes due to not storing state
RUN mkdir -p /cores
# TODO: could add apache setup here
# TODO: Remove?

# Next steps for creating versionsdb/crawl gamedir/publishing dgl files and building crawl is done after starting,
# because volumes need to be mounted to preserve state.
ENTRYPOINT ["sh", "-c", "$SRC_SCRIPTS/static/entrypoint.sh"]
HEALTHCHECK CMD ["sh", "-c", "$SRC_SCRIPTS/static/healthcheck.sh"]
