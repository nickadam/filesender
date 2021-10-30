FROM php:7.4-apache-buster AS downloads

ADD https://github.com/filesender/filesender/archive/refs/tags/master-filesender-2.30.tar.gz /

ADD https://github.com/simplesamlphp/simplesamlphp/releases/download/v1.19.3/simplesamlphp-1.19.3.tar.gz /

ADD https://github.com/krallin/tini/releases/download/v0.19.0/tini /

WORKDIR /

RUN tar xzvf /master-filesender-2.30.tar.gz

RUN tar xzvf /simplesamlphp-1.19.3.tar.gz

# move stuff around and chown new folders for filesender
RUN mkdir /opt/filesender && \
  mv filesender-master-filesender-2.30 /opt/filesender/filesender && \
  mv simplesamlphp-1.19.3 /opt/filesender/simplesaml && \
  cp /opt/filesender/filesender/config/config_sample.php /opt/filesender/filesender/config/config.php && \
  mkdir /opt/filesender/filesender/tmp /opt/filesender/filesender/files /opt/filesender/filesender/log && \
  chown www-data:www-data /opt/filesender/filesender/tmp /opt/filesender/filesender/files /opt/filesender/filesender/log && \
  cp -r /opt/filesender/simplesaml/config-templates/*.php /opt/filesender/simplesaml/config/

# generate snakeoil cert and key
RUN apt-get update && \
  apt-get install -y ssl-cert dialog apt-utils && \
  make-ssl-cert generate-default-snakeoil

FROM php:7.4-apache-buster

# install mta
RUN apt-get update && \
    apt-get dist-upgrade -y && \
    apt-get install -y --no-install-recommends msmtp-mta && \
    apt-get autoremove -y && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

COPY --from=downloads /opt/filesender /opt/filesender

COPY --from=downloads --chmod=755 /tini /usr/local/bin/

# copy entrypoint that sets environment variables in apache
COPY --chmod=755 /docker-entrypoint /

# copy snakeoil cert and key
COPY --from=downloads /etc/ssl/certs/ssl-cert-snakeoil.pem /etc/ssl/certs/ssl-cert-snakeoil.pem
COPY --from=downloads /etc/ssl/private/ssl-cert-snakeoil.key /etc/ssl/private/ssl-cert-snakeoil.key

# copy config and authsources from project that uses environment variables
COPY --chmod=644 config.php /opt/filesender/filesender/config/
COPY --chmod=644 authsources.php /opt/filesender/filesender/config/

# enable filesender
RUN a2enmod alias headers ssl && \
  a2ensite default-ssl && \
  ln -s /opt/filesender/filesender/config-templates/apache/filesender.conf /etc/apache2/sites-enabled/

ENTRYPOINT ["tini", "/docker-entrypoint"]
