FROM dunglas/frankenphp:php8.5-alpine

ENV ACCEPT_EULA=Y

RUN apk add --no-cache \
    bash \
    less \
    su-exec \
    unzip

# Eerst de overige extensies, zodat een eventuele fout hierin
# gescheiden blijft van de SQL Server-extensies.
RUN install-php-extensions \
    bcmath \
    ds \
    exif \
    ftp \
    gd \
    intl \
    opcache \
    pcntl \
    redis \
    zip

# De installer regelt ook de benodigde ODBC-afhankelijkheden.
RUN install-php-extensions \
    sqlsrv \
    pdo_sqlsrv

# Controleer dat beide extensies daadwerkelijk geladen kunnen worden.
RUN php --ri sqlsrv \
    && php --ri pdo_sqlsrv
