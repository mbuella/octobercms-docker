FROM php:7.1-apache
LABEL maintainer="Marlon Buella <marlon.b.buella@gmail.com>"

ARG APACHE_RUN_USER=web-user
ARG APACHE_RUN_GROUP=${APACHE_RUN_USER}
ARG APACHE_UID=0
ARG APACHE_GID=${APACHE_GID}

#PRE-PHP-INSTALL
RUN apt-get update \
  && apt-get install -y \  
    libfreetype6-dev \
    libicu-dev \
    libjpeg62-turbo-dev \
    libmcrypt-dev \
    libpng-dev \
    libxslt1-dev

RUN docker-php-ext-configure \
  gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/

#INSTALL PHP EXTENSIONS
RUN docker-php-ext-install \
  gd \
  intl \
  pdo_mysql \
  xsl \
  zip \
  soap \
  bcmath \
  mcrypt

#INSTALL COMPOSER
RUN   apt-get update \
    && apt-get install -y git unzip \
    && curl -sS https://getcomposer.org/installer -o composer-setup.php \
#    & php -r "if (hash_file('SHA384', 'composer-setup.php') === '93b54496392c062774670ac18b134c3b3a95e5a5e5c8f1a9f115f203b75bf9a129d5daa8ba6a13e2cc8a1da0806388a8') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;" \
    && php composer-setup.php --install-dir=/usr/local/bin --filename=composer

# enable apache modules
RUN a2enmod rewrite

WORKDIR /var/www/html

# user management

RUN if id "$APACHE_RUN_USER" >/dev/null 2>&1 ; then \
	echo "*** THE USER INSIDE THE CONTAINER WILL BE USED ***" ; \
	if [ $(id -u "$APACHE_RUN_USER") != "$APACHE_UID" ]; then \
		echo "*** SETTING THE UID FOR THE USER ***" ; \
		usermod -u "$APACHE_UID" "$APACHE_RUN_USER" ; \
	fi ; \
else \
	echo "CREATING NEW USER INSIDE THE CONTAINER" ; \
	useradd --non-unique -u "$APACHE_UID" "$APACHE_RUN_USER" ; \
fi

RUN if [ $(getent group "$APACHE_RUN_GROUP") ] ; then \
	echo "*** THE GROUP INSIDE THE CONTAINER WILL BE USED ***" ; \
	if [ $(getent group "$APACHE_RUN_GROUP" | awk -F: '{printf "%d\n", $3}') != "$APACHE_GID" ] ; then \
		echo "*** SETTING THE GID FOR THE GROUP ***" ; \		
		groupmod -g "$APACHE_GID" "$APACHE_RUN_GROUP" ; \
	fi ; \
	if [ $(id -Gn "$APACHE_RUN_USER"|grep -c "$APACHE_RUN_GROUP") != 1 ] ; then \
		usermod -a -G "$APACHE_RUN_GROUP" "$APACHE_RUN_USER" ; \
	fi ; \
else \
	echo "CREATING NEW GROUP INSIDE THE CONTAINER" ; \
	groupadd --non-unique -g "$APACHE_GID" "$APACHE_RUN_GROUP" ; \
	usermod -a -G "$APACHE_RUN_GROUP" "$APACHE_RUN_USER" ; \
fi

#RUN usermod -u "$APACHE_UID" "$APACHE_RUN_USER"
#RUN groupmod -g "$APACHE_GID" "$APACHE_RUN_GROUP"
RUN chown "$APACHE_RUN_USER" /var/run/apache2
#RUN chown "$APACHE_RUN_USER" /var/www/html

# copy files from host to image
COPY ./ /var/www/html
RUN find /var/www/html -type d -exec chown "$APACHE_RUN_USER" {} \;
RUN find /var/www/html -type f -exec chown "$APACHE_RUN_USER" {} \;
COPY ./docker/app/scripts/entrypoint /usr/local/bin
RUN chmod +x /usr/local/bin/entrypoint
RUN mv .env.october .env

# Set port
ADD ./docker/app/config/apache/000-default.conf /etc/apache2/sites-available                 
RUN ln -sf /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-enabled/000-default.conf
ADD ./docker/app/config/apache/ports.conf /etc/apache2

# Change to the app user
USER "$APACHE_RUN_USER"

ENTRYPOINT ["entrypoint"]

CMD ["apache2-foreground"]
