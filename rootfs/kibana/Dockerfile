FROM ubuntu:17.10
LABEL MAINTAINER="Jim Conner <snafu.x@gmail.com>"
LABEL DESCRIPTION="Samsung SDS container for Kibana"

ENV GOSU_VERSION 1.10
ENV TINI_VERSION v0.9.0
ENV KIBANA_VERSION 5.6.3

ENV GPGKEYSERVER ha.pool.sks-keyservers.net

ENV GOSU_GPGFINGERPRINT B42F6819007F00F88E364FD4036A9C25BF357DD4
ENV TINI_GPGFINGERPRINT 6380DC428747F6C393FEACA59A84159D7001A4E5
ENV ES_GPGFINGERPRINT   46095ACC8548582C1A2699A9D27D666CD88E42B4

ENV APT_BLD_DEPS        "apt-transport-https libfontconfig libfreetype6"

# add our user and group first to make sure their IDs get assigned consistently
RUN groupadd -r kibana && useradd -r -m -g kibana kibana

RUN export DEBIAN_FRONTEND=noninteractive && \
    apt-get -qq update && \
    apt-get install -y -qq --no-install-recommends $APT_BLD_DEPS \
            wget dirmngr ca-certificates  && \
    rm -rf /var/lib/apt/lists/*

# grab gosu for easy step-down from root
RUN set -x && \
	wget -O /usr/local/bin/gosu "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$(dpkg --print-architecture)" && \
	wget -O /usr/local/bin/gosu.asc "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$(dpkg --print-architecture).asc" && \
	export GNUPGHOME="$(mktemp -d)" && \
	gpg --keyserver $GPGKEYSERVER --recv-keys $GOSU_GPGFINGERPRINT && \
	gpg --batch --verify /usr/local/bin/gosu.asc /usr/local/bin/gosu && \
	rm -rf "$GNUPGHOME" /usr/local/bin/gosu.asc && \
	chmod +x /usr/local/bin/gosu && \
	gosu nobody true

# grab tini for signal processing and zombie killing
RUN set -x && \
	wget -O /usr/local/bin/tini "https://github.com/krallin/tini/releases/download/$TINI_VERSION/tini" && \
	wget -O /usr/local/bin/tini.asc "https://github.com/krallin/tini/releases/download/$TINI_VERSION/tini.asc" && \
	export GNUPGHOME="$(mktemp -d)" && \
	gpg --keyserver $GPGKEYSERVER --recv-keys $TINI_GPGFINGERPRINT && \
	gpg --batch --verify /usr/local/bin/tini.asc /usr/local/bin/tini && \
	rm -rf "$GNUPGHOME" /usr/local/bin/tini.asc && \
	chmod +x /usr/local/bin/tini && \
	tini -h

# elasticsearch
RUN set -ex; \
# https://artifacts.elastic.co/GPG-KEY-elasticsearch
	export GNUPGHOME="$(mktemp -d)" && \
	gpg --keyserver $GPGKEYSERVER --recv-keys $ES_GPGFINGERPRINT && \
  gpg -a --export $ES_GPGFINGERPRINT | apt-key add - && \
	rm -rf "$GNUPGHOME" && \
	apt-key list

# https://www.elastic.co/guide/en/kibana/5.6/deb.html
RUN echo 'deb https://artifacts.elastic.co/packages/5.x/apt stable main' > /etc/apt/sources.list.d/kibana.list

RUN set -x && \
	apt-get update && \
	apt-get install -y --no-install-recommends kibana=$KIBANA_VERSION && \
	rm -rf /var/lib/apt/lists/* && \
# the default "server.host" is "localhost" in 5+
	sed -ri "s!^(\#\s*)?(server\.host:).*!\2 '0.0.0.0'!" /etc/kibana/kibana.yml && \
	grep -q "^server\.host: '0.0.0.0'\$" /etc/kibana/kibana.yml && \
# ensure the default configuration is useful when using --link
	sed -ri "s!^(\#\s*)?(elasticsearch\.url:).*!\2 'http://elasticsearch:9200'!" /etc/kibana/kibana.yml && \
	grep -q "^elasticsearch\.url: 'http://elasticsearch:9200'\$" /etc/kibana/kibana.yml && \
	apt-get remove --purge --auto-remove -y -qq $APT_BLD_DEPS perl

ENV PATH /usr/share/kibana/bin:$PATH

COPY bin/docker-entrypoint.sh /
RUN chmod +x /docker-entrypoint.sh

EXPOSE 5601
ENTRYPOINT ["/docker-entrypoint.sh"]
CMD ["kibana"]
