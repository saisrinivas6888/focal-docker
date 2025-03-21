### 1 Build the Frontend (Webapp) ###
FROM node:16.3.0 AS nodebuild

WORKDIR /webapp
COPY webapp/ /webapp

RUN npm install --no-optional && \
    npm run pack

### 2 Build the Backend (Go Server) ###
FROM golang:1.21 AS gobuild

WORKDIR /go/src/focalboard
COPY . /go/src/focalboard

ARG TARGETOS
ARG TARGETARCH

RUN EXCLUDE_PLUGIN=true EXCLUDE_SERVER=true EXCLUDE_ENTERPRISE=true make server-docker os=${TARGETOS} arch=${TARGETARCH}

### 3 Final Image (Debian Slim) ###
FROM ubuntu:22.04 

WORKDIR /opt/focalboard
RUN mkdir -p /opt/focalboard/data/files && \
    chown -R nobody:nogroup /opt/focalboard

COPY --from=nodebuild --chown=nobody:nogroup /webapp/pack pack/
COPY --from=gobuild --chown=nobody:nogroup /go/src/focalboard/bin/docker/focalboard-server bin/
COPY --from=gobuild --chown=nobody:nogroup /go/src/focalboard/docker/server-config.json config.json

USER nobody

# ✅ Updated port 7000
EXPOSE 7000/tcp

VOLUME /opt/focalboard/data
CMD ["/opt/focalboard/bin/focalboard-server"]
