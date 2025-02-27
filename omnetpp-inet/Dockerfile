#
# Copyright (C) OpenSim Kft.
#
# SPDX-License-Identifier: MIT
#

FROM omnetpp-base:u24.04-1 AS base
LABEL maintainer="Rudolf Hornig <rudi@omnetpp.org>"

ENV INET_VERSION=4.4.2

# first stage - build omnet
FROM base AS builder
ARG VERSION
SHELL ["/bin/bash", "-c"]
WORKDIR /root
RUN wget https://github.com/omnetpp/omnetpp/releases/download/omnetpp-$VERSION/omnetpp-$VERSION-core.tgz \
         --referer=https://omnetpp.org/ -O omnetpp-core.tgz --progress=dot:giga && \
         tar xf omnetpp-core.tgz && rm omnetpp-core.tgz
RUN mv omnetpp-6.0.3 omnetpp
WORKDIR /root/omnetpp
# remove unused files and build
RUN python3 -m venv .venv --upgrade-deps --clear --prompt "omnetpp/.venv" && \
    source .venv/bin/activate && \
    # python3 -m pip install -r python/requirements.txt && \
    python3 -m pip install opp-env && \
    source ./setenv && \
    ./configure WITH_LIBXML=yes WITH_QTENV=no WITH_OSG=no WITH_OSGEARTH=no && \
    make -j $(nproc) MODE=release base && \
    rm -r doc out test samples misc config.log config.status

# second stage - copy only the final binaries (to get rid of the 'out' folder and reduce the image size)
FROM base
ARG VERSION
ENV OPP_VER=$VERSION
RUN mkdir -p /root/omnetpp
WORKDIR /root/omnetpp
COPY --from=builder /root/omnetpp/ .
RUN chmod 775 /root/ && \
    mkdir -p /root/models && \
    chmod 775 /root/models
WORKDIR /root/models
# Install inet
RUN wget https://github.com/inet-framework/inet/releases/download/v${INET_VERSION}/inet-${INET_VERSION}-src.tgz && \
    tar xfz inet-${INET_VERSION}-src.tgz && rm inet-${INET_VERSION}-src.tgz

WORKDIR /root/models/inet4.4
SHELL [ "/bin/bash", "-c", "source setenv" ]
RUN make makefiles && \
    make

SHELL ["/bin/bash", "-c"]
ENV HOME=/root/
RUN chmod -R a+rwX /root/
RUN echo 'PS1="omnetpp-$OPP_VER:\w\$ "' >> /root/.bashrc && \
    echo '[ -f "$HOME/omnetpp/setenv" ] && source "$HOME/omnetpp/setenv" -f' >> /root/.bashrc && \
    touch /root/.hushlogin
# SHELL ["/bin/bash", "--init-file", "/root/.bashrc"]
# RUN opp_env install openflow-latest
CMD ["/bin/bash", "--init-file", "/root/.bashrc"]
