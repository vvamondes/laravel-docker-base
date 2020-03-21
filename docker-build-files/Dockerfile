FROM ubuntu:18.04

# ENVS
ENV DEBIAN_FRONTEND=noninteractive
ENV TZ="America/Sao_Paulo"
ENV LANG en_US.utf8

# PACOTES
RUN apt-get update && apt-get install -y locales && rm -rf /var/lib/apt/lists/* \
    && localedef -i en_US -c -f UTF-8 -A /usr/share/locale/locale.alias en_US.UTF-8

RUN apt-get update && apt-get install -yq --no-install-recommends \
    rsyslog \
    apt-utils \
    curl \
    git \
    zip \
    curl \
    sudo \
    unzip \
    cron \
    vim \
    telnet \
    libicu-dev \
    libbz2-dev \
    libpng-dev \
    libjpeg-dev \
    libmcrypt-dev \
    libreadline-dev \
    libfreetype6-dev \
    tzdata \
    libpq-dev \
    libfreetype6-dev \
    libpng-dev \
    g++ \
    gcc \
    # Install git
    git \
    # Install apache
    apache2 \
    # Install php 7.2
    libapache2-mod-php7.2 \
    php7.2-common \
    php7.2-cli \
    php7.2-json \
    php7.2-curl \
    php7.2-fpm \
    php7.2-gd \
    php7.2-ldap \
    php7.2-mbstring \
    php7.2-mysql \
    php7.2-soap \
    php7.2-sqlite3 \
    php7.2-xml \
    php7.2-zip \
    php7.2-intl \
    php7.2-pgsql \
    php7.2-pdo-dblib \
    php-redis \
    php-bcmath \
    php-imagick \
    # Install tools
    openssl \
    vim \
    postgresql-client \
    iputils-ping \
    locales \
    ca-certificates \
    less \
    sendmail \
    mailutils \
    sendmail-bin \
    supervisor \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

# TIMEZONE
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

# APACHE
ENV APACHE_DOCUMENT_ROOT=/var/www/html/public
RUN sed -ri -e 's!/var/www/html!${APACHE_DOCUMENT_ROOT}!g' /etc/apache2/sites-available/*.conf
RUN sed -ri -e 's!/var/www/!${APACHE_DOCUMENT_ROOT}!g' /etc/apache2/apache2.conf /etc/apache2/conf-available/*.conf
RUN a2enmod rewrite headers

# COMPOSER
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# SETUP PPROJECT
COPY . /var/www/html
COPY .env /var/www/html/.env
WORKDIR /var/www/html

RUN composer install
RUN php artisan key:generate
RUN php artisan telescope:install
RUN php artisan telescope:publish
RUN php artisan horizon:install
RUN php artisan horizon:publish

RUN chmod -R 755 /var/www/html/storage
#RUN find /var/www/html -type f -exec chmod 644 {} \;
RUN find /var/www/html -type d -exec chmod 755 {} \;
RUN chgrp -R www-data storage bootstrap/cache
RUN chmod -R ug+rwx storage bootstrap/cache
RUN chmod 777 -R /var/www/html/storage/logs

# CRON
COPY docker-build-files/crontab /etc/cron.d/crontab
RUN crontab /etc/cron.d/crontab
RUN chmod -R 0600 /etc/cron.d/crontab
RUN chmod -R 0600 /var/spool/cron/crontabs/root
RUN touch /var/log/cron.log
COPY docker-build-files/000-default.conf /etc/apache2/sites-available/000-default.conf
COPY docker-build-files/laravel-worker.conf /etc/supervisor/conf.d

# SHOW DATE
RUN date

# PORT
EXPOSE 80

# UP
CMD service apache2 start && \
    php artisan migrate --force && \
    php artisan db:seed && \
    supervisord && \
    supervisorctl reread && \
    supervisorctl update && \
    supervisorctl start laravel-worker:* && \
    cron && \
    (sed -i '/session    required     pam_loginuid.so/c\#session    required   pam_loginuid.so' /etc/pam.d/cron) && \
    tail -F /var/log/cron.log
