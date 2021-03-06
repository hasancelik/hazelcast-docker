FROM alpine:3.13.5

# Versions of Hazelcast and Hazelcast plugins
ARG HZ_VERSION=5.0-SNAPSHOT
# Variant - empty for full, "slim" for slim
ARG HZ_VARIANT=""

# Build constants
ARG HZ_HOME="/opt/hazelcast"

# Runtime variables
ENV HZ_HOME="${HZ_HOME}" \
    CLASSPATH_DEFAULT="${HZ_HOME}/*:${HZ_HOME}/lib/*" \
    JAVA_OPTS_DEFAULT="-Djava.net.preferIPv4Stack=true -XX:MaxRAMPercentage=80.0 -XX:MaxGCPauseMillis=5" \
    PROMETHEUS_PORT="" \
    PROMETHEUS_CONFIG="${HZ_HOME}/config/jmx_agent_config.yaml" \
    LOGGING_LEVEL="" \
    CLASSPATH="" \
    JAVA_OPTS="" \
    PATH=${HZ_HOME}/bin:$PATH

# Expose port
EXPOSE 5701

COPY *.jar get-hz-dist-zip.sh ${HZ_HOME}/

# Install
RUN echo "Installing new APK packages" \
    && apk add --no-cache openjdk11-jre-headless bash curl libxml2-utils zip unzip \
    && echo "Downloading Hazelcast${HZ_VARIANT} distribution zip..." \
    && HAZELCAST_ZIP_URL=$(${HZ_HOME}/get-hz-dist-zip.sh) \
    && curl -sf -L ${HAZELCAST_ZIP_URL} --output ${HZ_HOME}/hazelcast-distribution.zip \
    && unzip -qq ${HZ_HOME}/hazelcast-distribution.zip 'hazelcast-*/**' -d ${HZ_HOME}/tmp/ \
    && mv ${HZ_HOME}/tmp/*/* ${HZ_HOME}/ \
    && echo "Setting Pardot ID to 'docker'" \
    && echo 'hazelcastDownloadId=docker' > "${HZ_HOME}/hazelcast-download.properties" \
    && zip -u ${HZ_HOME}/lib/hazelcast-${HZ_VERSION}.jar ${HZ_HOME}/hazelcast-download.properties \
    && echo "Granting read permission to ${HZ_HOME}" \
    && chmod -R +r ${HZ_HOME} \
    && echo "Cleaning APK packages and redundant files/folders" \
    && apk del libxml2-utils curl zip unzip \
    && rm -rf /var/cache/apk/* ${HZ_HOME}/get-hz-dist-zip.sh ${HZ_HOME}/hazelcast-distribution.zip ${HZ_HOME}/tmp

COPY log4j2.properties jmx_agent_config.yaml ${HZ_HOME}/config/

WORKDIR ${HZ_HOME}

RUN addgroup -S hazelcast && adduser -S hazelcast -G hazelcast
USER hazelcast

# Start Hazelcast server
CMD ["hz-start"]
