FROM debian:wheezy
MAINTAINER italoag <italoamorim@gmail.com>

RUN apt-get update \
    && apt-get upgrade -y \
    && apt-get clean \
    && DEBIAN_FRONTEND=noninteractive apt-get install -y \
      apache2 \
      php5 \
      libapache2-mod-php5 \
      php5-cli \
      php5-gd \
      php5-curl \
      php5-intl \ 
      php-pear \
      php5-imagick\
      php5-imap\
      php5-mcrypt\
      php5-memcache\
      php5-ming\
      php5-ps\
      php5-pspell\
      php5-recode\
      php5-sqlite\
      php5-tidy\
      php5-xmlrpc\
      php5-xsl \
      php5-mysql \
      mysql-server \
      mysql-client \
      curl \
      supervisor \
      ca-certificates \
      vim \
    && rm -rf /var/lib/apt/lists/*

#
# Apache Settings
#
RUN mv /etc/apache2/sites-available/default /etc/apache2/sites-available/default.dist
ADD apache2-default /etc/apache2/sites-available/default
RUN a2enmod rewrite

#
# Install WP-CLI
#
RUN curl -O https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar \
    && chmod +x wp-cli.phar \
    && mv wp-cli.phar /usr/local/bin/wp

#
# Install WordPress
#
RUN mkdir /var/www/wordpress
ADD wp-config-extra /wp-config-extra
WORKDIR /var/www/wordpress
RUN sed -i -e "s/^bind-address.*/bind-address = 0.0.0.0/" /etc/mysql/my.cnf \
    && service mysql start \
    && mysqladmin -u root password root \
    && mysql -uroot -proot -e \
      "CREATE DATABASE wordpress DEFAULT CHARACTER SET utf8; grant all privileges on wordpress.* to wordpress@'%' identified by 'wordpress';" \
    && wp core download --allow-root \
    && wp core config --allow-root \
      --dbname=wordpress \
      --dbuser=wordpress \
      --dbpass=wordpress \
      --dbhost=localhost \
      --extra-php < /wp-config-extra \
    && wp core install --allow-root \
      --admin_name=admin \
      --admin_password=admin \
      --admin_email=admin@example.com \
      --url=http://wp.dev \
      --title=WordPress \
    && wp theme update --allow-root --all \
    && wp plugin update --allow-root --all \
    && rm -rf /wp-config-extra
RUN chown -R www-data:www-data /var/www/wordpress

#
# Open ports
#
EXPOSE 80 3306

#
# Supervisor
#
RUN mkdir -p /var/log/supervisor
ADD supervisord.conf /etc/supervisor/conf.d/supervisord.conf

CMD ["/usr/bin/supervisord"]
