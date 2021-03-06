# Multi-stage build is used to keep github/SSH credentials out of the image
FROM alpine as intermediate

RUN apk add --update git openssh

ARG SSH_PRIVATE_KEY
RUN mkdir -p /root/.ssh/
RUN echo -n "${SSH_PRIVATE_KEY}" | base64 -d > /root/.ssh/id_rsa
RUN chmod 400 /root/.ssh/id_rsa
RUN ssh-keyscan github.com >> /root/.ssh/known_hosts

RUN git clone git@github.com:thedataplace/ckanext-cloudstorage.git && \
  git clone git@github.com:thedataplace/ckanext-pcc.git && \
  git clone git@github.com:thedataplace/ckanext-tdp.git


FROM gcr.io/dataplace-181911/ckan-base:2.7.4

MAINTAINER Gerben Neven <hello@gerritneven.nl>

# Install any extensions needed by your CKAN instance
# (Make sure to add the plugins to CKAN__PLUGINS in the .env file)
# For instance:
#RUN pip install -e git+https://github.com/ckan/ckanext-pages.git#egg=ckanext-pages && \
#    pip install -e git+https://github.com/ckan/ckanext-dcat.git@v0.0.6#egg=ckanext-dcat && \
#    pip install -r https://raw.githubusercontent.com/ckan/ckanext-dcat/v1.0.6/requirements.txt

RUN apk add --update git openssh libffi-dev
COPY config.ini production.ini
COPY wsgi.py wsgi.py
RUN apk add tzdata && \
  ln -fs /usr/share/zoneinfo/Etc/UTC /etc/localtime

RUN mkdir -p /packages/lib/python2.7/site-packages

# Copy cloned repo's over from the intermediate image
COPY --from=intermediate /ckanext-cloudstorage /srv/app/ckanext-cloudstorage
COPY --from=intermediate /ckanext-pcc /srv/app/ckanext-pcc
COPY --from=intermediate /ckanext-tdp /srv/app/ckanext-tdp

# Install extension
RUN pip install pycrypto \
  -e git+file:///srv/app/ckanext-tdp@v0.2.5#egg=ckanext-tdp \
  -e git+file:///srv/app/ckanext-pcc@0.1.0#egg=ckanext-pcc \
  -e git+file:///srv/app/ckanext-cloudstorage@0.0.2#egg=ckanext-cloudstorage \
  google-cloud-storage==1.7.0 \
  -e git+https://github.com/okfn/ckanext-spatial.git#egg=ckanext-spatial \
  -e git+https://github.com/thedataplace/ckanext-geoview.git@v0.0.12-patched-defs#egg=ckanext-geoview \
  -e git+https://github.com/ckan/ckanext-harvest.git#egg=ckanext-harvest

RUN pip install \
  -r src/ckanext-spatial/pip-requirements.txt \
  -r /srv/app/ckanext-tdp/requirements.txt


RUN pip install -r src/ckanext-harvest/pip-requirements.txt
RUN cd src/ckanext-harvest && python setup.py develop

# Install the extension(s) you wrote for your own project
# RUN pip install -e git+https://github.com/your-org/ckanext-your-extension.git@v1.0.0#egg=ckanext-your-extension
