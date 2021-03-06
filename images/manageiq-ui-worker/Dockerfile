ARG FROM_REPO=manageiq
ARG FROM_TAG=latest

FROM ${FROM_REPO}/manageiq-webserver-worker:${FROM_TAG}
MAINTAINER ManageIQ https://manageiq.org

ARG NPM_REGISTRY_OVERRIDE

LABEL name="manageiq-ui-worker" \
      summary="ManageIQ user interface worker image"

# Install httpd and remove the existing httpd config from manageiq-appliance
RUN yum -y install httpd --setopt=tsflags=nodocs && \
    rm -f /etc/httpd/conf.d/* && \
    yum clean all

COPY container-assets/npm_registry /tmp/npm_registry

# Compile assets for ManageIQ UI
RUN source /etc/default/evm && \
    export RAILS_USE_MEMORY_STORE="true" && \
    if [ ! -z "${NPM_REGISTRY_OVERRIDE}" ]; then \
      /tmp/npm_registry/npm_registry_setup.sh; \
    fi && \
    npm install -g yarn && \
    if [ ! -z "${NPM_REGISTRY_OVERRIDE}" ]; then \
      /tmp/npm_registry/yarn_registry_setup.sh; \
    fi && \
    rake update:ui && \
    RAILS_ENV=production rake evm:compile_assets && \
    rake evm:compile_sti_loader && \
    bin/rails log:clear tmp:clear && \
    npm cache clean --force && \
    rm -rvf ${APP_ROOT}/tmp/cache/assets && \
    rm -vf ${APP_ROOT}/log/*.log

# Build service UI
RUN source /etc/default/evm && \
    cd ${SUI_ROOT} && \
    yarn install && \
    yarn run available-languages && \
    yarn run build

RUN source /etc/default/evm && \
    yarn cache clean && \
    cd ${APP_ROOT} && \
    if [ ! -z "${NPM_REGISTRY_OVERRIDE}" ]; then \
      /tmp/npm_registry/npm_yarn_registry_cleanup.sh; \
    fi && \
    rm -rf /tmp/npm_registry

COPY container-assets/manageiq-http.conf /etc/httpd/conf.d
