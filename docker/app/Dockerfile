FROM php:7.3-fpm-alpine
ARG github_username
ARG github_password
RUN apk update && apk add git curl libpng-dev oniguruma-dev libxml2-dev vim zip unzip sudo bash
RUN docker-php-ext-install pdo_mysql mbstring exif pcntl bcmath gd
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer
ADD . /usr/share/nginx/html/laravel
WORKDIR /usr/share/nginx/html/laravel
RUN composer update
RUN chmod -R a+w storage/ bootstrap/cache
RUN cp .env.example .env
RUN php artisan key:generate