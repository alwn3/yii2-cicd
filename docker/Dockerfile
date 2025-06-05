FROM php:8.2-apache

# Install required system packages and PHP extensions
RUN apt-get update && apt-get install -y \
<<<<<<< HEAD
    git zip unzip libzip-dev libpq-dev libonig-dev libicu-dev \
    && docker-php-ext-install pdo pdo_mysql zip mbstring intl \
    && curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
=======
    git zip unzip libzip-dev libpq-dev libonig-dev curl \
    && docker-php-ext-install pdo pdo_mysql zip
>>>>>>> 296efe8 (site worked in docker run expose)

# Enable Apache mod_rewrite
RUN a2enmod rewrite

# Configure Apache DocumentRoot to /var/www/html/web
RUN sed -i 's|DocumentRoot /var/www/html|DocumentRoot /var/www/html/web|' /etc/apache2/sites-available/000-default.conf \
    && echo '<Directory "/var/www/html/web">\n\
        AllowOverride All\n\
        Require all granted\n\
    </Directory>' >> /etc/apache2/apache2.conf

# Install Composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

# Set working directory
WORKDIR /var/www/html

# Copy source code
COPY ./src/ /var/www/html

# Clean old dependencies
RUN rm -rf /var/www/html/vendor /var/www/html/composer.lock

# Install PHP dependencies fresh
RUN composer install --optimize-autoloader

# Set proper permissions
RUN chown -R www-data:www-data /var/www/html \
    && chmod -R 755 /var/www/html

# Expose HTTP port
EXPOSE 80

# Start Apache in the foreground
CMD ["apache2-foreground"]
