FROM ubuntu:16.04

############################################################################
# Information
LABEL maintainer="TerraMA² Team <terrama2-team@dpi.inpe.br>" \
      version="4.0.6"

############################################################################
# Environment Variables
ENV TERRAMA2_USER=terrama2
ENV TERRAMA2_TAG=4.0.6
ENV DEV_DIR=/home/${TERRAMA2_USER}/mydevel
ENV TERRAMA2_INSTALL_PATH=/opt/terrama2/${TERRAMA2_TAG}
ENV TERRAMA2_CODEBASE_PATH=${DEV_DIR}/terrama2/codebase
ENV TERRAMA2_3RDPARTY_PATH=${DEV_DIR}/terrama2/3rdparty
ENV PATH="$PATH:${TERRAMA2_INSTALL_PATH}/bin"

RUN apt-get update && \
    apt-get install -y cmake \
                       curl \
                       libquazip-dev \
                       git \
                       unzip \
                       sudo \
                       screen \
                       openssh-server \
                       # VMime Dependencies
                       doxygen \
                       # Process Control
                       supervisor \
                       # Locales
                       locales \
                       # for timezone
                       software-properties-common \
                       apt-utils \
                       tzdata && \
    rm -rf /var/lib/apt/lists/* && \
    # Creating USER
    groupadd $TERRAMA2_USER && \
    useradd $TERRAMA2_USER -s /bin/bash -m -g $TERRAMA2_USER && \
    # Defining User credential to connect
    echo $TERRAMA2_USER:$TERRAMA2_USER | chpasswd && \
    mkdir -p $TERRAMA2_3RDPARTY_PATH $TERRAMA2_CODEBASE_PATH && \
    chown -R $TERRAMA2_USER:$TERRAMA2_USER /home/$TERRAMA2_USER && \
    chmod g-w /home/$TERRAMA2_USER && \
    ssh-keygen -t rsa -b 4096 -C "terrama2-team@dpi.inpe.br" -N "" -f $HOME/.ssh/id_rsa && \
    mkdir /home/$TERRAMA2_USER/.ssh && \
    cat /root/.ssh/id_rsa.pub > /home/$TERRAMA2_USER/.ssh/authorized_keys && \
    cat /root/.ssh/id_rsa.pub > /root/.ssh/authorized_keys && \
    chown -R ${TERRAMA2_USER}:${TERRAMA2_USER} /home/$TERRAMA2_USER/.ssh && \
    chmod -R 700 /home/$TERRAMA2_USER/.ssh


############################################################################
# Dependencies

RUN cd ${TERRAMA2_3RDPARTY_PATH} && \
    curl -o terralib-5.3.1-ubuntu-16.04.tar.gz -L http://www.dpi.inpe.br/terralib5/download/download.php?FileName=terralib-5.3.1-ubuntu-16.04.tar.gz && \
    tar zxf terralib-5.3.1-ubuntu-16.04.tar.gz && \
    ./install.sh && \
    rm -rf * && \
    curl -sL https://deb.nodesource.com/setup_8.x | sudo -E bash - && \
    apt-get install -y nodejs && \
    cd ${TERRAMA2_CODEBASE_PATH} && \
    curl -O http://www.dpi.inpe.br/jenkins-data/terrama2/3rdparty/${TERRAMA2_TAG}/terrama2-vmime-${TERRAMA2_TAG}_0.9.2-ubuntu16.04_amd64.deb && \
    apt-get install -y libgsasl7 && \
    dpkg -i terrama2-vmime-${TERRAMA2_TAG}_0.9.2-ubuntu16.04_amd64.deb && \
    apt-get install -f -y && \
    rm -rf terrama2-vmime-${TERRAMA2_TAG}_0.9.2-ubuntu16.04_amd64.deb* v* ${TERRAMA2_CODEBASE_PATH} && \
    rm -rf /var/lib/apt/lists/*

# ############################################################################
# # TerraMA2
COPY files/install_terrama2.sh .

RUN ./install_terrama2.sh && rm install_terrama2.sh

COPY files/start_terrama2_service.sh /usr/local/bin

RUN mkdir -p /data && chown -R ${TERRAMA2_USER}:${TERRAMA2_USER} /data && \
    sed -i -e 's/# en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/' /etc/locale.gen && \
    dpkg-reconfigure --frontend=noninteractive locales && \
    update-locale LANG=en_US.UTF-8 && \
    echo "export LANG=en_US.UTF-8\n\
export LANGUAGE=en_US.UTF-8\n\
export LC_CTYPE=\"en_US.UTF-8\"\n\
export LC_NUMERIC=\"en_US.UTF-8\"\n\
export LC_TIME=\"en_US.UTF-8\"\n\
export LC_COLLATE=\"en_US.UTF-8\"\n\
export LC_MONETARY=\"en_US.UTF-8\"\n\
export LC_MESSAGES=\"en_US.UTF-8\"\n\
export LC_PAPER=\"en_US.UTF-8\"\n\
export LC_NAME=\"en_US.UTF-8\"\n\
export LC_ADDRESS=\"en_US.UTF-8\"\n\
export LC_TELEPHONE=\"en_US.UTF-8\"\n\
export LC_MEASUREMENT=\"en_US.UTF-8\"\n\
export LC_IDENTIFICATION=\"en_US.UTF-8\"\n\
export LC_ALL=en_US.UTF-8" | tee -a /etc/bash.bashrc && \
    # Configure ulimits
    sed -i "\$i${TERRAMA2_USER}        hard    nofile  1048576\n${TERRAMA2_USER}        soft    nofile  1048576" /etc/security/limits.conf

ENV TZ=America/Sao_Paulo

RUN ln -snf /usr/share/zoneinfo/${TZ} /etc/localtime && echo ${TZ} > /etc/timezone

VOLUME /data

WORKDIR ${TERRAMA2_INSTALL_PATH}
