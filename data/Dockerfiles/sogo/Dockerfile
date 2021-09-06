ARG DISTRO=debian:11-slim
ARG DEBIAN_FRONTEND=noninteractive

FROM $DISTRO as builder
WORKDIR /tmp
RUN set -x && apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y \ 
  gnupg python liboath-dev libldap2-dev apt-transport-https ca-certificates build-essential fakeroot devscripts cdbs libpopt-dev zlib1g-dev libexpat1-dev cmake
RUN apt-key adv --keyserver keyserver.ubuntu.com --recv-key 0x810273C4 \
  && echo "deb https://packages.inverse.ca/SOGo/nightly/5/debian buster buster" >/etc/apt/sources.list.d/sogo.list \
  && echo "deb-src https://packages.inverse.ca/SOGo/nightly/5/debian buster buster" >>/etc/apt/sources.list.d/sogo.list
RUN set -x && apt-get update && apt-get source sogo sope wbxml2
RUN cd /tmp/wbxml2-* && debuild -b -uc -us -d && dpkg -i /tmp/*.deb
RUN set -x && apt-get build-dep -y sope
RUN cd /tmp/sope-* && debuild -b -uc -us && dpkg -i /tmp/*.deb
RUN set -x && apt-get build-dep -y sogo
RUN cd /tmp/sogo-* && debuild -b -uc -us && rm /tmp/*-dbg_* /tmp/*-dev_*

FROM $DISTRO
ENV GOSU_VERSION 1.14
COPY --from=builder /tmp/*.deb /tmp/
RUN set -x \
  && apt-get update && apt-get -y upgrade && apt-get install -y --no-install-recommends \
    dirmngr cron curl \
    gnustep-base-runtime libgnustep-base1.27 libobjc4 \
    libwbxml2-0 libglib2.0-0 liblasso3 libmemcached11 liboath0 libsodium23 libzip4 gnustep-make zip \
    libmariadb3 mariadb-client libpq5 libytnef0 supervisor wget rsync netcat psmisc patch syslog-ng \
    syslog-ng-core syslog-ng-mod-redis dirmngr psmisc gettext \
  && dpkgArch="$(dpkg --print-architecture | awk -F- '{ print $NF }')" \
  && wget -O /usr/local/bin/gosu "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$dpkgArch" \
  && chmod +x /usr/local/bin/gosu \
  && gosu nobody true \
  && mkdir /usr/share/doc/sogo \
  && touch /usr/share/doc/sogo/empty.sh \
  && dpkg -i /tmp/*.deb \
  && rm -rf /var/lib/apt/*
  
COPY ./bootstrap-sogo.sh /bootstrap-sogo.sh
COPY syslog-ng.conf /etc/syslog-ng/syslog-ng.conf
COPY syslog-ng-redis_slave.conf /etc/syslog-ng/syslog-ng-redis_slave.conf
COPY supervisord.conf /etc/supervisor/supervisord.conf
COPY acl.diff /acl.diff
COPY stop-supervisor.sh /usr/local/sbin/stop-supervisor.sh
COPY docker-entrypoint.sh /

RUN chmod +x /bootstrap-sogo.sh /usr/local/sbin/stop-supervisor.sh

ENTRYPOINT ["/docker-entrypoint.sh"]

CMD exec /usr/bin/supervisord -c /etc/supervisor/supervisord.conf
