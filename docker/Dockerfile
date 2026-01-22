FROM php:8.2-fpm-bullseye

# Install system dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
    curl \
    wget \
    git \
    zip \
    unzip \
    libzip-dev \
    libpng-dev \
    libjpeg62-turbo-dev \
    libfreetype6-dev \
    libpq-dev \
    nodejs \
    npm \
    && rm -rf /var/lib/apt/lists/*

# Install PHP extensions
RUN docker-php-ext-configure gd --with-freetype --with-jpeg && \
    docker-php-ext-install -j$(nproc) \
    gd \
    zip \
    pdo \
    pdo_pgsql \
    bcmath \
    ctype \
    json \
    mbstring \
    tokenizer

# Install Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# Set working directory
WORKDIR /var/www/html

# Copy application files
COPY . .

# Install PHP dependencies
RUN composer install --no-interaction --optimize-autoloader

# Install Node dependencies (if needed for frontend build)
RUN npm install && npm run build || true

# Create necessary directories
RUN mkdir -p storage/logs storage/app/public storage/app/pdf \
    && chmod -R 777 storage bootstrap/cache

# Generate app key
RUN php artisan key:generate --force || true

EXPOSE 8000

CMD ["php-fpm"]
