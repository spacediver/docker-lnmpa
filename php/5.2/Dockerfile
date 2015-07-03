FROM eboraas/apache:wheezy

# Install needed packages
RUN apt-get update && apt-get install -y \
	autoconf ca-certificates curl file gcc \
	libjpeg62 libmysqlclient18 libpng12-0 libxml2 \
	make pkg-config sendmail --no-install-recommends \
	&& rm -r /var/lib/apt/lists/*

ENV PHP_INI_DIR /usr/local/etc/php

RUN mkdir -p $PHP_INI_DIR/conf.d

ENV PHP_EXTRA_CONFIGURE_ARGS --enable-fastcgi --enable-force-cgi-redirect --with-apxs2=/usr/bin/apxs2

ENV PHP_VERSION 5.2.17

COPY php-5.2.17-*.patch /tmp/

RUN rm -rf /var/www/html && mkdir -p /var/lock/apache2 /var/run/apache2 /var/log/apache2 /var/www/html && chown -R www-data:www-data /var/lock/apache2 /var/run/apache2 /var/log/apache2 /var/www/html

RUN mv /etc/apache2/apache2.conf /etc/apache2/apache2.conf.dist && rm -f /etc/apache2/conf-enabled/* /etc/apache2/sites-enabled/*
COPY apache2.conf /etc/apache2/apache2.conf

ENV PHP_EXTRA_BUILD_DEPS apache2-threaded-dev

RUN buildDeps=" \
		$PHP_EXTRA_BUILD_DEPS \
		bzip2 \
		libcurl4-openssl-dev \
		libjpeg-dev \
		libmysqlclient-dev \
		libpng12-dev \
		libreadline6-dev \
		libssl-dev \
		libxml2-dev \
		patch \
	"; \
	set -x \
	&& apt-get update && apt-get install -y $buildDeps --no-install-recommends && rm -rf /var/lib/apt/lists/* \
	&& curl -SL "http://museum.php.net/php5/php-$PHP_VERSION.tar.bz2" -o php.tar.bz2 \
	&& mkdir -p /usr/src/php \
	&& tar -xf php.tar.bz2 -C /usr/src/php --strip-components=1 \
	&& rm php* \
	&& cd /usr/src/php \
	&& patch -p1 < /tmp/php-5.2.17-libxml2.patch \
	&& patch -p1 < /tmp/php-5.2.17-openssl.patch \
	&& ln -s /usr/lib/x86_64-linux-gnu/libjpeg.a /usr/lib/libjpeg.a \
	&& ln -s /usr/lib/x86_64-linux-gnu/libjpeg.so /usr/lib/libjpeg.so \
	&& ln -s /usr/lib/x86_64-linux-gnu/libpng.a /usr/lib/libpng.a \
	&& ln -s /usr/lib/x86_64-linux-gnu/libpng.so /usr/lib/libpng.so \
	&& ln -s /usr/lib/x86_64-linux-gnu/libmysqlclient.so /usr/lib/libmysqlclient.so \
	&& ln -s /usr/lib/x86_64-linux-gnu/libmysqlclient.a /usr/lib/libmysqlclient.a \
	&& ./configure \
		--with-config-file-path="$PHP_INI_DIR" \
		--with-config-file-scan-dir="$PHP_INI_DIR/conf.d" \
		$PHP_EXTRA_CONFIGURE_ARGS \
		--enable-mbstring \
		--enable-pdo \
		--enable-soap \
		--with-curl \
		--with-gd \
		--with-jpeg-dir=/usr \
		--with-png-dir=/usr \
		--with-mysql \
		--with-mysqli \
        --with-libdir=/lib/x86_64-linux-gnu \
		--with-openssl \
		--with-pdo-mysql \
		--with-readline \
		--with-zlib \
	&& sed -i 's/-lxml2 -lxml2 -lxml2/-lcrypto -lssl/' Makefile \
	&& make -j"$(nproc)" \
	&& make install \
	&& { find /usr/local/bin /usr/local/sbin -type f -executable -exec strip --strip-all '{}' + || true; } \
	&& apt-get purge -y --auto-remove $buildDeps \
	&& make clean

COPY docker-php-ext-* /usr/local/bin/

# Setup timezone to Etc/UTC and fix extension path
RUN cat /usr/src/php/php.ini-recommended | sed 's/^;\(date.timezone.*\)/\1 \"Etc\/UTC\"/' > /usr/local/etc/php/php.ini \
	&& sed -i 's/\(extension_dir = \)\"\.\/\"/\1\"\/usr\/local\/lib\/php\/extensions\/no-debug-non-zts-20060613\/\"/' /usr/local/etc/php/php.ini \
	&& sed -i 's/^short_open_tag.*/short_open_tag = On/ /usr/local/etc/php/php.ini

COPY apache2-foreground /usr/local/bin/
WORKDIR /var/www/html

EXPOSE 80
CMD ["apache2-foreground"]
