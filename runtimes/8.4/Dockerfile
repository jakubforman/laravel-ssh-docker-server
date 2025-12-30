FROM php:8.4-fpm

# 1. User NAS ID settings (for DSM folders)
ARG NAS_UID=1024

# 2. Installing system packages
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

# 3. Installing PHP extensions
RUN docker-php-ext-configure gd --with-freetype --with-jpeg --with-webp
RUN docker-php-ext-install pdo_pgsql pgsql mbstring exif pcntl bcmath gd zip
RUN pecl install redis && docker-php-ext-enable redis

# 4. Installing Node.js and Composer
RUN curl -fsSL https://deb.nodesource.com/setup_20.x | bash - && apt-get install -y nodejs
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# 5. User and SSH settings (Secure)
# Creating a deployer without a password
RUN useradd -u ${NAS_UID} -g www-data -m -s /bin/bash deployer

# Creating a directory for SSH keys with strict permissions
RUN mkdir -p /home/deployer/.ssh && \
    chown deployer:www-data /home/deployer/.ssh && \
    chmod 700 /home/deployer/.ssh

# KSSHD configuration: No root, no passwords, only keys
RUN mkdir -p /var/run/sshd && \
    sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin no/' /etc/ssh/sshd_config && \
    sed -i 's/#PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config && \
    sed -i 's/#PubkeyAuthentication yes/PubkeyAuthentication yes/' /etc/ssh/sshd_config && \
    echo "AllowUsers deployer" >> /etc/ssh/sshd_config

# 6. Setup Shell
RUN echo 'export PS1="\[\e[32m\]\u@\H\[\e[m\]:\[\e[34m\]\w\[\e[m\]\\$ "' >> /root/.bashrc && \
    echo 'cd /var/www/html' >> /home/deployer/.bashrc && \
    echo 'export PS1="\[\e[33m\]\u@\H\[\e[m\]:\[\e[34m\]\w\[\e[m\]\\$ "' >> /home/deployer/.bashrc

# 7. Prepare Nginx config
WORKDIR /var/www/html
RUN rm /etc/nginx/sites-enabled/default
COPY nginx.conf /etc/nginx/sites-available/laravel.conf
RUN ln -s /etc/nginx/sites-available/laravel.conf /etc/nginx/sites-enabled/

# 8. Rules and folders
RUN chmod 755 /var/www && \
    mkdir -p /var/www/html/storage/logs && \
    mkdir -p /var/log/supervisor && \
    chown -R deployer:www-data /var/www/html && \
    chmod -R 775 /var/www/html && \
    touch /home/deployer/.ssh/authorized_keys && \
    chown deployer:www-data /home/deployer/.ssh/authorized_keys && \
    chmod 600 /home/deployer/.ssh/authorized_keys

# 9. Supervisor config file
COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf

EXPOSE 80 22

CMD ["/usr/bin/supervisord"]