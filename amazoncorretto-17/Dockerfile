#
# First stage just acquires the distribution archive,
# verifies and unpacks it.
#
FROM alpine:3 AS fetcher

RUN apk add bash coreutils libxslt wget gpg

ARG MDA_RELEASE=1.0.0

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
RUN yum -y install git opensc openssl softhsm sudo wget which && \
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
# Let the user modify the doc subdirectory and in particular
# the examples within doc/wiki
#
RUN chown -R user:user /opt/mda/doc

#
# By default, run a login shell.
#
CMD ["/bin/bash", "/etc/bootstrap.sh"]
