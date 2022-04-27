#
# PHP Dependencies
#
FROM composer:latest as vendor

WORKDIR /

COPY database/ database/

COPY composer.json composer.json
COPY composer.lock composer.lock

RUN composer install \
    --ignore-platform-reqs \
    --no-dev \
    --no-interaction \
    --no-plugins \
    --no-scripts \
    --prefer-dist

#
# Frontend
#
FROM node:latest as frontend

COPY . /var/www

RUN cd /var/www && \
    npm install && \
    npm run prod

#
# Application
#
FROM rpopuc/php-fpm-nginx:8.0

USER root

RUN rm -rf /var/lib/apt/lists/* && \
    apt-get update && \
    apt-get install -y mc

RUN docker-php-ext-install gd

COPY . /var/www
COPY .ci/config/nginx/nginx.conf /etc/nginx/nginx.conf
COPY --from=vendor /vendor/ /var/www/vendor/
COPY --from=frontend /var/www/public/js/ /var/www/public/js/
COPY --from=frontend /var/www/public/css/ /var/www/public/css/
COPY --from=frontend /var/www/public/mix-manifest.json /var/www/public/mix-manifest.json

RUN chown -R 1000.1000 /var/www

USER www-data

COPY .ci/.env.prod /var/www/.env

RUN composer du

WORKDIR /var/www
