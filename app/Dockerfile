FROM debian:12

ARG PHP_VERSION
ARG GESTSUP_VERSION

ENV PHP_VERSION=${PHP_VERSION}
ENV GESTSUP_VERSION=${GESTSUP_VERSION}

RUN apt-get update && apt-get upgrade -y && apt-get dist-upgrade -y
RUN apt-get install curl mariadb-client wget apache2 php${PHP_VERSION} php${PHP_VERSION}-mysql php${PHP_VERSION}-xml php${PHP_VERSION}-curl php${PHP_VERSION}-imap php${PHP_VERSION}-ldap php${PHP_VERSION}-zip php${PHP_VERSION}-mbstring php${PHP_VERSION}-gd unzip ntp php-intl -y
RUN echo "ServerName localhost" >> /etc/apache2/apache2.conf
COPY php.ini /etc/php/${PHP_VERSION}/apache2/php.ini
RUN mkdir -p /var/www/html

RUN set -eu; \
    ZIP="gestsup_${GESTSUP_VERSION}.zip"; \
    echo "Downloading GestSup version ${GESTSUP_VERSION}..."; \
    wget -q -O "/tmp/${ZIP}" "https://gestsup.fr/downloads/versions/current/version/${ZIP}"; \
    echo "Unzipping ${ZIP}..."; \
    unzip -qo "/tmp/${ZIP}" -d /var/www/html; \
    echo "Cleaning up default files..."; \
    rm -f "/tmp/${ZIP}" /var/www/html/index.html

RUN set -eu; \
    chown -R www-data:www-data /var/www/html; \
    find /var/www/html -type d -exec chmod 750 {} +; \
    find /var/www/html -type f -exec chmod 640 {} +; \
    chmod -R 770 /var/www/html/upload /var/www/html/images/model /var/www/html/backup /var/www/html/_SQL; \
    chown -R www-data:www-data /var/lib/php/sessions; \
    chmod 660 /var/www/html/connect.php

RUN set -eu; \
    SEC_CONF="/etc/apache2/conf-enabled/security.conf"; \
    if grep -qE '^ServerTokens\s+' "${SEC_CONF}"; then \
        sed -i 's/^ServerTokens.*/ServerTokens Prod/' "${SEC_CONF}"; \
    else \
        echo 'ServerTokens Prod' >> "${SEC_CONF}"; \
    fi; \
    if grep -qE '^ServerSignature\s+' "${SEC_CONF}"; then \
        sed -i 's/^ServerSignature.*/ServerSignature Off/' "${SEC_CONF}"; \
    else \
        echo 'ServerSignature Off' >> "${SEC_CONF}"; \
    fi

COPY entrypoint.sh /usr/local/bin/entrypoint.sh
RUN chmod +x /usr/local/bin/entrypoint.sh

ENTRYPOINT [ "entrypoint.sh" ]
