FROM php:8.3-fpm

# Системные зависимости
RUN apt-get update && apt-get install -y \
    git \
    unzip \
    libicu-dev \
    libpq-dev \
    libzip-dev \
    libonig-dev \
    zip \
    curl \
    && docker-php-ext-install \
        intl \
        pdo \
        pdo_mysql \
        pdo_pgsql \
        opcache \
        zip

# Xdebug
RUN pecl install xdebug \
    && docker-php-ext-enable xdebug
COPY ./docker/php/xdebug.ini /usr/local/etc/php/conf.d/xdebug.ini

# Composer
COPY --from=composer:2 /usr/bin/composer /usr/bin/composer

# Node.js (для сборки фронта)
RUN curl -fsSL https://deb.nodesource.com/setup_20.x | bash - \
    && apt-get install -y nodejs

WORKDIR /var/www/symfony

# Копируем composer.* для кэша зависимостей
COPY symfony/composer.* ./

RUN composer install --no-scripts --no-progress --prefer-dist --no-interaction

# Копируем проект
COPY symfony/ ./

# Сборка фронта (если есть)
RUN if [ -f package.json ]; then npm install && npm run build; fi

# Оптимизация Symfony
RUN php bin/console cache:clear --env=prod && \
    php bin/console cache:warmup --env=prod

RUN chown -R www-data:www-data /var/www/symfony

EXPOSE 9000

CMD ["php-fpm"]
