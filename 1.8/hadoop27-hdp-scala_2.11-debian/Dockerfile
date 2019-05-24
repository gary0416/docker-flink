# flink docker for k8s
# hadoop version is 2.7.3.2.6.3.0-235(hdp 2.6.3)

FROM openjdk:8-jre

# Install dependencies
RUN set -ex; \
  apt-get update; \
  apt-get -y install libsnappy1v5; \
  rm -rf /var/lib/apt/lists/*

# Grab gosu for easy step-down from root
ENV GOSU_VERSION 1.11
RUN set -ex; \
  wget -nv -O /usr/local/bin/gosu "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$(dpkg --print-architecture)"; \
  wget -nv -O /usr/local/bin/gosu.asc "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$(dpkg --print-architecture).asc"; \
  export GNUPGHOME="$(mktemp -d)"; \
  for server in ha.pool.sks-keyservers.net $(shuf -e \
                          hkp://p80.pool.sks-keyservers.net:80 \
                          keyserver.ubuntu.com \
                          hkp://keyserver.ubuntu.com:80 \
                          pgp.mit.edu) ; do \
      gpg --batch --keyserver "$server" --recv-keys B42F6819007F00F88E364FD4036A9C25BF357DD4 && break || : ; \
  done && \
  gpg --batch --verify /usr/local/bin/gosu.asc /usr/local/bin/gosu; \
  gpgconf --kill all; \
  rm -rf "$GNUPGHOME" /usr/local/bin/gosu.asc; \
  chmod +x /usr/local/bin/gosu; \
  gosu nobody true

# Configure Flink version
ENV FLINK_VERSION=1.8.0 \
    HADOOP_SCALA_VARIANT=scala_2.11

# Prepare environment
ENV FLINK_HOME=/opt/flink
ENV PATH=$FLINK_HOME/bin:$PATH
RUN groupadd --system --gid=9999 flink && \
    useradd --system --home-dir $FLINK_HOME --uid=9999 --gid=flink flink
WORKDIR $FLINK_HOME

# build flink-1.8.0-hdp-hadoop27-scala_2.11.tar.gz:
# prepare git,maven,npm env. maven: <mirrorOf>*,!confluent</mirrorOf>,and add http://packages.confluent.io/maven/
# do not -b release-1.8.0,version in pom is still 1.9 snapshot
# $ git clone https://github.com/apache/flink.git && cd flink && checkout tags/release-1.8.0
# -Pinclude-hadoop can add flink-shaded-hadoop2-uber-1.8.0-2.7.3.2.6.3.0-235.jar to lib/
# $ mvn clean install -DskipTests -Pvendor-repos -Pinclude-hadoop -Dhadoop.version=2.7.3.2.6.3.0-235 -Dzookeeper.version=3.4.6.2.6.3.0-235
# $ cd flink-dist/target/flink-1.8.0-bin/flink-1.8.0 && tar czvf ../flink-1.8.0-hdp-hadoop27-scala_2.11.tar.gz *
# move tgz and do cleanup
COPY flink-1.8.0-hdp-hadoop27-scala_2.11.tar.gz .
RUN tar xzf flink-1.8.0-hdp-hadoop27-scala_2.11.tar.gz && rm flink-1.8.0-hdp-hadoop27-scala_2.11.tar.gz && chown -R flink:flink .

# Configure container
COPY docker-entrypoint.sh /
ENTRYPOINT ["/docker-entrypoint.sh"]
EXPOSE 6123 8081
CMD ["help"]
