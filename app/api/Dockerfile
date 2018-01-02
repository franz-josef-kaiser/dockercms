FROM php:7-alpine

ARG MODE
ENV MODE ${MODE:-dev}

# Date and Timezohne
RUN ln -snf /usr/share/zoneinfo/${TIMEZONE} /etc/localtime \
	&& echo ${TIMEZONE} > /etc/timezone \
	&& printf '[PHP]\ndate.timezone = "%s"\n', ${TIMEZONE} \
			> /usr/local/etc/php/conf.d/tzone.ini \
	&& "date"

COPY ./conf/php.ini /usr/local/etc/php/

# PHP Basics
RUN set -x \
	&& apk upgrade --update \
 	&& apk add --no-cache \
 			autoconf \
 			coreutils \
			freetype-dev \
			libjpeg-turbo-dev \
			libmcrypt-dev \
			libpng-dev \
	&& docker-php-ext-install -j$(getconf _NPROCESSORS_ONLN) \
			mcrypt \
			zip \
	&& docker-php-ext-configure gd \
			--with-freetype-dir=/usr/include/ \
			--with-jpeg-dir=/usr/include/ \
	&& docker-php-ext-install -j$(getconf _NPROCESSORS_ONLN) gd \
	&& rm -rf /tmp/*

# set recommended PHP.ini settings
# see https://secure.php.net/manual/en/opcache.installation.php
COPY ./conf/opcache.ini /usr/local/etc/php/conf.d/opcache.ini

# Imagemagick
# @link https://github.com/docker-library/php/issues/105#issuecomment-278114879
RUN set -ex \
	&& apk add --no-cache --virtual .phpize-deps $PHPIZE_DEPS \
	&& apk add --no-cache imagemagick-dev \
			libtool \
	&& export CFLAGS="$PHP_CFLAGS" CPPFLAGS="$PHP_CPPFLAGS" LDFLAGS="$PHP_LDFLAGS" \
	&& pecl install imagick-3.4.3 \
	&& docker-php-ext-enable imagick \
	&& apk add --no-cache --virtual .imagick-runtime-deps imagemagick \
	&& apk del .phpize-deps \
	&& rm -rf /tmp/*

# PHP Composer
RUN curl -sS https://getcomposer.org/installer \
			| php -- --install-dir=/usr/local/bin --filename=composer

# XDebug – depends on the ${MODE} var
RUN if [[ -n "${MODE+dev}" ]]; then \
			echo " ~~~> Installing XDebug <~~~" \
			&& apk add --no-cache --virtual .phpize-deps $PHPIZE_DEPS \
						g++ \
						gcc \
			&& pecl install xdebug \
			&& docker-php-ext-enable xdebug \
			&& apk del .phpize-deps; \
		fi
# XDebug ini
COPY ./conf/xdebug.ini /usr/local/etc/php/conf.d/xdebug.ini

# Git

WORKDIR /usr/src/api

# ENTRYPOINT ["docker-entrypoint.sh"]

# CMD ["php-fpm"]