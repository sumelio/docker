FROM ubuntu:12.04

MAINTAINER Couchbase Docker Team <docker@couchbase.com>

# Install dependencies:
#  runit: for container process management
#  wget: for downloading .deb
#  python-httplib2: used by CLI tools
#  chrpath: for fixing curl, below
# Additional dependencies for system commands used by cbcollect_info:
#  lsof: lsof
#  lshw: lshw
#  sysstat: iostat, sar, mpstat
#  net-tools: ifconfig, arp, netstat
#  numactl: numactl
RUN apt-get update && \
    apt-get install -yq runit wget python-httplib2 chrpath \
    lsof lshw sysstat net-tools numactl  && \
    apt-get autoremove && apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

ARG CB_VERSION=3.0.3
ARG CB_RELEASE_URL=https://packages.couchbase.com/releases
ARG CB_PACKAGE=couchbase-server-enterprise_3.0.3-ubuntu12.04_amd64.deb
ARG CB_SHA256=13e925fa8b806aecd09751bdb7be1f8cfa188ad894e266108e8c44785c08e474

ENV PATH=$PATH:/opt/couchbase/bin:/opt/couchbase/bin/tools:/opt/couchbase/bin/install

# Create Couchbase user with UID 1000 (necessary to match default
# boot2docker UID)
RUN groupadd -g 1000 couchbase && useradd couchbase -u 1000 -g couchbase -M

# Install couchbase
RUN export INSTALL_DONT_START_SERVER=1 && \
    wget -N $CB_RELEASE_URL/$CB_VERSION/$CB_PACKAGE && \
    echo "$CB_SHA256  $CB_PACKAGE" | sha256sum -c - && \
    dpkg -i ./$CB_PACKAGE && rm -f ./$CB_PACKAGE

# Add runit script for couchbase-server
COPY scripts/run /etc/service/couchbase-server/run
RUN chown -R couchbase:couchbase /etc/service

# Add dummy script for commands invoked by cbcollect_info that
# make no sense in a Docker container
COPY scripts/dummy.sh /usr/local/bin/
RUN ln -s dummy.sh /usr/local/bin/iptables-save && \
    ln -s dummy.sh /usr/local/bin/lvdisplay && \
    ln -s dummy.sh /usr/local/bin/vgdisplay && \
    ln -s dummy.sh /usr/local/bin/pvdisplay

# Fix curl RPATH
RUN chrpath -r '$ORIGIN/../lib' /opt/couchbase/bin/curl

# Add bootstrap script
COPY scripts/entrypoint.sh /
ENTRYPOINT ["/entrypoint.sh"]
CMD ["couchbase-server"]

# 8091: Couchbase Web console, REST/HTTP interface
# 8092: Views, queries, XDCR
# 8093: Query services (4.0+)
# 8094: Full-text Search (4.5+)
# 11207: Smart client library data node access (SSL)
# 11210: Smart client library/moxi data node access
# 11211: Legacy non-smart client library data node access
# 18091: Couchbase Web console, REST/HTTP interface (SSL)
# 18092: Views, query, XDCR (SSL)
# 18093: Query services (SSL) (4.0+)
# 18094: Full-text Search (SSL) (4.5+)
EXPOSE 8091 8092 8093 8094 11207 11210 11211 18091 18092 18093 18094
VOLUME /opt/couchbase/var

