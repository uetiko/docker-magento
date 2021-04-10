FROM node:14.15.4-slim AS node
FROM php:7.1-fpm

COPY --from=node /usr/local/bin /usr/local/bin
COPY --from=node /usr/local/lib /usr/local/lib
COPY --from=node /opt /opt
COPY --from=composer:1.10.17 /usr/bin/composer /usr/bin/composer

RUN apt-get update && apt-get install -y \
  libc-bin libc6 \
  apt-utils \
  cron \
  git \
  apt-utils \
  libfreetype6-dev \
  libicu-dev \
  libjpeg62-turbo-dev \
  libmcrypt-dev \
  libpng-dev \
  libxslt1-dev \
  default-mysql-client \
  vim \
  zip zsh wget libxml2-dev libxslt-dev

RUN docker-php-ext-configure \
  gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/

RUN docker-php-ext-install \
  bcmath \
  gd \
  intl \
  mbstring \
  mcrypt \
  opcache \
  pdo_mysql \
  soap \
  xsl \
  simplexml \
  zip

RUN pecl channel-update pecl.php.net \
  && pecl install xdebug-2.6.0 \
  && docker-php-ext-enable xdebug

RUN printf '* *\t* * *\tapp\t%s/usr/local/bin/php /var/www/html/update/cron.php\n' >> /etc/crontab
RUN printf '* *\t* * *\tapp\t%s/usr/local/bin/php /var/www/html/bin/magento cron:run\n' >> /etc/crontab
RUN printf '* *\t* * *\tapp\t%s/usr/local/bin/php /var/www/html/bin/magento setup:cron:run\n#\n' >> /etc/crontab

COPY conf/php/php.ini /usr/local/etc/php/
COPY bin2/cronstart /usr/local/bin/

RUN wget https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh
RUN sh ./install.sh
RUN rm ./install.sh


RUN usermod -u 1000 www-data
RUN chsh -s /bin/zsh www-data

RUN mkdir /sock
RUN chown -R www-data:www-data /usr/local/etc/php/conf.d /sock

VOLUME /var/www

WORKDIR /var/www/html

EXPOSE 9001
ENV XDEBUG_PORT 9003
