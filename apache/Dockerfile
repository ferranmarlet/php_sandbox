ARG COMPOSER_VERSION="2.0.8"
ARG PHP_VERSION="8.0.1"

FROM php:${PHP_VERSION}-apache

ARG XDEBUG_VERSION="3.0.2"
ENV DEBIAN_FRONTEND noninteractive

# Install minimal system requirements, PHP composer and Node
RUN true \
       && apt-get -qq update \
       && apt-get -qy install --no-install-recommends \
       ca-certificates \
       wget \
       git \
       && wget -qq -O /etc/apt/trusted.gpg.d/php.gpg "https://packages.sury.org/php/apt.gpg" \
       && echo "deb https://packages.sury.org/php/ $(. /etc/os-release && echo $VERSION_CODENAME) main" > /etc/apt/sources.list.d/php${PHP_VERSION}.list \
       && wget -qq -O- "https://deb.nodesource.com/setup_10.x" | bash - \
       && apt-get -qq update \
       && apt-get -qy install --no-install-recommends \
       "libzip-dev" \
       nodejs \
       && apt-get -qy purge --auto-remove \
       lsb-release \
       gnupg \
       && rm -rf /var/lib/apt/lists/* \
       ;

# Install Composer
COPY --from=composer /usr/bin/composer /usr/bin/composer

RUN true \
 && docker-php-ext-install zip \
 && a2enmod rewrite \
 && pecl install "xdebug-${XDEBUG_VERSION}" \
 && docker-php-ext-enable xdebug \
 && mv "$PHP_INI_DIR/php.ini-development" "$PHP_INI_DIR/php.ini" \
 && echo "xdebug.mode=debug" >> /usr/local/etc/php/php.ini \
;

COPY vhosts.conf /etc/apache2/sites-available/000-default.conf

# Modify www-data user to match system user
ARG GROUP_ID="1000"
ARG USER_ID="1000"
RUN true \
 && groupmod www-data --gid "${GROUP_ID}" \
 && usermod www-data --uid "${USER_ID}" --gid www-data -d /var/www -s /bin/bash \
 && chown -R www-data:www-data /var/www \
;

COPY docker-entrypoint.sh /docker-entrypoint.sh
RUN chmod +x /docker-entrypoint.sh

USER www-data

CMD ["/docker-entrypoint.sh"]

EXPOSE 9000
