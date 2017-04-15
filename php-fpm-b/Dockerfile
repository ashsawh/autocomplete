FROM php:7.1.3-fpm

ENV ZEROMQ_VERSION 4.1.4
ENV COMPOSER_HOME /root/composer
ENV COMPOSER_VERSION master
ENV PHPREDIS_VERSION php7.1

WORKDIR /app

RUN apt-get update \
    && apt-get install -y \
      # for intl extension
      libicu-dev \
      # for mongodb
      libssl-dev \
      # for rabbit mq
      librabbitmq-dev \
      # for zero mq
      libsodium-dev \
      # for postgres
      libpq-dev \
      # for soap
      libxml2 \
      libxml2-dev \
      git \
      zip \
    && rm -rf /var/lib/apt/lists/* \
    && pecl install mongodb \
    && pecl install amqp-1.7.0alpha2 \
    && pecl install xdebug-2.5.0 redis-3.1.0 \
    && docker-php-ext-configure bcmath --enable-bcmath \
    && docker-php-ext-configure intl --enable-intl \
    && docker-php-ext-configure pcntl --enable-pcntl \
    && docker-php-ext-configure pdo_mysql --with-pdo-mysql \
    && docker-php-ext-configure pdo_pgsql --with-pgsql \
    && docker-php-ext-configure mbstring --enable-mbstring \
    && docker-php-ext-configure soap --enable-soap \
    && docker-php-ext-enable xdebug \
    && docker-php-ext-install \
        bcmath \
        intl \
        pcntl \
        pdo_mysql \
        pdo_pgsql \
        mbstring \
        soap \
    && curl -L http://download.zeromq.org/zeromq-${ZEROMQ_VERSION}.tar.gz | tar zx \
    && cd zeromq-${ZEROMQ_VERSION} \
    && ./configure \
    && make \
    && make install \
    && ldconfig \
    && pecl install zmq-1.1.3 \
    && rm -rf /app \
    && mkdir /app

# Copy configuration
COPY ./config/php-cli.ini /usr/local/etc/php/php.ini
COPY ./config/php7.ini /usr/local/etc/php/conf.d/
COPY ./config/amqp.ini /usr/local/etc/php/conf.d/
COPY ./config/zmq.ini /usr/local/etc/php/conf.d/
COPY ./config/mongodb.ini /usr/local/etc/php/conf.d/
COPY ./config/xdebug_fpm.ini /usr/local/etc/php/conf.d/

# Set up the application directory
VOLUME ["/app"]


# blackfire
RUN export VERSION=`php -r "echo PHP_MAJOR_VERSION.PHP_MINOR_VERSION;"` \
    && curl -A "Docker" -o /tmp/blackfire-probe.tar.gz -D - -L -s https://blackfire.io/api/v1/releases/probe/php/linux/amd64/${VERSION} \
    && tar zxpf /tmp/blackfire-probe.tar.gz -C /tmp \
    && mv /tmp/blackfire-*.so `php -r "echo ini_get('extension_dir');"`/blackfire.so \
    && printf "extension=blackfire.so\nblackfire.agent_socket=\${BLACKFIRE_PORT}" > $PHP_INI_DIR/conf.d/blackfire.ini \
    && rm -rf /tmp/* \

# composer
RUN docker-php-ext-install \
        zip \
    && curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer \
    && composer global require hirak/prestissimo

# Install Node
# gpg keys listed at https://github.com/nodejs/node
RUN set -ex \
  && for key in \
    9554F04D7259F04124DE6B476D5A82AC7E37093B \
    94AE36675C464D64BAFA68DD7434390BDBE9B9C5 \
    0034A06D9D9B0064CE8ADF6BF1747F4AD2306D93 \
    FD3A5288F042B6850C66B31F09FE44734EB7990E \
    71DCFD284A79C3B38668286BC97EC7A07EDE3FC1 \
    DD8F2338BAE7501E3DD5AC78C273792F7D83545D \
    B9AE9905FFD7803F25714661B63B535A4C206CA9 \
    C4F0DFFF4E8C1A8236409D08E73BC641CC11F4C8 \
  ; do \
    gpg --keyserver ha.pool.sks-keyservers.net --recv-keys "$key"; \
  done

ENV NPM_CONFIG_LOGLEVEL info
ENV NODE_VERSION 7.4.0

RUN curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/node-v$NODE_VERSION-linux-x64.tar.xz" \
  && curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/SHASUMS256.txt.asc" \
  && gpg --batch --decrypt --output SHASUMS256.txt SHASUMS256.txt.asc \
  && grep " node-v$NODE_VERSION-linux-x64.tar.xz\$" SHASUMS256.txt | sha256sum -c - \
  && tar -xJf "node-v$NODE_VERSION-linux-x64.tar.xz" -C /usr/local --strip-components=1 \
  && rm "node-v$NODE_VERSION-linux-x64.tar.xz" SHASUMS256.txt.asc SHASUMS256.txt \
  && ln -s /usr/local/bin/node /usr/local/bin/nodejs

# Install bower and gulp
RUN npm install -g bower grunt-cli gulp

CMD ["php-fpm"]