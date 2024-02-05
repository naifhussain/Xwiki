# Use a smaller base image for Java applications
FROM adoptopenjdk:11-jre-hotspot

# Labels and other metadata
LABEL org.opencontainers.image.authors='XWiki Development Team <committers@xwiki.org>'
LABEL org.opencontainers.image.url='https://hub.docker.com/_/xwiki'
LABEL org.opencontainers.image.documentation='https://hub.docker.com/_/xwiki'
LABEL org.opencontainers.image.source='https://github.com/xwiki/xwiki-docker.git'
LABEL org.opencontainers.image.vendor='xwiki.org'
LABEL org.opencontainers.image.licenses='LGPL-2.1'

# Install necessary tools and LibreOffice
RUN apt-get update && \
    apt-get --no-install-recommends -y install \
        curl \
        libreoffice \
        unzip \
        procps && \
    rm -rf /var/lib/apt/lists/*

# XWiki environment setup
ENV XWIKI_VERSION="16.0.0"
ENV XWIKI_URL_PREFIX "https://maven.xwiki.org/releases/org/xwiki/platform/xwiki-platform-distribution-war/${XWIKI_VERSION}"
ENV XWIKI_DOWNLOAD_SHA256 955c175d0ac0e7039eeafd8569d87ad8a7967092ad8d018decb80a42a7eb941f

# Remove existing webapps and create necessary directories
RUN rm -rf /usr/local/tomcat/webapps/* && \
    mkdir -p /usr/local/tomcat/temp && \
    mkdir -p /usr/local/xwiki/data

# Download and extract XWiki distribution
RUN curl -fSL "${XWIKI_URL_PREFIX}/xwiki-platform-distribution-war-${XWIKI_VERSION}.war" -o xwiki.war && \
    echo "$XWIKI_DOWNLOAD_SHA256 xwiki.war" | sha256sum -c - && \
    unzip -d /usr/local/tomcat/webapps/ROOT xwiki.war && \
    rm -f xwiki.war

# Download and copy MySQL JDBC driver
ENV MYSQL_JDBC_VERSION="8.3.0"
ENV MYSQL_JDBC_SHA256="94e7fa815370cdcefed915db7f53f88445fac110f8c3818392b992ec9ee6d295"
ENV MYSQL_JDBC_PREFIX="https://repo1.maven.org/maven2/com/mysql/mysql-connector-j/${MYSQL_JDBC_VERSION}"
ENV MYSQL_JDBC_ARTIFACT="mysql-connector-j-${MYSQL_JDBC_VERSION}.jar"
ENV MYSQL_JDBC_TARGET="/usr/local/tomcat/webapps/ROOT/WEB-INF/lib/${MYSQL_JDBC_ARTIFACT}"
RUN curl -fSL "${MYSQL_JDBC_PREFIX}/${MYSQL_JDBC_ARTIFACT}" -o $MYSQL_JDBC_TARGET && \
    echo "$MYSQL_JDBC_SHA256 $MYSQL_JDBC_TARGET" | sha256sum -c -

# Configure Tomcat environment
COPY tomcat/setenv.sh /usr/local/tomcat/bin/

# Setup the XWiki Hibernate configuration
COPY xwiki/hibernate.cfg.xml /usr/local/tomcat/webapps/ROOT/WEB-INF/hibernate.cfg.xml

# Set a specific distribution id in XWiki for this docker packaging
RUN sed -i 's/<id>org.xwiki.platform:xwiki-platform-distribution-war/<id>org.xwiki.platform:xwiki-platform-distribution-docker/' \
    /usr/local/tomcat/webapps/ROOT/META-INF/extension.xed

# Add scripts required to make changes to XWiki configuration files at execution time
COPY xwiki/docker-entrypoint.sh /usr/local/bin/docker-entrypoint.sh

# Make the XWiki directory persist on the host
VOLUME /usr/local/xwiki

# Set port to 80 to comply with Azure Web Apps requirements
ENV PORT 80

# Expose the port that your application will be listening on
EXPOSE 80

# Health check configuration (example, adjust as needed)
HEALTHCHECK --interval=30s --timeout=3s \
  CMD curl -f http://localhost/ || exit 1

# Entrypoint and CMD configuration
ENTRYPOINT ["docker-entrypoint.sh"]
CMD ["catalina.sh", "run"]
