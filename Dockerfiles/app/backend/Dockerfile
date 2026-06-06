FROM php:8.4-fpm

# Définir le répertoire de travail
WORKDIR /var/www/smartrh

# Installation des dépendances système
RUN apt-get update && apt-get install -y \
    git curl libpng-dev libonig-dev libxml2-dev \
    zip unzip libzip-dev libfreetype6-dev libjpeg-dev \
    libicu-dev wget default-mysql-client \
    && rm -rf /var/lib/apt/lists/*

# Installation des extensions PHP
RUN docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install -j$(nproc) \
    pdo_mysql mbstring exif pcntl bcmath gd zip intl opcache

# Installation de Redis
RUN pecl install redis && docker-php-ext-enable redis

# Installation de wkhtmltopdf
RUN apt-get update && apt-get install -y \
    xvfb xfonts-75dpi xfonts-base \
    && wget https://github.com/wkhtmltopdf/packaging/releases/download/0.12.6.1-2/wkhtmltox_0.12.6.1-2.bullseye_amd64.deb \
    && dpkg -i wkhtmltox_0.12.6.1-2.bullseye_amd64.deb || apt-get install -yf \
    && rm -rf /var/lib/apt/lists/* wkhtmltox_0.12.6.1-2.bullseye_amd64.deb \
    && ln -s /usr/local/bin/wkhtmltopdf /usr/bin/wkhtmltopdf \
    && ln -s /usr/local/bin/wkhtmltoimage /usr/bin/wkhtmltoimage

# Configuration de PHP-FPM pour écouter sur le port 9000
RUN echo "listen = 0.0.0.0:9000" >> /usr/local/etc/php-fpm.d/www.conf

# --- CORRECTION : Copie du script d'entrée depuis le bon chemin ---
# Le chemin doit correspondre à la structure de votre repo GitHub
COPY Dockerfiles/app/backend/entrypoint.sh /usr/local/bin/entrypoint.sh
RUN chmod +x /usr/local/bin/entrypoint.sh

# Copie de tout le code source de l'application
COPY . .

# Configuration des permissions pour Laravel
RUN mkdir -p storage/framework/{sessions,views,cache} bootstrap/cache \
    && chown -R www-data:www-data storage bootstrap/cache \
    && chmod -R 775 storage bootstrap/cache

# Exposer le port 9000
EXPOSE 9000

# Définir l'entrypoint et la commande de démarrage
ENTRYPOINT ["/usr/local/bin/entrypoint.sh"]

# CORRECTION : Utiliser php-fpm (le nom générique dans l'image officielle)
CMD ["php-fpm"]