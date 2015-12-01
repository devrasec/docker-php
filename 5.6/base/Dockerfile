FROM phusion/baseimage:0.9.17
MAINTAINER Luis Contreras <luiscon26@gmail.com>

# persistent / runtime deps
RUN apt-get update \
    && apt-get install -y ca-certificates curl librecode0 libsqlite3-0 \
    libxml2 --no-install-recommends && rm -r /var/lib/apt/lists/*

# phpize deps
RUN apt-get update && apt-get install -y autoconf file g++ gcc libc-dev \
    make pkg-config re2c --no-install-recommends && rm -r /var/lib/apt/lists/*

ENV PHP_INI_DIR /usr/local/etc/php
RUN mkdir -p $PHP_INI_DIR/conf.d

ENV PHP_EXTRA_CONFIGURE_ARGS --enable-fpm --with-fpm-user=www-data --with-fpm-group=www-data

ENV GPG_KEYS 0BD78B5F97500D450838F95DFE857D9A90D90EC1 6E4F6AB321FDC07F2C332E3AC2BF0BC433CFC8B3
RUN set -xe \
    && for key in $GPG_KEYS; do \
        gpg --keyserver ha.pool.sks-keyservers.net --recv-keys "$key"; \
    done

ENV PHP_VERSION 5.6.15

# --enable-mysqlnd is included below because it's harder to compile after the
# fact the extensions are (since it's a plugin for several extensions,
# not an extension in itself)
RUN buildDeps=" \
        $PHP_EXTRA_BUILD_DEPS \
        libcurl4-openssl-dev \
        libreadline6-dev \
        librecode-dev \
        libsqlite3-dev \
        libssl-dev \
        libxml2-dev \
        xz-utils \
    " \
    && set -x \
    && apt-get update && apt-get install -y $buildDeps --no-install-recommends && rm -rf /var/lib/apt/lists/* \
    && curl -SL "http://php.net/get/php-$PHP_VERSION.tar.xz/from/this/mirror" -o php.tar.xz \
    && curl -SL "http://php.net/get/php-$PHP_VERSION.tar.xz.asc/from/this/mirror" -o php.tar.xz.asc \
    && gpg --verify php.tar.xz.asc \
    && mkdir -p /usr/src/php \
    && tar -xof php.tar.xz -C /usr/src/php --strip-components=1 \
    && rm php.tar.xz* \
    && cd /usr/src/php \
    && ./configure \
        --with-config-file-path="$PHP_INI_DIR" \
        --with-config-file-scan-dir="$PHP_INI_DIR/conf.d" \
        $PHP_EXTRA_CONFIGURE_ARGS \
        --disable-cgi \
        --enable-mysqlnd \
        --with-curl \
        --with-openssl \
        --with-readline \
        --with-recode \
        --with-zlib \
        --with-pdo-mysql \
        --with-mysqli \
    && make -j"$(nproc)" \
    && make install \
    && { find /usr/local/bin /usr/local/sbin -type f -executable -exec strip --strip-all '{}' + || true; } \
    && apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false -o APT::AutoRemove::SuggestsImportant=false $buildDeps \
    && make clean

COPY docker-php-ext-* /usr/local/bin/

WORKDIR /var/www/html
COPY php-fpm.conf /usr/local/etc/

EXPOSE 9000
RUN mkdir -p /etc/service/php-fpm
ADD init_fpm /etc/service/php-fpm/run

CMD ["/sbin/my_init"]

# Clean up APT when done.
RUN apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*