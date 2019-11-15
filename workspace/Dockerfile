FROM xonefobic/workspace:latest

LABEL maintainer="XoneFobic <xonefobic@derp.af>"

ARG PHP_VERSION
ENV DEBIAN_FRONTEND noninteractive

USER root
ARG PUID=1000
ENV PUID ${PUID}
ARG PGID=1000
ENV PGID ${PGID}
RUN set -xe && \
    apt-get update -yqq && \
    pecl channel-update pecl.php.net && \
    groupadd -g ${PGID} derp && \
    useradd -u ${PUID} -g derp -m derp -G docker_env && \
    usermod -p "*" derp -s /bin/bash && \
    apt-get install -yqq \
    apt-utils \
    libzip-dev zip unzip \
    php${PHP_VERSION}-zip \
    nasm && \
    php -m | grep -q 'zip'
ARG TZ=UTC
ENV TZ ${TZ}
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ >/etc/timezone

# Aliases

USER root
COPY ./files/aliases.sh /root/aliases.sh
COPY ./files/aliases.sh /home/derp/aliases.sh
RUN sed -i 's/\r//' /root/aliases.sh && \
    sed -i 's/\r//' /home/derp/aliases.sh && \
    chown derp:derp /home/derp/aliases.sh && \
    echo "" >>~/.bashrc && \
    echo "# Load Custom Aliases" >>~/.bashrc && \
    echo "source ~/aliases.sh" >>~/.bashrc && \
    echo "" >>~/.bashrc

USER derp
RUN echo "" >>~/.bashrc && \
    echo "# Load Custom Aliases" >>~/.bashrc && \
    echo "source ~/aliases.sh" >>~/.bashrc && \
    echo "" >>~/.bashrc

# Composer

USER root
COPY ./files/composer.json /home/derp/.composer/composer.json
RUN chown -R derp:derp /home/derp/.composer
RUN echo "" >>~/.bashrc && echo 'export PATH="$HOME/.composer/vendor/bin:$PATH"' >>~/.bashrc

USER derp

ARG COMPOSER_RUN_GLOBAL_INSTALL=false
ENV COMPOSER_RUN_GLOBAL_INSTALL ${COMPOSER_RUN_GLOBAL_INSTALL}

RUN if [ ${COMPOSER_RUN_GLOBAL_INSTALL} = true ] ; then composer global install ; fi
RUN echo "" >> ~/.bashrc && echo 'export PATH="~/.composer/vendor/bin:$PATH"' >> ~/.bashrc

# Crontab

USER root

COPY ./files/crontab /etc/cron.d
RUN chmod -R 644 /etc/cron.d

# xDebug

USER root
ARG INSTALL_XDEBUG=false
RUN if [ ${INSTALL_XDEBUG} = true ]; then \
    apt-get install -y php7.3-xdebug && \
    sed -i 's/^;//g' /etc/php/7.3/cli/conf.d/20-xdebug.ini \
;fi

COPY ./files/xdebug.ini /etc/php/7.3/cli/conf.d/xdebug.ini

# Node

USER derp

ARG NODE_VERSION=Node
ENV NODE_VERSION ${NODE_VERSION}
ARG INSTALL_NODE=false
ENV NVM_DIR /home/derp/.nvm

RUN if [ ${INSTALL_NODE} = true ]; then \
    mkdir -p ${NVM_DIR} && \
    curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.35.1/install.sh | bash \
    && . ${NVM_DIR}/nvm.sh \
    && nvm install ${NODE_VERSION} \
    && nvm use ${NODE_VERSION} \
    && nvm alias ${NODE_VERSION} \
    && ln -s `npm bin --global` /home/derp/.node-bin \
;fi

RUN if [ ${INSTALL_NODE} = true ]; then \
    echo "" >> ~/.bashrc && \
    echo 'export NVM_DIR="$HOME/.nvm"' >> ~/.bashrc && \
    echo '[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"  # This loads nvm' >> ~/.bashrc \
;fi

USER root

RUN if [ ${INSTALL_NODE} = true ]; then \
    echo "" >> ~/.bashrc && \
    echo 'export NVM_DIR="/home/derp/.nvm"' >> ~/.bashrc && \
    echo '[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"  # This loads nvm' >> ~/.bashrc \
;fi

ENV PATH $PATH:/home/derp/.node-bin

RUN if [ ${INSTALL_NODE} = true ]; then \
    find $NVM_DIR -type f -name node -exec ln -s {} /usr/local/bin/node \; && \
    NODE_MODS_DIR="$NVM_DIR/versions/node/$(node -v)/lib/node_modules" && \
    ln -s $NODE_MODS_DIR/npm/bin/npm-cli.js /usr/local/bin/npm && \
    ln -s $NODE_MODS_DIR/npm/bin/npx-cli.js /usr/local/bin/npx \
;fi

# Yarn

USER derp

ARG INSTALL_YARN=false
ARG YARN_VERSION=latest
ENV YARN_VERSION ${YARN_VERSION}

RUN if [ ${INSTALL_YARN} = true ]; then \
    [ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh" && \
    if [ ${YARN_VERSION} = "latest" ]; then \
        curl -o- -L https://yarnpkg.com/install.sh | bash; \
    else \
        curl -o- -L https://yarnpkg.com/install.sh | bash -s -- --version ${YARN_VERSION}; \
    fi && \
    echo "" >> ~/.bashrc && \
    echo 'export PATH="$HOME/.yarn/bin:$PATH"' >> ~/.bashrc \
;fi

USER root

RUN if [ ${INSTALL_YARN} = true ]; then \
    echo "" >> ~/.bashrc && \
    echo 'export YARN_DIR="/home/derp/.yarn"' >> ~/.bashrc && \
    echo 'export PATH="$YARN_DIR/bin:$PATH"' >> ~/.bashrc \
;fi

ENV PATH $PATH:/home/derp/.yarn/bin

# Cleaning up

USER root

RUN apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* && \
    rm /var/log/lastlog /var/log/faillog

# Fin

WORKDIR /var/www