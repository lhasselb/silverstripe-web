FROM php:5.6-apache
LABEL maintainer Brett Tasker "<brett@silverstripe.com>"
ENV DEBIAN_FRONTEND=noninteractive

# Install components
RUN apt-get update -y && apt-get install -y \
		curl \
		git-core \
		gzip \
		libcurl4-openssl-dev \
		libgd-dev \
        libjpeg-dev \
		libfreetype6-dev \
		libpng12-dev \
		libpng-dev \
		libldap2-dev \
		libmcrypt-dev \
		libtidy-dev \
		libxslt-dev \
		zlib1g-dev \
		libicu-dev \
		g++ \
		openssh-client \
		libmagickwand-dev \
	--no-install-recommends && \
	curl -sS https://silverstripe.github.io/sspak/install | php -- /usr/local/bin && \
	curl -sS https://getcomposer.org/installer | php && mv composer.phar /usr/local/bin/composer && \
	pecl install xdebug-2.2.7 && \
	pecl install imagick-3.4.3 && \
	apt-get remove -y libmagickwand-dev && \
	apt-get install -y libmagickwand-6.q16-2 && \
	rm -r /var/lib/apt/lists/*

# Install PHP Extensions
RUN docker-php-ext-configure intl && \
    docker-php-ext-configure gd \
		--enable-gd-native-ttf \
        --with-freetype-dir=/usr/include/freetype2 \
        --with-png-dir=/usr/include \
        --with-jpeg-dir=/usr/include && \
	docker-php-ext-configure mysql --with-mysql=mysqlnd && \
	docker-php-ext-configure mysqli --with-mysqli=mysqlnd && \
	docker-php-ext-configure ldap --with-libdir=lib/x86_64-linux-gnu/ && \
	docker-php-ext-enable xdebug && \
	docker-php-ext-enable imagick && \
	sed -i '1 a xdebug.remote_autostart=1' /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini && \
        sed -i '1 a xdebug.remote_mode=req' /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini && \
        sed -i '1 a xdebug.remote_handler=dbgp' /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini && \
        sed -i '1 a xdebug.remote_connect_back=0 ' /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini && \
        sed -i '1 a xdebug.remote_port=9005' /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini && \
        sed -i '1 a xdebug.remote_host=192.168.0.16' /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini && \
        sed -i '1 a xdebug.remote_enable=1' /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini && \
        sed -i '1 a xdebug.idekey=VSCODE' /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini && \
        sed -i '1 a xdebug.remote_log=/tmp/xdebug/xdebug.log' /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini && \
	docker-php-ext-install -j$(nproc) \
		intl \
		gd \
		ldap \
		mcrypt \
		mysql \
		mysqli \
		pdo \
		pdo_mysql \
		soap \
		tidy \
		xsl

# Apache + xdebug configuration
RUN { \
                echo "<VirtualHost *:80>"; \
                echo "  DocumentRoot /var/www/html"; \
                echo "  LogLevel warn"; \
                echo "  ErrorLog /var/log/apache2/error.log"; \
                echo "  CustomLog /var/log/apache2/access.log combined"; \
                echo "  ServerSignature Off"; \
                echo "  <Directory /var/www/html>"; \
                echo "    Options +FollowSymLinks"; \
                echo "    Options -ExecCGI -Includes -Indexes"; \
                echo "    AllowOverride all"; \
                echo; \
                echo "    Require all granted"; \
                echo "  </Directory>"; \
                echo "  <LocationMatch assets/>"; \
                echo "    php_flag engine off"; \
                echo "  </LocationMatch>"; \
                echo; \
                echo "  IncludeOptional sites-available/000-default.local*"; \
                echo "</VirtualHost>"; \
	} | tee /etc/apache2/sites-available/000-default.conf

RUN echo "ServerName localhost" > /etc/apache2/conf-available/fqdn.conf && \
	echo "date.timezone = Europe/Berlin" > /usr/local/etc/php/conf.d/timezone.ini && \
	a2enmod rewrite expires remoteip cgid && \
	usermod -u 1000 www-data && \
	usermod -G staff www-data

EXPOSE 80
CMD ["apache2-foreground"]
