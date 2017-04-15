FROM jruby:9.1-jre-alpine

LABEL build.publish.majorversion="9.1-jre-alpine"
LABEL build.publish.imagebase="jruby"

ARG AWSCLI_VER
ENV PATH .:$PATH

RUN apk add --no-cache \
      bash \
      curl \
      openssl \
      python \
      py-pip \
      tar \
      # Install aws cli
      # Install and upgrade Pip
      && pip install --upgrade pip  \
      # Install AWS cli
      && pip install awscli==${AWSCLI_VER}  \
      && apk --purge -v del py-pip

# Install biscuit
RUN curl  -s -L -o /tmp/biscuit.tar.gz \
     https://github.com/dcoker/biscuit/releases/download/v0.1.3/biscuit-linux_386.tgz && \
    tar zxf /tmp/biscuit.tar.gz -C /usr/local/bin && \
    rm /tmp/biscuit.tar.gz

# timout script
COPY timeout3 /usr/bin/timeout3
ENV TALEND_EXECUTABLE "timeout3 -t 3600 java"
COPY talend_run /usr/bin/talend_run

ENTRYPOINT ["talend_run"]

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
ONBUILD ENV TALENDJOBDIR $ETL_ROOT/$TALENDPREFIX/$TALENDPREFIX
ONBUILD ENV TALENDJARNAME ${TALENDPREFIXLCASE}_${TALENDJOBVERUNDERSCORE}
ONBUILD ENV TALENDJOBCONTEXTSDIR $TALENDJOBDIR/$TALENDPROJECT/$TALENDJARNAME/contexts
ONBUILD ENV TALENDJOBCLASS $TALENDPROJECT.$TALENDJARNAME.$TALENDPREFIX
ONBUILD ENV JAVA_CP "-cp .:$TALENDJOBDIR:$TALENDJOBDIR/../lib/*:$TALENDJOBDIR/${TALENDPREFIX}_${TALENDJOBVERUNDERSCORE}.jar:"

# debug print variables
ONBUILD RUN printf "project:      %s\n\
job:          %s\n\
prefix:       %s\n\
prefixlcase:  %s\n\
job version:  %s\n\
job _version: %s\n\
job dir     : %s\n\
context dir : %s\n\
job class   : %s\n\
jar name    : %s\n\
" $TALENDPROJECT $TALENDJOB $TALENDPREFIX $TALENDPREFIXLCASE $TALENDJOBVER \
  $TALENDJOBVERUNDERSCORE $TALENDJOBDIR $TALENDJOBCONTEXTSDIR $TALENDJOBCLASS \
  $TALENDJARNAME;

# unpack the talend jar
ONBUILD ADD talend/$TALENDPREFIX.tar.gz $ETL_ROOT
ONBUILD RUN chmod +x $TALENDJOBDIR/*.sh
# align it with the TALEND ETLROOT variable inside the job
# (That root does not know the version info)
# and use a symlink for the entrypoint script
ONBUILD RUN ln -s $TALENDJOBDIR/${TALENDPREFIX}_run.sh $TALENDJOBDIR/run_talend_job.sh

ONBUILD WORKDIR $TALENDJOBDIR
