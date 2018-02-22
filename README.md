# TDP CKAN Docker setup

The `tdp` branch contains our dev setup & docker image files for production.

The `master` branch is to be kept in sync with the upstream. 

See instructions for development use below.
TODO: add prod build steps

------------------------------------------
# Docker Compose setup for CKAN

**Note**: :warning: This is a work in progress. There is likely to be issues so use with caution :warning:


This is a set of Docker images and configuration files to run a CKAN site

It is largely based on two existing projects:

* Keitaro's [CKAN Docker images](https://github.com/keitaroinc/docker-ckan)
* Docker Compose setup currently included in [CKAN core](https://github.com/ckan/ckan)


It includes the following images, all based on [Alpine Linux](https://alpinelinux.org/):

* CKAN: modified from keitaro/ckan (see [CKAN Images](#ckan-images)) for more details)
* DataPusher: modified from keitaro/datapusher
* PostgresSQL: mdillon's PostGIS image
* Solr: official Solr image with CKAN's schema
* Redis: standard Redis image

The site is configured via env vars (the base CKAN image loads [ckanext-envvars](https://github.com/okfn/ckanext-envvars)), that you can set in the `.env` file.

Copy the included `.env.example` and rename it to `.env` to modify it depending on your own needs.

Using the default values on the `.env.example` file will get you a working CKAN instance. There is a sysadmin user created by default with the values defined in `CKAN_SYSADMIN_NAME` and `CKAN_SYSADMIN_PASSWORD`(`ckan_admin` and `test` by default). I shouldn't be telling you this but obviously don't run any public CKAN instance with the default settings.

To build the images:

	docker-compose build

To start the containers:

	docker-compose up

## Development mode

To develop local extensions use the `docker-compose.dev.yml` file:

To build the images:

	docker-compose -f docker-compose.dev.yml build

To start the containers:

	docker-compose -f docker-compose.dev.yml up

See [CKAN Images](#ckan-images) for more details of what happens when using development mode.


## CKAN images

```
    +-------------------------+                +----------+
    |                         |                |          |
    | openknowledge/ckan-base +---------------->   ckan   | (production)
    |                         |                |          |
    +-----------+-------------+                +----------+
                |
                |
    +-----------v------------+                 +----------+
    |                        |                 |          |
    | openknowledge/ckan-dev +----------------->   ckan   | (development)
    |                        |                 |          |
    +------------------------+                 +----------+
    
    
```

The Docker images used to build your CKAN project are located in the `ckan/` folder. There are two Docker files:

* `Dockerfile`: this is based on `openknowledge/ckan-base` (with the `Dockerfile` on the `ckan-base/` folder), an image with CKAN with all its dependencies, properly configured and running on [uWSGI](https://uwsgi-docs.readthedocs.io/en/latest/) (production setup)
* `Dockerfile.dev`: this is based on `openknowledge/ckan-dev` (with the `Dockerfile` on the `ckan-dev/` folder), wich extends `openknowledge/ckan-base` to include:

  * Any extension cloned on the `src` folder will be installed in the CKAN container when booting up Docker Compose (`docker-compose up`). This includes installing any requirements listed in a `requirements.txt` (or `pip-requirements.txt`) file and running `python setup.py develop`.
  * The CKAN image used will development requirements needed to run the tests .
  * CKAN will be started running on the paster development server, with the `--reload` option to watch changes in the extension files.
  * Make sure to add the local plugins to the `CKAN__PLUGINS` env var in the `.env` file.

From these two base images you can build your own customized image tailored to your project, installing any extensions and extra requirements needed.
