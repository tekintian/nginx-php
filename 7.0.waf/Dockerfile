FROM tekintian/php:7.0.28
MAINTAINER Tekin Tian <tekintian@gmail.com>


ENV NGINX_INSTALL_DIR=/usr/local/nginx \
    NGINX_VERSION=1.13.9 \
    PCRE_VERSION=8.41 \
    OPENSSL_VERSION=1.1.0g \
    LuaJIT_VERSION=2.0.5 \
    LNM_VERSION=0.10.12rc2 \
    NDK_VERSION=0.3.1rc1 \
    RUN_USER=www \
    WWWROOT_DIR=/home/wwwroot \
    WWWLOGS_DIR=/home/wwwlogs \
    REMOTE_SRC_PATH=https://github.com/tekintian/nginx-php/raw/master

WORKDIR /tmp
#
#最新稳定版本openssl 
# https://www.openssl.org/source/openssl-1.1.0g.tar.gz
# https://www.openssl.org/source/snapshot/
#nginx http://nginx.taobao.org/download.html
# Luajit
# http://luajit.org/download.html
# http://luajit.org/download/LuaJIT-2.0.5.tar.gz
# https://github.com/openresty/lua-nginx-module/
# https://github.com/simplresty/ngx_devel_kit
#https://github.com/simplresty/ngx_devel_kit/archive/v0.3.1rc1.tar.gz
#WAF配置文件 https://github.com/loveshell/ngx_lua_waf
#pcre  ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre
#install build deps
RUN  buildDeps='\
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
# && ln -s /usr/local/luajit/lib/libluajit-5.1.so.2 /lib/libluajit-5.1.so.2 
#LuaJIT. https://github.com/openresty/lua-nginx-module/archive/v0.10.12rc2.tar.gz
    wget -c --no-check-certificate http://luajit.org/download/LuaJIT-$LuaJIT_VERSION.tar.gz && \
    tar xzf LuaJIT-$LuaJIT_VERSION.tar.gz && \
    cd LuaJIT-$LuaJIT_VERSION && make && \
    make install PREFIX=/usr/local/luajit && \
    export LUAJIT_LIB=/usr/local/luajit/lib && export LUAJIT_INC=/usr/local/luajit/include/luajit-2.0  && echo "/usr/local/luajit/lib" >> /etc/ld.so.conf && ldconfig && \
    cd /tmp/ && \
# download related src
    wget -c --no-check-certificate ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-${PCRE_VERSION}.tar.gz && \
    wget --no-check-certificate https://www.openssl.org/source/openssl-${OPENSSL_VERSION}.tar.gz && \

    wget -c --no-check-certificate https://github.com/openresty/lua-nginx-module/archive/v${LNM_VERSION}.tar.gz -O  lua-nginx-module-${LNM_VERSION}.tar.gz && \
    wget -c --no-check-certificate https://github.com/simplresty/ngx_devel_kit/archive/v${NDK_VERSION}.tar.gz -O ngx_devel_kit-${NDK_VERSION}.tar.gz && \

    wget -c --no-check-certificate http://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz && \
    #解压
    tar xzf pcre-${PCRE_VERSION}.tar.gz && \
    tar xzf openssl-${OPENSSL_VERSION}.tar.gz && \
    tar xzf lua-nginx-module-${LNM_VERSION}.tar.gz && \
    tar xzf ngx_devel_kit-${NDK_VERSION}.tar.gz && \
    tar xzf nginx-${NGINX_VERSION}.tar.gz  && \
# install nginx
  cd nginx-${NGINX_VERSION} && \
    sed -i 's@CFLAGS="$CFLAGS -g"@#CFLAGS="$CFLAGS -g"@' auto/cc/gcc && \
    ./configure \
        --prefix=${NGINX_INSTALL_DIR} \
        --user=${RUN_USER} --group=${RUN_USER} \
        --with-ld-opt="-Wl,-rpath,/usr/local/luajit/lib" \
        --add-module=/tmp/ngx_devel_kit-${NDK_VERSION} \
        --add-module=/tmp/lua-nginx-module-${LNM_VERSION} \
        --with-http_v2_module \
        --with-http_ssl_module \
        --with-http_gzip_static_module \
        --with-http_stub_status_module  \
        --with-http_realip_module \
        --with-http_flv_module \
        --with-http_mp4_module \
        --with-openssl=/tmp/openssl-${OPENSSL_VERSION} \
        --with-pcre=/tmp/pcre-${PCRE_VERSION} \
        --with-pcre-jit \
        --with-ld-opt='-ljemalloc' && \
    make -j2 && make install && \
    mkdir -p ${NGINX_INSTALL_DIR}/conf/rewrite && \
    mkdir -p ${NGINX_INSTALL_DIR}/logs/hack && \
    #备份默认的配置文件
    cp -a ${NGINX_INSTALL_DIR}/conf/nginx.conf ${NGINX_INSTALL_DIR}/conf/nginx.conf.bak && \
    cd /tmp/ && rm -rf /tmp/* && \
    #remove build deps
    apt-get purge -y --auto-remove ${buildDeps}

ADD ${REMOTE_SRC_PATH}/nginx_waf.conf ${NGINX_INSTALL_DIR}/conf/nginx.conf
#ADD ${REMOTE_SRC_PATH}/src/waf.zip ${NGINX_INSTALL_DIR}/conf/
ADD ${REMOTE_SRC_PATH}/nginx-init /etc/init.d/nginx
RUN sed -i "s@^worker_processes.*@worker_processes auto;\nworker_cpu_affinity auto;\n@" ${NGINX_INSTALL_DIR}/conf/nginx.conf

# run waf install script
ADD ${REMOTE_SRC_PATH}/install_waf.sh /tmp/install_waf.sh
RUN chmod 777 install_waf.sh && \
    bash install_waf.sh && \
    rm -rf /tmp/*

# ending
WORKDIR ${WWWROOT_DIR}

# expose port
EXPOSE 80 443

# Set the entrypoint script.
ADD ${REMOTE_SRC_PATH}/entrypoint.sh /entrypoint.sh
RUN chmod 777 /entrypoint.sh
ENTRYPOINT ["/entrypoint.sh"]

# Define the default command.
CMD ["nginx", "-g", "daemon off;"]