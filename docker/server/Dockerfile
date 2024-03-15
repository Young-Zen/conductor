#
# conductor:server - Build Conductor Server
#
# ===========================================================================================================
# 0. Builder stage
# ===========================================================================================================
FROM openjdk:17-bullseye AS builder

LABEL maintainer="Orkes OSS <oss@orkes.io>"

# Copy the project directly onto the image
COPY . /conductor
WORKDIR /conductor

# Build the server on run
RUN ./gradlew build -x test
WORKDIR /server/build/libs
RUN ls -ltr

# ===========================================================================================================
# 1. Bin stage
# ===========================================================================================================
FROM openjdk:17-jdk-slim

LABEL maintainer="Orkes OSS <oss@orkes.io>"

# Make app folders
RUN mkdir -p /app/config /app/logs /app/libs \
    && sed -i 's/deb.debian.org/mirrors.ustc.edu.cn/g' /etc/apt/sources.list \
    && apt-get update \
    && apt install -y --no-install-recommends procps inetutils-ping net-tools curl \
    && rm -rf /var/lib/apt/lists/*

# Copy the compiled output to new image
COPY docker/server/bin /app
COPY docker/server/config /app/config
COPY --from=builder /conductor/server/build/libs/*boot*.jar /app/libs/conductor-server.jar

# Copy the files for the server into the app folders
RUN chmod +x /app/startup.sh

HEALTHCHECK --interval=60s --timeout=30s --retries=10 CMD curl -I -XGET http://localhost:8080/health || exit 1

CMD [ "/app/startup.sh" ]
ENTRYPOINT [ "/bin/sh"]
