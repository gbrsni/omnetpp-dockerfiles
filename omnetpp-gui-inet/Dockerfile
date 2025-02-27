#
# Copyright (C) OpenSim Kft.
#
# SPDX-License-Identifier: MIT
#

FROM omnetpp-base:u24.04-1 AS base

LABEL maintainer="Rudolf Hornig <rudi@omnetpp.org>"

ENV INET_VERSION=4.4.2

RUN apt-get update -y && apt-get install -y --no-install-recommends qtbase5-dev qtchooser\
                          qt5-qmake qtbase5-dev-tools libqt5svg5 libqt5opengl5-dev qtwayland5 libwebkit2gtk-4.1-0

# first stage - build OMNeT++ with GUI
FROM base AS builder

ARG VERSION
WORKDIR /root
SHELL ["/bin/bash", "-c"]
RUN wget https://github.com/omnetpp/omnetpp/releases/download/omnetpp-$VERSION/omnetpp-$VERSION-linux-x86_64.tgz \
         --referer=https://omnetpp.org/ -O omnetpp-core.tgz --progress=dot:giga && \
         tar xf omnetpp-core.tgz && rm omnetpp-core.tgz

RUN mv omnetpp-6.0.3 omnetpp
WORKDIR /root/omnetpp
ENV PATH=/root/omnetpp/bin:$PATH
# remove unused files and build
COPY requirements.txt python/requirements.txt
# Needed since 6.0.3 refuses to build if it can't create desktop shortcuts
RUN mkdir /usr/share/desktop-directories/
RUN python3 -m venv .venv --upgrade-deps --clear --prompt "omnetpp/.venv" && \
    source .venv/bin/activate && \
    python3 -m pip install -r python/requirements.txt && \
    # python3 -m pip install numpy pandas matplotlib scipy seaborn posix_ipc && \
    source ./setenv && \
    ./configure WITH_LIBXML=yes WITH_OSG=no WITH_OSGEARTH=no && \
    make -j $(nproc) MODE=debug && \
    rm -r doc out test samples config.log config.status

# second stage - copy only the final binaries (to get rid of the 'out' folder and reduce the image size)
FROM base

ARG VERSION
ENV OPP_VER=$VERSION
RUN mkdir -p /root/omnetpp
WORKDIR /root/omnetpp
COPY --from=builder /root/omnetpp/ .
ENV PATH=/root/omnetpp/bin:$PATH
RUN chmod 777 /root/ && \
    mkdir -p /root/models && \
    chmod 777 /root/models && \
    touch ide/error.log && chmod 666 ide/error.log && \
    rm -f bin/omnest && \
    mv bin/opp_ide bin/opp_ide.bak && \
    sed 's!-Dosgi.instance.area.default=$IDEDIR/../samples!-Duser.home=/root/models -Dosgi.instance.area=@user.home!' bin/opp_ide.bak >bin/opp_ide && \
    rm bin/opp_ide.bak && chmod +x bin/opp_ide
WORKDIR /root/models
# Install inet (or maybe just download and leave building to the IDE?)
RUN wget https://github.com/inet-framework/inet/releases/download/v${INET_VERSION}/inet-${INET_VERSION}-src.tgz && \
    tar xfz inet-${INET_VERSION}-src.tgz && rm inet-${INET_VERSION}-src.tgz

# WORKDIR /root/models/inet4.4
# SHELL [ "/bin/bash", "-c", "source setenv" ]
# RUN make makefiles && \
#     make

# SHELL ["/bin/bash", "-c"]
RUN echo 'PS1="omnetpp-gui-$OPP_VER:\w\$ "' >> /root/.bashrc && chmod +x /root/.bashrc && \
    touch /root/.hushlogin
ENV HOME=/root/
RUN chmod -R a+rwX /root/
ENV XDG_RUNTIME_DIR=/tmp/xdg-runtime-root
COPY --chmod=0555 ./entrypoint.sh /entrypoint.sh
ENTRYPOINT ["/entrypoint.sh"]
CMD ["/bin/bash"]