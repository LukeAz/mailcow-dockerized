FROM debian:buster-slim
LABEL maintainer "LukeAz"

ARG DEBIAN_FRONTEND=noninteractive
ENV LC_ALL C

RUN set -x && apt-get update && apt-get install -y curl gpg redis-tools procps dnsutils netcat \
  && echo 'deb http://download.opensuse.org/repositories/home:/susenerf:/rspamd-exotic-debian/Raspbian_10/ /' | tee /etc/apt/sources.list.d/home:susenerf:rspamd-exotic-debian.list \
  && curl -fsSL https://download.opensuse.org/repositories/home:susenerf:rspamd-exotic-debian/Raspbian_10/Release.key | gpg --dearmor | tee /etc/apt/trusted.gpg.d/home_susenerf_rspamd-exotic-debian.gpg > /dev/null \
  && apt-get update \
  && apt-get install -y rspamd \
  && rm -rf /var/lib/apt/lists/* \
  && apt-get autoremove --purge \
  && apt-get clean \
  && mkdir -p /run/rspamd \
  && chown _rspamd:_rspamd /run/rspamd \
  && echo 'alias ll="ls -la --color"' >> ~/.bashrc

COPY settings.conf /etc/rspamd/settings.conf
COPY metadata_exporter.lua /usr/share/rspamd/plugins/metadata_exporter.lua
COPY docker-entrypoint.sh /docker-entrypoint.sh

ENTRYPOINT ["/docker-entrypoint.sh"]

STOPSIGNAL SIGTERM

CMD ["/usr/bin/rspamd", "-f", "-u", "_rspamd", "-g", "_rspamd"]
