FROM ubuntu:xenial
MAINTAINER tommylike <tommylikehu@gmail.com>
# Set env
USER root
ENV USER root

RUN apt -y update \
    && apt install -y wget \
    && apt install -y git \
        && apt install -y rpm \
        && apt install -y maven \
        && apt install -y createrepo \
    && apt install -y make build-essential libpcre3 libpcre3-dev openssl libssl-dev software-properties-common tzdata\
    && rm -rf /var/lib/{apt,dpkg,cache,log}/

# install java
RUN wget --no-cookies --no-check-certificate --header "Cookie: gpw_e24=http%3A%2F%2Fwww.oracle.com%2F; oraclelicense=accept-securebackup-cookie" "http://download.oracle.com/otn-pub/java/jdk/8u141-b15/336fa29ff2bb4ef291e347e091f7f4a7/jdk-8u141-linux-x64.tar.gz"
RUN mkdir /usr/local/java
RUN tar zxvf jdk-8u141-linux-x64.tar.gz -C /usr/local/java
RUN ln -s /usr/local/java/jdk1.8.0_141 /usr/local/java/jdk
ENV JAVA_HOME /usr/local/java/jdk
ENV JRE_HOME ${JAVA_HOME}/jre
ENV CLASSPATH .:${JAVA_HOME}/lib:${JRE_HOME}/lib
ENV PATH ${JAVA_HOME}/bin:$PATH

# app-publish
RUN git clone https://github.com/7Light/app-publish.git
WORKDIR /app-publish
RUN mvn clean install -s settings.xml


# install mmdb
RUN add-apt-repository -y ppa:maxmind/ppa \
    && apt -y update && apt -y install libmaxminddb0 libmaxminddb-dev mmdb-bin \
    && rm -rf /var/lib/{apt,dpkg,cache,log}/

# prepare nginx
RUN wget http://nginx.org/download/nginx-1.10.3.tar.gz \
    && tar -xvf nginx-1.10.3.tar.gz && rm nginx-1.10.3.tar.gz

# prepare nginx fancyindex
RUN git clone https://github.com/aperezdc/ngx-fancyindex.git ngx-fancyindex

# prepare geoip_module
RUN wget https://github.com/leev/ngx_http_geoip2_module/archive/master.tar.gz && \
    tar -xvf master.tar.gz && \
    mv ngx_http_geoip2_module-master ngx_http_geoip2_module && \
    rm master.tar.gz

RUN cd nginx-1.10.3 \
    && ./configure --prefix=/etc/nginx/ --add-module=../ngx-fancyindex --with-http_ssl_module --with-http_realip_module --add-module=../ngx_http_geoip2_module\
    && make && make install \
    && cd ../ && rm -rf nginx-1.10.3  && rm -rf ngx-fancyindex && rm -rf ngx_http_geoip2_module\
    && mkdir -p /var/log/nginx

# prepare geo database
RUN mkdir /etc/nginx/geoip && cd /etc/nginx/geoip && \
    wget https://mindspore.obs.cn-north-4.myhuaweicloud.com:443/GeoLite2-Country_20200331.tar.gz && \
    tar -xvf GeoLite2-Country_20200331.tar.gz && mv GeoLite2-Country_20200331 country && rm GeoLite2-Country_20200331.tar.gz &&\
    wget https://mindspore.obs.cn-north-4.myhuaweicloud.com:443/GeoLite2-City_20200331.tar.gz && \
    tar -xvf GeoLite2-City_20200331.tar.gz && mv GeoLite2-City_20200331 city && rm GeoLite2-City_20200331.tar.gz

RUN sed -i '47 a location /publish {\nproxy_pass http://localhost:8080;\nproxy_read_timeout 2400s;}'  /etc/nginx/conf/nginx.conf

ENV PATH="/etc/nginx/sbin:${PATH}"
ENV GEO_COUNTRY_DB_PATH="/etc/nginx/geoip/country/GeoLite2-Country.mmdb"
ENV GEO_CITY_DB_PATH="/etc/nginx/geoip/city/GeoLite2-City.mmdb"

WORKDIR /usr/local
RUN touch entrypoint.sh
RUN echo "#!/bin/bash\nnginx\njava -jar /app-publish/target/app-publish-1.0.jar" >>  /usr/local/entrypoint.sh
RUN chmod u+x /usr/local/entrypoint.sh
ENTRYPOINT ["/usr/local/entrypoint.sh"]