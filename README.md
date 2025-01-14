---
title: Matomo OpenShift
description: Documentation and resources required to deploy a Matomo web analytics server instance into an OpenShift environment.  Matomo is a fully featured web analytics server and is a great alternative to Google Analytics when data ownership and privacy compliance are a concern.
author: esune
resourceType: Components
personas: 
  - Developer
  - Product Owner
  - Designer
labels:
  - matomo
  - google
  - analytics
  - web
---
# Matomo OpenShift
Matomo is a fully featured web analytics server and is a great alternative to Google Analytics when data ownership and privacy compliance are a concern.

[Matomo OpenShift](https://github.com/BCDevOps/matomo-openshift) provides a set of OpenShift configurations to set up an instance of the Matomo web analytics server. See: [matomo.org](https://matomo.org/) for additional details regarding Matomo.

## Architecture
The service is composed by the following components:
- *matomo*: includes two containers within a single pod, the matomo analytics instance and matomo-proxy, which is the nginx used to proxy http requests.
- *matomo-db*: a [mariadb](https://mariadb.org) instance that will be used to store the analytics data.

## Deployment / Configuration
The templates provided in the `openshift` folder include everything that is necessary to create the required builds and deployments.

Since there are interdependencies between deployment configurations, please make sure to follow this order when creating them for the first time:
1) build and deploy the database
2) build and deploy the Matomo analytics server and proxy

Note: The TAG_NAME in the matomo-deploy is used for both the proxy and matomo which is annoying. That's why there is the rename from 3.11.0-fpm to latest

```
oc new-app -p NAME=mariadb -p OUTPUT_IMAGE_TAG=latest -f mariadb/mariadb-build.json
oc new-app -p NAME=matomo -p OUTPUT_IMAGE_TAG=latest -p SOURCE_IMAGE_TAG=3.11.0-fpm -f matomo/matomo-build.json 

oc new-app -p NAME=matomo-proxy OUTPUT_IMAGE_TAG=latest -f matomo-proxy/matomo-proxy-build.json

oc new-app -p IMAGE_NAMESPACE=matomo -p TAG_NAME=latest -p NAME=matomo-db -p PERSISTENT_VOLUME_CLASS=glusterfs-storage -f matomo-db/matomo-db-deploy.json 

oc new-app -p NAME=matomo -p IMAGE_NAMESPACE=matomo -p TAG_NAME=latest -p PERSISTENT_VOLUME_CLASS=glusterfs-storage -f matomo/matomo-deploy.json
```

## First Run
Once everything is up and running in OpenShift, follow the [instructions](https://matomo.org/docs/installation/#the-5-minute-matomo-installation) to create your superuser, set-up the connection to the database and initialize the Matomo dashboard.

To start tracking, copy the snippet for the appropriate website in the Matomo dashboard and place it in your website.
