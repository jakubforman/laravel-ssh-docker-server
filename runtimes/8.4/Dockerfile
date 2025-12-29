FROM php:8.4-fpm

# 1. Přidán nginx
RUN apt-get update && apt-get install -y \
    nginx \
    openssh-server \
    supervisor \
    git \
    unzip \
    zlib1g-dev \
    libzip-dev \
    libpq-dev \
    libpng-dev \
    libonig-dev \
    libxml2-dev \
    curl \
    ffmpeg \
    nano

RUN docker-php-ext-install pdo_pgsql pgsql mbstring exif pcntl bcmath gd

RUN curl -fsSL https://deb.nodesource.com/setup_20.x | bash - && apt-get install -y nodejs
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# 2. SSH nastavení (opraveno s -p)
RUN mkdir -p /var/run/sshd
RUN echo 'root:root' | chpasswd
RUN sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config

# 3. Odstraníme výchozí web Nginxu a vytvoříme si vlastní konfiguraci
RUN rm /etc/nginx/sites-enabled/default
COPY nginx.conf /etc/nginx/sites-available/laravel.conf
RUN ln -s /etc/nginx/sites-available/laravel.conf /etc/nginx/sites-enabled/

# 4. Supervisor konfigurace
COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf

WORKDIR /var/www/html
EXPOSE 80 22

CMD ["/usr/bin/supervisord"]