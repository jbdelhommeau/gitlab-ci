FROM php:7.2-alpine

LABEL version="1.2"
LABEL maintainer="Jean-Baptiste Delhommeau <jeanbadel@gmail.com>"

RUN apk --no-cache add git openssh bash tini nodejs-npm rsync make postgresql-client

# Install php lib
RUN set -eux; \
    apk add --no-cache --virtual .build-deps \
    $PHPIZE_DEPS \
    gettext-dev \
    icu-dev \
    postgresql-dev \
    zlib-dev \
    icu-libs \
    ; \
    docker-php-ext-install \
    gettext \
    intl \
    pcntl \
    pdo_pgsql \
    zip \
    bcmath \
    ;\
    runDeps="$( \
    scanelf --needed --nobanner --format '%n#p' --recursive /usr/local/lib/php/extensions \
    | tr ',' '\n' \
    | sort -u \
    | awk 'system("[ -e /usr/local/lib/" $1 " ]") == 0 { next } { print "so:" $1 }' \
    )"; \
    apk add --no-cache --virtual .phpexts-rundeps $runDeps; \
    \
    apk del .build-deps

RUN echo "memory_limit=-1" > "$PHP_INI_DIR/conf.d/memory-limit.ini" \
 && echo "date.timezone=${PHP_TIMEZONE:-UTC}" > "$PHP_INI_DIR/conf.d/date_timezone.ini"

COPY --from=composer:latest /usr/bin/composer /usr/bin/composer
ENV COMPOSER_CACHE_DIR=/tmp/.docker_composer
ENV COMPOSER_MEMORY_LIMIT=-1
ENV COMPOSER_ALLOW_SUPERUSER 1

RUN set -eux; \
    composer global require "symfony/flex" --prefer-dist --no-progress --no-suggest --classmap-authoritative

# Install PHPUnit
RUN curl -SL "https://phar.phpunit.de/phpunit-6.5.phar" -o /usr/local/bin/phpunit \
	&& chmod +x /usr/local/bin/phpunit

# Install PHP-CS & PHP-CS-Fixer
RUN cd /usr/local/bin \
    && curl -sL https://squizlabs.github.io/PHP_CodeSniffer/phpcs.phar -o phpcs \
    && chmod +x phpcs \
    && curl -sL http://cs.sensiolabs.org/download/php-cs-fixer-v2.phar -o php-cs-fixer \
    && chmod +x php-cs-fixer


RUN npm i -g eclint --cache /tmp/npm-cache

RUN mkdir -p ~/.ssh

# Not check host for ssh connection
RUN echo -e "Host *\n\tStrictHostKeyChecking no\n\n" > ~/.ssh/config

#Clean tmp dir
RUN rm -rf /var/cache/apk/* && \
    rm -rf /tmp/*

WORKDIR /app

ENTRYPOINT ["/sbin/tini", "--"]
