FROM registry.access.redhat.com/ubi9/ubi-minimal:9.3

USER root

ENV MAVEN_URL=https://archive.apache.org/dist/maven/maven-3/3.9.5/binaries/apache-maven-3.9.2-bin.tar.gz \
    GRAAL_CE_URL=https://github.com/graalvm/mandrel/releases/download/mandrel-23.1.0.0-Final/mandrel-java21-linux-amd64-23.1.0.0-Final.tar.gz \
    GRAALVM_HOME=/opt/mandrelJDK \
    HOME=/home/developer \
    APP_HOME=/home/developer \
    MAVEN_CONFIG=/home/developer/.m2

RUN microdnf -y install bash tar gzip unzip which findutils wget git bzip2 java-21-openjdk-devel gcc gcc-c++ libstdc++ libstdc++-devel glibc-static glibc-devel zlib-devel && \
    mkdir -p ${HOME} /usr/share/maven && \
    curl -fsSL ${MAVEN_URL} | tar -xzC /usr/share/maven --strip-components=1 && \
    mkdir -p ${GRAALVM_HOME} && \
    cd ${GRAALVM_HOME} && \
    curl -fsSL $GRAAL_CE_URL | tar -xzC ${GRAALVM_HOME} --strip-components=1 && \
    ln -s /usr/share/maven/bin/mvn /usr/bin/mvn && \
    microdnf -y clean all && rm -rf /var/cache/yum

RUN mkdir -p /usr/libexec/s2i ${MAVEN_CONFIG}

# create user
RUN rm -rf /tmp/* && \
    useradd -u 1001 -G root -d ${HOME} --shell /bin/bash developer && \
    chown -R 1001:0 ${HOME} && chmod 770 ${HOME} && chmod 770 ${HOME}/.m2

# copy s2i
COPY s2i /usr/libexec/s2i
LABEL io.openshift.s2i.scripts-url="image:///usr/libexec/s2i/"

# copy maven setup
COPY configure-maven.sh ${MAVEN_CONFIG}/
COPY configure.sh ${MAVEN_CONFIG}/
COPY settings.xml ${MAVEN_CONFIG}/

USER 1001
ENV PATH ${PATH}:${GRAALVM_HOME}/bin
WORKDIR ${HOME}

CMD ["/usr/libexec/s2i/usage"]
