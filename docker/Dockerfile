# 基础镜像
FROM alpine:latest

# 创建者信息
MAINTAINER ZTUO

# 添加更新源
RUN echo "http://mirrors.aliyun.com/alpine/latest-stable/main/" > /etc/apk/repositories && \
    echo "http://mirrors.aliyun.com/alpine/latest-stable/community/" >> /etc/apk/repositories

# 设置系统编码,解决中文会乱码
ENV LANG C.UTF-8
ENV LANGUAGE C.UTF-8

# 设置时区
ENV TIMEZONE Asia/Shanghai

# 修改时区
RUN apk -U upgrade && \
	apk add --no-cache tzdata && \
	cp /usr/share/zoneinfo/${TIMEZONE} /etc/localtime && \
	echo "${TIMEZONE}" > /etc/timezone && \
	apk del tzdata

# 安装JDK
ENV JAVA_HOME /opt/openjdk-14
ENV PATH $JAVA_HOME/bin:$PATH
ENV JAVA_VERSION 14-ea+8
ENV JAVA_URL https://download.java.net/java/early_access/alpine/8/binaries/openjdk-14-ea+8_linux-x64-musl_bin.tar.gz
ENV JAVA_SHA256 f1ea9cf80237326892858c4664ddb3d4b04f4aa484ed1b95d5c69179d1a514c1
RUN set -eux; \
	wget -O /openjdk.tgz "$JAVA_URL"; \
	echo "$JAVA_SHA256 */openjdk.tgz" | sha256sum -c -; \
	mkdir -p "$JAVA_HOME"; \
	tar --extract --file /openjdk.tgz --directory "$JAVA_HOME" --strip-components 1; \
	rm /openjdk.tgz; \
	java -Xshare:dump; \
	java --version; \
	javac --version

# 安装MYSQL
ENV MYSQL_VERSION 10.3.17-r0
RUN apk -U upgrade && \
	apk add --no-cache mysql=${MYSQL_VERSION} mysql-client=${MYSQL_VERSION} mariadb-server-utils=${MYSQL_VERSION} && \
	addgroup mysql mysql && \
	mkdir /scripts && \
	rm -rf /var/cache/apk/*
EXPOSE 3306
VOLUME ["/var/lib/mysql"]

# 安装NGINX
ENV NGINX_VERSION 1.17.3
ENV NJS_VERSION   0.3.5
ENV PKG_RELEASE   1

RUN set -x \
    && addgroup -g 102 -S nginx \
    && adduser -S -D -H -u 102 -h /var/cache/nginx -s /sbin/nologin -G nginx -g nginx nginx \
    && apkArch="$(cat /etc/apk/arch)" \
    && nginxPackages=" \
        nginx=${NGINX_VERSION}-r${PKG_RELEASE} \
        nginx-module-xslt=${NGINX_VERSION}-r${PKG_RELEASE} \
        nginx-module-geoip=${NGINX_VERSION}-r${PKG_RELEASE} \
        nginx-module-image-filter=${NGINX_VERSION}-r${PKG_RELEASE} \
        nginx-module-njs=${NGINX_VERSION}.${NJS_VERSION}-r${PKG_RELEASE} " \
    && case "$apkArch" in \
        x86_64) \
            set -x \
            && KEY_SHA512="e7fa8303923d9b95db37a77ad46c68fd4755ff935d0a534d26eba83de193c76166c68bfe7f65471bf8881004ef4aa6df3e34689c305662750c0172fca5d8552a *stdin" \
            && apk add --no-cache --virtual .cert-deps \
                openssl \
            && wget -O /tmp/nginx_signing.rsa.pub https://nginx.org/keys/nginx_signing.rsa.pub \
            && if [ "$(openssl rsa -pubin -in /tmp/nginx_signing.rsa.pub -text -noout | openssl sha512 -r)" = "$KEY_SHA512" ]; then \
                echo "key verification succeeded!"; \
                mv /tmp/nginx_signing.rsa.pub /etc/apk/keys/; \
            else \
                echo "key verification failed!"; \
                exit 1; \
            fi \
            && printf "%s%s%s\n" \
                "https://nginx.org/packages/mainline/alpine/v" \
                `egrep -o '^[0-9]+\.[0-9]+' /etc/alpine-release` \
                "/main" \
            | tee -a /etc/apk/repositories \
            && apk del .cert-deps \
            ;; \
        *) \
            set -x \
            && tempDir="$(mktemp -d)" \
            && chown nobody:nobody $tempDir \
            && apk add --no-cache --virtual .build-deps \
                gcc \
                libc-dev \
                make \
                openssl-dev \
                pcre-dev \
                zlib-dev \
                linux-headers \
                libxslt-dev \
                gd-dev \
                geoip-dev \
                perl-dev \
                libedit-dev \
                mercurial \
                bash \
                alpine-sdk \
                findutils \
            && su nobody -s /bin/sh -c " \
                export HOME=${tempDir} \
                && cd ${tempDir} \
                && hg clone https://hg.nginx.org/pkg-oss \
                && cd pkg-oss \
                && hg up ${NGINX_VERSION}-${PKG_RELEASE} \
                && cd alpine \
                && make all \
                && apk index -o ${tempDir}/packages/alpine/${apkArch}/APKINDEX.tar.gz ${tempDir}/packages/alpine/${apkArch}/*.apk \
                && abuild-sign -k ${tempDir}/.abuild/abuild-key.rsa ${tempDir}/packages/alpine/${apkArch}/APKINDEX.tar.gz "\
            && echo "${tempDir}/packages/alpine/" >> /etc/apk/repositories \
            && cp ${tempDir}/.abuild/abuild-key.rsa.pub /etc/apk/keys/ \
            && apk del .build-deps \
            ;; \
    esac \
    && apk add --no-cache $nginxPackages \
    && if [ -n "$tempDir" ]; then rm -rf "$tempDir"; fi \
    && if [ -n "/etc/apk/keys/abuild-key.rsa.pub" ]; then rm -f /etc/apk/keys/abuild-key.rsa.pub; fi \
    && if [ -n "/etc/apk/keys/nginx_signing.rsa.pub" ]; then rm -f /etc/apk/keys/nginx_signing.rsa.pub; fi \
    && sed -i '$ d' /etc/apk/repositories \
    && apk add --no-cache --virtual .gettext gettext \
    && mv /usr/bin/envsubst /tmp/ \
    && runDeps="$( \
        scanelf --needed --nobanner /tmp/envsubst \
            | awk '{ gsub(/,/, "\nso:", $2); print "so:" $2 }' \
            | sort -u \
            | xargs -r apk info --installed \
            | sort -u \
    )" \
    && apk add --no-cache $runDeps \
    && apk del .gettext \
    && mv /tmp/envsubst /usr/local/bin/ \
    && ln -sf /dev/stdout /var/log/nginx/access.log \
    && ln -sf /dev/stderr /var/log/nginx/error.log \
    && mkdir -p /mnt/upload/ \
    && mkdir -p /mnt/jar/ \
    && mkdir -p /etc/nginx/
STOPSIGNAL SIGTERM
EXPOSE 80
	
COPY nginx.conf /etc/nginx/nginx.conf
COPY db.sql /scripts/db.sql
COPY my.cnf /etc/my.cnf
COPY public-service.jar /mnt/jar/public-service.jar
COPY startup.sh /scripts/startup.sh
RUN chmod +x /scripts/startup.sh && \
    chown -R mysql:mysql /scripts/db.sql

ENTRYPOINT  ["/scripts/startup.sh"]

