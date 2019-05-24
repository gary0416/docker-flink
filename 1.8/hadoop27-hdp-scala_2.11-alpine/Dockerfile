# flink docker for k8s
# hadoop version is 2.7.3.2.6.3.0-235(hdp 2.6.3)

FROM openjdk:8-jre-alpine

# Install dependencies, bash and su-exec for easy step-down from root
RUN apk add --no-cache bash libc6-compat snappy 'su-exec>=0.2'

# Configure Flink version
ENV FLINK_VERSION=1.8.0 \
    HADOOP_SCALA_VARIANT=scala_2.11

# Prepare environment
ENV FLINK_HOME=/opt/flink
ENV PATH=$FLINK_HOME/bin:$PATH
RUN addgroup -S -g 9999 flink && \
    adduser -D -S -H -u 9999 -G flink -h $FLINK_HOME flink
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
