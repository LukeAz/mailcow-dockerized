FROM debian:bullseye-slim
LABEL maintainer "LukeAz"

ARG DEBIAN_FRONTEND=noninteractive
ENV LC_ALL C

RUN set -x && apt-get update && apt-get install -y \
  ca-certificates gnupg2 apt-transport-https dpkg-dev devscripts libssl-dev \
  glib2.0 ragel liblua5.4-dev cmake libsqlite3-dev libmagic-dev libicu-dev libpcre2-dev \
  zlib1g libsodium-dev git libjemalloc-dev libunwind-dev
RUN git clone --recursive https://github.com/rspamd/rspamd.git \
  && mkdir rspamd.build \
  && cd rspamd.build \
  && cmake ../rspamd -DENABLE_HYPERSCAN=OFF -DENABLE_LUAJIT=OFF \
  && make \
  && make install
RUN apt-get --no-install-recommends -y install redis-tools procps nano dnsutils netcat \
  && rm -rf /var/lib/apt/lists/* \
  && apt-get autoremove --purge \
  && apt-get clean \
  && mkdir -p /run/rspamd \
  && mkdir /var/log/rspamd && touch rspamd.log \
  && useradd _rspamd \
  && usermod -a -G _rspamd _rspamd \
  && chown _rspamd:_rspamd /run/rspamd \
  && chown _rspamd:_rspamd /var/log/rspamd \
  && echo 'alias ll="ls -la --color"' >> ~/.bashrc

COPY settings.conf /etc/rspamd/settings.conf
COPY metadata_exporter.lua /usr/share/rspamd/plugins/metadata_exporter.lua
COPY docker-entrypoint.sh /docker-entrypoint.sh

ENTRYPOINT ["/docker-entrypoint.sh"]

STOPSIGNAL SIGTERM

CMD ["/usr/local/bin/rspamd", "-f", "-u", "_rspamd", "-g", "_rspamd"]
