FROM openjdk:8u111-jre-alpine

EXPOSE 10389

RUN apk add --no-cache openssl tini
RUN mkdir /ldap
WORKDIR /ldap
ADD https://github.com/kwart/ldap-server/releases/download/2016-10-04/ldap-server.jar /ldap/

COPY --chmod=755 docker-entrypoint /

ENTRYPOINT ["tini", "/docker-entrypoint"]
