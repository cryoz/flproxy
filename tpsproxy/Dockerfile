## syntax=docker/dockerfile:labs
FROM debian:stable-slim
LABEL maintainer="cryoz"

ADD --link service /etc/service/

EXPOSE 8118
ENV TOR_PREF_COUNTRIES="se,gb,nl,de,us"

RUN apt update -y && apt upgrade -y && apt -y install privoxy tor torsocks runit tini curl python3 ca-certificates dnsutils --no-install-recommends \
&& addgroup --system tordocker \
&& adduser --system tordocker --ingroup tordocker \
&& chown tordocker:tordocker /etc/service \
&& chown -R tordocker:tordocker /etc/service/* \
&& mkdir /.tor \
&& chown tordocker:tordocker /.tor \
&& apt autoremove -y && apt-get clean \
&& rm -rf /var/lib/apt/lists/* \
&& curl -L -o /relayscanner.pyz 'https://github.com/ValdikSS/tor-relay-scanner/releases/download/1.0.2/tor-relay-scanner-1.0.2.pyz'

COPY <<EOT /docker-entrypoint.sh
#!/bin/sh
cp -f /etc/service/tor/torrc.base /etc/service/tor/torrc
python3 /relayscanner.pyz -g 1 --timeout 5 -c \${TOR_PREF_COUNTRIES} --torrc -o /etc/service/tor/bridges.conf > /dev/null
exec "\$@"
EOT

RUN chmod +x /docker-entrypoint.sh

HEALTHCHECK --interval=120s --timeout=30s --start-period=120s --start-interval=120s --retries=2 \
            CMD curl --fail --retry 2 --max-time 5 --retry-delay 5 --retry-max-time 30 -x http://127.0.0.1:8118 -s 'https://duckduckgogg42xjoc72x3sjasowoarfbgcmvfimaftt6twagswzczad.onion' -k > /dev/null && echo "HealthCheck succeeded..." || bash -c 'kill -s 15 -1 && (sleep 10; kill -s 9 -1)'

USER tordocker

ENTRYPOINT ["tini", "--", "/docker-entrypoint.sh"]
CMD ["runsvdir", "/etc/service"]
