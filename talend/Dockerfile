FROM jruby:9.2-jre8

LABEL build.publish.majorversion="9.2"
LABEL build.publish.imagebase="jre8"

ARG AWSCLI_VER
ENV PATH .:$PATH

# Consul Template version
ENV CONSUL_TEMPLATE_VERSION 0.19.4

# Envelope version
ENV ENVELOPE_VERSION 0.4.0

# Install dependencies including f360 certs and envelope
RUN apt-get update \
  && apt-get install -y --no-install-recommends \
  tini \
  gnupg \
  awscli \
  && rm -rf /var/lib/apt/lists/* \
  && curl -s -o /usr/local/share/ca-certificates/full360-root-ca.crt \
    https://s3.amazonaws.com/full360-cloudformation-init/public/full360-root-ca.crt \
  && update-ca-certificates \
  && curl -s -o /tmp/envelope.zip \
    https://releases-full360.s3.amazonaws.com/public/envelope/${ENVELOPE_VERSION}/envelope_${ENVELOPE_VERSION}_linux_amd64.zip \
  && unzip /tmp/envelope.zip -d /usr/local/bin/ \
  && rm /tmp/envelope.zip \
  # Install biscuit
  && curl  -s -L -o /tmp/biscuit.tar.gz \
    https://github.com/dcoker/biscuit/releases/download/v0.1.3/biscuit-linux_386.tgz \
  && tar zxf /tmp/biscuit.tar.gz -C /usr/local/bin \
  && rm /tmp/biscuit.tar.gz \
  # Install consul-template
  && curl -s -o /tmp/consul-template.zip \
    https://releases.hashicorp.com/consul-template/${CONSUL_TEMPLATE_VERSION}/consul-template_${CONSUL_TEMPLATE_VERSION}_linux_amd64.zip \
  && unzip /tmp/consul-template -d /usr/local/bin/ \
  && rm /tmp/consul-template.zip

# Add talend helper scripts
# This script makes sure that if a job gets stuck it will be restarted
COPY timeout3 /usr/bin/timeout3
ENV TALEND_EXECUTABLE "timeout3 -t 3600 java"
# This script runs the talend job with pre and post hooks
COPY talend_run /usr/bin/talend_run
# This script runs adds the block file to stop the job gracefully
COPY talend_block /usr/bin/talend_block

COPY .gnupg /root/.gnupg
RUN chown -R root:root /root/.gnupg \
  && chmod 600 /root/.gnupg ; chmod 600 /root/.gnupg/*

# Use tini - https://github.com/krallin/tini
# To manage the background calls and zombies
ENTRYPOINT ["/usr/bin/tini", "-g", "--", "envelope", "run", "talend_run"]

# On Build
ONBUILD ARG ETL_ROOT=/etl
ONBUILD ARG TALENDPROJECT
ONBUILD ARG TALENDJOB
ONBUILD ARG TALENDPREFIX
ONBUILD ARG TALENDPREFIXLCASE
ONBUILD ARG TALENDJOBVER
ONBUILD ARG TALENDJOBVERUNDERSCORE
ONBUILD ARG TALENDJARNAME
ONBUILD ARG JAVA_ARGS="-Xms256M -Xmx1024M"

ONBUILD ENV ETL_ROOT $ETL_ROOT
ONBUILD ENV TALENDPROJECT $TALENDPROJECT
ONBUILD ENV TALENDJOB $TALENDJOB
ONBUILD ENV TALENDPREFIX $TALENDPREFIX
ONBUILD ENV TALENDPREFIXLCASE $TALENDPREFIXLCASE
ONBUILD ENV TALENDJOBVER $TALENDJOBVER
ONBUILD ENV TALENDJOBVERUNDERSCORE $TALENDJOBVERUNDERSCORE
ONBUILD ENV JAVA_ARGS $JAVA_ARGS

# composite env variables
ONBUILD ENV TALENDROOTDIR $ETL_ROOT/$TALENDPREFIX
ONBUILD ENV TALENDJOBDIR $TALENDROOTDIR/$TALENDPREFIX
ONBUILD ENV TALENDJARNAME ${TALENDPREFIXLCASE}_${TALENDJOBVERUNDERSCORE}
ONBUILD ENV TALENDJOBCONTEXTSDIR $TALENDJOBDIR/$TALENDPROJECT/$TALENDJARNAME/contexts
ONBUILD ENV TALENDJOBCLASS $TALENDPROJECT.$TALENDJARNAME.$TALENDPREFIX
ONBUILD ENV JAVA_CP "-cp .:$TALENDJOBDIR:$TALENDJOBDIR/../lib/*:$TALENDJOBDIR/${TALENDPREFIXLCASE}_${TALENDJOBVERUNDERSCORE}.jar:"
ONBUILD ENV ENVMAP_DIR /root/envmaps

# debug print variables
ONBUILD RUN printf "project:      %s\n\
job:          %s\n\
prefix:       %s\n\
prefixlcase:  %s\n\
job version:  %s\n\
job _version: %s\n\
root dir    : %s\n\
job dir     : %s\n\
context dir : %s\n\
job class   : %s\n\
jar name    : %s\n\
" $TALENDPROJECT $TALENDJOB $TALENDPREFIX $TALENDPREFIXLCASE $TALENDJOBVER \
  $TALENDJOBVERUNDERSCORE $TALENDROOTDIR $TALENDJOBDIR $TALENDJOBCONTEXTSDIR \
  $TALENDJOBCLASS $TALENDJARNAME;

# unpack the talend jar
ONBUILD ADD talend/$TALENDPREFIX.tar.gz $ETL_ROOT
ONBUILD RUN chmod -f +x $TALENDROOTDIR/bin/*.sh

ONBUILD WORKDIR $TALENDJOBDIR
