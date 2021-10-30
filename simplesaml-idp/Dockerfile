FROM php:7.4-apache-buster AS build

RUN apt-get update && \
  apt-get install -y libldap2-dev

RUN docker-php-ext-install ldap

FROM php:7.4-apache-buster AS downloads

ADD https://github.com/simplesamlphp/simplesamlphp/releases/download/v1.19.3/simplesamlphp-1.19.3.tar.gz /

ADD https://github.com/krallin/tini/releases/download/v0.19.0/tini /

WORKDIR /

RUN tar xzvf /simplesamlphp-1.19.3.tar.gz

# generate snakeoil cert and key
RUN apt-get update && \
  apt-get install -y ssl-cert dialog apt-utils && \
  make-ssl-cert generate-default-snakeoil

FROM php:7.4-apache-buster

COPY --from=downloads /simplesamlphp-1.19.3 /var/simplesamlphp

COPY --from=downloads --chmod=755 /tini /usr/sbin/

COPY --chmod=644 simplesaml.conf /etc/apache2/sites-enabled/

# copy snakeoil cert and key
COPY --from=downloads /etc/ssl/certs/ssl-cert-snakeoil.pem /etc/ssl/certs/ssl-cert-snakeoil.pem
COPY --from=downloads /etc/ssl/private/ssl-cert-snakeoil.key /etc/ssl/private/ssl-cert-snakeoil.key

# copy ldap extention
copy --from=build /usr/local/etc/php/conf.d/docker-php-ext-ldap.ini /usr/local/etc/php/conf.d/docker-php-ext-ldap.ini
copy --from=build /usr/local/etc/php/conf.d/docker-php-ext-ldap.ini /usr/local/etc/php/conf.d/docker-php-ext-ldap.ini
copy --from=build /usr/local/lib/php/extensions/no-debug-non-zts-20190902/ldap.so /usr/local/lib/php/extensions/no-debug-non-zts-20190902/ldap.so

RUN a2enmod alias ssl && \
  a2ensite default-ssl && \
  cp -r /var/simplesamlphp/config-templates/*.php /var/simplesamlphp/config/

COPY --chmod=755 docker-entrypoint /

COPY --chmod=644 config.php /var/simplesamlphp/config/
COPY --chmod=644 authsources.php /var/simplesamlphp/config/

ENTRYPOINT ["tini", "/docker-entrypoint"]
