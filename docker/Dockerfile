#############
# Nginx Builder
#############

FROM debian:trixie-slim AS nginxbuilder

ARG OPENRESTY_VERSION=1.29.2.3
ARG LUA_VERSION=5.5.0
ARG LUAROCKS_VERSION=3.13.0

RUN apt-get update \
	&& apt-get install -y \
	build-essential \
	ca-certificates \
	libncurses-dev \
	libpcre2-dev \
	libreadline-dev \
	libssl-dev \
	openssl unzip \
	wget \
	zlib1g-dev \
	git \
	libmaxminddb-dev

# Lua build
COPY ./scripts/build-lua /tmp/build-lua
RUN /tmp/build-lua

# Nginx build
COPY ./scripts/build-openresty /tmp/build-openresty
RUN /tmp/build-openresty

#############
# Final Image
#############

FROM debian:trixie-slim AS final
LABEL maintainer="Jamie Curnow <jc@jc21.com>"
LABEL maintainer="Daan Selen <dselen@nerthus.nl>"

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

ARG TARGETPLATFORM
RUN echo "Base: debian:trixie-slim, ${TARGETPLATFORM:-linux/amd64}" > /built-for-arch

# OpenResty uses LuaJIT which has a dependency on GCC
RUN apt-get update \
	&& apt-get install -y --no-install-recommends \
	apache2-utils \
	ca-certificates \
	curl \
	figlet \
	jq \
	libncurses6 \
	libpcre2-8-0 \
	libreadline8 \
	openssl \
	perl \
	tzdata \
	unzip \
	zlib1g \
	gettext \
	wget \
	xz-utils \
	libmaxminddb-dev \
	moreutils \
	&& apt-get clean \
	&& rm -rf /var/lib/apt/lists/* \
	&& rm -rf /var/cache/* /var/log/* /tmp/* /var/lib/dpkg/status-old

COPY ./files/.bashrc /root/.bashrc

# Copy lua and luarocks builds from first image
COPY --from=nginxbuilder /tmp/lua /tmp/lua
COPY --from=nginxbuilder /tmp/luarocks /tmp/luarocks
COPY ./scripts/install-lua /tmp/install-lua

# Copy openresty build from first image
COPY --from=nginxbuilder /tmp/openresty /tmp/openresty
COPY ./scripts/install-openresty /tmp/install-openresty

# Copy crowdsec openresty bouncer install script
COPY ./scripts/install-crowdsec_openresty_bouncer /tmp/install-crowdsec_openresty_bouncer

ARG OPENRESTY_VERSION=1.29.2.3
ARG CROWDSEC_OPENRESTY_BOUNCER_VERSION=1.7.7
ENV SSL_CERT_FILE=/etc/ssl/certs/ca-certificates.crt \
	OPENRESTY_VERSION=${OPENRESTY_VERSION} \
	CROWDSEC_OPENRESTY_BOUNCER_VERSION=${CROWDSEC_OPENRESTY_BOUNCER_VERSION}

# Install openresty, lua, then clean up file system
RUN apt-get update \
	&& apt-get install -y gcc make socat git \
	&& /tmp/install-lua \
	&& /tmp/install-openresty \
	&& apt-get remove -y make gcc git wget gettext \
	&& apt-get autoremove -y \
	&& apt-get clean \
	&& rm -rf /var/lib/apt/lists/* \
	&& rm -rf /var/cache/* /var/log/* /tmp/* /var/lib/dpkg/status-old

LABEL org.label-schema.schema-version="1.0" \
	org.label-schema.license="MIT" \
	org.label-schema.name="nginx-full" \
	org.label-schema.description="A base image for use by Nginx Proxy Manager" \
	org.label-schema.url="https://github.com/nginxproxymanager/docker-nginx-full" \
	org.label-schema.vcs-url="https://github.com/nginxproxymanager/docker-nginx-full.git" \
	org.label-schema.cmd="docker run --rm -ti nginxproxymanager/nginx-full:latest"
