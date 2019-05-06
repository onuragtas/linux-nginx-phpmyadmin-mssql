FROM debian:latest


WORKDIR /var/www/html
RUN echo "alias ll='ls -la --color'" | tee --append /etc/bash.bashrc
ENV DEBIAN_FRONTEND noninteractive

RUN apt-get -y update && apt-get install -y apt-utils libc6 debconf subversion

RUN apt-get install -y sudo man git htop vim mc \
    software-properties-common \
    apt-transport-https lsb-release wget lynx telnet curl \
    parallel bzip2 acl gnupg

RUN adduser --gecos '' --uid 1000 --gid 50 --disabled-password php \
    && adduser php staff

RUN curl https://packages.microsoft.com/keys/microsoft.asc | apt-key add - \
    && curl https://packages.microsoft.com/config/debian/9/prod.list > /etc/apt/sources.list.d/mssql-release.list \
    && apt-get update \
    && ACCEPT_EULA=Y apt-get install -y msodbcsql17 \
    && sudo ACCEPT_EULA=Y apt-get install -y mssql-tools \
    && echo 'export PATH="$PATH:/opt/mssql-tools/bin"' >> ~/.bash_profile \
    && echo 'export PATH="$PATH:/opt/mssql-tools/bin"' >> ~/.bashrc \
    #&& source ~/.bashrc \
    && sudo apt-get install -y unixodbc-dev unixodbc libssl-dev

RUN apt-get install -y php7.0-fpm php7.0-cli php7.0-mbstring  php7.0-intl php7.0-zip php7.0-mysql php7.0-redis php7.0-memcache php7.0-gd php7.0-curl php7.0-xsl php7.0-mongodb php7.0-xdebug php7.0-dev php-pear php7.0-pgsql
RUN sed -i "s/^;\(date\.timezone\(\s*\)\?=\).*/\1 Europe\/Amsterdam/" /etc/php/7.0/cli/php.ini
RUN sed -i "s/^;\(date\.timezone\(\s*\)\?=\).*/\1 Europe\/Amsterdam/" /etc/php/7.0/fpm/php.ini
RUN sed -i "s/^\(memory_limit\(\s*\)\?=\).*/\1 4G/" /etc/php/7.0/cli/php.ini
RUN sed -i "s/^\(memory_limit\(\s*\)\?=\).*/\1 256M/" /etc/php/7.0/fpm/php.ini
RUN sed -i "s/^\(short_open_tag\(\s*\)\?=\).*/\1 On/" /etc/php/7.0/fpm/php.ini
RUN sed -i "s/^\(session\.save_handler\(\s*\)\?=\).*/\1 redis/" /etc/php/7.0/fpm/php.ini
RUN echo "session.save_path= tcp://172.28.1.4:6379" >> /etc/php/7.0/fpm/php.ini

RUN pecl install sqlsrv-5.2.0 pdo_sqlsrv-5.2.0 swoole
RUN echo "xdebug.remote_enable=1" >> /etc/php/7.0/fpm/php.ini
RUN echo "zend_extension=xdebug.so\nxdebug.default_enable=1\nxdebug.remote_enable=1\nxdebug.remote_handler=dbgp\nxdebug.remote_port=9001\nxdebug.remote_autostart=1\nxdebug.remote_connect_back=1\nxdebug.idekey='PHPSTORM'" >> /etc/php/7.0/mods-available/xdebug.ini
RUN echo -e "; priority=20\nextension=sqlsrv.so" > /etc/php/7.0/mods-available/sqlsrv.ini
RUN echo -e "; priority=20\nextension=pdo_sqlsrv.so" > /etc/php/7.0/mods-available/pdo_sqlsrv.ini
RUN echo -e "; priority=20\nextension=swoole.so" > /etc/php/7.0/mods-available/swoole.ini
RUN phpenmod -v 7.0 -s ALL mbstring intl pdo_mysql redis memcache gd curl xml xsl mongodb xdebug sqlsrv pdo_sqlsrv swoole zip



RUN apt-get -y install supervisor nginx nano
RUN apt-get -y update && apt-get -y dist-upgrade

RUN apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

RUN rm -rf /var/www/html
ADD . /var/www/html

RUN chown -R php.staff /var/www/html

EXPOSE 9000 80

# RUN ln -sf /dev/stdout /var/log/nginx/access.log \
#     && ln -sf /dev/stderr /var/log/nginx/error.log

CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/supervisord.conf"]