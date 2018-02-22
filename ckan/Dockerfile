# Multi-stage build is used to keep github/SSH credentials out of the image
FROM alpine as intermediate

RUN apk add --update git openssh

ARG SSH_PRIVATE_KEY
RUN mkdir -p /root/.ssh/
RUN echo "${SSH_PRIVATE_KEY}" > /root/.ssh/id_rsa
RUN chmod 400 /root/.ssh/id_rsa
RUN ssh-keyscan github.com >> /root/.ssh/known_hosts

RUN git clone git@github.com:thedataplace/ckanext-tdp.git  && \
    git clone -b static-uploader git@github.com:thedataplace/ckanext-cloudstorage.git


FROM ckan-base:2.7

MAINTAINER Gerben Neven <hello@gerritneven.nl>

# Install any extensions needed by your CKAN instance
# (Make sure to add the plugins to CKAN__PLUGINS in the .env file)
# For instance:
#RUN pip install -e git+https://github.com/ckan/ckanext-pages.git#egg=ckanext-pages && \
#    pip install -e git+https://github.com/ckan/ckanext-dcat.git@v0.0.6#egg=ckanext-dcat && \
#    pip install -r https://raw.githubusercontent.com/ckan/ckanext-dcat/v1.0.6/requirements.txt

# Copy cloned repo's over from the intermediate image
COPY --from=intermediate /ckanext-tdp /srv/app/ckanext-tdp
COPY --from=intermediate /ckanext-cloudstorage /srv/app/ckanext-cloudstorage

# Install extensions
RUN pip install -e git+file:///srv/app/ckanext-tdp#egg=ckanext-tdp \
    -e git+file:///srv/app/ckanext-cloudstorage@static-uploader#egg=ckanext-cloudstorage
# Install the extension(s) you wrote for your own project
# RUN pip install -e git+https://github.com/your-org/ckanext-your-extension.git@v1.0.0#egg=ckanext-your-extension
