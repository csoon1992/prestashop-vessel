FROM ubuntu:20.04

ARG psuid

RUN useradd -ms /bin/bash -u $psuid prestashop
WORKDIR /var/www/html

ENV GOSU_VERSION 1.7

# Avoid prompts when building
ENV DEBIAN_FRONTEND noninteractive
ENV TZ=UTC
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

RUN apt-get update \
  && apt-get install -y gnupg gosu curl ca-certificates zip unzip git supervisor sqlite3 \
  && mkdir -p ~/.gnupg \
  && echo "disable-ipv6" >> ~/.gnupg/dirmngr.conf \
  && apt-key adv --homedir ~/.gnupg --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys E5267A6C \
  && apt-key adv --homedir ~/.gnupg --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys C300EE8C \
  && echo "deb http://ppa.launchpad.net/ondrej/php/ubuntu focal main" > /etc/apt/sources.list.d/ppa_ondrej_php.list \
  && apt-get update \
  && apt-get install -y nginx php7.4-fpm php7.4-cli \
  php7.4-pgsql php7.4-sqlite3 php7.4-gd \
  php7.4-curl php7.4-memcached \
  php7.4-imap php7.4-mysql php7.4-mbstring \
  php7.4-xml php7.4-zip php7.4-bcmath php7.4-soap \
  php7.4-intl php7.4-readline php7.4-xdebug php7.4-pcov \
  php7.4-msgpack php7.4-igbinary php7.4-ldap \
  php-redis \
  && php -r "readfile('http://getcomposer.org/installer');" | php -- --install-dir=/usr/bin/ --filename=composer \
  && apt-get -y autoremove \
  && apt-get clean \
  && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* \
  && sed -i "s/pm\.max_children = .*/pm.max_children = 20/" /etc/php/7.4/fpm/pool.d/www.conf \
  && sed -i "s/pm\.start_servers = .*/pm.start_servers = 10/" /etc/php/7.4/fpm/pool.d/www.conf \
  && sed -i "s/pm\.min_spare_servers = .*/pm.min_spare_servers = 5/" /etc/php/7.4/fpm/pool.d/www.conf \
  && sed -i "s/pm\.max_spare_servers = .*/pm.max_spare_servers = 10/" /etc/php/7.4/fpm/pool.d/www.conf \
  && echo "daemon off;" >> /etc/nginx/nginx.conf \
  && ln -sf /dev/stdout /var/log/nginx/access.log \
  && ln -sf /dev/stderr /var/log/nginx/error.log \
  && sed -i 's/^;daemonize.*$/daemonize = no/g' /etc/php/7.4/fpm/php-fpm.conf \
  && sed -i 's@^error_log.*$@error_log = /proc/self/fd/2@g' /etc/php/7.4/fpm/php-fpm.conf \
  && echo "\n; Allow Vessel to set env vars for local dev\nclear_env=false" >> /etc/php/7.4/fpm/php-fpm.conf

RUN apt-get update && apt-get install -y mysql-client && rm -rf /var/lib/apt

RUN ln -sf /dev/stdout /var/log/nginx/access.log \
  && ln -sf /dev/stderr /var/log/nginx/error.log

COPY h5bp /etc/nginx/h5bp
COPY prestashopnginx.conf /etc/nginx/
COPY default /etc/nginx/sites-available/default
COPY php-fpm.conf /etc/php/7.4/fpm/php-fpm.conf
COPY xdebug.ini /etc/php/7.4/mods-available/xdebug.ini
COPY crontab /etc/cron.d/prestashop

EXPOSE 80
EXPOSE 8080

COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf
COPY start-container /usr/local/bin/start-container
RUN chmod +x /usr/local/bin/start-container

# Give execution rights on the cron job
RUN chmod a+x /etc/cron.d/prestashop

ENTRYPOINT ["start-container"]
