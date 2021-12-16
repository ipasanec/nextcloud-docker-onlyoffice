FROM nextcloud:fpm-alpine

RUN set -ex; \
    \
    apk add --no-cache \
        ffmpeg \
        imagemagick \
        imagemagick-dev \
        procps \
        samba-client \
        supervisor \
        libreoffice \
        ghostscript \
        bash \
        fuse \
    ;

RUN set -ex; \
    \
    apk add --no-cache --virtual .build-deps \
        $PHPIZE_DEPS \
        imap-dev \
        krb5-dev \
        openssl-dev \
        samba-dev \
        bzip2-dev \
    ; \
    \
    docker-php-ext-configure imap --with-kerberos --with-imap-ssl; \
    docker-php-ext-install \
        bz2 \
        imap \
    ; \
    pecl install smbclient; \
    docker-php-ext-enable smbclient; \
    docker-php-ext-enable imagick; \
    \
    runDeps="$( \
        scanelf --needed --nobanner --format '%n#p' --recursive /usr/local/lib/php/extensions \
            | tr ',' '\n' \
            | sort -u \
            | awk 'system("[ -e /usr/local/lib/" $1 " ]") == 0 { next } { print "so:" $1 }' \
    )"; \
    apk add --virtual .nextcloud-phpext-rundeps $runDeps; \
    apk del .build-deps

RUN apk --no-cache add  \
        wget            \
        ca-certificates \
        libstdc++
# Get and install glibc for alpine
ARG APK_GLIBC_VERSION=2.29-r0
ARG APK_GLIBC_FILE="glibc-${APK_GLIBC_VERSION}.apk"
ARG APK_GLIBC_BIN_FILE="glibc-bin-${APK_GLIBC_VERSION}.apk"
ARG APK_GLIBC_BASE_URL="https://github.com/sgerrand/alpine-pkg-glibc/releases/download/${APK_GLIBC_VERSION}"
RUN wget -q -O /etc/apk/keys/sgerrand.rsa.pub https://alpine-pkgs.sgerrand.com/sgerrand.rsa.pub \
    && wget "${APK_GLIBC_BASE_URL}/${APK_GLIBC_FILE}"       \
    && apk --no-cache add "${APK_GLIBC_FILE}"               \
    && wget "${APK_GLIBC_BASE_URL}/${APK_GLIBC_BIN_FILE}"   \
    && apk --no-cache add "${APK_GLIBC_BIN_FILE}"           \
    && rm glibc-*

RUN mkdir -p \
    /var/log/supervisord \
    /var/run/supervisord \
;

COPY supervisord.conf /

ENV NEXTCLOUD_UPDATE=1

CMD ["/usr/bin/supervisord", "-c", "/supervisord.conf"]
