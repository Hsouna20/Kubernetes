# Use the official PHP image as the base image
FROM php:8.0-cli

# Set the working directory inside the container
WORKDIR /var/www

# Install necessary packages
RUN apt-get update && apt-get install -y \
    git \
    zip \
    unzip

# Install Composer globally
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

# Add Composer's global bin directory to the PATH
ENV PATH="${PATH}:/root/.composer/vendor/bin"

# Install Laravel installer globally
RUN composer global require laravel/installer

# Create a new Laravel application
RUN laravel new my-laravel-app

# Set permissions to the Laravel storage and bootstrap/cache directories
RUN chown -R www-data:www-data my-laravel-app/storage my-laravel-app/bootstrap/cache

# Set the working directory to the Laravel application
WORKDIR /var/www/my-laravel-app
