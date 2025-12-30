FROM php:8.4-fpm

# 1. Nastavení ID uživatele (Sjednocení s NASem pro eliminaci 403 a pomalého zápisu)
ARG NAS_UID=1024

# 2. Instalace systémových balíčků
RUN apt-get update && apt-get install -y \
    nginx \
    openssh-server \
    supervisor \
    git \
    unzip \
    zlib1g-dev \
    libzip-dev \
    libpq-dev \
    libpng-dev libwebp-dev libjpeg62-turbo-dev libfreetype6-dev \
    libonig-dev \
    libxml2-dev \
    curl \
    ffmpeg \
    nano \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

# 3. Instalace PHP rozšíření
RUN docker-php-ext-configure gd --with-freetype --with-jpeg --with-webp
RUN docker-php-ext-install pdo_pgsql pgsql mbstring exif pcntl bcmath gd zip
RUN pecl install redis && docker-php-ext-enable redis

# 4. Instalace Node.js a Composeru
RUN curl -fsSL https://deb.nodesource.com/setup_20.x | bash - && apt-get install -y nodejs
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# 5. Nastavení uživatelů a SSH
# Vytvoření deployera se stejným ID jako máš na NASu, přidání do www-data
RUN useradd -u ${NAS_UID} -g www-data -m -s /bin/bash deployer

# Nastavení hesel (změň si 'root' na něco bezpečnějšího)
RUN echo 'root:root' | chpasswd && \
    echo 'deployer:root' | chpasswd

# Konfigurace SSHD
RUN mkdir -p /var/run/sshd && \
    sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin no/' /etc/ssh/sshd_config && \
    echo "AllowUsers deployer" >> /etc/ssh/sshd_config

# 6. Nastavení Shellu (Prompt a složka)
RUN echo 'export PS1="\[\e[32m\]\u@\H\[\e[m\]:\[\e[34m\]\w\[\e[m\]\\$ "' >> /root/.bashrc && \
    echo 'cd /var/www/html' >> /home/deployer/.bashrc && \
    echo 'export PS1="\[\e[33m\]\u@\H\[\e[m\]:\[\e[34m\]\w\[\e[m\]\\$ "' >> /home/deployer/.bashrc

# 7. Příprava aplikace a Nginxu
WORKDIR /var/www/html

# Odstranění default webu a kopie tvého configu
RUN rm /etc/nginx/sites-enabled/default
COPY nginx.conf /etc/nginx/sites-available/laravel.conf
RUN ln -s /etc/nginx/sites-available/laravel.conf /etc/nginx/sites-enabled/

# 8. Práva a složky (Klíčové proti 403 Forbidden)
# Musíme zajistit, aby Nginx mohl procházet cestu /var/www
RUN chmod 755 /var/www && \
    mkdir -p /var/www/html/storage/logs && \
    mkdir -p /var/log/supervisor && \
    chown -R deployer:www-data /var/www/html && \
    chmod -R 775 /var/www/html

# 9. Supervisor konfigurace
COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf

# Porty pro Web a SSH
EXPOSE 80 22

# Spuštění přes Supervisor
CMD ["/usr/bin/supervisord"]