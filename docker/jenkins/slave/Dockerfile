FROM openjdk:11-jdk

ADD ./data /opt
WORKDIR /opt

ENV GOSU_VERSION=1.11 \
    SWARM_VERSION=3.18 \
    MD5=09b86c29b9fd0479abe760bba9628a90 \
    COMPOSE_VERSION=1.25.4 \
    FINGER_PRINT=0EBFCD88

# grab gosu for easy step-down from root
RUN apt-get -y update \
    && apt-get install -y --no-install-recommends ca-certificates wget bzip2 python \
    && rm -rf /var/lib/apt/lists/* \
    && wget -O /usr/local/bin/gosu "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$(dpkg --print-architecture)" \
    && wget -O /usr/local/bin/gosu.asc "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$(dpkg --print-architecture).asc" \
    && export GNUPGHOME="$(mktemp -d)" \
    && GPG_KEYS=B42F6819007F00F88E364FD4036A9C25BF357DD4 \
    && gpg --keyserver hkp://:p80.pool.sks-keyservers.net:80 --recv-keys "$GPG_KEYS" \
    || gpg --keyserver ha.pool.sks-keyservers.net --recv-keys "$GPG_KEYS" \
    || gpg --keyserver pgp.mit.edu --recv-keys "$GPG_KEYS" \
    || gpg --keyserver keyserver.pgp.com --recv-keys "$GPG_KEYS" \
    && gpg --batch --verify /usr/local/bin/gosu.asc /usr/local/bin/gosu \
    && rm -rf "$GNUPGHOME" /usr/local/bin/gosu.asc \
    && chmod +x /usr/local/bin/gosu \
    && gosu nobody true

# grab swarm-client.jar
RUN mkdir -p /var/jenkins_home \
    && useradd -d /var/jenkins_home/slave -u 1000 -m -s /bin/bash jenkins \
    && curl -o /bin/swarm-client.jar -SL https://repo.jenkins-ci.org/releases/org/jenkins-ci/plugins/swarm-client/$SWARM_VERSION/swarm-client-$SWARM_VERSION.jar \
    && echo "$MD5  /bin/swarm-client.jar" | md5sum -c -

# get docker ce
RUN apt-get -y update \
    && apt-get -y install apt-transport-https ca-certificates curl gnupg-agent software-properties-common acl \
    && curl -fsSL https://download.docker.com/linux/debian/gpg | apt-key add - \
    && apt-key fingerprint $FINGER_PRINT \
    && add-apt-repository \
    "deb [arch=amd64] https://download.docker.com/linux/debian \
    $(lsb_release -cs) \
    stable" \
    && apt-get -y update \
    && apt-get -y install docker-ce docker-ce-cli containerd.io

# get docker compose
RUN curl -L "https://github.com/docker/compose/releases/download/$COMPOSE_VERSION/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose \
    && chmod +x /usr/local/bin/docker-compose

# post install to docker on linux
RUN usermod -aG docker jenkins

# start on boot
RUN systemctl enable docker

# get azure stuff
RUN apt-get update \
    && apt-get install lsb-release gnupg \
    && curl -sL https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/microsoft.asc.gpg > /dev/null \
    && add-apt-repository \
    "deb [arch=amd64] https://packages.microsoft.com/repos/azure-cli/ \
    $(lsb_release -cs) \
    main" \
    && tee /etc/apt/sources.list.d/azure-cli.list \
    && apt-get update \
    && apt-get install azure-cli

VOLUME /var/jenkins_home/slave
WORKDIR /var/jenkins_home/slave

ENTRYPOINT ["/opt/docker-entrypoint.sh"]
CMD ["java", "-jar", "/bin/swarm-client.jar"]