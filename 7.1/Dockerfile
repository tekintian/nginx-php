FROM tekintian/php:7.1.24
LABEL maintainer="TekinTian tekintian@gmail.com"

ENV NGINX_INSTALL_DIR=/usr/local/nginx \
    NGINX_VERSION=1.15.7 \
    PCRE_VERSION=8.42 \
    OPENSSL_VERSION=1.1.0j \
    RUN_USER=www \
    WWWROOT_DIR=/home/wwwroot \
    WWWLOGS_DIR=/home/wwwlogs \
    REMOTE_SRC_PATH=https://github.com/tekintian/nginx-php/raw/master

WORKDIR /tmp
#
#最新稳定版本openssl 
# https://www.openssl.org/source/openssl-1.1.0j.tar.gz
#nginx http://nginx.taobao.org/download.html
#pcre  ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre
#install build deps
RUN buildDeps='\
                gcc \
                g++ \
                make \
                cmake \
                autoconf \
                libssl-dev \
                libcurl4-openssl-dev \
                libxslt-dev \
                libicu-dev \
                libxml2-dev \
                libjpeg-dev \
                libpng-dev \
                libfreetype6-dev \
                libsasl2-dev \
                libevent-dev \
                libpcre3-dev \
                libgpgme11-dev \
                pkg-config \
                patch \
                libgpgme11-dev \
                --no-install-recommends ' \
        && apt-get update && apt-get install -y ${buildDeps} && rm -rf /var/lib/apt/lists/* && \
    wget -c --no-check-certificate https://ftp.pcre.org/pub/pcre/pcre-${PCRE_VERSION}.tar.gz && \
    wget --no-check-certificate https://www.openssl.org/source/openssl-${OPENSSL_VERSION}.tar.gz && \
    wget -c --no-check-certificate http://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz && \
    #解压
    tar xzf pcre-${PCRE_VERSION}.tar.gz && \
    tar xzf openssl-${OPENSSL_VERSION}.tar.gz && \
    tar xzf nginx-${NGINX_VERSION}.tar.gz && \
# install nginx
    cd nginx-${NGINX_VERSION} && \
    sed -i 's@CFLAGS="$CFLAGS -g"@#CFLAGS="$CFLAGS -g"@' auto/cc/gcc && \
    ./configure \
        --prefix=${NGINX_INSTALL_DIR} \
        --user=${RUN_USER} --group=${RUN_USER} \
        --with-http_v2_module \
        --with-http_ssl_module \
        --with-ipv6 \
        --with-http_gzip_static_module \
        --with-http_realip_module \
        --with-http_flv_module \
        --with-http_mp4_module \
        --with-openssl=/tmp/openssl-${OPENSSL_VERSION} \
        --with-pcre=/tmp/pcre-${PCRE_VERSION} \
        --with-pcre-jit \
        --with-ld-opt='-ljemalloc' && \
    make && make install && \
    mkdir -p ${NGINX_INSTALL_DIR}/conf/rewrite && \
    mkdir -p ${NGINX_INSTALL_DIR}/logs/hack && \
    #备份默认的配置文件
    cp -a ${NGINX_INSTALL_DIR}/conf/nginx.conf ${NGINX_INSTALL_DIR}/conf/nginx.conf.bak && \
    cd /tmp/ && rm -rf /tmp/* && \
    #remove build deps
    apt-get purge -y --auto-remove ${buildDeps}

ADD ${REMOTE_SRC_PATH}/nginx.conf ${NGINX_INSTALL_DIR}/conf/nginx.conf
ADD ${REMOTE_SRC_PATH}/nginx-init /etc/init.d/nginx
RUN sed -i "s@^worker_processes.*@worker_processes auto;\nworker_cpu_affinity auto;\n@" ${NGINX_INSTALL_DIR}/conf/nginx.conf

# run install script
ADD ${REMOTE_SRC_PATH}/install.sh /tmp/install.sh
RUN chmod 777 install.sh && \
    bash install.sh && \
    rm -rf /tmp/*

# ending
WORKDIR ${WWWROOT_DIR}
RUN mkdir -p ${WWWLOGS_DIR} && \
    mkdir -p ${WWWROOT_DIR}/default && \
    echo "<?php phpinfo();" > /${WWWROOT_DIR}/default/index.php

# expose port
EXPOSE 80 443

# Set the entrypoint script.
ADD ${REMOTE_SRC_PATH}/entrypoint.sh /entrypoint.sh
RUN chmod 777 /entrypoint.sh
ENTRYPOINT ["/entrypoint.sh"]

# Define the default command.
CMD ["nginx", "-g", "daemon off;"]