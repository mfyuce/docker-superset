# VERSION 1.0 (apache-superset version:0.29.rc4)
# AUTHOR: Abhishek Sharma<abhioncbr@yahoo.com>
# DESCRIPTION: docker apache superset container
# BUILD: docker build --rm -t docker-apache-superset:0.28.1 -f docker-files/DockerFile .
# Modified/revamped version of the https://github.com/apache/incubator-superset/blob/master/contrib/docker/Dockerfile

ARG PY_VER=3.6.9
FROM python:${PY_VER}
MAINTAINER Abhishek Sharma <abhioncbr@yahoo.com>

# Build argument[version of apache-superset to be build: pass value while building image]
ARG SUPERSET_VERSION

ENV SUPERSET_HOME=/home/superset/
#ENV SUPERSET_DOWNLOAD_URL=https://github.com/apache/incubator-superset/archive/$SUPERSET_VERSION.tar.gz

# Add a normal superset group & user
# Change group & user id as per your requirement.
RUN groupadd -g 5006 superset
RUN useradd --create-home --no-log-init --uid 5004 --gid 5006 --home ${SUPERSET_HOME} --shell /bin/bash superset

# Configure environment
ENV LANG=C.UTF-8 \
    LC_ALL=C.UTF-8

RUN apt-get update -y
# Install dependencies to fix `curl https support error` and `elaying package configuration warning`
RUN apt-get install -y apt-transport-https apt-utils

# Install common useful packages
RUN apt-get install -y vim less curl netcat postgresql-client default-mysql-client redis-tools gdebi-core

#docker build was failing because of cryptography package failure wirl libssl-dev.
#instead of libssl-dev it is not set to `libssl1.0-dev`
RUN apt-get update -y && apt-get install -y build-essential libssl-dev \
    libffi-dev python3-dev libsasl2-dev libldap2-dev libxi-dev

RUN apt-get -y install dos2unix

# Install nodejs for custom build
# https://superset.incubator.apache.org/installation.html#making-your-own-build
# https://nodejs.org/en/download/package-manager/
#RUN curl -sL https://deb.nodesource.com/setup_10.x | bash -
#RUN apt-get install -y nodejs
#RUN curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add -; \
#    echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list; \
#    apt-get update; \
#    apt-get install -y yarn


RUN apt-get install -y --no-install-recommends ca-certificates \
&& wget http://security.debian.org/debian-security/pool/updates/main/f/firefox-esr/firefox-esr_68.7.0esr-1~deb8u1_amd64.deb \
&& gdebi -n firefox-esr_68.7.0esr-1~deb8u1_amd64.deb && rm firefox-esr_68.7.0esr-1~deb8u1_amd64.deb \
&& curl -L -c cookie.txt https://github.com/mozilla/geckodriver/releases/download/v0.24.0/geckodriver-v0.24.0-linux64.tar.gz | tar xz -C /usr/local/bin \
&& rm -rf /var/lib/apt/lists/*

WORKDIR $SUPERSET_HOME

# Download & install superset 0.29.rc4 version
#RUN wget -O superset.tar.gz $SUPERSET_DOWNLOAD_URL
#RUN tar -xzf superset.tar.gz -C $SUPERSET_HOME --strip-components=1 && rm superset.tar.gz
COPY ./superset $SUPERSET_HOME/

RUN find $SUPERSET_HOME/superset -name "*.sh" | xargs dos2unix

RUN mkdir -p /home/superset/.cache
RUN mkdir -p /home/superset/config
COPY docker-files/database-dependencies.txt .
RUN pip install --upgrade setuptools pip \
    && pip install -r requirements.txt -r requirements-dev.txt -r docker/requirements-extra.txt -r database-dependencies.txt \
    && pip install -e . \
    && rm -rf /root/.cache/pip

ENV PATH=${SUPERSET_HOME}/superset/bin:$PATH \
    PYTHONPATH=${SUPERSET_HOME}config/

# Install superset python packages
# RUN pip install --install-option="--prefix=$SUPERSET_HOME" superset==$SUPERSET_VERSION

COPY script/docker-entrypoint.sh /usr/local/bin/
RUN chmod +x /usr/local/bin/docker-entrypoint.sh
RUN dos2unix  /usr/local/bin/docker-entrypoint.sh
RUN ln -s /usr/local/bin/docker-entrypoint.sh /entrypoint.sh # backwards compat

# copy superset_condfig.py file
COPY config/superset_config.py ${SUPERSET_HOME}config/

RUN chown -R superset:superset $SUPERSET_HOME

USER superset

#RUN cd superset/assets && yarn
#RUN cd superset/assets && npm ci && npm run build && rm -rf node_modules

HEALTHCHECK CMD ["curl", "-f", "http://localhost:8088/health"]
ENTRYPOINT ["docker-entrypoint.sh"]
EXPOSE 8088 5555

