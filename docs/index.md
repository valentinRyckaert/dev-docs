# Presentation

## What is TeSS?

Before talking about HEP Training, we need to talk about TeSS, the open-source project which HEP Training is based on.

Training e-Support Service (TeSS) is a web application coded in Ruby On Rails. Its objective is to bring training materials from various sources into one place, so people can find and access to them easily. Moreover, TeSS provide various elements relative to theses materials such as events, collections, workflows, spaces etc...

If you are intressted in TeSS and get involved into it, you will also hear about mTeSS-X, a "sub-project" of TeSS which added some functionalities to the application.


## What is HEP Training?

HEP Training is built on the top of the TeSS project. This plateform is used inside CERN by teams working on High Energy Physics (HEP).

Three of the four expermients within the LHC has a team working on HEP. Their work often rely on complex IT tools, and they need to share documentations, courses and tutorials on those. HEP Training is the webstie in which they can register these training materials.


## Technical overview

The TeSS project uses multiple tools. You can find the definition of each of them on the [Dictionnary](dictionnary) page.

#### Backend
- Ruby On Rails (framework)
- PostgreSQL (database)

#### Frontend
- Embedded Ruby (ERB - template system)
- SCSS (style)
- vanilla Javascript
- Handlebars.js

#### Authentication
- Ruby On Rails built-in system or OIDC
- sendmail (subscribtions, reset password...)

#### Contenerization
- Docker

#### Tests
- Ruby On Rails bult-in test suite
- Github Actions

#### Code Documentation
- Rdoc

#### Deployment (HEP Training only)
- Kubernetes
- Helm
- OKD (open-source version of OpenShift)
