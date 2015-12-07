FROM luistmac/php:base
MAINTAINER Luis Contreras <luiscon26@gmail.com>

# Install common modules
RUN buildDeps=" \
        bzip2 \
        gettext \
        libbz2-dev \
        libc-client2007e-dev \
        libfreetype6-dev \
        libjpeg-turbo8-dev \
        libmcrypt-dev \
        libmemcached-dev \
        libpcre3-dev \
        libpng12-dev \
    " \
    && set -x \
    && apt-get update && apt-get install -y $buildDeps && rm -rf /var/lib/apt/lists/* \
    && docker-php-ext-install \
        bcmath \
        bz2 \
        calendar \
        exif \
        ftp \
        gettext \
        iconv \
        mcrypt \
    && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
    && docker-php-ext-install gd \
    && docker-php-ext-configure imap --with-kerberos --with-imap-ssl \
    && docker-php-ext-install imap \
    && curl -SL http://xdebug.org/files/xdebug-2.3.3.tgz -o xdebug.tgz \
    && mkdir -p /usr/src/php/ext/xdebug \
    && tar -xzf xdebug.tgz -C /usr/src/php/ext/xdebug --strip-components=1 \
    && rm -rf xdebug.tgz \
    && docker-php-ext-configure xdebug \
    && docker-php-ext-install xdebug opcache \
    && curl -SL https://pecl.php.net/get/memcached-2.2.0.tgz -o memcached.tgz \
    && mkdir -p /usr/src/php/ext/memcached \
    && tar -xzf memcached.tgz -C /usr/src/php/ext/memcached --strip-components=1 \
    && rm -rf memcached.tgz \
    && docker-php-ext-configure memcached --disable-memcached-sasl \
    && docker-php-ext-install memcached \
    && curl -SL https://github.com/twigphp/Twig/archive/v1.23.1.tar.gz -o twig.tar.gz \
    && mkdir -p /usr/src/php/ext/twig \
    && tar -xzf twig.tar.gz -C /usr/src/php/ext/twig --strip-components=3 Twig-1.23.1/ext/twig \
    && rm -rf twig.tar.gz \
    && docker-php-ext-configure twig \
    && docker-php-ext-install twig \
    && apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false -o APT::AutoRemove::SuggestsImportant=false $buildDeps \
    && apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

COPY php-fpm.conf /usr/local/etc/
COPY php.ini /usr/local/etc/php/

CMD ["/sbin/my_init"]