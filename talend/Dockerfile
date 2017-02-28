FROM openjdk:8-jdk-alpine


LABEL build.publish.majorversion="8-openjdk"
LABEL build.publish.imagebase="alpine"

ARG AWSCLI_VER
ENV PATH .:$PATH

ONBUILD ARG TALENDJOB
ONBUILD ARG TALENDPREFIX
ONBUILD ARG TALENDJOBVER
ONBUILD ARG TALENDJOBVERUNDERSCORE

ONBUILD ENV TALENDJOB $TALENDJOB
ONBUILD ENV TALENDPREFIX $TALENDPREFIX
ONBUILD ENV TALENDJOBVER $TALENDJOBVER
ONBUILD ENV TALENDJOBVERUNDERSCORE $TALENDJOBVERUNDERSCORE


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

# unpack the talend jar
ONBUILD ADD talend/$TALENDPREFIX.tar.gz /etl/

# align it with the TALEND ETLROOT variable inside the job
# (That root does not know the version info)
# and use a symlink for the entrypoint script
ONBUILD RUN ln -s /etl/$TALENDPREFIX/$TALENDPREFIX/${TALENDPREFIX}_run.sh /etl/$TALENDPREFIX/$TALENDPREFIX/run_talend_job.sh

ONBUILD WORKDIR /etl/$TALENDPREFIX/$TALENDPREFIX

ENTRYPOINT ["run_talend_job.sh"]
