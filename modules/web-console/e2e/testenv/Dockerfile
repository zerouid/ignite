#
# Licensed to the Apache Software Foundation (ASF) under one or more
# contributor license agreements.  See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.
# The ASF licenses this file to You under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with
# the License.  You may obtain a copy of the License at
#
#      http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#

FROM ubuntu:14.04

ENV NPM_CONFIG_LOGLEVEL info
ENV NODE_VERSION 8.6.0

# Before package list update.
RUN set -ex  && \
      for key in \
        9554F04D7259F04124DE6B476D5A82AC7E37093B \
        94AE36675C464D64BAFA68DD7434390BDBE9B9C5 \
        FD3A5288F042B6850C66B31F09FE44734EB7990E \
        71DCFD284A79C3B38668286BC97EC7A07EDE3FC1 \
        DD8F2338BAE7501E3DD5AC78C273792F7D83545D \
        B9AE9905FFD7803F25714661B63B535A4C206CA9 \
        C4F0DFFF4E8C1A8236409D08E73BC641CC11F4C8 \
        56730D5401028683275BD23C23EFEFE93C4CFFFE \
      ; do \
        gpg --keyserver ha.pool.sks-keyservers.net --recv-keys "$key" || \
        gpg --keyserver pgp.mit.edu --recv-keys "$key" || \
        gpg --keyserver keyserver.pgp.com --recv-keys "$key" ; \
    done

# Update package list & install.
RUN apt-get update && \
    apt-get install -y nginx-light curl xz-utils git dos2unix

# Install Node JS.
RUN curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/node-v$NODE_VERSION-linux-x64.tar.xz"  && \
  curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/SHASUMS256.txt.asc" && \
  gpg --batch --decrypt --output SHASUMS256.txt SHASUMS256.txt.asc && \
  grep " node-v$NODE_VERSION-linux-x64.tar.xz\$" SHASUMS256.txt | sha256sum -c - && \
  tar -xJf "node-v$NODE_VERSION-linux-x64.tar.xz" -C /usr/local --strip-components=1 && \
  rm "node-v$NODE_VERSION-linux-x64.tar.xz" SHASUMS256.txt.asc SHASUMS256.txt

# Install global node packages.
RUN npm install -g pm2

# Install frontend & backend apps.
RUN mkdir -p /opt/web-console

# Copy source.
WORKDIR /opt/web-console
COPY frontend ./frontend
COPY backend ./backend


# Install node modules.
RUN cd /opt/web-console/frontend && npm install --no-optional --prod && npm run build
RUN cd /opt/web-console/backend && npm install --only=production --no-optional

# Returns to base path.
WORKDIR /opt/web-console

# Copy nginx config.
COPY ./e2e/testenv/nginx/nginx.conf /etc/nginx/nginx.conf
COPY ./e2e/testenv/nginx/web-console.conf /etc/nginx/web-console.conf

# Setup entrypoint.
COPY ./e2e/testenv/entrypoint.sh .
RUN chmod 755 /opt/web-console/entrypoint.sh && dos2unix /opt/web-console/entrypoint.sh

# Clean up.
RUN apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

EXPOSE 9001

ENTRYPOINT ["/opt/web-console/entrypoint.sh"]
