FROM php:8.3-fpm

ARG APP_ID=1000
RUN groupadd -g "$APP_ID" app \
  && useradd -g "$APP_ID" -u "$APP_ID" -d /var/www -s /bin/bash app

RUN mkdir -p /etc/nginx/html /var/www/html /sock \
  && chown -R app:app /etc/nginx /var/www /usr/local/etc/php/conf.d /sock

# Change the PHP-FPM configuration to run as 'app'
RUN sed -i 's/user = www-data/user = app/' /usr/local/etc/php-fpm.d/www.conf
RUN sed -i 's/group = www-data/group = app/' /usr/local/etc/php-fpm.d/www.conf
RUN sed -i 's/;listen.owner = www-data/listen.owner = app/' /usr/local/etc/php-fpm.d/www.conf
RUN sed -i 's/;;listen.group/;listen.group/' /usr/local/etc/php-fpm.d/www.conf
RUN sed -i 's/;listen.group = www-data/listen.group = app/' /usr/local/etc/php-fpm.d/www.conf

RUN curl -fsSL https://deb.nodesource.com/setup_20.x | bash -

RUN apt-get update && apt-get install -y \
    cron \
    default-mysql-client \
    git \
    gnupg \
    gzip \
    libbz2-dev \
    libfreetype6-dev \
    libicu-dev \
    libjpeg62-turbo-dev \
    libmagickwand-dev \
    libmcrypt-dev \
    libonig-dev \
    libpng-dev \
    libsodium-dev \
    libssh2-1-dev \
    libwebp-dev \
    libxslt1-dev \
    libzip-dev \
    lsof \
    mailutils \
    msmtp \
    nodejs \
    procps \
    strace \
    vim \
    zip \
    zlib1g-dev \
  && rm -rf /var/lib/apt/lists/*

RUN pecl channel-update pecl.php.net && pecl install \
    redis-6.0.2 \
    ssh2-1.3.1 \
    swoole-5.1.1 \
    xdebug-3.3.2 \
  && pecl clear-cache \
  && rm -rf /tmp/pear

RUN curl -L https://github.com/Imagick/imagick/archive/28f27044e435a2b203e32675e942eb8de620ee58.zip -o imagick.zip \
    && unzip imagick.zip \
    && rm imagick.zip \
    && cd imagick-28f27044e435a2b203e32675e942eb8de620ee58 \
    && phpize \
    && ./configure --with-php-config=/usr/local/bin/php-config \
    && make \
    && make install \
    && echo "extension=imagick.so" >> $PHP_INI_DIR/conf.d/imagick.ini \
    && cd .. \
    && rm -rf imagick-28f27044e435a2b203e32675e942eb8de620ee58

RUN docker-php-ext-configure \
    gd --with-freetype --with-jpeg --with-webp \
  && docker-php-ext-install \
    bcmath \
    bz2 \
    calendar \
    exif \
    ftp \
    gd \
    gettext \
    intl \
    mbstring \
    mysqli \
    opcache \
    pcntl \
    pdo_mysql \
    soap \
    sockets \
    sodium \
    sysvmsg \
    sysvsem \
    sysvshm \
    xsl \
    zip \
  && docker-php-ext-enable \
    imagick \
    redis \
    ssh2 \
    xdebug

RUN curl -sS https://getcomposer.org/installer | \
  php -- --version=2.6.6 --install-dir=/usr/local/bin --filename=composer

RUN mkdir -p /var/www/html/var /var/www/html/pub/static /var/www/html/generated /vendor && \
    chown -R app:app /var/www/html/var /var/www/html/pub/static /var/www/html/generated /vendor

USER app:app
VOLUME /var/www
WORKDIR /var/www/html