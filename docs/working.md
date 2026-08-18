# Working on HEP Training

This page presents the infrastructure around HEP Training: what are the repositories, the third-party dependencies and the CERN environment which HEP Training is part of.

## GitHub organization
_[https://github.com/hep-training](https://github.com/hep-training)_

HEP Training has a Github organization. You will find there four repositories:

- [TeSS](https://github.com/hep-training/TeSS): the HEP Training web application
- [user-docs](https://github.com/hep-training/user-docs): the user documentation for HEP Training
- [dev-docs](https://github.com/hep-training/dev-docs): the developer documentation you are currently reading
- [helm-chart](https://github.com/hep-training/helm-chart): a Helm Chart and a CLI to deploy HEP Training on OKD

TeSS, user-docs and helm-chart are all a fork of their TeSS counterparts. They have a "main" branch, which is the same as the parent repository, and a "heptraining" branch, which is the production one. This orgnization helps to have a reference of the parent repository of each code.

On the other hand, dev-docs has only a "main" branch since it is not a fork of a TeSS project.

## CERN infrastructure

At CERN, HEP Training is surrounded by four main components:

- [OKD](./dictionnary.md#okd-openshift): the instance which HEP Training is deployed on
- GMS: the Group Management System of the CERN where the groups for private spaces are fetched
- [CERN SSO](./dictionnary.md#cern-sso): the system for authentification on HEP Training
- Application Portal: hosted by CERN, contains an application which is the proxy for HEP Training in order to fetch the GMS API
- DBOD