#
# First stage just acquires, verifies and unpacks
# the distribution archives.
#
FROM alpine:3 AS fetcher

RUN apk add bash coreutils libxslt wget gpg gpg-agent

ARG MDA_RELEASE=0.10.0
ARG XMLSECTOOL_RELEASE=3.0.0

#
# Acquire, verify and extract the xmlsectool distribution.
#
# The distribution is unpacked into /opt/xmlsectool
#
WORKDIR /opt
ADD fetch-xmlsectool .
ADD PGP_KEYS .
RUN ./fetch-xmlsectool
RUN unzip xmlsectool-${XMLSECTOOL_RELEASE}-bin.zip
RUN mv xmlsectool-${XMLSECTOOL_RELEASE} xmlsectool
WORKDIR /opt/xmlsectool
RUN chmod -R +r .
RUN ln -s xmlsectool.sh xmlsectool
RUN chmod +rx xmlsectool.sh xmlsectool

#
# Acquire, verify and extract the MDA distribution.
#
# The distribution is unpacked into /opt/fetched/mda-dist.
#
WORKDIR /opt
ADD fetch-mda .
RUN ./fetch-mda
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
# Install latest version of MDA and xmlsectool.
#
WORKDIR /opt
COPY --from=fetcher /opt/fetched/mda-dist /opt/mda
COPY --from=fetcher /opt/xmlsectool       /opt/xmlsectool

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
