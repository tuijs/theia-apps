ARG NODE_VERSION=10
FROM ibmcom/ibmjava:8-sdk as java-base

FROM node:$NODE_VERSION
COPY --from=java-base /opt/ibm/java /opt/ibm/java

ENV JAVA_HOME=/opt/ibm/java \
    PATH=/opt/ibm/java/jre/bin:/opt/ibm/java/bin/:$PATH

RUN apt-get update

#Add native compilation dependencies (required by Scala native)
RUN apt-get install -y clang libgc-dev libunwind-dev

#Install RE2 regular expression engine (required by Scala native projects and has no deb available)
RUN git clone https://github.com/google/re2.git && \
  cd re2 && \
  make && \
  make test && \
  make install && \
  cd .. && \
  rm -rf re2

#Add basic dependencies
RUN apt-get install -y curl apt-transport-https zip

#Install bloop
ENV BLOOP_HOME=/opt/bloop \
    PATH=/opt/bloop:$PATH
RUN curl -L https://github.com/scalacenter/bloop/releases/download/v1.4.0-RC1/install.py | python - --dest $BLOOP_HOME

#Additional debian repos
RUN curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add - && \
  echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list && \
  echo "deb https://dl.bintray.com/sbt/debian /" | tee -a /etc/apt/sources.list.d/sbt.list && \
  apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 2EE0EA64E40A89B84B2DF73499E82A75642AC823 && \
  apt-get update

#Install language and plugin required debian dependencies
RUN apt-get install -y maven gradle sbt yarn

#Cleanup debian repo lists
RUN rm -rf /var/lib/apt/lists/*

# See : https://github.com/theia-ide/theia-apps/issues/34
RUN adduser --disabled-password --gecos '' theia
RUN chmod g+rw /home && \
    mkdir -p /home/project && \
    chown -R theia:theia /home/theia && \
    chown -R theia:theia /home/project;
WORKDIR /home/theia
USER theia

#Install sdkman
RUN curl -s "https://get.sdkman.io" | bash && \
    bash -c "source $HOME/.sdkman/bin/sdkman-init.sh && \
    rm -rf $HOME/.sdkman/archives/* && \
    rm -rf $HOME/.sdkman/tmp/*"

ARG version=latest
ADD $version.package.json ./package.json
ARG GITHUB_TOKEN
RUN yarn --cache-folder ./ycache && rm -rf ./ycache && \
    NODE_OPTIONS="--max_old_space_size=4096" yarn theia build ; \
    yarn theia download:plugins
EXPOSE 3000
ENV SHELL=/bin/bash \
    THEIA_DEFAULT_PLUGINS=local-dir:/home/theia/plugins
ENTRYPOINT [ "yarn", "theia", "start", "/home/project", "--hostname=0.0.0.0" ]
