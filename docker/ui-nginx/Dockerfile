#
# conductor:ui - Netflix Conductor UI
#
# ===========================================================================================================
# 0. Builder stage
# ===========================================================================================================
FROM node:20-alpine AS builder
LABEL maintainer="Netflix OSS <conductor@netflix.com>"

# Install the required packages for the node build
# to run on alpine
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.tuna.tsinghua.edu.cn/g' /etc/apk/repositories && apk update && apk add --no-cache python3 py3-pip make g++

# Copies package.json to Docker environment in a separate layer as a performance optimization
COPY ./ui/package.json /conductor/ui/
WORKDIR /conductor/ui

# Installs all node packages. Cached unless package.json changes
RUN yarn install && mkdir -p public && cp -r node_modules/monaco-editor public/

# Copies everything else over to Docker environment
# node_modules excluded in .dockerignore.
COPY ./ui .

# Include monaco sources into bundle (instead of using CDN)
ENV REACT_APP_MONACO_EDITOR_USING_CDN=false

# Build the client
RUN yarn build

# ===========================================================================================================
# 1. Bin stage
# ===========================================================================================================

FROM nginx:alpine
LABEL maintainer="Netflix OSS <conductor@netflix.com>"

# Copy compiled UI assets to nginx www directory
WORKDIR /usr/share/nginx/html
RUN rm -rf ./*
COPY --from=builder /conductor/ui/build .
COPY docker/ui-nginx/nginx/nginx.conf /etc/nginx/conf.d/default.conf
COPY docker/ui-nginx/nginx/.conductor_passwd /etc/nginx/conf.d/.conductor_passwd

EXPOSE 5000

CMD [ "nginx", "-g", "daemon off;"]