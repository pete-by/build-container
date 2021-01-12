ARG MAVEN_FOLDER=apache-maven
ARG MAVEN_HOME=/usr/local/$MAVEN_FOLDER
ARG HELM_HOME=/usr/local/helm

FROM ubuntu:20.04 as base

MAINTAINER peter@axamit.com

ARG MAVEN_FOLDER
ARG MAVEN_HOME
ARG MAVEN_VERSION=3.6.3
ARG MAVEN_FILENAME=apache-maven-${MAVEN_VERSION}-bin.tar.gz

ARG HELM_HOME
ARG HELM_VERSION=v3.4.2
ARG HELM_FILENAME=helm-${HELM_VERSION}-linux-amd64.tar.gz

#
#  Install and configure Maven.
#
WORKDIR /usr/local

# Install wget
RUN  apt-get update \
  && apt-get install -y wget \
  && rm -rf /var/lib/apt/lists/*

# Turn on verbose, but print basic info and errors
RUN wget -nv https://apache.mirror.colo-serv.net/maven/maven-3/$MAVEN_VERSION/binaries/$MAVEN_FILENAME \
&& tar xzf $MAVEN_FILENAME
# Create a soft link
RUN ln -s apache-maven-$MAVEN_VERSION $MAVEN_FOLDER

# Copy default settings.xml into Maven's conf folder
COPY settings.xml $MAVEN_HOME/conf/settings.xml

RUN wget -nv https://get.helm.sh/$HELM_FILENAME \
    && tar xzf $HELM_FILENAME && ls -a  \
    && mkdir -p $HELM_HOME/bin && cp ./linux-amd64/helm $HELM_HOME/bin/helm

# Use JDK (note it can be distroless)
FROM adoptopenjdk/openjdk11:jdk-11.0.9.1_1-centos as jdk

RUN yum install sudo -y && \
    yum install git -y

ARG MAVEN_HOME
ARG HELM_HOME
ARG USER=jenkins
ARG USERGROUP=jenkins

# Set environment variables
ENV M2_HOME=$MAVEN_HOME
ENV MAVEN_HOME=$MAVEN_HOME
ENV MAVEN_REPO=/home/jenkins/.m2

COPY --from=base $MAVEN_HOME $MAVEN_HOME
COPY --from=base $HELM_HOME $HELM_HOME

ENV PATH=$MAVEN_HOME/bin:$HELM_HOME/bin:$PATH

#
# add the AWS cli to the container
#
# RUN pip install awscli

#
#  Add the jenkins user so the artifacts created from within the container by Jenkins can be created as the user jenkins.
#
# WORKDIR /
# RUN adduser jenkins

#
# Make the entry-point script executable.
#
WORKDIR /

COPY docker-entrypoint.sh /usr/local/bin/

RUN mkdir /home/$USER \
    && groupadd -r $USERGROUP -g 1000 && useradd -d /home/$USER -u 1000 -r -g $USERGROUP $USER \
    && chmod +x /usr/local/bin/docker-entrypoint.sh \
    && chown -R $USER:$USERGROUP /home/$USER \
    && chown $USER:$USERGROUP /usr/local/bin/docker-entrypoint.sh \
    && chown -R $USER:$USERGROUP $MAVEN_HOME

USER ${USER}

ENTRYPOINT ["docker-entrypoint.sh"]

CMD ["echo","Build container is ready"]
