FROM php:8.1.13-fpm-buster

RUN mkdir -p /var/cache/composer && chown -R www-data:www-data /var/cache/composer
ENV COMPOSER_HOME=/var/cache/composer

ENV PHPREDIS_VERSION 5.3.4

ENV NODEJS_VERSION 18

RUN php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');" && php composer-setup.php --install-dir=/usr/local/bin --filename=composer
RUN apt-get update && apt-get install -y \
        wget \
        libssl-dev \
        libjpeg62-turbo-dev \
        libpng-dev \
        libzip-dev \
        libicu-dev \
        libwebp-dev \
        libjpeg62-turbo-dev \
        libpng-dev libxpm-dev \
        libfreetype6-dev \
        git \
    && docker-php-ext-configure gd --with-freetype=/usr/include/ --with-jpeg=/usr/include/ \
    && docker-php-ext-install -j$(nproc) gd \
    && docker-php-ext-configure pdo_mysql \
    && docker-php-ext-install -j$(nproc) pdo_mysql \
    && docker-php-ext-configure zip \
    && docker-php-ext-install -j$(nproc) zip \
    && docker-php-ext-configure phar \
    && docker-php-ext-install -j$(nproc) phar \
    && docker-php-ext-configure intl \
    && docker-php-ext-install -j$(nproc) intl \
    && pecl install redis-$PHPREDIS_VERSION \
    && docker-php-ext-enable redis \
    # opcache directory
    && mkdir -p /usr/local/php/opcache && chown www-data:www-data /usr/local/php/opcache

RUN apt-get update && apt-get install -y procps

RUN pecl install xdebug \
    && docker-php-ext-enable xdebug

ADD php.ini /usr/local/etc/php/php.ini
ADD fpm.conf /usr/local/etc/php-fpm.d/www.conf

RUN curl -fsSL https://deb.nodesource.com/setup_$NODEJS_VERSION.x | bash - && apt install -y nodejs
RUN npm install -D tailwindcss@latest postcss@latest autoprefixer@latest

RUN mkdir -p /var/lib/nginx/body /var/lib/nginx/proxy /var/lib/nginx/fastcgi && \
    apt install -y nginx sudo supervisor && \
    mkdir -p /var/log/supervisor

ADD nginx /etc/nginx
ADD supervisor/supervisord.conf /etc/supervisor/conf.d/supervisord.conf

EXPOSE 80 5173 9000

WORKDIR /var/www

ADD entrypoint.sh /
CMD ["bash", "/entrypoint.sh"]