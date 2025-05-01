FROM php:8.4-fpm

# Install system packages
RUN apt-get update && apt-get install -y \
    nginx \
    supervisor \
    git \
    unzip \
    ghostscript \
    imagemagick \
    libpng-dev \
    libjpeg-dev \
    libfreetype6-dev \
    libzip-dev \
    libonig-dev \
    libxml2-dev \
    libcurl4-openssl-dev \
    libicu-dev \
    libxslt-dev \
    libpq-dev \
    libssl-dev \
    libmagickwand-dev \
    libmemcached-dev \
    openssh-server \
    openssh-client \
    locales \
    ruby \
    ruby-dev \
    nano \
    && docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install -j$(nproc) \
        pdo \
        pdo_mysql \
        gd \
        mbstring \
        zip \
        opcache \
        intl \
        xsl \
        mysqli \
        xml

# Set up locales
RUN sed -i -e 's/# en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/' /etc/locale.gen && \
    locale-gen
ENV LANG en_US.UTF-8
ENV LANGUAGE en_US:en
ENV LC_ALL en_US.UTF-8

# Install PECL extensions
RUN pecl install imagick && docker-php-ext-enable imagick
RUN pecl install redis && docker-php-ext-enable redis
RUN pecl install memcached && docker-php-ext-enable memcached

# Install sass and compass
RUN gem install sass
RUN gem install compass

# Configure SSH server
RUN mkdir /var/run/sshd
# Set root password to 'root'
RUN echo 'root:root' | chpasswd
RUN sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config
# Explicitly allow password authentication in the server config
RUN sed -i 's/#PasswordAuthentication yes/PasswordAuthentication yes/' /etc/ssh/sshd_config

# Allow reading PDF via imagick
RUN sed -i 's/<policy domain="coder" rights="none" pattern="PDF" \/>/<policy domain="coder" rights="read|write" pattern="PDF" \/>/' /etc/ImageMagick-6/policy.xml

# Clean up
RUN apt-get clean && rm -rf /var/lib/apt/lists/*

# Optional: install Composer globally
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# Create global Drush alias
RUN echo 'alias drush="/var/www/vendor/bin/drush"' >> /etc/bash.bashrc

# Supervisor config to run both PHP-FPM and NGINX
COPY ./supervisord.conf /etc/supervisor/conf.d/supervisord.conf

# Create entrypoint script to check for project-specific config files
RUN echo '#!/bin/bash\n\
DOCKER_CONFIG_DIR="/var/www/.docker"\n\
\n\
# Check and copy nginx config if exists\n\
if [ -f "$DOCKER_CONFIG_DIR/etc/nginx/nginx.conf" ]; then\n\
  echo "Using project-specific nginx config"\n\
  cp "$DOCKER_CONFIG_DIR/etc/nginx/nginx.conf" /etc/nginx/nginx.conf\n\
fi\n\
\n\
# Check and copy PHP config if exists\n\
if [ -f "$DOCKER_CONFIG_DIR/etc/php/php.ini" ]; then\n\
  echo "Using project-specific php.ini"\n\
  cp "$DOCKER_CONFIG_DIR/etc/php/php.ini" /usr/local/etc/php/conf.d/zzz-project.ini\n\
fi\n\
\n\
# Check and copy PHP-FPM config if exists\n\
if [ -f "$DOCKER_CONFIG_DIR/etc/php/php-fpm.conf" ]; then\n\
  echo "Using project-specific php-fpm.conf"\n\
  cp "$DOCKER_CONFIG_DIR/etc/php/php-fpm.conf" /usr/local/etc/php-fpm.d/zzz-project.conf\n\
fi\n\
\n\
# Check and copy supervisor config if exists\n\
if [ -f "$DOCKER_CONFIG_DIR/supervisord.conf" ]; then\n\
  echo "Using project-specific supervisord.conf"\n\
  cp "$DOCKER_CONFIG_DIR/supervisord.conf" /etc/supervisor/conf.d/supervisord.conf\n\
fi\n\
\n\
# Start SSH service\n\
service ssh start\n\
\n\
# Execute the command passed to docker run\n\
exec "$@"' > /usr/local/bin/docker-entrypoint.sh

RUN chmod +x /usr/local/bin/docker-entrypoint.sh

WORKDIR /var/www

EXPOSE 80 22

ENTRYPOINT ["/usr/local/bin/docker-entrypoint.sh"]
CMD ["/usr/bin/supervisord"]
