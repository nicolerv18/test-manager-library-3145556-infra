# PHP 8.3 runtime used by the `api` service: Laravel + PostgreSQL + Composer.
FROM php:8.3-cli

RUN apt-get update \
 && apt-get install -y --no-install-recommends git unzip libpq-dev libzip-dev \
 && docker-php-ext-install pdo_pgsql pgsql zip bcmath \
 && rm -rf /var/lib/apt/lists/*

COPY --from=composer:2 /usr/bin/composer /usr/local/bin/composer
ENV COMPOSER_ALLOW_SUPERUSER=1

# The cover limit (2 MB) is enforced by the API and answered as a contract error (422).
# PHP's own upload limits must be higher so the request reaches the application first.
RUN { \
      echo "upload_max_filesize=8M"; \
      echo "post_max_size=10M"; \
      echo "date.timezone=UTC"; \
      echo "expose_php=0"; \
    } > /usr/local/etc/php/conf.d/library.ini

WORKDIR /app
