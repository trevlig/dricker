FROM ubuntu:14.04

ENV DEBIAN_FRONTEND noninteractive

# set the locale
RUN locale-gen en_US.UTF-8
ENV LANG en_US.UTF-8
ENV LANGUAGE en_US:en
ENV LC_ALL en_US.UTF-8

# apt-get
RUN apt-get update &&   \
    apt-get -y install  \
        git             \
        curl            \
        openssh-server  \
        supervisor      \
        pwgen           \
        php5            \
        php-apc         \
        apache2         \
        libapache2-mod-php5 \
        php5-cli        \
        php5-curl       \
        php5-mysql      \
        php5-gd         \
        php5-mcrypt     \
        php5-json       \
        sqlite3         \
        libsqlite3-dev  \
        php5-sqlite     \
        && \
    apt-get -y clean

# php
COPY config/php.ini /etc/php5/apache2/php.ini
COPY config/php.ini /etc/php5/cli/php.ini

# composer
RUN curl -sS https://getcomposer.org/installer | php && \
    mv composer.phar /usr/local/bin/composer

# drush 7
RUN composer global require drush/drush:7.* && \
    composer global update && \
    ln -s /root/.composer/vendor/bin/drush /usr/local/bin/drush

# drupal
RUN drush dl drupal-7.41                                && \
    mv drupal-7.41 /var/www/drupal                      && \
    mkdir -p /var/www/drupal/sites/default/files        && \
    mkdir -p /var/www/drupal/sites/all/modules/contrib  && \
    mkdir -p /var/www/drupal/sites/all/modules/custom   && \
    cd /var/www/drupal                                  && \
    drush si -y minimal                                 \
    --db-url=sqlite:sites/default/files/.ht.sqlite      \
    --account-name=admin --account-pass=admin           \
    --site-name="Drupal"                                && \
    chown -R www-data:www-data /var/www                 && \
    chmod -R u+w /var/www/drupal/sites/default


# override default apache conf
COPY config/apache.conf /etc/apache2/sites-enabled/000-default.conf

# enable apache rewrite module
RUN a2enmod rewrite

# ssh
RUN mkdir /var/run/sshd && \
    echo 'root:root' | chpasswd && \
    sed -i 's/PermitRootLogin without-password/PermitRootLogin yes/' /etc/ssh/sshd_config

# ssh login fix. otherwise user is kicked off after login
RUN sed 's@session\s*required\s*pam_loginuid.so@session optional pam_loginuid.so@g' -i /etc/pam.d/sshd

# supervisor
COPY config/supervisord.conf /etc/supervisor/conf.d/supervisord.conf

EXPOSE 22 80

CMD ["/usr/bin/supervisord"]
