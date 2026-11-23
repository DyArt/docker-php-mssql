FROM php:8.6-fpm-alpine

ENV ACCEPT_EULA=Y \
    COMPOSER_ALLOW_SUPERUSER=1 \
    PATH="/var/www/vendor/bin:$PATH"

COPY --from=composer:2 /usr/bin/composer /usr/local/bin/composer

# Runtime-pakketten
RUN apk add --no-cache \
        bash openssh-client rsync gnupg \
        icu-libs libzip bzip2 gmp libpq unixodbc \
        freetype libjpeg-turbo libpng libwebp imagemagick

# Microsoft ODBC 18 met GPG-verificatie
RUN curl -O https://download.microsoft.com/download/ade174b7-8cea-4543-91a6-c33ae320c2f0/msodbcsql18_18.7.1.1-1_amd64.apk \
    && curl -O https://download.microsoft.com/download/ade174b7-8cea-4543-91a6-c33ae320c2f0/msodbcsql18_18.7.1.1-1_amd64.sig \
    && curl https://packages.microsoft.com/keys/microsoft.asc | gpg --import - \
    && gpg --verify msodbcsql18_18.7.1.1-1_amd64.sig msodbcsql18_18.7.1.1-1_amd64.apk \
    && apk add --no-cache --allow-untrusted msodbcsql18_18.7.1.1-1_amd64.apk \
    && rm -f msodbcsql18_18.7.1.1-1_amd64.*

# Extensies; PIE alleen tijdens de build gemount
RUN --mount=type=bind,from=ghcr.io/php/pie:bin,source=/pie,target=/usr/local/bin/pie \
    set -eux; \
    apk add --no-cache --virtual .build-deps \
        $PHPIZE_DEPS git unzip linux-headers \
        unixodbc-dev postgresql-dev icu-dev libzip-dev bzip2-dev gmp-dev \
        freetype-dev libjpeg-turbo-dev libpng-dev libwebp-dev imagemagick-dev; \
    docker-php-ext-configure gd --with-freetype --with-jpeg --with-webp; \
    docker-php-ext-install -j"$(nproc)" \
        bcmath bz2 exif gd gmp intl pdo_mysql pdo_pgsql zip ftp pcntl; \
    pie install --no-cache microsoft/sqlsrv; \
    pie install --no-cache microsoft/pdo_sqlsrv; \
    # iamgick is not compatible with PHP 8.6 yet
    # pie install --no-cache imagick/imagick; \
    # ds is not compatible with PHP 8.6 yet
    # pie install --no-cache php-ds/ext-ds; \
    # redis is not compatible with PHP 8.6 yet
    # pie install --no-cache phpredis/phpredis; \
    apk del .build-deps; \
    rm -rf /tmp/* /var/cache/apk/* /var/tmp/*

# RUN php -v && php -m | grep -Ei '^(sqlsrv|pdo_sqlsrv|imagick|ds|redis|gd|intl|pdo_pgsql|Zend OPcache)$'

WORKDIR /var/www
