FROM php:8-fpm-alpine

WORKDIR /var/www/html

RUN set -ex \
    && apk add --update --no-cache --virtual .build-deps \
        ${PHPIZE_DEPS} \
        shadow \
        build-base \
        git \
        autoconf \
        g++ \
        libtool \
        make \
        libgcrypt \
        pcre-dev \
    && apk add --no-cache \
        libzip-dev \
        libsodium-dev \
        libxml2-dev \
        oniguruma-dev \
        openssl-dev \
        curl-dev \
        icu-dev \
        sqlite-dev \
        busybox-suid \
        zlib-dev \
        freetype-dev \
        libjpeg-turbo-dev \
        imagemagick-dev \
        libpng-dev \
        tidyhtml-dev \
        postgresql-dev \
    && git clone \
        https://github.com/krakjoe/apcu.git \
        /usr/src/php/ext/apcu \
    && git clone \
        https://github.com/Imagick/imagick.git \
        /usr/src/php/ext/imagick

RUN docker-php-ext-configure gd --with-freetype --with-jpeg

RUN pecl install redis

RUN docker-php-ext-install \
    gd \
    curl \
    iconv \
    tidy \
    tokenizer \
    bcmath \
    pcntl \
    exif \
    sodium \
    zip \
    xml \
    intl \
    mbstring \
    pdo_sqlite \
    pdo_mysql \
    pdo_pgsql \
    apcu \
    imagick

RUN docker-php-ext-enable \
    redis \
    gd \
    apcu \
    imagick

# Clean up
RUN rm -rf /var/cache/apk/* \
    && rm -rf /tmp/*

# Add crontab file in the cron directory
COPY crontab /etc/cron.d/cron

RUN chmod 0644 /etc/cron.d/cron

# Add UID '1000' to www-data
RUN usermod -u 1000 www-data && groupmod -g 1000 www-data

RUN chown -R www-data:www-data /var/www/html

# Change current user to www
USER www-data
