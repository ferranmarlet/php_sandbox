ARG COMPOSER_VERSION="1.9.3"
ARG DEBIAN_TAG="stable-20191224-slim"
ARG PHP_VERSION="7.3"

FROM composer:${COMPOSER_VERSION} AS composer

FROM debian:${DEBIAN_TAG}

ENV DEBIAN_FRONTEND noninteractive

# Install minimal system requirements
RUN true \
 && apt-get -qq update \
 && apt-get -qy install --no-install-recommends \
        ca-certificates \
        wget \
 && rm -rf /var/lib/apt/lists/* \
;

# Install minimal PHP composer dependencies
ARG PHP_VERSION
RUN true \
 && wget -qq -O /etc/apt/trusted.gpg.d/php.gpg "https://packages.sury.org/php/apt.gpg" \
 && echo "deb https://packages.sury.org/php/ $(. /etc/os-release && echo $VERSION_CODENAME) main" > /etc/apt/sources.list.d/php${PHP_VERSION}.list \
 && apt-get -qq update \
 && apt-get -qy install --no-install-recommends \
        "php${PHP_VERSION}-cli" \
        "php${PHP_VERSION}-curl" \
        "php${PHP_VERSION}-mbstring" \
        "php${PHP_VERSION}-xml" \
        "php${PHP_VERSION}-zip" \
        unzip \
 && rm -rf /var/lib/apt/lists/* \
;

# Application specific dependencies
RUN true \
 && apt-get -qq update \
 && apt-get -qy install --no-install-recommends \
        "php${PHP_VERSION}-bcmath" \
        "php${PHP_VERSION}-fpm" \
        "php${PHP_VERSION}-json" \
        "php${PHP_VERSION}-mysql" \
        "php${PHP_VERSION}-odbc" \
        "php${PHP_VERSION}-opcache" \
        "php${PHP_VERSION}-pgsql" \
        "php${PHP_VERSION}-soap" \
        "php${PHP_VERSION}-xmlrpc" \
        "php${PHP_VERSION}-xsl" \
 && rm -rf /var/lib/apt/lists/* \
 && ln -s "/usr/sbin/php-fpm${PHP_VERSION}" /usr/sbin/php-fpm \
 && ln -s "/etc/init.d/php${PHP_VERSION}-fpm" /etc/init.d/php-fpm \
 && mkdir /run/php/ \
;

# Install Microsoft ODBC Sql
RUN true \
 && wget -qq -O /etc/apt/trusted.gpg.d/microsoft.asc "https://packages.microsoft.com/keys/microsoft.asc" \
 && echo "deb https://packages.microsoft.com/debian/10/prod $(. /etc/os-release && echo $VERSION_CODENAME) main" > /etc/apt/sources.list.d/microsoft.list \
 && apt-get -qq update \
 && ACCEPT_EULA=Y \
    apt-get -qy install --no-install-recommends \
        msodbcsql17 \
 && rm -rf /var/lib/apt/lists/* \
;

# Build and configure SqlSrv extensions
RUN true \
 && apt-get -qq update \
 && apt-get -qy install --no-install-recommends \
        build-essential \
        php${PHP_VERSION}-dev \
        php-pear \
        unixodbc-dev \
 && pecl install \
        pdo_sqlsrv \
        sqlsrv \
 && echo "extension = pdo_sqlsrv.so \n; priority = 20" > "/etc/php/${PHP_VERSION}/mods-available/pdo_sqlsrv.ini" \
 && echo "extension = sqlsrv.so \n; priority = 20" > "/etc/php/${PHP_VERSION}/mods-available/sqlsrv.ini" \
 && phpenmod \
        pdo_sqlsrv \
        sqlsrv \
 && apt-get -qy purge --auto-remove \
        build-essential \
        php${PHP_VERSION}-dev \
        php-pear \
        unixodbc-dev \
 && rm -rf \
        /tmp/pear \
        /var/lib/apt/lists/* \
;

# Install Webkit HTML to PDF
RUN true \
 && wget -qq -O /tmp/wkhtmltox.deb "https://github.com/wkhtmltopdf/wkhtmltopdf/releases/download/0.12.5/wkhtmltox_0.12.5-1.$(. /etc/os-release && echo $VERSION_CODENAME)_amd64.deb" \
 && apt-get -qq update \
 && apt-get -qy install --no-install-recommends \
        /tmp/wkhtmltox.deb \
 && ln /usr/local/bin/wkhtmltopdf /usr/bin/wkhtmltopdf \
 && rm -rf \
        /tmp/wkhtmltox.deb \
        /var/lib/apt/lists/* \
;

# Install NodeJS
RUN true \
 && wget -qq -O- "https://deb.nodesource.com/setup_10.x" | bash - \
 && apt-get -qy install --no-install-recommends \
        nodejs \
 && apt-get -qy purge --auto-remove \
        lsb-release \
        gnupg \
 && rm -rf /var/lib/apt/lists/* \
;

# Install dev tools
RUN true \
 && apt-get -qq update \
 && apt-get -qy install --no-install-recommends \
        "php${PHP_VERSION}-xdebug" \
        sudo \
        unzip \
 && phpdismod xdebug \
 && rm -rf /var/lib/apt/lists/* \
;

# Modify www-data user to match system user
ARG GROUP_ID=1001
ARG USER_ID=1001
RUN true \
 && groupmod www-data --gid "${GROUP_ID}" \
 && usermod www-data --uid "${USER_ID}" --gid www-data -d /var/www -s /bin/bash \
 && mkdir -p /var/www \
 && chown -R www-data:www-data /var/www \
 && echo 'www-data ALL=(ALL) NOPASSWD:SETENV: /etc/init.d/php-fpm *' > /etc/sudoers.d/www-data \
;

# Install Composer
COPY --from=composer /usr/bin/composer /usr/bin/composer

COPY docker-entrypoint.sh /docker-entrypoint.sh
RUN chmod +x /docker-entrypoint.sh

USER www-data

CMD ["/docker-entrypoint.sh"]

EXPOSE 9000
