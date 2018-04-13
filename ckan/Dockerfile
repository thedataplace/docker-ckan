# Multi-stage build is used to keep github/SSH credentials out of the image
FROM alpine as intermediate

RUN apk add --update git openssh

ARG SSH_PRIVATE_KEY
RUN mkdir -p /root/.ssh/
RUN echo "${SSH_PRIVATE_KEY}" > /root/.ssh/id_rsa
RUN chmod 400 /root/.ssh/id_rsa
RUN ssh-keyscan github.com >> /root/.ssh/known_hosts

RUN git clone -b 0.0.2 git@github.com:thedataplace/ckanext-cloudstorage.git && \
    git clone git@github.com:thedataplace/ckanext-exeter.git  && \
    git clone git@github.com:thedataplace/ckanext-libraries.git  && \
    git clone git@github.com:thedataplace/ckanext-pcc.git  && \
    git clone -b v0.1.2 git@github.com:thedataplace/ckanext-tdp.git


FROM ckan-base:2.7.3

MAINTAINER Gerben Neven <hello@gerritneven.nl>

# Install any extensions needed by your CKAN instance
# (Make sure to add the plugins to CKAN__PLUGINS in the .env file)
# For instance:
#RUN pip install -e git+https://github.com/ckan/ckanext-pages.git#egg=ckanext-pages && \
#    pip install -e git+https://github.com/ckan/ckanext-dcat.git@v0.0.6#egg=ckanext-dcat && \
#    pip install -r https://raw.githubusercontent.com/ckan/ckanext-dcat/v1.0.6/requirements.txt

RUN apk add --update git openssh
COPY config.ini production.ini
COPY wsgi.py wsgi.py

RUN mkdir -p /packages/lib/python2.7/site-packages
ENV PYTHONUSERBASE /packages

# Copy cloned repo's over from the intermediate image
COPY --from=intermediate /ckanext-cloudstorage /srv/app/ckanext-cloudstorage
COPY --from=intermediate /ckanext-exeter /srv/app/ckanext-exeter
COPY --from=intermediate /ckanext-libraries /srv/app/ckanext-libraries
COPY --from=intermediate /ckanext-pcc /srv/app/ckanext-pcc
COPY --from=intermediate /ckanext-tdp /srv/app/ckanext-tdp

# Install extension
RUN pip install pycrypto \
    -e git+file:///srv/app/ckanext-exeter#egg=ckanext-exeter \
    -e git+file:///srv/app/ckanext-libraries#egg=ckanext-libraries \
    -e git+file:///srv/app/ckanext-tdp@v0.1.2#egg=ckanext-tdp \
    -e git+file:///srv/app/ckanext-pcc@0.1.0#egg=ckanext-pcc \
    -e git+file:///srv/app/ckanext-cloudstorage@0.0.2#egg=ckanext-cloudstorage \
    google-cloud-storage==1.7.0 \
    -e git+https://github.com/okfn/ckanext-spatial.git#egg=ckanext-spatial \
    -e git+https://github.com/ckan/ckanext-geoview.git@v0.0.13#egg=ckanext-geoview \
    -r /srv/app/ckanext-tdp/requirements.txt

RUN pip install \
    -r src/ckanext-spatial/pip-requirements.txt \
    -r src/ckanext-geoview/pip-requirements.txt

# Install the extension(s) you wrote for your own project
# RUN pip install -e git+https://github.com/your-org/ckanext-your-extension.git@v1.0.0#egg=ckanext-your-extension
