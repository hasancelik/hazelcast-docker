FROM alpine:3.13.2

# Versions of Hazelcast and Hazelcast plugins
ARG HZ_VERSION=4.1.2
ARG CACHE_API_VERSION=1.1.1
ARG JMX_PROMETHEUS_AGENT_VERSION=0.14.0
ARG LOG4J2_VERSION=2.13.3

# Build constants
ARG HZ_HOME="/opt/hazelcast"

# JARs to download
ARG HAZELCAST_ALL_URL="https://repository.hazelcast.com/release/com/hazelcast/hazelcast-enterprise-all/${HZ_VERSION}/hazelcast-enterprise-all-${HZ_VERSION}.jar"
ARG CACHE_API_URL="https://repo1.maven.org/maven2/javax/cache/cache-api/${CACHE_API_VERSION}/cache-api-${CACHE_API_VERSION}.jar"
ARG PROMETHEUS_AGENT_URL="https://repo1.maven.org/maven2/io/prometheus/jmx/jmx_prometheus_javaagent/${JMX_PROMETHEUS_AGENT_VERSION}/jmx_prometheus_javaagent-${JMX_PROMETHEUS_AGENT_VERSION}.jar"
ARG LOG4J2_URLS="https://repo1.maven.org/maven2/org/apache/logging/log4j/log4j-core/${LOG4J2_VERSION}/log4j-core-${LOG4J2_VERSION}.jar https://repo1.maven.org/maven2/org/apache/logging/log4j/log4j-api/${LOG4J2_VERSION}/log4j-api-${LOG4J2_VERSION}.jar"

# Runtime variables
ENV HZ_HOME="${HZ_HOME}" \
    CLASSPATH_DEFAULT="${HZ_HOME}/*:${HZ_HOME}/lib/*" \
    JAVA_OPTS_DEFAULT="-Djava.net.preferIPv4Stack=true -Dhazelcast.logging.type=log4j2 -Dlog4j.configurationFile=${HZ_HOME}/log4j2.properties -XX:MaxRAMPercentage=80.0 -XX:MaxGCPauseMillis=5 --add-modules java.se --add-exports java.base/jdk.internal.ref=ALL-UNNAMED --add-opens java.base/java.lang=ALL-UNNAMED --add-opens java.base/java.nio=ALL-UNNAMED --add-opens java.base/sun.nio.ch=ALL-UNNAMED --add-opens java.management/sun.management=ALL-UNNAMED --add-opens jdk.management/com.sun.management.internal=ALL-UNNAMED" \
    PROMETHEUS_PORT="" \
    PROMETHEUS_CONFIG="${HZ_HOME}/jmx_agent_config.yaml" \
    LOGGING_LEVEL="" \
    CLASSPATH="" \
    JAVA_OPTS=""

# Expose port
EXPOSE 5701

COPY *.sh *.yaml *.jar *.properties ${HZ_HOME}/

# Install
RUN echo "Installing new APK packages" \
    && apk add --no-cache openjdk11-jre-headless bash curl perl-xml-xpath zip \
    && echo "Downloading Hazelcast and related JARs" \
    && mkdir "${HZ_HOME}/lib" \
    && cd "${HZ_HOME}/lib" \
    && for JAR_URL in ${HAZELCAST_ALL_URL} ${CACHE_API_URL} ${PROMETHEUS_AGENT_URL} ${LOG4J2_URLS}; do curl -sf -O -L ${JAR_URL} || exit $?; done \
    && mv jmx_prometheus_javaagent-*.jar jmx_prometheus_javaagent.jar \
    && echo "Setting Pardot ID to 'docker'" \
    && echo 'hazelcastDownloadId=docker' > "hazelcast-download.properties" \
    && zip -u hazelcast-all-*.jar hazelcast-download.properties \
    && echo "Granting read permission to ${HZ_HOME}" \
    && chmod -R +r ${HZ_HOME} \
    && echo "Cleaning APK packages" \
    && apk del perl-xml-xpath curl zip \
    && rm -rf /var/cache/apk/* ${HZ_HOME}/get-hz-all-url.sh

WORKDIR ${HZ_HOME}

RUN addgroup -S hazelcast && adduser -S hazelcast -G hazelcast
USER hazelcast

# Start Hazelcast server
CMD ["/opt/hazelcast/start-hazelcast.sh"]