FROM ubuntu:20.04 AS builder

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && \
    apt-get install -y \
    build-essential \
    libpcre3-dev \
    libssl-dev \
    perl \
    make \
    curl \
    zlib1g-dev \
    libpq-dev \
    libgd-dev \
    wget \
    && rm -rf /var/lib/apt/lists/*

ARG OPENRESTY_VERSION=1.21.4.1
WORKDIR /tmp

RUN wget https://openresty.org/download/openresty-${OPENRESTY_VERSION}.tar.gz && \
    tar -xzf openresty-${OPENRESTY_VERSION}.tar.gz && \
    cd openresty-${OPENRESTY_VERSION} && \
    ./configure \
        --prefix=/opt/openresty \
        --with-pcre-jit \
        --with-ipv6 \
        --without-http_redis2_module \
        --with-http_iconv_module \
        --with-http_postgres_module \
        -j8 && \
    make -j8 && \
    make install

FROM ubuntu:20.04

RUN apt-get update && \
    apt-get install -y \
    libpcre3 \
    libssl1.1 \
    libpq5 \
    curl \
    && rm -rf /var/lib/apt/lists/*

COPY --from=builder /opt/openresty /opt/openresty
COPY nginx.conf /opt/openresty/nginx/conf/nginx.conf
COPY lua-scripts/ /opt/openresty/nginx/lua-scripts/

ENV PATH=/opt/openresty/nginx/sbin:$PATH

WORKDIR /opt/openresty/nginx

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
