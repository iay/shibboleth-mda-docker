#
# First stage just acquires the distribution archive,
# verifies and unpacks it.
#
FROM alpine:3 AS fetcher

RUN apk add bash coreutils libxslt wget gpg

ARG MDA_RELEASE=0.10.0

WORKDIR /opt
ADD fetch-mda-snapshot .
ADD snapshot-version.xsl .
RUN ./fetch-mda-snapshot
WORKDIR /opt/fetched/mda-dist
RUN chmod -R +r .
RUN ln -s mda.sh mda
RUN chmod +rx mda.sh mda

#
# Main build stage is the delivered environment.
#
FROM amazoncorretto:17

LABEL maintainer="Ian Young <ian@iay.org.uk>"

#
# Pull in commonly required tools.
#
RUN yum -y install git sudo wget which && \
    yum clean all

#
# Default GPG agent configuration.
#
ADD gpg-agent.conf /etc/gnupg/

#
# Bash profile.
#
ADD profile.d/ /etc/profile.d/

#
# Install latest version of Maven.
#
WORKDIR /opt
COPY --from=fetcher /opt/fetched/mda-dist /opt/mda
#RUN wget --no-verbose https://archive.apache.org/dist/maven/maven-3/${mavenver}/binaries/apache-maven-${mavenver}-bin.tar.gz && \
#    tar xfz apache-maven-${mavenver}-bin.tar.gz && \
#    rm *.tar.gz

#
# Bootstrap script.
#
ADD bootstrap.sh /etc/bootstrap.sh

#
# Do all our work as a normal user, in a home directory.
#
RUN adduser user
ENV userdir=/home/user
WORKDIR ${userdir}

#
# By default, run a login shell.
#
CMD ["/bin/bash", "/etc/bootstrap.sh"]
