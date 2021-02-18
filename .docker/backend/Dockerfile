FROM php:8.0-fpm-alpine as backend

ARG APP_ENV

LABEL maintainer="Christian Meinhard"

ARG TIME_ZONE='Europe/Berlin'
ENV DEFAULT_TZ ${TIME_ZONE}

WORKDIR /var/www

# Import extension installer
COPY --from=mlocati/php-extension-installer /usr/bin/install-php-extensions /usr/bin/

# Install extensions
RUN install-php-extensions bcmath pdo_mysql redis

# Install the xdebug extension when APP_ENV is local
RUN if [ ${APP_ENV} = local ]; then \
    install-php-extensions xdebug \
    && echo "xdebug.mode=debug" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini \
    && echo "xdebug.client_host = host.docker.internal" >>/usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini \
;fi

# Install Composer
COPY --from=composer:2.0 /usr/bin/composer /usr/local/bin/composer

# Install extra packages
RUN apk --no-cache add bash mysql-client

#Set correct time
RUN apk add tzdata\
    && cp /usr/share/zoneinfo/${DEFAULT_TZ} /etc/localtime \
    && apk del tzdata

# Add UID '1000' to www-data
RUN apk add shadow && usermod -u 1000 www-data && groupmod -g 1000 www-data

# Copy existing application directory permissions
COPY --chown=www-data:www-data . /var/www

# Change current user to www-data
USER www-data

FROM backend as worker

# Start worker for Laravel Applications
CMD ["php", "/var/www/artisan", "queue:work"]