FROM nextcloud:21.0-beta

RUN set -ex; \
    \
    apt-get update; \
    apt-get install -y --no-install-recommends \
        curl \
        ffmpeg \
        libmagickcore-6.q16-6-extra \
        dnsutils \
        procps \
        smbclient \
        p7zip \
        p7zip-full \
        supervisor \
#       libreoffice \
    ; \
    rm -rf /var/lib/apt/lists/*

RUN set -ex; \
    \
    savedAptMark="$(apt-mark showmanual)"; \
    \
    apt-get update; \
    apt-get install -y --no-install-recommends \
        libbz2-dev \
        libc-client-dev \
        libkrb5-dev \
        libsmbclient-dev \
    ; \
    \
    docker-php-ext-configure imap --with-kerberos --with-imap-ssl; \
    docker-php-ext-install \
        bz2 \
        imap \
    ; \
    pecl install smbclient; \
    pecl -v install rar; \
    docker-php-ext-enable smbclient; \
    \
# reset apt-mark's "manual" list so that "purge --auto-remove" will remove all build dependencies
    apt-mark auto '.*' > /dev/null; \
    apt-mark manual $savedAptMark; \
    ldd "$(php -r 'echo ini_get("extension_dir");')"/*.so \
        | awk '/=>/ { print $3 }' \
        | sort -u \
        | xargs -r dpkg-query -S \
        | cut -d: -f1 \
        | sort -u \
        | xargs -rt apt-mark manual; \
    \
    apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false; \
    rm -rf /var/lib/apt/lists/*


RUN mkdir -p \
    /var/log/supervisord \
    /var/run/supervisord \
;

COPY supervisord.conf /

ENV NEXTCLOUD_UPDATE=1

COPY ./entrypoint.sh /entrypoint.sh
ENTRYPOINT ["/entrypoint.sh"]

CMD ["/usr/bin/supervisord", "-c", "/supervisord.conf"]
