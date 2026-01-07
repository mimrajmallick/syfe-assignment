FROM wordpress:6.1-php8.1-apache

# Install extra PHP extensions
RUN apt-get update && apt-get install -y \
    libzip-dev \
    libpng-dev \
    && docker-php-ext-install zip gd \
    && apt-get clean

# Copy custom config
COPY wp-config.php /usr/src/wordpress/

# Set permissions
RUN chown -R www-data:www-data /var/www/html

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=30s --retries=3 \
    CMD curl -f http://localhost/ || exit 1

EXPOSE 80
