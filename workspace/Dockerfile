ARG LARADOCK_PHP_VERSION
ARG BASE_IMAGE_TAG_PREFIX=latest

FROM laradock/workspace:${BASE_IMAGE_TAG_PREFIX}-${LARADOCK_PHP_VERSION}

LABEL maintainer="Mahmoud Zalt <mahmoud@zalt.me>"
LABEL maintainer="Santiago Marulanda <smarulanda97@outlook.com>"

ARG LARADOCK_PHP_VERSION

ENV DEBIAN_FRONTEND noninteractive

USER root

###########################################################################
# Laradock non-root user:
###########################################################################

ARG PUID=1000
ENV PUID ${PUID}
ARG PGID=1000
ENV PGID ${PGID}

RUN set -xe; \
    apt-get update -yqq && \
    pecl channel-update pecl.php.net && \
    groupadd -g ${PGID} laradock && \
    useradd -l -u ${PUID} -g laradock -m laradock -G docker_env && \
    usermod -p "*" laradock -s /bin/bash && \
    apt-get install -yqq apt-utils libzip-dev zip unzip php${LARADOCK_PHP_VERSION}-zip nasm && \
    php -m | grep -q 'zip'

###########################################################################
# Set Timezone
###########################################################################

ARG TZ=UTC
ENV TZ ${TZ}

RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

###########################################################################
# Composer:
###########################################################################

USER root

COPY ./composer.json /home/laradock/.composer/composer.json

RUN chown -R laradock:laradock /home/laradock/.composer

RUN echo "" >> ~/.bashrc && echo 'export PATH="$HOME/.composer/vendor/bin:$PATH"' >> ~/.bashrc

# Update composer
ARG COMPOSER_VERSION=2
ENV COMPOSER_VERSION ${COMPOSER_VERSION}

RUN set -eux; \
    if [ "$COMPOSER_VERSION" = "1" ] || [ "$COMPOSER_VERSION" = "2" ]; then \
        composer self-update --${COMPOSER_VERSION}; \
    else \
        composer self-update ${COMPOSER_VERSION}; \
    fi

USER laradock

ARG COMPOSER_REPO_PACKAGIST
ENV COMPOSER_REPO_PACKAGIST ${COMPOSER_REPO_PACKAGIST}

ARG COMPOSER_GLOBAL_INSTALL=false
ENV COMPOSER_GLOBAL_INSTALL ${COMPOSER_GLOBAL_INSTALL}

RUN if [ ${COMPOSER_GLOBAL_INSTALL} = true ]; then \
    composer global install; \
fi

RUN if [ ${COMPOSER_REPO_PACKAGIST} ]; then \
    composer config -g repo.packagist composer ${COMPOSER_REPO_PACKAGIST}; \
fi

RUN echo "" >> ~/.bashrc && echo 'export PATH="~/.composer/vendor/bin:$PATH"' >> ~/.bashrc

###########################################################################
# Non-root user : PHPUnit path
###########################################################################

USER laradock

RUN echo "" >> ~/.bashrc && echo 'export PATH="/var/www/vendor/bin:$PATH"' >> ~/.bashrc

###########################################################################
# Crontab
###########################################################################

USER root

COPY ./crontab /etc/cron.d

RUN chmod -R 644 /etc/cron.d

###########################################################################
# Certificate Authorities
###########################################################################

USER root

COPY ./ca-certificates/* /usr/local/share/ca-certificates/

RUN update-ca-certificates

###########################################################################
# PHP extensions
###########################################################################

USER root

ARG INSTALL_BZ2=false
ARG INSTALL_GMP=false
ARG INSTALL_GNUPG=false
ARG INSTALL_SSH2=false
ARG INSTALL_SOAP=false
ARG INSTALL_XSL=false
ARG INSTALL_PHPDBG=false
ARG INSTALL_PHPREDIS=false
ARG INSTALL_MEMCACHED=false
ARG INSTALL_XMLRPC=false
ARG INSTALL_PHPDECIMAL=false
ARG PHP_VERSION=${LARADOCK_PHP_VERSION}

RUN set -eux; \
    if [ ${INSTALL_BZ2} = true ]; then \
        apt-get -yqq install php${LARADOCK_PHP_VERSION}-bz2; \
    fi; \
    if [ ${INSTALL_GMP} = true ]; then \
        apt-get -yqq install php${LARADOCK_PHP_VERSION}-gmp; \
    fi; \
    if [ ${INSTALL_GNUPG} = true ]; then \
        apt-get -yqq install php${LARADOCK_PHP_VERSION}-gnupg; \
    fi; \
    if [ ${INSTALL_SSH2} = true ]; then \
        apt-get -yqq install libssh2-1-dev php${LARADOCK_PHP_VERSION}-ssh2; \
    fi; \
    if [ ${INSTALL_SOAP} = true ]; then \
        apt-get -yqq install libxml2-dev php${LARADOCK_PHP_VERSION}-soap; \
    fi; \
    if [ ${INSTALL_XSL} = true ]; then \
        apt-get -yqq install libxslt-dev php${LARADOCK_PHP_VERSION}-xsl; \
    fi; \
    if [ ${INSTALL_PHPDBG} = true ]; then \
        apt-get install -y --force-yes php${LARADOCK_PHP_VERSION}-phpdbg; \
    fi; \
    if [ ${INSTALL_PHPREDIS} = true ]; then \
        apt-get update && apt-get install -yqq php${LARADOCK_PHP_VERSION}-redis; \
    fi; \
    if [ ${INSTALL_MEMCACHED} = true ]; then \
        apt-get -y install php${LARADOCK_PHP_VERSION}-igbinary && apt-get -y install php${LARADOCK_PHP_VERSION}-memcached; \
    fi; \
    if [ ${INSTALL_XMLRPC} = true ]; then \
        apt-get install -yqq php${LARADOCK_PHP_VERSION}-xmlrpc; \
    fi; \
    if [ ${INSTALL_PHPDECIMAL} = true ] && ! [ $(php -r "echo PHP_MAJOR_VERSION;") = "5" ]; then \
        apt-get install -yqq libmpdec-dev pecl install decimal &&  \
        echo "extension=decimal.so" >> /etc/php/${LARADOCK_PHP_VERSION}/mods-available/decimal.ini &&  \
        ln -s /etc/php/${LARADOCK_PHP_VERSION}/mods-available/decimal.ini /etc/php/${LARADOCK_PHP_VERSION}/cli/conf.d/30-decimal.ini && \
        php -m | grep -q 'decimal'; \
    fi

###########################################################################
# Node / NVM:
###########################################################################

# Check if NVM needs to be installed
ARG NODE_VERSION=node
ENV NODE_VERSION ${NODE_VERSION}
ARG INSTALL_NODE=false
ARG INSTALL_NPM_CHECK_UPDATES_CLI=false
ARG NPM_REGISTRY
ENV NPM_REGISTRY ${NPM_REGISTRY}
ARG NPM_FETCH_RETRIES
ENV NPM_FETCH_RETRIES ${NPM_FETCH_RETRIES}
ARG NPM_FETCH_RETRY_FACTOR
ENV NPM_FETCH_RETRY_FACTOR ${NPM_FETCH_RETRY_FACTOR}
ARG NPM_FETCH_RETRY_MINTIMEOUT
ENV NPM_FETCH_RETRY_MINTIMEOUT ${NPM_FETCH_RETRY_MINTIMEOUT}
ARG NPM_FETCH_RETRY_MAXTIMEOUT
ENV NPM_FETCH_RETRY_MAXTIMEOUT ${NPM_FETCH_RETRY_MAXTIMEOUT}
ENV NVM_DIR /home/laradock/.nvm
ARG NVM_NODEJS_ORG_MIRROR
ENV NVM_NODEJS_ORG_MIRROR ${NVM_NODEJS_ORG_MIRROR}

RUN if [ ${INSTALL_NODE} = true ]; then \
    mkdir -p $NVM_DIR && \
    curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.2/install.sh | bash && \
    . $NVM_DIR/nvm.sh && \
    nvm install ${NODE_VERSION} && \
    nvm use ${NODE_VERSION} && \
    nvm alias ${NODE_VERSION} && \
    npm cache clear --force &&  \
    npm config set fetch-retries ${NPM_FETCH_RETRIES} &&  \
    npm config set fetch-retry-factor ${NPM_FETCH_RETRY_FACTOR} &&  \
    npm config set fetch-retry-mintimeout ${NPM_FETCH_RETRY_MINTIMEOUT} &&  \
    npm config set fetch-retry-maxtimeout ${NPM_FETCH_RETRY_MAXTIMEOUT} &&  \
    if [ ${NPM_REGISTRY} ]; then \
        npm config set registry ${NPM_REGISTRY}; \
    fi && \
    if [ ${INSTALL_NPM_CHECK_UPDATES_CLI} = true ]; then \
        npm install -g npm-check-updates; \
    fi \
;fi

RUN if [ ${INSTALL_NODE} = true ]; then \
    echo "" >> ~/.bashrc && \
    echo 'export NVM_DIR="$HOME/.nvm"' >> ~/.bashrc && \
    echo '[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"  # This loads nvm' >> ~/.bashrc; \
fi

# Add NVM binaries to root's .bashrc
USER root

RUN if [ ${INSTALL_NODE} = true ]; then \
    echo "" >> ~/.bashrc && \
    echo 'export NVM_DIR="/home/laradock/.nvm"' >> ~/.bashrc && \
    echo '[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"  # This loads nvm' >> ~/.bashrc \
;fi

RUN if [ ${INSTALL_NODE} = true ]; then \
    find $NVM_DIR -type f -name node -exec ln -s {} /usr/local/bin/node \; && \
    NODE_MODS_DIR="$NVM_DIR/versions/node/$(node -v)/lib/node_modules" && \
    ln -s $NODE_MODS_DIR/npm/bin/npm-cli.js /usr/local/bin/npm && \
    ln -s $NODE_MODS_DIR/npm/bin/npx-cli.js /usr/local/bin/npx \
;fi

RUN if [ ${NPM_REGISTRY} ]; then \
    . ~/.bashrc && npm config set registry ${NPM_REGISTRY} \
;fi

COPY ./.npmrc /root/.npmrc
COPY ./.npmrc /home/laradock/.npmrc

###########################################################################
# Check PHP version:
###########################################################################

RUN set -xe; php -v | head -n 1 | grep -q "PHP ${LARADOCK_PHP_VERSION}."

###########################################################################
# DNS utilities:
###########################################################################

USER root

ARG INSTALL_DNSUTILS=false

RUN if [ ${INSTALL_DNSUTILS} = true ]; then \
    apt-get update && apt-get install -y dnsutils; \
fi

###########################################################################
# Final Touch
###########################################################################

USER root

RUN apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* && rm -f /var/log/lastlog /var/log/faillog

WORKDIR /var/www
