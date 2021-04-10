FROM node:14.15.4-slim AS node
FROM php:7.3-fpm

COPY --from=node /usr/local/bin /usr/local/bin
COPY --from=node /usr/local/lib /usr/local/lib
COPY --from=node /opt /opt
COPY --from=composer /usr/bin/composer /usr/bin/composer

RUN apt-get update && apt-get install -y \
  apt-utils cron git vim zsh wget \
  libc-bin libc6 \
  libfreetype6-dev \
  libicu-dev \
  libjpeg62-turbo-dev \
  libpng-dev \
  libxslt1-dev \
  default-mysql-client \
  libxml2-dev libxslt-dev \
  zip libc-client-dev libzip-dev

RUN docker-php-ext-configure \
  gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/

RUN docker-php-ext-configure \
    zip --with-libzip

RUN docker-php-ext-install -j$(nproc) \
  bcmath \
  gd \
  intl \
  mbstring \
  opcache \
  pdo_mysql \
  soap \
  xsl \
  simplexml 

RUN rm -f /usr/local/etc/php/conf.d/*sodium.ini 
RUN rm -f /usr/local/lib/php/extensions/*/*sodium.so 
RUN apt-get remove libsodium* -y  
RUN mkdir -p /tmp/libsodium  
RUN curl -sL https://github.com/jedisct1/libsodium/archive/1.0.18-RELEASE.tar.gz | tar xzf - -C  /tmp/libsodium 
RUN cd /tmp/libsodium/libsodium-1.0.18-RELEASE/ && ./configure \
&& make && make check && make install  
RUN cd / &&  rm -rf /tmp/libsodium &&  pecl install -o -f libsodium
RUN cd /tmp 

RUN docker-php-ext-enable sodium

RUN apt autoremove
RUN apt-get clean

RUN printf '* *\t* * *\tapp\t%s/usr/local/bin/php /var/www/html/update/cron.php\n' >> /etc/crontab
RUN printf '* *\t* * *\tapp\t%s/usr/local/bin/php /var/www/html/bin/magento cron:run\n' >> /etc/crontab
RUN printf '* *\t* * *\tapp\t%s/usr/local/bin/php /var/www/html/bin/magento setup:cron:run\n#\n' >> /etc/crontab

COPY conf/php/php.ini /usr/local/etc/php/

RUN wget https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh
RUN sh ./install.sh
RUN rm ./install.slh


RUN usermod -u 1000 www-data
RUN chsh -s /bin/zsh www-data

VOLUME /var/www

WORKDIR /var/www/html
